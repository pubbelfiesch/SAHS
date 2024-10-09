This part explains how to install and setup SMB to create a general purpose share folder in the local network. optionally, steps to copy existing data from external hard drives locally on the server to the share will also be shown.

| Part | Description & Link|
| ----------- | ----------- |
| 00 | [Overview and ReadMe](https://github.com/pubbelfiesch/SAHS/) |
| 01 | [Alpine installation until the server can be remotely managed via SSH](part_1.md) | 
| 02 | [Setup of local network shares via SMB, permissions, and file transfer from existing HDDs](part_2.md) |
| 03 | [Setup of a self hosted VPN for remote access to the files, and networking](part_3.md) |
| 04 | [Installation of Docker, NGINX reverse proxy, IMMICH, and Nextcloud (optional step)](part_4.md) |
| 05 | [Setup of a secondary machine for off-site backups](part_5.md) |
| 99 | [Philosophical considerations, and cursed knowledge, reflecting why the nas is build as it is](part_99)

# Step 1: Install Samba and dependencies
Install the following packages, and their documentation:
```
doas apk update
doas apk add mandoc man-pages tmux samba rsync parted lsblk avahi dbus sipcalc ncurses-terminfo
doas apk add doas-doc nano-doc tmux-doc samba-doc rsync-doc parted-doc util-linux-doc avahi-doc dbus-doc sipcalc-doc
```
Should you have highly specific with a customized command of one of these packages, the documentation might be more helpful than a google search. These `man`pages can be used as follows:
```
man rsync
```
Once confirmed working, press `q`to exit.
  
Note: In later stages we will modify configuration files of these packages. Should your customizations lead to non-function of a package, you can easily reinstall it with fresh configuration files:
```
doas apk remove --purge samba
doas apk add samba
```

# Step 2: Setting up your large media storage device
Until now i only used the 260GB NVME SSD for the Alpine operating system. For my planned usecase i need more storage space, which is why in the following steps i will be configuring a 12TB HDD. If you are just starting out, you can skip this step and only work with the primary storage device.
  
I want to use a 12TB HDD for with ZFS in a single disk configuration. This is not ideal, and a dedicated chapter is provided in [part_0.md](part_0.md). The remaining steps of this tutorial are agnostic to the exact specification of your storage setup, so feel free to substitute your own.
  
Start by installing ZFS
```
doas apk install linux-lts zfs zfs-lts zfs-doc
```
To not manually start the service after each boot, confingure your server to start zfs at system initialisation.
```
doas rc-update add zfs-import sysinit
doas rc-update add zfs-mount sysinit
```
==Reboot==

Now we need to identify the disk(s) that we will group into a ZFS pool. We will use `lsblk`, which we previously installed.
```
lsblk
```
Sample output:
```
# Sample only
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    0  10.9T  0 disk 
├─sda1        8:1    0  10.9T  0 part 
└─sda9        8:9    0     8M  0 part 
nvme0n1     259:0    0 238.5G  0 disk 
├─nvme0n1p1 259:1    0   512M  0 part /boot/efi
├─nvme0n1p2 259:2    0     4G  0 part [SWAP]
└─nvme0n1p3 259:3    0   234G  0 part /
```
`nvme0n1`can be identified as my primary boot storage with my os on it by the size of it. Similarly, `sda` seems to be the large HDD. `sda1` is our target partition we will further use.
  
Sidenote: Should your HDD only show the letters without a number at the end, e.g. `sda` instead of `sda1`, then this is an indication that no partition table is present on the disk. This happens if your freshly wiped the disk for using it in a new nas, and did not initialise it since. To fix this potential issue, you can use parted. Be warned, this will delete all data from the HDD:
```
doas parted /dev/sda
mklabel GPT
```
Use `q`to quit out of the parted promt. Run `lsblk` again to confirm that the partitions are present like in the sample output above.

To create the ZFS pool with the name "nas", use 
```
doas zpool create nas /dev/sda
```
Advanced users with RaidZ1/2/3 in mind could use something like:
```
# Careful to get the right drives!
doas zpool create nas raidz /dev/sda /dev/sdc /dev/sdd
```
Storing data in the ZFS pool requires so-called datasets. Imagine buckets that fill the pool. For this setup, we will only create one dataset called "data" to store all future data, either user data, or self hosted service data. Advanced users are free to create their own setup.
```
doas zfs create nas/data
```
Now we have the SSD to store our operating system, and the HDD ZFS pool for the nas content.
  
To check if this worked, you can use the following commands:
- `zfs list` should display your dataset
- `zpool lost` should list your zpool
- `cd /nas/data` and `ls` should work and reveal an empty "folder".

to be continued
