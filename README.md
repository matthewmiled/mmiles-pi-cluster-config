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
 
* The 64 bit OS seems to try to connect to wifi using the 5 GHz band - which often seems to cut out. To force it to use the 2.4 GHz, SSH into the pi and `sudo nano` the file `/etc/wpa_supplicant/wpa_supplicant.conf`. Inside it, add the line:
  *  `freq_list=2412 2417 2422 2427 2432 2437 2442 2447 2452 2457 2462 2467 2472`

 
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
  * Can also add this to `bashrc`

* On the node you want to be master, run the command:
  * `microk8s.add-node`
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
matt@mmiles-pi-master:/etc $ microk8s.kubectl get nodes

NAME                  STATUS   ROLES    AGE   VERSION
mmiles-pi-master      Ready    <none>   54m   v1.27.2
mmiles-pi-worker-00   Ready    <none>   7s    v1.27.2
```

* If you disconnect a pi, the STATUS should change to `NotReady`

# TODO

* Now have the pi's and cluster set up at the most basic level - if you SSH into one and do `microk8s.kubectl get nodes` - you can see the devices listed.

* The internet connection of the devices is still pretty intermittent - when they're connected to 2.4GHz it seems fine, but they often change to 5GHz which then makes them disconnect and unable to SSH into them. The fix stated above doesn't really work. Need to make this more robust, or even connect them directly via ethernet.

* SSH currently uses a user/password and the IP - should set up SSH keys.

* Devices currently have dynamic IPs, so if these were to change we would have to keep putting in new IPs into the `ssh matt@<ip>` command. Can change this to static in the home wifi admin dashboard I think.

* Set up k9s so can access cluster without directly ssh-ing into one of the nodes/pis.

* Create some namespaces

* Look at setting up helm etc.
