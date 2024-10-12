This part explains how to setup tailscale to connect to your homeserver from outside your immediate local network. 

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

# Step 2: Install docker
## Main docker installation
Having a locally accessible data pool is nice, but i prevere to have some services to manage my data. E.g. having an automatic backup of my phone picture to the server, synchronize notes across devices, and a nextcloud instance for managing and editing files via a webbrowser instead of a command line. In my case, i would like to access at least the photo sync service via the internet, not only VPN, to share pictures with non-nas users. 
Lets start by installing docker, the basic service which all other additional services depend on.
```
doas apk add docker docker-compose
```
Next, lets make docker able to run without root privilliges. Add your current user to the docker group, which was automatically setup during the previous step.
```
doas adduser §{User} docker
```
Lets add docker to the services that will be started at boot.
```
doas rc-update add docker boot
```
reboot.
## Optional installation of portainer
Managing docker via command line is fine, but when later editing compose files with 10th of lines, a webUI is nicer. So lets install portainer
```
docker volume create portainer_data
```
If you have permission errors, search help for correct docker rootless usage.
```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.21.3
```
On a second device within your local network (or tailscale network), type the ip of the server, and add the port of portainer.
```
https://192.168.178.XXX:9443/timeout.html#!/2/docker/dashboard
```
Upon your first visit to the site you will be asked to set your portainer administrator username and password. Next, click on the "new environment" option, and then click on the only option presented to you.

# Install nginx proxy manager
We could expose the server to the internet and map ports to specific services. I prefere to have a dedicated url/domain name i can type into the address bars of browsers and apps, i.e. `photos.pubbelfiesch.com` instead of `244.183.221.055:2283`. This is achieved with a reverse proxy manager. An additional benefit is using HTTPS for basically every service, so that these can securely be accessed even in non-secired wifis. Lets install it in the portainer web user interface.  
Login to your local portainer web instance, and add a "stack". Copy paste the following text into the presented field.
```
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```
Make sure to name it, otherwise portainer will not let you create the service. On the bottom of the page, click "deploy stack", and wait a bit. Once automatically redirected to the `stacks` overview page, you can now access nginx-proxy-manager via the webui on your servers ip:
```
http://192.168.178.XXX:81/login
```
The credential for the first login are 
```
admin@example.com
changeme
```
Upon first login, you will be asked to change the username and password. 
A complete guide on how to setup the local dns resolver etc can be found [here](https://www.youtube.com/watch?v=qlcVx-k-02E&t=456s).


