#####  Dockerfile FROM Instruction
The Dockerfile FROM instruction specifies the base image / Linux distro we want to use to build a container.
Every Dockerfile must have a FROM instruction.

``````sh
docker pull alpine:3.8

FROM alpine:3.8
docker image remove alpine:3.8

#run
docker build --file Dockerfile .
``````
#####  

``````sh

``````
