https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

##### Static Pod

Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane (for example, a Deployment); instead, the kubelet watches each static Pod (and restarts it if it fails).

Choose a directory, say /etc/kubernetes/manifests and place a web server Pod definition there, for example /etc/kubernetes/manifests/static-web.yaml

##### Filesystem-hosted static Pod manifest

``````sh
ssh my-node1    # ssh into the host or node
 ls -l /etc/kubernetes/manifests/
vi /etc/kubernetes/manifests/static-web.yaml

``````

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
---
kubectl get pods
kubectl get pods -A -owide
rm -rf static-web.yaml
systemctl restart kubelet
``````
