

##### UID (user identifier)
number assigned by Linux to each user on the system. UIDs are stored in the /etc/passwd file

root user has the UID of 0
100 UIDs for system use
New users are assigned UIDs starting from 500 or 1000, For example, new users in Ubuntu start from 1000

---------
cat /etc/passwd

sudo useradd john
cat/etc/passwd | grep john

-----

###### GIDs (group IDs)
The GID of 0 corresponds to the root group
the first 100 GIDs are usually reserved for system use
GID of 100 usually represents the users group
New groups are usually assigned GIDs starting from 1000:

GIDs are stored in the /etc/groups file:

------
cat /etc/groups
cat /etc/group | grep accounting

----