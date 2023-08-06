- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector

- https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/

##### nodeSelector - Assign Pods to Nodes

 You can add the nodeSelector field to your Pod specification and specify the node labels you want the target node to have.

#### Add a label to a node
``````sh
kubectl get nodes --show-labels

kubectl label nodes <your-node-name> disktype=ssd
kubectl get nodes --show-labels
kubectl label nodes <your-node-name> disktype-  # remove the labels with the key

``````
##### Create a pod that gets scheduled to your chosen node
This pod configuration file describes a pod that has a node selector, disktype: ssd. This means that the pod will get scheduled on a node that has a disktype=ssd label

``````sh
cat << EOF | kubectl apply -f -
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
EOF

``````
#### Verify
``````sh
kubectl get pods -owide

``````
