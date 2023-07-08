# Installing OS

* For this build - we are using the Raspberry Pi OS Lite (64 Bit). The underlying Debian version is 11 (bullseye)`.

* There is a [Raspberry Pi Imager tool](https://www.raspberrypi.com/software/) that can be used to write the OS to an SD card.

* It will give you the option to set up WiFi automatically, and also SSH access.
  * At the moment we're just using SSH password access

* Once flashed, cd into the SD card volumne from terminal and create an ssh file:
  * `cd /Volumes/bootfs`
  * `touch ssh`

# SSH Access
