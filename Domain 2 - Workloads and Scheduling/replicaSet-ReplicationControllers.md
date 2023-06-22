
##### Overview on Replication Controllers
- A ReplicationController is a Kubernetes resource that ensures its pods are always kept running.
- If the pod disappears for any reason, such as in the event of a node disappearing from the cluster or because the pod was evicted from the node, the ReplicationController notices the missing pod and creates a replacement pod.
- The ReplicationController in general, are meant to create and manage multiple copies (replicas) of a pod

A ReplicationController has three essential parts:

- A label selector: which determines what pods are in the ReplicationController’s scope
- A replica count: which specifies the desired number of pods that should be running
- A pod template: which is used when creating new pod replicas

##### Creating a replication controller]
To get the kind and apiVersion of replication controller we will check list of api-resources
Now we have the kind and apiVersion value needed to create our first replication controller.

``````sh
kubectl api-resources
----

[root@controller ~]# cat replication-controller.yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: dev
spec:
  replicas: 3
  selector:
    app: myapp
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: dev
    spec:
      containers:
      - name: nginx-container
        image: nginx
--
kubectl create -f replication-controller.yml

kubectl get pods
kubectl get rc

kubectl delete pod myapp-rc-c57qm

kubectl get pods

kubectl get pods -o wide

kubectl describe rc myapp-rc
``````

##### Changing the pod template
change the value of replicas to 4 and save the template
``````sh
kubectl edit rc myapp-rc

kubectl get pods
kubectl get rc

``````
##### Horizontally scaling pods
t’s incredibly simple to change the desired number of replicas, this also means scaling pods horizontally is trivial.
Assuming you suddenly expect that load on your application is going to increase so you must deploy more pods until the load is reduced, in such case you can easily scale up the number of pods runtime.

``````sh
kubectl scale rc myapp-rc --replicas=6
kubectl scale rc myapp-rc --replicas=3
kubectl get pods
kubectl get rc
kubectl delete rc myapp-rc --cascade=false

``````
##### Using replica sets instead of replication controller
ReplicaSet was introduced. It’s a new generation of ReplicationController and replaces it completely (ReplicationControllers will eventually be deprecated).

- Also, for example, a single ReplicationController can’t match pods with the label env=production and those with the label env=devel at the same time. It can only match either pods with the env=production label or pods with the env=devel label. But a single ReplicaSet can match both sets of pods and treat them as a single group.

we have the KIND value i.e. ReplicaSet, to get the apiVersion of this kind we will use kubectl explain
``````sh
kubectl api-resources
kubectl explain ReplicaSet | head -n 2
--

[root@controller ~]# cat rs1.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: nginx

--
kubectl apply -f rs1.yml
kubectl get pods -o wide
kubectl describe pods myapp-rc-6vjv4
kubectl get rs
kubectl describe rs/frontend
``````

##### Horizontally scaling Pod
``````sh
kubectl label pod lab-nginx-58f9bf94f7-pc8zf app-

---
kubectl scale rs/frontend --replicas=6
kubectl get pods -o wide
kubectl scale rs/frontend --replicas=3
kubectl delete rs/frontend
``````
