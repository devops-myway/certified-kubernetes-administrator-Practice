
##### Note: Host Files /etc/hosts
The hosts file is one of the existing facilities for addressing networks within a computer.
To view your hosts file run in terminal cat /etc/hosts
Initially, it contains a single mapping: IP 127.0.0.1. This is your machine's local address, also called the loopback.

IP Address       Hostname   [URL]         Alias
127.0.0.1       localhost
12.345.678.90    www.example.com

You can test the operation through a web browser or through the utility curl, try it in terminal:
curl localhost
curl 127.0.0.1
private_ip      name_to_identify_hosts_hostname


hostnamectl - used to query and change the system hostname via the hosts file and related settings.
The static hostname is stored in /etc/hostname

sudo su nano /etc/hostname or vim /etc/hosts 
update the host server names with entries on the dir /etc/hosts

private_ip          name_to_identify_host1
private_ip          name_to_identify_host2
private_ip          name_to_identify_host3

Exit and reconnect via ssh
----
 hostname
 hostnamectl --help
 hostname --version

``````sh
#View All the Host Names
hostnamectl status

#Set a Particular Host Name
hostnamectl     set-hostname    new_host_name

e.g: hostnamectl        set-hostname       server01

``````
using vim or nano /etc/hosts. To update the information for internal networking, change the host that is associated with the main IP address for your server

``````sh
$  vim /etc/hosts   
127.0.0.1      localhost localhost.localdomain
123.45.67.89   hostname.domain.com   hostname
