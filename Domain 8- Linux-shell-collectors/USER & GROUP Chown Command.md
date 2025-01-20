https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command/

##### File Permissions Understanding
- By default, Linux uses the root user (UID 0) to run processes. This means that all files and directories created within the filssystem are owned by the root user.
- By default, the root user has full access (read, write, and execute) to all files and directories within the filesystem.
- it's generally considered a best practice to avoid running applications as the root user inside a filesystem.
- Instead, you should create a non-root user and run your application with that user's permissions.
- Linux chown command is used to change a file's ownership, directory, or symbolic link for a user or group.
- The chown stands for change owner. In Linux, each file is associated with a corresponding owner or group.

chown [OPTION]... [OWNER][:[GROUP]] FILE...  

- -c, --changes: It is used to display the detailed output like verbose, but it is reported when only a change is made
- -v, --verbose: It is used to display a diagnostic for every processed file.
- -R, --recursive: It is used to perform operations on files and directories recursively.

The command adds an entry to the /etc/passwd, /etc/shadow, /etc/group, and /etc/gshadow files.

##### Create User (useradd/adduser) while creating Users
- -u uid: Set the user’s ID to be uid. Unless you know what you’re doing, omit this option and accept the default
- -p <password>: --password <password>	: Sets the user's password (not recommended).
- -g group: --gid <name or number>	: Set the user’s initial (default) group to group
- -G group1,group2,: Make the user a member of the additional, existing groups group1, group2, and so on.
- -d dir: Set the user’s home directory to be dir,
- -m: Copy all files from your system skeleton directory, /etc/skel, into the newly created home directory
- -k: to get new users started If you prefer to copy from a different directory (-k your_preferred_directory).
- -M: create a user without a home directory
- -e: To set an expiry date for a user account
- -c: To add a comment or description for a user
- -s shell: Set the user’s login shell to be shell
- -p: To set an unencrypted password for the user

``````sh
useradd <options> <username>
sudo useradd test_account

sudo useradd -d /home/test_user test_user
sudo useradd -M test_user
sudo useradd -e 2020-05-30 test_user
sudo useradd -c "This is a test user" test_user
sudo useradd -s /bin/sh test_user

``````
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
