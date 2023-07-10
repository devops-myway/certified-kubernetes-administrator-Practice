https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

###### Working with ETCDCTL
We need to pass the following three pieces of information to etcdctl to take an etcd snapshot.

- etcd endpoint (–endpoints)
- ca certificate (–cacert)
- server certificate (–cert)
- server key (–key)

install the etcd client utility if not done
``````sh
sudo apt install etcd-client
etcdctl -h
--
sudo ls -l /etc/
sudo ls -l /etc/kubernetes/manifests
sudo cat /etc/kubernetes/manifests/etcd.yaml
---
kubectl get po -n kube-system
kubectl describe pod etcd-master-node -n kube-system

--- # grab the important keys
- --advertise-client-urls=https://172.31.42.63:2379  # endpoint
- --cert-file=/etc/kubernetes/pki/etcd/server.crt
- --key-file=/etc/kubernetes/pki/etcd/server.key
- --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

``````

#### Backing up an etcd cluster
All Kubernetes objects are stored on etcd. Periodically backing up the etcd cluster data is important to recover Kubernetes clusters under disaster scenarios, such as losing all control plane nodes. 

Backing up an etcd cluster can be accomplished in two ways: etcd built-in snapshot and volume snapshot.

##### Volume snapshot
If etcd is running on a storage volume that supports backup, such as Amazon Elastic Block Store, back up etcd data by taking a snapshot of the storage volume.
Execute the command to perform a backup. You can replace /opt/backup/etcd.db with the location and name of your choice.

``````sh
# create a simple text file of your choice in the root directory
ls -l /
sudo mkdir -p /opt/backup
touch backup.db
sudo mv backup.db /opt/backup
ls -l /opt/backup
file location: /opt/backup/backup.db
---

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>

--
sudo ETCDCTL_API=3 etcdctl --endpoints=https://172.31.42.63:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key\
  snapshot save /opt/backup/backup.db
-- # result
Snapshot saved at /opt/backup/backup.db

``````
##### Verify the Snapshot
``````sh
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/backup/backup.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| bedc4925 |   195026 |       1050 |     2.8 MB |
+----------+----------+------------+------------+

``````
##### Kubernetes etcd Restore Using Snapshot Backup
``````sh

ETCDCTL_API=3 etcdctl snapshot restore --data-dir <data-dir-location> snapshotdb

sudo ETCDCTL_API=3 etcdctl snapshot restore --data-dir /opt/backup backup.db
``````

