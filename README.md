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
 
* The hostname that is defined in the imager tool (e.g. `mmiles-master`) can be seen/changed in (assuming you have ssh'd into the node):
  * `nano etc/hostname`
  * `nano etc/hosts`
 
* The 64 bit OS seems to try to connect to wifi using the 5 GHz band - which often seems to cut out. To force it to use the 2.4 GHz, SSH into the pi and create a file at `/mnt/boot/system-connections` and populate it with:

```
[802-11-wireless]
band=bg
``` 
 
# Installing MicroK8s

* Firstly, add the following lines to the file at `/boot/cmdline.txt`:
  * `cgroup_enable=memory cgroup_memory=1`
  * Then run `sudo reboot`

* MicroK8s is only compatible with the arm64 CPU architecture. The 64 bit Linux Raspberry Pi OS uses arm64, but the 32 bit uses armhf:
  * Can tell which architecture is used by running `dpkg --print-architecture`

* `sudo apt-get update`
* `sudo apt-get install snapd`
* `sudo snap install microk8s --classic`

* Give your user permissions to access MicroK8s:
  * `sudo usermod -a -G microk8s <user>`
  * `newgrp microk8s`
 
* Adding following to PATH to ensure you can execute microk8s commands:
  * `export PATH=$PATH:/snap/bin`
  * Can also add this to 

* On the node you want to be master, run the command:
  * `sudo microk8s.add-node`
  * It should spit out a connection string in the form of `<master_ip>:<port>/<token>`
 
* On a worker node - do all of the same but instead of running the master node command, run:
  * `microk8s.join <master_ip>:<port>/<token>`

* You will likely see a message such as:
  * `Connection failed. The hostname (<worker_name>) of the joining node does not resolve to the IP "<worker_ip>". Refusing join (400).`
  * Back in your master node, do a `sudo nano` of the file at `etc/hosts`
  * At the bottom, add `<worker_ip>    <worker_name>`
 
* Back on the worker node, execute the below again and it should connect to the cluster:
  * `microk8s.join <master_ip>:<port>/<token>`
 
```
microk8s.kubectl get nodes

NAME                  STATUS   ROLES    AGE   VERSION
mmiles-pi-master      Ready    <none>   54m   v1.27.2
mmiles-pi-worker-00   Ready    <none>   7s    v1.27.2
```
