# NTP

## Introduction

Having a different time between different devices can lead to major errors with the synchronization between the different devices.
Thus, we can synchronize the time of our different devices to make sure they all follow the same time.

To do that, we first need to connect to a ntp server and synchronize with it as a client.
Our router will synchronize with an external ntp server.
Then our other servers will follow the router and synchronize with it.

---

## Configuring

First, we install ntp

```
sudo apt install ntp
```

Then we choose to use the ntp server from <b>Write name here</b>

in the file _/etc/ntp.conf_: we introduce the ntp server we will be a client of.

```
# pool: <http://www.pool.ntp.org/join.html>
server 0.sk.pool.ntp.org
server 1.sk.pool.ntp.org
server 2.sk.pool.ntp.org
server 3.sk.pool.ntp.org
```

Now we restart the ntp daemon and check its status.

```
sudo service ntp restart
sudo service ntp status
```

Now that we’ve been synchronized with the external ntp, we can make our router a new ntp server to synchronize the servers with it.

First, we will need to add a rule to the firewall to allow ntp clients to connect.

```
enter iptables rule here
```

Then we make our router a ntp server.

To do this on each server (server 1, sever 2, and server client) à
we install ntpdate.

```
sudo apt install ntpdate
```

And we ask it to link with the router.

```
sudo ntpdate 192.168.1.1
```

Or

```
sudo ntpdate 192.168.2.1
```

For the client
We just need to add the router as a ntp host for all of them.
In _/etc/hosts_, we add this line

```
192.168.1.1 ntp-host
```

Or

```
192.168.2.1 ntp-host
```

For the client

---

## Testing

To check if it works as intended, we can look at the synchronization: `ntpq -p`

Ntp only uses the UTC time-zone. Thus, on each server (and the router) we will have to precise which time-zone we are in.
To do this we have to use this command on each server:

```
sudo timedatectl set-timezone Europe/Bratislava
```
