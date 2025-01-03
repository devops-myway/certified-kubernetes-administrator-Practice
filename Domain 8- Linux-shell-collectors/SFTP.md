##### Transferring data with SFTP
- SFTP, which stands for Secure File Transfer Protocol: is a separate protocol packaged built into SSH that can implement FTP commands over a secure connection
- SFTP uses an entirely different protocol based on SSH (secure shell) and uses strong encryption for authentication information as well as the data transferred.
- FTP is an insecure protocol that should only be used in limited cases or on networks you trust.
- SFTP is preferable to FTP because of its underlying security features and ability to piggy-back on an SSH connection
- By default, SFTP uses the SSH protocol to authenticate and establish a secure connection.
- Because of this, the same authentication methods are available that are present in SSH.

##### How to Connect with SFTP
- If you can connect to the machine using SSH, then you have completed all of the necessary requirements necessary to use SFTP to manage files.
``````sh
ssh sammy@your_server_ip_or_remote_hostname
ssh -i ~/.ssh/private_key user_name@your_server_ip_or_remote_hostname
exit

#Now we can establish an SFTP session by issuing the following command:
sftp sammy@your_server_ip_or_remote_hostname
#If you are working on a custom SSH port (not the default port 22), then you can open an SFTP
sftp -oPort=custom_port sammy@your_server_ip_or_remote_hostname


``````
##### Getting Help in SFTP

``````sh
help
?
pwd
ls
ls -la
cd testDirectory
lpwd

``````
##### Transferring Files with SFTP
- If we want to download files from our remote host, we can do so using the get command
``````sh
get remoteFile
get remoteFile localFile
get -r someDirectory
get -Pr someDirectory

``````
##### Transferring Local Files to the Remote System
- Transferring files to the remote system works the same way, but with a put command
``````sh
put localFile
put -r localDirectory
df -h
!
``````
##### Simple File Manipulations with SFTP
- SFTP allows you to perform some kinds of filesystem housekeeping. For instance, you can change the owner of a file on the remote system with:
``````sh
chown userID file
get /etc/passwd
!less passwd
chgrp groupID file
get /etc/group
!less group

chmod 777 publicFile
lumask 022
ln
rm
rmdir

``````
##### Setting up a SFTP server
- You can run an SFTP server in a Windows environment, e.g. using the open source software FileZilla Server.
- A Linux server can be set-up after installing the required libraries (libssh2, OpenSSH).
``````sh
#Create a dedicated group for all future SFTP users
groupadd sftpusers
#First create a folder on a volume with sufficient free space:
mkdir -p /data/sftp
#Set permissions:
chown root:sftpusers /data/sftp
chmod 775 /data/sftp
#Create one or more SFTP users, assigning them to the previously created group:
useradd -g sftpusers -d / -s /sbin/nologin USERNAME
#Set the password for the new user
passwd USERNAME
#Edit the SSHD configuration at /etc/ssh/sshd_config
Match Group sftpusers
ChrootDirectory /data/sftp
ForceCommand internal-sftp
#Restart the SSH services
service sshd restart
``````
