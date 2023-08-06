
https://github.com/kubernetes/kube-state-metrics/tree/main/docs
https://github.com/kubernetes-sigs/metrics-server


##### Metrics Server
- The metrics server is a Kubernetes tool used to gather data on resource usage for pods, all the API objects like deployments, pods, nodes, daemonsets, Statefulsets, etc. in your Kubernetes cluster.
- Metrics Server collects resource metrics from Kubelets and exposes them in Kubernetes apiserver through Metrics API for use by Horizontal Pod Autoscaler and Vertical Pod Autoscaler.
- Metrics API can also be accessed by kubectl top, making it easier to debug autoscaling pipelines.
- Since the metric server is deployed as a pod, you can use it to get metrics from different sources like kubelet, Kubernetes API, and cAdvisor.
- metrics-server is used for “resource” metrics (CPU and memory), for kubectl top and HorizontalPodAutoscaling resource metrics. Kube state metrics service exposes all the metrics on /metrics.

##### Set Up the Metrics Server - Installation
For the kubectl top command to work, you need to have metrics API installed. with instructions here: https://github.com/kubernetes-sigs/metrics-server

The first step is to clone the metrics server repository from GitHub:
``````sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl get deploy -n kube-system


``````
###### ERROR on the metric deployment:
Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing --kubelet-insecure-tls to Metrics Server).
if you notice: couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
``````sh
kubectl logs deploy/metrics-server -n kube-system
kubectl edit deploy/metrics-server -n kube-system
Edit downloaded file and add:
 - --kubelet-insecure-tls
 to args list below:

...
labels:
    k8s-app: metrics-server
spec:
  containers:
  - args:
    - --cert-dir=/tmp
    - --secure-port=443
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --kubelet-use-node-status-port
    - --metric-resolution=15s
    - --kubelet-insecure-tls # add this line
``````
You can verify that the installation was successful by checking the status of the deployment:
``````sh
kubectl get deploy/metrics-server -n kube-system
# you will see the below:
metrics-server   1/1     1            1           4m33s
``````
Once the metrics server is up and running, you can query it for resource usage data using the kubectl top command
``````sh
kubectl top pods --namespace default
``````
###### Kubectl Top Pod/Node: Collecting Kubernetes Metrics
CPU and memory usage for kubernetes monitoring
You can check whether the Metrics API is installed by running the following command:
``````sh
kubectl top node
-----------------------------------------------------------------
NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
kind-control-plane   338m         4%     1662Mi          10%
-----------------------------------------------------------------
kubectl get pod -n kube-system
kubectl get pods --all-namespaces | grep metrics-server
kubectl run pod1 --image=nginx -n default
kubectl top pod pod1 -n default

``````




