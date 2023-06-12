https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

##### Kubernetes Deployment Strategies
Kubernetes deployment strategies are the way that pods are updated, downgraded or created into new versions of applications.
Kubernetes deployment strategies work by replacing pods of previous versions of your application with pods of the new version.
The main benefits of these Kubernetes deployment strategies are that it mitigates the risk of disruptions and downtime of services.


###### canary deployment
A canary deployment is an upgraded version of an existing deployment, with all the required application code and dependencies. It is used to test out new features and upgrades to see how they handle the production environment. This is done by creating a new replica set with the updated version of the software while keeping the original replica set running.

If issues are detected during the canary deployment, it can be quickly rolled back to the original replica set. When you add the canary deployment to a Kubernetes cluster, it is managed by a service through selectors and labels.

The amount of traffic that the canary gets corresponds to the number of pods it spins up. In most cases, you start by routing a smaller percentage of traffic to the canary and increase the number over time

Canary deployments are a valuable tool for minimizing risk and ensuring high availability in complex distributed systems, by allowing for controlled testing of changes before they are released to the entire system.

#### Example 1.0 
We created 3 replicas of Nginx pods for the Kubernetes cluster. All the pods have the label version: "1.0"

``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
 name: nginx
spec:
 selector:
  matchLabels:
   app: nginx
 replicas: 3
 template:
  metadata:
   labels:
    app: nginx
    version: "1.0"
  spec:
   containers:
    - name: nginx
      image: nginx:alpine
      resources:
      limits:
       memory: "128Mi"
       cpu: "50m"
      ports:
      - containerPort: 80
      volumeMounts:
      - mountPath: /usr/share/nginx/html
        name: index.html
      volumes:
      - name: index.html
        hostPath:
          path: /Users/sofija/Documents/nginx/v1
----
k apply -f nginx-deployment.yaml
k get pods -o wide
``````
##### Create the Service
The type of service – LoadBalancer. It instructs the service to balance workloads between pods with the labels app: nginx and version: "1.0".

``````sh
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
spec:
 type: LoadBalancer
selector:
 app: nginx
 version: "1.0"
ports:
- port: 8888
  targetPort: 80
---
kubectl apply -f nginx-deployment.service.yaml

``````
##### Check First Version of Cluster

``````sh
http://localhost:8888
``````
##### Create a Canary Deployment
With version 1 of the application in place, you deploy version 2, the canary deployment.
The name in the metadata is nginx-canary-deployment.
It has the label version: “2.0”.
It is linked to a html file index.html which consists of:

``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
 name: nginx-canary-deployment
spec:
 selector:
  matchLabels:
   app: nginx
 replicas: 3
 template:
  metadata:
   labels:
    app: nginx
    version: "2.0"
  spec:
   containers:
    - name: nginx
      image: nginx:alpine
      resources:
      limits:
       memory: "128Mi"
       cpu: "50m"
      ports:
      - containerPort: 80
      volumeMounts:
      - mountPath: /usr/share/nginx/html
        name: index.html
      volumes:
      - name: index.html
        hostPath:
          path: /Users/sofija/Documents/nginx/v2
---
k apply -f nginx-canary-deployment.yaml
k get pods -o wide
``````
###### create the Service file
To test out the updated pods, you need to modify the service file and direct part of the traffic to version: "2.0".

Find and remove the line version: “1.0”. The file should include the following:
``````sh
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
spec:
 type: LoadBalancer
selector:
 app: nginx
ports:
- port: 8888
  targetPort: 80
--
k apply -f nginx-deployment.service.yml
``````
##### Roll Back Canary Deployment
If you notice the canary is not performing as expected, you can roll back the deployment and delete the upgraded pods with:

``````sh
kubectl delete deployment.apps/nginx-canary-deployment

``````
##### Roll Out Upgraded Deployment
If you conclude the canary deployment is performing as expected, you can route all incoming traffic to the upgraded version. There are three ways to do so:

``````sh
# Upgrade the first version by modifying the Docker image and building a new deployment
kubectl delete deployment.apps/nginx-canary-deployment

#  You can keep the upgraded pods and remove the ones with the version 1 label:
kubectl delete deployment.apps/nginx

``````