###### The Root Directory
This is typically located at /root, and resides one directory deep within the root directory /. 
The /root path is treated as any typical user’s home directory, and does not serve a similar purpose to that of the root directory /.

``````sh
# – requires given linux commands to be executed with root privileges either directly as a root user or by use of sudo command
$ – requires given linux commands to be executed as a regular non-privileged user

cd / && pwd
cd /root #root user’s home directory, /root
# cd
OR
# cd ~

``````
###### Virtual and Pseudo Process Related Files:

``````sh
/proc/cpuinfo – CPU Information
/proc/filesystems – It keeps the useful info about the processes that are running currently.
/proc/mount – Mounted File-system Information.

``````

##### System Configuration Files:

``````sh
/etc/hosts – Information of IP and corresponding hostnames.
/etc/resolv.conf – DNS being used by System.
/etc/passwd – It contains username, password of the system, users in a shadow file.
/etc/bashrc – It is used by bash shell that contains system defaults and aliases.

``````


