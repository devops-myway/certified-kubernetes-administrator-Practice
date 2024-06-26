

##### Pushing a Docker Image to a Self-Hosted Registry
A Docker registry is a service that manages container image repositories.
It allows us to do things like create repositories, push and pull images, and manage repository access.
Cloud-hosted Docker registries are also called public registries. Docker Hub is a well-known example of a public registry.

#### Tagging Images for Self-Hosted Registries
When pushing an image, the image name defines where the image will be pushed.
The image name must contain the registry host, the port, and the repository name.

- HOST: The optional registry hostname where the image is located. If no host is specified, Docker's public registry at docker.io is used by default
- PORT_NUMBER: The registry port number if a hostname is provided
- PATH: The path of the image, consisting of slash-separated components. NAMESPACE or organization's name/]REPOSITORY. If no namespace is specified, library is used, which is the namespace for Docker Official Images 

``````sh
[registry host]:[registry port]/[repository name]:[tag name]
[HOST[:PORT_NUMBER]/]PATH[:TAG]

docker build -t my-username/my-image .   #image during a build
docker image tag my-username/my-image another-username/another-image:v1  #already built an image,

docker build -t localhost:5000/my-fancy-app:1.0.0 .

``````
##### Publishing images
Once you have an image built and tagged, you're ready to push it to a registry.
``````sh
docker push my-username/my-image

#Re-Tagging an Existing Image
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

docker tag my-fancy-app:1.0.0 localhost:5000/my-fancy-app:1.0.0
``````
##### Try it out
``````sh
git clone https://github.com/docker/getting-started-todo-app

docker build -t <YOUR_DOCKER_USERNAME>/concepts-build-image-demo .
docker build -t mobywhale/concepts-build-image-demo .
docker image ls
docker image history mobywhale/concepts-build-image-demo

``````
##### Push the image
Push the image using the docker push command:
``````sh
docker login [OPTIONS] [SERVER]
$ docker login localhost:5000
docker push [OPTIONS] NAME[:TAG]
docker push <YOUR_DOCKER_USERNAME>/concepts-build-image-demo
$ docker push localhost:5000/my-fancy-app:1.0.0

``````
