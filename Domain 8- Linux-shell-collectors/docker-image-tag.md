##### What Is Docker Image Tag
A tag is essentially a label assigned to a Docker image to help identify it.t typically consists of two components:

- Image name (also known as repository name): This is the name of the image.
  
- Tag: This is an optional identifier, commonly used to represent a specific version or variant of the image. If no tag is specified, Docker automatically assigns the latest tag to the image by default (more on this later

Consider the tag Ubuntu:18.04. In this case, Ubuntu is the name of the Docker image, and 18.04 is the tag representing a specific version of Ubuntu

##### Why Tag a Docker Image
But in a real-world situation where you have a big development team and tens or hundreds of Docker images floating around, it's crucial to have a consistent and meaningful tagging strategy in place

Here are some reasons and benefits of tagging Docker images:

- Version control: Tagging Docker images allows you to maintain different versions of the same application, making it easier to roll back to a previous version if needed. This can be especially useful in cases where a new release introduces a bug or breaks compatibility with other components
- Traceability and accountability: Tags provide a way to track the origin and history of an image, which is important for understanding the evolution of an application. This can help you identify when a particular change was introduced or who is responsible for a specific update.
- Simplified deployment and automation: A consistent tagging strategy makes it easier to automate deployment processes and manage the application lifecycle. For example, a continuous integration and continuous deployment (CI/CD) pipeline can automatically build, test, and deploy new versions of the application based on the tags assigned to the Docker images.

##### Tag a Docker image during the build process
run the following command to create an image tagged my-app:1.0.0
dot (.): after the build command indicates that the current directory or path (containing the Dockerfile) is the build context.
-t flag:  to tag the Docker image with the name my-image and version 1.0.0

Docker terminology, a Repository= image name, is a collection of different versions of an image with the same name but tagged with different versions
``````sh
vi Dockerfile

FROM alpine
LABEL maintainer=kodekloud
RUN apk update && apk add curl
ENTRYPOINT ["curl", "--version"]

docker build . -t my-image:1.0.0
docker image ls

``````
##### Tag a Docker image after the build process
 You may want to assign a new tag to an existing Docker image. it could specify a specific version or a particular environment like development or production
 
 You can use the docker image tag command,
- docker image tag: This is the base command that tells Docker you want to tag an image.
  
- SOURCE_IMAGE[:tag]: This is the name of the existing Docker image that you want to tag.
  
- TARGET_IMAGE[:tag]: This is the name you want to apply to the image.

The image with the ID 806dd150c8fb is now associated with two different tags: my-image:1.0.0 and my-image:1.0.0-alpine.
Both tags are just pointers to the same Docker image (with ID "806dd150c8fb").

``````sh
docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

docker image tag my-image:1.0.0 my-image:1.0.0-alpine
docker image ls

``````
##### Tagging a Docker image for Docker Hub
To push or upload a Docker image to Docker Hub or registry, you need to tag it appropriately.
you need to prefix your image name with your Docker Hub username
``````sh
docker push <DOCKER_HUB_USERNAME/IMAGE_NAME[:tag]>

``````
