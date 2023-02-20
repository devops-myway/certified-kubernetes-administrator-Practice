##### Overview on Kubernetes Volumes

- Now you must have a basic idea on Kubernetes Volumes. These are a component of a pod and are thus defined in the pod’s specification much like containers.
- They aren’t a standalone Kubernetes object and cannot be created or deleted on their own.
- A volume is available to all containers in the pod, but it must be mounted in each container that needs to access it.
- In each container, you can mount the volume in any location of its filesystem.

Different volume types in Kubernetes:

emptyDir	                  Localhost
hostPath	                  Localhost
glusterfs	                  GlusterFS cluster
downwardAPI	                  Kubernetes Pod information
nfs	                          NFS server
awsElasticBlockStore	      Amazon Web Service Amazon Elastic Block Store
gcePersistentDisk	          Google Compute Engine persistent disk
azureDisk	                  Azure disk storage
projected	                  Kubernetes resources; currently supports secret, downwardAPI, and configMap
secret	                      Kubernetes Secret resource
vSphereVolume	              vSphere VMDK volume
gitRepo	                      Git repository

##### Using emptyDir on Disk
emptyDir is the simplest volume type, which will create an empty volume for containers in the same Pod to share
``````sh
[root@controller ~]# cat shared-volume-emptyDir.yml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-emptyDir
spec:
  containers:
  - name: alpine1
    image: alpine
    command: ["/bin/sleep", "999999"]
    volumeMounts:
    - mountPath: /alpine1
      name: data
  volumes:
  - name: data
    emptyDir: {}
--
kubectl create -f shared-volume-emptydir.yml

kubectl get pods shared-volume-emptydir

kubectl get pods shared-volume-emptydir
----
kubectl exec -it shared-volume-emptydir -c alpine1 -- touch /alpine1/someFile.txt
-- # So the containers are created on worker1 node. List the available docker containers on worker1
docker ps
``````
##### Using hostPath
hostPath acts as data volume in Docker.
hostPath volumes are the first type of persistent storage, because both the gitRepo and emptyDir volumes’ contents get deleted when a pod is torn down, whereas a hostPath volume’s contents don’t.

Here we are creating a new Pod using hostPath as /tmp/data:
``````sh
[root@controller ~]# cat shared-volume-hostpath.yml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-hostpath
spec:
  containers:
  - name: alpine1
    image: alpine
    command: ["/bin/sleep", "999999"]
    volumeMounts:
    - mountPath: /alpine1
      name: data
  volumes:
  - name: data
    hostPath:
      path: /tmp/data
---
 kubectl create -f shared-volume-hostpath.yml

 kubectl get pods shared-volume-hostpath -o wide
-- #Now we if create an empty file in the shared path on any of the container: 
kubectl exec -it shared-volume-hostpath -c alpine1 -- touch /alpine1/someFile.txt
``````
