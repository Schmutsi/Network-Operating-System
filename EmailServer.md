# Email Server

## Introduction

In order to manage an email server system on our Debian machine we are using SMTP and IMAP protocols to respectively send and receive emails.

The Simple Mail Transfer Protocol (SMTP) is an internet standard communication protocol for electronic mail transmission. Email server could use SMTP both for sending and receiving emails, but this protocol is more commonly used only for sending emails. Indeed, the Internet Message Access Protocol (IMAP) is the other internet standard protocol which is in charge of retrieving emails from mail server (it replaces the older POP3 protocol).

We also create a new route link to our domain name to have access to this email through a user-friendly interface to manage emails: http://mail.sos4.cc.uniza.sk

---

## SMTP: Postix

### Installing Postfix

Postfix is the module we use to deal with STMP install on our server 1.

A configuration through a special shell is done just after installing with the following line:

```
$ sudo apt install postfix
```

We get this kind of window to initialize Postfix for an internet site mail configuration.

![Alt text](/assets/postfixConfig.PNG 'Postfix config shell')

### Configuring Postfix

In the `/etc/postfix/main.cf` file we realize some modifications according to our domain name defined on mail.sos4.cc.uniza.sk and we restrict the listening on our ipv4 interfaces addresses.

```
myhostname = mail.sos4.cc.uniza.sk
mydomain = sos4.cc.uniza.sk
. . .
mydestination = $myhostname, mail.sos4.cc.uniza.sk, sos4.cc.uniza.sk, localhost
. . .
inet_interfaces = 192.168.1.9, 127.0.0.1
inet_protocols = ipv4
```

### Testing email sending

With the following line we can send emails from our root user of the server to the mail address we want.

```
$ echo "If this mail body is received it means that our server is able to send emails" | mail -s "testing Postfix sending email subject line" schmutz@stud.uniza.sk
```

### Adjusting firewall rules

Because SMTP use the TCP (Transfer Communication Protocol) on port 25 we need to update the firewall configurations to make query exchange between our server and the net.

```
$ sudo iptables -A INPUT -p tcp --dport 25 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
$ sudo iptables -A OUTPUT -p tcp --sport 25 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

---

## IMAP: Dovecot

### Configuring Dovecot

We install dovecot and some complementary modules to deal with IMAP to manage the email sending part.

```
$ sudo apt install dovecot-imapd
$ sudo apt install dovecot-sieve dovecot-solr dovecot-antispam
```

Quickly, we edit main configuration file `/etc/dovecot/dovecot.conf`, to enable IMAP protocol.

```
# Enable installed protocols
protocols = imap
!include_try /usr/share/dovecot/protocols.d/*.protocol
```

Then, we edit the authenticate file `/etc/dovecot/conf.d/10-auth.conf`

```
disable_plaintext_auth = no
auth_mechanisms = plain login
```

And the `/etc/dovecot/10-master.conf` file as below

```
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }
```

---

_SSL should be done now_

---

### Adjusting firewall rules

As in the previous section on SMTP, IMAP uses TCP on port 143. Thus, we allow communication through this port in the iptables configurations.

```
$ sudo iptables -A INPUT -p tcp --dport 143 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
$ sudo iptables -A OUTPUT -p tcp --sport 143 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

## Visualizing emails: Rainloop

### Configuring a Rainloop database

As we did for WordPress, we create a database for Rainloop.

Thus, in the MariaDB shell accessed with `$ mysql`, we enter the following lines.

```
$ MariaDB [(none)]> CREATE DATABASE rainsos4db;
$ MariaDB [(none)]> GRANT ALL PRIVILEGES ON rainsos4db.* TO 'sos4'@'localhost';
$ MariaDB [(none)]> FLUSH PRIVILEGES;
$ MariaDB [(none)]> EXIT;
```

### Installing Rainloop

Then, download the latest version of Rainloop

```
sudo wget http://www.rainloop.net/repository/webmail/rainloop-latest.zip
```

### Configuring Nginx for Rainloop

Next, we will need to create a Virtual Host nginx configuration file for Rainloop on _mail.sos4.cc.uniza.sk_. We can create a new Virtual Host configuration file here _/etc/nginx/sites-available/rainloop.conf_ and fill it with the following lines.

```
server {
  listen 80;

  server_name mail.sos4.cc.uniza.sk;
  root /var/www;

  index index.php;

  . . .

}
```

We enable this server block by creating a symbolic link as we have done before.

Eventually, we connect this dynamic web site to our DNS and add a new route http://mail.sos4.cc.uniza.sk in the _/etc/bind/primary/zone.private_ file as bellow.

```
@ IN MX 10 mail

mail A 158.193.153.48
```

### Accessing Rainloop Dashboard

We access the RainLoop dashboard using the URL http://mail.sos4.cc.uniza.sk/?admin.

We Provide the default username `admin` and password `12345` to be logged in the admin rainloop dahsbord.

We can easily change default username and password.

Then, in the domains section, we add our domain name sos4.cc.uniza.sk link to our server 1 for IMAP and SMTP. It's important to uncheck the case "User Authentication" related to IMAP section, in order to prevent "Authentication failed" error when we would like to send emails next.

### Testing user Rainloop interface

This time we can log in with our user sos4 created before on http://mail.sos4.cc.uniza.sk.

As we can see on the Rainloop interface screenshot, we are now able to send and receive emails with the sos4@sos4.cc.uniza.sk email address.

![Postfix config shell image](/assets/rainloopClient.PNG 'Postfix config shell')
