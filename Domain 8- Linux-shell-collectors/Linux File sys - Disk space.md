

##### Linux File System
Linux file system is the collection of data and/or files stored in a computer’s hard disk or storage, your computer relies on this file system to ascertain the location and positioning of files in your storage, were it not there, the files would act as if they are invisible, obviously causing many problems.

There are actually many different file systems that exist for Linux:

Ext
Ext2
Ext3
Ext4 - ext4, standing for “fourth extended system”, was created in 2006. Because this file system overcomes numerous limitations that the third extended system had, it is both widely used, and the default file system that most Linux distros use.

###### What file system does my system use
If you’re wondering which file system your distro has by default, use Short for “disk free”, df is a command used to display the free disk space in Linux and other similar operating systems. It is also used to understand and ascertain the file systems that are mounted.

By default, df displays values in 1-kilobyte blocks. but when you run the “df” command yourself, there is no mention of any types of file systems.
``````sh
df

df -T    # it will display the file system type
``````
Display Usage in Megabytes and Gigabytes
``````sh
df -h
``````
##### Display a Specific File System
``````sh
df -h /dev/sda2

df -h /
``````
##### The du command displays disk usage.
``````sh
du
du -h
du -hs /etc/kernel-img.conf
``````

##### Free command - RAM Usage
The Linux free command outputs a summary of RAM usage, including total, used, free, shared, and available memory and swap space.
``````sh
free [options]

free -h
free -h --total

``````
For powers of 1024, use:

-b, --bytes
-k, --kibi
-m, --mebi
-g, --gibi
--tebi
--pebi
``````sh
free -m
``````