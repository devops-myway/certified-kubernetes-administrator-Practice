
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/


##### Kubernetes SecurityContext Overview
A SecurityContext they allow you to control the behaviour of the running pods and containers and how they interact with the host server.
If applied at the Pod level the settings will apply to all containers in the Pod. If the SecurityContext is defined at both the Pod and container level, the container level SecurityContext will take precedence.

Here are some of the settings which can be configured as part of Kubernetes SecurityContext field. These settings can be applied at the Pod or container level.

- runAsUser : allows you to run containers as a specified user.
- runAsGroup :allows you to run containers as a specified group.
- fsGroup: allows you to run containers with and a specific file system group.
- allowPrivilegeEscalation: controls whether any process inside the container can gain more privilege to perform the respective task.

##### Example1: Without Security Context as Root user
The risks behind it, is the root user here is the same root user on your host and sharing the same kernel.
Another level of administration permissions can be given to the root user if we add a security context as below:
``````sh
apiVersion: v1
kind: Pod
metadata:
 name: security-context-demo-default
spec:
 volumes:
 - name: sec-ctx-vol-default
   emptyDir: {}
 containers:
 - name: sec-ctx-demo-default
   image: busybox:1.28
   command: [ "sh", "-c", "sleep 1h" ]
   volumeMounts:
   - name: sec-ctx-vol-default
     mountPath: /data/demo
  
-------
kubectl exec -it security-context-demo-default -- sh

#id
uid=0(root) gid=0(root) groups=10(wheel)

``````
##### Applying Security Contexts on Pod Level
securityContext is part of pod/spec or pod/spec/containers sections in the deployment file

kubectl explain pod.spec.securityContext | more
kubectl explain pod.spec.containers.securityContext | more

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false

--
kubectl apply -f security-context-demo.yml

kubectl get pod security-context-demo
kubectl exec -it security-context-demo -- sh

----
ps  #Also let’s check the processes that are running in the container:
-----
PID   USER     TIME  COMMAND
    1 1000      0:00 sleep 1h
    6 1000      0:00 sh
----
cd /data
ls -l
---

cd /data/demo
touch testfile.txt
echo "head no well" > testfile.txt
ls -lt
---
# ow let’s try to create the same file under /
cd /
touch testfile.txt
touch: testfile: Permission denied
------

# Then checking the user running the container process through id command
id
exit
``````

##### Set capabilities for a Container
With Linux capabilities, you can grant certain privileges to a process without granting all the privileges of the root user.
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-3
spec:
  containers:
  - name: sec-ctx-3
    image: gcr.io/google-samples/node-hello:1.0
  
-------
kubectl apply -f security-context-demo-3.yaml

kubectl get pod security-context-demo-3

kubectl exec -it security-context-demo-3 -- sh

# In your shell, list the running processes:
ps aux
cd /proc/1
cat status
exit
``````
#### Capabilities

Next, run a Container that is the same as the preceding container, except that it has additional capabilities set.
The configuration adds the CAP_NET_ADMIN and CAP_SYS_TIME capabilities:
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
  securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
-------
kubectl apply -f security-context-demo-4.yaml

kubectl get pod security-context-demo-4

kubectl exec -it security-context-demo-4 -- sh

# In your shell, list the running processes:
ps aux
cd /proc/1
cat status
exit
``````
