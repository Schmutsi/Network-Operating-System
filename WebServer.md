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

---

## Static web creation

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

We also uncomment the following line to avoid possible hash bucket memory problem in the `nginx.conf` file.

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

### Testing

Eventually, if all these configurations were done properly we would be able to access to our static web site (html file host on our server) at the following adress : http://www.sos4.cc.uniza.sk

_Test private and public ip adresses curl www.sos4.cc.uniza.sk 158.193.153.105 and 192.168.1.9_

---

## Dynamic web creation

### Install PHP and Maria DB

First we install MariaDB, PHP and other complementary modules.

```
$ sudo apt-get install nginx mariadb-server mariadb-client php-cgi php-common php-fpm php-mbstring php-zip php-net-socket php-gd php-mysql php-bcmath unzip wget git -y
```

### Configure a WordPress Database

Then, we create a database and user for WordPress. WordPress will use this database to store its information, and the user to have access to the database.

In the MariaDB prompt `mysql -u root -p` we enter the following lines :

```
$ MariaDB [(none)]> CREATE DATABASE wpsos4db;
$ MariaDB [(none)]> CREATE USER 'sos4'@'localhost' identified by 'sos';
$ MariaDB [(none)]> GRANT ALL PRIVILEGES ON wpsos4db.* TO 'sos4'@'localhost';
$ MariaDB [(none)]> FLUSH PRIVILEGES;
$ MariaDB [(none)]> EXIT;
```

### Install Wordpress

After the database configuration we install WordPress by downloading the latest release of WordPress.

```
$ cd /var/www/html/
$ wget https://wordpress.org/latest.tar.gz
$ tar -xvzf latest.tar.gz
```

We need to adapt the file `/../.../.././../wp-config.php` with the database and the user we have just created.

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */define( 'DB_NAME', 'wpsos4db' );

/** MySQL database username */define( 'DB_USER', 'sos4' );

/** MySQL database password */define( 'DB_PASSWORD', 'sos' );

/** MySQL hostname */define( 'DB_HOST', 'localhost' );
```

### Configure Nginx for WordPress

Next, we will need to create a Virtual Host nginx configuration file for WordPress. You can create a new Virtual Host configuration file here `/etc/nginx/sites-available/wordpress.conf` and fill it with the following lines.

```

...

TO COMPLETE

...

```

We enable this server block by creating a symbolic link as we done before.

Eventually, we connect this dynamic web site to our DNS and add a new route *http://wordpress.sos4.cc.uniza.sk* in the `/etc/bind/primary/zone.private` file as bellow.

```
wordpress.sos4.cc.uniza.sk
```

### Access the WordPress Site

When we browse the *http://wordpress.sos4.cc.uniza.sk* website for the first time we are redirected to the wordpress installation page.

After that we can easily connect to the admin panel of wordpress and customize our website.

As a good french group we decided to create a very brief website on french specialities (our french food miss us...).

_end attaching the photo_

Because WordPress is quiet RAM-greddy, we start our website only when we want with the following command line, otherwise we stop php and mysql modules.

```
$ sudo systemctl start php7.4-fpm mysqld
$ sudo systemctl stop php7.4-fpm mysqld
```

---

## HTTPS set-up (SSL certificate)

_Need to read everything and check the language_
