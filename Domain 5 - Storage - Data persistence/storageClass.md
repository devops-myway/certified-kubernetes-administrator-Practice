
https://kubernetes.io/docs/concepts/storage/storage-classes/
https://kubernetes-csi.github.io/docs/drivers.html
https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/
https://github.com/ondat/use-cases
https://docs.ondat.io/docs/operations/storageclasses/


##### Kubernetes StorageClass
- Distributed storage systems:
Distributed storage systems enable us to store data that can be made available clusterwide. Excellent
So this is another area where Kubernetes typically outsources the job to plugins, whether those are from cloud providers like Azure and AWS or they are components like NFS or Ceph configured on-prem.
External storage systems like these connect to Kubernetes by way of the Container Storage Interface (CSI). This provides a standard interface between different types of storage systems—including open and closed source solutions from various vendors—and the storage abstractions in the Kubernetes API.

where sc = StorageClass

##### Dynamic provisioning with Storage Classes

The StorageClass API resource sits over, manages, and dynamically creates PersistentVolumes.
Using a StorageClass is necessary to dynamically provision storage, and in general it is the preferable target for storage requests via PersistentVolumeClaims.
You will typically prefer to interact with StorageClasses (SCs) rather than individual PVs.

Each StorageClass contains the fields provisioner, parameters, and reclaimPolicy, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned.

- PROVISIONER:
Refers to the CSI plugin through which Kubernetes will be connecting dynamically generated PVs to whatever external storage system you are using. e.g AzureFile, AWSElasticBlockStore etc.
- reclaimPolicy:
PersistentVolumes that are dynamically created by a StorageClass will have the reclaim policy specified in the reclaimPolicy field of the class,which can be either Delete or Retain. If no reclaimPolicy is specified when a StorageClass object is created, it will default to Delete
- Parameters:
Storage Classes have parameters that describe volumes belonging to the storage class.
Different parameters may be accepted depending on the provisioner. For example, the value io1, for the parameter type, and the parameter iopsPerGB are specific to EBS.

##### Example 1
StorageClass, for example, enables dynamic creation of “fast-storage” volumes by a CSI volume plugin called “csi-driver.example.com”
``````sh
cat <<EOF | kubectl create --filename -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast-storage
provisioner: csi-driver.example.com
parameters:
  type: pd-ssd
  csi.storage.k8s.io/provisioner-secret-name: mysecret
  csi.storage.k8s.io/provisioner-secret-namespace: mynamespace
EOF
``````
###### PersistentVolumeClaim object creation
Dynamic provisioning is triggered by the creation of a PersistentVolumeClaim object. The following PersistentVolumeClaim, for example, triggers dynamic provisioning using the StorageClass above.
``````sh
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-request-for-storage
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-storage

``````
##### Attaching and Mounting PVC
You can reference a PersistentVolumeClaim that is bound to a CSI volume in any pod or pod template
``````sh
kind: Pod
apiVersion: v1
metadata:
  name: my-pod
spec:
  containers:
    - name: my-frontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: my-csi-volume
  volumes:
    - name: my-csi-volume
      persistentVolumeClaim:
        claimName: my-request-for-storage

``````
##### Example2: 
The following manifest creates a storage class "sc-local" which provisions SSD-like persistent disks as an example
``````sh
cat <<EOF | kubectl create --filename -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
 name: sc-local
provisioner: k8s.io/minikube-hostpath
parameters:
  type: pd-ssd
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: false
EOF
----
kubectl apply -f sc.yml
kubectl get sc
``````
##### create PVCs from the SC
``````sh
cat <<EOF | kubectl create --filename -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: tododb-claim
spec:
 storageClassName: sc-local
 accessModes:
   - ReadWriteOnce
 resources:
   requests:
     storage: 1Gi
EOF
-----
kubectl get pv
``````
##### deploy our MySQL server
``````sh
cat <<EOF | kubectl create --filename -
apiVersion: apps/v1
kind: Deployment
metadata:
 labels:
   app: todo-mysql
 name: todo-mysql
spec:
 replicas: 1
 selector:
   matchLabels:
     app: todo-mysql
 template:
   metadata:
     labels:
       app: todo-mysql
   spec:
     volumes:
       - name: todo-volume
         persistentVolumeClaim:
           claimName: tododb-claim
     containers:
     - image: ericgregory/todo-mysql
       name: todo-mysql
       ports:
       - containerPort: 80
       volumeMounts:
       - mountPath: "/var/lib/mysql"
         name: todo-volume
EOF
-----
 kubectl exec -it todo-mysql-5798c74978-dksct -- /bin/bash

``````
Go ahead and open MySQL:
``````sh
bash4.4# mysql -u root -p

mysql> USE todo_db;
mysql> CREATE TABLE IF NOT EXISTS test123 ( \
    -> id INTEGER PRIMARY KEY);
kubectl delete pod todo-mysql-5798c74978-dksct
``````
