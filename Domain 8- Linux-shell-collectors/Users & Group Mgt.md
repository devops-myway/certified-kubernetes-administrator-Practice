https://phoenixnap.com/kb/usermod-linux#:~:text=The%20usermod%20command%20is%20one,%2C%20default%20shell%2C%20and%20more.

##### How to Create/Add Users in Linux
The "useradd" command provides various options, resulting in a comprehensive way to automate identity and access management.

Running the command creates a new user account or updates an existing user according to the values in:

/etc/default/useradd - The default values for the useradd command.
/etc/login.defs - Configuration control values for the login package.

``````sh
useradd <options> <username>

``````
##### Understanding User Management Files
Linux stores user and group data in specific files and directories that contain user account info, passwords, and group configurations.
The main files and directories for storing user data in Linux include:
- /etc/passwd. The passwd file contains a list of user accounts and the corresponding user ID, group ID, home directory, and the default shell.It is readable by most users, but only root and sudo accounts can add new users or remove and modify existing user data.
- /etc/group. The group file contains a list of user groups. Each line in the file represents a group and displays the group name, GID, and group members
- /etc/login.defs. The login.defs file contains system-wide user account policy settings, like the password aging policy.
- /etc/sudoers. The sudoers file specifies which users have elevated permissions, on which machines, and for which directories.

##### Linux User Management Commands

``````sh
who                       #Check Currently Logged Users
who -H                    # Display the header
cat /etc/passwd           # List all users

``````
##### Create User (useradd/adduser)
-u <uid>: --uid <uid>	: Unique numerical value ID.
-p <password>: --password <password>	: Sets the user's password (not recommended).
-g <name or number>: --gid <name or number>	: Establishes the user's initial login group.

``````sh
useradd <options> <username>
sudo useradd test_account

``````
##### Creating New Users in Linux
Creating new users in Linux does the following:

1. Provides a unique UID and GID.

- 0 is reserved for root and assigned automatically.
- 1-999 is for system or service accounts and services.
- 1000 and above are for regular users.

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
##### Adding a User with Specific Group ID

``````sh
sudo useradd -g <group name or GID> <username> or sudo useradd -G <group1,group2,group3> <username>
id <username>

sudo useradd -g 1000 test2
id test2
# output
uid=1000(test2) gid=1000(kb) groups=1000(kb)

``````
##### Adding a System User
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
