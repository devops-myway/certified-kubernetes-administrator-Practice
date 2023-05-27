https://wangwei1237.github.io/Kubernetes-in-Action-Second-Edition/docs/Exposing_pods_via_services.html

##### Exposing Services to External Clients

A Kubernetes Service is an object you create to provide a single, stable access point to a set of pods that provide the same service.

A service can be backed by more than one pod. When you connect to a service, the connection is passed to one of the backing pods.

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
kubectl create deployment nginx-lab-1 --image=nginx --replicas=3 --dry-run=client -o yaml > nginx-lab-1.yml

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
``````
###### To create the service, you’ll tell Kubernetes to expose the Deployment you created earlier, here port 80 is the default port on which our nginx application would be listening on.
``````sh
kubectl expose deployment nginx-lab-1 --type=NodePort --port=80
--
kubectl expose pod quiz --name quiz
---
[root@controller ~]# kubectl describe svc nginx-lab-1
Name:                     nginx-lab-1
Namespace:                default
Labels:                   app=dev
Annotations:              <none>
Selector:                 app=dev
Type:                     NodePort
IP:                       10.108.252.53
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32481/TCP
Endpoints:                10.36.0.1:80,10.44.0.1:80,10.44.0.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

``````

##### Accessing cluster-internal services
- The ClusterIP services you created in the previous section are accessible only within the cluster, from other pods and from the cluster nodes.
- use the kubectl exec command to run a command like curl in an existing pod and get it to connect to the service.

To use the service from a pod, run a shell in the quote-001
 In my case, the quiz service uses cluster IP 10.96.136.190, whereas the quote service uses IP 10.96.74.151
``````sh
kubectl exec -it quote-001 -c nginx -- sh

 curl http://10.96.136.190

 curl http://10.96.74.151
``````

##### Access container outsusteride the cl
Now to access the container externally from the outside network we can use the public IP of individual worker node along with the NodePort

curl https://<PUBLIC-IP>:<NODE-PORT>
``````sh
kubectl get pods -o wide

``````
##### Creating a service through a YAML descriptor

``````sh
[root@controller ~]# cat nginx-deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dev
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dev
  template:
    metadata:
      labels:
        app: dev
    spec:
      containers:
      - image: nginx
        name: nginx
--
kubectl create -f nginx-deploy.yml

kubectl get pods -o wide

``````

``````sh
##### Creating a NodePort service
--
[root@controller ~]# cat nginx-service.yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-deploy
  labels:
    app: dev
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: dev
-- # another example of nodeport service
apiVersion: v1
kind: Service
metadata:
  name: kiada
spec:
  type: NodePort
  selector:
    app: kiada
  ports:
  - name: http
    port: 80
    nodePort: 30080
    targetPort: 8080
  - name: https
    port: 443
    nodePort: 30443
    targetPort: 8443
----
kubectl create -f nginx-service.yml
kubectl get svc
kubectl get svc --show-labels

kubectl describe service nginx-deploy

``````
##### Accessing a NodePort service

looking at the INTERNAL-IP and EXTERNAL-IP columns
``````sh
kubectl get nodes -o wide
curl http://node_ip_address:32096

curl 172.18.0.4:30080

``````

##### Access container inside the cluster
``````sh
kubectl exec nginx-deploy-58f9bf94f7-4cwlr -- curl -s http://10.110.95.181
``````
##### Access container outside the cluster
``````sh
kubectl get svc

Then try to access the pod using public IP of the respective worker node:
``````

##### Delete Kubernetes Service
``````sh
kubectl get svc

kubectl delete service nginx-deploy
``````
##### Listing services
When you create a service, it’s assigned an internal IP address that any workload running in the cluster can use to connect to the pods that are part of that service

``````sh
kubectl get svc -o wide

``````
##### Creating a LoadBalancer service


``````sh
apiVersion: v1
kind: Service
metadata:
  name: kiada
spec:
  type: LoadBalancer
  selector:
    app: kiada
  ports:
  - name: http
    port: 80
    nodePort: 30080
    targetPort: 8080
  - name: https
    port: 443
    nodePort: 30443
    targetPort: 8443

Apply the manifest with kubectl apply

kubectl get svc kiada

``````
##### Endpoints object
A service is typically backed by a set of pods whose labels match the label selector defined in the Service object.

``````sh
kubectl describe svc kiada

kubectl get endpoints

----- # Inspecting an Endpoints object more closely
kubectl get ep kiada -o yaml


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