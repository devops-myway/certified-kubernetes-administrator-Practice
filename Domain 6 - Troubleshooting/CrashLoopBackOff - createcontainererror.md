
###### CreateContainerError
Kubernetes throws a CreateContainerError when there’s a problem in the creation of the container, but unrelated with configuration, like a referenced volume not being accessible or a container name already being used.

##### How to troubleshoot CreateContainerError and CreateContainerConfigError

``````sh
kubectl describe pod
kubectl logs or kubectl logs –all-containers
kubectl get events

``````
##### CrashLoopBackOff
CrashLoopBackOff is a Kubernetes state representing a restart loop that is happening in a Pod. a container in the Pod is started, but crashes and is then restarted, over and over again.
CrashLoopBackOff is not an error on itself, but indicates that there’s an error happening that prevents a Pod from starting properly.

Some of the errors linked to the actual application are:

Misconfigurations: Like a typo in a configuration file.
A resource is not available: Like a PersistentVolume that is not mounted.
Wrong command line arguments: Either missing, or the incorrect ones.
Bugs & Exceptions: That can be anything, very specific to your application

##### How to detect a CrashLoopBackOff in your cluster?

Are not in READY condition (0/1).
Their status displays CrashLoopBackOff.
Column RESTARTS displays one or more restarts.
``````sh
kubectl get pods
----
nginx-5796d5bc7c-2jdr5   0/1       CrashLoopBackOff   2          1m
nginx-5796d5bc7c-xsl6p   0/1       CrashLoopBackOff   2          1m

``````
##### How to debug, troubleshoot and fix a CrashLoopBackOff state

``````sh
kubectl describe pod the-pod-name
kubectl logs mypod --all-containers
kubectl logs mypod -c mycontainer

kubectl get events
kubectl get events --field-selector involvedObject.name=mypod


kubectl describe deployment mydeployment
``````