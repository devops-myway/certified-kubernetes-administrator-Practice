#####  Dockerfile LABEL
https://docs.docker.com/reference/dockerfile/#label

The LABEL instruction is a key-value pair that allows you to add multiple labels and multi-line values.
With this instruction, you can add additional information about your Docker image, such as the version, description, maintainer, etc.
``````sh
vi Dockerfile

FROM alpine:3.8
LABEL version="demo label"
LABEL description="This text illustrates \
that label values can span multiple lines."
CMD echo "container with labels"


# Run
docker build --tag tutorial:demo --file Dockerfile  .

# To show all the labels for an image, run
docker inspect tutorial:demo

docker images --filter "label=version=demo label"

``````
#####  

``````sh

``````
