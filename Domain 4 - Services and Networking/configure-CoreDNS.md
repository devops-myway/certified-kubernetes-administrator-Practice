#### Resolving services via DNS

- Kubernetes clusters typically run an internal DNS server that all pods in the cluster are configured to use. 
- In most clusters, this internal DNS service is provided by CoreDNS, whereas some clusters use kube-dns. 
- You can see which one is deployed in your cluster by listing the pods in the kube-system namespace.

A service is resolvable under the following DNS names:

<service-name>, if the service is in the same namespace as the pod performing the DNS lookup,
<service-name>.<service-namespace> from any namespace, but also under
<service-name>.<service-namespace>.svc, and
<service-name>.<service-namespace>.svc.cluster.local  - The default domain suffix is cluster.local but can be changed at the cluster level.

- The reason you don’t need to specify the fully qualified domain name (FQDN) when resolving the service through DNS is because of the search line in the pod’s /etc/resolv.conf file.

``````sh
# For the quote-001 pod

cat /etc/resolv.conf

search kiada.svc.cluster.local svc.cluster.local cluster.local localdomain
nameserver 10.96.0.10
options ndots:5

-- # you can list all the services in your cluster to find out IP address is in the nameserver line
kubectl get svc -A


``````