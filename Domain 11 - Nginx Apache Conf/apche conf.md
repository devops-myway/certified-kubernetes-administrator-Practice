
https://httpd.apache.org/docs/2.4/vhosts/examples.html
https://www.digitalocean.com/community/conceptual-articles/introduction-to-web-servers

##### Apache Notes
The Apache HTTP server is the most widely-used web server in the world. It provides many powerful features including dynamically loadable modules, robust media support, and extensive integration with other popular software.

##### Installing Apache

``````sh
sudo apt update
sudo apt install apache2

``````
##### checking the status of the webserver
``````sh
sudo systemctl enable apache2
sudo systemctl status apache2
sudo systemctl stop apache2
sudo systemctl start apache2
sudo systemctl restart apache2

http://your_server_ip # enter your servcer ip address

``````
###### Setting Up Virtual Hosts (Recommended)
Create the directory for your_domain
create a sample index.html page

``````sh
sudo mkdir /var/www/your_domain
sudo chmod -R 755 /var/www/your_domain
sudo vi /var/www/your_domain/index.html
---- 
<html>
    <head>
        <title>Welcome to Your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain virtual host is working!</h1>
    </body>
</html>


``````
##### Serving Content of index.html

``````sh
sudo vi /etc/apache2/sites-available/your_domain.conf

``````
Paste in the below configuration block, which is similar to the default, but updated for our new directory and domain name:

DocumentRoot: to our new directory
ServerAdmin: to an email that the your_domain site administrator can access.
ServerName: which establishes the base domain that should match for this virtual host definition
ServerAlias: which defines further names that should match as if they were the base name.

``````sh
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName your_domain
    ServerAlias www.your_domain
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

``````
1- Let’s enable the file with the a2ensite tool:
2- Disable the default site with a2dissite defined in 000-default.conf
3- Test for configuration errors:

``````sh
sudo a2ensite your_domain.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest

sudo systemctl restart apache2

You can test this by navigating to http://your_domain
``````

##### Example of multi-virtual host
The server has two IP addresses. On one (172.20.30.40), we will serve the "main" server, server.example.com and on the other (172.20.30.50), we will serve two or more virtual hosts

``````sh
Listen 80

# This is the "main" server running on 172.20.30.40
ServerName server.example.com
DocumentRoot "/www/mainserver"

<VirtualHost 172.20.30.50>
    DocumentRoot "/www/example1"
    ServerName www.example.com

    # Other directives here ...
</VirtualHost>

<VirtualHost 172.20.30.50>
    DocumentRoot "/www/example2"
    ServerName www.example.org

    # Other directives here ...
</VirtualHost>


``````
##### Running different sites on different ports.
``````sh
Listen 80
Listen 8080

<VirtualHost 172.20.30.40:80>
    ServerName www.example.com
    DocumentRoot "/www/domain-80"
</VirtualHost>

<VirtualHost 172.20.30.40:8080>
    ServerName www.example.com
    DocumentRoot "/www/domain-8080"
</VirtualHost>

<VirtualHost 172.20.30.40:80>
    ServerName www.example.org
    DocumentRoot "/www/otherdomain-80"
</VirtualHost>

<VirtualHost 172.20.30.40:8080>
    ServerName www.example.org
    DocumentRoot "/www/otherdomain-8080"
</VirtualHost>
``````
``````sh

``````

##### Content

/var/www/html: The actual web content, which by default only consists of the default Apache page you saw earlier, is served out of the /var/www/html directory. This can be changed by altering Apache configuration files.

##### Server Configuration

/etc/apache2: The Apache configuration directory. All of the Apache configuration files reside here.

/etc/apache2/apache2.conf: The main Apache configuration file. This can be modified to make changes to the Apache global configuration. This file is responsible for loading many of the other files in the configuration directory.

/etc/apache2/ports.conf: This file specifies the ports that Apache will listen on. By default, Apache listens on port 80 and additionally listens on port 443 when a module providing SSL capabilities is enabled.

/etc/apache2/sites-available/: The directory where per-site virtual hosts can be stored. Apache will not use the configuration files found in this directory unless they are linked to the sites-enabled directory. Typically, all server block configuration is done in this directory and then enabled by linking to the other directory with the a2ensite command.

/etc/apache2/sites-enabled/: The directory where enabled per-site virtual hosts are stored. Typically, these are created by linking to configuration files found in the sites-available directory with the a2ensite. Apache reads the configuration files and links found in this directory when it starts or reloads to compile a complete configuration.

/etc/apache2/conf-available/, /etc/apache2/conf-enabled/: These directories have the same relationship as the sites-available and sites-enabled directories but are used to store configuration fragments that do not belong in a virtual host. Files in the conf-available directory can be enabled with the a2enconf command and disabled with the a2disconf command.

/etc/apache2/mods-available/, /etc/apache2/mods-enabled/: These directories contain the available and enabled modules, respectively. Files ending in .load contain fragments to load specific modules, while files ending in .conf contain the configuration for those modules. Modules can be enabled and disabled using the a2enmod and a2dismod commands.

##### Server Logs
/var/log/apache2/access.log: By default, every request to your web server is recorded in this log file unless Apache is configured to do otherwise.

/var/log/apache2/error.log: By default, all errors are recorded in this file. The LogLevel directive in the Apache configuration specifies how much detail the error logs will contain.
