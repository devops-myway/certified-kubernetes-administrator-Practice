#####  Dockerfile HEALTHCHECK
HEALTHCHECK automate checks the health of your containers on a schedule you specify.

HEALTHCHECK [OPTIONS] CMD command

The options that can appear before CMD are:
``````sh
--interval=DURATION (default: 30s)
--timeout=DURATION (default: 30s)
--start-period=DURATION (default: 0s)
--retries=N (default: 3)

``````
#####  

``````sh
vi Dockerfile

FROM alpine:3.8
HEALTHCHECK --interval=3s --timeout=1s \
   CMD curl -f http://localhost/ || exit 1

# Build the image using
docker build --tag tutorial:demo --file Dockerfile  .

# Let's start up a container
docker stop -t 0 tutorial ;   docker container prune -f  
docker run -ti -d --name tutorial tutorial:demo /bin/sh -c 'while true; do sleep 60; done'

# Test
docker ps -a

``````
You cannot execute: docker ps --filter status=healthy
You cannot execute: docker ps --filter status=unhealthy

You have to use:
docker ps -a | grep '(healthy)'
and
docker ps -a | grep '(unhealthy)'
