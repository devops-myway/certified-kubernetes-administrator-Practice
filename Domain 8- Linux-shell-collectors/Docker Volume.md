##### Dockerfile VOLUME
https://docs.docker.com/reference/dockerfile/#volume

The VOLUME: instruction or directive creates a mount point and it creates a directory in the Docker managed /var/lib/docker/volumes/

Confusing: marks it as holding externally mounted volumes from native host or other containers.
It does not hold any external mounted volumes - it creates a new volume internal to the container.

The shorter the list the better you will able to see / find the volumes we are going to create
``````sh
docker volume list

# Result
DRIVER              VOLUME NAME
local               de82d92daf539b7147770877704ed438b053c50744e874e7a6b86655cd50cf44

#Run
docker volume inspect de82d92daf539b7147770877704ed438b053c50744e874e7a6b86655cd50cf44

ls /var/lib/docker/volumes/de82d92daf539b7147770877704ed438b053c50744e874e7a6b86655cd50cf44/_data/

docker stop -t 0 tutorial
docker container prune -f;docker ps -a
docker volume list    #you will notice the volume is gone.

``````
##### Example 1
The name /myvol exists only inside this container.
myvol is the name and mount point of the volume ONLY inside the container.
No other container can refer to the myvol name / mountpoint.


Enter the ls command - see that myvol exists inside the container.
Enter cat myvol/greeting

``````sh

vi Dockerfile

FROM alpine:3.8
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol

--
docker build --tag tutorial:demo --file Dockerfile  .
docker stop -t 0 tutorial
docker container prune -f
docker ps -a
docker run --rm -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'
docker exec -it tutorial /bin/sh -c ls
# cat myvol/greeting
# exit
``````
##### More About anon Volumes

Let's run our container with /myvol again.
docker volume list should show one anon volume - random volume name
Let's start 3 other Alpine containers using an anon volume /myvol as well.
``````sh
docker run -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'

docker run -d --name alpine1 -v /myvol alpine:3.8  /bin/sh -c 'while true; do sleep 60; done'
docker run -d --name alpine2 -v /myvol alpine:3.8  /bin/sh -c 'while true; do sleep 60; done'
docker run -d --name alpine3 -v /myvol alpine:3.8  /bin/sh -c 'while true; do sleep 60; done'

#Run
docker volume list
docker volume prune

docker stop -t 0 tutorial
docker stop -t 0 alpine1
docker stop -t 0 alpine2
docker stop -t 0 alpine3
``````
#####


``````sh


``````
