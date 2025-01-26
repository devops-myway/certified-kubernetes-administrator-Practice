##### How to check disk space in Linux with df and du
- Keeping track of disk space usage is crucial for effective resource management. Two of the most commonly used tools for this task are df and du
- explore how to use these commands, with a special focus on their "human-readable" (-h) versions

##### The df command
- The df (disk free) command is used to get a report of the total, used and available space on mounted file systems.
- To make the output more readable, you can use the -h (human-readable) option, which displays sizes in more understandable units (KB, MB, GB).
- Explanation of Columns using df -h
- 
- Filesystem: The name of the storage device.
- Size: Total capacity of the filesystem.
- Used: Used space.
- Avail: Available space.
- Use%: Percentage of space used.
- Mounted on: Mount point on the file system.
``````sh
df -h

Typical Exit:

Filesystem	          Size	        Used	    Avail	    Use%	      Mounted on
udev	                7.8G	        0	        7.8G	     0%	           /dev
tmpfs	                881M	        788K	    880M	      1%	        /run
/dev/xvda3	          28G	          19G	      7.2G	      73%	        /
tmpfs	                4.3G	          0	      4.3G	      0%	      /dev/shm
tmpfs	                5.0M	        0	        5.0M	      0%	      /run/lock
tmpfs	                4.0G	        9.6M	    4.0G	      1%	      /tmp
/dev/xvda1	          447M	        57M	      362M	      14%	        /boot
tmpfs	                881M	        0	        881M	      0%	        /run/user/0

``````
##### The du command
- the du (disk usage) command is used to estimate the disk space usage of the specified file or directory.
- In df, look for mount points near 100% usage, as this indicates that they are nearly full.
- In du, identify directories that take up an unusually large amount of space, which might suggest a need for cleanup or reorganisation.
- Size: Displayed at the beginning of each line, representing the space used by the directory or file.
- Path: The specific location of the directory or file.
``````sh
du -h /path/to/directory
du -h | sort -h

Size	          Directory
1.5M	          /home/user/Documents
500K	          /home/user/Downloads

``````
