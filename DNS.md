# DNS

## Introduction

### Domain Name System

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

### Installing Bind

In order to configure a primary and secondary architecture from our server we will configure bind9 on both servers. First, install bind9.

`$ sudo apt-get install bind9`

Then, we also need to install utils of the package.

`$ sudo apt-get install bind9-utils`

Configuration will append in the bind folder accessible from the root by this command.

`$ cd /etc/bind`

### Configuring primary server

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

On one hand, for the inside view, we only allow access to private IP addresses which came from our local network 192.168.1.0/24 and from the localhost 127.0.0.0/8. We do it with the `match-clients` command which restricts the listening of IPs and with `allow-queries` command which authorize actions of the following IPs.

The zone configuration of our network entitled "sos4.cc.uniza.sk" is linked to the _zone.private_ file. Configurations will be saved in this file, as the primary server.

```
view inside {match-clients{192.168.1.0/24; 127.0.0.0/8;};
        zone "sos4.cc.uniza.sk" {
                file "/etc/bind/primary/zone.private";
                type primary;
                allow-query {192.168.1.0/24; 127.0.0.0/8;};
        };
};
```

On the other hand, for the public view, we allow access (`match-clients` and `allow-query` command) to all IP addresses excepted ones which came from our local network 192.168.1.0/24 and from the localhost 127.0.0.0/8.

```
view public {match-clients{!192.168.1.0/24;any;};
        zone "sos4.cc.uniza.sk" {
                file "/etc/bind/primary/zone.public";
                type primary;
                allow-query {!192.168.1.0/24;any;};
        };
};
```

We create a folder _primary_ and files _zone.public_ and _zone.private_. In these zone files we define the zone names and domain names associated to the primary server.

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

### Configuring secondary server

We do the same things as in the primary server with the following differences :

-   We name the created folder _secondary_ instead of _primary_

-   In the _named.conf.local_ file replace 'primary' by 'secondary' and associate this secondary server to the primary one with its IP address in both public and inside views.

-   We need to copy zone files from the primary server to the secondary one in order to respect the structure of the primary-secondary server configuration.

In order to synchronize zone files we first should authorize the primary server to write in the secondary server's folders and files as following

`$ debian@sos4-server2:/etc/bind$ sudo chmod a+w secondary`

Then we have to fixe an error `apparmor denied` by enable apparmor with

`$ sudo systemctl disable apparmor`

Besides, in order to copy readable file with text format and not binary format we add the following line in the server2's _named.conf.local_ file `masterfile-format text`.

Eventually, because there are two different zone files we need to distinguish them, otherwise we will get the same content in both _zone.private_ and _zone.public_ files. To achieve this task we used _tsig-keygen_ and create keys corresponding to each private and public zone files.

First we create the 2 keys :

`>> sudo tsig-keygen -a hmac-sha512 private-key`

`>> sudo tsig-keygen -a hmac-sha512 public-key`

Then we add them to our files `named.conf.local` on both primary and secondary servers outside the views :

```
key "private-key" {
        algorithm hmac-sha512;
        secret "wjvkVCgX37qGeQk4to8qy12Eiztf+sSWCnXwuJ1R7CBuN5WktWjyeaV7pVUtknVq84O3ZdMskr0GR94ZEObMNw==";
};

key "public-key" {
        algorithm hmac-sha512;
        secret "RU3Wai0jX5J8RzlOLXjuW80m2yCY0JkGNhuAvUbqOqlmpG3mvVWnrgsYgkDXVMZbwkszKWHy+UAbkv+6oPabtQ==";
};

```

We add in the primary server's inside view (analogous thing in the public view) updates to allow access to IP addresses passing private key. We used ACL as interfaces to insert keys in the `match-client` command.

```
acl insideacl {
!key "public-key";key "private-key"; 127.0.0.0/8; 192.168.1.0/24;
};

...

match-clients{insideacl;}

...

allow-transfer {key "private-key";};
...
```

And we add in the secondary server's inside view (analogous thing in the public view):

```
type secondary;
primaries {192.168.1.9 key "private-key";};

```

---

## Testing

First and foremost it's important to restart bind in order to create zone files in the secondary server copied from the primary server's zone files.

`>> sudo systemctl restart bind9`

Then, we should check that our zone files has been created or edited there : `debian@sos4-server2:/etc/bind/secondary$`

Then, to test if our DNS configuration is working, we need to check if computers inside and outside our network can find our zone files and point the domain to the right address.

To verify if the configuration works properly, we make bind ask himself on the localhost IP 0 or 127.0.0.1. We should obtain the private IP address of each server associated to its right domain name :

**For primary**

```
>> host ns1.sos4.cc.uniza.sk 0

debian@sos4-server1:~$ host ns1.sos4.cc.uniza.sk 0
Using domain server:
Name: 0
Address: 0.0.0.0#53
Aliases:

ns1.sos4.cc.uniza.sk has address 192.168.1.9
```

**And for secondary**

```
>> host ns2.sos4.cc.uniza.sk


debian@sos4-server2:~$ host ns2.sos4.cc.uniza.sk 0
Using domain server:
Name: 0
Address: 0.0.0.0#53
Aliases:

ns2.sos4.cc.uniza.sk has address 192.168.1.10
debian@sos4-server2:~$
```

The DNS is properly configured
