#####  Reducing Docker Layers
The 3 Docker instructions, that create layers are:

- RUN: run a command
- COPY: include a local file
- ADD: include a local or remote file

That's it. Reducing the number of those instructions is how to reduce the number of final layers.

##### Pros of reducing layers:
Fewer layers to publish or download, likely reducing image size.
Chaining RUN instructions reduces cache-ability for non-deterministic commands such as apt-get update.
Chaining RUN instructions increases idempotency, which may desirable:

``````sh
# Don't use a cached "update" with an "install"
RUN apt-get update && \
    apt-get install -y <packages>


RUN set -euo pipefail && \
    apk --update add --no-cache autoconf automake g++ gcc make && \
    for PYTHON_VERSION in $(seq 2 3); do \
        (./configure PYTHON="$(which python${PYTHON_VERSION})" &&
        make -j$(nproc)) || exit 1; \
    done && \
    rm -rf /tmp/*
``````
#####  Use multi-stage builds
Multi-stage builds are great because only the layers from the last stage end up in the final image.

``````sh
FROM alpine AS builder
# COPY some things
# RUN some things
# RUN some more things
# COPY even more things
# RUN one last command that outputs an executable file "/app"

FROM alpine
COPY --from=builder /app /usr/local/bin/app
CMD ["app"]

``````
#####  Use a base image with fewer layers

``````sh
docker pull alpine:3.12.0
docker inspect --format '{{range .RootFS.Layers}}{{println .}}{{end}}' alpine:3.12.0
``````
#####  Combine multiple COPY and ADD instructions

``````sh
.
├── app
├── css
├── entrypoint.sh
├── img
└── js

COPY app app
COPY entrypoint.sh app/
COPY css static/css
COPY img static/img
COPY js static/js
``````
#####  Combine multiple RUN instructions

``````sh
# Install build dependencies
RUN apt-get update
RUN apt-get install -y autoconf automake g++ gcc make

# Configure and make
RUN ./configure
RUN make
RUN make install

# Cleanup
RUN rm -rf /var/lib/apt/lists/*

#You can combine those commands with &&
RUN apt-get update && \
    apt-get install -y autoconf automake g++ gcc make && \
    ./configure && \
    make && \
    make install && \
    rm -rf /var/lib/apt/lists/*

#Run
# Copying every file you need
COPY . ./

# Some very long command
RUN sleep 300
``````
