
##### Users and Groups in Linux
There are two types of users: system users and regular users
System users are responsible for running non-interactive and background processes on a system. For example: mail, daemon, syslog, and so on.
Regular users are the users that actually log into the system and perform their designated tasks interactively.

3 type OWNERSHIP
1.user, 2.group, 3.other

3 type PERMISSION

1.Read, 2.Write, 3.Excute

``````sh
# check details of users on a system by looking into the /etc/passwd file
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
##### View Ownerships and Permissions in Linux
We can use long listing which is the ls command with flag -l
File type:
For regular files that contain simple data it is blank -
For a directory which is a special file, it is d
``````sh
ls -l

``````
##### How to Read Symbolic Permissions
r stands for read. It is indicated in the first character of the triad.
w stands for write. It is indicated in the second character of the triad.
x stands for execution. It is indicated in the third character of the triad.

- Symbolic mode: this method uses symbols like u, g, o to represent users, groups, and others. Permissions are represented as  r, w, x for read write and execute, respectively. You can modify permissions using +, - and =.
- Absolute mode: this method represents permissions as 3-digit octal numbers ranging from 0-7.
0=>No permission, 1=>Excute, 2=>Write, 3=>Execute + Write

4=>Read, 5=>Read + Execute, 6=>Read +Write, 7=>Read + Write +Execute

``````sh
ls -l

``````
##### Add permissions mode changes
+	Adds a permission to a file or directory
–	Removes the permission
=	Sets the permission if not present before. Also overrides the permissions if set earlier.

``````sh
chmod u+x mymotd.sh
--
chmod 451 file-name
sudo chmod 400

``````
##### Change Ownership using the chown Command

You can change the ownership of a file or folder using the chown command

``````sh
chown user filename
--
chown user:group filename
-- # How to change directory ownership
chown -R admin /opt/script
-- # How to change group ownership
chown :admins /opt/script
``````