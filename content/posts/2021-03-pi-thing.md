---
title: "Raspberry Pi temperature monitor"
date: 2021-03-13T19:00:00Z
draft: false
summary: |
  I want to monitor the temperature in all the rooms in my house. 
  Why? I like comfort and efficiency. If I know the temperature of each room in my house, with periodic measurements...
---

I want to monitor the temperature in all the rooms in my house.

**Why?** I like comfort and efficiency. If I know the temperature of each room in my house, with periodic measurements twenty four hours a day, I can tune the heating system to be comfortable and energy efficient. Because I'm a bit of a geek, this is classed as 'fun' which means I don't need to offset the cost of the project against my heating bill.

**What?** I want a way to have a log of the temperature in each room of my house at regular intervals without having to be there in person, or - for that matter - be awake.

**How?** Well, that's the question, isn't it? My starting assumption is that I need a sensor in every room. This rules out a single smart thermostat which might learn my habits, but would be woefully under-informed about the temperatures across my house.

Contents:

* [Priorities](#priorities)
* [Purchases](#purchases)
* [microSD OS image](#microsd-os-image)
* [#First boot](#first-boot)
* [Configure with Ansible](#configure-with-ansible)

## Priorities

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

### Arduino approach

| Cost | Thing |
|------|-------|
|£20 |Arduino uno / £6 Arduino nano|
|£5 |Real time clock board/Data logging shield|
|£6 |Temperature sensor|
|£8 |Bluetooth module??|

Total: £39 per unit. I reckon that component list wouldn't work - the nano and uno have different form factors, the bluetooth has to talk to ... something, but it gives me a place to start. I've heard I get a WiFi-enabled-Arduino-compatible board from China for cheaps. So that's an option.

### Raspberry Pi approach

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

## Purchases

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

## microSD OS image

The first order of business is to get a process for bootstrapping the Raspberry Pi Zeros.

### Install the official Raspberry Pi Imager

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

### Transfer the OS image to the microSD card

![Raspberry Pi Imager](/static/posts/2021-03-pi-thing-pi-imager.png)

1. Open Raspberry Pi Imager
2. Plug in microSD card
3. Choose `Raspberry Pi OS Lite (32-bit)`
4. Choose microSD card
5. CHoose Write
6. MacOS is likely to ask you for various permissions for the imager
7. The imaging unmounts the SD card, so unplug it and plug it in again

### Prepare the OS image for first boot

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

## #First boot

1. Assemble the Raspberry Pi Zero in the case
2. Insert the microSD card
3. Run `arp -a` to list the devices connected to your network
4. Plug the power supply into the wall and plug the lead into the microUSB connector (it's the one closest to the corner of the board).
5. Wait a minute or two for the Pi to boot
6. Run `arp -a` and spot the difference. The host name for me is `raspberrypi.myhomedomain`

## Configure with Ansible

There are a number of bootstrap tasks to perform that will let me connect to the Pi with Ansible in a consistent way.

* Create SSH keys
* Create a new user
* Enable SSH access for that user with that key and remove its password
* Enable passwordless sudo for that user
* Set the hostname
* Clean up (delete) the default user

Ideally, I'd like to complete those tasks automatically with Ansible too, but I don't know how yet, so in the interim:

### Create pubic-private keypair

```zsh
$ ssh-keygen -t rsa -f ./rpi-key
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ./rpi-key.
Your public key has been saved in ./rpi-key.pub.
The key fingerprint is:
SHA256:JRdQyqJfuDglJ4r+SqsdFXIF8FlXSi9QAE3ZY2nHXnU ak@Ashpool
The key's randomart image is:
+---[RSA 3072]----+
|  ..o=*==*+  .. E|
|   . +o=*+o..  . |
|  . = .o*++.     |
|   o o o =.      |
|    = + S        |
| . o B o         |
|..o o o          |
|o... .           |
|o++.             |
+----[SHA256]-----+

$ ls rpi-key*
rpi-key     rpi-key.pub

```

### Log in using SSH

```zsh
$ ssh pi@raspberrypi.myhomedomain
The authenticity of host 'raspberrypi.myhomedomain (172.80.43.99)' can't be established.
ECDSA key fingerprint is SHA256:6ryJBtgVc6k1liVATfBBXnapErFLAArbhVhwoFu7aw0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'raspberrypi.myhomedomain,172.80.43.99' (ECDSA) to the list of known hosts.
pi@raspberrypi.myhomedomain's password:
Linux raspberrypi 5.4.83+ #1379 Mon Dec 14 13:06:05 GMT 2020 armv6l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ $
```

### Create a new user

```zsh
pi@raspberrypi:~ $ sudo adduser robot
Adding user `robot' ...
Adding new group `robot' (1001) ...
Adding new user `robot' (1001) with group `robot' ...
Creating home directory `/home/robot' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for robot
Enter the new value, or press ENTER for the default
  Full Name []: Ansible Robot
  Room Number []:
  Work Phone []:
  Home Phone []:
  Other []:
Is the information correct? [Y/n] y
pi@raspberrypi:~ $
```

### Give that user passwordless sudo access

```zsh
pi@raspberrypi:~ $ sudo adduser robot sudo
Adding user `robot' to group `sudo' ...
Adding user robot to group sudo
Done.
pi@raspberrypi:~ $

sudo visudo /etc/sudoers.d/010_robot-nopasswd
```

Enter `robot ALL=(ALL) NOPASSWD: ALL`

Save and quit the editor

Test:

```zsh
pi@raspberrypi:~ $ logout
Connection to raspberrypi.myhomedomain closed.

$ ssh robot@raspberrypi

robot@raspberrypi:~ $ sudo whoami
root
```

### Give robot user same groups as pi

```zsh
sudo cat /etc/group | grep pi

sudo usermod -G adm,dialout,cdrom,audio,video,plugdev,games,users,input,netdev robot
```

### Authorise public key for that user

From my dev laptop:

```zsh
$ ssh-copy-id -i ./rpi-key.pub robot@raspberrypi.myhomedomain
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "./rpi-key.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
robot@raspberrypi.myhomedomain's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'robot@raspberrypi.myhomedomain'"
and check to make sure that only the key(s) you wanted were added.

$ ssh robot@raspberrypi.myhomedomain -i ./rpi-key
Linux raspberrypi 5.4.83+ #1379 Mon Dec 14 13:06:05 GMT 2020 armv6l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
robot@raspberrypi:~ $
```

### Remove password for user

```zsh
robot@raspberrypi:~ $ sudo passwd robot -d
passwd: password expiry information changed.
robot@raspberrypi:~ $
```

### Remove default user 'pi'

```zsh
robot@raspberrypi:~ $ sudo deluser --remove-home pi
Looking for files to backup/remove ...
Removing files ...
Removing user `pi' ...
Warning: group `pi' has no more members.
Done.
robot@raspberrypi:~ $
```

### Set hostname

```zsh
robot@raspberrypi:~ $ echo "mon-livingroom" | sudo tee /etc/hostname
mon-livingroom

robot@raspberrypi:~ $ sudo sed -i 's/raspberrypi/mon-livingroom/g' /etc/hosts

robot@raspberrypi:~ $ sudo reboot

$ ssh robot@mon-livingroom.myhomedomain -i ./rpi-key
The authenticity of host 'mon-livingroom.myhomedomain (172.80.43.99)' can't be established.
ECDSA key fingerprint is SHA256:6ryJBtgVc3k1liVATeBBXnapErFLAArbhVhwoFu7aw0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'mon-livingroom.myhomedomain' (ECDSA) to the list of known hosts.
Linux mon-livingroom 5.4.83+ #1379 Mon Dec 14 13:06:05 GMT 2020 armv6l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Mar 26 22:05:57 2021 from 172.80.41.127
robot@mon-livingroom:~ $
```

### Ansible bit

This is what I want set up on my Raspberry Pi:

* Daily automatic security updates
* Reboots as needed (or weekly)

Services to set up:

* Prometheus

Application metrics to collect:

* Hostname/location
* Temperature

System metrics to collect:

<!-- 
`ansible-playbook -i hosts site.yml --user <user> --ask-pass -vvvv`

 -->
To be continued...
