https://kubernetes.io/docs/concepts/services-networking/service/

##### Exposing Services to External Clients

A Kubernetes Service is a method for exposing a network application that is running and passing connection to one or more backing pods in your cluster.

Add labels to Pod objects and specify the label selector in the Service object. The pods whose labels match the selector are part of the service registered service endpoints.

- The shorthand for services is svc

#### Understanding different Kubernetes Service Types

ClusterIP: It exposes the service within the Kubernetes cluster. This Service is only reachable from within the cluster. It can not be accessed from outside the cluster.

NodePort: It will expose the service on a static port on the deployed node. This service can be accessed from outside the cluster using the NodeIP:Nodeport.

LoadBalancer: Exposes the Service externally using a cloud provider's load balancer, this creates a Public IP on the Cloud provider

ExternalName: which is a relatively new object that works on DNS names and redirection is happening at the DNS level.
Service without selector: which is used for direct connections based on IP port combinations without an endpoint. And this is useful for connections to a database or between namespaces.

#### Using kubectl expose
The easiest way to create a service is through kubectl expose

``````sh
kubectl create deployment nginx-lab-1 --image=nginx --replicas=3 --dry-run=client -o yaml > nginx-lab-1.yaml

---- # modify few sections and following is my final template file to create a new deployment nginx-lab-1 with a label app=dev and 3 replicas.
[root@controller ~]# cat my-deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

------ # cat quote for pod
kubectl create service nodeport nginx --tcp=80:80 --dry-run=client -oyaml > nginx-svc.yaml
kubectl apply -f nginx-svc.yaml


apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  externalTrafficPolicy: Local
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:    
    app: nginx
  type: NodePort
--
kubectl apply -f nginx-lab-1.yml
kubectl get pods
kubectl get deployment | grep nginx
kubectl get replicaset | grep nginx
---
kubectl apply -f my-service.yml
kubectl get service | grep nginx
kubectl describe service nginx

--
 the nginx application can be accessed through this service on NodeIp:NodePort
 open another terminal---------s
 
 curl nopeip:nodeport
``````
###### Ex2 expose a pod to nodeport and loadbalancer type services
``````sh
kubectl run nginx-pod --image=nginx --port=80 --labels="app=nginx" --dry-run=client -oyaml > nginx-pod.yaml
kubectl apply -f nginx-pod.yaml 

kubectl expose pod nginx-pod --port=8080 --target-port=80 --name=nginx-svc --type=NodePort --dry-run=client -oyaml > nginx-np-svc.yaml
vi nginx-np-svc.yaml
manually add = nodePort: 30000
kubectl apply -f nginx-np-svc.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
      nodePort: 30000
----

on bear service - copy the public-ip: node-port = http://34.240.137.16:30000/
``````

##### Accessing LoadBalancer Service

``````sh
kubectl run nginx-pod --image=nginx --port=80 --labels="app=nginx" --dry-run=client -oyaml > nginx-pod.yaml
kubectl apply -f nginx-pod.yaml

kubectl expose pod nginx-pod --port=8080 --target-port=80 --name=nginx-svc --type=NodePort --dry-run=client -oyaml > nginx-np-svc.yaml
----
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
      nodePort: 32000
----
On bear server:
- Create Target Groups - register 2 worker nodes and manaully add the nodeport of 32000
- Create application Loadbalancer - internet facing, register the target group with zones of the worker nodes
- Then test the load balancer with DNS Name on browser - lb-svc-493868679.eu-west-1.elb.amazonaws.com

``````
##### Endpoints object
A service is typically backed by a set of pods whose labels match the label selector defined in the Service object.

``````sh
kubectl describe svc kiada

kubectl get endpoints

----- # Inspecting an Endpoints object more closely
kubectl get ep kiada -oyaml


``````
##### Creating a service without a label selector

``````sh
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - name: http
    port: 80

``````
##### Creating an Endpoints object

``````sh
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
- addresses:
  - ip: 1.1.1.1
  - ip: 2.2.2.2
  ports:
  - name: http
    port: 88

``````
