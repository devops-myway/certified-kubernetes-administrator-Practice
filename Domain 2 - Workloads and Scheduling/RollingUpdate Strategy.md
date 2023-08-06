

##### Using Kubernetes RollingUpdate strategy

You have two ways of updating all those pods. You can do one of the following:

- Recreate: Delete all existing pods first and then start the new ones. This will lead to a temporary unavailability.
- RollingUpdate: Updates Pod one at a time to guarantee availability of the application. This is the preferred approach and you can further tune its behaviour.

The RollingUpdate strategy options are used to guarantee a certain minimal and maximal amount of Pods to be always available:

- maxUnavailable: The maximum number of Pods that can be unavailable during updating. The value could be a percentage (the default is 25%).
- maxSurge: The maximum number of Pods that can be created over the desired number of ReplicaSet during updating. the value of maxSurge cannot be 0
--
``````sh
cat << EOF | kubectl apply -f -
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
EOF
---
kubectl get pods
kubectl get deploy
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

 kubectl apply -f rolling-nginx.yml --record
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
