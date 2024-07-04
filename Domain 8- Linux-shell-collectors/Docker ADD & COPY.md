##### ADD and COPY

https://docs.docker.com/reference/dockerfile/#add

- The ADD: instruction is used to copy files, directories, or remote files from URL to your Docker images, from the 'src' to the absolute path 'dest'. Also, you can set up the default ownership of your file. ADD does Double Duty ... it adds and untars / unzips.

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

##### Recursive Copy of dirs Using Dockerfile COPY
Purpose: demo recursive copy of dirs using Dockerfile COPY
 Since we used WORKDIR demo-work-dir the current dir is demo-work-dir

 Therefore the COPY will copy all files from demo-directories into demo-work-dir.
``````sh
yum install tree
mkdir -p demo-directories/dir-a/dir-a1/dir-a2

# Let's add some files into each of those dirs.
touch demo-directories/demo-file
touch demo-directories/dir-a/file-a
touch demo-directories/dir-a/dir-a1/file-a1
touch demo-directories/dir-a/dir-a1/dir-a2/file-a2

# Dockerfile
FROM alpine:3.8
WORKDIR demo-work-dir
COPY demo-directories/ .

# Run
docker build --tag tutorial:demo --file Dockerfile .

docker stop -t 0 tutorial ; docker container prune -f;docker ps -a
docker run -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'

# Enter ls -R at the # shell prompt.
docker exec -it tutorial /bin/sh
/demo-work-dir # ls -R
``````
