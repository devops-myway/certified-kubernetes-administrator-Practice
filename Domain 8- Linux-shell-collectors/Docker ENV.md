#####  Dockerfile ENV Variables
You can use environment variables to send values to running containers.
They are part of the environment in which a container runs: you will see how this works below.

Syntax:

ENV
ENV =

``````sh
ENV myVar1=1
ENV myVar42=42
ENV myAlfaVar=alfa value abc

``````
#####  Example 2

``````sh
vi Dockerfile
FROM alpine:3.8
ENV myVar1 1
ENV my42 42
ENV myVar42=42
ENV myAlfaVar='alfa abc'

# Run the Build
docker build --tag tutorial:demo --file Dockerfile  .

# Run the container
docker stop -t 0 tutorial ;   docker container prune -f  
docker run -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'

docker exec -it tutorial /bin/sh

/ # printenv

# Run
docker stop -t 0 tutorial ;   docker container prune -f  

docker run -e  'my42=44000' -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'

# If you now enter the container you will see my42 is 44000.
docker exec -it tutorial /bin/sh

/ # printenv
``````
