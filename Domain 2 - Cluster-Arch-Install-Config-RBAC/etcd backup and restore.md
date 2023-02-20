##### Documentation Link

-  https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

-----------------------------------
##### Here is what you should know about etcd backup.

etcd has a built-in snapshot mechanism.
etcdctl is the command line utility that interacts with etcd for snapshots.

The snapshot file contains all the Kubernetes states and critical information
Backing up an etcd cluster can be accomplished in two ways:
etcd built-in snapshot and volume snapshot


Step 1: Log in to the control plane.

Step 2: If you don’t have etcdctl in your cluster control plane, install it using the following command.

##### Install etcd Using apt-get:
-------------------------
```sh
sudo apt-get update

sudo apt-get -y install etcd or sudo apt install etcd-client

# We need to pass the following three pieces of information to etcdctl to take an etcd snapshot.

etcd endpoint (–endpoints)
ca certificate (–cacert)
server certificate (–cert)
server key (–key)
You can get the above details in two ways.

From the etcd static pod manifest file located at /etc/kubernetes/manifests/etcd.yaml the location.

cat  /etc/kubernetes/manifests/etcd.yaml

or

# where trusted-ca-file, cert-file and key-file can be obtained from the description of the etcd Pod

kubectl get po -n kube-system
kubectl describe pod etcd-master-node -n kube-system

```

##### Using Volume snapshot instead of the built-in snap shot
-----------------------------------------------------
If etcd is running on a storage volume that supports backup, such as Amazon Elastic Block Store, back up etcd data by taking a snapshot of the storage volume

Snapshot using etcdctl options------------
```sh

ETCDCTL_API=3 etcdctl -h 

# For example, you can take a snapshot by specifying the endpoint, certificates etc as shown below:

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>
 
typical example:

ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd.db
 
 On a successful execution you will get a “Snapshot saved at /opt/backup/etcd.db”
 
 Verify the snapshot:
 
 ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshotdb
 
or

ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/backup/etcd.db

```

##### Restoring an etcd cluster:
-------------------
A restore operation is employed to recover the data of a failed cluster.
Before starting the restore operation, a snapshot file must be present. It can either be a snapshot file from a previous backup operation, or from a remaining data directory

```sh
ETCDCTL_API=3 etcdctl --endpoints 10.2.0.9:2379 snapshot restore snapshotdb

example:
ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd.db

If you want to use a specific data directory for the restore, you can add the location using the --data-dir

ETCDCTL_API=3 etcdctl snapshot restore --data-dir <data-dir-location> snapshotdb

example:

ETCDCTL_API=3 etcdctl --data-dir /opt/etcd snapshot restore /opt/backup/etcd.db

```
