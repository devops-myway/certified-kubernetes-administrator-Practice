https://kubernetes.io/docs/concepts/services-networking/service/
https://minikube.sigs.k8s.io/docs/handbook/accessing/

##### Exposing Services to External Clients

A Kubernetes Service is a method for exposing a network application that is running and passing connection to one or more backing pods in your cluster.

Add labels to Pod objects and specify the label selector in the Service object. The pods whose labels match the selector are part of the service registered service endpoints.

- The shorthand for services is svc

###### Service Pods

In a Kubernetes Cluster there are three group of PODs present as following :

Frontend PODs: [Pods which are running the frontend part of the web application on them]
Backend PODs: [Pods which are running the backend part of the web application on them]
Database PODs: [Pods which are running the database application instances on them]

##### How do the pods communicate with eachother

There are three sets of PODs [frontend, backend & database]

We created two ClusterIP services (back-end & redis) to ease out the process of inter pod communication.

#### Understanding different Kubernetes Service Types

ClusterIP: It exposes the service within the Kubernetes cluster. This Service is only reachable from within the cluster. It can not be accessed from outside the cluster.

NodePort: It will expose the service on a static port on the deployed node. This service can be accessed from outside the cluster using the NodeIP:Nodeport.

LoadBalancer: Exposes the Service externally using a cloud provider's load balancer, this creates a Public IP on the Cloud provider

ExternalName: which is a relatively new object that works on DNS names and redirection is happening at the DNS level.
Service without selector: which is used for direct connections based on IP port combinations without an endpoint. And this is useful for connections to a database or between namespaces.

#### Using kubectl expose command
Create a pod with image nginx called nginx and expose its port 80.
Confirm that ClusterIP has been created. Also check endpoints
Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget.
Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

``````sh
kubectl run nginx --image=nginx
kubectl expose po/nginx --port=80 --name=svc-nginx --type=ClusterIP
kubectl get svc/svc-nginx -owide
kubectl get ep svc-nginx
kubectl run testpod --image=busybox -- sh -c 'sleep 3600'
kubectl exec -it testpod -- sh
wget -O- 10.101.191.158

``````

``````sh

vi nginx1.yaml

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
kubectl expose deploy/nginx1 --port=80 --target-port=80 --name=nginx1-svc --type=NodePort
kubectl create service nodeport nginx-svc --tcp=80:80 --dry-run=client -oyaml > svc1.yaml
kubectl apply -f svc1.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx-svc
  name: nginx-svc
spec:
  ports:
  - name: nginx-svc
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30000
  selector:
    app: nginx
  type: NodePort
status:
  loadBalancer: {}
--
kubectl apply -f nginx1.yml
kubectl get deploy/nginx1
kubectl get svc/nginx1svc
kubectl get pods
kubectl get nodes -owide
---
the nginx application can be accessed through this service on NodeIp:NodePort
 open another terminal 
 curl nopeip:nodeport
 curl 172.30.1.2:31977
``````
###### Ex2 expose a pod to nodeport and loadbalancer type services
``````sh
kubectl run nginx-pod --image=nginx --port=80 --labels="app=nginx" --dry-run=client -oyaml > nginx-pod.yaml
kubectl apply -f nginx-pod.yaml 

kubectl expose pod/nginx-pod --port=8080 --target-port=80 --name=nginx-svc --type=NodePort --dry-run=client -oyaml > nginx-np-svc.yaml
vi nginx-np-svc.yaml
manually add = nodePort: 30001
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
      nodePort: 30001
----
kubectl apply -f nginx-np-svc.yaml
kubectl describe svc/nginx-svc
kubectl get po
kubectl get node -owide
curl 172.30.1.2:30001 # To access the server content on NodeIP: NodePort
``````

##### Accessing LoadBalancer Service

``````sh
kubectl run nginx2pod --image=nginx --port=80 --labels="app=nginx" --dry-run=client -oyaml > nginx2pod.yaml
kubectl apply -f nginx2pod.yaml

kubectl expose pod/nginx2pod --port=8080 --target-port=80 --name=nginx2svc --type=LoadBalancer --dry-run=client -oyaml > nginx2svc.yaml
----
apiVersion: v1
kind: Service
metadata:
  name: nginx2svc
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
Manually Test without LoadBalance:
kubectl get nodes
kubectl get nodes -owide
kubectl get ep/nginx2svc
curl 172.30.1.2:32238
----
On bear server:
- Create Target Groups - register 2 worker nodes and manaully add the nodeport of 32000
- Create application Loadbalancer - internet facing, register the target group with zones of the worker nodes
- Then test the load balancer with DNS Name on browser - lb-svc-493868679.eu-west-1.elb.amazonaws.com

---- using minikube to Test load balancer:

# in separate terminal run
minikube tunnel

# working terminal run
kubectl get svc
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
 hello-minikube1   LoadBalancer   10.96.184.178   127.0.0.1   8080:30791/TCP   40s
http://REPLACE_WITH_EXTERNAL_IP:8080


``````
##### Endpoints object
A service is typically backed by a set of pods whose labels match the label selector defined in the Service object.

``````sh
kubectl describe svc/service_name
kubectl get endpoints/svc_name

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
