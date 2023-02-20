##### Notes:

Services in the cluster can easily make requests of other services in the cluster, but for an external service to communicate with the cluster resources, it needs to pass through authentication and authorization steps.

The kubectl proxy command:

It creates a communication channel between your local machine and your Kubernetes API server by reading the cluster configuration and credentials/users from your kube-config file.

##### How Does kubectl proxy Work

A proxy has been started, and now you can directly access the Kubernetes API by sending requests to http://127.0.0.1:8080/api
``````sh
kubectl proxy --port=8080

# Here’s a curl command to fetch the details of a pod named “nginx”

curl http://localhost:8080/api/v1/namespaces/default/pods/nginx
``````
##### Exploring kubectl proxy in detail
address: This option helps you to change the IP address of the exposed proxy. Its default value is 127.0.0.1 (i.e. localhost)

port: It enables you to choose which port the proxy will be exposed from. If you do not provide this option, port 8001 will be used for exposing the proxy.

``````sh
kubectl proxy --address="127.0.0.1"

kubectl proxy --port=81

``````
##### kubectl port-forward syntax
- We can use kubectl to set up a proxy that will forward all traffic from a local port we specify to a port associated with the Pod we determine.
- This can be performed using kubectl port-forward command. kubectl port-forward makes a specific Kubernetes API request.
- That means the system running it needs access to the API server, and any traffic will get tunneled over a single HTTP connection.

kubectl port-forward <resource-type/resource-name> [local_port]:<pod_port>

``````sh
# Sample command to perform port forwarding on pod:
kubectl port-forward pods/my-pod 8080:80

# Sample command to perform port forwarding on deployment:
kubectl port-forward deployment/my-deployment 8080:80

# Sample command to perform port forwarding on replicaset:
kubectl port-forward replicaset/my-replicaset 8080:80

# Sample command to perform port forwarding on service:
kubectl port-forward service/my-service 8080:80

``````
##### Perform kubectl port-forward in background

To perform kubectl port-forward in background you can append the command with &

``````sh
kubectl port-forward pods/my-pod 8080:80 &

``````
##### Perform port-forwarding on Pods

To perform kubectl port-forward in background you can append the command with &

``````sh
# vim nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
 namespace: default
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
--
kubectl create -f nginx.yaml
kubectl get pods
kubectl get pods -o wide
          
``````
``````sh
[root@controller ~]# kubectl port-forward pod/nginx 8080:80
Forwarding from 127.0.0.1:8080 -> 80
--
curl 127.0.0.1:8080
          
``````
##### Method-3: Listen on a random port locally, forwarding to 80 in the pod

``````sh
[root@controller ~]# kubectl port-forward pod/nginx :80
Forwarding from 127.0.0.1:40159 -> 80
Forwarding from [::1]:40159 -> 80
--
curl 127.0.0.1:40159
          
``````
##### Perform port-forwarding on Kubernetes Deployment Pods

``````sh
# vim nginx-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: dev
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2

-- # vim nginx=-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-deploy
spec:
  type: Nodeport
  selector:
    app: dev
  ports:
    - protocol: TCP
      port: 80
---
kubectl create -f nginx-deploy.yaml
kubectl create -f nginx-service.yaml

kubectl get svc
kubectl get pods -owide
--
[root@controller ~]# kubectl port-forward service/nginx-deploy  8080:80

[root@controller ~]# curl 127.0.0.1:8080
``````