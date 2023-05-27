https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
https://www.containiq.com/post/kubernetes-namespaces

###### Note:
- Kubernetes uses namespaces to organize objects in the cluster.
- You can think of each namespace as a folder that holds a set of objects.

Four namespaces are defined when a cluster is created:

default: this is where all the Kubernetes resources are created by default
kube-node-lease: an administrative namespace where node lease information is stored - may be empty and/or non-existing
kube-public: a namespace that is world-readable. Generic information can be stored here but it's often empty
kube-system: contains all infrastructure pods and  don’t poke around heavily in namespace.

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
In order to easily switch between Namespaces we use the concept of switching context.

Let's investigate our current context on our cluster info:
``````sh
kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority: C:\Users\alwyn\.minikube\ca.crt
    server: https://192.168.99.117:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: C:\Users\alwyn\.minikube\client.crt
    client-key: C:\Users\alwyn\.minikube\client.key

``````
To see the current context we are working in, use
Let's create contexts for our 2 Namespaces : development and tutorials.
``````sh
kubectl config current-context
kubectl config set-context development --namespace=development --cluster=minikube --user=minikube
kubectl config set-context tutorials --namespace=tutorials --cluster=minikube --user=minikube

# Let's view config again:
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
  name: my-minikube-pod
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
##### Create Pod in Development Namespace
Let's switch context to development and create a Pod there.
Only this one Pod that I created in this Namespace is listed.
``````sh
kubectl config use-context development 

vi myNamespacedPod-1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-development-pod
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
##### Create Pod in Tutorials Namespace
``````sh
kubectl config use-context tutorials 

vi myNamespacedPod-2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-tutorials-pod
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
##### Using Namespaces
To separate your Kubernetes resources just switch context and do your work there. Objects automatically created in your current context.
Each context only shows its single Pod.
``````sh
kubectl config use-context tutorials
kubectl get po

kubectl config use-context development
kubectl get po

kubectl config use-context minikube
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
kubectl config use-context minikube

kubectl delete pod/my-minikube-pod

kubectl config use-context development 

kubectl delete pod/my-development-pod

kubectl config use-context tutorials  

kubectl delete pod/my-tutorials-pod

``````

##### Assign Resource Quota To Namespace
assign some resource quota limits to our newly created namespace. This will make sure the pods deployed in this
namespace will not consume more system resources than mentioned in the resource quota.

``````sh
# Create a file named resourceQuota.yaml
---------
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-quota
  namespace: deployment-demo
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
-------

kubectl create -f resourceQuota.yaml

kubectl describe ns deployment-demo

``````
##### Kubernetes namespaces and the kubectl context
``````sh
kubectl config set-context --current --namespace=<insert-namespace-name-here>
kubectl config view --minify | grep namespace

``````
##### Namespaces and Resource Quotas
You can implement Resource Quotas on a namespace so that cluster resources are allocated the way you need them to be.
``````sh
apiVersion: v1
kind: ResourceQuota
metadata:
  name: test-cpu-quota
  namespace: demo
spec:
  hard:
    requests.cpu: "200m"  
    limits.cpu: "300m"  

``````