# Network topology

## Introduction
To create an effective network, we need to first configure its topolgy, attributing the different servers ips, gateways and configuring the access to the internet.

In our topology, we have access to three servers and one router. The router will be the one managing the internet access. Two private servers will be allocated for the local faculty while the last server will be used by clients outside of the faculty.

### Ip address

An ip address is basically the authentification number of your device. While we can now also use the ipv6 type, we will configure in ipv4.
An ipv4 ip address consist of 4 bytes and thus range from 0.0.0.0 to 255.255.255.255.
We can dissociate these addresses into two types : private and public, private ipv4 address have been given special ranges : 
    - class a : any address that begins with 10.
    - class b : any address between 172.16.0.0 and 172.31.255.255
    - class c : any address that begins with 168.192.
    
All of the servers will use private ip addresses to comunicate with the router. The public address will be allocated by the roouter and used to comunicate with the internet.

#### configuration
On server 1 and  2, find the directory : /etc/network/interfaces.d
in it there will be one file : 50.cloud.init.etc
place the choosen Ip. 
Server 1 will be 168.192.1.9 and Sever 2 168.192.1.10


<b><i> write code here</i></b>
    
The mask and the /24 have a similar use, they serve to detremine the range of ip address with which our own device can comunicate directly. the /24 explainthat 4 bits are used to define that range, as such any ip address that start with 168.192.1. is in the same range as our servers. For the markdown, 255 means that every ip address in the range have that same byte, and 0 that it change for each of them. any other number will make the byte more devieded (for exemple, class b private addresses range is 17.16.0.0/12, 12 is not a multiple of 8, so the second byte is not completly allocated o name the range, as such, the markdown here is 255.240.0.0)
The gateway is the address that will represent this range.

For the router, we need to first allocate the different interfaces. Ens 3 is the one it will use when connecting to the outisde world. it uses its public address (158.193.153.48). Ens 4 is the one it will use when interacting with server 1 and 2 and thus is defined by their gateway. and ens 5 is the one that will interact with the client server. This one uses a non-static address, as the client is always different.


### NAT

#### Introduction

The Network Address Translation, is the way to allow the differents server to interact with the internet through the router. We will need to explain to the router who it can allow easy access too, and how to interact with each address.

#### confguration

To do this, we first download iptables

<b><i> write code here</i></b>

with it we can now co,figurate the bind.
