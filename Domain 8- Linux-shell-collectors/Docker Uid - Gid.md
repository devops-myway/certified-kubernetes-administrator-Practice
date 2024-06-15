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

###### UID and GID for Dockerfile
docker run --rm ruby:2.4 bundle exec rails g ...
docker-compose run --rm web composer install

owner and group of files that these commands generate are always root
#####  Dockerfile
The official document says we can do it by USER postgres, but we can also set group with :

USER 1000:1000
RUN addgroup demo && adduser -DH -G demo demo
First command creates group called demo. Second command creates demo user and adds him to previously created demo group.

Flags stands for:

-G Group
-D Don't assign password
-H Don't create home directory

##### Creating a User With Same UID in Container
we can create a user with a specific uid using the useradd command with the flag -u

``````sh
useradd baeldung -u 1000    #while inside the container, we could create a user baeldung with uid 1000.

# we’ve created the user, the mounted files and folders will now show baeldung as the owner
/  ls -l /opt/mount
total 4
drwxrwxr-x    2 baeldung baeldung      4096 Dec 27 07:48 inner-dir
-rw-rw-r--    1 baeldung baeldung         0 Dec 27 07:48 log1.txt
-rw-rw-r--    1 baeldung baeldung         0 Dec 27 07:48 log2.txt
``````
#### Customize Image With Dockerfile
We could put the script into a Dockerfile and bake it into the image itself. For instance, we could put the command that creates user baeldung with uid 1000 into the Dockerfile:

``````sh
FROM ubuntu:latest
RUN useradd baeldung -u 1000


``````
Alternatively, we could parameterize the uid value using an environment variable using the ARG Dockerfile directive. During build time, we’ll then pass the value using the –build-arg flag. To do that, we simply change the value 1000 to an environment variable:

Using the ARG directive, we can define a variable that we’ll pass during build time. Then, we could run the docker build command while specifying the –build-arg flag:

``````sh
FROM ubuntu:latest

ARG HOST_UID

RUN useradd baeldung -u $HOST_UID
----
sudo docker build --build-arg HOST_UID=$(id -u) --tag ubuntu-custom:latest .
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

