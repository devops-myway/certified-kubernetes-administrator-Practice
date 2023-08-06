

https://kubernetes.io/docs/concepts/workloads/pods/

##### Multi-Container Pods
The multi-container pods are the pods that contain two or more related containers that share resources like network space, shared volumes, etc and work together as a single unit.
An extra container in your pod to enhance or extend the functionality of the main container.

###### SideCar Pattern - container + container(share same resource and do other functions)
Every application has multiple components taking care of several tasks such as monitoring, logging, configuration, etc
A sidecar pattern is used to enhance or extend the existing functionality of the Main container. Some great examples are using a sidecar for monitoring and logging and adding an agent for these purposes.

#### Ex1.
if you look at the manifest you’ll see we have two containers: app-container and log-exporter-sidecar. The app-container continuously streams logs to /var/log/app.log, and the sidecar container mounts the logs to the nginx HTML directory.

run a port-forward on port 80, you should be able to access the logs using a browser.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  volumes:
  - name: logs 
    emptyDir: {}
  containers:
  - name: app-container
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> /var/log/app.log; sleep 2;done"]
    volumeMounts:
    - name: logs
      mountPath: /var/log
  - name: log-exporter-sidecar
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
    - name: logs
      mountPath: /usr/share/nginx/html
EOF

---
kubectl exec -it po/sidecar-pod -c app-container -- sh -c 'cat /var/log/app.log'
kubectl exec -it po/sidecar-pod -c log-exporter-sidecar -- sh -c 'ls -l /usr/share/nginx/html'
kubectl exec -it po/sidecar-pod -c log-exporter-sidecar -- sh -c 'cat /usr/share/nginx/html/app.log'

``````
##### Example 2: multi-container pod with volumes
The sidecar container with the image busybox puts the message “I am from sidecar container” in the index.html in the shared volume and we have the main container with the image nginx serving at the port 80 and read the index.html from the volume.

``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-cont-pod
  name: multi-cont-pod
spec:
  volumes:
  - name: var-logs
    emptyDir: {}
  containers:
  - image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo 'Hi I am from Side Car container' >> /var/log/index.html; sleep 3600;done"]
    name: sidecar-container
    volumeMounts:
    - name: var-logs
      mountPath: /var/log
  - image: nginx
    name: main-container
    ports:
      - containerPort: 80
    volumeMounts:
    - name: var-logs
      mountPath: /usr/share/nginx/html
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
EOF
-------

kubectl exec -it po/multi-cont-pod -c sidecar-container -- sh -c 'cat /var/log/index.html'
kubectl exec -it po/multi-cont-pod -c main-container -- sh -c 'ls -l /usr/share/nginx/html'
kubectl exec -it po/multi-cont-pod -c main-container -- sh -c 'cat /usr/share/nginx/html/index.html'

``````