
##### Overview on Kubernetes DaemonSets

- A DaemonSet contains a Pod template and uses it to create multiple Pod replicas, just like Deployments, ReplicaSets, and StatefulSets.
- A DaemonSet is typically used to deploy infrastructure Pods that provide some sort of system-level service to each cluster node
- The Kube Proxy component, which is responsible for routing traffic for the Service objects you create in your cluster, is usually deployed via a DaemonSet in the kube-system Namespace
- The Container Network Interface (CNI) plugin that provides the network over which the Pods communicate is also typically deployed via a DaemonSet.

##### Deploying Pods with a DaemonSet
``````sh
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: demo
spec:
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: demo
        image: busybox
        command:
        - sleep
        - infinity
---
kubectl get ds
kubectl get ds -o wide
kubectl describe ds demo
--
# Understanding a DaemonSet’s status
$ kubectl get ds demo -o yaml
...
status:
  currentNumberScheduled: 2
  desiredNumberScheduled: 2
  numberAvailable: 2
  numberMisscheduled: 0
  numberReady: 2
  observedGeneration: 1
  updatedNumberScheduled: 2

``````


#####  Using a DaemonSet to run a pod on every node

``````sh
[root@controller ~]# cat fluentd-daemonset.yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v0.14.10
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers\
--
kubectl create -f fluentd-daemonset.yml
kubectl get ds
kubectl get pods
kubectl get pods -owide

``````
##### Create DaemonSet using Node Selectors
``````sh
[root@controller ~]# cat nginx-daemonset.yml
apiVersion: apps/v1
kind: "DaemonSet"
metadata:
  labels:
    app: nginx
    ssd: "true"
  name: nginx-fast-storage
spec:
  selector:
    matchLabels:
      ssd: "true"
  template:
    metadata:
      labels:
        app: nginx
        ssd: "true"
    spec:
      nodeSelector:
        ssd: "true"
      containers:
        - name: nginx
          image: nginx:1.10.0
--
kubectl create -f nginx-daemonset.yml
kubectl get ds
kubectl get pods -owide
kubectl get nodes --show-labels

``````
#####  Adding required labels to the node
``````sh
kubectl label nodes worker-1.example.com ssd=true
kubectl get ds
kubectl get pods -o wide

``````
#####  Removing the required label from the node
kubectl label node <nodename> <labelname>-
``````sh
kubectl label node worker-1.example.com ssd-
kubectl get pods -o wide
``````
#####  Deleting DaemonSets

``````sh
kubectl get ds
``````

#####  The RollingUpdate strategy
To update the Pods of the demo DaemonSet, use the kubectl apply command to apply the manifest file ds.demo.v2.rollingUpdate.yaml

``````sh
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: demo
spec:
  minReadySeconds: 30
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
        ver: v2
    spec:
      ...
--
kubectl get pods -l app=demo -L ver
kubectl get pods -l app=demo -L ver
kubectl get pods -l app=demo -L ver

``````