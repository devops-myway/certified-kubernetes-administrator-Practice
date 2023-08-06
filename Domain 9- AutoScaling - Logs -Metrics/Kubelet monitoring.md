https://grafana.com/solutions/kubernetes/kubernetes-monitoring-introduction/

###### What is the Kubelet and possible troubles

The Kubernetes Kubelet runs in both control plane and worker nodes, as the primary node agent for all the nodes. The Kubelet service needs to be up and running permanently.
The Kubelet works in a declarative way, receiving PodSpecs and ensuring that the containers defined there are currently running and in a healthy state.

If the Kubelet is not working properly, has crashed, or it is down for any reason, the Kubernetes node will go into a NotReady state, and no new pods will be created on that node.

when Kubelet is down or not working properly; Neither Liveness nor Readiness probes will be executed, so if a workload already running on a Pod starts to fail or doesn’t work properly when Kubelet is down, it won’t be restarted, causing impact on stability, availability, and performance of such application.

###### How to monitor Kubelet - Mtrics
These metrics facilitate analysis of CPU, memory, disk, and network utilization at the container level.
Once you’ve identified a busy node using the cluster-level metrics, you can use the node’s cAdvisor output to find containers that are using too many resources and causing contention with other objects.

The Kubelet process that runs on your Kubernetes nodes provides detailed metrics that can be scraped using Prometheus. These are accessed via the /metrics endpoint on port 10255.

``````sh
kubectl get nodes -o wide
-----
NAME   	STATUS   ROLES              	AGE   VERSION   INTERNAL-IP	EXTERNAL-IP   OS-IMAGE         	KERNEL-VERSION  	CONTAINER-RUNTIME
minikube   Ready	control-plane,master   74d   v1.23.3   192.168.49.2   <none>    	Ubuntu 20.04.2 LTS   5.4.0-122-generic   docker://20.10.12

This node has the IP address 192.168.49.2. Now you can query its metrics data by accessing port 10255:
``````
``````sh
curl http://192.168.49.2:10255/metrics
``````

###### Cadvisor
Kubelet also provides cAdvisor metrics at the /metrics/cadvisor endpoint. cAdvisor (Container Advisor) is a tool that analyzes the resource usage and performance characteristics of your containers.

curl http://192.168.49.2:10255/metrics/cadvisor

###### Kubernetes logs
Beyond performance metrics and resource utilization, it’s important to capture and retain Kubernetes logs so you can refer to them in the future.
Multiple Kubernetes components write events and errors directly to log files on the host file system. Here are some common paths to inspect.

###### On the Kubernetes control plane node, look at the following:

/var/log/kube-apiserver.log: This log contains events that occurred within the Kubernetes API server.

/var/log/kube-scheduler.log: A record of scheduling decisions, created when Kubernetes decides which node to place a pod to.

/var/log/kube-controller-manager.log: The log from the Kubernetes controller manager, which is responsible for running all the built-in controllers that watch for activity in your cluster.

###### On individual worker nodes, look at the following:

/var/log/kubelet.log: The kubelet process is responsible for maintaining communication between the nodes and the Kubernetes control plane server. This log file should be your starting point when you’re troubleshooting why a node has become unavailable.

/var/log/kube-proxy.log: This is where the logs from the kube-proxy process are stored. The kube-proxy process is responsible for routing traffic through services to pods on the node.


##### Kubernetes events
Kubernetes maintains a comprehensive history of all the events that occur in your cluster. You can view events in your terminal using kubectl:
``````sh
kubectl get events


# kubelet config file
root@node01:~# journalctl -u kubelet 
.
.
.
May 30 13:43:55 node01 kubelet[8858]: E0530 13:43:55.004939    8858 reflector.go:148] vendor/k8s.io/client-go/informers/factory.go:150: Failed to watch *v1.Node: failed to list *v1.Node: Get "https://controlplane:6553/api/v1/nodes?fieldSelector=metadata.name%3Dnode01&limit=500&resourceVersion=0": dial tcp 192.24.132.5:6553: connect: connection refused
.
Check the kubelet.conf file at /etc/kubernetes/kubelet.conf

cat /etc/kubernetes/kubelet.conf

``````
