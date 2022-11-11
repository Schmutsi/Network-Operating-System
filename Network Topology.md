# Network topology

## Introduction

To create an effective network, we need to first configure its topology, attributing the different servers ips, gateways and configuring the access to the internet.

In our topology, we have access to three servers and one router. The router will be the one managing the internet access. Two private servers will be allocated for the local faculty while the last server will be used by clients outside of the faculty.

### Ip address

An ip address is basically the authentication number of a device. While we can now also use the ipv6 type, we will configure in ipv4.
An ipv4 ip address consist of 4 bytes and thus range from 0.0.0.0 to 255.255.255.255.
We can dissociate these addresses into two types : private and public, private ipv4 address have been given special ranges :

-   class a : any address that begins with 10.
-   class b : any address between 172.16.0.0 and 172.31.255.255
-   class c : any address that begins with 168.192.

All of the servers will use private ip addresses to communicate with the router. The public address will be allocated by the router and used to communicate with the internet.

### configuration

On server 1 and 2, find the directory : /etc/network/interfaces.d
in it there will be one file : 50.cloud.init.etc
place the chosen Ip.
Server 1 will be 168.192.1.9 and Sever 2 168.192.1.10

`

`

The mask and the /24 have a similar use, they serve to determine the range of ip address with which our own device can comumnicate directly. the /24 explains that 4 bits are used to define that range, as such any ip address that start with 168.192.1. is in the same range as our servers. For the markdown, 255 means that every ip address in the range has that same byte, and 0 that it changes for each of them. Any other number will make the byte more devided (for example, class b private addresses range is 17.16.0.0/12, 12 is not a multiple of 8, so the second byte is not completely allocated to name the range, as such, the markdown here is 255.240.0.0)
The gateway is the address that will represent this range.

For the router, we need to first allocate the different interfaces. Ens 3 is the one it will use when connecting to the outside world. it uses its public address (158.193.153.48). Ens 4 is the one it will use when interacting with server 1 and 2 and thus is defined by their gateway. and ens 5 is the one that will interact with the client server. This one uses a non-static address, as the client is always different.

## NAT

### Introduction

The Network Address Translation, is the way to allow the different server to interact with the internet through the router. We will need to explain to the router who it can allow easy access too, and how to interact with each address.

### configuration

To do this, we would first download iptables.
With this we can now define our interfaces roles in the network.

First we define localhost and ens3 our interface linked to the Internet as an inputs.
`sudo iptables -A INPUT -i lo -j ACCEPT  
sudo iptables -A INPUT -i ens3 -p tcp --dport 22  `
Now we can connect to the router through these ports ( localhost or port tcp22 of the ip address 158.163.105.48

Then we define ens3 as a postrouting.
`sudo iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE  `

and finally we can advance to the forwarding and tell the router how to react to queries from the private interfaces to the public one.
We want the private interfaces to be able to go to the internet wihout any problem, but we need the internet to be unable to access the private interfaces wihout prior request. This is what the stte Related Establish condition means.
`sudo iptables -A FORWARD -i ens3 -o ens4 -m state --state RELATED,ESTABLISHED -j ACCEPTED  
sudo iptables -A FORWARD -i ens3 -o ens5 -m state --state RELATED,ESTABLISHED -j ACCEPTED  
sudo iptables -A FORWARD -i ens4 -o ens3 -j ACCEPTED  
sudo iptables -A FORWARD -i ens5 -o ens3 -j ACCEPTED  `

Now that our Network address translation is working, we just need to address Portforwarding to allow us to access the sevrers from the router
`sudo iptables -t nat -A PREROUTING -p tcp --dport 2221 -j DNAT --to-destination 192.168.1.9:22  
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.1.10:22  
sudo iptables -t nat -A PREROUTING -p tcp --dport 2223 -j DNAT --to-destination 192.168.2.8:22  `

and now we can access all those in Putty by presnting the router address and changing the port to 2221, 2222 or 2223.


iptables is unable to save the configuration, meaning it would all diseapear if we reboot the router. to fix this problem, we need to download iptables-persistent
`sudo apt-get install iptables-persistent  ` and allow the saving of all our ip.v4 configuration.

We now have access to a document in /etc/iptables/ named rules.v4 where everything we've done is stored and saved.