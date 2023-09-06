
###### Questions - Role, RoleBinding ClsuterRole & RoleBinding
Context -
You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace.
- Task
Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types:
✑ Deployment
✑ Stateful Set
✑ DaemonSet
Create a new ServiceAccount named cicd-token in the existing namespace app-team1.
Bind the new ClusterRole deployment-clusterrole to the new ServiceAccount cicd-token, limited to the namespace app-team1.

``````sh
kubectl create ns app-team1
kubectl create sa cicd-token -n app-team1

kubectl api-resources
kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployments,statefulsets,daemonsets
kubectl create rolebinding deployment-role-binding --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token --namespace=app-team1
kubectl auth can-i create deployments --as=system:serviceaccount:app-team1:cicd-token -n app-team1
``````
##### Question2 - Draining of Node - Cluster Configuration
https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-nodes-and-cluster

- Task
Set the node named ek8s-node-0 as unavailable and reschedule all the pods running on it.

``````sh
kubectl get nodes
kubectl drain --ignore-daemonsets ek8s-node-0
kubectl create deploy test-dep --image=nginx --replicas=3  # test pod to confirm the schedule

kubectl uncordon ek8s-node-0   # the retest for scheduling

``````
##### Q3 - Upgrade the Cluster - Cluster Configuration

- Task 1-
Given an existing Kubernetes cluster running version 1.20.0, upgrade all of the Kubernetes control plane and node components on the master node only to version 1.20.1.
Be sure to drain the master node before upgrading it and uncordon it after the upgrade.
You are also expected to upgrade kubelet and kubectl on the master node.

- Task 2-
Given an existing Kubernetes cluster running version 1.22.1, upgrade all of the Kubernetes control plane and node components on the master node only to version 1.22.2.
Be sure to drain the master node before upgrading it and uncordon it after the upgrade.
You are also expected to upgrade kubelet and kubectl on the master node.
Note Below:
Do not upgrade, the worker nodes, etcd, the container manager, the CNI plugin, the DNS service and any other add-ons.

``````sh
ssh controlplane
sudo -i

apt update
apt-cache madison kubeadm
kubectl drain controlplane --ignore-daemonsets

apt-get update && apt-get install -y kubeadm=1.27.x-00 kubelet=1.27.x-00 kubectl=1.27.x-00
sudo kubeadm upgrade apply v1.27.x --etcd-upgrade=false
apt-mark hold kubelet kubectl kubeadm
sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon <node-to-uncordon>
kubectl get nodes

--
ssh controlplane
sudo -i
kubectl drain k8s-master --ignore-daemonsets
apt-get install kubeadm=1.20.1-00 kubelet=1.20.1-00 kubectl=1.20.1-00
kubeadm upgrade apply 1.20.1 --etcd-upgrade=false
systemctl daemon-reload
systemctl restart kubelet
kubectl uncordon k8s-master
kubectl get nodes

``````
##### Q4 - Backup & Restore Cluster - Cluster Configuration
##### Question 1 Task -
First, create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /var/lib/backup/etcd-snapshot.db.
Note: The following TLS certificates/key are supplied for connecting to the server with etcdctl:
- CA certificate: /opt/KUIN00601/ca.crt
- Client certificate: /opt/KUIN00601/etcd-client.crt
- Client Key: /opt/KUIN00601/etcd-client.key
Next, restore an existing, previous snapshot located at /var/lib/backup/etcd-snapshot-previous.db.

##### Q2 Another Question: Backup & Restore
Create a snapshot of the etcd instance running at https://127.0.0.1:2379, saving the snapshot to the file path /srv/data/etcd-snapshot.db.
The following TLS certificates/key are supplied for connecting to the server with etcdctl:
- CA certificate: /opt/KUCM00302/ca.crt
- Client certificate: /opt/KUCM00302/etcd-client.crt
- Client key: /opt/KUCM00302/etcd-client.key
A. See the solution below.

``````sh
# Backup
vi bck1.sh
#! /bin/bash  -- add the script here
chmod u+x bck1.sh
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/opt/KUCM00302/ca.crt --cert=/opt/KUCM00302/etcd-client.crt --key=/opt/KUCM00302/etcd-client.key\
  snapshot save /srv/data/etcd-snapshot.db

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/opt/KUIN00601/ca.crt --cert=/opt/KUIN00601/etcd-client.crt --key=/opt/KUIN00601/etcd-client.key \
  snapshot save /var/lib/backup/etcd-snapshot.db

# Restore
ETCDCTL_API=3 etcdctl snapshot restore /var/lib/backup/etcd-snapshot-previous.db

# configure the etcd-data directory hostpath to the new directory
cat /etc/kubernetes/manifests/etcd.yaml
hostPath:
--- #configure the etcd-data

``````

###### Q5 - Cluster Configuration/Installation using Kubeadam

For this item, you will have to ssh to the nodes ik8s-master-0 and ik8s-node-0 and complete all tasks on these nodes. Ensure that you return to the base node (hostname: node-1) when you have completed this item.
Context
As an administrator of a small development team, you have been asked to set up a Kubernetes cluster to test the viability of a new application.
- Task
You must use kubeadm to perform this task. Any kubeadm invocations will require the use of the --ignore-preflight-errors=all option.
Configure the node ik8s-master-O as a master node.
Join the node ik8s-node-o to the cluster.
solution
You must use the kubeadm configuration file located at /etc/kubeadm.conf when initializing your cluster.
You may use any CNI plugin to complete this task, but if you don't have your favourite CNI plugin's manifest URL at hand, Calico is one popular option:
https://docs.projectcalico.org/v3.14/manifests/calico.yaml

Note: Docker is already installed on both nodes and apt has been configured so that you can install the required tools
``````sh
ssh ik8s-master-0
sudo -i
sudo swapoff -a
##### Install Container-Runtime

vi ipv.sh ------------
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
lsmod | grep br_netfilter
lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

##### Install Containerd

apt update
apt install containerd -y
##### Configure cgroup driver
mkdir -p /etc/containerd/
containerd config default > /etc/containerd/config.toml
sudo systemctl restart containerd

##### Installing kubeadm, kubelet and kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
mkdir -p /etc/apt/keyrings/
curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

#####  Initializing the Cluster
kubeadm init --pod-network-cidr 192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

or

export KUBECONFIG=/etc/kubernetes/admin.conf  # If you are the root user
export KUBECONFIG=/etc/kubeadm.conf    # using exam kubeadm configuration file

###### Installing a Pod network add-on 

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

##### Joining your nodes
ssh ik8s-node-0
sudo -i
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>

``````

##### Q6 - Network Policy Questions
- Task 1
Create a new NetworkPolicy named allow-port-from-namespace in the existing namespace fubar.
Ensure that the new NetworkPolicy allows Pods in namespace internal to connect to port 9000 of Pods in namespace fubar.
Further ensure that the new NetworkPolicy:
✑ does not allow access to Pods, which don't listen on port 9000
✑ does not allow access from Pods, which are not in namespace internal

- Task2

Create a new NetworkPolicy named allow-port-from-namespace in the existing namespace echo.
Ensure that the new NetworkPolicy allows Pods in namespace my-app to connect to port 9000 of Pods in namespace echo.
Further ensure that the new NetworkPolicy:
* does not allow access to Pods, which don't listen on port 9000
* does not allow access from Pods, which are not in namespace my-app

``````sh
kubectl create ns fubar
kubectl create ns internal
kubectl get ns --show-labels

vi net1.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: fubar
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: internal
      ports:
        - protocol: TCP
          port: 9000
---

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: echo
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: my-app   # any pod in a namespace with the label kubernetes.io/metadata.name=my-app
      ports:
        - protocol: TCP
          port: 9000

``````
##### 6 - Workloads and Services
- Task
Reconfigure the existing deployment front-end and add a port specification named http exposing port 80/tcp of the existing container nginx.
Create a new service named front-end-svc exposing the container port http.
Configure the new service to also expose the individual Pods via a NodePort on the nodes on which they are scheduled.

- Task 2-
Create and configure the service front-end-service so it's accessible through NodePort and routes to the existing pod named front-end.

``````sh
kubectl create deployment front-end --image=nginx --replicas=1 --dry-run=client -oyaml > dep1.yaml
kubectl create deployment front-end --image=nginx --replicas=1 --port=80 --dry-run=client -oyaml > dep2.yaml
kubectl apply -f dep1.yaml
vi dep2.yaml  # manually add the name: http
kubectl expose deployment front-end --target-port=http --name=front-end-svc --type=NodePort --dry-run=client -oyaml>svc1.yaml
kubectl apply -f svc1.yaml
kubectl describe svc front-end-svc

--- # Task 2
kubectl create deployment front-end --image=nginx --port= 80 --replicas=2 --dry-run=client -oyaml > dep2.yaml
kubectl expose deployment front-end -- port=80 --target-port=80 --name=front-end-service --type=NodePort --dry-run=client -oyaml>svc2.yaml

``````

##### Questions Services - Ingress

Task -
Create a new nginx Ingress resource as follows:
✑ Name: pong
✑ Namespace: ing-internal
✑ Exposing service hello on path /hello using service port 5678

``````sh
kubectl get ns
kubectl get svc, ing -n ing-internal

vi ing1.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pong
  namespace: ing-internal
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /hello
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 5678

kubectl apply -f ing1.yaml
curl -kL internal_ip/hello

``````

##### 7 - Workloads - Deployments Scaling
Task -
Scale the deployment presentation to 3 pods.

``````sh
kubectl scale deploy/presentation --replicas=3

``````

##### Qustions - Deployments
- Task 1-
Create a deployment as follows:
Name: nginx-app
Using container nginx with version 1.11.10-alpine
The deployment should contain 3 replicas
Next, deploy the application with new version 1.11.13-alpine, by performing a rolling update.
Finally, rollback that update to the previous version 1.11.10-alpine.

- Task 2-

Create a deployment spec file that will:
Launch 7 replicas of the nginx Image with the label app_runtime_stage=dev deployment name: kual00201
Save a copy of this spec file to /opt/KUAL00201/spec_deployment.yaml
(or /opt/KUAL00201/spec_deployment.json).
When you are done, clean up (delete) any new Kubernetes API object that you produced during this task

``````sh
vi dep1.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.11.10-alpine
        ports:
        - containerPort: 80

kubectl apply -f dep1.yaml
kubectl set image deployment/nginx-app nginx=nginx:1.11.13-alpine --record
kubectl rollout history deployment/nginx-app
kubectl rollout undo deployment/nginx-app --to-revision=1
---

vi /opt/KUAL00201/spec_deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: kual00201
  labels:
    app_runtime_stage: dev
spec:
  replicas: 7
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.11.10-alpine
        ports:
        - containerPort: 80

kubectl apply -f /opt/KUAL00201/spec_deployment.yaml
kubectl delete deploy/kual00201
``````


##### 8 - Scheduling - NodeSelectors

https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/
- Task -
Schedule a pod as follows:
✑ Name: nginx-kusc00401
✑ Image: nginx
✑ Node selector: disk=ssd

- Task 2-
Schedule a pod as follows:
Name: nginx-kusc00101
Image: nginx
Node selector: disk=ssd

``````sh
kubectl run nginx-kusc00401 --image=nginx --dry-run=client -oyaml>pod1.yaml
vi pod1.yaml

# Using nodeName overrules using nodeSelector or affinity and anti-affinity rules
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401
spec:
  nodeSelector:
    disk: ssd
  containers:
  - name: nginx
    image: nginx
--- # 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00101
spec:
  nodeSelector:
    disk: ssd
  containers:
  - name: nginx
    image: nginx
  
``````
##### Questions: Scheduling: Taints
https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-nodes-and-cluster

- Task 1-
Check to see how many nodes are ready (not including nodes tainted NoSchedule) and write the number to /opt/KUSC00402/kusc00402.txt.

- Task 2-
Check to see how many worker nodes are ready (not including nodes tainted NoSchedule) and write the number to /opt/KUCC00104/kucc00104.txt

``````sh
mkdir -p /opt/KUSC00402/
touch /opt/KUSC00402/kusc00402.txt
#### This is correct but the 2nd solution is the best
kubectl taint nodes foo dedicated=special-user:NoSchedule
kubectl get nodes -o='custom-columns=NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value,TaintEffect:.spec.taints[*].effect'
kubectl get nodes -o=custom-columns=NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value,TaintEffect:.spec.taints[*].effect| grep -c NoSchedule > to1.txt
cat to1.txt

#### The best solution to this problem
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True" | wc -l

JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True" | wc -l > /opt/KUSC00402/kusc00402.txt
 cat /opt/KUSC00402/kusc00402.txt
``````

##### Questions - Scheduler
- Task 1-
Schedule a Pod as follows:
✑ Name: kucc8
✑ App Containers: 2
✑ Container Name/Images:
- nginx
- consul

- Task 2-
Create a pod named kucc8 with a single app container for each of the following images running inside (there may be between 1 and 4 images specified):
nginx + redis + memcached

``````sh
kubectl run kucc8 --image=nginx --dry-run=client -oyaml> pod2.yaml

vi pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: kucc8
  name: kucc8
spec:
  containers:
  - image: nginx
    name: nginx
  - image: consul
    name: consul

kubectl apply -f pod2.yaml
kubectl get po

-----
vi pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: kucc8
  name: kucc8
spec:
  containers:
  - image: nginx
    name: nginx
  - image: redis
    name: redis
  - image: memcached
    name: memcached               

kubectl apply -f pod1.yaml
kubectl get po
``````

#### Questions - Storages - Volumes
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes
- Task 1 -
Create a persistent volume with name app-data, of capacity 2Gi and access mode ReadOnlyMany. The type of volume is hostPath and its location is /srv/app-data.

- Task 2- 
Create a pod as follows:
Name: non-persistent-redis container Image: redis
Volume with name: cache-control
Mount path: /data/redis
The pod should launch in the staging namespace and the volume must not be persistent.

``````sh
vi pv1.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-data
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: srv/app-data
    type: Directory

kubectl apply -f pv1.yaml
kubectl get pv

--- # Task2
vi pod3.yaml

apiVersion: v1
kind: Pod
metadata:
  name: non-persistent-redis
  namespace: staging
spec:
  containers:
  - image: redis
    name: redis
    volumeMounts:
    - mountPath: /data/redis
      name: cache-control
  volumes:
  - name: cache-control
    emptyDir: {}

kubectl apply -f pod3.yaml

``````

##### Questions - Storage - Volumes

Task -
Create a new PersistentVolumeClaim:
✑ Name: pv-volume
✑ Class: csi-hostpath-sc
✑ Capacity: 10Mi
Create a new Pod which mounts the PersistentVolumeClaim as a volume:
✑ Name: web-server
✑ Image: nginx
✑ Mount path: /usr/share/nginx/html
Configure the new Pod to have ReadWriteOnce access on the volume.
Finally, using kubectl edit or kubectl patch expand the PersistentVolumeClaim to a capacity of 70Mi and record that change.
``````sh
In this case, the pv has been dynamically provisioned with sc.

kubectl get sc,pv,pvc

# The sc should look like this below:
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-hostpath-sc
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate

vi pvc1.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Mi
  storageClassName: csi-hostpath-sc

vi pod-mount.yaml

apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: pv-volume

kubectl edit pvc/pv-volume
# manually expand the capacity to 70Mi


``````

##### Questions: Container Logs - Monitor
https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-and-finding-resources

- Task 1-
Monitor the logs of pod foo and:
✑ Extract log lines corresponding to error file-not-found
✑ Write them to /opt/KUTR00101/foo

- Task 2-
Monitor the logs of pod foo and:
Extract log lines corresponding to error unable-to-access-website
Write them to /opt/KULM00201/foo
``````sh
kubectl logs -f foo | grep "file-not-found" > /opt/KUTR00101/foo
cat /opt/KUTR00101/foo

kubectl logs foo | grep "unable-to-access-website" > /opt/KULM00201/foo
cat /opt/KULM00201/foo

``````

##### 12- SideCar Containers - Logging Architecture
https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-logging-agent

Context -
An existing Pod needs to be integrated into the Kubernetes built-in logging architecture (e.g. kubectl logs). Adding a streaming sidecar container is a good and common way to accomplish this requirement.
- Task -
Add a sidecar container named sidecar, using the busybox image, to the existing Pod big-corp-app. The new sidecar container has to run the following command:
/bin/sh -c "tail -n+1 -f /var/log/big-corp-app.log"
Use a Volume, mounted at /var/log, to make the log file big-corp-app.log available to the sidecar container.

``````sh
vi sd1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: big-corp-app
spec:
  volumes:
  - name: vol
  emptyDir: {}
  containers:
  - name: count
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/big-corp-app.log;
        i=$((i+1));
        sleep 1;
      done      
    volumeMounts:
    - name: vol
      mountPath: /var/log
  - name: sidecar
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -F /var/log/big-corp-app.log']
    volumeMounts:
    - name: vol
      mountPath: /var/log

---
kubectl apply -f sd1.yaml
kubectl logs big-corp-app -c sidecar  # to view the output of the logs
``````

###### Top Commands - Metrics - Sort-by
https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-and-finding-resources
- Task 1-
From the pod label name=overloaded-cpu, find pods running high CPU workloads and write the name of the pod consuming most CPU to the file /opt/
KUTR00401/KUTR00401.txt (which already exists).

- Task 2-
List all persistent volumes sorted by capacity, saving the full kubectl output to /opt/KUCC00102/volume_list. Use kubectl 's own functionality for sorting the output, and do not manipulate it any further.

- Task 3-

Create a file:
/opt/KUCC00302/kucc00302.txt that lists all pods that implement service baz in namespace development.
The format of the file should be one pod name per line.

- Task 4-
From the pod label name=cpu-utilizer, find pods running high CPU workloads and write the name of the pod consuming most CPU to the file /opt/KUTR00102/KUTR00102.txt (which already exists).

``````sh
cat /opt/KUTR00401/KUTR00401.txt 
kubectl top pod -l name=overloaded-cpu --sort-by=cpu > /opt/KUTR00401/KUTR00401.txt 
cat /opt/KUTR00401/KUTR00401.txt
--
# List PersistentVolumes sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage > /opt/KUCC00102/volume_list
cat /opt/KUCC00102/volume_list
--

kubectl get svc -n development
kubectl describe svc/baz -n development
kubectl get po -l  name=foo --sort-by=.metadata.name > /opt/KUCC00302/kucc00302.txt
--
cat /opt/KUTR00102/KUTR00102.txt
kubectl top pod -l name=overloaded-cpu --sort-by=cpu > /opt/KUTR00102/KUTR00102.txt
cat /opt/KUTR00102/KUTR00102.txt
``````

###### Questions - troubleshooting

- Task 1-
A Kubernetes worker node, named wk8s-node-0 is in state NotReady.
Investigate why this is the case, and perform any appropriate steps to bring the node to a Ready state, ensuring that any changes are made permanent.
ssh to the failed node using:
ssh wk8s-node-0
sudo -i

- Task 2-
Given a partially-functioning Kubernetes cluster, identify symptoms of failure on the cluster.
Determine the node, the failing service, and take actions to bring up the failed service and restore the health of the cluster. Ensure that any changes are made permanently.
You can ssh to the relevant I nodes (bk8s-master-0 or bk8s-node-0) using:
ssh <nodename>
You can assume elevated privileges on any node in the cluster with the following command: sudo –i

- Task 3-
Configure the kubelet systemd- managed service, on the node labelled with name=wk8s-node-1, to launch a pod containing a single container of Image httpd named webtool automatically. Any spec files required should be placed in the /etc/kubernetes/manifests directory on the node.
You can ssh to the appropriate node using:
ssh wk8s-node-1
You can assume elevated privileges on the node with the following command:
sudo –i

``````sh
kubectl get nodes
ssh wk8s-node-0
sudo -i
systemctl status kubelet
systemctl enable kubelet
systemctl restart kubelet
exit
kubectl get nodes 
--- # task 2

ssh bk8s-master-0 
sudo -i
vi /var/lib/kubelet/config.yaml
systemctl restart kubelet
systemctl enable kubelet
exit
kubectl get nodes

-- #Task3
sudo -i
vi /var/lib/kubelet/config.yaml
cd /etc/kubernetes/manifests
vi pod.yaml
systemctl restart kubelet
systemctl enable kubelet
exit
kubectl get po

``````

##### Questions - DaemonSets
Ensure a single instance of pod nginx is running on each node of the Kubernetes cluster where nginx also represents the Image name which has to be used. Do not override any taints currently in place. Use DaemonSet to complete this task and use ds-kusc00201 as DaemonSet name.

``````sh
kubectl get nodes

vi d1.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-kusc00201
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx

kubectl apply -f d1.yaml
kubectl get ds -n kube-system
kubectl describe ds -n kube-system
``````

##### Question - InitContainer
Perform the following tasks:
Add an init container to hungry-bear (which has been defined in spec file /opt/KUCC00108/pod-spec-KUCC00108.yaml)
The init container should create an empty file named /workdir/calm.txt
If /workdir/calm.txt is not detected, the pod should exit
Once the spec file has been updated with the init container definition, the pod should be created

``````sh
vi  /opt/KUCC00108/pod-spec-KUCC00108.yaml

aiVersion: v1
kind: Pod
metadata:
  name: hungry-bear
spec:
  volumes:
    - name: workdir
      emptyDir: {}
  containers:
  - name: checker
    image: alpine
    command: ['/bin/sh', '-c', "if [ -f /workdir/calm.txt]; then sleep 1000; else exit']; fi"]
    volumeMounts:
    - name: workdir
      mountPath: /workdir
  initContainers:
  - name: create
    image: alpine
    command: ['/bin/sh', '-c', "while true; do touch /workdir/calm.txt; sleep 3600; done"]
    volumeMounts:
    - name: workdir
      mountPath: /workdir

kubectl apply -f /opt/KUCC00108/pod-spec-KUCC00108.yaml
``````

##### Questions - Secrets
https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-with-data-from-multiple-secrets

- Task 1:
Create a Kubernetes secret as follows:
Name: super-secret password: bob
Create a pod named pod-secrets-via-file, using the redis Image, which mounts a secret named super-secret at /secrets.
Create a second pod named pod-secrets-via-env, using the redis Image, which exports password as CONFIDENTIAL


``````sh
kubectl create secret generic super-secret --from-literal=password=bob

vi pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secrets-via-file
spec:
  containers:
    - name: redis
      image: redis
      volumeMounts:
        - name: super-secret 
          mountPath: /super
          readOnly: true
  volumes:
    - name: super-secret 
      secret:
        secretName: super-secret
----
vi pod3.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-secrets-via-env
spec:
  containers:
    - name: redis
      image: redis
      env:
      - name: CONFIDENTIAL
        valueFrom:
          secretKeyRef:
            name: super-secret
            key: password


``````

##### QUESTIONS- DNS Service Discovery
- Task 1-
Create a deployment as follows:
Name: nginx-random
Exposed via a service nginx-random
Ensure that the service & pod are accessible via their respective DNS records
The container(s) within any pod(s) running as a part of this deployment should use the nginx Image
Next, use the utility nslookup to look up the DNS records of the service & pod and write the output to /opt/KUNW00601/service.dns and /opt/KUNW00601/pod.dns respectively.

``````sh
vi dep2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-random
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
kubectl apply -f dep2.yaml
#service
kubectl expose deploy/nginx-random --port=80 --name=nginx-random-svc --type=NordPort 
kubectl run testpod --image=ubuntu --command -- sh -c "sleep 3000"
kubectl exec -it testpod -- sh
apt update
apt install dnsutils

kubectl exec -it testpod -- sh -c "nslookup nginx-random-svc" > /opt/KUNW00601/service.dns
kubectl exec -it testpod -- sh -c "cat /etc/resolv.conf"
kubectl exec -it testpod -- sh -c "nslookup 10.244.0.198" > /opt/KUNW00601/pod.dns

``````

###### 19

``````sh


``````

#####  20
``````sh


``````
``````sh


``````
``````sh


``````
``````sh


``````
``````sh


``````