
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/


##### Kubernetes SecurityContext Overview
A SecurityContext is a Kubernetes object, defined as part of the Pod spec, that describes the privileges and access control settings for a Pod.
If applied at the Pod level the settings will apply to all containers in the Pod. If the SecurityContext is defined at both the Pod and container level, the container level SecurityContext will take precedence.

Here are some of the settings which can be configured as part of Kubernetes SecurityContext field. These settings can be applied at the Pod or container level.

- runAsUser : allows you to run containers as a specified user.
- runAsGroup :allows you to run containers as a specified group.
- fsGroup: allows you to run containers with and a specific file system group.
- allowPrivilegeEscalation: controls whether any process inside the container can gain more privilege to perform the respective task.

##### Sample Pod definition with a SecurityContext defined

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
ps
-----
PID   USER     TIME  COMMAND
    1 1000      0:00 sleep 1h
    6 1000      0:00 sh
----
cd /data
ls -l
---

cd demo
echo hello > testfile

# Run the following command:
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