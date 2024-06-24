##### 
Linux chown command is used to change a file's ownership, directory, or symbolic link for a user or group.
The chown stands for change owner. In Linux, each file is associated with a corresponding owner or group.

chown [OPTION]... [OWNER][:[GROUP]] FILE...  

-c, --changes: It is used to display the detailed output like verbose, but it is reported when only a change is made
-v, --verbose: It is used to display a diagnostic for every processed file.
-R, --recursive: It is used to perform operations on files and directories recursively.

##### Display the UID, GID, and Groups
The numeric user ID (UID) can be used instead of the username.
To list the UID and GID, execute the id command as follows:
``````sh
chown [OPTIONS] USER[:GROUP] FILE(s) or Directories

chown user:DevOps deploy.sh
chown linuxize file1 dir1
chown 1000 file2


``````
#### Manage File Ownership Using the chown Command
For groups, start their names with a colon (:) to allow the command to distinguish it from a user account
``````sh
chown :GROUP FILE
chown :www-data file1

``````
##### How to Change Symbolic Links Ownership
if you try to change the owner and the group of the symbolic link symlink1 that points to /var/www/file1

``````sh
chown www-data: symlink1
chown -h www-data symlink1

``````
##### How to Recursively Change the File Ownership
To recursively operate on all files and directories under the given directory, use the -R (--recursive) option.
If the directory contains symbolic links, pass the -h option

``````sh
# change the ownership of all files and subdirectories under the /var/www directory to a new owner and group named www-data
chown -R USER:GROUP DIRECTORY

chown -R www-data: /var/www
chown -hR www-data: /var/www


``````
