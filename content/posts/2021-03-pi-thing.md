---
title: "Raspberry Pi temperature monitor"
date: 2021-03-13T19:00:00Z
draft: false
---

I want to monitor the temperature in all the rooms in my house. 

**Why?** I like comfort and efficiency. If I know the temperature of each room in my house, with periodic measurements twenty four hours a day, I can tune the heating system to be comfortable and energy efficient. Because I'm a bit of a geek, this is classed as 'fun' which means I don't need to offset the cost of the project against my heating bill.

**What?** I want a way to have a log of the temperature in each room of my house at regular intervals without having to be there in person, or - for that matter - be awake.

**How?** Well, that's the question, isn't it? My starting assumption is that I need a sensor in every room. This rules out a single smart thermostat which might learn my habits, but would be woefully under-informed about the temperatures across my house. 

Contents:

* [Priorities](#-priorities)
* [Purchases](#-purchases)
* [microSD OS image](#-microsd-os-image)
* [#First boot](#-#first-boot)
* [Configure with Ansible](#-configure-with-ansible)

# Priorities

I've had lots of recommendations for good approaches. There are trade-offs to be made for each:

* Battery vs mains
* Portability
* Low maintenance

Some applications need the freedom to put the sensors anywhere. My preference is to have a system that I maintain (in person) about once a year. I'm happy to give up a power socket per room to get that, and have the sensor within reach of that power socket.

* Wifi vs bluetooth vs Z-Wave vs Zigbee vs ethernet
* Dedicated device vs DIY mashup
* Arduino vs Raspberry Pi
* Afinity to my skills

I've done my time writing assembler and embedded C (for now, at least). I learnt both at university, and applied those skills in my first job after my degree. I've helped write I<sup>2</sup>C libraries and (different job) an X10 module in Python. I think I've got the apptitude to use any of those options. ￼But do I want to?

* Alignment to my areas of interest

I'm working in the wonderful world of DevOps, where Python and Ansible and Prometheus are regularly in the spotlight. I'm a manager in my day-job, so I *don't* actually get to play with the toys (except spreadsheets). It'd be cool to have a solution that uses those.

* Price per unit: Cheaper is better, as long as the fun-factor is maintained.
* Development time: Quicker is better. There aren't many fun-project-hours in a given month.
* Availablity of equipment: If it needs a lathe, a 3D printer or an underground network of tunnels, it's out.

## Arduino approach
| Cost | Thing |
|------|-------|
|£20 |Arduino uno / £6 Arduino nano|
|£5	 |Real time clock board/Data logging shield|
|£6	 |Temperature sensor|
|£8	 |Bluetooth module??|

Total: £39 per unit. I reckon that component list wouldn't work - the nano and uno have different form factors, the bluetooth has to talk to ... something, but it gives me a place to start. I've heard I get a WiFi-enabled-Arduino-compatible board from China for cheaps. So that's an option.

## Raspberry Pi approach

| Cost | Thing |
|------|-------|
|£10 |Raspberry Pi Zero W|
|£4  |DS18B20 temperature sensor|
|£0  |4.7kohm resistor - free with sensor|
|£1  |40 pin header|
|£6  |USB PSU|
|£3  |Case|
|£3  |Shipping|

Total: £27 per unit

Blocker: I can order only one Raspberry Pi Zero W per customer? What madness is this?

I'm going to give the Raspberry Pi route a try. It hits a lot of the sweet spots, and it'll be fun - even if I can't source 10 RPi Zero Ws because of the bizarre sales limits.

# Purchases

Here's what I'm starting out with.

Central control server:

* Raspberry Pi 4
* [microSD card](https://www.jeffgeerling.com/blog/2019/raspberry-pi-microsd-card-performance-comparison-2019)
* [Case](https://thepihut.com/products/flirc-raspberry-pi-4-case)
* Pi 4 official power supply

Temperature sensor pi:

* Waterproof DS18B20 Digital temperature sensor + extras
* Raspberry Pi Zero W
* microSD card
* Case
* Micro-USB power supply


# microSD OS image

The first order of business is to get a process for bootstrapping the Raspberry Pi Zeros. 

## Install the official Raspberry Pi Imager 
...from https://www.raspberrypi.org/software/.

(You could use this zsh script I wrote to do that, but honestly it seems like a lot of trouble to go to.)

```zsh
IMAGER="/Applications/Raspberry Pi Imager.app"
# Install RPi imager software if it's not already available
if [[ ! -a $IMAGER ]]; then
    IMAGER_DOWNLOAD="https://downloads.raspberrypi.org/imager/imager_1.5.dmg"
    wget $IMAGER_DOWNLOAD
    DISKIMAGE=$(hdiutil attach ./imager_1.5.dmg | grep 'Raspberry' | cut -f1)
    cp -R "/Volumes/Raspberry Pi Imager/Raspberry Pi Imager.app" /Applications/
    hdiutil detach $DISKIMAGE
fi
```

## Transfer the OS image to the microSD card

![Raspberry Pi Imager](/static/static/posts/2021-03-pi-thing-pi-imager.png)

1. Open Raspberry Pi Imager
2. Plug in microSD card
3. Choose `Raspberry Pi OS Lite (32-bit)`
4. Choose microSD card
5. CHoose Write
6. MacOS is likely to ask you for various permissions for the imager
7. The imaging unmounts the SD card, so unplug it and plug it in again

## Prepare the OS image for first boot

The first boot will be headless (no keyboard/monitor) and wireless (i.e. I want it to connect to my Wifi with no further configuration nesessary). The Pi OS image checks for a few additional files in the /boot directory on first boot. That enables me to enable SSH and WiFi.

From a terminal, with the microSD card plugged back in...

`cd /Volumes/boot`

Enable SSH:

`touch ssh`

Enable and configure WiFi:

```zsh
cat << EOF > wpa_supplicant.conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="YOUR_SSID"
    psk="YOUR_WIFI_PASSWORD"
    key_mgmt=WPA-PSK
}
EOF
```

Eject the microSD card safely using Finder, and unplug it.

# #First boot

1. Assemble the Raspberry Pi Zero in the case
2. Insert the microSD card
3. Run `arp -a` to list the devices connected to your network
4. Plug the power supply into the wall and plug the lead into the microUSB connector (it's the one closest to the corner of the board).
5. Wait a minute or two for the Pi to boot
6. Run `arp -a` and spot the difference. The host name for me is `raspberrypi.myhomedomain`
<!-- 
# Configure with Ansible

Step zero for Ansible is to run a playbook on the Pi to provide it with it's long-term hostname and configure ssh correctly.

`ansible-playbook -i hosts site.yml --user <user> --ask-pass -vvvv`
 -->
To be continued...