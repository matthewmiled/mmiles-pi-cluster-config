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

* Using another PC or phone, log into admin dashboard of the router that you configured the OS to connect to in the Raspberry Pi Imager tool (i.e. your home WiFI network).

* You should see the name of your pi in the 'Connected Devices' (this name depends on what you set when you flashed the SD card). Get the IP address of the device.

* Using the `username` you set in imager tool, SSH into the device:
  * `ssh <user?@<ip-address>`
  * Then enter the password you set in the imager tool
 
* The hostname that is defined in the imager tool (e.g. `mmiles-master-`) can be seen/changed in (assuming you have ssh'd into the node):
  * `nano etc/hostname`
  * `nano etc/hosts`
 
* The 64 bit OS seems to try to connect to wifi using the 5 GHz band - which often seems to cut out. To force it to use the 2.4 GHz, SSH into the pi and create a file at `sudo nano /mnt/boot/system-connections` and populate it with:

```
[802-11-wireless]
band=bg
``` 
 
# Installing MicroK8s

* MicroK8s is only compatible with the arm64 CPU architecture. The 64 bit Linux Raspberry Pi OS uses arm64, but the 32 bit uses armhf:
  * Can tell which architecture is used by running `dpkg --print-architecture`

* `sudo apt-get update`
* `sudo apt-get install snapd`
* 
