https://docs.docker.com/build/building/multi-stage/

#####  Using Multi-Stage Docker Builds
Multi-stage Docker builds can greatly reduce the size of final built images, and the savings can be extreme with Go.
It's important to keep the size of Docker images small as it reduces the size that needs to be published, hosted, and downloaded.

Publishing is usually infrequent, and hosting is free through a number of container registries, 
so those aren't a big deal - but the download size will affect everyone and every cluster that wants to use your image.

As a general rule of thumb, reducing the number of layers in a final image helps reduce the size.

###### Multi-Stage DockerFile
Docker has a feature called multi-stage builds where multiple FROM instructions can be used in a single Dockerfile,
each with an optional alias. Each stage can use a different base (e.g. golang:1.14, node:lts, alpine:latest, etc.), 
and each stage can copy files from previous stages with the COPY instruction.

``````sh
main.go

package main
import "fmt"
func main() {
    fmt.Println("hello world")
}

go run main.go

``````
#####  # Single-stage build
8 layers! Granted, 5 of them are coming from the base image - but that's a lot, especially for such a small application.
``````sh
FROM golang:1.14.4-alpine
WORKDIR /go/src/app
COPY main.go .
RUN go build -o /go/bin/app main.go
CMD ["app"]

docker build -t example .
docker run example

docker image ls example
docker run golang:1.14.4-alpine sh -c "du -sh /usr/local/go/*"

docker inspect example --format '{{range .RootFS.Layers}}{{println .}}{{end}}'

``````
#####  Multi-stage build
Change the Dockerfile to add an extra FROM instruction that will create a second stage that will COPY the built binary from the first stage:
7.64MB! Roughly 2% of the size it was before!

The second FROM instruction starts a new build stage with the scratch image as its base. The COPY --from=0 line copies 
just the built artifact from the previous stage into this new stage. 

``````sh
FROM golang:1.14.4-alpine AS builder
WORKDIR /go/src/app
COPY main.go .
RUN go build -o /go/bin/app main.go

FROM alpine:latest
COPY --from=builder /go/bin/app /usr/local/bin/app
CMD ["app"]

docker build -t example .
docker run example

docker image ls example

``````

``````sh


``````
