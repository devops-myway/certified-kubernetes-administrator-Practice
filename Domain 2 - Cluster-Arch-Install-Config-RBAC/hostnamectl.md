
##### Note:
hostnamectl - used to query and change the system hostname and related settings.
The static hostname is stored in /etc/hostname

sudo su nano /etc/hostname or vim /etc/hosts 
update the host server names with entries on the dir /etc/hosts

private_ip name_to_identify_host1
private_ip name_to_identify_host2
private_ip name_to_identify_host3

Exit and reconnect via ssh
----
 hostname
 hostnamectl --help
 hostname --version

``````sh
#View All the Host Names
hostnamectl status

#Set a Particular Host Name
hostnamectl set-hostname new_host_name

e.g: hostnamectl set-hostname server01

``````
using vim or nano /etc/hosts. To update the information for internal networking, change the host that is associated with the main IP address for your server

``````sh
$  vim /etc/hosts    
127.0.0.1      localhost localhost.localdomain
123.45.67.89   hostname.domain.com   hostname

--- # Change the domain name (where required)
$  vim /etc/resolv.conf
domain abc.com            <--- This would be the domain.
nameserver 173.203.4.8
nameserver 173.203.4.9

``````