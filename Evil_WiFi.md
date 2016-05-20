# SoftAP WiFi
Software based Access Point 

## Introduction
We are going to use a Raspberry Pi and an Edimax WiFi dongle to create WiFi with any name and using MITMProxy will monitor data communication that clients are doing.

## Tools
We are going to use below hardware and software tools.

### Hardware
+ Raspberry Pi 2 - Model B
+ Edimax EW-7811Un

Q: How to confirm if the WiFi hardware is suitable for the purpose?
A: Connect the hardware, and follow below steps.

First run `iwconfig` and confirm the device name; in my case it was wlan0.

Then run 'iw wlan0' and looks for **Capability**. 
It should have **AP Mode** in the list of supported modes.

If *AP Mode* is not in the list, the WiFi hardware will not be able to work as a WiFi access point. 

A thing that confused me for days, I was using Alfa AWUS036H and was able to see the access point in list of available access points in my mobile phone, however, there was no communication happening when I connected my phone to the WiFi access point. So even if you are able to see the access point in the available WiFi access point list, that does not mean it is going to work.



### Software
+ Raspbian
+ IPTables
+ hostapd
+ udhcpd
+ MITMProxy

### Steps

#### Setup Raspbian
Download and install Raspbian in an SD Card following the steps in this [link](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) to setup the SD card.

Once the SD card is ready insert the card in the RPi and also connect the Edimax dongle in one of the USB ports. For internet access I connected the RPi to my router using a LAN cable, but you are free to use any other method like via a Internet dongle from cell phone providers.

Once everything is connected, connect the RPi power plug and boot it.

I used SSH to connect to the RPi but you are free to access anyother method you prefer.

In case, you want to use SSH, you can either use your router config application (generally first IP of your home network like 192.168.0.1 or 192.168.1.1) to find the IP assigned to the RPi or use nmap command
`nmap -Pn -p 22 192.168.1.0/24`

The command tells nmap to look for open 22 port (SSH) and report any found. The output from nmap will tell you the IP address and the device type which should be *Raspbery Pi Foundation*.

Once found you can connect to the RPi using below command from PuTTY or console
`ssh pi@192.168.1.2`

#### Configure WiFi hardware
Once you are logged in, assign static IP address to the WiFi card. This is required so that when we configure our DHCP server, the IP our card which will act as a router will remain same. If not it will confuse the clients which will not know whom to send traffic to reach internet.

####  Compatible Hostapd
This is the most confusing and frustrating part of the whole process. The incompatibility of different drivers and other open source softwares which uses those driver are brutally confusing sometimes.

Follow the steps in **_Prerequisites_** section in [this great tutorial](http://www.daveconroy.com/turn-your-raspberry-pi-into-a-wifi-hotspot-with-edimax-nano-usb-ew-7811un-rtl8188cus-chipset/) by Dave Conroy.

Download the compatible hostapd (for rtl871xdrv drivers) for Edimax from this [link](http://www.daveconroy.com/wp3/wp-content/uploads/2013/07/hostapd.zip).

#### udhcpd

#### hostapd

#### IPTables
Allow traffic from wlan0 to b forwarded to eth0

`$ iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT`

Allow traffic from eth0 to wlan0 only if related

`$ iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT`

Do IP Masquerade after traffic routed to eth0

`$ iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`

Redirect all traffic destined to port 80 and 443 to our proxy server port 8080

`$ iptables -t nat -A PREROUTING -i wlan0 -p tcp —dport 80 -j REDIRECT —to-port 8080`

`$ iptables -t nat -A PREROUTING -i wlan0 -p tcp —dport 443 -j REDIRECT —to-port 8080`









