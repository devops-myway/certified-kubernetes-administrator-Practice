##### Documentation Link

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

##### Ugrading and updating a version kubernetes cluster: 
--------------------------------------------------
I will be upgrading Kubernetes version 1.23 to 1.24 and replace the version numbers as per existing versions
version - major.minor.patch : 1.23.x

The steps are as follows:
Upgrade a primary control plane node.
Upgrade additional control plane nodes.
Upgrade worker nodes.

upgrading the following components:
kubeadm
kubelet
kubectl

swap must always be disabled
----------------
##### step1: Know the cluster and node names
```sh
kubectl get nodes
```

##### step2: Determine which version to upgrade to on the control plane
```sh
ssh into the master node

sudo apt update
sudo apt-cache madison kubeadm
# find the latest 1.25 version in the list
# it should look like 1.25.x-00, where x is the latest patch
```

##### step3: Upgrading control plane nodes

The upgrade procedure on control plane nodes should be executed one node at a time

Call "kubeadm upgrade  --- sudo kubeadm upgrade plan

My current version is 1.23.6. I will be upgrading it to one higher version, ie, 1.24.6-0

Upgrade kubeadm: unhold kubeadm and Install the required version
----------------------------------

 # replace x in 1.25.x-00 with the latest patch version
 sudo apt-mark unhold kubeadm && \
 sudo apt-get update && sudo apt-get install -y kubeadm=1.25.x-00 && \
 sudo apt-mark hold kubeadm


 ##### Verify that the download works and has the expected version:
 ----------------------
 ```sh
 kubeadm version or kubeadm version -o json

 ```
 
 ##### Verify the upgrade plan:
 -------------------
 ```sh
 kubeadm upgrade plan
 ```

 ##### Choose a version to upgrade to, and run the appropriate command. For example:
 ---------------------
 # replace x with the patch version you picked for this upgrade
sudo kubeadm upgrade apply v1.25.x

-------------------------------------------
##### For the other control plane nodes - Do the steps below
------------------------------------------
Same as the first control plane node but use

sudo kubeadm upgrade node

instead of 

sudo kubeadm upgrade apply

Also calling kubeadm upgrade plan and upgrading the CNI provider plugin is no longer needed
----------------------------------------

##### Drain the node:
-------------
Prepare the node for maintenance by marking it unschedulable and evicting the workloads:

# replace <node-to-drain> with the name of your node you are draining
kubectl drain <node-to-drain> --ignore-daemonsets

Upgrade kubelet and kubectl
----------------------
Upgrade the kubelet and kubectl:


# replace x in 1.25.x-00 with the latest patch version
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.25.x-00 kubectl=1.25.x-00 && \
apt-mark hold kubelet kubectl

##### Restart the kubelet:
-------------------
sudo systemctl daemon-reload
sudo systemctl restart kubelet


##### Uncordon the node:
---------------
Bring the node back online by marking it schedulable:

# replace <node-to-drain> with the name of your node
kubectl uncordon <node-to-drain>

##### Upgrade worker nodes---------------------------

Upgrade kubeadm:
-------------
# replace x in 1.25.x-00 with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.25.x-00 && \
apt-mark hold kubeadm

Call "kubeadm upgrade
-----------------------
sudo kubeadm upgrade node

Drain the node:
---------------
Prepare the node for maintenance by marking it unschedulable and evicting the workloads:

# replace <node-to-drain> with the name of your node you are draining
kubectl drain <node-to-drain> --ignore-daemonsets

Upgrade kubelet and kubectl:
----------------------------
# replace x in 1.25.x-00 with the latest patch version
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.25.x-00 kubectl=1.25.x-00 && \
apt-mark hold kubelet kubectl

Restart the kubelet:
------------------
sudo systemctl daemon-reload
sudo systemctl restart kubelet

Uncordon the node:
---------------------
Bring the node back online by marking it schedulable

# replace <node-to-drain> with the name of your node
kubectl uncordon <node-to-drain>

Verify the status of the cluster:
------------------------------
The STATUS column should show Ready for all your nodes, and the version number should be updated

kubectl get nodes




