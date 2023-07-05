
https://kubernetes.io/docs/concepts/storage/volumes/

##### Kubernetes Volumes

The containers in a Kubernetes pod can access a data directory called a Kubernetes volume.
In Kubernetes, there are various volume types such as:

- Persistent Volumes
- EmptyDir Volumes
- Ephemeral Volumes
- Kubernetes Volumes configMap
- Kubernetes hostPath Volumes

###### emptyDir

emptyDir are volumes that get created empty when a Pod is created. While a Pod is running its emptyDir exists. If a container in a Pod crashes the emptyDir content is unaffected. Deleting a Pod deletes all its emptyDirs.

##### Basic emptyDir Example

``````sh
vi myVolumes-Pod.yaml
--
apiVersion: v1
kind: Pod
metadata:
  name: myvolumes-pod
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container    
    command: [    'sh', '-c', 'echo The Bench Container 1 is Running ; sleep 3600']    
    volumeMounts:
    - mountPath: /demo
      name: demo-volume
  volumes:
  - name: demo-volume
    emptyDir: {}

----
kubectl apply -f myVolumes-Pod.yaml
kubectl exec myvolumes-pod -it -- /bin/sh
--
pwd
ls
ls demo/
echo test > demo/textfile
ls demo/
cat demo/textfile
exit

---
kubectl delete -f myVolumes-Pod.yaml --force --grace-period=0

``````

emptyDir volume named demo-volume. The {} at the end means we do not supply any further requirements for the emptyDir .
This emptyDir spec makes it available to all containers in the Pod. Our example mounts the emptyDir at the mountPath: /demo
``````sh
volumeMounts:
    - mountPath: /demo
      name: demo-volume
---      
  volumes:
  - name: demo-volume
    emptyDir: {}
``````
##### Pod with 3 Containers Sharing emptyDir Use

``````sh
vi myVolumes-Pod.yaml
------
apiVersion: v1
kind: Pod
metadata:
  name: myvolumes-pod
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container-1    
    command: ['sh', '-c', 'echo The Bench Container 1 is Running ; sleep 3600']    
    volumeMounts:
    - mountPath: /demo1
      name: demo-volume

  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container-2    
    command: ['sh', '-c', 'echo The Bench Container 2 is Running ; sleep 3600']    
    volumeMounts:
    - mountPath: /demo2
      name: demo-volume

  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container-3    
    command: ['sh', '-c', 'echo The Bench Container 3 is Running ; sleep 3600']    
    volumeMounts:
    - mountPath: /demo3
      name: demo-volume

  volumes:
  - name: demo-volume
    emptyDir: {}

---
kubectl apply -f myVolumes-Pod.yaml
kubectl exec myvolumes-pod -c myvolumes-container-1 -it -- /bin/sh
 --
 ls
 echo test1 > demo1/textfile1
 exit

 --
 kubectl exec myvolumes-pod -c myvolumes-container-2 -it -- /bin/sh
 ls
 echo test2 > demo2/textfile2
 exit
 ----
 kubectl exec myvolumes-pod -c myvolumes-container-3 -it -- /bin/sh
 ls
 ls demo3/
 ls demo3/
 cat demo3/textfile1
 cat demo3/textfile2
 cat demo3/textfile3
 exit
``````
##### emptyDir Created in RAM

``````sh
vi myVolumes-Pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myvolumes-pod
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container

    command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']

    volumeMounts:
    - mountPath: /demo
      name: demo-volume
  volumes:
  - name: demo-volume
    emptyDir:
      medium: Memory  
---
kubectl apply -f myVolumes-Pod.yaml
kubectl exec myvolumes-pod -it -- /bin/sh

``````
###### PersistentVolume and PersistentVolumeClaim
signed in to the Kubernetes node where you want to make available some of its disk space.

Create a directory : /mnt/persistent-volume

``````sh
mkdir /mnt/persistent-volume

echo persistent data >  /mnt/persistent-volume/persistent-file
``````
``````sh
vi myPersistent-Volume.yaml

kind: PersistentVolume
apiVersion: v1
metadata:
  name: my-persistent-volume
  labels:
    type: local
spec:
  storageClassName: pv-demo 
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/persistent-volume"
``````
``````sh
kubectl apply -f myPersistent-Volume.yaml
kubectl get pv  my-persistent-volume

``````
###### Create a PersistentVolumeClaim

We claim usage of 10Mi of this PersistentVolume via a PersistentVolumeClaim 
``````sh
vi myPersistent-VolumeClaim.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-persistent-volumeclaim
spec:
  storageClassName: pv-demo 
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
``````
``````sh
kubectl apply -f myPersistent-VolumeClaim.yaml
kubectl get pvc my-persistent-volumeclaim

``````
##### Create a Pod that refers to your PersistentVolumeClaim
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: myvolumes-pod
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container

    command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']

    volumeMounts:
      - mountPath: "/my-pv-path"
        name: my-persistent-volumeclaim-name

  volumes:
    - name: my-persistent-volumeclaim-name
      persistentVolumeClaim:
       claimName: my-persistent-volumeclaim 

---
kubectl apply -f myVolumes-Pod.yaml
kubectl exec myvolumes-pod -i -t -- /bin/sh
ls
ls /my-pv-path/
cat /my-pv-path/persistent-file
echo more data >> /my-pv-path/persistent-file
cat /my-pv-path/persistent-file
exit

``````
Run on the node:
``````sh
cat   /mnt/persistent-volume/persistent-file
kubectl delete -f myVolumes-Pod.yaml --force --grace-period=0

``````
