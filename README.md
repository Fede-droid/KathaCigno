# How to implement IPtables through Katharà
The steps that you will find in this guide are the following:

1. Introduction

2. My network scheme

3. Basic device configuration

4. Introduction to firewalls and iptables

5. IPtables implementation

6. Test

## Introduction
Hello everyone, I am Federico, a graduate in computer engineering from the University of Pavia.

In this guide, I will show you how to configure network devices using the network I have developed. I will explain how to implement IPtables using Katharà and how to manage access requests to various web servers through a dynamic method. Let's get started!

## My network scheme
![image](https://github.com/Fede-droid/KathaCigno/assets/80335626/11fb55dd-53ab-4397-89ab-46c11dd74f08)

Here you can see two public networks in blue and one private network in green. These networks are represented by letters and consist of various devices such as clients, routers, and servers. Each device has two cells next to it, containing letters and numbers. 

- The upper cell indicates the device's final IP address;
- The lower cell indicates the associated network interface.

For example, let's consider the "public network 2". We observe the "client1", which has the IP address 11.0.0.7. The same applies to all other devices in the network. The router, in particular, has two network interfaces and therefore two IP addresses since it combines two different domains.

## Basic device configuration
### First step
Create the file "lab.conf" and enter all the network devices associated with their network interface and domain, like this:

> client1[0]=C  
- 0 mean eth0
- 1 mean eth1

Repeat the same process for all other devices.

### Second step
For each device, create a configuration file named:

> [device].startup

Inside the file, include all the necessary commands to ensure that all devices can successfully send ping requests.

### Third step
Since I used Apache web servers, each server has a dedicated HTML page. Create the following directories and files for each server:

> [nameserver]/var/www/html/index.html


## Introduction to firewalls and iptables
A Firewall is "a network security device that monitors incoming and outgoing traffic using a predefined set of security rules to allow or block events." It can be a hardware device or a software application.

Iptables is one of the main components for network security in a Linux system. It is a kernel-level firewall that allows us to filter traffic by creating rules. Iptables operates based on a set of rules that determine whether to allow or drop packets.

### Properties in my network:

I have a private network, so only devices within the network can send packets to the external world. Conversely, a device in the public network, for example, cannot send ping requests to the private network.
Additionally, as you can see in the image, "public network 1" can be considered as a server farm. The characteristic of this network is that we have a main server, "server2," and the other two servers support the main server.


## Iptables implementation
### r3.startup
Let's start by examining this router, which manages our private network.

> iptables-legacy -A FORWARD -i eth1 -o eth0 -j DROP

This command ensures that any traffic coming from the outside (through the eth1 network interface) and forwarded (as observed in the FORWARD command) to the private network (towards the eth0 network interfaces) is dropped (-j DROP).

However, a device within the private network can send ping requests correctly without any issues. The problem is that the return packets will not be allowed due to the newly added IPtables rule.

To solve this problem, we use the following command:

> iptables-legacy -A FORWARD -i eth1 -o eth0 -m state --state ESTABLISHED -j ACCEPT

This command states that any traffic coming from eth1 and forwarded to eth0, which is part of an established connection (-m state --state ESTABLISHED), should be accepted (-j ACCEPT).

In summary, we have used IPtables rules to manage traffic at the FORWARDING level.

Now let's move to the INPUT level.

The same logic applies to the router itself since it is part of the private network and cannot receive packets from the external world (public networks). The following two IPtables rules are sufficient:

> iptables-legacy -A INPUT -i eth1 -m state --state ESTABLISHED -j ACCEPT

> iptables-legacy -A INPUT -i eth1 -j DROP
#### NAT
We are in a private network, so to ensure correctness, we need to make sure that packets leaving the private network do not have the internal device's IP address but rather the router's IP address. Here comes the NAT (Network Address Translation):

> iptables-legacy -t nat -A POSTROUTING -s 10.0.0.0/16 -j MASQUERADE

With this rule, we specify that packets with a source address (-s) of 10.0.0.0/16 should be masqueraded (-j MASQUERADE) with the router's IP address. The IP address is automatically changed in the POSTROUTING phase.
### r2.startup
This router manages the server farm. In the farm, I made an assumption: we have a main server, SW2, and two support servers. Essentially, only server2 is visible to the networks, so devices will request access to the HTML page of server2.

However, I don't want to overload the main server with all the requests. Therefore, I implemented a dynamic method using IPtables, called Round Robin Load Balancing.

> iptables-legacy -A PREROUTING -t nat -p tcp -d 1.0.0.1 --dport 80 
                -m statistic --mode nth --every 3 --packet 0 -j DNAT --to-destination 1.0.0.1:80

> iptables-legacy -A PREROUTING -t nat -p tcp -d 1.0.0.1 --dport 80 
                -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 1.0.0.6:80

> iptables-legacy -A PREROUTING -t nat -p tcp -d 1.0.0.1 --dport 80 
                -j DNAT --to-destination 1.0.0.2:80

In essence, the requests are dynamically distributed among the various servers. The first request is directly handled by SW2, the second by SW3, the third by SW4, and if there is a fourth request, it goes back to being handled by SW2, and the cycle continues.

I assumed that since they are support servers, they are not accessible from external networks. Then we have the Iptables rules:

> iptables-legacy -A FORWARD -i eth1 -d 1.0.0.6 -m state --state ESTABLISHED -j ACCEPT
>
> iptables-legacy -A FORWARD -i eth1 -d 1.0.0.6 -j DROP
>
> iptables-legacy -A FORWARD -i eth1 -d 1.0.0.2 -m state --state ESTABLISHED -j ACCEPT
>
> iptables-legacy -A FORWARD -i eth1 -d 1.0.0.2 -j DROP

## Test
Now, what I recommend, and what I have done myself, is to execute the project using Katharà's personal commands and try sending ping requests from one device to another. You will see that everything works fine. :)
## Conclusion
I hope this guide serves you well in understanding how to configure IPtables rules and manage internal devices in private and public networks.

Thank you all for your attention!

See you soon,
Federico
