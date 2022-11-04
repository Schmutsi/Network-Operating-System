login as: debian
Authenticating with public key "imported-openssh-key"
Linux sos4-server-router 5.10.0-19-cloud-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/\*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Nov 3 11:30:21 2022 from 158.193.103.19
debian@sos4-server-router:~$ sudo iptables -L -v -n
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination
debian@sos4-server-router:~$ sudo iptables -t nat A POSTROUTING -o ens3 -j MASQ
Bad argument `A' Try `iptables -h' or 'iptables --help' for more information.
debian@sos4-server-router:~$ sudo iptables -t nat -A POSTROUTING -o ens3 -j MASQiptables v1.8.7 (nf_tables): Chain 'MASQ' does not exist
Try `iptables -h' or 'iptables --help' for more information. debian@sos4-server-router:~$ sudo iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE debian@sos4-server-router:~$ sudo iptables -A INPUT -i lo -j ACCEPT debian@sos4-server-router:~$ sudo iptables -A INPUT -i ens3 -p tcp --dport 22 debian@sos4-server-router:~$ sudo iptables -A FORWARD -i ens3 -o ens4 -m state --state RELATED,ESTABLISHED -j ACCEPTED iptables v1.8.7 (nf_tables): Chain 'ACCEPTED' does not exist Try `iptables -h' or 'iptables --help' for more information.
debian@sos4-server-router:~$ sudo iptables -A FORWARD -i ens3 -o ens4 -m state --state RELATED,ESTABLISHED -j ACCEPT
debian@sos4-server-router:~$ sudo iptables -A FORWARD -i ens4 -o ens3 -j ACCEPT debian@sos4-server-router:~$ sudo iptables -L -v -n
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination
0 0 ACCEPT all -- lo _ 0.0.0.0/0 0.0.0.0/0  
 414 31984 tcp -- ens3 _ 0.0.0.0/0 0.0.0.0/0 tcp dpt:22

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination
0 0 ACCEPT all -- ens3 ens4 0.0.0.0/0 0.0.0.0/0 state RELATED,ESTABLISHED
0 0 ACCEPT all -- ens4 ens3 0.0.0.0/0 0.0.0.0/0

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target prot opt in out source destination
debian@sos4-server-router:~$ iptables -t nat -A prerouting -p tcp --dport 2221 -j NAT --to-destination 192.168.1.9:22
-bash: iptables: command not found
debian@sos4-server-router:~$ sudo iptables -t nat -A prerouting -p tcp --dport 2221 -j NAT --to-destination 192.168.1.9:22
iptables v1.8.7 (nf_tables): unknown option "--to-destination"
Try `iptables -h' or 'iptables --help' for more information.
debian@sos4-server-router:~$ sudo iptables -t nat -A prerouting -p tcp --dport 2221 -j NAT -to-destination 192.168.1.9:22
iptables v1.8.7 (nf_tables): table 'o-destination' does not exist
Perhaps iptables or your kernel needs to be upgraded.
debian@sos4-server-router:~$ sudo iptables -t nat -A prerouting -p tcp --dport 2221 -j NAT -todestination 192.168.1.9:22
iptables v1.8.7 (nf_tables): table 'odestination' does not exist
Perhaps iptables or your kernel needs to be upgraded.
debian@sos4-server-router:~$ sudo iptables -t nat -A prerouting -p tcp --dport 2221 -j NAT -to-destination 192.168.1.9:22
iptables v1.8.7 (nf_tables): table 'o-destination' does not exist
Perhaps iptables or your kernel needs to be upgraded.
debian@sos4-server-router:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 2221 -j NAT -to-destination 192.168.1.9:22
iptables v1.8.7 (nf_tables): table 'o-destination' does not exist
Perhaps iptables or your kernel needs to be upgraded.
debian@sos4-server-router:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 2221 -j DNAT -to-destination 192.168.1.9:22
iptables v1.8.7 (nf_tables): table 'o-destination' does not exist
Perhaps iptables or your kernel needs to be upgraded.
debian@sos4-server-router:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 2221 -j DNAT --to-destination 192.168.1.9:22
debian@sos4-server-router:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.1.10:22
debian@sos4-server-router:~$ ^C
