###### The Root Directory
This is typically located at /root, and resides one directory deep within the root directory /. 
The /root path is treated as any typical user’s home directory, and does not serve a similar purpose to that of the root directory /.

###### 

``````sh
/ – The Root Directory :
Everything on your Linux system is located under the / directory, similar to the C:\ directory on Windows.

/root – Root Home Directory:
The /root directory is the home directory of the root user. Instead of being located at /home/root, it's located at /root. This is distinct from /, which is the system root directory.

/bin – Essential User Binaries:
The /bin directory contains the essential user binaries (programs) that must be present when the system is mounted in single-user mode.
Applications such as Firefox, if they aren't installed as Snaps, are stored in /usr/bin, while important system programs and utilities such as the bash shell are located in /bin.
The /usr directory may be stored on another partition. Placing these files in the /bin directory ensures the system will have these important utilities even if no other file systems are mounted. The /sbin directory is similar: it contains essential system administration binaries.

/boot – Static Boot Files :
The /boot directory contains the files needed to boot the system.

/dev – Device Files:
the /dev directory contains a number of special files that represent devices.
/dev/sda represents the first SATA drive in the system. If you wanted to partition it, you could start a partition editor and tell it to edit /dev/sda.

/etc – Configuration Files:
The /etc directory contains configuration files, which can generally be edited by hand in a text editor. 
/etc/hosts – Information of IP and corresponding hostnames.
/etc/resolv.conf – DNS being used by System.
/etc/passwd – It contains username, password of the system, users in a shadow file.
/etc/bashrc – It is used by bash shell that contains system defaults and aliases.

/home – Home Folders  or ~ :
home directory contains a home folder for each user. For example, if your user name is bob, you have a home folder located at /home/bob.
Each user only has write access to their own home folder and must obtain elevated permissions (become the root user) to modify other files on the system.

/lib – Essential Shared Libraries:
The /lib directory contains libraries needed by the essential binaries in the /bin and /sbin folder. Libraries needed by the binaries in the /usr/bin folder are located in /usr/lib.

/mnt – Temporary Mount Points:
/mnt directory is where system administrators mounted temporary file systems while using them.
if you're mounting a Windows partition to perform some file recovery operations, you might mount it at /mnt/windows

/opt – Optional Packages:
The /opt directory contains subdirectories for optional software packages. It's commonly used by proprietary software that doesn't obey the standard file system hierarchy. or example, a proprietary program might dump its files in /opt/application when you install it.

/proc – Kernel and Process Files:
The /proc directory similar to the /dev directory because it doesn't contain standard files
/proc/cpuinfo – CPU Information
/proc/filesystems – It keeps the useful info about the processes that are running currently.
/proc/mount – Mounted File-system Information.

/run – Application State Files:
The /run directory gives applications a standard place to store transient files they require like sockets and process IDs.

/sbin – System Administration Binaries:
The /sbin directory is similar to the /bin directory. It contains essential binaries that are generally intended to be run by the root user for system administration.

/tmp – Temporary Files:
Applications store temporary files in the /tmp directory. These files are generally deleted whenever your system is restarted and may be deleted.

/usr – User Binaries & Read-Only Data:
The /usr directory contains applications and files used by users, as opposed to applications and files used by the system.
The /usr/local directory is where locally compiled applications install to by default. The /usr directory also contains other directories.
non-essential applications are located inside the /usr/bin directory instead of the /bin directory.

/var – Variable Data Files:
The /var directory is the writable counterpart to the /usr directory, which must be read-only in normal operation.
Log files and everything else that would normally be written to /usr during normal operation are written to the /var directory.


``````

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

``````



