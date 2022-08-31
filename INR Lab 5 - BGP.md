---
tags: Internetworking and routing
---
:::success
# INR Lab 5 - BGP
Name: Ivan Okhotnikov
:::


## Task 1 - Preparation:

:::warning
a. Select a virtual routing solution that you would like to try. For example (Mikrotik, vyos, Pfsense).
b. GNS3 already has a template for these routers (Mikrotik, vyos, Pfsense), try to use these templates as it will save you a lot of time and troubleshooting.
c. Try to draw a network scheme before you start the lab. This will help you in the deployment phase.
d. The network scheme should include at least two networks, each one of them should have at least 3 routers, these routers can be the same routers from the OSPF lab.
e. This lab should be done in teams of 2, 3, or 4. As long as each team member uses a different network device
f. Connect one of your OSPF ASBR router interfaces (BGP interface) to your physical interface (bridge can also be used)
g. Agree with other teams on a subnet that your team will use (DO NOT USE 10.1.1.0/24)
h. Check that you can ping your teammates ASBR router from your ASBR router
:::

## Implementation

:::info
>a. Select a virtual routing solution that you would like to try. For example (Mikrotik, vyos, Pfsense).
b. GNS3 already has a template for these routers (Mikrotik, vyos, Pfsense), try to use these templates as it will save you a lot of time and troubleshooting.
c. Try to draw a network scheme before you start the lab. This will help you in the deployment phase.
d. The network scheme should include at least two networks, each one of them should have at least 3 routers, these routers can be the same routers from the OSPF lab.

<center>

![](https://i.imgur.com/YfTTd1z.png)
Figure 1: My topology
</center>
:::

:::info
>f. Connect one of your OSPF ASBR router interfaces (BGP interface) to your physical interface (bridge can also be used)
g. Agree with other teams on a subnet that your team will use (DO NOT USE 10.1.1.0/24)


We agreed to use a subnet
`217.1.1.0/24`
Where: 
`217.1.1.2` - this is me
`217.1.1.3` - this is Vladimir
`217.1.1.4` - this is Alexander

I connected BGP to the interface of my network card (enp1s0) and assigned it an ip address: `217.1.1.2/24`

To assign an ip address to Mikrotik, I used the command:
```
/ip address add address=217.1.1.2/24 interface=ether3
```
:::


:::info
> h. Check that you can ping your teammates ASBR router from your ASBR router

<center>

![](https://i.imgur.com/oem01KU.png)
Figure 2: Ping to `Alexander` and `Vladimir`
</center>
:::


## Task 2 - Deployment:

:::warning
a. Define an AS number that your ASBR router will use, again agree with the other teams on whichAS number each team will use (64512 to 65534)
b. Enable BGP and start advertising your OSPF network to your peer.
c. Can your peers reach your internal subnets? And can you reach their internal subnets?
d. How can the OSPF Internal router know about your peer’s OSPF Internal router? One way is to redistribute BGP routes into the OSPF routing table, but is this a practical method? why?
:::

## Implementation

:::info
> a. Define an AS number that your ASBR router will use, again agree with the other teams on which AS number each team will use (64512 to 65534)

I created a `BGP` instance:
```
routing bgp instance add router-id=10.10.6.5 as=65182 name=bgp65182 redistribute-ospf=yes redistribute-other-bgp=yes
```

In it, I specified the `router-id`, and my AS number=65182
Also I'm allowed `redistribute-ospf` and `redistribute-other-bgp` so that I can broadcast my OSPF and in the future become a transfer between `Vladimir` and `Alexander`

:::

:::info
> b. Enable BGP and start advertising your OSPF network to your peer.

After creating the instance, I set up a connection with `Alexander` and `Vladimir`, their IP addresses are `217.1.1.3` and `217.1.1.4`, respectively
To do this, I performed the addition of a `bgp peer` with the indication of its external interface `ether3`, `bgp-instance` and their `IP` and `remote-as`

To do this, I executed the following commands:

**For Alexander:**
```
/routing bgp peer add address-families=ip instance=bgp65182 name=Alexander remote-address=217.1.1.4 remote-as=65208 update-source=ether3
```

**For Vladimir:**
```
/routing bgp peer add address-families=ip instance=bgp65182 name=Vladimir remote-address=217.1.1.3 remote-as=65000 update-source=ether3
```


In order for the connection to work, they also needed to create a connection with me, specifying my `IP` and `remote-as`

After the connection is successfully established, I can see their routes (if they have allowed OSPF transmission)

<center>

![](https://i.imgur.com/Y0HXkjn.png)
Figure 3: Information about my `BGP instance`

![](https://i.imgur.com/9fY0elV.png)
Figure 4: Checking that the connection between me and my friends is established

![](https://i.imgur.com/P3MkVH8.png)
Figure 5: Received routes from `Vladimir` and `Alexander`
</center>
:::
:::info
> c. Can your peers reach your internal subnets? And can you reach their internal subnets?

For example, I will ping to one of the machines in `Vladimir's OSPF` 
(Figure 6):

<center>

![](https://i.imgur.com/9NT0EaI.png)
Figure 6: Ping to the address `10.10.3.2`, which belongs to `Vladimir`
</center>
:::

:::info
> d. How can the OSPF Internal router know about your peer’s OSPF Internal router? One way is to redistribute BGP routes into the OSPF routing table, but is this a practical method? why?

This approach is not practical because with a large number of BGP duplicates of IP addresses will not work
To allow this, you can use a VPN tunnel
:::

## Task 3 - Verification:

:::warning
a. How can you check if you have an established status with your peer?
b. How can you check in the routing table of ASBR and OSPF Internal routers to see which networks did you receive from your neighbors?
c. Use traceroute to verify that you have full BGP and OSPF connectivity.
:::

## Implementation

:::info
> a. How can you check if you have an established status with your peer?

I can check the connection statuses using the command:
```
/routing bgp peer print 
```
<center>

![](https://i.imgur.com/8L2oBH3.png)
Figure 7: My BGP connections
</center>
:::

:::info
> b. How can you check in the routing table of ASBR and OSPF Internal routers to see which networks did you receive from your neighbors?

I can use the `/ip r p` command to output all routes
If the gateway is the IP address of the partner, it means that I received the route from it

<center>

![](https://i.imgur.com/7qh2UiT.png)
Figure 8: My routes
</center>
:::

:::info
> c. Use traceroute to verify that you have full BGP and OSPF connectivity.

For example, I will reach the address in the OSPF of Ilya `10.10.6.1`, which is connected to Vladimir `217.1.1.3` via BGP with which I am connected

<center>

![](https://i.imgur.com/lQyfdZG.png)
Figure 9: `Ilya's` Topology
</center>

Using the command below, I can follow the route to Ilya's computer:

```
tool traceroute 10.10.6.1
```

<center>

![](https://i.imgur.com/fx3EQpZ.png)
Figure 10: Traceroute to Ilya's computer in OSPF
</center>
:::

## Task 4 - Transit:

:::warning
a. Peer with other teams and send them the routers that you receive from your peer in task 2
b. Can the new peer reach your network and your teammate network through you?
:::

## Implementation

:::info
>a. Peer with other teams and send them the routers that you receive from your peer in task 2

I am connected with `Vladimir` and `Alexander`
`Figure 11` shows the general connection diagram, and `Figure 12` shows a ping from `Vladimir` to the computer that is in my `OSPF`

<center>

![](https://i.imgur.com/E6EKGMm.png)
Figure 11: The scheme of our connection V - `Vladimir`, Alex - `Alexander`

![](https://i.imgur.com/y19I9r2.png)
Figure 12: Ping from `Vladimir` to the computer in my `OSPF`
</center>

:::

:::info
>b. Can the new peer reach your network and your teammate network through you?

Yes, they can
To do this, I specified the `redistribute-ospf` flag when creating my instance
:::

## Task 5 - ISP (Bonus)

:::warning
a. Become a transit for all/most of the networks that are available.
:::

## Implementation
:::info
a. Become a transit for all/most of the networks that are available.


I became a transit between Alexander and Vladimir
Vladimir has become a transit for the rest
For example, a ping from `Vladimir` to `Alexander` (Figure 14)

So that they can see each other's routes, I allowed `redistribute-other-bgp` in my instance
So that they can connect to each other, I set the nat masquerade rule to their IP addresses

**For Vladimir:**
```
/ip firewall nat add chain=srcnat action=masquerade src-address=217.1.1.3/24
```
**For Alexander:**
```
/ip firewall nat add chain=srcnat action=masquerade src-address=217.1.1.4/24
```

<center>

![](https://i.imgur.com/IgULGnv.png)
Figure 13: Alexander's topology

![](https://i.imgur.com/HGl8W8o.png)
Figure 14: Ping from `Vladimir` to `Alexander`
</center>
:::