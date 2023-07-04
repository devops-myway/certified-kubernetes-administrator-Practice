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
[root@controller ~]# cat persistent-volume.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-share-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
   - ReadWriteMany
   - ReadOnlyMany
  persistentVolumeReclaimPolicy: Recycle
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs_share
    server: 192.168.43.48

--
kubectl create -f persistent-volume.yml
kubectl get pv
``````
##### 2. Claiming a PersistentVolume by creating a PersistentVolumeClaim

``````sh
[root@controller ~]# cat persistent-volume-claim.yml
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
kubectl create -f persistent-volume-claim.yml

kubectl get pvc

kubectl describe pvc nfs-share-pvc
``````
##### 3. Using a PersistentVolumeClaim in a pod
Here I am creating an nginx container to use the PV storage:
Nobody else can claim the same volume until you release it. To use it inside a pod, you need to reference the PersistentVolumeClaim by name inside the pod’s volume
``````sh
[root@controller ~]# cat nfs-share-pod.yml
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
kubectl create -f nfs-share-pod.yml

kubectl get pods pod-nfs-share

kubectl exec -it pod-nfs-share -- df -h /var/www

kubectl exec -it pod-nfs-share -- ls -l /var/www
``````

##### 5.1 Create StorageClass
Storage classes let an administrator configure your cluster with custom persistent storage .
A storage class has a name in the metadata (it must be specified in the annotation to claim), a provisioner, and parameters.

As I don't have access to cloud environment, I will define a storage class for using local volumes. Local volumes are similar to HostPath, but they persist across pod restarts and node restarts.

``````sh
[root@controller ~]# cat storage-class.yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
--
kubectl create -f storage-class.yml
kubectl get sc
``````
##### Creating a local persistent volume


``````sh
[root@controller ~]# cat local-pv-sc.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
   - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /tmp/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker-2.example.com
 ---
kubectl get nodes

kubectl create -f local-pv-sc.yml

kubectl get pv
kubectl describe pv local-pv
``````
##### Making Persistent Volume Claims

``````sh
[root@controller ~]# cat local-pvc.yml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-storage-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 1Gi
--
kubectl create -f local-pvc.yml

kubectl get pvc

kubectl describe pvc local-storage-claim
``````
##### Create a Pod


``````sh
[root@controller ~]# cat local-pv-pod.yml
kind: Pod
apiVersion: v1
metadata:
  name: local-pod
spec:
  containers:
    - name: local-pod
      image: nginx
      ports:
      - containerPort: 80
        name: "httpd-server"
      volumeMounts:
      - mountPath: "/mnt/tmp"
        name: persistent-volume
  volumes:
    - name: persistent-volume
      persistentVolumeClaim:
        claimName: local-storage-claim
---
kubectl create -f local-pv-pod.yml
kubectl get pods local-pod
kubectl get pvc
``````
