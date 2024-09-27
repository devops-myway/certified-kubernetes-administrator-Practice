https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command/

##### 
Linux chown command is used to change a file's ownership, directory, or symbolic link for a user or group.
The chown stands for change owner. In Linux, each file is associated with a corresponding owner or group.

chown [OPTION]... [OWNER][:[GROUP]] FILE...  

- -c, --changes: It is used to display the detailed output like verbose, but it is reported when only a change is made
- -v, --verbose: It is used to display a diagnostic for every processed file.
- -R, --recursive: It is used to perform operations on files and directories recursively.

The command adds an entry to the /etc/passwd, /etc/shadow, /etc/group, and /etc/gshadow files.

##### Display the UID, GID, and Groups
The numeric user ID (UID) can be used instead of the username.
To list the UID and GID, execute the id command as follows:
``````sh
chown [OPTIONS] USER[:GROUP] FILE(s) or Directories

chown user:DevOps deploy.sh
chown linuxize file1 dir1
chown 1000 file2

sudo useradd jane
sudo id jane    #uid=1005(jane) gid=1005(jane) groups=1005(jane)

``````
#### How to Add a New User and Create Home Directory
``````sh
sudo useradd jane
sudo id jane    #uid=1005(jane) gid=1005(jane) groups=1005(jane)

``````
#### How to Add a New User and Create Home Directory
``````sh
sudo useradd jane
sudo id jane    #uid=1005(jane) gid=1005(jane) groups=1005(jane)

``````
#### Creating a User with a Specific User ID
``````sh
sudo useradd -m -d /opt/jane jane

``````
#### Creating a User with a Specific User ID
Invoke the useradd command with the -u (--uid) option to create a user with a specific UID.
For instance, to create a new user named jane with a UID of 1500, you would type:
You can verify the user’s UID, using the id command
``````sh
sudo useradd -u 1500 jane
id -u jane      #1500

``````
#### Creating a User with a Specific Group ID
The -g (--gid) option allows you to create a user with a specific initial login group. You can specify either the group name or the GID number. The group name or GID must already exist.
``````sh
sudo useradd -g users jane
id -gn jane

sudo useradd -g users -G wheel,docker john  #uid=1002(john) gid=100(users) groups=100(users),10(wheel),993(docker)

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
