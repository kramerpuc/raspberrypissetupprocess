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

Then, still on CLI, use "sudo usermod -aG docker $USER" to add the current user to the docker group.  Quick "sudo reboot" to make sure the group changed, as my Pi had problems adding docker to the group without resetting. 

