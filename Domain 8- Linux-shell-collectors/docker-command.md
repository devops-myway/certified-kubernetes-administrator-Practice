
##### Docker’s COPY Command
The COPY command is a fundamental tool in Docker, primarily used during the building of Docker images. Its core function is to duplicate files and directories from a specified location into the Docker image.  COPY doesn’t complicate the process by extracting compressed files.

The source represents the local file or directory you want to copy, and destination is the location within the Docker image where you want the copied files to reside.
``````sh
COPY source destination

# Dockerfile copy from host locations files to docker images during build
FROM ubuntu:18.04
COPY ./myapp /usr/src/myapp

----
docker cp containerID:source destination

``````
##### ADD Command
the ADD command is its ability to extract compressed files during the copying process.
The ADD command can download and copy files from a URL.
it’s crucial to note that downloading files with the ADD command can lead to unpredictable Dockerfile behavior and is generally discouraged.

source can be a local file, directory, or URL, and destination is the location within the Docker image where you want the copied files to reside.
``````sh
ADD source destination

# Dockerfile
FROM ubuntu:18.04
ADD http://example.com/big.tar.xz /usr/src/things/

``````
##### 

``````sh

``````
