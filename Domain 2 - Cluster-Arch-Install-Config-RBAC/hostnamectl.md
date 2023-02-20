##### Note:
hostnamectl - used to query and change the system hostname and related settings.
The static hostname is stored in /etc/hostname

sudo su nano /etc/hostname
update the host server names with entries on the dir /etc/hosts

private_ip name_to_identify_host1
private_ip name_to_identify_host2
private_ip name_to_identify_host3

Exit and reconnect via ssh
----

 hostnamectl --help
 hostname --version

``````sh
#View All the Host Names
hostnamectl status

#Set a Particular Host Name
hostnamectl set-hostname name

e.g: hostnamectl set-hostname server01

# Changing Host Names Remotely or Example: To set server3 as host name on a remote server called as 172.102.2.24 we can use the following command:

hostnamectl set-hostname -H [username]@HostName

hostnamectl set-hostname server3 -H root@172.102.2.24

``````