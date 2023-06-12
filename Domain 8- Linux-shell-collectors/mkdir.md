
##### How to Make Parent Directories
if you want to create “dirtest2” in “dirtest1” inside the Linux directory (i.e., Linux/dirtest1/dirtest2)
using the -p option
``````sh
mkdir directory_name

mkdir –p Linux/dirtest1/dirtest2

``````
##### How to Set Permissions When Making a Directory
The mkdir command by default gives rwx permissions for the current user only.
To add read, write, and execute permission for all users, add the -m option with the user 777 when creating a directory.
``````sh
mkdir –m777 DirM

``````

``````sh


``````