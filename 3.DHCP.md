# DHCP

## Introduction

Sometimes, we don’t know exactly how many devices would be connected to a server. Thus, we can’t allocate them with previously planned ip addresses. In this scenario, it isn’t possible to establish static ip address like done in the previous network configuration.

Thus, we use a dhcp server, it will allocate ip addresses to devices trying to connect to it.
In our configuration, we will have the Client server configured with a dhcp address.
To do this, we will have our server 1 act as a dhcp server, it will be the one allocating the ip addresses. The router will need to be a dhcp relay in order to connect the server client and our dhcp server.

---

## Configuring

First, we install isc-dhcp-server

```
sudo apt-get install isc-dhcp-server
```

in the files created, we search for _/etc/dhcp/dhcpd.conf_ and enter our basic needs

```
option domain-name "sos4.cc.uniza.sk";
option domain-name-servers ns1.sos4.cc.uniza.sk, ns2.sos4.cc.uniza.sk;

default-lease-time 600;
max-lease-time 7200;
```

We establish the name of our domain and the different servers and decide of a lease time (how long do we reserve an ip address to a specific device).
We also need to specify the address of the router that will serve as a relay for the dhcp.
Then on the same file:

```
subnet 192.168.1.0 netmask 255.255.255.0 {
range 192.168.1.40 192.168.1.60;

option routers 192.168.1.1;

}

subnet 192.168.2.0 netmask 255.255.255.0 {
range 192.168.2.2 192.168.2.254;

option routers 192.168.2.1;
}
```

In here we create specify two ranges of ip addresses that will can be used by the external devices. We specify the subnet and netmask of the network, and then the range of ips.
The `option routers` line specify which gateway to rely on, the ip address of the interface on the router.

Finally, we get _/etc/default/isc-dhcp-server_
and precise which interfaces to listen too.

```
INTERFACESv4="ens3"
INTERFACESv6=""
```

the interface linked to the router is ens3 in server one.

Then we just have to restart the server to make it work.

```
sudo service isc-dhcp-server restart
```

Now that the dhcp is configured on server1

We need to install a relay on the router.

We install isc-dhcp-relay

```
sudo apt-get install isc-dhcp-relay
```

get to get /etc/default/isc-dhcp-relay
and provide which interfaces to listen to, and the dhcp server's private ip address.

```
# What servers should the DHCP relay forward requests to?
SERVERS="192.168.1.9"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="ens4 ens5"
```

Finally, we just have to modify the 'static' in server client's /etc/interface.d/50-cloud-init by 'dhcp' to make it all work.


Then we just have to restart the server to make it work.

```
sudo service isc-dhcp-relay restart
```
