#### Docker Image Tag

https://docs.docker.com/reference/cli/docker/image/tag/
https://dev.to/zahraajawad/build-tag-and-push-docker-images-to-docker-hub-and-amazon-elastic-container-registry-4hbm

In Docker, a "Tag" is an identifying tag attached to a Docker image to distinguish it from different versions of the same image
or to specify other uses for it.
The tag is appended to the image name with a validated colon (:) to create a unique name for this version or interpretation of the image.

• SOURCE_IMAGE: is the name of the image you want to tag.
• TAG: is the tag associated with the source image (optional).
• TARGET_IMAGE: is the new name you want to give to the image.
• TAG: is the new tag you want to assign to the image (optional).

``````sh
docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
docker tag [image] [username of docker hub or acr or ecr or gcr/image]:[tag name]

# Acr
docker tag mywebapp:v1 mywebappacrdemo.azurecr.io/mywebapp:v1
docker push mywebappacrdemo.azurecr.io/mywebapp:v1

# ECR
aws ecr create-repository --repository-name my-repo-name --region your- region
aws ecr get-login-password --region your-region | docker login --username AWS --password-stdin your-account-id.dkr.ecr.your-region.amazonaws.com

docker tag my-image:latest your-account-id.dkr.ecr.your- region.amazonaws.com/my-repo-name:latest
docker push your-account-id.dkr.ecr.your-region.amazonaws.com/my-repo-name:latest
``````

##### Tag an Image by Docker tag Command
Each image has a tag, so when the image is pulled without any tag, it pulls the latest.
when we run the docker Images command, we show that the image is tagged and takes the name that is specified to it

``````sh
# Tag without name, by the command
docker tag [image] [user name of docker hub/image]
docker tag nginx xyz/nginx

#Tag with name, by the command
docker tag [image] [user name of docker hub/image]:[tag name]

docker tag nginx xyz/nginx:web
docker image ls

``````
#### Upload Image to DockerHub
to upload the image to Docker Hub (the account must be available on Docker Hub to be able to upload the image), 
you must first login to your Docker Hub account with the command:
To make sure the image has been pushed to Docker Hub, go to Account and note that the image has been pushed.m
``````sh
docker login
# Docker Hub username and password

docker push [dockerhub username or ecr or acr username/image:tag]
``````
#### 

``````sh

``````
