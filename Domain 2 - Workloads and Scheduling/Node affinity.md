https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/


##### Node affinity
Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels.
There are two types of node affinity:

- requiredDuringSchedulingIgnoredDuringExecution:
The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
- preferredDuringSchedulingIgnoredDuringExecution:
The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

You can use the operator field to specify a logical operator for Kubernetes to use when interpreting the rules.
You can use In, NotIn, Exists, DoesNotExist, Gt and Lt

#####  Add a label to a node

``````sh
kubectl get nodes --show-labels
kubectl label nodes <your-node-name> disktype=ssd
kubectl label nodes <your-node-name> disktype=hdd
kubectl get nodes --show-labels

``````

##### Schedule a Pod using required node affinity
This manifest describes a Pod that has a requiredDuringSchedulingIgnoredDuringExecution node affinity,disktype: ssd. This means that the pod will get scheduled only on a node that has a disktype=ssd label.

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: with-node-affinity
    image: nginx
    imagePullPolicy: IfNotPresent
------
kubectl apply -f https://k8s.io/examples/pods/pod-nginx-required-affinity.yaml
kubectl get pods --output=wide

``````

###### Schedule a Pod using preferred node affinity 
This manifest describes a Pod that has a preferredDuringSchedulingIgnoredDuringExecution node affinity,disktype: ssd. This means that the pod will prefer a node that has a disktype=ssd label.
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: NotIn
            values:
            - ssd          
  containers:
  - name: anti-affinity
    image: nginx
    imagePullPolicy: IfNotPresent

-------
kubectl apply -f https://k8s.io/examples/pods/pod-nginx-preferred-affinity.yaml
kubectl get pods -owide
``````