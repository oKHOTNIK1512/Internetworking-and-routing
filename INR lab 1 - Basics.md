---
tags: Internetworking and routing
---
:::success
# INR lab 1 - Basics
**Name: Ivan Okhotnikov**
:::


## Task 1: Installation:
:::info
What are the different ways you can configure internet access in GNS3? Test them with a
single PC and give a one-line description of each. What are the differences between them?
:::


## Implementation:

<center>

![](https://i.imgur.com/vIXixpF.png)


Figure 1: Types of internet connection on GNS3

</center>

### Types of internet connection: 
:::    success
Communication with nat. This connection uses the ```virbr0``` bridge interface on my computer (ip address pool 192.168.122.x)
:::
<center>

![](https://i.imgur.com/0EIIPxY.png)
Figure 2: Check internet connection
</center>

---

:::    success
Connection with ```enp1s0``` interface. This type of connection allows you to connect directly to the computer's network card via the ```enp1s0``` interface
:::

<center>

![](https://i.imgur.com/PCUBoo5.png)
Figure 3: Check internet connection
</center>

---

:::    success
```Tap0``` connection.
To configure this connection, you will need to do some manipulations on the working computer
:::

**Install packages:**
```
sudo apt install uml-utilities
sudo apt install bridge-utils
```
**Create Tap connection for my user**
```
sudo tunctl -u st11 -g netdev -t tap0
sudo ifconfig tap0 10.1.1.1 netmask 255.255.255.0 up
```
**Create bridge connection**
```
sudo ip link add br0 type bridge
```
**Add ```enp1s0``` and ```tap0``` interfaces in bridge ```br0```**
```
sudo brctl addif br0 enp1s0
sudo brctl addif br0 tap0
```
**Enable interfaces**
```
sudo ifconfig tap0 up
sudo ifconfig enp1s0 up
sudo ifconfig br0 up
```
**Since the Internet will no longer work on the workstation, we enable the Internet on the workstation via the ```br0``` interface**
```
sudo dhclient -v br0
```

<center>

![](https://i.imgur.com/nhUtEYV.png)
Figure 4: Checking ```tap0``` and internet connections using the ping utility
</center>

---

## Task 2: Switching

:::info
a. Create the following topology in GNS3
:::

## Implementation:

I created this topology

The result is shown below (Figure 5).

<center>

![](https://i.imgur.com/WMAFHfv.png)

Figure 5: Created topology

</center>

---

:::info
b. Install OpenSSH-server on both VMs and Nginx web server on the Web VM.
:::

## Implementation:
To install ssh, you need to open a terminal on a virtual machine and run the following command:
```
sudo apt-get install openssh-server command
```

To install nginx:
```
sudo apt-get install nginx
```
---

:::info
c. What is the IP of the mask corresponding to /28? How many machines can you configure under
this subnet?
:::

## Answer:
The mask /28 corresponds to the mask ```255.255.255.240```
Number of IP addresses: 16
(of which 2 ip addresses are reserved)
Available for use by clients: 14 addresses

---
:::info
Configure the VMs with private static IPs under a /28 subnet.
:::

## Implementation:

To configure a static ip address, you need to edit the netplay file for the administrator and the web
```
sudo nano /etc/netplan/50-cloud-init.yaml
```
<center>

![](https://i.imgur.com/KdAVUjt.png)

Figure 6: Configuration example for a web machine
</center>

Next, we will apply the changes
```
sudo netplan apply
```
---

:::    success
e. Check that you have connectivity between them. Hint: use ping, traceroute, mtr
:::
## Implementation:
<center>

![](https://i.imgur.com/eBM0cdV.png)
Figure 7: Admin ping from the Web machine
</center>
<center>

![](https://i.imgur.com/AlTskN6.png)
Figure 8: Web ping from the Admin machine
</center>

---

:::info
f. Make sure your web server is accessible from the Admin VM. Hint: use curl or wget
:::

## Implementation:
<center>

![](https://i.imgur.com/55s6u86.png)

Figure 9: Checking the working of the web server from the administrator's machine
</center>

---

## Task 3: Routing

:::info
a. Select a virtual Routing solution (Gateway) such as Mikrotik, PfSense, VyOS, Untangle NG,
OpenWrt, Cumulus VX).
:::

I chose the Mikrotik gateway as the routing solution

---

:::info
b. Change the topology as follows.
:::
<center>

![](https://i.imgur.com/bigXLBy.png)
Figure 10: New topology
</center>

## Implementation:
Since the task requires further Internet connection and access to my local machine, it was decided to finalize the topology by adding NAT access to it
<center>

![](https://i.imgur.com/v1ekYMN.png)
Figure 11: My topology
</center>
---

:::info
с. Connect your Gateway to the internet and to your workstation/laptop.
:::

## Implementation:
In order to connect the Internet to my gateway, you need to open it in command-line mode and register its ip address manually (or specify it as a dhcp client)
The command for Mikrotik:
```
/ip address add address=192.168.122.200/24 interface=ether3
```
where ```192.168.122.200``` is the required ip address for the client, and ```ether3``` is the name of the interface

---

:::info
d. Configure port forwarding for HTTP and ssh to Web and Admin respectively.
:::

## Implementation:
To configure port forwarding on Mikrotik, you must first configure the masquerade rules for the required internal subnet
```
/ip firewall nat add action=masquerade chain=srcnat src-address=192.168.10.1/28  
```

We will configure the forwarding of the external port 8080 to the port 80 of the Web
```
/ip firewall nat add action=netmap chain=dstnat dst-address=192.168.122.200 dst-port=8080 in-interface=ether3 protocol=tcp src-address=192.168.10.1/28 to-addresses=192.168.10.3 to-ports=80
/ip firewall nat add action=netmap chain=dstnat dst-port=8080 in-interface=ether3 protocol=tcp to-addresses=192.168.10.3 to-ports=80 
```
We will configure the forwarding of the external port 2222 to the port 22 of the Admin
```
/ip firewall nat add action=netmap chain=dstnat dst-address=192.168.122.200 dst-port=2222 in-interface=ether3 protocol=tcp src-address=192.168.10.1/24 to-addresses=192.168.10.2 to-ports=22 
/ip firewall nat add action=netmap chain=dstnat dst-port=2222 in-interface=ether3 protocol=tcp to-addresses=192.168.10.2 to-ports=22
```

<center>

![](https://i.imgur.com/8h9SePR.png)
Figure 12: As a result, we got such a NAT table
</center>
---

:::info
e. Check that you can ssh to the Admin and access your web page from your workstation/laptop.
:::

## Implementation:

<center>

![](https://i.imgur.com/1aemK4j.png)
Figure 13: Checking the availability of the web server from the workstation using the curl utility
</center>
<center>

![](https://i.imgur.com/47WTVKx.png)
Figure 14: Checking ssh availability from a workstation
</center>
---





## References:

1: [Configuring port forwarding on the Mikrotik router](https://www.technotrade.com.ua/Articles/mikrotik-port-forwarding.php)
2: [Configuring a static IP address on ubuntu](https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/)
3: [Configuring the tap interface](https://gns3.com/community/featured/linux-tap-interface-user-access)