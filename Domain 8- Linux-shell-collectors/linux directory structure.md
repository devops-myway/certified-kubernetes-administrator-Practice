

###### What is Top Level in Linux?
/ is the top level directory of a Linux system. The name “top level” means the “root”.

cd /



###### Linux/Unix file system hierarchy

/bin – binary or executable programs. contains core system administrators program e.g cat
/etc – system configuration files.
/home – home directory. It is the default current directory.
/opt – optional or third-party software.
/tmp – temporary space, typically cleared on reboot.
/usr – User related programs.
/var – log files.

###### Some other directories in the Linux system:

/mnt – It contains temporary mount directories for mounting the file system.
/lib – It contains kernel modules and a shared library.
/dev – It is the location of the device files such as dev/sda1, dev/sda2, etc.
/sys – It is a virtual filesystem for modern Linux distributions to store and allows modification of the devices connected to the system

##### Log Files:
/var/log/lastlog – It stores user last login info.
/var/log/messages – It has all the global system messages.
/var/log/wtmp – It keeps a history of login and logout information.

cd /var/log
ls

###### Version Information File:
/version – It displays the Linux version information.

###### Virtual and Pseudo Process Related Files:
/proc/cpuinfo – CPU Information
/proc/filesystems – It keeps the useful info about the processes that are running currently.
/proc/mount – Mounted File-system Information.

##### User Related Files:

/usr/bin – It contains most of the executable files.
/usr/lib – It contains object files and libraries.
/usr/sbin – It contains commands for Super User, for System Administration.

##### System Configuration Files:
/etc/hosts – Information of IP and corresponding hostnames.
/etc/resolv.conf – DNS being used by System.
/etc/passwd – It contains username, password of the system, users in a shadow file.
/etc/bashrc – It is used by bash shell that contains system defaults and aliases.

###### Device Files:
/dev/hda – Device file for the first IDE HDD.
/dev/hdc – A pseudo-device that output garbage output is redirected to /dev/null

