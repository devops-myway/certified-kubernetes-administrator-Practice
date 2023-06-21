

##### LAMP Stack on Ubuntu

A “LAMP” stack is a group of open source software that is typically installed together in order to enable a server to host dynamic websites and web apps written in PHP.

 This term is an acronym which represents the Linux operating system with the Apache web server. The site data is stored in a MySQL database, and dynamic content is processed by PHP.

#### Installing Apache Server

``````sh
sudo apt update
sudo apt install apache2

http://your_server_ip

``````

##### Installing Mysql
``````sh
sudo apt install mysql-server
----
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
exit
sudo mysql_secure_installation
----
sudo mysql
exit

``````
##### Installing Php

``````sh
sudo apt install php libapache2-mod-php php-mysql
php -v

``````

##### Creating a Virtual Host for your Website
``````sh
sudo mkdir /var/www/your_domain
sudo vi /etc/apache2/sites-available/your_domain.conf
----
<VirtualHost *:80>
    ServerName your_domain
    ServerAlias www.your_domain 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
---

``````
###### Enable Virtual Host

``````sh
sudo a2ensite your_domain
sudo a2dissite 000-default
sudo apache2ctl configtest
sudo systemctl reload apache2


``````
##### Inde.html file
Your new website is now active, but the web root /var/www/your_domain is still empty. Create an index.html file in that location to test that the virtual host works as expected:

``````sh
vi /var/www/your_domain/index.html

---
<html>
  <head>
    <title>your_domain website</title>
  </head>
  <body>
    <h1>Hello World!</h1>

    <p>This is the landing page of <strong>your_domain</strong>.</p>
  </body>
</html>

``````
then go to your browser and access your server’s domain name or IP address:
``````sh
http://server_domain_or_IP

``````

###### A Note About DirectoryIndex on Apache

With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file. This is useful for setting up maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors

To change this behavior, you’ll need to edit the /etc/apache2/mods-enabled/dir.conf file and modify the order in which the index.php file is listed within the DirectoryIndex directive.

``````sh
sudo nano /etc/apache2/mods-enabled/dir.conf
----
<IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
----
sudo systemctl reload apache2

``````
##### Testing PHP Processing on your Web Server

``````sh
nano /var/www/your_domain/info.php
--
<?php
phpinfo();
---
http://server_domain_or_IP/info.php

sudo rm /var/www/your_domain/info.php

``````

##### Testing Database Connection from PHP (Optional)
``````sh
sudo mysql

CREATE DATABASE example_database;
CREATE USER 'example_user'@'%' IDENTIFIED BY 'password';
GRANT ALL ON example_database.* TO 'example_user'@'%';
exit

mysql -u example_user -p
SHOW DATABASES;

``````
``````sh
CREATE TABLE example_database.todo_list (
	item_id INT AUTO_INCREMENT,
	content VARCHAR(255),
	PRIMARY KEY(item_id)
);

INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
SELECT * FROM example_database.todo_list;
exit

``````
Now you can create the PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor:
``````sh
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}

``````
``````sh
http://your_domain_or_IP/todo_list.php
``````

