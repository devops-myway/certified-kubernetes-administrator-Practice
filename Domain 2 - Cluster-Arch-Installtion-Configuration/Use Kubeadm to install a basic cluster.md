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

#### Control plane- master Security Group

|Protocol	 |Direction	|Port Range	|Purpose	|Used By
|-------|-----------|-------|---------------|----------|
|TCP	|Inbound	|6443	|Kubernetes API server	|All
|TCP	|Inbound	|2379-2380	|etcd server client API	|kube-apiserver, etcd
|TCP	|Inbound	|10250	|Kubelet API	Self, |Control plane
|TCP	|Inbound	|10259	|kube-scheduler	|Self
|TCP	|Inbound	|10257	|kube-controller-manager	|Self

#### Worker node(s) - Security Group

|Protocol|	Direction|	Port Range|	Purpose|	Used By
|---------|----------|------------|-------|---------|
|TCP	|Inbound	|10250	|Kubelet API	|Self, Control plane
|TCP	|Inbound	|30000-32767	|NodePort Services|	All

###### Note:
For installing Weave Net (Add-on), you should make sure the following ports are not blocked by your firewall:
Open Both Master and Workers security Groups to TCP 6783 and ignore the UDP 6783/6784.

- Swap disabled. You MUST disable swap in order for the kubelet to work properly and this is done as below

##### step0.1: Installing a container runtime
-------------------------------------------------
download the Containerd runtime with the command and use a simple shell script to execute this task with permissions
``````sh
sudo swapoff -a
sudo apt update
sudo apt upgrade
sudo apt-get install wget
sudo apt-get install curl

``````
##### Step 1: Installing containerd from the official binaries
``````sh
wget https://github.com/containerd/containerd/releases/download/v1.6.8/containerd-1.6.8-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.6.8-linux-amd64.tar.gz       # Unpack that file into /usr/local/ with the command:

``````

Note: systemd: If you intend to start containerd via systemd, you should also download the containerd.service
Then, download the systemd service file and set it up so that you can manage the service via systemd.
-------------
``````sh

wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mv /usr/local/lib/systemd/system/##

sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd

``````

##### Step 2: Installing runc:runc command line tool which is used to deploy containers with Containerd.
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

##### Step4: How to configure Containerd: With everything installed, we can now configure Containerd. Create a new directory to house the Containerd configurations with:
--------------------
``````sh

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
##### Step8: Initializing your control-plane node - only on the control plane
--------------------------------------------------------------------
``````sh

sudo kubeadm init --pod-network-cidr 192.168.0.0/16

To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

``````

##### Step9: Installing a Pod network add-on
-----------------------------------
``````sh

kubectl apply -f <add-on.yaml>

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

kubectl get pods --all-namespaces  -- for testing purpose

``````
##### step10: Joining your nodes
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
##### Step 11: Verification
-----------
``````sh
kubectl get nodes
kubectl run nginx --image=nginx
kubectl get pods -o wide

``````
