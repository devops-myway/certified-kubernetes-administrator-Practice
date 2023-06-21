
###### How To Configure Nginx as a Reverse Proxy
A reverse proxy is the recommended method to expose an application server to the internet. Whether you are running a Node.js application in production or a minimal built-in web server with Flask, these application servers will often bind to localhost with a TCP port.

This means by default, your application will only be accessible locally on the machine it resides on.

##### Installing Nginx

``````sh
sudo apt update
sudo apt install nginx

systemctl status nginx

``````

##### Configuring your Server Block
Create and open a new Nginx configuration file using nano or your preferred text editor:

Insert the following into your new file, making sure to replace your_domain and app_server_address. If you do not have an application server to test with, default to using http://127.0.0.1:8000

``````sh
sudo vi /etc/nginx/sites-available/your_domain
---
server {
    listen 80;
    listen [::]:80;

    server_name your_domain www.your_domain;
        
    location / {
        proxy_pass app_server_address;
        include proxy_params;
    }
}

``````
Nginx provides some recommended header forwarding settings you have included as proxy_params, and the details can be found in /etc/nginx/proxy_params:


``````sh
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

``````
Enable this configuration file by creating a link from it to the sites-enabled directory that Nginx reads at startup:

``````sh
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

``````
##### Testing your Reverse Proxy with Gunicorn (Optional)

``````sh
your_domain

sudo apt update
sudo apt install gunicorn

nano test.py
--
def app(environ, start_response):
    start_response("200 OK", [])
    return iter([b"Hello, World!"])
``````

``````sh
gunicorn --workers=2 test:app
your_domain

``````

``````sh


``````