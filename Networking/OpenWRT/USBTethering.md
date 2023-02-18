# Smartphone USB Tethering
All steps are pulled from [OpenWRT Docs](https://openwrt.org/docs/guide-user/network/wan/smartphone.usb.tethering)

## Installation

For the easiest installation, have a wired upstream internet connection to boot-strap this process. You will need: the router, your tethering phone, necessary cables, a laptop and an upstream internet connection via Ethernet for initial setup. Instead of a wired upstream connection to plug into the router WAN port, is also possible to download necessary packages below, through your laptop while tethered to your phone, the same way you can get the OpenWrt distribution for your router.

A common protocol for Android based devices for tethering via USB is RNDIS:

```
opkg update
opkg install kmod-usb-net-rndis
```

However Android devices come with great diversity, therefor some require different protocols. For instance newer devices may use NCM, others may require EEM or even need a subset implementation.

NOTE: Try the following protocol(s) if you don't have a `usb0` interface come up or device keeps disconnecting:

```
opkg install kmod-usb-net-cdc-ncm

# Huawei may need its own implementation!
opkg install kmod-usb-net-huawei-cdc-ncm

# More protocols:
kmod-usb-net-cdc-eem
kmod-usb-net-cdc-ether
kmod-usb-net-cdc-subset
```
Extra steps depending on USB type and drivers for your router:
```
opkg update
opkg install kmod-nls-base kmod-usb-core kmod-usb-net kmod-usb-net-cdc-ether kmod-usb2
```
Additional steps for iOS devices:
```
opkg update
opkg install kmod-usb-net-ipheth usbmuxd libimobiledevice usbutils

# Call usbmuxd
usbmuxd -v
 
# Add usbmuxd to autostart
sed -i -e "\$i usbmuxd" /etc/rc.local
```
If you need to manually download the packages on another device for bootstrapping, see get_additional_software_packages. The Kernel modules will be in the URL of form downloads.openwrt.org/releases/[your release]/targets/[your target]/generic/packages/ and other packages (iOS stuff in this case) in downloads.openwrt.org/releases/[release]/packages/[instruction set]/packages/.
## 2. Smartphone

Connect the smartphone to the USB port of the router with the USB cable, and then enable USB Tethering from the Android settings. Turn on the phone's Developer Options [Find the Build information in the About Phone menu, and tap rapidly 7 x]. There is a Default USB Configuration: USB Tethering option. The phone will now immediately turn on USB Tethering mode when plugged into a configured router [or laptop], without further commands. However, it is necessary to remove the screen lock on the phone. A locked phone will not start USB Tethering by itself.

For iPhones, you may have to disable and re-enable the Personal Hotspot/Allow Others to Join setting on the iPhone to force the OpenWrt DHCP client to get an IP address from the eth1 iPhone interface. Disabling and re-enabling the Personal Hotspot/Allow Others to Join setting on the iPhone is also required if you disconnect the iPhone from the OpenWrt USB port and re-connect it later, unless you cache Trust records (see watchdog section and/or LeJeko's Github repository in reference section).

iPhones starting from iOS 11 will terminate the USB data connections after one hour by default to improve security. This can easily be changed via:

Settings > Touch ID/Face ID & Passcode > USB Accessories > ON (macworld)
## 3.a Command-line interface

On the router, enter:
```
# Enable tethering
uci set network.wan.ifname="usb0"
uci set network.wan6.ifname="usb0"
uci commit network
/etc/init.d/network restart
```
For iPhones, replace the interface name usb* with eth* depending on router.

It should be all working at this point. To activate wireless connections to the router, go to Network, Wireless and set then enable the interfaces.
3.b Web interface

Go to Network, Interfaces. You can either assign the existing WAN to usb0 like 3.a above, or create a whole new interface if you want to swap between the WAN Ethernet port and your tethering device (such as in a dual-wan fail-over situation). To make changes in the web interface equivalent to the above command line instructions: simply edit the existing default WAN interface, and change the physical device to usb0, then Save & Apply.

Instead, to create a whole new interface, make a new one called TetheringWAN, and bind to it the new *usb0* network device (restart if you do not see it yet. And, for some cases, the new interface may be called '*eth1*: check what the log is showing in your case). Set the protocol to DHCP client mode or DHCPv6 client mode if the ISP assigns IPv6, and under the Firewall Settings tab, place it into the WAN zone. Save changes.

## 3.b Web interface

Go to Network, Interfaces. You can either assign the existing WAN to usb0 like 3.a above, or create a whole new interface if you want to swap between the WAN Ethernet port and your tethering device (such as in a dual-wan fail-over situation). To make changes in the web interface equivalent to the above command line instructions: simply edit the existing default WAN interface, and change the physical device to usb0, then Save & Apply.

Instead, to create a whole new interface, make a new one called TetheringWAN, and bind to it the new *usb0* network device (restart if you do not see it yet. And, for some cases, the new interface may be called '*eth1*: check what the log is showing in your case). Set the protocol to DHCP client mode or DHCPv6 client mode if the ISP assigns IPv6, and under the Firewall Settings tab, place it into the WAN zone. Save changes.

See the following screenshots.

![Image](Networking\OpenWRT\Screenshots\image_create_new_interface.png?raw=true)

First page of the Create Interface wizard.

![Image2](Networking\OpenWRT\Screenshots\image_create_new_interface_set_firewall_region.png?raw=true)
Firewall tab of the Create Interface Wizard. Very important to set it as WAN.

![Image3](Networking\OpenWRT\Screenshots\image_create_new_interface_end_result.png?raw=true)
And the end result in the Interfaces page.

After committing the changes the new TetheringWAN should be activated. Otherwise, restart it with the buttons you find in the Interface page of LuCI web interface. 

# Modifying TTL on OpenWRT Routers
## Case 1: Standard routed setup
Information pulled from this [doc](https://www.maroonmed.com/ttl-modification-for-outgoing-traffic-with-openwrt/)

Scenario: Standard routed setup with separate LAN and WAN (tether) interfaces
1. Using the OpenWRT package manager via LuCI or opkg CLI, install the iptables-mod-ipopt package.
2. Navigate to **Network → Firewall → Custom Rules**.
3. Add the following line: 
```
iptables -t mangle -I POSTROUTING -o usb0 -j TTL --ttl-set 65
```
4. Click **Restart Firewall** to save

If necessary, change usb0 to wlan0 (or whichever interface name corresponds to your tether interface.)
