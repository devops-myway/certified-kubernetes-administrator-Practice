#####  Dockerfile RUN
https://docs.docker.com/reference/dockerfile/#run

The RUN instruction will execute its given commands and create the resulting output in a new layer on top of the current image.
RUN is used to create probably the largest part of an image. It runs most of the commands you are used to use at the shell when setting up a server / vps.

Typical example.
``````sh
RUN mkdir ...
RUN useradd ..
RUN apt install ...
RUN copy ...
RUN unzip ...

``````
#####  Dockerfile Run Example

``````sh
vi Dockerfile

FROM alpine:3.8
RUN echo $HOME
RUN ["echo", "$HOME"]

docker build --no-cache --tag tutorial:demo --file Dockerfile  .

``````
#####  

``````sh

``````
