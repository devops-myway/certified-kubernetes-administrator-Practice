https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

#### Taints and Tolerations
Taints and tolerations are a mechanism that allows you to ensure that pods are not placed on inappropriate node.
Taints are added to nodes, When you taint a node, it will repel all the pods except those that have a toleration for that taint.
while tolerations are defined in the pod specification.

A taint can produce three possible effects:
- NoSchedule: The Kubernetes scheduler will only allow scheduling pods that have tolerations for the tainted nodes
- PreferNoSchedule: The Kubernetes scheduler will try to avoid scheduling pods that don’t have tolerations for the tainted nodes.
- NoExecute: Kubernetes will evict the running pods from the nodes if the pods don’t have tolerations for the tainted nodes

#### Use Cases for Taints and Tolerations
Dedicated Nodes: If you need to dedicate a group of worker nodes for a set of users, you can add a taint to those nodes.
``````sh
kubectl taint nodes nodename dedicated=groupName:NoSchedule

``````
Nodes with Special Hardware
``````sh
kubectl taint nodes nodename special=true:NoSchedule

``````
##### How to Use Taints and Tolerations
The taint below with key name app, with a value frontend, and has the effect of NoSchedule, which means that no pod will be placed on this node until the pod has defined a toleration for the taint.

Try to deploy an app on the cluster without any toleration configured in the app deployment specification.
``````sh
kubectl taint nodes cluster01-worker-1 app=frontend:NoSchedule

------
kubectl create ns frontend

kubectl run nginx --image=nginx --namespace frontend

kubectl get events -n frontend

``````

To successfully place the pod on the worker node, we need to edit the deployment and add a toleration of the taint we configured earlier on the node.
``````sh
kubectl get deployment nginx -n frontend -o yaml

------
kubectl edit deployment nginx -n frontend
--
  spec:
        containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
        tolerations:
        - effect: NoSchedule
          key: app
          operator: Equal
          value: frontend
-----
kubectl get events -n frontend

kubectl get pods -n frontend

``````