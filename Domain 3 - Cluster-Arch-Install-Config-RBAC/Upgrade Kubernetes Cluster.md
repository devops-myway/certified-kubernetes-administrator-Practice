##### Documentation Link

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

##### Ugrading ControlePlane Nodes 1
```sh
kubectl get nodes
ssh controlplane   # ssh into the node
sudo -i     # for elevated privilleges
```

##### Drain the node
Prepare the node for maintenance by marking it unschedulable and evicting the workloads:
```sh
kubectl drain <node-to-drain> --ignore-daemonsets

```
##### step2: Upgrade Kubeadm, kubelet and kubectl
```sh
apt update
apt-cache madison kubeadm
apt-get update && apt-get install -y kubeadm=1.27.x-00 kubelet=1.27.x-00 kubectl=1.27.x-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

##### step3: Uncordon the node
Bring the node back online by marking it schedulable:
```sh
kubectl uncordon <node-to-uncordon>

```
 ##### Upgrading worker nodes 2

```sh
ssh node01
sudo -i 
kubectl drain <node-to-drain> --ignore-daemonsets  # Drain the node for manitanance

```
###### Upgrade kubeamd, kubelet and kubectl
```sh
apt-get update && apt-get install -y kubeadm=1.27.x-00 kubelet=1.27.x-00 kubectl=1.27.x-00 
sudo systemctl daemon-reload
sudo systemctl restart kubelet

```
##### Uncordon the node
```sh
kubectl uncordon <node-to-uncordon>
```







