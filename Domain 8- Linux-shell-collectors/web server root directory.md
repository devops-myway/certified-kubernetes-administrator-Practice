##### web server root directory go in linux
- /var has been used for system data that changes all the time, for example cache files, logs, runtime data (lock files, for example), mail server storage, printer spooling, etc
- Distros use /var/www because it is for "transient and temporary files". The files installed there are just for checking if the server is working.
- There is an argument to be made for using /usr/local/<app> if the web site files are static, but the most appropriate place is in /srv/<app> or /srv/www/<app>
- Packages use /usr/share for static HTML content, or /var/lib for dynamic variable content
``````sh
# by default most of Linux Flavors use

# the default public web root for Apache
var/www/html    

# For Ubuntu and docker images:
/usr/share/nginx/html/

/usr/share/nginx/html

# You can simply map nginx's root folder to the location of your website:
vi /etc/nginx/sites-enabled/default

cat /etc/nginx/sites-enabled/default |grep "root"
on my machine it was :root /usr/share/nginx/www;
``````
``````sh


``````
