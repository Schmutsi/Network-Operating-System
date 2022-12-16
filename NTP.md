## Introduction

Having a different time between different devices can lead to major errors with the synchronisation between the different devices.
Thus we can synchronise the time of our different devices to make sure they all follow the same time.

To do that, we first need to connect to an ntp server and synchronise with it as a client.
Our router will will synchronise with an external ntp server.
Then our others servers will follow the router and synchronise with it.


## Configuration
First we install ntp
```
sudo apt install ntp

```
Then we choose to use the ntp server from <b>Write name here</b>

in the file /etc/ntp.conf : we introduce the ntp server we will be a client of.
```
# pool: <http://www.pool.ntp.org/join.html>
server 0.sk.pool.ntp.org
server 1.sk.pool.ntp.org
server 2.sk.pool.ntp.org
server 3.sk.pool.ntp.org
```
now we restart the ntp daemon and check it’s status
```
sudo service ntp restart
sudo service ntp status
```


Now that we’ve been synchronised with the external ntp, we can make our router a new ntp server to synchronise the servers with it.

First we will need to add a rule to the firewall to allow ntp clients to connect.
```
enter iptables rule here
```

Then we make our router a ntp server.

To do this on each server (server 1, sever 2, and server client)à
we install ntpdate
```
sudo apt install ntpdate
```
and ask it to link with the router

```
sudo ntpdate 192.168.1.1
```
or
```
sudo ntpdate 192.168.2.1
```
for the client
We just need to add the router as a ntp host for all of them.
In /etc/hosts, we add this line
```
192.168.1.1 ntp-host
```
or
```
192.168.2.1 ntp-host
```for the client


to check if it works as intended we can look at the synchronisation : `ntpq -p`


Ntp only uses the UTC time-zone. Thus on each server (and the router) we will have to precise which time-zone we are in.
To do this we have to use this command on each server:
```
sudo timedatectl set-timezone Europe/Bratislava
```
