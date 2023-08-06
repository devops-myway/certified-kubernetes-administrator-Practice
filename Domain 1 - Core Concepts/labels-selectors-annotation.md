
https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/


##### Labels -  Maps (aka Dictionaries)
- Labels are attached to Kubernetes objects and are simple key: value pairs or maps(dictionary).
- Labels are used to store identifying information about a thing that you might need to query against.
- Labels are used for organization and selection of subsets of objects, and can be added to objects at creation time and/or modified at any time during cluster operations.
- You will see them on pods, replication controllers, replica sets, services, and so on.

``````sh
“labels”: {
“tier”: “frontend”
env: prod
}

``````

##### Selectors -  Maps (aka Dictionaries)
- Labels are queryable — which makes them especially useful in organizing things
-  A label selector is a string that identifies which labels you are trying to match
- You will see them on pods, replication controllers, replica sets, services, and so on.

``````sh
tier = frontend
tier != frontend
environment in (production, qa)
``````
##### Annotations
- Annotations are bits of useful information you might want to store about a pod (or cluster, node, etc.) that you will not have to query against.
- They are also key/value pairs and have the same rules as labels.
- Examples of things you might put there are the pager contact, the build date, or a pointer to more information someplace else—like a URL.

##### Method-1: Assign labels while creating a new object

``````sh
kubectl create deployment label-nginx-example --image=nginx --dry-run=client -oyaml > label-nginx-example.yml
-- # clean up the template and add a label app: prod
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: labelex
  labels:
    env: develop
spec:
  containers:
  - name: sise
    image: quay.io/openshiftlabs/simpleservice:0.4.0
    ports:
    - containerPort: 9870
EOF
-------                 
kubectl get pods --show-labels
kubectl label pods labelex owner=ijaz
kubectl label pods labelex owner=ahmad --overwrite
kubectl get pods --show-labels

kubectl get pods --selector owner=ahmad
kubectl get pods -l env=develop

``````
#####  Kubernetes also supports set-based selectors. To demonstrate this, let’s create another pod with env=prod and owner=ijaz

``````sh
$ cat prod-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: labelexother
  labels:
    env: prod
    owner: ijaz
spec:
  containers:
  - name: sise
    image: quay.io/openshiftlabs/simpleservice:0.4.0
    ports:
    - containerPort: 9870
-----                   
$ kubectl apply -f prod.pod.yaml 
kubectl get pods -l 'env in (prod, develop)'
kubectl delete pods -l 'env in (prod, develop)'
``````
#####  Method-3: Assign a new label to existing deployments runtime using kubectl

I have another deployment nginx-deploy on my cluster, so I will assign label tier: backend to this deployment:
``````sh
kubectl label deployment nginx-deploy tier=backend

kubectl get deployments -owide --show-labels

``````
#####  Kubernetes labels example use cases - Services and Deployments
The most common use case for labels is using label selectors in Kubernetes services and deployment objects. A service object targets pods based on label selectors.
The below specification will create a new service object named "app-service", targeting TCP port 9370 on any pod with the app=MyApp label

``````sh
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9370     

``````
``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80   

``````

#####  Using Labels in Advance Scheduling
Using labels and selectors is also common in advanced Kubernetes scheduling. Examples of advanced K8s scheduling include node selector and pod affinity.
Then, in the deployment specification, you can use the label disktype=ssd as node selector to place the pod on the node with label disktype=ssd.
``````sh
kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd.     
---
apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels:
      env: test
  spec:
    containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
    nodeSelector:
      disktype: ssd    

``````
#####  Removing labels

``````sh
kubectl label pod lab-nginx-58f9bf94f7-pc8zf app-

---
kubectl get pods --show-labels

kubectl get deployments --show-labels

kubectl label deployment lab-nginx app-

kubectl get deployments --show-labels
``````