---
title: "Deploy Pihole on RaspberryPi ZeroW to adblock your home network"
date: 2021-02-10T20:42:44+11:00
draft: false 
tags: ["RaspberryPi", "Pi hole", "AdBlocking","Tracker Blocking", "Networking"]
---

## Problem
We all have seen and annoyed seeing those google ads or facebook ads on websites and apps. Mostly of the times the ads over takes the actual content of the website, especially on those forums. Ads also shows up on free email services, apps on your phones, smart tv, they are everywhere. More than annoyance, they eat up a lot of bandwidth and ofter makes the website/apps slower to respond. There are ad blockers available for your browsers, but that just solves the problem on that device, but devices like smart tv doesn't have an easy way to block ad on the device.

## Solution
The one solution that can remove ads from every device on your home network is to install an ad blocker to your network. Simple and cheap yet effective solution is to use a free, open-source software called Pihole on your network. Here is how pihole works, you need to install pihole on a device on your device and channel all your internet traffic from your network to pihole. Pihole cross references the request from your device with the list of ad servers that it has an if it finds a  match it blocks the request, else it will let the request through to thn internet and you get served with the website that you were looking for. By block the ad requests, you browser or apps does not get the ad content, hence does not show an ad or shows a blank space. More about [Pihole here](https://pi-hole.net/).

## Installation
Pihole software is so light weight that it can be installed on a single board computer like raspberry pi. I have chosen the lowest Pi available Raspberry Pi Zero W, which has a built in wifi support.

**Things you need for below setup**
  - Raspberry Pi Zero W
  - Micro SD card 16gb
  - MicroSD Card reader
  - USB power supply for Pi Zero W

**Step 1 Installing OS**
Pi requires an OS that can run pihole software. Pihole supports a lot of linux distribution, i choose Raspbian OS (now known as Raspberry PI OS). We will need a micro SD card, atleast 16GB. Plug the micros sd card into a computer using a card reader and download the raspberry pi imager from [raspberrypi.org](https://www.raspberrypi.org/software/) and use the software to flash the micro-sd card with the OS. For more advanced users the website gives a manual installation option too.

**Step 2 Enable Remote access and SSH**
We need to configure Pi zero W to access the pizero w over wifi and to access it using SSH to install software and access pihole UI. While the sd card is plugged in to the computer (you might have to remove and add the sd card after installation of OS) you need to do two things
 - Add a file named *ssh* (without any extension) to the drive or partition called boot on your SD Card. This will enable SSH connection to your PI.
 - Add a file named *wpa_supplicant.conf* to the same boot partition with the below content
    ```
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    country=<Insert 2 letter ISO 3166-1 country code here>

    network={
        ssid="<Name of your wireless LAN>"
        psk="<Password for your wireless LAN>"
    }
    ```
now the SD card can be installed into the raspberry pi and connect raspberry to USB power. Once powered on you can use SSH to connect into the Pi terminal. Default username is *pi* and password is *raspberry*

**Step 3 Install Pihole**
Installing Pihole is as easy as typing in the command *curl -sSL https://install.pi-hole.net | bash* into the pi terminal and run through the steps. Alternate ways and further configuration guide to installation can be found at [Pihole github page](https://github.com/pi-hole/pi-hole/#one-step-automated-install). You can also setup the password for pihole admin console.

**Step 4 Hooking up Pi to the network**
Once the pihole is installed and Pi is up and running, configure your router to have the Pi configured with static ip address within your network. 
Next step is to configure your routers DNS server Ip address to your PIs static IP address. This means the all the requests from devices in your network will be sent to the PI by the router for DNS resolution. 
Pihole will get these requests and will block those request to domains that is marked as adserver in its list.
If you type in the IP address of the pi onto your browser of any device on the network, it will load the pihole page and you can load the admin page which shows the request from devices and requests that were blocked in a beautiful dashboard, like below.

![Pihole dashboard](/blogimages/piholedashboard.png)

You can even login to the admin account to further view detailed data and configure a lot more on your pihole. More about configuration and setup can be found at [pihole documentation](https://docs.pi-hole.net/)

This setup solves ad problems on your home or private network, or the network that uses the router that has Pi configured as DNS server. If you want ad blocking on your even outside your network then you can even configure VPN on your pihole and your device. More about [VPN connection can be found on pihole documentation](https://docs.pi-hole.net/guides/vpn/openvpn/overview/)

