https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
https://kubernetes.io/blog/2020/04/two-phased-canary-rollout-with-gloo/
https://earthly.dev/blog/canary-deployment-in-k8s/

##### Kubernetes Deployment Strategies
Kubernetes deployment strategies are the way that pods are updated, downgraded or created into new versions of applications.
Kubernetes deployment strategies work by replacing pods of previous versions of your application with pods of the new version.
The main benefits of these Kubernetes deployment strategies are that it mitigates the risk of disruptions and downtime of services.


###### canary deployment
A canary deployment is an improved iteration of an existing deployment that includes all the necessary dependencies and application code. It exists alongside the original deployment and allows you to compare the behavior of different versions of your application side by side. It helps you test new features on a small percentage of users with minimal downtime and less impact on user experience.


#### How You Can Use Canary Deployments in Kubernetes

At its core, a canary deployment implements a clone of your production environment with a load balancer routing user traffic between the available environments based on your parameters.

Once added to your Kubernetes cluster, the canary deployment will be controlled by services using selectors and labels. This service provides or forwards traffic to the labeled Kubernetes environment or pod, making it simple to add or remove deployments.

Initially, you can have a specific percentage of users test the modified application, compare both the application deployments, and increase the user percentage as your monitoring and user tests produce no errors. This percentage can be gradually increased until all the users have tested the newer version of the application, and then the older version can be taken offline.

###### setWeight steps
The Rollout makes a best effort attempt to achieve the percentage listed in the last setWeight step between the new and old version.
- In the case where the setWeight is 15% or 25%, the Rollout attempts to get there by rounding up the calculation (i.e. the new ReplicaSet has 2 pods since 15% of 10 rounds up to 2 and the old ReplicaSet has 9 pods since 85% of 10 rounds up to 9).

##### Example 1
CNBC broadcasting team need to test stability of deployment for two different software image version.
- Task 1-
Create a namespace cnbc, and make sure pod count in the namespace is not more than 10
Create a deployment (cnbc-1) with image nginx:1.14.2
Create a cluster IP service, which will drive traffic to the deployment
Update  deployment to (cnbc-2)with image nginx:latest , using canary deployment update strategy
Add labels to deployment as upgrade=canary
Redirect 60% of the incoming traffic to old deployment  and 40% of the incoming traffic to new deployment
``````sh
vi pod1.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: cnbc
---
vi dep1.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cnbc-1
  namespace: cnbc
  labels:
    app: nginx
spec:
  replicas: 6
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl apply -f dep1.yaml
``````
##### Create the Service Object

``````sh
vi svc1.yaml

apiVersion: v1
kind: Service
metadata:
  name: cnbc-svc
  namespace: cnbc
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  selector:
    app: nginx
---
kubectl apply -f svc1.yaml
kubectl get svc
kubectl get all -n echo

``````

##### Create a Canary Deployment

``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cnbc-2
  namespace: cnbc
  labels:
    app: nginx
    upgrade: canary
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
      upgrade: canary
  template:
    metadata:
      labels:
        app: nginx
        upgrade: canary
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
                                    
---
k apply -f dep2.yaml
k get pods -o wide

``````
###### Example 2
Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

``````sh

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-1 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
``````
##### Service Object


``````sh
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  labels:
    app: my-app
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: my-app
``````
##### Canary Rollout

``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-2 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
``````
##### Test the Rollout
``````sh

``````
``````sh

``````