https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/

#### Taints and Tolerations
Taints and tolerations are a mechanism that allows you to ensure that pods are not placed on inappropriate node.
Taints are added to nodes, When you taint a node, it will repel all the pods except those that have a toleration for that taint.
while tolerations are defined in the pod specification.

A taint can produce three possible effects:
- NoSchedule: The Kubernetes scheduler will only allow scheduling pods that have tolerations for the tainted nodes
- PreferNoSchedule: The Kubernetes scheduler will try to avoid scheduling pods that don’t have tolerations for the tainted nodes.
- NoExecute: Kubernetes will evict the running pods from the nodes if the pods don’t have tolerations for the tainted nodes.

##### Dedicated Nodes
``````sh
kubectl taint nodes nodename dedicated=groupName:NoSchedule

``````
``````sh
kubectl get nodes --show-labels
kubectl label nodes <your-node-name> disktype=ssd
kubectl get nodes --show-labels

``````
###### Create a pod that gets scheduled to your chosen node
``````sh
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
--
Kubectl apply -f 
kubectl get pods --output=wide
``````
##### Create a pod that gets scheduled to specific node
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: foo-node # schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
---
kubectl apply -f
kubectl get pods --output=wide
``````
##### How to Use Taints and Tolerations
``````sh
kubectl get nodes
kubectl taint nodes node01 key1=value1:NoSchedule
kubectl describe node/node01
``````

``````sh
vi test-pod.yaml

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
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
--
kubectl apply -f test-pod.yaml
kubectl get pod/nginx -owide # watch as it schedule with tolerations on the dedicated pod
``````
