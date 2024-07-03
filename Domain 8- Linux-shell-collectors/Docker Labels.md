#####  Dockerfile LABEL
https://docs.docker.com/reference/dockerfile/#label

Images can have an unlimited number of labels. Those labels can be used to describe the image.
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
