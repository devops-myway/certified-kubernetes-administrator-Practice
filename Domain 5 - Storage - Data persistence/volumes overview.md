
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
vi vol-pod.yaml
--
apiVersion: v1
kind: Pod
metadata:
  name: vol-pod
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container    
    command: [    'sh', '-c', 'echo The Bench Container 1 is Running ; sleep 3600']    
    volumeMounts:
    - mountPath: /opt
      name: demo-volume
  volumes:
  - name: demo-volume
    emptyDir: {}

----
kubectl apply -f myVolumes-Pod.yaml
kubectl logs vol-pod
kubectl exec myvolumes-pod -it -- /bin/sh
--

ls -l /opt
cat << EOF > /opt/text.txt
impossible is nothing
EOF
cat /opt/text.txt
exit

---
kubectl delete -f vol-Pod.yaml --force --grace-period=0

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

###### Create Hostpath Volume
A hostPath volume mounts a file or directory from the host node's filesystem into your Pod

``````sh
mkdir /mnt/persistent-volume

echo persistent data >  /mnt/persistent-volume/persistent-file
``````
``````sh
vi vol-hp.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx
    name: test-container
    volumeMounts:
    - mountPath: /opt
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /opt
      type: Directory
----
kubectl exec -it pod/test-pd -- sh
cat << EOF > /opt/test.txt
> this is a test for hostpath
> EOF
cat /opt/test.txt
``````
``````sh


``````
