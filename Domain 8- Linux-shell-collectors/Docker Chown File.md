##### Using chown to Change File Ownership

I used COPY to copy one file on each line, not ADD.
``````sh
touch gamefile
touch anothergame

vi Dockerfile

FROM alpine:3.8
COPY --chown=games:games gamefile /root/
COPY --chown=35:35 anothergame /root/

#Build the image using
docker build --tag tutorial:demo --file Dockerfile  .

# start up a container to see the result.

docker stop -t 0 tutorial ;   docker container prune -f  
docker run -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'
docker exec -it tutorial /bin/sh

/ # ls -l /root
``````
##### What happened : extract from the Dockerfile:
Notice the first copy command uses a username and a groupname to change ownerships.
Notice the second copy uses a UID and a GID to change ownerships. 35 is the UID for the games user id.

GID = group identifier (GID)

UID = user identifier (GID)

``````sh
COPY --chown=games:games gamefile /root
COPY --chown=35:35 anothergame /root

``````
