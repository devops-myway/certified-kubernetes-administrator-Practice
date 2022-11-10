 ##### Document Links:
 https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/#context

 ### Notes about Kubernetes Context:
 context is an entity defined inside kubeconfig file to alias cluster parameters with a human-readable name. It’s important to understand that a Kubernetes context only applies to the client side.

 You can think of Kubernetes contexts as a kind of shortcut that allows you to access cluster, user, and namespace parameters conveniently.

 This allows you to define multiple contexts in your configuration file, which you can then use to target multiple Kubernetes clusters, or the same cluster with a different set of users or namespaces.

 ###### Typically there are three parts to a kubeconfig:
- Clusters: This section lists all the clusters that you have access to and it details about the URL for the Kubernetes API server (where the cluster accepts commands from kubectl), certificate authority, and a name to identify the cluster.

- Contexts: Each context consists of the cluster name, user, and namespace that you can access when you invoke it. Note that in the example, the namespace information is omitted because it’s using the default namespace.

- Users: The users section identifies each user by a unique name and includes the relevant authentication information, which could be client certificates (as is the case here), bearer tokens, or an authenticating proxy.

 ####

 ### Commands Below:

 ##### all context commands
    kubectl config --help

##### show all contexts
    kubectl config get-contexts

##### show current context
    kubectl config current-context

##### switch to another context
    kubectl set-context context-name

##### change user or cluter name for any context
    kubectl config set-context --help
    kubectl config set-context context-name --user=user_name --cluster=cluster_name

##### change user or cluster for current context
    kubectl config set-context --current --user=user_name --cluster=cluster_name

##### change default namespace
    kubectl get pod
    kubectl config set-context --current --namespace=myapp
    kubectl get pod

##### Create a new context example

``````sh
# to create a new context for development called “dev-context”, which points to the namespace “dev-namespace”, and the user “dev-user”

kubectl config set-context \
dev-context \
--namespace=dev-namespace \
--cluster=docker-desktop \
--user=dev-user

``````

##### modifying your kubeconfig
kubectl config view

##### Multiple kubeconfig file 

````sh
# kubeconfig-multiple-context.yaml

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: xxxxxx
    server: https://172.31.44.88:6443
  name: development
- cluster:
    certificate-authority-data: xxxxxx
    server: https://251.15.20.12:6443
  name: staging  
contexts:
- context:
    cluster: development
    user: dev-admin
  name: dev-admin@development
- context:
    cluster: development
    namespace: myapp
    user: my-script
  name: my-script@development
- context:
    cluster: staging
    user: staging-admin
  name: staging-admin@staging
current-context: dev-admin@development
kind: Config
preferences: {}
users:
- name: dev-admin
  user:
    client-certificate-data: xxxx
    client-key-data: xxxx
- name: staging-admin
  user:
    client-certificate-data: xxxx
    client-key-data: xxxx 
- name: my-script
  user:
    token: xxxxxxx

````