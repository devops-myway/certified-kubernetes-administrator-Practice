##### ADD and COPY

https://docs.docker.com/reference/dockerfile/#add

ADD does Double Duty ... it adds and untars / unzips.
COPY does not automate untars tar files.

Best Practice: only use ADD when you need this double duty functionality.

The ADD tarredfiles.tar /root in the Dockerfile automate untars tarredfiles.tar into /root/.

The COPY more-tarredfiles.tar /root in the Dockerfile merely copies more-tarredfiles.tar into /root/.

``````sh
touch in-tar-file-1
touch in-tar-file-2
touch in-tar-file-3

tar -cvf tarredfiles.tar in-tar-*

touch more-in-tar-file-1 ; touch more-in-tar-file-2 ; touch more-in-tar-file-3 ;
tar -cvf more-tarredfiles.tar more-in-tar-*

vi Dockerfile

FROM alpine:3.8
ADD tarredfiles.tar /root
COPY more-tarredfiles.tar /root

docker build --tag tutorial:demo --file Dockerfile  .
docker stop -t 0 tutorial   ;   docker container prune -f   ;   docker ps -a

# Let's start up a container to see the result.
docker stop -t 0 tutorial ;   docker container prune -f  
docker run -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'
docker exec -it tutorial /bin/sh

/ # ls /root

``````



``````
