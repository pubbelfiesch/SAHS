# SAHS
Simple Alpine based home server, a.k.a DIY NAS

# Overview
These instructions describe on how to create a simple home server. Features:
- Lightweight linux basis (Alpine)
- Network share for Windows, MacOs, Linux, using SMB, and next to nonexistant rights management
- Backups via a secondary machine outside the local network (to be implemented)
- Docker environment for services, without crazy network settings
Most of them are ~~stolen~~ inspired from the excellent guide 3.0 of [dither8](https://dither8.xyz/guide/alpine-linux-nas-3/ "Link to dither8's website"). This git repository is a backup in case his site is no longer available. 

# Step 0: Preconsiderations and architecture
Main NAS hardware is an HP ProDesk 600 G4 with Intel 8500T, 32GB ram, 260Gb nvme SSD, 1x 12TB Ironwolf HDD
Backup server is an HP ProDesk 405 G4 with AMD Ryzen 2200G, 8GB Ram, 128GB SSD, 1x 12TB Ironwolf HDD (recertified)
ZFS is used as file system, and ZFS replication is used as backup mechanism (to be implemented). No Mirror/raid is used per machine.
No typical diy NAS operating system is used due to personal preference, mostly to start with a minimalist approach and learn from there.

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
## Install rights escalaltion software
Before switching to the new user, we must have a rights escalation tool installed. Debian users will be familiar with sudo. We will use a similar command "doas". This must be installed using the alpine package manager apk and requires that your machine has internet access. If not, check your network settings (dhcp, interfaces,..). Lets start with a general system update check:
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
Doas needs some setup. Edit the following configuration 
```
nano /etc/doas.d/doas.conf
```
check that the line of text included lists your administrative usergroup. It should read something like this:
```
permit persist :wheel
```
Logout using the exit command, or reboot. Login using your previously set up user. Check if doas is working, e.g.
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
I logged into my router (Fritz!Box), navigated to "heimnetz"->"netwerk", selected the server based on its ip from the list, clicked edit, and assigned a fixed ip (192.167.167.XXX) While you are in your FritzBox interface, make sure to enable 1GB/s speeds under "Netzwerkeinstellungen"->"Lan-Einstellungen". Alternatively, advanced users can set an ip within the linux in /etc/network/interfaces.
## SSH
SSH lets you connect remotely to the terminal of the server. on a Windows/macOs/linux device, open a commandline / terminal, and type
```
ssh yourusername@192.168.178.xxx
```
Substitute your created user name, and the assigned ip address. If you skipped the fixed ip address, use "ip a" on the server, and use the displayed local address. Once promted, enter your password. If this worked, you can now put the server in a closet, away from the monitor.

To be continued
