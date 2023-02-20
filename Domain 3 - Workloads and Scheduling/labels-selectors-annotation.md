
##### Labels -  Maps (aka Dictionaries)
- Labels are attached to Kubernetes objects and are simple key: value pairs or maps(dictionary).
- Labels are used to store identifying information about a thing that you might need to query against.
- Labels are used for organization and selection of subsets of objects, and can be added to objects at creation time and/or modified at any time during cluster operations.
- You will see them on pods, replication controllers, replica sets, services, and so on.

``````sh
“labels”: {
“tier”: “frontend”
env: prod
}

``````

##### Selectors -  Maps (aka Dictionaries)
- Labels are queryable — which makes them especially useful in organizing things
-  A label selector is a string that identifies which labels you are trying to match
- You will see them on pods, replication controllers, replica sets, services, and so on.

``````sh
tier = frontend
tier != frontend
environment in (production, qa)
``````
##### Annotations
- Annotations are bits of useful information you might want to store about a pod (or cluster, node, etc.) that you will not have to query against.
- They are also key/value pairs and have the same rules as labels.
- Examples of things you might put there are the pager contact, the build date, or a pointer to more information someplace else—like a URL.

##### Method-1: Assign labels while creating a new object

``````sh
kubectl create deployment label-nginx-example --image=nginx --dry-run=client -oyaml > label-nginx-example.yml
-- # clean up the template and add a label app: prod

[root@controller ~]# cat label-nginx-example.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: prod
  name: label-nginx-example
spec:
  replicas: 2
  selector:
    matchLabels:
      app: prod
  template:
    metadata:
      labels:
        app: prod
    spec:
      containers:
      - image: nginx
        name: nginx
---
kubectl create -f label-nginx-example.yml

kubectl get deployments --show-labels

kubectl get pods --show-labels

``````
#####  Assign a new label to existing pod runtime as a patch

will assign new label "tier: frontend" to our existing Pods from the deployment label-nginx-example
``````sh
[root@controller ~]# cat update-label.yml
spec:
  template:
    metadata:
      labels:
        tier: frontend
--- #Next patch the deployment with this YAML file
kubectl patch deployment label-nginx-example --patch "$(cat update-label.yml)"

kubectl describe deployment label-nginx-example

kubectl get pods --show-labels

``````
#####  Method-3: Assign a new label to existing deployments runtime using kubectl

I have another deployment nginx-deploy on my cluster, so I will assign label tier: backend to this deployment:
``````sh
kubectl label deployment nginx-deploy tier=backend

kubectl get deployments --show-labels

``````
#####  Using labels to list resource objects

``````sh
kubectl get pods --show-labels

kubectl get deployments --show-labels

kubectl get all --show-labels

--- #To list all the deployments using type: dev label:
kubectl get deployments -l type=dev

kubectl get pods -l app=prod
``````

#####  Using Selector to list resource objects
I will create another deployment here with two labels and use one of the label as selector:
``````sh
[root@controller ~]# cat lab-nginx.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dev
    tier: backend
  name: lab-nginx
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
---
kubectl create -f lab-nginx.yml

kubectl get pods --show-labels

kubectl get pods --selector "app=dev"
kubectl get pods -l app=dev

kubectl get pods --selector "tier=backend"
kubectl get pods -l tier=backend
``````
#####  Removing labels

``````sh
kubectl label pod lab-nginx-58f9bf94f7-pc8zf app-

---
kubectl get pods --show-labels

kubectl get deployments --show-labels

kubectl label deployment lab-nginx app-

kubectl get deployments --show-labels
``````