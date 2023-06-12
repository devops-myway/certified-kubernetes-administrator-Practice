https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/

https://kubernetes.io/docs/concepts/policy/resource-quotas/#requests-vs-limits

#### Note:
- Kubernetes uses namespaces to organize objects in the cluster.
- You can think of each namespace as a folder that holds a set of objects.

Four namespaces are defined when a cluster is created:

- default: this is where all the Kubernetes resources are created by default

- kube-node-lease: an administrative namespace where node lease information is stored - may be empty and/or non-existing

- kube-public: a namespace that is world-readable. Generic information can be stored here but it's often empty

- kube-system: contains all infrastructure pods and  don’t poke around heavily in namespace.

##### List available namespaces

``````sh
kubectl get ns or kubectl get namespaces
kubectl get all --all-namespaces

``````
##### To list the pods from a specific namespace
you can also use --namespace instead of -n
``````sh
kubectl get pods -n default

``````
##### Creating a namespace
``````sh
kubectl create ns dev
kubectl get ns

kubectl create namespace test --dry-run=client -oyaml
--
kind: Namespace
apiVersion: v1
metadata:
  name: test
  labels:
    name: test
kubectl apply -f test.yaml
--
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: test
  labels:
    name: mypod
spec:
  containers:
  - name: mypod
    image: nginx
-----
kubectl apply -f pod.yaml --namespace=test
kubectl get pod -n test
``````
##### Get details of namespace
``````sh
kubectl describe ns default
kubectl get ns default -oyaml

``````
##### Create a Namespace with below manifest file
kubectl apply -f dev-namespace.yaml
kubectl apply -f tutorials-namespace.yaml
``````sh
vi dev-namespace.yaml

kind: Namespace
apiVersion: v1
metadata: 
 name: development
 labels: 
  name: development
-------
vi tutorials-namespace.yaml

kind: Namespace
apiVersion: v1
metadata: 
 name: tutorials
 labels: 
  name: tutorials

---
kubectl get namespaces --show-labels
``````

##### Set-Context to Switch between Namespaces using kubeconfig file.
A Kubernetes namespace provides the scope for Pods, Services, and Deployments in the cluster.

Users interacting with one namespace do not see the content in another namespace


``````sh
kubectl config view
kubectl config current-context

-------
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://130.211.122.180
  name: lithe-cocoa-92103_kubernetes
contexts:
- context:
    cluster: lithe-cocoa-92103_kubernetes
    user: lithe-cocoa-92103_kubernetes
  name: lithe-cocoa-92103_kubernetes
current-context: lithe-cocoa-92103_kubernetes
kind: Config
preferences: {}
users:
- name: lithe-cocoa-92103_kubernetes
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
    token: 65rZW78y8HbwXXtSXuUw9DbP4FLjHi4b
- name: lithe-cocoa-92103_kubernetes-basic-auth
  user:
    password: h5M0FtUUIflBSdI7
    username: admin

``````

Let's create contexts for our 2 Namespaces : dev and prod. The value of "cluster" and "user" fields are copied from the current context.
``````sh

kubectl config set-context dev --namespace=development --cluster=lithe-cocoa-92103_kubernetes --user=lithe-cocoa-92103_kubernetes

kubectl config set-context prod --namespace=production --cluster=lithe-cocoa-92103_kubernetes --user=lithe-cocoa-92103_kubernetes

# Let's view config again:
kubectl config view
``````
To get the current context, use: Let's create a Pod in this current context to see how it is kept separate / hidden from other contexts.
IMPORTANT : Note that nowhere in that YAML spec do we specify a Namespace to create this Pod in.
Kubernetes objects are automatically created in the current context.
``````sh
kubectl config current-context

vi myNamespacedPod-3.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-default-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The Pod is running ... now sleeping && sleep 3600']

-------
kubectl apply -f myNamespacedPod-3.yaml
kubectl get po 

``````
##### Create Pod in Dev Namespace
Let's switch context to development and create a Pod there.
all requests we make to the Kubernetes cluster from the command line are scoped to the development namespace.

``````sh
kubectl config use-context dev
kubectl config current-context
--------

vi myNamespacedPod-1.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: my-dev-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The Pod is running ... now sleeping && sleep 3600']
----
kubectl apply -f myNamespacedPod-1.yaml 
kubectl get po
``````
##### Create Pod in prod Namespace
``````sh
kubectl config use-context prod 

vi myNamespacedPod-2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-prod-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The Pod is running ... now sleeping && sleep 3600']
----
kubectl apply -f myNamespacedPod-2.yaml
kubectl get po
``````

##### Kubectl api-resources in Namespaces
Not all Kubernetes resources are in a Namespace.
Get a list of Kubernetes API resources in a Namespace:
``````sh
kubectl api-resources --namespaced=true

kubectl api-resources --namespaced=false

``````
### Clean up
delete all pods. Note each delete command will take 30 seconds since I did not force delete it immediately using --grace-period=0 --force  on the delete command.

``````sh
kubectl config use-context dev

kubectl delete pod/my-dev-pod

kubectl config use-context prod 

kubectl delete pod/my-prod-pod

``````

##### Cross Namespace communication
Namespaces are “hidden” from each other, but they are not fully isolated by default. A service in one Namespace can talk to a service in another Namespace.
you can use the built-in DNS service discovery and just point your app at the Service’s name. 
Services in Kubernetes expose their endpoint using a common DNS pattern. It looks like this:

``````sh
<Service Name>.<Namespace Name>.svc.cluster.local

``````
if you need to access a Service in another Namespace just use the Service name plus the Namespace name.
For example, if you want to connect to the “database” service in the “test” or production namespace, you can use the following address:

``````sh
database.test
database.production

``````