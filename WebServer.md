# Web Server

## Introduction

The term web server can refer to hardware or software, or both of them working together.

-   On the hardware side, a web server is a computer that stores web server software and a website's component files.

-   On the software side, a web server includes several parts that control how web users access hosted files. At a minimum, this is an HTTP server. An HTTP server can be accessed through the domain names of the websites it stores, and it delivers the content of these hosted websites to the end user's device.

To publish a website, you need either a static or a dynamic web server. In our case, we will start by configuring a static web server and then by configuring a dynamic one.

A static web server sends its hosted files as-is to your browser (like an html file) while a dynamic web server sends its hosted files dynamically updated thanks to an application server and a database to your browser.

---

## Configuration

### Nginx installation

Nginx and Apach are the two more common web server and we choose to install Nginx to our server 1.

```
$ sudo apt-get install nginx
```

### NAT and firewall configuration

It's necessary to modify the firewall setting to allow outside access to default web ports. Thus we modify iptables configuration in our router.

First, in the FORWARD chain we remove the conditions of states ESTABLISHED and RELATED and accept all requests from the outside (from en3 to ens4).

```
$ sudo iptables -D FORWARD 1
$ sudo iptables -A FORWARD -i ens3 -o ens4 -j ACCEPT
```

Then, in the INPUT and OUTPUT chains we accept all request from port 80 (http requests) and 443 (https requests) with the state ESTABLISHED. We also allow requests with the state NEW in the INPUT chain.

```
$ sudo iptables -A INPUT -p tcp --dport 80, 443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
$ sudo iptables -A OUTPUT -p tcp --sport 80, 443 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

### Static web creation

_Now the web server should work properly by giving the default nginx landing page when we enter in our browser http://158.193.153.105 but it doesn't work_

In order to display our own html page host on our server, we create this file at this following path: `/var/www/sos4.cc.uniza.sk/html/index.html`

```
<html>
    <head>
        <title>Welcome to sos4 web site</title>
    </head>
    <body>
        <h1>Success! Your Nginx server is successfully configured for <em>sos4.cc.uniza.sk</em>. </h1>
<p>This is a sample page.</p>
    </body>
</html>
```

When we created the folder `/var/www/sos4.cc.uniza.sk/html` we assign ownership of the directory with the `$USER` environment variable, which should reference our current system user. Besides we give permission to our web root to the folder as below:

```
$ sudo chown -R $USER:$USER /var/www/sos4.cc.uniza.sk/html
$ sudo chmod -R 755 /var/www/sos4.cc.uniza.sk
```

In order Nginx to serve this content, we create a configuration file with correct directives that point to our custom web root there `/etc/nginx/sites-available/sos4.cc.uniza.sk`:

```
server {
        listen 80;

        root /var/www/sos4.cc.uniza.sk/html;
        index index.html index.htm index.nginx-debian.html;

        server_name sos4.cc.uniza.sk www.sos4.cc.uniza.sk;

        location / {
               try_files $uri $uri/ =404;
        }
}
```

Next, we enable this server block by creating a symbolic link to our custom configuration file inside the `sites-enabled` directory, which Nginx reads from during startup:

```
$ sudo ln -s /etc/nginx/sites-available/sos4.cc.uniza.sk /etc/nginx/sites-enabled/
```

We also uncomment the following line to avoid possible hash bucket memory problem in the nginx.conf file.

```
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
```

Before testing, these two lines are useful to respectively check our syntax and restart the web server to enable our changes.

```
$ sudo nginx -t
$ sudo systemctl restart nginx
```

Morover, in the file `/etc/resolv.conf` we add the public ip address to make the web server also works on this address.

```
nameserver 192.168.1.9
nameserver 158.193.153.105
```

_Eventually, to test our static web server configuration we can do some curl commands with server ip addresses and our domain name wherever on our architecture._

```
$ curl www.sos4.cc.uniza.sk
$ curl 158.193.153.105
$ curl 192.168.1.9

```

_It should also work on the browser_

If it worked properly we would obtain our html file hosted on our server.

### Dynamic web creation

### HTTPS set-up

---

## Testing
