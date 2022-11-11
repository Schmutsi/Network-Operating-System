# DHCP

## Introduction

The Dynamic Host Configuration Protocol, is another way to assign an ip address. Instead of giving it a name exactly and being forced to have planned every computer using the network. 
We can have the network itself assign a valid ip address to a user.

We will use the dhcp for the client server.
However, instead of creating the dhcp server in the router, we will create it in server 1 to allow a better control. (ne me demande pas ce que j'ai voulu dire par la jsp, j'essaie juste de donner une raison random pour pourquoi on fait ça)

## configuration

First we install isc-dhcp-server

in the files created, we search for /etc/dhcp/dhcpd.conf and enter our basic needs

`code to write here`

Then we get /etc/default/isc-dhcp-server
and precise wich interfaces to listend too.

``enter code her`

now that the dhcp is configured on server1

We need to instal a relay on the router.

we install isc-dhcp-relay
get to get /etc/default/isc-dhcp-relay
and provide wich interfaces to listend too, and the dhcp server's private ip address.

Finally, we just have to modify the 'static' in server client's /etc/interface.d/50-cloud-init with dhcp to make it all work
