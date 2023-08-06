##### Documentation Link

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

##### Ugrading and updating a version kubernetes cluster: 

```sh
kubectl get nodes
```

##### step2: Determine which version to upgrade to on the control plane
```sh
ssh into the master node
sudo apt update
sudo apt-cache madison kubeadm
```
##### step3: Upgrading control plane nodes

My current version is 1.23.6. I will be upgrading it to one higher version, ie, 1.24.6-0
Upgrade kubeadm: unhold kubeadm and Install the required version

 ###### replace x in 1.25.x-00 with the latest patch version 
```sh
sudo apt-mark unhold kubeadm && \
sudo apt-get update && apt-get install -y kubeadm=1.27.x-00 && \
sudo apt-mark hold kubeadm
```
 ##### Verify that the download works and has the expected version:

```sh
kubeadm version
kubeadm upgrade plan
sudo kubeadm upgrade apply v1.27.x
```
##### Drain the node

```sh
kubectl drain <node-to-drain> --ignore-daemonsets

```
###### Upgrade kubelet and kubectl
```sh
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.27.x-00 kubectl=1.27.x-00 && \
apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

```
##### Uncordon the node
```sh
kubectl uncordon <node-to-uncordon>
```
#### Upgrade the Worker Nodes
```sh
ssh workernode
sudo apt-get update && sudo apt-get install -y kubeadm=1.27.x-00
sudo kubeadm upgrade node

kubectl drain <node-to-drain> --ignore-daemonsets

```
##### Install Kubelet
```sh
sudo apt-get update && sudo apt-get install -y kubelet=1.27.x-00 kubectl=1.27.x-00 && \
sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon <node-to-uncordon>

```
```sh


```






