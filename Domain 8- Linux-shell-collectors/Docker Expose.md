#####  Dockerfile EXPOSE Ports
https://docs.docker.com/reference/dockerfile/#expose

Dockerfile uses EXPOSE to expose ports of the container. The EXPOSE instruction declares the ports the container listens on.
A port is the official way a process allows other processes to contact it and send it commands.

For example with Apache port 80 in normally exposed. Apache hides isolated within its container. The ONLY way to get it to do something is to access it via port 80.

To actually publish the port when running the container, use the -p flag on docker run to publish and map one or more ports

By default, EXPOSE assumes TCP. You can also specify UDP:

EXPOSE 80/tcp

EXPOSE 80/udp

-t, --tcp display only TCP sockets
-a, --all display all sockets
-n, --numeric don't resolve service names, show numeric port number

``````sh
vi Dockerfile

FROM alpine:3.8
EXPOSE 80/tcp
EXPOSE 80/udp

# Run
docker build --tag tutorial:demo --file Dockerfile  .

# Run
docker run -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'

# We must publish them. We do this using -p 30000:80/tcp on the docker run command.
# The 30000 specifies the host port number. 80/tcp specifies the container port number.

docker stop -t 0 tutorial; ;   docker container prune -f  
docker run  -p 30000:80/tcp  -ti -d --name tutorial tutorial:demo /bin/sh -c '\''while true; do sleep 60; done'\'''

# 0.0.0.0:30000->80/tcp
# Port 30000 on localhost is mapped to port 80 in the container.
# We can check that port 30000 is now open on the host:
# The ss command is used to show information about sockets.
# It is similar to netstat command. ( The netstat command no longer exists in default installation of the CentOS distro.

ss -t -a -n
``````
#####  Let's publish udp port 80 as well. Run:

``````sh
docker stop -t 0 tutorial ;   docker container prune -f  
docker run -p 40080:80/udp 30000:80/tcp -

ss -u -a -n
``````
#####  

``````sh

``````
