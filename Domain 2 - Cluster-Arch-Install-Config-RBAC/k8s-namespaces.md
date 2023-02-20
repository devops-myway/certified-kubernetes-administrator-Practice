https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

###### Note:
- Kubernetes uses namespaces to organize objects in the cluster.
- You can think of each namespace as a folder that holds a set of objects.
- Resource limitation through quota can be implemented at a Namespace level also
- Use namespaces to separate customer environments within one Kubernetes cluster
- By default, the kubectl command-line tool interacts with the default namespace.
- If you want to use a different namespace, you can pass kubectl the --namespace flag.
- For example, kubectl --namespace=mystuff references objects in the mystuff namespace.
- If you want to interact with all namespaces - for example, to list all Pods in your cluster you can pass the --all-namespaces flag

Four namespaces are defined when a cluster is created:

default: this is where all the Kubernetes resources are created by default
kube-node-lease: an administrative namespace where node lease information is stored - may be empty and/or non-existing
kube-public: a namespace that is world-readable. Generic information can be stored here but it's often empty
kube-system: contains all infrastructure pods

##### List available namespaces

``````sh
kubectl get ns
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
kubectl get ns default -o yaml

``````
##### Create resource objects in other namespaces
``````sh
[root@controller ~]# cat nginx-app.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  namespace: app
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80


kubectl create -f nginx-app.yml

kubectl get pods -n app nginx-app

``````
##### Using kubectl command to create resource objects in other namespace
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx-dev
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

#We will create this Pod inside "dev" namespace:
kubectl create -f nginx-dev.yml -n dev

kubectl get pods -n dev

``````
##### Deleting a Pod using name
``````sh
kubectl delete pod nginx-dev

kubectl delete ns dev

kubectl delete pods -n app --all

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