# Network topology

## Introduction
To create an effective network, we need to first configure its topolgy, attributing the different servers ips, gateways and configuring the access to the internet.

In our topology, we have access to three servers and one router. The router will be the one managing the internet access. Two private servers will be allocated for the local faculty while the last server will be used by clients outside of the faculty.

### Ip address

An ip address is basically the authentification number of your device. While we can now also use the ipv6 type, we will configure in ipv4.
An ipv4 ip address consist of 4 bytes and thus range from 0.0.0.0 to 255.255.255.255.
We can dissociate these addresses into two types : private and public, private ipv4 address have been given special ranges : 
    - any address that begins with 10.
    - any address between 172.16.0.0 and 172.31.255.255
    - any address that begins with 168.192.
    
All of the servers will use private ip addresses to comunicate with the router. The public address will be allocated by the roouter and used to comunicate with the internet.

#### configuration
On server 1 and  2, find the directory : /etc/network/interfaces.d
in it there will be one file : 50.cloud.init.etc
place the choosen Ip. 
Server 1 will be 168.192.1.9 and Sever 2 168.192.1.10
