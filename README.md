# Installing OS

* For this build - we are using the `Ubuntu Server 20.04.5LTS (64-BIT)` OS

* There is a [Raspberry Pi Imager tool](https://www.raspberrypi.com/software/) that can be used to write the OS to an SD card.

* It will give you the option to set up WiFi automatically (we're not doing this), and also SSH access.
  * At the moment we're just using SSH password access

 
# Networking

[add network diagram here]

* This project uses the TP-Link TL-WR802N Nano Router to create a private wireless network, separate to the main home Wifi network.

* We are using WISP (Hotspot) mode in the nano router - so it wirelessly connects to the main home network, and then connects to a network switch via ethernet. The network switch is the TP-Link TL-SG105S and this then connects to the Pis via ethernet. The private network name is `mmiles-cluster-network`

* Within the main home router admin panel, it was found that the DHCP range was 192.168.1.10 - 192.168.1.254
    * So the nano router needs to have a different range to this. The range for the private network (configured via the nano router) is 192.168.0.100 - 192.168.0.199
 
* Once the nano router is setup and the `mmiles-cluster-network` network can access the internet, it is connected via ethernet to the network switch.

* The network switch is then connected to each of the Pis.

* When you place the SD cards into the Pis and boot them up, they should appear in the `DHCP Clients` list, with assigned IP addresses within the range stated above.

* In the `Address Reservation` section of the TP Link Nano router admin panel, we are going to set these as static/reserved IPs:

[Add images of TP link admin panel here]

* 192.168.0.101 (Master Pi)
* 192.168.0.102 (Worker 00)
* 192.168.0.103 (Worker 01)

# SSH Access

* Using the `username` you set in imager tool, SSH into the device:
  * `ssh <user?@<ip-address>`
  * Then enter the password you set in the imager tool
 
 
# Installing k3s

```
sudo apt upgrade -y

sudo apt install -y docker.io

sudo docker info

sudo sed -i \
'$ s/$/ cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1/' \
/boot/firmware/cmdline.txt

sudo reboot
```

On the 'master' Pi:

`curl -sfL https://get.k3s.io | sh -`

Then get a token:

`sudo cat /var/lib/rancher/k3s/server/node-token`

Then on the worker node:

`curl -sfL https://get.k3s.io | K3S_URL=https://$YOUR_SERVER_NODE_IP:6443 K3S_TOKEN=$YOUR_CLUSTER_TOKEN sh -`

Back on your master node, there should be some config stored in k3s config file:

`sudo k3s kubectl config view --raw`

This probably won't appear on the worker nodes.

We're going to copy this into the standard `.kube/config` file:

```
export KUBECONFIG=~/.kube/config

mkdir ~/.kube 2> /dev/null

sudo k3s kubectl config view --raw > "$KUBECONFIG"

chmod 600 "$KUBECONFIG"
```

We'll also add `export KUBECONFIG=~/.kube/config` to `~/.profile` and `~/.bashrc` on the master node to make it persist on reboot.


Now you should be able to run:

`k3s kubectl get nodes` or just `kubectl get nodes`

And see the master and any worker nodes you've attached.


# Local Machine Access

Firstly install kubectl on your machine:

`brew install kubernetes-cli`

Then go into your master pi and get the k3 config:

`sudo k3s kubectl config view --raw`

Copy it and put it in your local machines `~.kube/config` file.

You will probably need to replace the server filed in the config from:

`server: https://127.0.0.1:6443`

To whatever the actual IP of your master node is:

`server: https://192.168.0.101:6443`

Then if you do a `kubectl get nodes` from local machine, you should get the status of pi cluster:

```
NAME                  STATUS     ROLES                  AGE   VERSION
mmiles-pi-worker-00   NotReady   <none>                 69m   v1.28.4+k3s2
mmiles-pi-master      Ready      control-plane,master   81m   v1.28.4+k3s2
mmiles-pi-worker-01   Ready      <none>                 13m   v1.28.4+k3s2
```

(Node 00 isn't ready because it's disconnected)



# TODO

* Now have the pi's and cluster set up at the most basic level - if you SSH into one and do `microk8s.kubectl get nodes` - you can see the devices listed.

* The internet connection of the devices is still pretty intermittent - when they're connected to 2.4GHz it seems fine, but they often change to 5GHz which then makes them disconnect and unable to SSH into them. The fix stated above doesn't really work. Need to make this more robust, or even connect them directly via ethernet.

* SSH currently uses a user/password and the IP - should set up SSH keys.

* Devices currently have dynamic IPs, so if these were to change we would have to keep putting in new IPs into the `ssh matt@<ip>` command. Can change this to static in the home wifi admin dashboard I think.

* Set up k9s so can access cluster without directly ssh-ing into one of the nodes/pis.

* Create some namespaces

* Look at setting up helm etc.
