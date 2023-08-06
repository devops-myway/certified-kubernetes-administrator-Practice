https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
https://etcd.io/docs/v3.4/dev-guide/interacting_v3/
https://sysdig.com/blog/kubernetes-security-harden-kube-system/

###### ETCD Version
``````sh
kubectl get ns
kubectl get po -n kube-system
kubectl describe po/etcd -n kube-system
--
image: registry.k8s.io/etcd:3.5.7-0
``````
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

ETCDCTL_API=3 etcdctl snapshot restore <data-dir-location> --data-dir new_directory

sudo ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/backup.db --data-dir /var/lib/etcd-from-backup
``````
##### To a new localtion and next, update the /etc/kubernetes/manifests/etcd.yaml
``````sh
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd-backup

ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
snapshot restore /opt/snapshot-pre-boot.db

``````
###### the restored files are located at the new folder /var/lib/etcd-backup, so now configure etcd to use that directory:
The YAML file, is to change the hostPath for the volume called etcd-data from old directory (/var/lib/etcd) to the new directory (/var/lib/etcd-from-backup).
``````sh
cat /etc/kubernetes/manifests/etcd.yaml   # to change the default directory of the etcd data store to the updated one
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data

``````
###### kube-system security: Core components
kube-apiserver: The central communications hub of the cluster. Provides REST endpoints to interact with the other cluster entities and stores the distributed state in the etcd backend.

etcd: The database backend where the cluster configuration, state and related information persists.
One of the easiest and most effective whitelist that you can configure is the list of allowed processes
``````sh
kubectl get po -n kube-system
kubectl describe po/ -n kube-system
kubectl exec -it etcd --namespace=kube-system sh

``````
``````sh
kubectl exec -it kube-apiserver --namespace=kube-system sh

``````

###### What is the IP address of the External ETCD datastore used in cluster2
cat /etc/kubernetes/manifests/kube-apiserver.yaml
--etcd-servers=https://192.29.15.14:2379

cat /etc/kubernetes/manifests/etcd.yaml
--listen-client-urls=https://127.0.0.1:2379,https://192.29.15.21:2379

--advertise-client-urls=https://192.29.15.21:2379

ssh etcd_server or ssh 192.29.15.14

-----

