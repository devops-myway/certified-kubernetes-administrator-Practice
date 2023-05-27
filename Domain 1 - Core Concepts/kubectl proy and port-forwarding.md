
https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/


##### Notes:
kubectl port-forward:
forwards connections to a local port to a port on a pod. is more generic as it can forward TCP traffic.
Port forwarding mostly used for the purpose of getting access to internal cluster resources and debugging.
sing port forwarding you could get on your ‘localhost’ any services launched in your cluster.

kubectl port-forward <resource-type resource-name> [local_port]:<pod_port>


##### Perform kubectl port-forward in background

To perform kubectl port-forward in background you can append the command with &

``````sh
kubectl port-forward pods/my-pod 8080:80 &

``````
##### Perform port-forwarding on Pods MongoDB deployment and service

``````sh
# vim nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
 namespace: default
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
--
kubectl apply -f nginx.yaml
kubectl get pods
kubectl get pods -o wide
          
``````
Forward a local port to a port on the Pod
``````sh
kubectl port-forward mongo-75f59d57f4-4nd6q 28015:27017
kubectl port-forward deployment/mongo 28015:27017
kubectl port-forward service/mongo 28015:27017
-------- # output is similar to below:
Forwarding from 127.0.0.1:28015 -> 27017
Forwarding from [::1]:28015 -> 27017
          
``````
##### Method-2: For example, if you have Redis installed in the cluster on 6379, by using a command like this:

``````sh
kubectl port-forward redis-master-765d459796-258hz 7000:6379
          
``````
##### kubectl proxy
kubectl proxy can only forward HTTP traffic.
The easiest and most common way to access the cluster is through kubectl proxy. This creates a local web server that securely proxies data to the dashboard through the Kubernetes API server.

Run a proxy to the Kubernetes API server on an arbitrary local port # The chosen port for the server will be output to stdout
``````sh
kubectl proxy --port=8081
          
``````