https://kubernetes.io/docs/concepts/workloads/controllers/deployment/


##### Overview on Kubernetes Deployment
Kubernetes also provides Deployment resource that sits on top of ReplicaSets and enables declarative application updates.
- When running Pods in datacenter, additional features may be needed such as scalability, updates and rollback etc which are offered by Deployments
- A Deployment is a higher-level resource meant for deploying applications and updating them declaratively, instead of doing it through a ReplicationController or a ReplicaSet, which are both considered lower-level concepts.
- When you create a Deployment, a ReplicaSet resource is created underneath. ReplicaSets replicate and manage pods, as well.
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
Important commands to pay attention:
kubectl set -h
kubectl rollout -h

we will create a new deployment using kubectl using --dry-run so that actually a deployment is not created but just verified

``````sh
kubectl create deployment nginx-deploy --image=nginx --replicas=3 --dry-run=client -o yaml > nginx-deploy.yml

cat << EOF | kubectl apply -f -
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
EOF
---

kubectl rollout status deployment/nginx-deployment
kubectl get pods
kubectl get rs
kubectl describe deploy/nginx-deployment
kubectl rollout history deploy/nginx-deployment
kubectl rollout history deploy/nginx-deployment --revision=2

``````
