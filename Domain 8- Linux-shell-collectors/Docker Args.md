#####  Dockerfile ARG
https://docs.docker.com/reference/dockerfile/#arg

The ARG instruction defines a variable that users can pass at build-time to the builder with the docker build command using the --build-arg = flag.
Arguments are normally used to pass things like user names, directory names, software package version numbers, etc to the build.

Note how WORKDIR /root/${target_dir} replace the ${target_dir} with dir1
 RUN echo ${target_dir} shows dir1 as well.
``````sh
ARG <name>[=<default value>]


vi Dockerfile

FROM alpine:3.8
ARG target_dir=dir1
WORKDIR /root/${target_dir}
RUN pwd
RUN echo  ${target_dir}
COPY gamefile /root/${target_dir}
COPY anothergame /root/${target_dir}

#Run
docker build --tag tutorial:demo --file Dockerfile  .


``````
#####  

``````sh

``````
