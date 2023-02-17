# TAK Server Install

Instructions for installing TAK Server on CentOS 7

# CentOS
First you will need the [CentOS](http://isoredirect.centos.org/centos/7/isos/x86_64/) ISO (CentOS 7). Setup either a VM or install on baremetal.

Follow the prompts on the install, be sure to enable your networking on the install screen, and also set the install to be "infrastructure server".

Be sure to create an admin password and make the user you create an admin.

# TAKServer
Once your CentOS server is setup update the packages.

`sudo yum update -y && sudo yum upgrade -y`

Then install 

`sudo yum install epel-release -y`


Make sure git is installed

`sudo yum install git -y`

then clone the TakServer repo

`git clone https://github.com/TAK-Product-Center/Server.git`

You will also need to make sure Java 11 is installed. (JDK & JRE)

`sudo yum install java-11-openjdk-devel -y`


You will also need to install patch

`sudo yum install patch -y`

As well as Postgres

`sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y`

Once those are installed you can build the project.


## Build Project

Navigate into the src directory and clean and build the project

```
cd Server/src
./gradlew clean bootWar
```

If that completes you are ready to move on.

Run the following:

`./gradlew clean buildRpm`

This will generate the rpm image for you.


## Install RPM

Next navigate to the following directory

`cd Server/src/takserver-package/takserver/build/distributions`

In here you should see the server rpm:

`takserver-<version>-RELEASE<Number>.noarch.rpm`

At the time of writing this my rpm is `takserver-4.5-RELEASE72.noarch.rpm`

Run `sudo yum install takserver-4.5-RELEASE72.noarch.rpm` to install the server.


## Setup DB

There is a db install script pre-made that you will have to run.

`sudo ./opt/tak/db-utils/takserver-setup-db.sh`


## Reload Service

After the db setup script is complete you can reload the services

`sudo systemctl daemon-reload`

At this point you can set TAK Server to start at boot

`sudo systemctl enable takserver`


# Certificates

First you will have to become the `tak` user that is created.

`sudo su tak`

Then create env variables:
```
export STATE=NY
export CITY=NYC
export ORGANIZATION=my-organizaton
export ORGANIZATIONAL_UNIT=my-unit
```

Navigate to 
`/opt/tak/certs/`

Then run 
`./makeRootCa.sh`

Give a name for your CA: example-name

Create a server certificate:

`./makeCert.sh server takserver`


## Client Certs

For each client that you want on your network copy the following command and change `user` to the user you want to add: ex -> `Alpha`.

`./makeCert.sh client user`

## Admin UI Cert

Generate an admin cert to gain access to the admin UI.

`./makeCert.sh client admin`


## Reload

After you have created the certs restart the TAK Server.

`sudo systemctl restart takserver`

Then authorize the admin cert.

`sudo java -jar /opt/tak/utils/UserManager.jar certmod -A /opt/tak/certs/files/admin.pem`


Also, the generated CA trustores and certs will be here:

`/opt/tak/certs/files`



# Firewall
Setup the following firewall rules:
```
sudo firewall-cmd --permanent --zone=public --add-port 8089/tcp
sudo firewall-cmd --permanent --zone=public --add-port 8443/tcp
sudo firewall-cmd --reload
```

After reloading the firewall check that the ports are opened by running:

`sudo firewall-cmd --list-ports`

The output should look like this
```
8089/tcp 8443/tcp
```

# Web UI

In order to access the webUI you will need to download the admin certificate that you created in the previous step.

You can do this a number of ways.

1. SFTP
2. SCP
3. FileZilla 


Choose whichever is best for you. 
The file you are looking for is here:

`/opt/tak/certs/files/admin.p12`

Once you have this cert you will have to import it in your browser.

Firefox:
Settings -> Preferences -> Privacy & Security -> Certificates -> View Certificates

Select `Your Certificates` and import the downloaded cert. 

The password is `atakatak`

Then navigate to:

`https://yourip:8443/`


## (Optional) Create Admin Credentials

Create Login Credentials for local admin account:
```
sudo java -jar /opt/tak/utils/UserManager.jar usermod -A -p <password> <username>
```


# Setup Wizard

Secure: https://yourip:8443/setup/

Insecure with user/pass: http://yourip:8080/setup


# Other Configurations

After running through the wizard you may want to disable port 8080.

`sudo nano /opt/tak/CoreConfig.xml`

then remove
```
<connector port="8080" tls="false" _name="http_plaintext"/>
```

save the changes and restart tak server.

`sudo systemctl restart takserver`

