##### The API server
is the central management entity and the only component that talks directly with the distributed storage component etcd.

The API server has the following core responsibilities:

- To serve the Kubernetes API. This API is used cluster-internally by the master components, the worker nodes, and your Kubernetes-native apps, as well as externally by clients such as kubectl.
- To proxy cluster components, such as the Kubernetes dashboard, or to stream logs, service ports, or serve kubectl exec sessions.

Serving the API means:

Reading state: getting single objects, listing them, and streaming changes
Manipulating state: creating, updating, and deleting objects

To check your API server on the controller with below command.
The API server is stateless (that is, its behavior will be consistent regardless of the state of the cluster) and is designed to scale horizontally

``````sh
kubectl get pods -n kube-system

``````
##### Kubernetes HTTP Request Flow

The kubectl command is translated into an HTTP API request in JSON format and is sent to the API server. 
Then, the API server returns a response to the client, along with any requested information.

Authentication:
- In Kubernetes, every API call needs to authenticate with the API server, regardless of whether it comes from outside the cluster, such as those made by kubectl, or a process inside the cluster, such as those mad by kubelet.
- When an HTTP request is sent to the API server, the API server needs to authenticate the client sending this request.
- The HTTP request will contain the information required for authentication, such as the username, user ID, and group.
- The authentication method will be determined by either the header or the certificate of the request.
- To deal with these different methods, the API server has different authentication plugins, such as ServiceAccount tokens, which are used to authenticate ServiceAccounts, and at least one other method to authenticate users, such as X.509 client certificates.

Authorization:
- After authentication is successful, the attributes from the HTTP request are sent to the authorization plugin to determine whether the user is permitted to perform the requested action

#### Create a Service Account using service-account.yml file with following content:

``````sh
[root@controller ~]# cat service-account.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: read-only-user
  namespace: default

``````
#### create a ClusterRole object and assign it some permissions
 ClusterRole with the ability to list all the Pods in any namespace, but nothing else

``````sh
[root@controller ~]# cat cluster-role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  namespace: default
  name: read-only-user-cluster-role
rules:
  - verbs:
      - "list"
    apiGroups:
      - ""
    resources:
      - "pods"

``````
####  ClusterRoleBinding object that will bind the created ServiceAccount and ClusterRole
 ClusterRole with the ability to list all the Pods in any namespace, but nothing else
 
``````sh
[root@controller ~]# cat cluster-role-binding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-only-user-cluster-role-binding
  namespace: default
roleRef:
  name: read-only-user-cluster-role
  kind: ClusterRole
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: read-only-user
    namespace: default

``````
####  Run the following command to create this RBAC policy, as well as our ServiceAccount
 
``````sh
kubectl apply -f service-account.yml -f cluster-role.yml -f cluster-role-binding.yml

# Now I am will use my root user which has Cluster Admin privilege to list the pods:

kubectl get pods --all-namespaces

# kubectl provides a tool that you can call by using kubectl auth can-i to check whether an action is allowed for the current user.

 kubectl auth can-i get pods --all-namespaces

``````
####  Kubernetes API Terminology
 
``````sh
kubectl api-resources

kubectl api-versions

``````
####  Access API using curl
 kubectl has a great feature called kubectl proxy, which is the recommended approach for interacting with the API server
 custom port number 8080 while adding an & (ampersand) sign at the end of our command to allow the proxy to run in the terminal background so that we can continue working in the same terminal window
``````sh
kubectl proxy --port=8080 &

~]# curl http://127.0.0.1:8080/apis | less

``````