---
tags: Internetworking and routing
---
:::success
# INR Lab 3 - VLANs & Fault Tolerance
**Name: Ivan Okhotnikov**
:::


## Task 1 - VLANs
:::warning
In some networks, you might need to share a switch between dierent services that shouldn’t be able to communicate with eachother for security, bandwidth or administration reasons. You will be doing that to your network but first you will need to extend it.
:::

## Implementation:

:::info
> Configure the switches and make sure you have connectivity between the hosts.

<center>

![](https://i.imgur.com/i9Qh1GK.png)
Figure 1: My network configuration in GNS3
</center>

For configuration, I will use my own VLAN (without tags)
To configure it, you need to register it in the cumulus network settings
`Figure 3` shows that the configuration was made successfully

```
auto bridge
iface bridge
    bridge-vlan-aware yes #the bridge uses a VLAN
    bridge-ports swp1 swp2 swp3 #ports that are part of the bridge
```
<center>

![](https://i.imgur.com/GWYAMaK.png)
Figure 2: Checking the settings on the HR computer that everything is configured successfully and I have access to `External` computers
</center>


:::


:::info
> How do VLANs work at a packet level?

    When using a virtual VLAN, each packet is signed with a tag 
    (the exception will be the native VLAN), which indicates which
    VLAN the traffic belongs to

> What are the two major protocols used for this?

    The following protocols are used:
    - 802.1 Q Trunk Protocol 
    - VTP (Vlan Trunking Protocol)
    
> What do we mean by Native VLAN?

    This is how the VLAN is designated, in which traffic is transmitted
    without a tag (there is no tag that would be attached to the frame).
    When the switch receives frames without tagging, it automatically
    designates them as VLAN Native

:::

:::info
> Configure the VLANs on the switches to isolate the two virtual networks
![](https://i.imgur.com/ewjDB58.png)


I was not able to correctly isolate clients using cumulus switches, so I did isolation at the gateway level (by creating vlan interfaces and creating rules for isolation)


First, you will need to create two virtual networks

The first VLAN will have the address: `10.10.1.1`
The second VLAN will have the address: `10.10.2.1`

To do this, I will create these two interfaces in Mikrotik
```
/interface/vlan/add interface=ether3 name=vlan10 vlan-id=10
/interface/vlan/add interface=ether3 name=vlan20 vlan-id=20
```

Both VLANs will exit the ether3 interface
The next step is to configure the ip address for vlan10 and vlan20

```
/ip/address/add address=10.10.1.1/24 interface=vlan10
/ip/address/add address=10.10.2.1/24 interface=vlan20
```

To limit them from each other I will add firewall filter rules
```
/ip/firewall/filter add chain=forward \
action=drop src-address=10.10.1.0/24 dst-address=10.10.2.0/24

/ip/firewall/filter add chain=forward \
action=drop src-address=10.10.2.0/24 dst-address=10.10.1.0/24
```

To configure the Internet connection, you need to add a masquerade rule for VLAN interfaces
```
/ip/firewall/nat/add action=masquerade \
chain=srcnat src-address=10.10.1.1/24 
/ip/firewall/nat/add action=masquerade \
chain=srcnat src-address=10.10.2.1/24 
```

It remains only to configure switches

Configuration example for Internal
```
auto swp2
iface swp2
 bridge-vids 20 # IDs of the VLANs used

auto swp3
iface swp3
    bridge-vids 10 20 # IDs of the VLANs used

auto bridge
iface bridge
    bridge-ports swp1 swp2 swp3 # ports of the bridge
    bridge-vlan-aware yes # the bridge uses a VLAN
    bridge-stp on
    bridge-vids 10 20 # IDs of the VLANs used
```


Configuration for Administration
```
auto bridge
iface bridge
    bridge-ports swp1 swp2 swp3 # ports of the bridge
    bridge-vlan-aware yes # the bridge uses a VLAN
    bridge-stp on # preventing bridge loops
    bridge-vids 10 20 # IDs of the VLANs used

auto swp2
iface swp2
    bridge-access 10 # declaring the physical port as the access port

auto swp3
iface swp3
    bridge-access 20 # declaring the physical port as the access port
```

Configuration for the IT Department

```
auto bridge
iface bridge
    bridge-ports swp1 swp2 # ports used for vlans
    bridge-vids 20 # IDs of the VLANs used
    bridge-stp on # preventing bridge loops
    bridge-vlan-aware yes # the bridge uses a VLAN

auto swp2
iface swp2
    bridge-access 20 # declaring the physical port as the access port
```
:::

:::info
> Ping between ITManager and HR , do you have replies?

There are no answers between them

<center>

![](https://i.imgur.com/2xykvGk.png)

Figure 3: Ping between ITManager and Manager
</center>

> Ping between ITManager and Management , do you have replies?

There is an answer between them and the ping command works
<center>

![](https://i.imgur.com/8VwTHiM.png)
Figure 4: Ping between ITManager and Management
</center>


> Capture the traic of the last ping and show in the packet the VLANs indication.

In the headers of the sent traffic in the wireshark program, you can see the VLAN identificationIn the headers of the sent traffic in the wireshark program, you can see the VLAN identification
<center>

![](https://i.imgur.com/HN0aRPQ.png)
Figure 5: Captured traffic in the wireshark program
</center>

:::


:::info
> Configure Inter-VLAN Routing between Management VLAN and HR VLAN

<center>

![](https://i.imgur.com/HnaKjXk.png)
Figure 6: firewall filter rules on Mikrotik
</center>

To set up routing between the HR VLAN and the management VLAN, I need to allow their interaction in the mikrotik forwarding rules
To do this, I will delete the rules that were created earlier

```
/ip/firewall/filter/remove 0
/ip/firewall/filter/remove 1
```

:::

:::info
> Show that you can now ping between them.

Now the ping command between them works (Figure 7)

<center>

![](https://i.imgur.com/ZAPS75A.png)
Figure 7: Ping between HR VLAN and Management VLAN
</center>

:::

:::info
> Capture the traffic going to and out of the router and show the different traffic of the sub-interfaces.


For this example, the traffic between HR and IT Department was caught (since I removed the isolation rules)

I can notice that before the gateway, the traffic has one VLAN signature, and after the gateway, it changes to the one in which subnet the receiving side is located

<center>

![](https://i.imgur.com/GEX9GtB.png)
Figure 8: Traffic on the way to the mikrotik gateway
</center>

<center>

![](https://i.imgur.com/FAZLwX9.png)
Figure 9: Traffic from the gateway to the client
</center>
:::



## Task 2 - Fault Tolerance

:::warning
Fault tolerence is important in a network but cables, routers and sotfware can be very unreliable. To mitigate this, we oen rely on redunduncy i.e. doubling everything. You'll be learning how to do this and what problems come out of it.
:::


## Implementation:

:::info
>1. What is Link Agregation ?

Link aggregation is the combination of several network connections in parallel in order to increase its throughput. This is necessary in order to achieve redundancy and fault tolerance (in case one or more connections are broken)

---
>How does it work (breifly.) ?


Several physical interfaces are linked into one and by balancing the traffic between the interfaces, a large bandwidth is achieved

---
>What are the possible configuration modes?

There are three modes for configuring link aggregation:

- LACP (Link Aggregation Control Protocol) is a standard protocol that performs tasks not only to increase throughput, but also to increase fault tolerance
- PAgP (Port Aggregation Protocol) is a Cisco network protocol for solving the same tasks as LACP
- Static aggregation-aggregation without using protocols (an error in the configuration can lead to the formation of loops)
:::

:::info
>Use link agregation between Web and the Gateway so that you have Load Balancing and Fault Tolerance .

To configure fault tolerance and load balancing, I first created the `bond0` interface, specified for it to work with the 2nd and 3rd layers of the OSI model, set the primary interface `ether2` and the slave interfaces to work `ether4, ether2`


**Mikrotik**
```
/interface/bonding/add name=bond0 mode=balance-alb \
transmit-hash-policy=layer-2-and-3  slaves=ether4,ether2 primary=ether2
```

Then I just need to configure the network for the necessary interfaces on the `Web` machine, in my case these are subnets`192.168.10.2/24` `192.168.30.2/24` 
(**Watch Figure 10**)

<center>

![](https://i.imgur.com/kQa9yFp.png)
Figure 10: Attached IP addresses to the interfaces on my Mikrotik gateway
</center>

**Web**
```
network:
    version: 2
    renderer: networkd
    ethernets:
        ens3:
          dhcp4: no
          addresses:
            - 192.168.10.2/24
          gateway4: 192.168.10.1 #ether2

        ens4:
          dhcp4: no
          addresses:
            - 192.168.30.2/24 
          gateway4: 192.168.10.1 #ether4
```

:::warning
Since I am connected to cumulus via a native VLAN that comes from Mikrotik, the order of naming the interfaces does not matter. The system itself will understand where to connect with what ip address
:::

:::info
> Test the Fault Tolerance by stoping one of the cables and see if you have any down time.

During the execution of the ping command, no downtime was detected (as well as interruptions)
:::

:::info
>7. Capture the traic send a boradcast ping request to the PCs connected to the Internal Network.

<center>

![](https://i.imgur.com/Ivew4g5.png)
Figure 11: Captured traffic in the wireshark program
</center>

---
>What can you notice?

I noticed repeated ARP and NDP requests, a loop formed in the network.
Request to broadcast ip 255 to detect the host (but this does not happen)

---
>Why did this happen?

This happened because a route was added that causes the network to loop (from HR LAN -> Administration - > it Department -> Administration, etc.)

---
>What are the consequences of this for the network ?

The consequence of such a situation may be a network failure (complete or partial)
:::
:::info
> 8. Enable back STP on the Switches and do the experiment again.
> Can you see STP traffic? Explain it breifly.

Yes, I can see the STP traffic
The STP protocol is designed to prevent loops in the network.
It creates a link tree that contains only one active path

<center>

![](https://i.imgur.com/PCyQzgo.png)
Figure 12: Captured traffic in the wireshark program
</center>


---
> Configure the switches to have the Internal as the Root switch.

To configure Internal as the root switch, you must specify the mstpctl-treeprio parameter for it.
By default, the value of this parameter is 32768.
This number must be a multiple of 4096.
The bridge with the lowest value of this parameter will be considered the root one.
Therefore, we can set it any value that is lower than 32768 and a multiple of 4096 (for example, 8192)

:::

:::info
> 9. Would we need STP between routers?

STP is usually used for switches and there is not much need for it in routers. In my case, Mikrotik already has RSPT support for the bridge because it performs two functions simultaneously.
:::

## References:
1. [Manual:Interface/VLAN](https://wiki.mikrotik.com/wiki/Manual:Interface/VLAN)
1. [VLAN wiki](https://ru.wikipedia.org/wiki/VLAN)
1. [Native VLAN XGU.ru](http://xgu.ru/wiki/Native_VLAN)
1. [VLAN Trunking with Cumulus Linux](https://blog.scottlowe.org/2015/06/25/vlan-trunking-cumulus-linux/)
1. [Link aggregation](https://en.wikipedia.org/wiki/Link_aggregation)
1. [Link aggregation XGU.ru](http://xgu.ru/wiki/%D0%90%D0%B3%D1%80%D0%B5%D0%B3%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D0%BA%D0%B0%D0%BD%D0%B0%D0%BB%D0%BE%D0%B2)