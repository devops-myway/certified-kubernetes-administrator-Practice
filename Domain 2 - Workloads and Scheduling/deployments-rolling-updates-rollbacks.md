https://kubernetes.io/docs/concepts/workloads/controllers/deployment/


##### Overview on Kubernetes Deployment
Kubernetes also provides Deployment resource that sits on top of ReplicaSets and enables declarative application updates.
- When running Pods in datacenter, additional features may be needed such as scalability, updates and rollback etc which are offered by Deployments
- A Deployment is a higher-level resource meant for deploying applications and updating them declaratively, instead of doing it through a ReplicationController or a ReplicaSet, which are both considered lower-level concepts.
- When you create a Deployment, a ReplicaSet resource is created underneath. Replica-Sets replicate and manage pods, as well.
- When using a Deployment, the actual pods are created and managed by the Deployment’s ReplicaSets, not by the Deployment directly

##### Create Kubernetes Deployment resource

In the deployment spec, following properties are managed:

- replicas: explains how many copies of each Pod should be running 
- strategy: explains how Pods should be updated
- selector: uses matchLabels to identify how labels are matched against the Pod
- template: contains the pod specification and is used in a deployment to create Pods
- scale deployment: kubectl scale deployment nginx-deployment --replicas 10
- set image : kubectl set image deployment nginx-deployment nginx=nginx:1.91 --record
- rollout undo: kubectl rollout undo deployment nginx-deployment

we will create a new deployment using kubectl using --dry-run so that actually a deployment is not created but just verified

``````sh
kubectl create deployment nginx-deploy --image=nginx --dry-run=client -o yaml > nginx-deploy.yml

-- Basic Template to cleanup
[root@controller ~]# vi nginx-deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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

---
kubectl apply -f nginx-deploy.yml
kubectl rollout status deployment/nginx-deployment
kubectl get pods
kubectl get rs
kubectl describe deploy/nginx-deployment
kubectl rollout history deploy/nginx-deployment
kubectl rollout history deploy/nginx-deployment --revision=2

``````
##### Using Kubernetes RollingUpdate

You have two ways of updating all those pods. You can do one of the following:

- Recreate: Delete all existing pods first and then start the new ones. This will lead to a temporary unavailability.
- RollingUpdate: Updates Pod one at a time to guarantee availability of the application. This is the preferred approach and you can further tune its behaviour.

The RollingUpdate strategy options are used to guarantee a certain minimal and maximal amount of Pods to be always available:

- maxUnavailable: The maximum number of Pods that can be unavailable during updating. The value could be a percentage (the default is 25%).
- maxSurge: The maximum number of Pods that can be created over the desired number of ReplicaSet during updating. the value of maxSurge cannot be 0
--
``````sh
vi rolling-nginx.yml
----
[root@controller ~]# vi rolling-nginx.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-nginx
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: rolling-nginx
  template:
    metadata:
      labels:
        app: rolling-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9
---
kubectl apply -f rolling-nginx.yml
kubectl get pods
kubectl get deployments
kubectl get event --field-selector involvedObject.name=rolling-nginx-74cf96d8bb-6f54x
``````
##### Check rollout history
But why CHANGE-CAUSE is showing NONE? It is because we have not used --record while creating our deployment. 
The --record argument will add the command under CHANGE-CAUSE for each revision history
``````sh
kubectl rollout history deployment rolling-nginx

kubectl delete deployment rolling-nginx
--
deployment.apps/rolling-nginx
REVISION  CHANGE-CAUSE
1         <none>

-- # this time I will use --record along with kubectl create:

 kubectl create -f rolling-nginx.yml --record

 kubectl rollout history deployment rolling-nginx

kubectl set image deployment rolling-nginx nginx=nginx:1.15 --record
``````
##### Monitor the rolling update status
To monitor the rollout status you can use:
``````sh
kubectl scale --replicas=4 deployment/rolling-nginx
kubectl set image deployment/rolling-nginx nginx=nginx:1.16 --record
kubectl rollout pause deployment/rolling-nginx
kubectl rollout status deployment rolling-nginx
kubectl get pods -l app=rolling-nginx
kubectl rollout history deployment/rolling-nginx
kubectl rollout undo deploy/rolling-nginx --to-revision=1

``````
##### Rolling back (undo) an update
To monitor the rollout status you can use:
``````sh
kubectl rollout history deployment/rolling-nginx
kubectl rollout undo deployment/rolling-nginx --to-revision=2
kubectl rollout status deployment rolling-nginx

``````
