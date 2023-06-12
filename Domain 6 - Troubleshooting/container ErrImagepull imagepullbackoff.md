

##### Container Images

A container is a group of processes executing in isolation from the underlying system. A container image contains all the resources needed to run those processes: the binaries, libraries, and any necessary configuration.
A container registry is a repository for container images, where there are two basic actions:

Push: upload an image so it’s available in the repo

Pull: download an image to use it in a container - By only providing the name for the image, the image with tag latest will be pulled.
```sh
docker pull nginx
kubectl run mypod nginx
docker pull nginx:1.23.1-alpine
kubectl run mypod nginx:1.23.1-alpine
```
##### Image Pull Policy
Kubernetes features the ability to set an Image Pull Policy (imagePullPolicy field) for each container.
Based on this, the way the kubelet retrieves the container image will differ.
There are three different values for imagePullPolicy:

Always: kubelet will check the repository every time when pulling images for this container.
IfNotPresent: kubelet will only pull images from the repository if it doesn’t exist in the node locally.
Never: kubelet will never try to pull images from the image registry. If there’s an image cached locally (pre-pulled), it will be used to start the container.

If the image is not present locally, Pod creation will fail with ErrImageNeverPull error.

##### ErrImagePull
The status ErrImagePull is displayed when kubelet tried to start a container in the Pod, but something was wrong with the image specified in your Pod, Deployment, or ReplicaSet manifest.

```sh
kubectl get pods
-----
NAME    READY   STATUS             RESTARTS   AGE
goodpod 1/1     Running            0          21h
mypod   0/1     ErrImagePull       0          4s

```
Which means:

Pod is not in READY status
Status is ErrImagePull

the below is pointing to a 400 Error (BadRequest), since probably the image indicated is not available or doesn’t exist
```sh
kubectl logs mypod --all-containers
----
Error from server (BadRequest): container "mycontainer" in pod "mypod" is waiting to start: trying and failing to pull image

```
##### ImagePullBackOff
ImagePullBackOff is a Kubernetes waiting status, a grace period with an increased back-off between retries. Note that ImagePullBackOff is not an error,
 it’s just a status reason that is caused by a problem when pulling the image.
Back-off time is increased each retry, up to a maximum of five minutes.

```sh
kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
goodpod 1/1     Running            0          21h
mypod   0/1     ImagePullBackOff   0          84s

```
```sh
kubectl describe pod mypod
State:          Waiting
Reason:       ImagePullBackOff

```

##### Debugging ErrImagePull and ImagePullBackOff

There are several potential causes of why you might encounter an Image Pull Error. Here are some examples:

Wrong image name
Wrong image tag
Wrong image digest
Network problem or image repo not available
Pulling from a private registry but not imagePullSecret was provided

The best course of action would be to check:
```sh
kubectl describe pod podname
kubect logs podname –all-containers
kubectl get events --field-selector involvedObject.name=podname

```