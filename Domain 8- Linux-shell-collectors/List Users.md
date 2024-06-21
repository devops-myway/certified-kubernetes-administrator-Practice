##### How to List Users in Linux
In Linux distributions, users are accounts that can interact with and manage the operating system.
They are divided into two types – normal and system users.

System users are created automatically when you install the operating system or a package.
You set up a regular account manually using the useradd command.

A normal user typically has four-digit numbers, while the system account commonly uses 1-999

Every Linux distribution stores all the users and their information within the /etc/passwd file. which comprises of line:

``````sh
# Each line represents a regular or system user account. Here’s what it means:

username:x:userID:groupID:userinfo:/homedir:/bin/bash

username – the account’s name.
x – the user’s encrypted password. The actual value is in /etc/shadow.
userID – the user’s identification number.
groupID – the group ID number the user belongs to.
Userinfo – extra information about the user.
/homedir – the user’s home directory.
/bin/bash – the user’s default login shell.

``````
##### How to List Users Using the cat Command

``````sh
cat /etc/passwd

``````
##### How to List Users With awk and cut Commands for Custom Output

``````sh
awk -F':' '{ print $1}' /etc/passwd

``````
##### Combine Commands With Pipes for Advanced User Listing
a pipe (|) lets you pass output from a command to another utility

``````sh
cat /etc/passwd | grep JohnDoe

``````
