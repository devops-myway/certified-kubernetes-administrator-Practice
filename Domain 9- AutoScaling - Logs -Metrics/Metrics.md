
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
git clone https://github.com/kubernetes-sigs/metrics-server.git

git clone https://github.com/devopscube/kube-state-metrics-configs.git

``````
Next, navigate to the deploy/kubernetes directory within the repository and apply the required manifests to your cluster:
This will create several resources within your cluster, including a deployment, a service, and a cluster role binding.
``````sh
cd metrics-server/deploy/kubernetes
kubectl apply -f .

kubectl apply -f kube-state-metrics-configs/

``````
You can verify that the installation was successful by checking the status of the deployment:
``````sh
kubectl get deployments metrics-server -n kube-system

kubectl get deployments kube-state-metrics -n kube-system

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

``````

##### Using Kubectl Top
kubectl top: is a command used to list all the running nodes and pods along with their resource utilization. It provides you a snapshot of resource utilization metrics like CPU, memory, and storage on each running node. Each node in Kubernetes comes with cAdvisor, which is an open-source agent that monitors resource usage about containers. kubectl command gets resource utilization metrics from cAdvisor via the metrics-server.

``````sh
kubectl top pod --namespace web-app
kubectl top pod nginx-84ac2948db-12bce --namespace web-app --containers

--- # You can also use it to view metrics from your nodes:

kubectl top node
``````
##### How Do You Enable Horizontal Pod Autoscaling (HPA)
You can use the metric server to enable horizontal pod autoscaling (HPA), which allows you to automatically increase or decrease the number of pods in a deployment based on resource utilization.
``````sh
kubectl autoscale deployment <deployment-name> --cpu-percent=50 --min=1 --max=10

kubectl get hpa
kubectl describe hpa
kubectl delete hpa
``````
###### How to read the output from kubectl top
CPU Usage explained:
CPU(cores) 338m means 338 millicpu. 1000m is equal to 1 CPU, hence 338m means 33.8% of 1 CPU.
CPU% It is displayed only for nodes, and it stands for the total CPU usage percentage of that node.

Memory Usage explained:
Memory Memory being used by that node. Mi under memory stands for mebibytes. Memory% It is also displayed only for nodes, and it stands for total memory usage percentage of that node.

###### Monitoring containers with cAdvisor in Kubernetes
cAdvisor (short for "Container Advisor") is a daemon that collects the data about the resource usage and performance of your containers.
cAdvisor is integrated with the kubelet binary, and it exposes the metrics on /metrics/cadvisor endpoint.
For the kubectl top command to work, you need to have metrics API installed. with instructions here: https://github.com/kubernetes-sigs/metrics-server
Therefore we don't need to install cAdvisor on the Kubernetes cluster explicitly. it also allows you to export the collected data to different storage driver plugins e.g Prometheus,ElasticSearch etc.




