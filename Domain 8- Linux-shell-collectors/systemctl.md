
##### systemctl
Linux provides fine-grained control over system services through systemd, using the systemctl command.
Services can be turned on, turned off, restarted, reloaded, or even enabled or disabled at boot
You can use the systemctl command to manage services and control when they start.

systemctl [command] [service_name]

##### Restart a service

After editing the /etc/ssh/sshd_config file, use the systemctl restart

``````sh
sudo systemctl restart sshd
sudo systemctl status sshd

``````
##### Stop and start a service

After editing the /etc/ssh/sshd_config file, use the systemctl restart

``````sh
sudo systemctl stop sshd
sudo systemctl start sshd
``````
##### How To Check If a Service is Running on Linux

``````sh
sudo systemctl status apache2

``````
##### How to Restart a Service
``````sh
sudo systemctl restart SERVICE_NAME
sudo systemctl restart apache2
``````
##### How to Reload a Service
``````sh
sudo systemctl reload SERVICE_NAME
sudo systemctl reload apache2
``````
##### How to Enable the Service at Boot
``````sh
sudo systemctl enable SERVICE_NAME
sudo systemctl enable apache2
``````

``````sh

``````
