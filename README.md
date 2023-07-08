# Installing OS

* For this build - we are using the Raspberry Pi OS Lite (64 Bit). The underlying Debian version is 11 (bullseye)`.

* There is a [Raspberry Pi Imager tool](https://www.raspberrypi.com/software/) that can be used to write the OS to an SD card.

* It will give you the option to set up WiFi automatically, and also SSH access.
  * At the moment we're just using SSH password access

* Once flashed, cd into the SD card volumne from terminal and create an ssh file:
  * `cd /Volumes/bootfs`
  * `touch ssh`

# SSH Access

* Insert SD card into pi

* Using another PC or phone, log into admin dashboard of the router that you configured the OS to connect to in the Raspberry Pi Imager tool.

* You should see the name of your pi in the 'Connected Devices' (this name depends on what you set when you flashed the SD card). Get the IP address of the device.

* Using the `username` you set in imager tool, SSH into the device:
  * `ssh <user?@<ip-address>`
  * Then enter the password you set in the imager tool
 
* 
