
##### Check Linux Disk Space Using df Command
Managing disk space on a Linux server is an important task. For example, package manager applications notify you how much disk space will be required for an installation.

df command stands for disk free. and it shows you the amount of space taken up by different drives. By default, df displays values in 1-kilobyte blocks
``````sh
df
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