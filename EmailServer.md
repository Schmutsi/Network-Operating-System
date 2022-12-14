# Email Server

## Introduction

---

## SMTP : Postix

---

## IMAP : Dovecot

---

## Visualize emails : Rainloop

```
sudo wget http://www.rainloop.net/repository/webmail/rainloop-latest.zip


sudo iptables -A INPUT -p tcp --dport 25 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

​sudo​ iptables -A OUTPUT -p tcp --sport ​25​ -m conntrack --ctstate ESTABLISHED -j ACCEPT

```
