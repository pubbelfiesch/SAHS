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
  
> [!Note]
> In later stages we will modify configuration files of these packages. Should your customizations lead to non-function of a package, you can easily reinstall it with fresh configuration files:
> ```
> doas apk remove --purge samba
> doas apk add samba
> ```

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
  
> [!Note]
> Should your HDD only show the letters without a number at the end, e.g. `sda` instead of `sda1`, then this is an indication that no partition table is present on the disk. This happens if your freshly wiped the disk for using it in a new nas, and did not initialise it since. To fix this potential issue, you can use parted. Be warned, this will delete all data from the HDD:
> ```
> doas parted /dev/sda
> mklabel GPT
> ```
> Use `q`to quit out of the parted promt. Run `lsblk` again to confirm that the partitions are present like in the sample output above.

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

# Step 3: Creating the root folder structure
The first layer of folders in the future network share need some additional attention. In the following text, i will call these "main folders", and the zfs dataset folder `/nas/data` will be called the "base folder". If you are planning on re-using a the structure from an existing share or external hard drive, follow the optional subchapter below. If you only plan on re-using the data, feel free to skip the next subchapter, and transfer the files afterwards via the network share.

# Transfer existing folders & files to the future share
Plug your external storage into the server, preferably via SATA or USB3-4. Identify the new storage media via the `lsblk` command. To access the content of the HDD, the relevant partition needs to be mounted to a folder. Start by creating a new empty folder.
```
doas mkdir /media/hddexternal
```
Next, mount the relevant partition to the new folder using:
```
doas mount /dev/sde1 /media/hddexternal
```
> [!Tip]
> Check that no error or permission errors appear. If so, make sure to not only mount the disk, i.e. `/dev/sde/`, but the partition `dev/sde1/`. If `lsblk` reports NTFS as file system for your partition, check google for additional command parameters.

Navigate to the new folder to see the content.
```
cd /media/hddexternal
ls
```

copy the relevant folder(s) to the zfs dataset "base folder". The previously installed tool rsync is perfect for that. In the following example, we transfer the entire content of the mounted partition. Take note of no trailing "/", otherwise the content will appear in a new subfolder `/nas/data/hddextern` instead of `/nas/data/`
```
doas rsync -a --progress /media/hddexternal /nas/data
```
Depending on your HDD and its content, this might take a while. Personal example: A couple of hundred of GB of pictures (avg 10mb / pic) transfered at an average of 100 MB/s, although i benchmarked both involved HDD at theoretical 280MB/s. Larger files, however, transfered at almost max speed. rsync gives a nice status per file, but lacks an overall status indicator, so make sure to take your time.

# Main folder structure & permissions
Create the folderstructure in the main zfs datashare (the future root folder of the network share). In my case, i use
```
doas mkdir -p /nas/data/media
doas mkdir -p /nas/data/ingest
doas mkdir -p /nas/data/homes
doas mkdir -p /nas/data/programdata
```
`media` will be used for photo backup from cameras, automatic sync from smartphones, audiobooks, ebooks, etc... The `ingest` folder will be the chaotic box of folders that you tell yourself one day will be sorted into other folders. `homes` will contain personal folders for nas users. We do not need to generate any of these intended subfolders at this time.

It is important to set the permissions of the base folder. First, lets look at the base folder
```
doas chown root:root /nas/data
doas chmod 755 /nas/data
```
The main folders need to be accessed by the nas users. Therfore, each time a new main folder is introduced to the base folder, make sure to run the following commands:
```
doas chgrp -R nas /nas/data/*
doas chmod -R 770 /nas/data/*
```
The first command recursivly sets the group on all sub-folders of `/nas/data`, that's what the capital -R options does. And the asterisk *, is a wildcard which pattern matches. In this case: anything after `/nas/data`. Second command changes the scope of those permissions to give owner and group (`root` and `nas` respectivly) full access but anyone else no permission. Check the permissions using
```
ls /nas/data/ -l
```
Advanced users can check [the documentation of permissions](http://linuxcommand.org/lc3_lts0090.php) to create customized structures and persmission.

# Step 3: Make the files accessible via local network
This steps explains how to setup samba, the smb share.
# Check the hostname
Future users of the nas will be able to automatically discover the server in the network tab of their local machine. The name displayed to them is dependent on the hostname of the server. To change the hostname, edit `/etc/hostname` and change the name. Make sure that its a fully qualified hostname, ending in a top-level domain. In my case, i chose `mynas.local`.
```
doas nano /etc/hostname
```
After saving, check it by typing the following command, which will output the current hostname.
```
hostname
```
# Samba configuration
Samba was already installed in Part 2 Step 1. However, which folders shall be accessible by whom, is defined in a configuration file. The default file contains many many settings and examples. To keep things simple, we will start with the most simple setup. At first, we backup the default configuration.
```
doas cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```
Open the `smb.conf` in a text editor.
```
doas nano /etc/samba/smb.conf
```
Remove the entire content. `ctrl + k` removes entire lines at once. Here is my sample configuration content.
```
#======================= Global Settings =====================================
[global]

   workgroup = WORKGROUP
   server role = standalone server
   force group = nas
   inherit permissions = yes
   force create mode = 0750
   force directory mode = 0770
   client min protocol = SMB3
   map to guest = Bad User

#============================ Share Definitions ==============================

[Share]
    comment = Files shared with all nas users
    path = /nas/data
    valid users = @nas
    guest ok = no
    read only = no
```
> [!Note]
> These settings work for Windows > 7, MacOS, and Linux.
> Only users within the "nas" group will be able to access the files on the server.

# Autostart samba upon boot
Previously, we installed samba. This is the software server that makes the files and folders accessible to the nas user. Make sure that it is automatically started upon server boot.
```
doas rc-update add samba
doas rc-service samba start
doas rc-update add wsddn
```
wsddn is an additional service we previouly installed to see the server in the windows network overview. 
to be continued
  
> [!Tip]
> If you have trouble with services not starting up with the server, have a look at this[link to rc-update examples](https://www.cyberciti.biz/faq/how-to-enable-and-start-services-on-alpine-linux/)
> especially the combination of `rc-service --list | grep -i YourDesiredServiceName` and `doas rc-update add YourDesiredService default` works like a charm for me.

# Samba password and onboarding new nas users
Samba recognises the linux server users as possible users of the file share. However, the password for the share must be manually set. Start by setting your own samba password.
```
doas smbpasswd -a YourCurrentUserOrAnotherUserOfTheNas
```
For adding other users to the nas, execute the following commands:
```
doas adduser NewUserNameOfYourChoosing
doas adduser NewUserNameOfyourChoosing nas
doas smbpasswd -a NewUserNameOfYourChoosing
```

# Troubleshooting
Congratulations, you should now be able to see the server in Windows Explorer Network, or in the MacOs Finder Network. If not, check the following things:
- Try to manually connect to the server from your windows/macos/linux machine. Windows users can type into the explorer file address the ip of the server with two backslashs, e.g. `\\192.168.178.xxx`. MacOs users can "connect to server" and type `smb://192.168.178.xxx`.
- Check that each user of the nas has a serverside linux AND a samba password set.
- Check that samba boots when the server boots (see rc-update example above)
- Check that the samba configuration is correct
