##### Notes:

Services in the cluster can easily make requests of other services in the cluster, but for an external service to communicate with the cluster resources, it needs to pass through authentication and authorization steps.

The kubectl proxy command:

It creates a communication channel between your local machine and your Kubernetes API server by reading the cluster configuration and credentials from your kube-config file.

##### How Does kubectl proxy Work

A proxy has been started, and now you can directly access the Kubernetes API by sending requests to http://127.0.0.1:8080/api
``````sh
kubectl proxy --port=8080

# Here’s a curl command to fetch the details of a pod named “nginx”

curl http://localhost:8080/api/v1/namespaces/default/pods/nginx
``````
##### Exploring kubectl proxy in detail
address: This option helps you to change the IP address of the exposed proxy. Its default value is 127.0.0.1 (i.e. localhost)

port: It enables you to choose which port the proxy will be exposed from. If you do not provide this option, port 8001 will be used for exposing the proxy.

``````sh
kubectl proxy --address="127.0.0.1"

kubectl proxy --port=81

``````