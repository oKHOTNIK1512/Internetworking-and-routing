---
tags: Internetworking and routing
---
:::success
# INR Lab 4 - OSPF
**Name: Ivan Okhotnikov**
:::


## Task 1 - Prepare your network topology
:::warning
 1. In the GNS3 project, select and install a virtual routing solution that you would like to use:
Mikrotik (recommended), Pfsense, vyos and so on.
 2. Prepare a simple network consistingdefault route of at least 3 routers, each one of them has a different subnet, and they should be able to reach each other (for example by a switch/router in the middle or a bus topology).

:::

## Implementation:

<center>

![](https://i.imgur.com/Cp1CIGA.png)
Figure 1: My topology
</center>

My topology consists of 7 subnets
**Router 1:**
```
10.10.1.1/24
10.10.2.1/24
```

**Router 2:**
`10.10.10.1/24`

**Router 3:**
`10.10.20.1/24`

**Router 4:**
```
10.10.30.1/24
```
**Router 5:**
```
10.10.40.1/24
10.10.50.1/24
```


## Task 2 - OSPF Learning & Configuring

:::warning
1. Deploy OSPF in your chosen network topology.
2. Which interface you will select as the OSPF router ID and why?
3. What is the difference between advertising all the networks VS manual advertising (per interface or per subnet)? Which one is better?
4. If you have a static route in a router, how can you let your OSPF neighbors know about it? Approve and show it on practice.
5. Enable OSPF with authentication between the neighbors and verify it.
6. Bonus: if one of the routers has multiple subnets, try to use route summarization
:::

## Implementation:
:::info
> 1. Deploy OSPF in your chosen network topology.


OSPF - A protocol for implementing dynamic routing. This protocol works on the basis of channel state tracking technology.
It uses Dijkstra's algorithm to find the shortest path.

To deploy OSPF, you need to configure this protocol on all routers
To do this, I will use the command:
`routing ospf network add area=backbone network=X`
Where `X` is the network used. 

For example, for `Router 1`:
`routing ospf network add area=backbone network=10.10.1.0/24`
`routing ospf network add area=backbone network=10.10.2.0/24`

After successful configuration I can see that the routes have appeared:
`/ip/route/print`

<center>

![](https://i.imgur.com/VoHHWzp.png)
Figure 2: Routes for `Router 1`
</center>

<center>

![](https://i.imgur.com/Cp1CIGA.png)
Figure 3: My topology
</center>

Since there are routes, I can ping other clients (for example, client 4, which is distant)
<center>

![](https://i.imgur.com/rp35CHG.png)
Figure 4: Ping command to other `clients`
</center>
:::

:::info
> 2. Which interface you will select as the OSPF router ID and why?

To determine the route, I need to select an interface that will always be active. To do this, I can select any active network interface.
:::

:::info
> 3. What is the difference between advertising all the networks VS manual advertising (per interface or per subnet)? Which one is better?


If I don't want to show my internal subnet, I will limit myself to manually prescribing the necessary routes (manual advertising)
If it is important for me that all my internal subnets are visible, I will use advertising in all networks
:::

:::info
> 4. If you have a static route in a router, how can you let your OSPF neighbors know about it? Approve and show it on practice.

I can report using `redistribute-static=as-type-1`
It allows you to redistribute static routes

`type1` - a metric for OSPF that describes the sum of the internal and external cost of the route

For example, I will add Client6 to the topology
I will assign him a static IP in Microtik: `10.10.60.2`

**Router5:**
```
/ip/address/add interface=ether4 address=10.10.60.1/24
```

<center>

![](https://i.imgur.com/x45tJbi.png)
Figure 5: Add client to my topology
</center>

Now you need to inform the others that such a client has appeared

**Router1:**
Setting the route:
```
/ip/route/add dst-address=10.10.60.0/24 gateway=10.10.2.2 distance=110
```

`gateway` - IP address **Router 5**
`dst-address` - the subnet to which I configure routing

I am saying that the new route is important and it must be used:
```
/routing/ospf/instance set 0 redistribute-connected=as-type-1
```
<center>

![](https://i.imgur.com/KydDwHz.png)
Figure 6: List of routes including the new one

![](https://i.imgur.com/9ADdVTW.png)
Figure 7: Checking that I can ping a new client
</center>

:::

:::info
> 5. Enable OSPF with authentication between the neighbors and verify it.


To enable authorization, I need to add the ospf interface, the authorization key and the authorization type (I will use simple and md5). This is necessary for validating traffic at the interface level

<center>

![](https://i.imgur.com/x45tJbi.png)
Figure 8: My topology
</center>

For example, I have paved the authentication access between Client1 `10.10.10.2` and Client4 `10.10.40.2`

To do this, enter the commands on the router:

**Router 1**
```
/routing ospf interface add interface=ether1 authentication=md5 authentication-key=md5hashkey
/routing ospf interface add interface=ether2 authentication=simple authentication-key=testtest
```

**Router 2**
```
/routing ospf interface add interface=ether1 authentication=md5 authentication-key=md5hashkey
/routing ospf interface add interface=ether2 authentication=md5 authentication-key=md5hashkey
```

**Router 5**
```
/routing ospf interface add interface=ether1 authentication=simple authentication-key=testtest
/routing ospf interface add interface=ether2 authentication=simple authentication-key=testtest
```

To summarize: `Router 1` implements authentication using a simple method on one interface, and using `md5` on the other (figure 9)

<center>

![](https://i.imgur.com/R3HCpUy.png)
Figure 9: interfaces configured for authorization in `Router1`
</center>

<center>

![](https://i.imgur.com/GSy0Pi1.png)
Figure 10: Checking with the ping command that everything is configured correctly
</center>


However, other clients will lose access to the network because their routers are not configured for authentication. I can fix this by enabling authentication for everyone

<center>

![](https://i.imgur.com/SHNw9Dm.png)
Figure 11: Checking the availability of the subnet with the ping command on a router that is not configured for authentication
</center>

:::

:::info
> 6. Bonus: if one of the routers has multiple subnets, try to use route summarization

The tasks above have already been completed with this condition

:::




## Task 3 - OSPF Verification


:::warning
1. How can you check if you have a full adjacency with your router neighbor?
2. How can you check in the routing table which networks did you receive from your neighbors?
3. Use traceroute to verify that you have a full OSPF network.
4. Which router is selected as DR and which one is BDR ?
5. Check what is the cost for each network that has been received by OSPF in the routing table.
:::

## Implementation:

:::info
> 1. How can you check if you have a full adjacency with your router neighbor?

I can check this with the command: `/routing ospf neighbor print`
For example, I can give the output of this command for `Router5` (figure 12)
<center>

![](https://i.imgur.com/OlCylAH.png)
Figure 12: Neighbors `Router5`
</center>

:::

:::info
> 2. How can you check in the routing table which 
networks did you receive from your neighbors?

To do this, I can use the command:`/ip r p`
An example of the response for `Router5` is presented in `Figure 13`
<center>

![](https://i.imgur.com/UBaQU0A.png)
Figure 13: Routing tables from neighbors for `Router5`
</center>


:::

:::info
> 3. Use traceroute to verify that you have a full OSPF network.
> 
Using the trace utility, I can see the hosts through which I pass to access the desired IP address
For example, I performed a trace from Client1 (Router2) to Client4 (Router 5)
<center>

![](https://i.imgur.com/Rh5OVkb.png)

Figure 14: trace from `Client1` to `Client4`
</center>


:::

:::info
> 4. Which router is selected as DR and which one is BDR?

`DR` - designated-router (it has the highest priority)
`BDR` - backup-designated-router

If the assigned router stops responding, its priority will be lowered and a backup router will take its place

To see which host is used as a DR, and which BDR I can use the command: `/routing ospf interface print status` (`Figure 15` for `Router1`)
For each `OSPF` interface, `DR` (designed-router), priorities and `BDR` (backup-designed-router) will appear (if there is one)

<center>

![](https://i.imgur.com/7d3PSsp.png)
Figure 15: information about `OSPF` interfaces for `Router1`
</center>

:::

:::info
> 5. Check what is the cost for each network that has been received by OSPF in the routing table.

For this check, I can use the command: `/routing ospf route print`
The output of this command (`Figure 16`) will contain the destination address, state, cost, gateway and interface (to which it belongs)
<center>

![](https://i.imgur.com/PoBPNoq.png)
Figure 16: `OSPF` routes for `Router2`
</center>


:::



## Bonus - Multi-Area network

:::warning
1. In the case if until now every router was in area 0 , try to create more networks and assign them to different OSPF areas.
2. Why every area has to be connected to area 0 ?
3. What can you do if you have an area X which is not connected to area 0, but to area Y ?
4. Verify that area X can reach all the network.
:::

## Implementation:

:::info
> 1. In the case if until now every router was in area 0 , try to create more networks and assign them to different OSPF areas.

Before starting work, I modify the topology

<center>

![](https://i.imgur.com/ICbz0Po.png)
Figure 17: Modified topology
</center>

To create a new area, you need to add it to Mikrotik (Router6)
This is done with the following command: `/routing ospf area add name=area_80 area-id=0.0.0.80`
Where area-id is the area ID (you will need it)
Creating the same area in Router7 (they will be linked by ID)

It remains only to add the ospf network with `area=area_80` and everything will work (Figure 18):

**Router7**
```
/routing ospf area add name=area_80 area-id=0.0.0.80
/routing ospf network add area=area_80 network=10.10.80.0/24
/routing ospf network add area=area_80 network=10.10.90.0/24
```

**Router6**
```
/routing ospf area add name=area_80 area-id=0.0.0.80
/routing ospf network add area=area_80 network=10.10.70.0/24
/routing ospf network add area=area_80 network=10.10.80.0/24
```
:::

:::info
> 2. Why every area has to be connected to area 0?

In subnets, OSPF is limited. It is because of this that there should be a backbone area (backbone) or area 0 as the main one, and only then all other areas are connected to it (either using a physical or virtual connection)
The reason for this is the distance vector used by the inter-regional routing.

:::

:::info
> 3. What can you do if you have an area X which is not connected to area 0, but to area Y ?

I can configure area X as a transit zone to connect area 0 and area Y
:::

:::info
> 4. Verify that area X can reach all the network.


<center>

![](https://i.imgur.com/E8jnH8O.png)
Figure 18: Checking what PC1 can do ping PC2
</center>
:::


## References:
1. [Mikrotik OSPF specification](https://mikrotik.axiom-pro.ru/library/mtman/OSPF.pdf)
2. [Mikritik Manual:Routing/OSPF](https://wiki.mikrotik.com/wiki/Manual:Routing/OSPF)
3. [OSPF DR/BDR Election explained](https://networklessons.com/ospf/ospf-drbdr-election-explained)
5. [Why must we go to area 0?](https://networkinferno.net/why-must-we-go-to-area-0)
6. [OSPF with no backbone area...](http://www.labnfun.ru/2017/12/ospf-with-no-backbone-area.html)