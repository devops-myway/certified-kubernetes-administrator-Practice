
##### systemctl
Suppose you're making configuration changes to a Linux server.
You can use the systemctl command to manage services and control when they start.

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
##### Control whether the service starts with the system

You can use the enable and disable subcommands to manage those defaults

``````sh
sudo systemctl disable sshd
sudo systemctl enable sshd
``````