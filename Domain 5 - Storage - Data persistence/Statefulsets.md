- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

##### What is a StatefulSet
The StatefulSet API resource is an abstraction for managing stateful applications on Kubernetes. Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

###### Limitations:
- The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner based on the requested storage class, or pre-provisioned by an admin.
- StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods. You are responsible for creating this Service.
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.

##### Create the PV from requested storageClass
``````sh
cat << EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
 name: sc-local
provisioner: k8s.io/minikube-hostpath
parameters:
 {}
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: false
EOF
``````
##### Create the Secret
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
 name: mysqlpwd
data:
 password: b2N0b2JlcmZlc3Q=
EOF
``````
##### Create the Headless Service
A Headless Service, named todo-mysql, is used to control the network domain.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
 name: todo-mysql
 labels:
   app: todo-mysql
spec:
 type: ClusterIP
 selector:
   app: todo-mysql
 ports:
   - port: 3306
     protocol: TCP
 clusterIP: "None"
EOF
``````
##### Deploy the StatefulSet
The StatefulSet, named todo-mysql, has a Spec that indicates that 1 replicas of the mysql:8 container will be launched in unique Pods.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: StatefulSet
metadata:
 name: todo-mysql
spec:
 selector:
   matchLabels:
     app: todo-mysql
 serviceName: todo-mysql
 replicas: 1
 template:
   metadata:
     labels:
       app: todo-mysql
   spec:
     terminationGracePeriodSeconds: 10
     containers:
     - name: todo-mysql
       image: mysql:8
       ports:
       - containerPort: 3306
         name: todo-mysql
       env:
       - name: MYSQL_ROOT_PASSWORD
         valueFrom:
           secretKeyRef:
             name: mysqlpwd
             key: password
       - name: MYSQL_DATABASE
         value: "todo_db"
       volumeMounts:
       - mountPath: /var/lib/mysql
         name: todo-volume
 volumeClaimTemplates:
 - metadata:
     name: todo-volume
   spec:
     storageClassName: sc-local
     accessModes: [ "ReadWriteOnce" ]
     resources:
       requests:
         storage: 1Gi
EOF
``````

##### Getting a StatefulSet up and running
The StatefulSet maintains a primary pod at ordinal number 0, using a predictable name that applications can connect to reliably if needed
Since our pod is named using an ordinal, we can jump right into it without searching for its name
Now that we’re inside the container, start MySQL and Enter our root password: octoberfest
``````sh
kubectl get pods
kubectl exec -it po/todo-mysql-0 -- sh
mysql -u root -p  # octoberfest
----
mysql> USE todo_db;
mysql> CREATE TABLE IF NOT EXISTS Todo (task_id int NOT NULL AUTO_INCREMENT, task VARCHAR(255) NOT NULL, status VARCHAR(255), PRIMARY KEY (task_id));
mysql> SHOW TABLES;
mysql> INSERT INTO Todo (task, status) VALUES ('Hello','ongoing');
mysql> SELECT * FROM Todo;
exit
``````

##### Replicate StatefulSet
if we create a replica, do you think our data will persist across replicas of the pod?
``````sh
kubectl scale statefulsets todo-mysql --replicas=2
---
kubectl exec -it todo-mysql-1 -- sh
mysql -u root -p
Enter password: octoberfest

mysql> USE todo_db;
mysql> exit
-- # test the headless service for replication
kubectl run testpod --image=ubuntu -- sh -c 'sleep 3600'
kubectl exec -it testpod -- sh
kubectl get svc
kubectl scale sts/todo-mysql --replicas=4
---
apt update && upgrade
apt install dnsutils

nslookup todo-mysql  # headless service 

``````

##### Force Delete StatefulSet Pods
 The StatefulSet controller is responsible for creating, scaling and deleting members of the StatefulSet. It tries to ensure that the specified number of Pods from ordinal 0 through N-1 are alive and ready. StatefulSet ensures that, at any time, there is at most one Pod with a given identity running in a cluster.

Delete Pods:
 You can perform a graceful pod deletion with the following command:
 If you want to delete a Pod forcibly using kubectl version >= 1.5, do the following:
``````sh
kubectl scale sts/todo-mysql --replicas=0

kubectl delete pods <pod> --grace-period=0 --force
``````


##### Note: volumeClaimTemplates
The spec.volumeClaimTemplates : The volumeClaimTemplates will provide stable storage using PersistentVolumes provisioned by a PersistentVolume Provisioner.
the serviceName: A Headless Service, named todo-mysql, is used to control the network domain.
``````sh
serviceName: "todo-mysql"  # from the headless-service name
----
 volumeClaimTemplates:    # from the storage class dynamic provisioner
 - metadata:
     name: todo-volume
   spec:
     storageClassName: sc-local
     accessModes: [ "ReadWriteOnce" ]
     resources:
       requests:
         storage: 1Gi
``````
