# Pi docker home server

The goal of this project is to use git, and a pi, to create a home server.

On this server we will install: 
- Git for pulling version controled configuration files
- Vim for editing files
- Docker for running services
- Pip3 to get docker-compose

After this we will build up a few docker containers. 
- Pihole for network wide ad block
- Transmission for the torrent love
- OpenVpn to have safe external access
- Owncloud for network storage

## Setting up the Pi

Step one is to [download](https://www.raspberrypi.org/downloads/raspbian/ "raspbian download page") the pi image
I will download the  [Raspbian Buster Lite torrent](https://downloads.raspberrypi.org/raspbian_lite_latest.torrent "raspbian_lite_latest.torrent" )
Unzip the image before you burn to a flash drive

The internet seems to like [Etcher](https://electronjs.org/apps/etcher "Flash OS images to SD cards & USB drives, safely and easily" ) for burning the image, so we will use it on arch.

` yay -S etcher `

While electron apps are nice as a minimlist I willl remove this after we are done. 
Open etcher and burn the image to your microSD card

` yay -Rns etcher `

### Configuring the pi for ssh and wifi

I attached a monitor and keyboard so that I could edit the files localy before attempting over ssh.
Let's set a few things with some of pi's built in tools

` sudo raspi-config `

Network options set up your network info.
Might as well change the pi users password at this time too.

` sudo raspi-config `

To configure a pi image to accept ssh connections add a file called ssh no extention to the boot partation.
` cd /boot `
` sudo touch rss `

### Post Pi install configurations steps.

Connect to the Pi with ssh

` ssh pi@IP `

Next we will update and upgrade our system.

` sudo apt-get update `
` sudo apt-get upgrade -y `

Now we should install some of the programs we will need local to the pi.

` sudo apt-get install git vim python3-pip -y `

### Setting up docker on your pi. 

We will use the get docker methoud to pull in docker on this pi

` curl -fsSL https://get.docker.com -o get-docker.sh `
` sudo sh get-docker.sh `

Now lets start docker at boot 

` sudo systemctl enable docker `

Next step is to add the Pi user to the docker group so we do not need to sudo our docker images.

` sudo usermod -aG docker pi `

Let's reboot our pi and test to be sure all is well

` sudo reboot `

After loging back in run this command to see if you get any errors. 

` docker ps `

Now is a good time to install docker-compose

` sudo pip3 install docker-compose `

## Pihole

Pihole is a nice tool you can use use to block ads on your network wide.
Let's manually build a docker compose file for this one. 

Create a docker directory with a sub directory for pihole.
` mkdir docker `
` cd docker `
` mkdir pihole `
` cd pihole `
Next we will create our Dockerfile

` touch Dockerfile `
` vim Dockerfile `
 
This file will need to be configured for your needs.

` version: "3"
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: 'your timezone'
      WEBPASSWORD: 'set a secure password here or it will be random'
    # Volumes store your data between container upgrades
    volumes:
       - './etc-pihole/:/etc/pihole/'
       - './etc-dnsmasq.d/:/etc/dnsmasq.d/'
    dns:
      - 127.0.0.1
      - 1.1.1.1
    # Recommended but not required (DHCP needs NET_ADMIN)
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN
    restart: always `

Now that we have a real docker compose file let's rename it to docker-compose.yaml

` cp Dockerfile docker-compose.yaml `

Now let's test if this will run.

` docker up docker-config.yaml `

Next step is to connect with a browser at http://192.168.1.208/admin/ if all is well we will close this down.

` ctl-c `

Then restart with 

`docker-compose -d `

The next step is to redirect your router to use this pi as the dns server, you will need to find out how to use a 
custom DNS for your routers firmware. Google is your best resource "google rounter type set DNS server". The IP 
for the DNS will be the IP of your pi.

At this point I will make my intal commit before we move on to other services.

