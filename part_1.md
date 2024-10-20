This file describes how to set up an alpine linux server from intitial installation, until it can be disconnected from monitor and keyboard, and managed via a network connection.

# Overview
| Part | Description & Link|
| ----------- | ----------- |
| 00 | [Overview and ReadMe](https://github.com/pubbelfiesch/SAHS/) |
| 01 | [Alpine installation until the server can be remotely managed via SSH](part_1.md) | 
| 02 | [Setup of local network shares via SMB, permissions, and file transfer from existing HDDs](part_2.md) |
| 03 | [Optional: Setup of docker, a reverse proxy, and a local domain](part_3.md) |
| 04 | [Installation and setup of a VPN, Tailscale, for external access](part_4.md) |
| 05 | [Setup of a secondary machine for off-site backups](part_5.md) |
| 99 | [Philosophical considerations, and cursed knowledge, reflecting why the nas is build as it is](part_99)

# Step 1: Alpine Installation
Download the most recent version of [Alpine](https://www.alpinelinux.org/downloads/ "Alpine Linux Download") (3.20.3 at the time of writing). Additionaly, old-school installers will need: a usb stick, LAN connection, monitor, and keyboard. Boot from it on your designated hardware. Follow the instructions from [the alpine installation manual](https://wiki.alpinelinux.org/wiki/Installation). In short, start the process by entering "root" as user in the presented promt, and then type
```
setup-alpine
```
My selections:
- de->de as keyboard layout
- hostname mynas.local
- enabeled and selected DHCP for all network interfaces (unnecessary ones not connecting will make booting slower, but can be disableded later)
- DNS letf at default
- root password *****************
- Timezone Europe/Berlin
- No proxy
- Default apk mirror
- user account name yournamehere
- Network Time Protocol (NTP) left at default
- No SSH key provided at this time
- nvme0n1 as disk for installing apline
- entered "sys" as usage mode
  
with these steps, you successfully installed alpine linux on your target hardware. Reboot. It will directly start from your your internal storage, and you can remove the installation media.

# Step 2: Essential post-setup instructions
The main goal here is to create your own, non-root administrative account.
## Add Admin Account
Add a user to the usergrup that will administer the server. If not created during initial setup, use the following command, and complete the setup, including setting the password.
```
adduser yournamehere
```
Once the account exists, add them to the administrator user group. In my case that group is called "wheel".
```
adduser yournamehere -G wheel
```
## Install rights elevation software
Before switching to the new user, we must have a rights elevation tool installed. Debian users will be familiar with sudo. We will use a similar command "doas". This is easiest installed using the alpine package manager apk and requires that your machine has internet access. If not, check your network settings (dhcp, interfaces,..). Lets start with a general system update check:
```
apk update
apk add --upgrade apk-tools
apk upgrade --available
sync
reboot
```
Next, install doas. Additionally, install a text editor of your choice. I will be using nano
```
apk add doas nano
```
Doas needs some setup. Use the freshly installed text editor nano to edit the configuration.
```
nano /etc/doas.d/doas.conf
```
check that the line of text included lists your administrative usergroup. It should read something like this:
```
permit persist :wheel
```
To save the file, use the key combination `ctrl + s`, then `ctrl + x` to exit.  
Now lets change users. Logout using the `exit` command, or reboot. Login using your previously set up user. Check if doas is working, e.g.
```
doas apk update
```
If there are not permission errors, the setup worked :)

## remove root login
Disable the root login by resetting the password.
```
doas passwd -l root
```
If we need elevated privileges from now on, we will use the doas command.

## disable unnecessary network interfaces
List the network interfaces using
```
ip -c a
```
The interface showing your current ip address is the one we want to keep, and disable the other ones. This will speed up the boot process due to the machine no longer waiting for DHCP. Remove all lines assocaited with the unnecessary interface (ignore loopback) in the /etc/network/interfaces file.
```
doas nano /etc/network/interfaces
```
For me, i only kept the loopback one and the eth0 (on dhcp).

# Step 3: Fixed Ip and SSH access
Until now you probably connected the server hardware to a monitor/TV, and are uncomfortably halfway sitting and standing somewhere. Lets fix that.
## Assign fixed ip
There are two easy ways to accomplish it: Change the configuration within your switch/router to always give the same ip, or change the network discovery mode of the server, e.g. by modifying `/etc/network/interfaces`. I had bad luck with losing my `ìnterfaces`configuration during upgrades, so i will describe only the first approach here:
Log into your router or switch (in my case it's a Fritz!Box), navigate to "heimnetz"->"netwerk", select the server based on its ip from the list, click edit, and assign a fixed ip (192.167.167.XXX). While you are in your FritzBox interface, make sure to enable 1GB/s speeds under "Netzwerkeinstellungen"->"Lan-Einstellungen".
## SSH
SSH lets you connect remotely to the terminal of the server. on a Windows/macOs/linux device, open a commandline / terminal, and type
```
ssh yourusername@192.168.178.xxx
```
Substitute your created user name, and the assigned ip address. If you skipped the fixed ip address, use "ip a" on the server, and use the displayed local address. Once promted, enter your password. If this worked, you can now put the server in a closet, away from the monitor.

Keep it up, you are doing great :)
