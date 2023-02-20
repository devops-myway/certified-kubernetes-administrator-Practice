##### Notes:
When the kubectl expose command is run, it takes the name of the resource, the port, the target port that should be used to expose, the protocol over which to expose, and the type of the service.

kubectl expose uses selectors to expose the resources to a given port. resources that can be exposed using a service are pods, deployments, replica sets, and replication controllers.

##### Using kubectl Expose

Start by creating two deployments that need to interact with each other.
Note that the curl command can be used to call an IP service from inside a container.

``````sh
#To create the internal deployment with two pods of nginx:latest image. This will be accessible only from inside the cluster

kubectl create deployment internal-deployment --image=nginx --replicas=2
kubectl get

#To make an external deployment with three pods of the nginx:latest image. This deployment will be available from outside the cluster.

kubectl create deployment external-deployment --image=nginx --replicas=3
kubectl get

kubectl get pods
``````
``````sh
#Expose the internal-deployment using ClusterIP

kubectl expose deployment internal-deployment --port=80 --target-port=8008 --name=internal-service

kubectl describe service internal-service

#To access the pods with ClusterIP, you can run a BusyBox curl container, then run nslookup for the internal service

kubectl run curl --image=radial/busyboxplus:curl -i --tty

$ ns lookup internal-service

# exit the pod using Ctrl+D

``````

``````sh
# To expose the external deployment as a NodePort service or LoadBalancer,

kubectl expose deployment external-deployment --port=80 --target-port=8000 --name=external-service --type=NodePort

kubectl describe service external-service

kubectl expose deployment external-deployment --port=80 --target-port=8000 --name=lb-service --type=LoadBalancer

kubectl describe service lb-service
``````