# DNS

## Introduction

### DNS

Explanation what is a DNS and what it means

Our associated domaine name is sos4.cc.uniza.sk

### Primary and secondary architecture

Explanation how it works


## Configuration

### Bind Installation

``sudo apt-get install bind9``

We also need to install utils of the package

``sudo apt-get install bind9-utils``

Then all the modifications will occure in the bind folder, to access it from the root by the command 

``cd /etc/bind``

### Primary

file named.conf.options put the fowarders 1.1.1.1 and 8.8.8.8

file named.conf.default-zones include all the code with the default view as following 

```
view default { ... };
```

file  named.conf.local write inside view and public view respectively with private and public IP

```
view inside {
...
};
```

```
view public {
...
};
```

create folder primary and files zone.public and zone.private

```
@ IN SOA ns1.cc.sk. (...)
@ IN NS ns1.cc.sk.
@ IN NS ns2.cc.sk.

ns1 A IP adress
ns2 A IP adress
```

### Secondary

Do the same as the primary server configureation with the following differences :

- folder secondary
- in the file named.conf.local remplacer par secondary et ajouter la ligne primaries { private IP serveur 1 primary ;};
- c'est tout ?

## Testing

Bind might ask himself on the localhost IP '0' or '127.0.0.1'

For primary

``host ns1.sos4.cc.uniza.sk 0``

And for secondary

``host ns2.sos4.cc.uniza.sk 0``

When it's working we can see the IP adress of the server tested (server 1 or server 2) as below :

...


