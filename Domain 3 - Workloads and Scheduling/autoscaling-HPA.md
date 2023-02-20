##### Overview on Horizontal Pod Autoscaler

- An HPA object watches the resource consumption of pods that are managed by a controller (Deployment, ReplicaSet, or StatefulSet) at a given interval and controls the replicas by comparing the desired target of certain metrics with their real usage.

#### How Horizontal Pod Autoscaler works
- cAdvisor acts as a container resource utilization monitoring service, which is running inside kubelet on each node.
- The CPU utilizations we just monitored are collected by cAdvisor and aggregated by Heapster.
- Heapster is a service running in the cluster that monitors and aggregates the metrics.
- It queries the metrics from each cAdvisor.

##### Install and configure Kubernetes Metrics Server
Download the latest available manifest file required to deploy metrics-server:
https://github.com/kubernetes-sigs/metrics-server/releases

``````sh
[root@controller ~] wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

--
kubectl napply -f components.yaml

kubectl get deployment -n kube-system

#Verify the connectivity status
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml

kubectl top nodes
``````

##### Example-1: Autoscaling applications using HPA for CPU Usage
Now that our metrics-server is configured and running properly, we will create Horizontal Pod Autoscaler (HPA) to automate the process of scaling the application running on a deployment.

#### Create deployment

``````sh
[root@controller ~]# cat nginx-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type: dev
  name: nginx-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      type: dev
  template:
    metadata:
      labels:
        type: dev
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
kubectl create -f nginx-deploy.yaml
kubectl get deployment

``````

#### Create Horizontal Pod Autoscaler

``````sh

kubectl autoscale deployment nginx-deploy --cpu-percent=10 --min=1 --max=5

-- # we could have also used following YAMLfile to create this HPA resource:
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-deploy
spec:
  minReplicas: 1
  maxReplicas: 5
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deploy ## Name of the deployment
  targetCPUUtilizationPercentage: 10
--
kubectl get hpa
kubectl get pods

-- # Verify kubernetes autoscaling UP and DOWN
root@nginx-deploy-7979f78fb5-2qhtc:/# dd if=/dev/zero of=/dev/null
kubectl get hpa
kubectl get pods
kubectl describe hpa nginx-deploy
``````
