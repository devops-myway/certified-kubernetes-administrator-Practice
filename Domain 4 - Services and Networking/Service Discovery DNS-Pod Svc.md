
- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
- https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/


##### Note 
There are two objects that you could access via DNS within your Kubernetes cluster and those are Services and Pods. Now Pods can get a dns entry, but you’d be wondering why, since we’ve learned that accessing pods should be done through a Service. For this reason, pods don’t get a DNS entry by default with CoreDNS.

Services get a DNS entry of the service name appended to the namespace and then appended to “svc.cluster.local”. Similarly pods get entries of PodIPAddress appended to the namespace and then appended to .pod.cluster.local if Pod DNS is enabled in your cluster.

#### How DNS resolution works in Kubernetes clusters
The startup parameters of kubelet in a cluster include 
- --cluster-dns=<dns-service-ip> : The parameters are used to configure the IP address
- --cluster-domain=<default-local-domain> : The parameters are used to configure the suffix of the base domain name for the DNS server.

#### Namespaces of Services
Each time you deploy a new service or pod, the DNS service sees the calls made to the Kube API and adds DNS entries for the new object. Then other containers within a Kubernetes cluster can use these DNS entries to access the services within it by name.

A Service named kube-dns is deployed to expose these workloads to DNS queries in the cluster. Two backend pods named coredns are deployed for CoreDNS.
The DNS configuration file in the pod is /etc/resolv.conf. The file contains the following content:

- nameserver 10.32.0.10
- search <namespace>.svc.cluster.local svc.cluster.local cluster.local
- options ndots:5

The parameters in the dnsConfig for DNS policy also has the below parameters:
- nameserver:
Specifies the IP addresses of the DNS servers.
- search:
Specifies the suffixes that are used for DNS queries. More suffixes indicate more DNS queries. For a clusters, suffixes are
kube-system.svc.cluster.local, svc.cluster.local, and cluster.local
- options:
Specifies the options for the DNS configuration file. You can specify multiple key-value pairs. For example, ndots:5 specifies that if the number of dots in the domain name string is greater than 5, the domain name is a fully qualified domain name and is directly resolved. If the number of dots in the domain name string is less than 5, the domain name is appended with the suffixes specified by the search parameter before it is resolved.

##### DNS Records
What objects get DNS records
- Services:
Normal" (not headless) Services are assigned DNS A and/or AAAA records, depending on the IP family or families of the Service, with a name as below which resolves to the cluster IP of the Service:

my-svc.my-namespace.svc.cluster-domain.example

Headless Services (without a cluster IP) Services are also assigned DNS A and/or AAAA records:
my-svc.my-namespace.svc.cluster-domain.example

##### Pods
Pod has the following DNS resolution:
pod-ip-address.my-namespace.pod.cluster-domain.example.
172-17-0-3.default.pod.cluster.local

Any Pods exposed by a Service have the following DNS resolution available:
pod-ip-address.service-name.my-namespace.svc.cluster-domain.example

##### Configure DNS resolution
A Service named kube-dns is deployed in a cluster to provide DNS resolution services for the cluster.
Two backend pods named coredns are deployed for the kube-dns Service. You can run the following command to query information about the coredns pods:
``````sh
kubectl get svc kube-dns -n kube-system

kubectl get deployment coredns -n kube-system

``````

##### Use dnsPolicy to configure DNS policies for a cluster in different scenarios
DNS policies can be set on a per-Pod basis. Currently Kubernetes supports the following Pod-specific DNS policies. These policies are specified in the dnsPolicy field of a Pod Spec.

- ClusterFirst:
This policy indicates that a pod uses CoreDNS to resolve domain names. The /etc/resolv.conf file contains the address of the DNS server that is provided by CoreDNS, which is kube-dns. This is the default DNS policy for workloads in a cluster.
- None:
This policy indicates that a pod ignores the DNS settings of the cluster. You must customize the DNS settings by using the dnsConfig field.
- Default:
This policy indicates that a pod inherits the DNS settings from the node where the pod is deployed. In a cluster, nodes are created based on Elastic Compute Service (ECS) instances. Therefore, a pod directly uses the /etc/resolv.conf file of the ECS instance-based node where the pod is deployed. This file contains the address of a DNS server that is provided.
- ClusterFirstWithHostNet:
This policy indicates that a pod in HostNetwork mode uses the ClusterFirst policy. If you do not specify a policy for a pod, the pod uses the Default policy.

###### DNS in Action
``````sh
cat << EOF | kubectl apply -f -
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: nginx-deployment 
  labels: 
    app: nginx
spec: 
  replicas: 2 
  selector:
    matchLabels:
      app: nginx 
  template: 
    metadata:
      labels: 
        app: nginx
    spec:
      containers:
      - name: nginx-container 
        image: nginx 
        ports:
        - containerPort: 80
EOF

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30001
    protocol: TCP
  selector:
    app: nginx
EOF
``````
``````sh
kubectl get svc

``````
##### Deploy a test Pod to test DNS resoltion
Deploy another container where we can run some DNS lookups
``````sh
kubectl run testpod --image=ubuntu --command -- sh -c "sleep 3600"
kubectl exec -it po/testpod -- sh
--
apt update && upgrade
apt install dnsutils
nslookup nginxsvc
---
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   nginxsvc.default.svc.cluster.local
Address: 10.99.23.119

``````

###### Scenario 1: In this scenario, you must specify dnsPolicy: ClusterFirst for the DNS policy settings.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  namespace: default
spec:
  containers:
  - image: ubuntu
    command:
      - sleep
      - "3600"
    imagePullPolicy: Always
    name: ubuntu
  dnsPolicy: ClusterFirst
EOF
``````
##### Scenario 2: Customize DNS settings for a pod
To customize DNS settings for a Deployment, you must specify dnsPolicy: None for the DNS policy settings
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 192.0.2.1 # this is an example
    searches:
      - ns1.svc.cluster-domain.example
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
EOF
----- # When the Pod above is created, the container test gets the following contents in its /etc/resolv.conf file
nameserver 192.0.2.1
search ns1.svc.cluster-domain.example my.dns.search.suffix
options ndots:2 edns0
------- # For IPv6 setup, search path and name server should be set up like this:
kubectl exec -it dns-example -- cat /etc/resolv.conf
``````
###### Scenario 3: Enable pods in HostNetwork mode to access services in a cluster
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet

``````
##### Use the hostAliases field to configure the /etc/hosts file in a pod
If you want to map a specified domain name to a static IP address for DNS resolutions within a specified pod, you can add the hostAliases field to the configurations of the pod to modify the /etc/hosts file.
The hostAliases field is added to the spec section in the pod configurations. After the pod is launched, the /etc/resolv.conf file is initialized with the following content:
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: hostaliases-pod
spec:
  hostAliases:
  - ip: "127.0.**.**"
    hostnames:
    - "foo.local"
    - "bar.local"
  - ip: "10.1.**.**"
    hostnames:
    - "foo.remote"
  containers:
  - name: cat-hosts
    image: busybox:1.28
    command:
    - cat
    args:
    - "/etc/hosts"
----------
# Kubernetes-managed hosts file.
127.0.**.**	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.**.**	hostaliases-pod

# Entries added by HostAliases.
127.0.**.**	foo.local	bar.local
10.1.**.**	foo.remote	bar.remote
``````
