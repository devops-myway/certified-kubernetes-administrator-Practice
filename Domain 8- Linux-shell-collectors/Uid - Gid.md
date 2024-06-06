

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
###### UID and GID for Dockerfile: https://github.com/moby/moby/pull/34263
A --chown flag has finally been added to COPY:

COPY --chown=patrick hostPath containerPath

--chown=someuser:555
--chown=anyuser:anygroup
--chown=1001:1002
--chown=333:agroupname

