
https://www.digitalocean.com/community/conceptual-articles/introduction-to-web-servers

##### Nginx
Nginx is one of the most popular web servers in the world and is responsible for hosting some of the largest and highest-traffic sites on the internet.

When using the Nginx web server, server blocks (similar to virtual hosts in Apache) can be used to encapsulate configuration details and host more than one domain from a single server. We will set up a domain called your_domain, but you should replace this with your own domain name.

/var/www/html: The actual web content, which by default only consists of the default Nginx page you saw earlier, is served out of the /var/www/html directory. This can be changed by altering Nginx configuration files



##### Installing Nginx

``````sh
sudo apt update
sudo apt install nginx
systemctl status nginx

http://your_server_ip   # test the nginx server

``````
##### Managing the Nginx Process

``````sh
sudo systemctl stop nginx
sudo systemctl start nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl disable nginx
sudo systemctl enable nginx

``````
###### Setting Up Server Blocks (Recommended)

``````sh
sudo mkdir -p /var/www/your_domain/html
sudo chmod -R 755 /var/www/your_domain

sudo vi /var/www/your_domain/html/index.html
---
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>
``````

##### Serving Content
Nginx to serve this content, it’s necessary to create a server block with the correct directives.

``````sh
sudo vi /etc/nginx/sites-available/your_domain
--
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
--
``````
let’s enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:


``````sh
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
---
sudo vi /etc/nginx/nginx.conf
sudo nginx -t
sudo systemctl restart nginx

``````

``````sh

``````

``````sh

``````

##### Content
/var/www/html: The actual web content, which by default only consists of the default Nginx page you saw earlier, is served out of the /var/www/html directory. This can be changed by altering Nginx configuration files.

##### Server Configuration
/etc/nginx: The Nginx configuration directory. All of the Nginx configuration files reside here.

/etc/nginx/nginx.conf: The main Nginx configuration file. This can be modified to make changes to the Nginx global configuration.

/etc/nginx/sites-available/: The directory where per-site server blocks can be stored. Nginx will not use the configuration files found in this directory unless they are linked to the sites-enabled directory. Typically, all server block configuration is done in this directory, and then enabled by linking to the other directory.

/etc/nginx/sites-enabled/: The directory where enabled per-site server blocks are stored. Typically, these are created by linking to configuration files found in the sites-available directory.

/etc/nginx/snippets: This directory contains configuration fragments that can be included elsewhere in the Nginx configuration. Potentially repeatable configuration segments are good candidates for refactoring into snippets.

##### Server Logs
/var/log/nginx/access.log: Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.
/var/log/nginx/error.log: Any Nginx errors will be recorded in this log