##### Documentation Links: This is everything you need to know about starting a cluster using the documentation with Kubeadm

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- https://kubernetes.io/docs/reference/ports-and-protocols/
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
- https://github.com/containerd/containerd/blob/main/docs/getting-started.md
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/

##### Step 0.0: Before we begin be sure of the below setup
-----------------
- Use Linux distributions based on Debian via Ec2 instances and ssh (login) with 2x worker nodes and 1 control plane
- 2 GB or more of RAM or 2 CPUs or more for the Instances or machines
- Certain ports are open on your machines and this should be at security Group levels
Check required ports: https://kubernetes.io/docs/reference/ports-and-protocols/


##### Create the ssh key using via cli below or ppk for Putty - move private key to .ssh hidden folder and restrict access
``````sh
    mv ~/Downloads/k8s-node.pem ~/.ssh
    chmod 400 ~/.ssh/k8s-node.pem
``````
##### ssh into ec2 instance with its public ip
``````sh
    ssh -i ~/.ssh/k8s-node.pem ubuntu@public-ip
``````
##### set host names of nodes
``````sh
    sudo nano /etc/hosts
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

|Protocol	 |Direction	|Port Range	|Purpose	|Used By
|-------|-----------|-------|---------------|----------|
|TCP	|Inbound	|6443	|Kubernetes API server	|All (0.0.0.0/16 - Anywhere)
|TCP	|Inbound	|2379-2380	|etcd server client API	|kube-apiserver, etcd (within vpc-ip)
|TCP	|Inbound	|10250	|Kubelet API	Self, |Control plane (within vpc-ip)
|TCP	|Inbound	|10259	|kube-scheduler	|Self  (within vpc-ip)
|TCP	|Inbound	|10257	|kube-controller-manager	|Self  (within vpc-ip)

#### Worker node(s) - Security Group

|Protocol|	Direction|	Port Range|	Purpose|	Used By
|---------|----------|------------|-------|---------|
|TCP	|Inbound	|10250	|Kubelet API	|Self, Control plane (within vpc)
|TCP	|Inbound	|30000-32767	|NodePort Services|	All (Anywhere)

###### Note: Add-on Security Group to be added on Master and Worker Nodes

For installing Weave Net (Add-on), you should make sure the following ports are not blocked by your firewall:
Open Both Master and Workers security Groups to TCP 6783 and ignore the UDP 6783/6784.

- Swap disabled. You MUST disable swap in order for the kubelet to work properly and this is done as below

#### Installing a Container Runtime (CRI) so Kubelet can talk to it --- Installtion steps for all nodes


##### step0.1: Installing a Container Runtime on ALL NODES
-------------------------------------------------
download the Containerd runtime with the command and use a simple shell script to execute this task with permissions
``````sh
sudo swapoff -a
sudo apt update
sudo apt upgrade
sudo apt-get install wget
sudo apt-get install curl

``````
##### step0.2: Install Conatinerd Package on Ubuntu with Apt-Get
-------------------------------------------------

``````sh
sudo apt-get update -y

sudo apt-get install -y containerd

``````

##### Step 1: Installing Containerd from the official binaries
``````sh
wget https://github.com/containerd/containerd/releases/download/v1.6.8/containerd-1.6.8-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.6.8-linux-amd64.tar.gz       # Unpack that file into /usr/local/ with the command:

``````

###### Note:
systemd: If you intend to start containerd via systemd, you should also download the containerd.service
Then, download the systemd service file and set it up so that you can manage the service via systemd.
-------------
``````sh

wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mv /usr/local/lib/systemd/system/##

sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd

``````

##### Step 2: Installing unc:runc command line tool which is used to deploy containers with Containerd.
--------------
``````sh
wget https://github.com/opencontainers/runc/releases/download/v1.1.3/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

``````

##### Step 3: Installing CNI plugins: Container Network Interface, which is used to provide the necessary networking functionality
------------
``````sh

wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz       #Unpack the CNI file into our new directory with:

``````

#### Step4: Configure Containerd - After the CRI installations done!!

With everything installed, we can now configure Containerd. Create a new directory to house the Containerd configurations with:
--------------------
``````sh
# The default configuration can be generated via containerd config default > /etc/containerd/config.toml.

sudo mkdir /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml  # Create the configurations with

``````

##### Step5: Configuring the systemd cgroup driver
---------
To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set
``````sh

sudo nano /etc/containerd/config.toml

 [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
 SystemdCgroup = true

``````

##### Step6: If you apply this change, make sure to restart containerd:
---------
``````sh

sudo systemctl restart containerd
sudo systemctl status containerd

``````

##### Step7: Install and configure prerequisites: Forwarding IPv4 and letting iptables see bridged traffic
---------------------------------------
``````sh
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

``````
##### Installing kubeadm, kubelet and kubectl - All Nodes

You will install these packages on all of your machines:
- kubeadm: the command to bootstrap the cluster.

- kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.

- kubectl: the command line util to talk to your cluster.

``````sh
#Update the apt package index and install packages needed to use the Kubernetes apt repository
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

#Download the Google Cloud public signing key:
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

#Add the Kubernetes apt repository:
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

#Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

``````

##### Step8: Initializing your Master Node or Control-Plane Node - Only on the Control Plane
--------------------------------------------------------------------
``````sh

sudo kubeadm init --pod-network-cidr 192.168.0.0/16

To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

``````

##### Step9: Installing a Pod Network add-on or Container Network Interface - CNI
You must deploy a Container Network Interface (CNI) based Pod network add-on so that your Pods can communicate with each other.
Cluster DNS (CoreDNS) will not start up before a network is installed.

-----------------------------------
``````sh

kubectl apply -f <add-on.yaml>

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

kubectl get pods --all-namespaces  -- for testing purpose

``````
##### step10: Joining your Worker-Nodes : Only on the Worker Nodes
----------------------------
``````sh
SSH to the machine

Become root (e.g. sudo su -)

Install a runtime if needed

Run the command that was output by kubeadm init. For example:

kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>

If you do not have the token, you can get it by running the following command on the control-plane node:

kubeadm token list

``````
##### Step 11: Verification of pods deployments- Master node and watch the pods schedule to worker nodes
-----------
``````sh
kubectl get nodes
kubectl run nginx --image=nginx
kubectl get pods -o wide

``````
