##### How Kubernetes Persistent Volume and Persistent Volume Claim works
Let us understand in a step by step instruction how a Persistent Volume can be configured and used:

- The cluster administrator will sets up the underlying storage and then registers it in Kubernetes by creating a PersistentVolume resource through the Kubernetes API server. In this image we have a NFS backed solution.
- When creating the PersistentVolume, the admin specifies its size and the access modes it supports.
- When a cluster user needs to use persistent storage in one of their pods, they first create a PersistentVolumeClaim manifest, specifying the minimum size and the access mode they require.
- The user then submits the PersistentVolumeClaim manifest to the Kubernetes API server, and Kubernetes finds the appropriate PersistentVolume and binds the volume to the claim.
- The PersistentVolumeClaim can then be used as one of the volumes inside a pod
- Other users cannot use the same PersistentVolume until it has been released by deleting the bound PersistentVolumeClaim.

##### 1.1 Capacity
The persistent volume has a capacity of 1 gibibytes (a single gibibyte (GiB) is 2 to the power of 30 bytes).

kilobyte	1000	K	kibibyte	1024	Ki
megabyte	1000*2	M	mebibyte	1024*2	Mi
gigabyte	1000*3	G	gibibyte	1024*3	Gi

##### 1.2 Volume Mode
Here you can specify whether you want a filesystem ("Filesystem") or raw storage ("Block"). If you don't specify volume mode, then the default is "Filesystem"

#### 1.3 Access Modes
There are three access modes:

RWO — ReadWriteOnce—Only a single node can mount the volume for reading and writing.
ROX — ReadOnlyMany—Multiple nodes can mount the volume for reading.
RWX — ReadWriteMany—Multiple nodes can mount the volume for both reading and writing.

##### 1.4 Reclaim Policy
The reclaim policy determines what happens when a persistent volume claim is deleted. There are three different policies:

Retain: The volume will need to be reclaimed manually
Delete: The associated storage asset, such as AWS EBS, GCE PD, Azure disk, or OpenStack Cinder volume, is deleted
Recycle: Delete content only (rm -rf /volume/*)

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