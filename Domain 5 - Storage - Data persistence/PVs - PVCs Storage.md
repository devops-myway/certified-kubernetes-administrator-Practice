https://kubernetes.io/docs/concepts/storage/persistent-volumes/


##### The pyramid of persistent data
Stateless applications — apps that don’t store or modify changeable data—this is perfect e.g deployments, etc.
Containers are typically presumed to be ephemeral and immutable, which is to say that we expect any given container to be short-lived, replaceable, and unchanging in its contents.

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

##### Types of Persistent Volumes
PersistentVolume types are implemented as plugins. Kubernetes currently supports the following plugins:

cephfs - CephFS volume
csi - Container Storage Interface (CSI)
fc - Fibre Channel (FC) storage
hostPath - HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead)
iscsi - iSCSI (SCSI over IP) storage
local - local storage devices mounted on nodes.
nfs - Network File System (NFS) storage
rbd - Rados Block Device (RBD) volume

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

##### Binding
PersistentVolumeClaim binds are exclusive, regardless of how they were bound. A PVC to PV binding is a one-to-one mapping, using a ClaimRef which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim.

##### Class
A PV can have a class, which is specified by setting the storageClassName attribute to the name of a StorageClass. A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class.

##### 1. Create Persistent Volume

``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-app
  labels:
    app: my-app
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
EOF
--
kubectl get pv
``````
##### 2. Claiming a PersistentVolume by creating a PersistentVolumeClaim

``````sh
cat << EOF | kubectl apply -f -
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
EOF
--

kubectl get pvc
kubectl describe pvc nfs-share-pvc
``````
##### 3. Using a PersistentVolumeClaim in a pod
Here I am creating an nginx container to use the PV storage:
``````sh
cat << EOF | kubectl apply -f -
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
       name: "pv-port"
    volumeMounts:
    - name: demo-pvc
      mountPath: /var/lib
  volumes:
  - name: demo-pvc
    persistentVolumeClaim:
      claimName: nfs-share-pvc
EOF
--
kubectl get pods pod-nfs-share
kubectl exec -it pod-nfs-share -- df -h /var/lib
kubectl exec -it pod-nfs-share -- ls -l /var/lib
``````

