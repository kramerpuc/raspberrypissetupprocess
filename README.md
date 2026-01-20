# raspberrypissetupprocess
Set up process for configuring raspberry pis for my set up.  Feel free to borrow.  This guide references a variety of external sources and uses dockge 

**Step 1. Raspberry Imager**

Use Raspberry Imager to create an SD with the appropriate flash.  Use Headless for anything that is not for user friendly access, and authenticate raspberry pi connect for each.
Raspberry Pi connect is key for the set up process, it allows you to easily remote into Pis on your network. 

After writing the image, put the SD card inside of your 

**Step 2. Internal Configuration**

First, use "sudo apt update && sudo apt upgrade -y" to update and upgrade all programs on the Raspberry Pi image.

Next, use "hostname -I" to find the Raspberry Pi ipaddress, then FIND A METHOD FOR FINDING NORMAL GATE WAY ADDRESS, In my case it is just XXX.XXX.XXX.1

Next, use "sudo nmtui", navigate to "edit connections", and edit eth0 or wlan0, whichever you are using.  I use eth0, as I have an ethernet connection for my Pis. Select the IPV4 connection and switch automatic to manual.  Put the address to the hostname ipaddress, and the gateway address to the one you just found.  Or, configure a different way.  I am not good enough at this yet to do that, so I rely on using a Static IP from an automatically used one.  

**Step 3. Docker**

Docker is our next stop.  

Get it set up with "curl -sSL https://get.docker.com | sh" on the CLI on the Raspberry Pi.

Then, still on CLI, use "sudo usermod -aG docker $USER" to add the current user to the docker group.  Quick "sudo reboot" to make sure the group changed, as my Pi had problems adding docker to the group without resetting. After resetting, use the "groups" command to ensure the current user has docker access.

**Step 4. DockGe**

DockGe is a pretty cool way to manage all docker containers, but I am still pretty new to it.  To set it up, I heavily reference this guide here: https://pimylifeup.com/raspberry-pi-dockge/ 

wget should already be installed, so we don't need to worry about it.  After ensuring that docker is running, we will navigate to the root directory and make a dockge directory using the command "sudo mkdir -p /opt/dockge && cd /opt/dockge", and then make another directory using "sudo mkdir -p /opt/stacks"

any Docker compose files must be inside the stacks directory for DockGe to manage them.  

Then, ensure you are within the /opt/dockge/ directory, and use wget to download the compose file. "sudo wget https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml"

Dockge operates on port 5001.

The last step on the Raspberry Pi is to use "docker compose up -d" for docker to download the DockGe container and start it up.  
After giving Docker sometime to start it up, we can visit https:// IPADDRESS :5001/ and sign in.

DockGe is still a bit confusing to me.  to my understanding, you make a docker compose file, put it in the compose category, and then deploy. It is really simple to get running, and really simple to edit after you understand docker a bit.  

**Step 5. Pihole**

Pihole is a DNS FTL that blocks ads at a network level.  We can use it in conjunction with a local DNS to keep everything local, and in my set up, I run 2 computers with Pihole and PowerDNS for redundancy.  One downside of Pihole is that when the computer running it goes down, the network goes down, so redundancy is necessary.  

I use https://docs.pi-hole.net/docker/ for pihole through Docker.  

On one of my PCs, I use ports 80 and 81 for NGINX, so I have edited the configuration file in a couple different places.

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      # DNS Ports
      - "5335:53/tcp"
      - "5335:53/udp"
      # Default HTTP Port
      - "82:80/tcp"
      # Default HTTPs Port. FTL will generate a self-signed certificate
      - "443:443/tcp"
      # Uncomment the below if using Pi-hole as your DHCP Server
      #- "67:67/udp"
      # Uncomment the line below if you are using Pi-hole as your NTP server
      #- "123:123/udp"
    environment:
      # Set the appropriate timezone for your location from
      # https://en.wikipedia.org/wiki/List_of_tz_database_time_zones, e.g:
      TZ: 'Europe/London'
      # Set a password to access the web interface. Not setting one will result in a random password being assigned
      FTLCONF_webserver_api_password: 'correct horse battery staple'
      # If using Docker's default `bridge` network setting the dns listening mode should be set to 'ALL'
      FTLCONF_dns_listeningMode: 'ALL'
    # Volumes store your data between container upgrades
    volumes:
      # For persisting Pi-hole's databases and common configuration file
      - './etc-pihole:/etc/pihole'
      # Uncomment the below if you have custom dnsmasq config files that you want to persist. Not needed for most starting fresh with Pi-hole v6. If you're upgrading from v5 you and have used this directory before, you should keep it enabled for the first v6 container start to allow for a complete migration. It can be removed afterwards. Needs environment variable FTLCONF_misc_etc_dnsmasq_d: 'true'
      #- './etc-dnsmasq.d:/etc/dnsmasq.d'
    cap_add:
      # See https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
      # Required if you are using Pi-hole as your DHCP server, else not needed
      - NET_ADMIN
      # Required if you are using Pi-hole as your NTP client to be able to set the host's system time
      - SYS_TIME
      # Optional, if Pi-hole should get some more processing time
      - SYS_NICE
    restart: unless-stopped

Set a custom password and change your time zone, paste in the compose.yaml, and deploy.

**Step 6. PowerDNS**

