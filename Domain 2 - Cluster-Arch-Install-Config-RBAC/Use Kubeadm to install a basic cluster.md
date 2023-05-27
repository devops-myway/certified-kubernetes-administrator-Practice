##### Documentation Links: This is everything you need to know about starting a cluster using the documentation with Kubeadm

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/

##### Step 0.0: Before we begin be sure of the below setup
-----------------
- Use Linux distributions based on Debian via Ec2 instances and ssh (login) with 2x worker nodes and 1 control plane (master node)
- 4 GB or more of RAM or 2 vCPUs or more for the Instances or machines
- Certain ports are open on your machines and this should be at security Group levels
- Check required ports: https://kubernetes.io/docs/reference/ports-and-protocols/
##### Note:
Be mindful of the security groups port 443 should not be used from my observation as it may affect kubelet installtion process.

##### Create the ssh key using via cli below or ppk for Putty - move private key to .ssh hidden folder and restrict access
``````sh
sudo su and cd /
sudo apt update and sudo apt upgrade
cat /etc/os-release
sudo apt install net-tools
ifconfig -a
###### Move to the ssh_folder directory and move the .pem public key
    mv cluster-keypair.pem ~/.ssh
    chmod 400 ~/.ssh/k8s-node.pem
``````
##### ssh into ec2 instance with its public ip
``````sh
    ssh -i ~/.ssh/k8scluster-keypair.pem ubuntu@public-ip
``````
##### set host names of nodes- Setup /etc/hosts file
``````sh
    sudo nano /etc/hosts 
    or 
    sudo vim /etc/hosts
``````
##### get priavate ips of each node and add this to each server
``````sh
45.14.48.178 master
45.14.48.179 worker1
45.14.48.180 worker2
``````
##### we can now use these names instead of typing the IPs, when nodes talk to each other. After that, assign a hostname to each of these servers.

##### on master server
``````sh
sudo hostnamectl --help

sudo hostnamectl set-hostname master

exit and reconnect again
``````
##### on worker1 server
``````sh
    sudo hostnamectl set-hostname worker1 
    exit and reconnect again
``````
##### on worker2 server
``````sh
sudo hostnamectl set-hostname worker2
exit and reconnect again
``````

#### Control plane- master Security Group

https://kubernetes.io/docs/reference/networking/ports-and-protocols/


Inbound Rules:
|Protocol	 |Direction	|Port Range	|Purpose	|Used By
|-------|-----------|-------|---------------|----------|
|TCP	|Inbound	|6443	|Kubernetes API server	|All (0.0.0.0 - Anywhere)
|TCP	|Inbound	|2379-2380	|etcd server client API	|kube-apiserver, etcd (within vpc-ip)
|TCP	|Inbound	|10250	|Kubelet API	Self, |Control plane (vpc-ip pruvate network)
|TCP	|Inbound	|10259	|kube-scheduler	|Self  (within vpc-ip)
|TCP	|Inbound	|10257	|kube-controller-manager	|Self  (within vpc-ip)
|TCP	|Inbound	|22	|ssh	|anywhere 0.0.0.0
|TCP	|Inbound	|6783	|add-on for weave	|anywhere 0.0.0.0

#### Worker node(s) - Security Group

|Protocol|	Direction|	Port Range|	Purpose|	Used By
|---------|----------|------------|-------|---------|
|TCP	|Inbound	|10250	|Kubelet API	|Self, Control plane (within vpc)
|TCP	|Inbound	|30000-32767	|NodePort Services|	All (Anywhere)
|TCP	|Inbound	|22	|ssh	|anywhere
|TCP	|Inbound	|6783	|add-on for weave	|anywhere 0.0.0.0

##### step0.1: Installing a Container Runtime on ALL NODES
-------------------------------------------------
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

``````sh

sudo swapoff -a
#verify if swap is off
sudo free -m
sudo apt update
sudo apt upgrade

``````

##### step2.1: Installing a container runtime CRI - using containerd - ALL NODES

Note: The containerd.io packages in DEB and RPM formats are distributed by Docker (not by the containerd project).
The containerd.io package contains runc too, but does not contain CNI plugins.
See the Docker documentation for how to set up apt-get or dnf to install containerd.io packages:
containerd:
https://github.com/containerd/containerd/blob/main/docs/getting-started.md

goto Option 2: From apt-get or dnf
click ubuntu: https://docs.docker.com/engine/install/ubuntu/

###### Installing CRI with containerd.io package mgr

# Install containerd which is simple and faster
sudo apt-get update
sudo apt-get -y install containerd

------------------------
##### installing using docker repo on ubuntu as below

``````sh
#! /bin/bash

#Uninstall old versions
sudo apt-get remove docker docker-engine docker.io containerd runc
#Install using the repository
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker’s official GPG key:
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

#Use the following command to set up the repository:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Then, install the package "containerd.io" as the Containerd Container Runtime.
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl start containerd
sudo systemctl enable containerd
sudo systemctl status containerd

``````
#### Step4: Goto Advanced topics - Containerd configuration for Kubernetes
containerd uses a configuration file config.toml for handling its demons.
When installing containerd using official binaries, you will not get the configuration file.
So, generate the default configuration file using the below commands.

sudo cat /etc/containerd/config.toml    verify if cri is disabled, else do below steps.
disabled_plugins = ["cri"]  and this has directory has to be removed and manually created.
disabled_plugins = []       after manual generation of the config.toml file

containerd uses a configuration file located in /etc/containerd/config.toml
The default configuration can be generated via containerd config default > /etc/containerd/config.toml
``````sh
sudo rm -rf /etc/containerd/config.toml   # check if this file exist and rm the existing one from docker and create a new default config.toml file
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml 
sudo cat /etc/containerd/config.toml   # verify again for cri removed from disabled plugin  disabled_plugins = []
``````
##### Step5: Configuring the systemd cgroup driver
---------
To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set
``````sh

sudo vi /etc/containerd/config.toml

 [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
 SystemdCgroup = true

``````
##### Step6: If you apply this change, make sure to restart containerd:
---------
``````sh
sudo systemctl restart containerd
sudo systemctl status containerd

``````
##### Step2: Install and configure prerequisites
----------
create a shell script e.g install-cont.sh with
#! /bin/bash
chmod u+x install-contanerd.sh

``````sh
#! /bin/bash
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

# To verify the modules are loaded and running and system variables are set to 1
sudo lsmod | grep br_netfilter
sudo lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

``````
##### Install core Binaries: kubeadm, kubelet and kubectl - All NODES

You will install these packages on all of your machines:
- kubeadm: the command to bootstrap the cluster.

- kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.

- kubectl: the command line util to talk to your cluster.

``````sh
# Create a simple bash file and chmod u+x bashfile.sh

#! /bin/bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Always sudo into /etc/apt/keyrings directory to verify if the location exit after installation of containerd.
#sudo mkdir -p /etc/apt/keyrings
#Download the Google Cloud public signing key: the below wasnt working so it has been updated.
# curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
#Add the Kubernetes apt repository:
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

#Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

``````

##### Step8: Initializing your Master Node or Control-Plane Node - ONLY CONTROL PLANE

``````sh

sudo kubeadm init --pod-network-cidr 192.168.0.0/16

To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl cluster-info
kubectl get nodes
``````

##### Step9: Installing a Pod Network add-on or Container Network Interface - CNI - ONLY CONTROL PLANE
You can install a Pod network add-on with the following command on the control-plane node or a node that has the kubeconfig credentials:
See a list of add-ons that implement the Kubernetes networking model: https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

Add TCP 6783 to the secrutiy groups of the all nodes.
You must deploy a Container Network Interface (CNI) based Pod network add-on so that your Pods can communicate with each other.
Cluster DNS (CoreDNS) will not start up before a network is installed.
We will use weave-net networking plugin for our Kubernetes Cluster. You must permit traffic to flow through TCP 6783 and UDP 6783/6784 on all the nodes, which are Weave’s control and data ports.
Once a Pod network has been installed, you can confirm that it is working by checking that the CoreDNS Pod is Running in the output of: 


``````sh

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

kubectl get pods --all-namespaces # to check if the coreDNS is running
kubectl get pods --all-namespaces -o wide

``````
##### step10: Joining your Worker-Nodes : Only on the Worker Nodes
----------------------------
``````sh
sudo kubeadm token list
sudo kubeadm token create  #if the token expired

sudo kubeadm join 172.31.29.2:6443 --token 8mg8tb.u2l510xf7kse98u0 \
        --discovery-token-ca-cert-hash sha256:65f5391c70d01b9dc49e90708129cdbffad07ad28d6fd53a9cbe14e925ad7409

``````
##### Step 11: Verification of pods deployments- Master node and watch the pods schedule to worker nodes
-----------
``````sh
kubectl get nodes
kubectl run nginx --image=nginx
kubectl get pods -o wide
kubectl get pods --all-namespaces -owide

``````

