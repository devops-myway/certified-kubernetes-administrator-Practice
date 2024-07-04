https://docs.docker.com/reference/dockerfile/#copy

#### Docker COPY as non root?
While building a Docker image, how do I COPY a file into the image so that the resulting file is owned by a user other than root?

All files and directories copied from the build context are created with a UID and GID of 0,
unless the optional --chown flag specifies a given username, groupname, or UID/GID combination to request specific 
ownership of the copied content.

``````sh
COPY --chown=<user>:<group> <hostPath> <containerPath>
COPY [OPTIONS] <src> ... <dest>
COPY [OPTIONS] ["<src>", ... "<dest>"]
COPY [--chown=<user>:<group>] [--chmod=<perms> ...] <src> ... <dest>
``````
#### Chown UID and GID
If a username or groupname is provided, the container's root filesystem /etc/passwd and /etc/group files 
will be used to perform the translation from name to integer UID or GID respectively.

``````sh
COPY --chown=<user>:<group> <hostPath> <containerPath>

COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
COPY --chown=myuser:mygroup --chmod=644 files* /somedir/
``````
