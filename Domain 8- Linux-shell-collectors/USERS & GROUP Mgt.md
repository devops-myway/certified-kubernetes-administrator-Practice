https://phoenixnap.com/kb/usermod-linux#:~:text=The%20usermod%20command%20is%20one,%2C%20default%20shell%2C%20and%20more.

##### Understanding Group
- The idea of groups is central to the Linux operating system. All users are members of at least one group. These groups can be broadly classified into Primary and Secondary Groups.
- Primary Group: Typically, the primary group shares the same name as the user, which dictates the group ownership of any files they create
- Every user in the Linux system is assigned to a primary group, and the information about a user’s primary group can be found in the /etc/passwd file.
- Secondary Group: Users have the flexibility to join multiple secondary groups or even opt not to be part of any
- Secondary Group: These secondary groups are often created to handle permissions for certain software applications and files
- Group members automatically inherit the read, write, and execute permissions admins set for that group
- The /etc/group file contains the details about the secondary groups a user belongs to.
- 
##### How to Create a User Group
- you can use the -g <GID> flag with the groupadd command
- Options to provide GID or UID for different users:
- 0 is reserved for root and assigned automatically.
- 1-999 is for system or service accounts and services.
- 1000 and above are for regular users.
- When creating a new group, you can assign a unique group ID (GID) to distinguish it from other groups
- If you’re creating a system group reserved for system accounts and services, you need to assign it a GID of less than 1000
- For this, you can use the -r flag with the groupadd command:
``````sh
Syntax:
groupadd <groupname>

groupadd mynewgroup
grep mynewgroup /etc/group   #This command uses the grep utility to extract information about the group from the /etc/group file.

groupadd -g 1005 mynewgroup
groupadd -r <systemgroupname>
``````
##### Add an Existing User to a Group
- You can add an existing user to a group with the usermod or the gpasswd command
- The -a flag tells the command to append the user to additional groups
- The -G flag indicates the supplementary group(s) to add the user to.

##### Add Existing Users to a Group with the usermod Command
``````sh
The syntax of the usermod command to add a user to a group is:
# usermod -a -G <groupname> <username>

 usermod -a -G sudo harry
 
``````
##### Add Existing Users to a Group with the gpasswd Command
- The -a flag indicates that you wish to add the user to the group. Alternatively, you can use the — add user flag.
``````sh
# Add Existing Users to a Group with the gpasswd Command
The gpasswd command offers another way of adding a user to a group. The typical syntax of the command is:

# gpasswd -a <username> <groupname>

gpasswd -a tom sudo
``````
##### Add a User to Several Groups at Once
- You can use the usermod command with the -aG flag, followed by a list of groups separated by commas
- Always use the -a option with usermod -G to append the user to groups so you don’t accidentally remove them from other groups they might already be a part of
``````sh
syntax of this command:
# usermod -aG <group1>,<group2>,<group3> <username>

usermod -aG developers,designers,admins john

``````
##### Create a New User and Add Them to a Group(s)
- you can use the useradd command with the b flags.
- new user account named Susan and add it to the developers group.
- set a password for this new user
- Use the useradd command with the -g flag, followed by the desired primary group name
- Note that the lowercase -g flag is used for primary groups and the uppercase -G for secondary groups
``````sh
useradd -m -G <groupname> <newuser>
sudo passwd <username>

useradd -g <primary_groupname> <username>
useradd -g admins johny

#Change the Primary Group of an Existing User
sudo usermod –g <group_name> <user_name>
sudo usermod -g developers johny

#Delete a User from a Group with the gpasswd Command
gpasswd -d username groupname

#How to Delete a Group
groupdel <groupname>
groupdel designers

``````
##### List All Groups on the System
- listing the contents of the /etc/group file to see all the groups (and the associated user accounts) on the system
``````sh
cat /etc/group

# List All Groups for a User Account
groups <username>
id <username>

``````
``````sh
https://www.redswitches.com/blog/add-user-to-group-linux/

``````
##### Understanding User Management Files
Linux stores user and group data in specific files and directories that contain user account info, passwords, and group configurations.
The main files and directories for storing user data in Linux include:
- /etc/passwd. The passwd file contains a list of user accounts and the corresponding user ID, group ID, home directory, and the default shell.It is readable by most users, but only root and sudo accounts can add new users or remove and modify existing user data.
- /etc/group. The group file contains a list of user groups. Each line in the file represents a group and displays the group name, GID, and group members
- /etc/login.defs. The login.defs file contains system-wide user account policy settings, like the password aging policy.
- /etc/sudoers. The sudoers file specifies which users have elevated permissions, on which machines, and for which directories.

##### Create User (useradd/adduser)
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
##### Creating New Users in Linux
Creating new users in Linux does the following:

1. Provides a unique UID and GID.

- 0 is reserved for root and assigned automatically. - ROOT USER
- 1-999 is for system or service accounts and services. - SERVICE ACCOUNT USERS
- 1000 and above are for regular users.  - REGULAR ACCOUNT

2. Edits files that store account information
- /etc/passwd - Lists all registered users on the system.
- /etc/shadow - Stores encrypted user passwords.
- /etc/group - Defines user groups.
- /etc/gshadow - Stores encrypted group passwords.

##### Adding a User in Linux
The x character represents and hides the user's password for security reasons. The encrypted password is in the /etc/shadow file.
``````sh
sudo useradd <username> : sudo useradd test
sudo passwd <username>  : Enter Password
sudo cat /etc/passwd | grep <username>  : sudo cat /etc/passwd | grep test

# The fields are in the following format
test:x:1001:1001::/home/test:/bin/sh
username:password:UID:GID:info:/home/directory:shell/path
``````
##### Adding a User with Specific User ID

``````sh
sudo useradd -u <uid> <username>
id <username>                       #check id with id

sudo useradd -u 1234 test1
id test1

``````
##### Change Your Own Password for Another user

``````sh
passwd             #chnage your current user password

sudo passwd user1  # change another user password

sudo passwd root   #change the root password

sudo passwd -e user1   # force a pwssword Reset
``````
##### Adding a User with Specific Group ID

``````sh
sudo useradd -g <group name or GID> <username> or sudo useradd -G <group1,group2,group3> <username>
id <username>

sudo useradd -g 1000 test2
id test2
# output
uid=1000(test2) gid=1000(kb) groups=1000(kb)

``````
##### Adding a System User or Service Account
Programs and systems create service or system user accounts, which are different from regular users. Programs such as MySQL or Tomcat require a unique user account to work on the system, and daemons typically create system users during installation.

``````sh
sudo useradd -r <username>                 # To create a system user use -r option
sudo cat /etc/passwd | grep <username>     # check the users information

sudo useradd -r testsvc
cat /etc/passwd | grep testsvc

# output
testsvc:x:998:998::/home/testsvc:/bin/sh
``````
##### The adduser Command
The adduser command is an alternative way to add users to a Linux system and acts as a simple interactive front end for useradd.
``````sh
sudo adduser <username>
sudo cat /etc/passwd | grep <username>

``````
##### Use the usermod Command in Linux
The "usermod" command is one of the several Linux commands system administrators have at their disposal for user management
The usermod command modifies configuration files containing user account information.
``````sh
usermod [options] [username]

#Add Information to a User
sudo usermod -c "[information]" [username]

sudo usermod -c ""This is mike from Sales" mike
getent passwd mike

``````
##### Set User’s Home Directory

``````sh
sudo usermod -d [directory-location] [username]

sudi usermod -d /var/mike mike
getent passwd mike
``````
##### Change User’s Login Name


``````sh
sudo usermod -l [newname] [oldname]

sudo usermod -l micheal mike
``````
##### Change User’s UID

``````sh
sudo usermod -u [new-UID] [username]

sudo usermod -u 1401 mike
getent passwd mike
``````
##### Linux User Group Management Commands
User groups in Linux streamline the management of permissions and access rules for a collection of user accounts.

- Use the "getent" command to confirm the new group is in the Linux /etc/group file:
- command displays the details of test_group, including its name and ID.
``````sh
sudo groupadd test_group
getent group test_group

sudo adduser test_account test_group    #add user to group
``````
##### How to Add or Remove Members From Group (usermod)
Use the usermod command with the -a (append) and -G (groups) options to append a user to an existing group while keeping them in their current groups:
``````sh
sudo usermod –aG test_group test_account2
groups test_account     #Confirm if the user has been added by using the groups command

``````
##### Delete user from Group
To remove a user from a specific group, use the deluser command:
``````sh
sudo deluser test_account test_group

``````
##### Display All Groups a User Is a Member Of (id)
Enter the id command to list the groups an individual user is a member of:
The -n and -G options instruct id to only list group names instead of numeric IDs.
``````sh
id test_account
id -nG test_account
``````
##### List all Groups and Members

``````sh
getent group
``````
