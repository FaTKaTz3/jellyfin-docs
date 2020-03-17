---
uid: network-reverse-proxy-apache
title: Apache Reverse Proxy
---

## Apache HTTP Server Project

"The [Apache HTTP Server Project](https://httpd.apache.org/) is an effort to develop and maintain an open-source HTTP server for modern operating systems including UNIX and Windows. The goal of this project is to provide a secure, efficient and extensible server that provides HTTP services in sync with the current HTTP standards."

Before you begin, make sure you have a sub domain of jellyfin. Also, this config uses letsencrypt to handle the SSL Certificate.

Create jellyfin.example.com.conf in /etc/apache2/sites-available. Replace example.com with your domain. 

```
cd /etc/apache2/sites-available
sudo nano jellyfin.example.com.conf
```

Use the following Apache virtual host configuration to create your reverse proxy with SSL. Replace USER with your admin username. Replace example.com with your domain name. Replace SERVER_IP with your server's IP address.

```
<VirtualHost *:80>
	ServerAdmin USER@example.com
	ServerName example.com

	Redirect permanent / https://example.com

	ErrorLog /var/log/apache2/example.com-error.log
	CustomLog /var/log/apache2/example.com.log combined

</VirtualHost>

<IfModule mod_ssl.c>
<VirtualHost *:443>
	ServerAdmin USER@example.com
	ServerName example.com

	ProxyPreserveHost On

	ProxyPass "/embywebsocket" "ws://SERVER_IP:8096/embywebsocket"
	ProxyPassReverse "/embywebsocket" "ws://SERVER_IP:8096/embywebsocket"

	ProxyPass "/" "http://SERVER_IP:8096/"
	ProxyPassReverse "/" "http://SERVER_IP:8096/"

	
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
    Protocols h2 http/1.1

	ErrorLog /var/log/apache2/example.com-error.log
	CustomLog /var/log/apache2/example.com.log combined
</VirtualHost>
</IfModule>
```

Enable your new Website
```
sudo a2ensite jellyfin.example.com.conf
```

Restart Apache
```
sudo systemctl restart apache2
```
