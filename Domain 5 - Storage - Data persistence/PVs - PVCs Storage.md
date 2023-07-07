https://kubernetes.io/docs/concepts/storage/persistent-volumes/


##### The pyramid of persistent data
For stateless applications — apps that don’t store or modify changeable data—this is perfect e.g deployments, etc.
Containers are typically presumed to be ephemeral and immutable, which is to say that we expect any given container to be short-lived, replaceable, and unchanging in its contents.
But most apps are stateful: they change into different states as users or clients create, update, or delete data that may persist over time.

Top fo the pyramid of data persistence and reliability:
- Volumes mounted to Pods: 
Pod-mounted volumes are most suited to temporary caches and other short-term uses that you don’t expect to last any longer than a given Pod—for example, this technique can be used to mount a Secret to a Pod.
- Volumes mounted to Nodes:
If you need data to persist beyond the lifespan of a particular Pod, you might consider using a volume mounted at the Node level. Now persistent data can be made available to any Pod running on that Node, but the limitation should be immediately clear: Only Pods on that Node can access the data.

#### Lifecycle of a volume and claim
PersistentVolume (PV):
is a system resource for storage in the same way that a node is a system resource for compute.
PersistentVolumes are administrator-facing objects: system resources to be defined for others’ use.

Provisioning PersistentVolumes: There are two ways PVs may be provisioned: statically or dynamically
- static provisioning PVs: A cluster administrator creates a number of PVs. They carry the details of the real storage, which is available for use by cluster users. They exist in the Kubernetes API and are available for consumption.
- dynamic provisioning PVs: When none of the static PVs the administrator created match a user's PersistentVolumeClaim, the cluster may try to dynamically provision a volume specially for the PVC.
This provisioning is based on StorageClasses: the PVC must request a storage class and the administrator must have created and configured that class for dynamic provisioning to occur. Claims that request the class "" effectively disable dynamic provisioning for themselves.

PersistentVolumeClaim (PVC):
A request to provision available storage resources. This is essentially your app's voucher for storage utilization.


##### 1.1 Capacity
The persistent volume has a capacity of 1 gibibytes (a single gibibyte (GiB) is 2 to the power of 30 bytes).

kilobyte	1000	K	kibibyte	1024	Ki
megabyte	1000*2	M	mebibyte	1024*2	Mi
gigabyte	1000*3	G	gibibyte	1024*3	Gi

##### 1.2 Volume Mode
Here you can specify whether you want a filesystem ("Filesystem") or raw storage ("Block"). If you don't specify volume mode, then the default is "Filesystem"

#### 1.3 Access Modes
There are three access modes:

- RWO — ReadWriteOnce—Only: a single node can mount the volume for reading and writing.
- ROX — ReadOnlyMany—Multiple: nodes can mount the volume for reading.
- RWX — ReadWriteMany—Multiple: nodes can mount the volume for both reading and writing.

##### 1.4 Reclaim Policy
The reclaim policy determines what happens when a persistent volume claim is deleted. There are three different policies:
- Retain: The volume will need to be reclaimed manually
- Delete: The associated storage asset, such as AWS EBS, GCE PD, Azure disk, or OpenStack Cinder volume, is deleted
- Recycle: Delete content only (rm -rf /volume/*)

##### 1. Create Persistent Volume

``````sh
vi emp-volume.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: busybox:1.28
        command: ['sh', '-c'] 
        args: 
        - while true; do 
            echo "$(date) INFO some app data" >> /var/log/myapp.log;
            sleep 5;
          done
          volumeMounts:
        - name: log
          mountPath: /var/log   
      - name: log-sidecar
        image: busybox:1.28
        command: ['sh', '-c'] 
        args:
        - tail -f /var/log/myapp.log 
        volumeMounts:
        - name: log
          mountPath: /var/log
        volumes: 
      - name: log
        emptyDir: {}
---
kubectl apply -f emp-volume.yaml
kubectl exec -it my-app-5d74b5cc88-7n474 -c my-app -- sh -c 'cat /var/log/myapp.log'
kubectl exec -it my-app-5d74b5cc88-7n474 -c log-sidecar -- sh -c 'cat /var/log/myapp.log'


``````
##### 2. Claiming a PersistentVolume by creating a PersistentVolumeClaim

``````sh
vi persistent-volume-claim.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nfs-share-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
  storageClassName: ""

--
kubectl apply -f persistent-volume-claim.yaml
kubectl get pvc
kubectl describe pvc nfs-share-pvc
``````
##### 3. Using a PersistentVolumeClaim in a pod
Here I am creating an nginx container to use the PV storage:
``````sh
vi nfs-share-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-nfs-share
spec:
  containers:
  - image: nginx
    name: pv-container
    ports:
     - containerPort: 80
       name: "http-server"
    volumeMounts:
    - name: data
      mountPath: /var/www
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: nfs-share-pvc
--
kubectl apply -f nfs-share-pod.yml
kubectl get pods pod-nfs-share
kubectl exec -it pod-nfs-share -- df -h /var/www
kubectl exec -it pod-nfs-share -- ls -l /var/www
``````

