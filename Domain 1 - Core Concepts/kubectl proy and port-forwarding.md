
https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/


##### Notes:
kubectl port-forward:
In the context of developing applications on Kubernetes, it is often useful to quickly access a service from your local environment without exposing it using, for example, a load balancer or an ingress resource. forwards connections to a local port to a port on a pod. is more generic as it can forward TCP traffic.

kubectl port-forward <resource-type/resource-name> [local_port_o_choice]:<pod_taregt_port>


##### Perform kubectl port-forward in background

To perform kubectl port-forward in background you can append the command with &

``````sh
kubectl port-forward pods/my-pod 8080:80 &

``````
##### Perform port-forwarding on Pods MongoDB deployment and service

``````sh
kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/main/specs/pf/app.yaml
kubectl get svc -owide
kubectl port-forward service/simpleservice 8080:80
          
``````
Forward a local port to a port on the Pod
``````sh
curl localhost:8080/info
---- #output
{"host": "localhost:8080", "version": "0.5.0", "from": "127.0.0.1"}
``````

##### kubectl proxy
kubectl proxy can only forward HTTP traffic.
The easiest and most common way to access the cluster is through kubectl proxy. This creates a local web server that securely proxies data to the dashboard through the Kubernetes API server.

Run a proxy to the Kubernetes API server on an arbitrary local port # The chosen port for the server will be output to stdout
``````sh
kubectl proxy --port=8081
          
``````
