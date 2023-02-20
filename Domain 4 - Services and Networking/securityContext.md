
##### Kubernetes SecurityContext Overview
- To enforce policies on the pod level, we can use Kubernetes SecurityContext field in the pod specification.
- A security context is used to define different privilege and access level control settings for any Pod or Container running inside the Pod.

Here are some of the settings which can be configured as part of Kubernetes SecurityContext field:

- runAsUser: to specify the UID with which each container will run
- runAsNonRoot: flag that will simply prevent starting containers that run as UID 0 or root.
- runAsGroup: The GID to run the entrypoint of the container process
- fsGroup: we can specify the Group (GID) for filesystem ownership and new files. This can be applied for entire Pod and not on each container.
- allowPrivilegeEscalation: controls whether any process inside the container can gain more privilege to perform the respective task.

##### Define runAsUser for entire Pod

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: pod-as-user-guest
  namespace: test1
spec:
  securityContext:
    runAsUser: 1025
  containers:
  - name: one
    image: golinux-registry:8090/secure-context-img:latest
    command: ["/bin/sleep", "999999"]
  - name: two
    image: golinux-registry:8090/secure-context-img:latest
    command: ["/bin/sleep", "999999"]

--
kubectl create -f security-context-runasuser-1.yaml 

kubectl get pods -n test1
NAME                READY   STATUS    RESTARTS   AGE
pod-as-user-guest   2/2     Running   0          4s
----
kubectl exec -it pod-as-user-guest -n test1 -c one -- id
kubectl exec -it pod-as-user-guest -n test1 -c two -- id
--
kubectl delete pod pod-as-user-guest -n test1
``````

##### Define runAsUser for container

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: pod-as-user-guest
  namespace: test1
spec:
  containers:
  - name: one
    image: golinux-registry:8090/secure-context-img:latest
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 1025
  - name: two
    image: golinux-registry:8090/secure-context-img:latest
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 1026

--
kubectl create -f security-context-runasuser-1.yaml 

kubectl get pods -n test1
----
kubectl exec -it pod-as-user-guest -n test1 -c one -- id
kubectl exec -it pod-as-user-guest -n test1 -c two -- id

``````
