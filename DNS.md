# DNS

_Questions : On écrit au présent ou au passé ?_

_On se met d'accord sur la typo pour avoir la même modèle_

## Introduction

### DNS

The Domain Name System (DNS) is the hierarchical and distributed naming system used to identify computers reachable through the Internet or other Internet Protocol (IP) networks. [(Wikipedia)](https://en.wikipedia.org/wiki/Domain_Name_System)

Thus, first aim of a DNS server is to translate a domain name, understood by a human, into an IP address, understood by a computer.

For our configuration our associated domain name is "sos4.cc.uniza.sk".

### Primary and secondary architecture

To configure ours DNS servers we will assign different role to our two servers. We will use our server 1 as the primary server and server 2 as the secondary one.

This Primary and Secondary architecture is a common method to configure DNS explaining bellow.

The primary DNS server contains a DNS record that has the correct IP address for the hostname. If the primary DNS server is unavailable, the device contacts a secondary DNS server, containing a recent copy of the same DNS records.

There are three main benefits of having a secondary DNS server for a domain name:

-   Provide redundancy in case the primary DNS server goes down.
-   Distribute the load between primary and secondary servers.
-   Part of secure DNS strategy and preventing DDoS (Distribute Of Denial Service) attacks.
    [(NS1)](https://ns1.com/resources/primary-dns-vs-secondary-dns-and-advanced-use-cases)

_the following picture illustrate well how it works
[test](/What-is-Primary-DNS.svg)_

---

## Configuration

### Bind Installation

In order to configure a primary and secondary architecture from our server we will configure bind9 on both servers. First, install bind9.

`sudo apt-get install bind9`

Then, we also need to install utils of the package.

`sudo apt-get install bind9-utils`

Configuration will append in the bind folder accessible from the root by this command.

`cd /etc/bind`

### Primary server

In the _named.conf.options_ file, we put the forwarders 1.1.1.1 and 8.8.8.8. Which are respectively the public DNS to browse to the internet and Google's DNS.

```
forwarders {
    1.1.1.1.;
    8.8.8.8.;
 };
```

In the _named.conf.default-zones_ file, we include all the code with the default view as following.

```
view default { ... };
```

In the _named.conf.local_ file, we write inside view (aka private view) and public view respectively with private and public IP addresses.

On one hand, for the inside view, we only allow access (`match-clients` command) to private IP adresses which came from our local network 192.168.1.0/24 and from the localhost 127.0.0.0/8.

The zone configuration of our network entitled "sos4.cc.uniza.sk" is linked to the _zone.private_ file. Configurations will be saved in this file, as the primary server.

`allow-queries` from ? ... ? Same as `match-clients` ?

```
view inside {match-clients{192.168.1.0/24; 127.0.0.0/8;};
        zone "sos4.cc.uniza.sk" {
                file "/etc/bind/primary/zone.private";
                type primary;
                allow-query {192.168.1.0/24; 127.0.0.0/8;};
        };
};
```

On the other hand, for the public view, we allow access (`match-clients` command) to all IP adresses excepted ones which came from our local network 192.168.1.0/24 and from the localhost 127.0.0.0/8.

```
view public {match-clients{!192.168.1.0/24;any;};
        zone "sos4.cc.uniza.sk" {
                file "/etc/bind/primary/zone.public";
                type primary;
                allow-query {!192.168.1.0/24;any;};
        };
};
```

Create a folder _primary_ and files _zone.public_ and _zone.private_. In these zone files we define the zone names and domain names associated to the primary server.

This file starts with a Start Of Authority (SOA) record, which has information about the zone, like the main name server, the administrator's email, and how long to wait between refreshes.

Next come NS and A records for the name servers. We have two name servers ns1 and ns2 identified by the IP 158.193.153.105 for ns1.cc.sk and 158.193.153.110 for ns2.cc.sk in the public file case.

```
@ IN SOA ns1.cc.sk. admin.cc.sk. (
        2022031002 ;
        3H ;
        1H ;
        2W ;
        1H );

@ IN    NS ns1.sos4.cc.uniza.sk.
        NS ns2.sos4.cc.uniza.sk.

ns1 A 192.168.1.9
ns2 A 192.168.1.10
```

In the _zone.private_ file IP addresses 192.168.1.9 and 192.168.1.10 should replace the public addresses as below :

```
ns1 A 158.193.153.105
ns2 A 158.193.153.110
```

Be careful to the synthax, don't forget to put semicolons at the end of each line for the _named.conf_ files. And for the _zone_ files don't forget to put dots at the end of each domain name.

### Secondary server

We do the same things as in the primary server with the following differences :

-   We name the created folder _secondary_ instead of _primary_

-   In the _named.conf.local_ file replace 'primary' by 'secondary' and add the following ligne who associate this secondary server to the primary one with its IP address.

```
type secondary;
primaries { 192.168.1.9;};
```

-   _c'est tout ?_ Quid de l'histoire de sauvegarder les configurations des zones du primary dans le secondary ?

## Testing

To test if our DNS configuration is working, we need to check if computers inside and outside our network can find our zone files and point the domain to the right address.

In our case to verify we make bind ask himself on the localhost IP 0 or 127.0.0.1. and we should obtain ? ... ?

**For primary**

```
>> host ns1.sos4.cc.uniza.sk

```

**And for secondary**

```
>> host ns2.sos4.cc.uniza.sk

```

When it's working, ?... explanations\* ...?, we can see the IP address of the server tested (server 1 or server 2) as below :
