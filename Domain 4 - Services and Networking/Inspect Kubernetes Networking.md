#### Finding a Pod’s Cluster IP

To find the cluster IP address of a Kubernetes pod, use the kubectl get pod command on your local machine, with the option -o wide
The IP column will contain the internal cluster IP address for each pod. You can list all pods in all namespaces by adding the flag --all-namespaces.

``````sh
kubectl get pod -o wide

``````
#### Finding a Service’s IP

We can find a Service IP using kubectl as well. In this case we will list all services in all namespaces

``````sh
kubectl get service --all-namespaces
``````
#### Finding and Entering Pod Network Namespaces

Each Kubernetes pod gets assigned its own network namespace. Network namespaces (or netns) are a Linux networking primitive that provide isolation between network devices.

It can be useful to run commands from within a pod’s netns, to check DNS resolution or general network connectivity
``````sh
docker ps
docker inspect --format '{{ .State.Pid }}' container-id-or-name
nsenter -t your-container-pid -n ip addr

``````
#### Finding a Pod’s Virtual Ethernet Interface

Each pod’s network namespace communicates with the node’s root netns through a virtual ethernet pipe.
On the node side, this pipe appears as a device that typically begins with veth and ends in a unique identifier, such as veth77f2275 or veth01.

``````sh
nsenter -t your-container-pid -n ip addr

ip addr
``````
#### Inspecting Iptables Rules

To dump all iptables rules on a node, use the iptables-save command:

``````sh
iptables-save
iptables -t nat -L KUBE-SERVICES

``````
#### Querying Cluster DNS

One way to debug your cluster DNS resolution is to deploy a debug container with all the tools you need, then use kubectl to exec nslookup on i
Another way to query the cluster DNS is using dig and nsenter from a node. If dig is not installed, it can be installed with apt on Debian-based Linux distributions:

``````sh
apt install dnsutils

# First, find the cluster IP of the kube-dns service:
kubectl get service -n kube-system kube-dns

nsenter -t 14346 -n dig kubernetes.default.svc.cluster.local @10.32.0.10

``````