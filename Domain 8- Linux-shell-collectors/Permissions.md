
##### Users and Groups in Linux
There are two types of users: system users and regular users:
- System users are responsible for running non-interactive and background processes on a system. For example: mail, daemon, syslog, and so on.
- Regular users are the users that actually log into the system and perform their designated tasks interactively.

permission	on a file	on a directory:
- r (read)	read file content (cat)	read directory content (ls)
- w (write)	change file content (vi)	create file in directory (touch)
- x (execute)	execute the file	enter the directory (cd)

there are Ten characters (-rw-rw-r--) before the user owner. We'll describe these ten characters here.

position	characters	ownership:
- 1	-	denotes file type
- 2-4	rw-	permission for user
- 5-7	rw-	permission for group
- 8-10	r--	permission for other

Octal Table:
binary	octal	permissions
- 000	0	---
- 001	1	--x
- 010	2	-w-
- 011	3	-wx
- 100 4	r--
- 101	5	r-x
- 110	6	rw-
- 111	7	rwx

Generally implemented options are:

- -R: It stands for recursive, i.e., add objects to subdirectories.
- -V: It stands for verbose, display objects modified (unmodified objects are not displayed).
- -c, --changes: It is similar to the verbose option, but the difference is that it is reported if a change has been made.

The permission statement is represented in indicators such as:
- u+x, u-x.
- Where 'u' stands for 'user,'
- '+' stands for add,
- '-' stands for remove,
- 'x' stands for executable (which).
- 0=>No permission, 1=>Excute, 2=>Write, 3=>Execute + Write
- 4=>Read, 5=>Read + Execute, 6=>Read +Write, 7=>Read + Write +Execute

``````sh
# check details of users on a system by looking into the /etc/passwd file

ls -lh
chmod <options> <permissions> <file name>

cat /etc/passwd

``````
##### Superuser or the root user.
there is the superuser, or root user, that has elevated rights.
This user has the power create and modify users as well as to override any file ownership and permission.
``````sh
sudo -i  # simple way to switch to an interactive session as a root users

``````
##### Groups in Linux
 Groups are a collection of users.
 We can view groups on a system by viewing the /etc/group file
``````sh
cat /etc/group

``````

##### How to Read Symbolic Permissions


- Symbolic mode: this method uses symbols like u, g, o to represent users, groups, and others. Permissions are represented as  r, w, x for read write and execute, respectively. You can modify permissions using +, - and =.
- Absolute mode: this method represents permissions as 3-digit octal numbers ranging from 0-7.

``````sh
ls -l

``````
##### Add permissions mode changes
- +	Adds a permission to a file or directory
- –	Removes the permission
- =	Sets the permission if not present before. Also overrides the permissions if set earlier.

``````sh
chmod u+x mymotd.sh
--
chmod 451 file-name
sudo chmod 400

``````
##### Change Ownership using the chown Command

The chown command allows you to change the user and/or group ownership of a given file, directory, or symbolic link.

- USER is the user name or the user ID (UID) of the new owner.
- GROUP is the new group’s name or the group ID (GID).
- FILE(s) is the name of one or more files, directories, or links.
- Numeric IDs should be prefixed with the + symbol.

- USER - If only the user is specified, the specified user will become the owner of the given files. The group ownership is not changed
- USER: - When the username is followed by a colon :, and the group name is not given, the user will become the owner of the files, and the files group ownership is changed to the user’s login group.
- USER:GROUP - If both the user and the group are specified (with no space between them), the user ownership of the files is changed to the given user and the group ownership is changed to the given group.
- :GROUP - If the User is omitted and the group is prefixed with a colon :, only the group ownership of the files is changed to the given group.
- If only a colon : is given, without specifying the user and the group, no change is made.
``````sh
chown [OPTIONS] USER[:GROUP] FILE(s)
ls -l filename.txt

chown USER FILE
chown linuxize file1
chown linuxize file1 dir1

chown 1000 file2    #change the ownership of a file named file2 to a new owner with a UID of 1000

``````
##### Change the Owner and Group of a File
``````sh
chown USER:GROUP FILE

chown linuxize:users file1    #change the ownership of a file named file1 to a new owner named linuxize and group users
chown linuxize: file1        #the group of the file is changed to the specified user’s login group:

chown :www-data file1      #change the owning group of a file named file1 to www-data

``````
##### How to Change Symbolic Links Ownership
For example, if you try to change the owner and the group of the symbolic link symlink1 that points to /var/www/file1, chown will change the ownership of the file or directory the symlink points to:

``````sh
chown www-data: symlink1

chown -h www-data symlink1     #To change the group ownership of the symlink itself, use the -h option

``````
##### How to Recursively Change the File Ownership
To recursively operate on all files and directories under the given directory, use the -R (--recursive) option

The following example will change the ownership of all files and subdirectories under the /var/www directory to a new owner and group named www-data:
``````sh
chown -R USER:GROUP DIRECTORY

chown -R www-data: /var/www

chown -hR www-data: /var/www  #If the directory contains symbolic links, pass the -h option:

``````
