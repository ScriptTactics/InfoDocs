# SDR++ Server

[SDR++](https://www.sdrpp.org/) is the bloat-free SDR receiver. It is limited in functionality but it can get you off the ground and running quickly with an SDR.

The server version of this application has been containerized using docker and is available on my [GitHub](https://github.com/ScriptTactics/sdrppserver_container).


# Setup

You will need the following.

1. A Linux, Windows, or Mac machine capable of installing Docker. (Note the architecture must be 64-bit x86)
2. A second machine (optional) capable of installing the SDR++ client software.
3. A SDR dongle. This guide uses the [RTL_SDR](https://www.amazon.com/RTL-SDR-Blog-RTL2832U-Software-Defined/dp/B0BMKB3L47/ref=sr_1_1?keywords=RTL-SDR%2BBlog&qid=1676645722&sr=8-1&th=1). 

# Design

Next we will go over the archtecture design of how everything connects to each other. Both machine must be on the same network*


*(You can create a VPN connection, ZeroTier connection, or reverse proxy to allow both machines to connect to each other while not being on the same physical network. This guide will not cover how to do that.)

## Machine 1

This machine will be running the [SDR++ Server Docker container](https://github.com/ScriptTactics/sdrppserver_container). It will also have the [RTL_SDR](https://www.amazon.com/RTL-SDR-Blog-RTL2832U-Software-Defined/dp/B0BMKB3L47/ref=sr_1_1?keywords=RTL-SDR%2BBlog&qid=1676645722&sr=8-1&th=1) connected to it.

## Machine 2

This machine will have the SDR++ Client software installed from [SDR++](https://www.sdrpp.org/).


# Machine 1

Linux: Debian/Ubuntu


## Docker Setup

The following set of instructions will go over installing Docker on the machine.

### Install gnome terminal
```bash
sudo apt install gnome-terminal
```

### Setup Docker Repository

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

```

2. Add Docker's official GPG key:
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

3. Use the following command to set up the repository:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine

1. Update the apt package index:
```bash
sudo apt update
```

2. Install Docker Engine, containerd, and Docker Compose.
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
3.Verify that the Docker Engine installation is successful by running the hello-world image:
```bash
sudo docker run hello-world
```

### Docker Engine post-installation steps
To create the `docker` group and add your user:

1. Create the `docker` group.
```bash
sudo groupadd docker
```
2. Add your user to the `docker` group.
```bash
sudo usermod -aG docker $USER
```
You can also run the following command to activate the changes to groups:
```bash
newgrp docker
```
4. Verify that you can run `docker` commands without `sudo`.
```bash
docker run hello-world
```
## SDR++ Server Setup

First connect the RTL_SDR to Machine 1.

Then install git if its not already installed
```bash
sudo apt install git
```
Then clone the [repo](https://github.com/ScriptTactics/sdrppserver_container.git)

Navigate inside the folder:
```bash
cd sdrppserver_container
```
Once inside the folder verify the RTL_SDR is connected by running:

```bash
lsusb
```

Once you see the device ID you will have to modify the start script.

Run this edit the script:
```bash
sudo nano runcontainer.sh
```

Then modify the following line:

```bash
--device=<RTL_SDR>
```

The full script should then look like this:

```bash
#!/bin/bash

 docker run -d -p 5259:5259 --restart unless-stopped --name='sdrppserver' --device=<RTL_SDR> --volume=/home/$USER/sdrpp:/config sdrppserver_local
```

Press Ctl+x, then y to save and exit.

Next type:

```bash
sudo chmod +x runcontainer.sh
```

After that command type:

```bash
./runcontainer.sh
```

If there are no errors you should see a container ID appear on the screen.

Type
```bash
docker ps
```
To see the running container

# Machine 2

Install SDR++ for your respective operating system. Make sure you select nightly build.

Extract the .zip and then launch the application.

In Source select `SDR++ Server`.

Then set the IP to your linux server ip.
