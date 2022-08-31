---
tags: Internetworking and routing
---
:::success
# INR Lab 2 - IPv4 & IPv6
**Name: Ivan Okhotnikov**
:::


## Task 1 - Ports and Protocols
:::info
 1. Check the open ports and listening Unix sockets against ssh and http on Admin and Web respectively.
:::

## Implementation:

To solve this problem, I chose the netstat command-line utility
It allows you to view open ports and listening sockets

To display the necessary information for us, you need to enter the command: ```netstat -pl```

:::success
Required program flags:
-l display of listening sockets
-n show numerical address
:::
<center>

![](https://i.imgur.com/kzFxir4.png)
Figure 1: Open ports and listening sockets for the Web

![](https://i.imgur.com/uyBZRmg.png)
Figure 2: Open ports and listening sockets for the Admin
</center>

:::info
 22 - port for ssh connection
 80 - port for http connection
:::

----
:::info
2. Scan your gateway from the outside. What are the known open ports?
:::

## Implementation:

To display the necessary information for us, you need to enter the command: ``` sudo nmap 192.168.122.200```
<center>

![](https://i.imgur.com/m6mqFCh.png)
Figure 3: Open ports for the gateway
</center>

---

:::info
3. A gateway has to be transparent, you should not see any port that is not specifically forwarded. Adjust your firewall rules to make this happen. Disable any unnecessary services and scan again.
:::

## Implementation:

In my case, the gateway is Mikrotik
Since most of the open ports are opened by the services of the gateway itself, it is necessary to disable them

To view the services, use the command: ```/ip service print  ```

<center>

![](https://i.imgur.com/Egb8Pdl.png)
Figure 4: Microtik services
</center>

To disable unnecessary services, use the command: ```/ip service disable x```, where x is number of service

<center>

![](https://i.imgur.com/poubl0w.png)
Figure 5: Disabled Microtik services
</center>

When all services are disabled, one port remains open
This port is ```2000```

To close it, enter the command: ```/tool bandwidth-server set enabled=no```
 
<center>

![](https://i.imgur.com/dY6NZUF.png)
Figure 6: Re-checking open ports
</center>


---
:::info
**4. It suppose that some scanners start by scanning the known ports and pinging a host to see if it is alive.**
4.1. Scan the Worker VM from Admin. Can you see any ports?
4.2. Block ICMP traffic on Worker and change the port for SSH to one that is above 10000.
4.3. Scan it without extra arguments.
4.4. Now make necessary changes to the command to force the scan on all possible ports.
4.5. Gather some information about your open ports on Web ( ssh and http )
:::

## Implementation:

:::info
**4.1. Scan the Worker VM from Admin. Can you see any ports?**

The scan showed 1 open port - 22
It belongs to the ssh service

<center>

![](https://i.imgur.com/3oK5Aa0.png)

Figure 7: Open port of the Worker
</center>
:::

:::info
**4.2. Block ICMP traffic on Worker and change the port for SSH to one that is above 10000.**


To close the traffic for ICMP, you need to make edits to iptables:
```
sudo iptables -A INPUT --proto icmp -j DROP
```
<center>

![](https://i.imgur.com/OqA6FYB.png)
Figure 8: To check that the traffic is limited for ICMP, we will execute the ping command from the Admin machine to the Worker machine
</center>

---

In order to change the ssh port, you need to make edits to the ``/etc/ssh/sshd_config`` file: 

```
sudo nano /etc/ssh/sshd_config
```
<center>

![](https://i.imgur.com/qbuh6YQ.png)
Figure 9: Change the port to 22000
</center>

After saving the changes, you need to restart the sshd service
```
 sudo service sshd restart
```
:::

:::info
**4.3. Scan it without extra arguments.**
<center>

![](https://i.imgur.com/qPbXDwX.png)
Figure 10: Scanning the Worker did not give any results, due to the fact that ports above 10000 are not checked
</center>
:::

:::info
**4.4. Now make necessary changes to the command to force the scan on all possible ports.**

For advanced scanning, you must specify a range of ports:
```
sudo nmap -p0-65535 -f 192.168.20.2
```
<center>

![](https://i.imgur.com/lcfgi11.png)
Figure 11: The scanner showed an open port at the Worker 
</center>


:::

:::info
**4.5. Gather some information about your open ports on Web ( ssh and http )**

<center>

![](https://i.imgur.com/iXf11jq.png)
Figure 12: Open ports on the `Web` machine
</center>
:::

---


## Task 2 - Traffic Captures & IPv6

:::info
**1. Access your Web Page from the outside and capture the traffic between the gateway and the bridged interface.**
- Can you see what is being sent?
- What kind of information can you get from this?
- What do the headers mean?
:::

## Implementation:
:::info
**Can you see what is being sent?**

I can see the initialization process of sending the request, the request itself and the end of sending

<center>

![](https://i.imgur.com/SFEZIpx.png)
Figure 12: Captured traffic
</center>
:::

:::info
**What kind of information can you get from this?**
I can see:
- Request protocol
- Request weight
- Mac and port and ip address of the recipient
- Request content (headers and payload)
:::

:::info
**What do the headers mean?**
---

**HTTP headers include:**
- Server host
- Request path
- User Agent
- Sending method
- The http version

**TCP headers include:**
- Weight
- Sequence number
- Acknowledge
- Start and end port
- The checksum
- The status of the checksum
- Flags
:::

---
:::info
**2. SSH to the Admin from the outside and capture the traffic (make sure to start capturing before connecting to the server).**
- Can you see what is being sent?
- What kind of information can you get from this?
- What are the names of the ciphers used?
:::

## Implementation:
:::info
**Can you see what is being sent?**

I can see:
- Setting up a `TCP` connection
- Selection of encryption algorithms, creation of a session key (which is destroyed when the connection is closed),
- User authorization (can be with a key or password)
- After a successful connection, the `Workstation` can send commands to the `Worker` machine

<center>

![](https://i.imgur.com/aXRYnmq.png)

Figure 13: Wireshark Traffic Capture Window
</center>


:::

:::info
**What kind of information can you get from this?**
Unlike an http request, I can't view the contents of the requests, but I can find out information about the sender and recipient

:::

:::info
**What are the names of the ciphers used?**
In my case, an algorithm was used for data encryption and authorization `chacha20-poly-1305@openssh.com`. The Diffie-Hellman algorithm was used to generate the session key
    
:::

---
:::info
**3. Configure Burp Suite as a proxy on your machine and intercept your HTTP traffic.**
- Show that you can modify the contents by changing something in the request.
- Why are you able to do this here and not in an SSH connection?
- Do you know any other tools that are analogues to Burp suite ? List and give a one-line description of them.
:::

## Implementation:

:::info
**Show that you can modify the contents by changing something in the request.**

Having previously launched and configured a proxy for the ``Burp Suite`` program on the computer, we will make a request in the address bar (in my case, this is a ``Google site``)

**The page will not load until we allow it in the program**

After sending the request to google.com Our request will open in the ``Burp Suite`` program.

<center>

![](https://i.imgur.com/y0vlZd5.png)
Figure 14: The request sent to the server
</center>

I can modify it right in this window. After changing the request, click ``Forward``

<center>

![](https://i.imgur.com/7XTKbsh.png)
Figure 15: Modified request to the server
</center>

I can also intercept and modify the response from the server
Let's change the response from the server for demonstration

<center>

![](https://i.imgur.com/CiSbSzq.png)
Figure 16: Modified response from the server
</center>

<center>

![](https://i.imgur.com/bLHHpqS.png)
Figure 17: Result of changing the response
</center>



:::

:::info
**Why are you able to do this here and not in an SSH connection?**
Using an http proxy, you can only manage http requests
Due to the fact that ssh does not work via http, it will not be possible to replace requests
:::

:::info
**Do you know any other tools that are analogues to Burp suite ? List and give a one-line description of them.**

**Charles** - HTTP debugging and testing application written in Java.
**Telerik Fiddler** - Program used to register changes and check HTTP / S traffic between a computer and a web server.
:::


---
:::info
4. Configure IPv6 from the Web Server to the Worker. This includes IPs on the servers and the default gateways.
:::

## Implementation:

To configure ipv6, you must first configure mikrotik to work with ipv6

```/ipv6 address add address=x interface=y```, where x is the ipv6 address, y is the interface

<center>

![](https://i.imgur.com/ySlUQ2y.png)
Figure 18: Configuration for the gateway
</center>

After configuring the gateway, we will configure ipv6 operation on other devices (for example, the `Admin` machine is specified)

```sudo nano /etc/netplan/50-cloud-init.yaml```
<center>

![](https://i.imgur.com/d5EE8fm.png)

Figure 19: Configuration for the `Admin` machine
</center>
To apply the settings, enter the command: ``sudo netplan apply```



---

:::info
**5. Access the Web Page using IPv6 from `Admin` while capturing again.**
- Can you see the difference? What's the difference?
- Attach you IPv6 captures in a folder captures with your report.
:::
## Implementation:

Curl was used to send a request to the web server
```
curl http://[2001:db8:0:1::3]
```
The IP addresses of the recipient and sender have changed
The Internet Protocol Version 6 drop-down list has been added to the requests

<center>

![](https://i.imgur.com/Uczi8N5.png)
Figure 20: Response from the IPv6 `Web` machine

</center>


---



## References:
1. [How to block or unblock ping requests on Ubuntu Server 20.04 LTS](https://linuxhint.com/block-unblock-ping-requests-to-ubuntu-server/)
1. [Check listening ports with netstat](https://docs.rackspace.com/support/how-to/checking-listening-ports-with-netstat/)
1. [Running a quick NMAP scan to inventory my network](https://www.redhat.com/sysadmin/quick-nmap-inventory)
1. [How to change the ssh port on Linux or Unix server](https://www.cyberciti.biz/faq/howto-change-ssh-port-on-linux-or-unix-server/)
1. [Host Discovery (“Ping Scanning”)](https://nmap.org/book/host-discovery.html)
1. [Capturing Live Network Data](https://www.wireshark.org/docs/wsug_html_chunked/ChCapCapturingSection.html)
1. [Manual:IPv6/Address](https://wiki.mikrotik.com/wiki/Manual:IPv6/Address)
1. [Order your CloudVPS with SSDOnly
How to configure IPv6 with Netplan on Ubuntu 18.04](https://www.snel.com/support/how-to-configure-ipv6-with-netplan-on-ubuntu-18-04/)
1. [How To Scan All TCP and UDP Ports with Nmap?](https://www.poftut.com/how-to-scan-all-tcp-and-udp-ports-with-nmap/)