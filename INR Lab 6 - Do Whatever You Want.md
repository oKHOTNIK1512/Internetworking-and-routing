---
tags: Internetworking and routing
---
:::success
# INR Lab 6 - Do Whatever You Want
Name: Ivan Okhotnikov
:::

:::warning
Notes:
● Make the report as technical as possible (no installation guide please).
● Try to include a network scheme in your report
● If you paste some data (routing table), please make sure it is readable and the format did not change
● If you want to include a command in the report, please highlight it (bold, italic, different format, ...)
● You need to complete at least 2 task
:::

## Task 1 - Network Automation:
:::warning
a. Try to redo one of the labs that you have already done before but this time try to use configuration automation using for example some libraries in python or some scripting engine in your preferred network device
:::

I decided to automate the first laboratory work
There it was necessary on the router

In this laboratory work, it was necessary to configure the interfaces on Microtik, to make the Internet work

<center>

![](https://i.imgur.com/kejdtAR.png)
Figure 1: My topology
</center>


To solve the problem of automating this process, I used one feature of Microtik routers - their first port (ether1) acts as a dhcp client by default


Therefore, the first thing I did on the automation machine was to configure the dhcp server

To do this, I used
the isc-dhcp-client program
It is installed as follows: `sudo apt-get install isc-dhcp-server`

After successful installation, you need to configure everything:

`sudo nano /etc/dhcp/dhcpd.conf`
```
default-lease-time 7200;
max-lease-time 43200;

subnet 192.168.0.0 netmask 255.255.255.0 {
 range 192.168.0.4;
 option routers 192.168.0.1;
 option domain-name-servers 192.168.0.1;

}
```

I will specify the default connection rental time and the maximum
Then you need to specify the subnet in which the dhcp server will work and the mask
I chose one address as the ip address available for connection (so that I didn't have to search for the right client during the setup process)
It is also important to uncomment the line:
```
#authoritative;
```

This completes the configuration of the dhcp server



To automate work with telnet, I chose the `expect` program
It allows you to execute commands when necessary (in my case, if it meets the desired line)

The contents of my settings script:
```
#!/usr/bin/expect

# Preliminary timeout
set timeout 20

# Getting values from the console
set hostName [lindex $argv 0]
set userName [lindex $argv 1]
set ether2Address [lindex $argv 2]
set ether3Address [lindex $argv 3]
set ether4Nat [lindex $argv 4]


# Connecting to telnet
spawn telnet $hostName


# Authorization (since the microtic does not have a password , it will not be entered)
expect "Login:"
send "$userName\r"
expect "Password:"
send "\r";


# If the configuration is performed on a router that was launched for the first time
expect "\]: ";
send "n\r";

# I set static ip addresses (I take the subnets from the parameters that I received earlier)
expect "> ";
send "/ip address add address=$ether2Address.1/28 interface=ether2\r";
expect "> ";
send "/ip address add address=$ether3Address.1/28 interface=ether3\r";
expect "> ";
send "/ip address add address=$ether4Nat.200/24 network=$ether4Nat.1 interface=ether4\r";
expect "> ";

#Configuring masquerade rules for the subnet where web and admin are located
send "/ip firewall nat add action=masquerade chain=srcnat src-address=$ether2Address.1/28 \r"
expect "> ";

## port forwarding 8080 -> 80
send "/ip firewall nat add action=netmap chain=dstnat dst-address=$ether4Nat.200 dst-port=8080 in-interface=ether3 protocol=tcp src-address=$ether2Address.1/28 to-addresses=ether2Address.3 to-ports=80 \r"
expect "> ";
send "/ip firewall nat add action=netmap chain=dstnat dst-port=8080 in-interface=ether3 protocol=tcp to-addresses=$ether2Address.3 to-ports=80\r"
expect "> ";

## port forwarding 2222 -> 22
send "/ip firewall nat add action=netmap chain=dstnat dst-address=$ether4Nat.200 dst-port=2222 in-interface=ether3 protocol=tcp src-address=$ether3Address.1/24 to-addresses=$ether3Address.2 to-ports=22\r";
expect "> ";
send "/ip firewall nat add action=netmap chain=dstnat dst-port=2222 in-interface=ether3 protocol=tcp to-addresses=$ether3Address.2 to-ports=22\r";
expect "> ";

#It remains only to add a route for Internet access and disconnect from the configuration host (Automation) and everything will work
send "/ip route add dst-address=0.0.0.0/0 gateway=192.168.122.1 distance=1\r"
expect "> ";
send "/ip dhcp-client remove 0\r"
expect "> ";

```

Example of running a script:

```
./telnet.sh 192.168.0.4 admin+t 192.168.10 192.168.20 192.168.122
```

The first parameter indicates the ip address of the Mikrotik router
The second one is for setting the subnet for `ether2`
The third one is for setting a subnet for `ether3`
The fourth one contains information about the subnet for configuring the connection with `NAT`

After setting up the connection with me will be interrupted since I am forcibly disconnecting the connection using the dhcp client


## Implementation:


## Task 2 - Web Proxy:
:::warning
a. Try to configure a VM to use a proxy application that is installed on your host, This is important in case you want to do some traffic analysis on some application that is running inside the VM.
b. Try to deploy a transparent proxy
:::

## Implementation:
:::info
>a. Try to configure a VM to use a proxy application that is installed on your host, This is important in case you want to do some traffic analysis on some application that is running inside the VM.


Before starting work, I installed a squid proxy server on my server and configured it for incoming connections

`sudo nano /etc/squid/squid.conf`

```
http_port 8888
http_access allow all
```

Since my client was inside GNS3, and the proxy server was on my machine, the connection diagram looks like this (Figure 2):

<center>

![](https://i.imgur.com/EnBOYZz.png)
Figure 2: Connection diagram
</center>

On the client, I set the value for the environment variable so that it uses my proxy for http requests
```
export http_proxy="http://192.168.122.1:8888"
```

FigureN shows the process of installing a proxy server and connecting to it during an http request

<center>

![](https://i.imgur.com/8FrauIk.png)
Figure 3: Example of executing an http request using my proxy server
</center>

:::

:::info
> b. Try to deploy a transparent proxy

To do this, I need to listen to two ports
Basic and transparent
Otherwise, it will not work
```
http_port 3128
http_port 8888 transparent
```

In order for the transparent proxy to work, you need to add rules to my local machine (where squid is installed) iptables, which would allow redirecting requests from port 80 to my proxy server
This is done in this way:
```
sudo iptables -t nat -A PREROUTING -i virbr0 -p tcp --dport 80 -j REDIRECT --to-port 8888
sudo iptables -t nat -A PREROUTING -i virbr0 -p tcp --dport 80 -j DNAT --to 192.168.122.1:8888
```

<center>


![](https://i.imgur.com/GCDOFGh.png)
Figure 4: My iptables rules
</center>

Deployment is complete
It's time to move on to the tests
To do this, I need to see my http headers in the request
To do this, I wrote a script in the PHP programming language
The content of the script:
```
<?php
var_export($_SERVER);
?>
```

It is available at the link: `http://bot-os.ru/1.php`

Now I will send an http request from the client (after disabling the proxy from it)
To do this, I will use the curl command (figure 5)

<center>

![](https://i.imgur.com/cH4tDsJ.png)
Figure 5: curl request to `http://bot-os.ru/1.php`
</center>

The `HTTP_X_FORWARDED_FOR` header contains information about the client's ip address, and `HTTP_VIA` indicates that a connection was used through the squid proxy server


:::


## References:

1. [Bash scripts, part 11: expect and automation of interactive utilities](https://habr.com/ru/company/ruvds/blog/328436/)
1. [Squid Transparent proxy server : How to configure](https://linuxtechlab.com/squid-transparent-proxy-server-complete-configuration/)
2. [Iptables forward port 80 to remote squid](https://serverfault.com/questions/720369/iptables-forward-port-80-to-remote-squid)
