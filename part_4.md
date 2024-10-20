The files and services on the server are currently only accessible via local network. If you need data on the go via internet, you can chose to make your server accessible via internet. However, in this tutorial, we will only use a VPN. This tunnel allows specific devices with specific software to access the data without exposing the data to security risks.

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


# Step 1: Install tailscale
## Install on another device
Tailscale is a free-for-personal use tunnel software, which has a nice user-interface and user-experience. Lets make use of that by first installing tailscale on a device that has more than a command line interface :) I chose my laptop. Visit [tailscale.com](https://tailscale.com/) and download the installer for your plattform. Login with a method of your choice.
> [!Tip]
> Dont be afraid to use GitHub as login, you will not need to type cryptic login parameters via command line into the server.
Tailscale will generate a dedicated tunnel ipv4&6 for each device that joins your network. Your network is defined by the devices associated to the login credentials you used to login to tailscale. 
## Install tailscale on the server
Install tailscale on the server by using the following command:
```
curl -fsSL https://tailscale.com/install.sh | doas sh
```
Once completed it will output a link to login. Copy paste that link into a browser, and login with the same method as in the previous step.  
Thats it. The two devices are now connected via tunnel. In your non-server tailscale client, you should be now able to see the server via its hostname in the network. Check out if everything is working by connect to server with the server tailscale IP address.
```
ssh yourUserName@10X.93.ZZ.YY
```
Android users can use the app JuiceSSH.
  
Even the network share should be working via the new tailscale ipv4. Congrats, you can now access your ssh and data from everywhere in the world, as long as you and the server are connected to the tailscale VPN. 
