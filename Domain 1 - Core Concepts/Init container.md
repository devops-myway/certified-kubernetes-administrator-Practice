https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

##### Init Containers

An init container can be defined as a container with modified operational rules and behavior. They normally contain utilities and setup scripts that are not present in the app image.

The main difference between init containers and regular containers are:
- Init containers always run to completion.
- Each init container must complete successfully before the next one starts.

If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds. However, if the Pod has a restartPolicy of Never, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.
In addition to the above, init containers do not support livenessProbe, readinessProbe, startupProbe or lifecycle since they must run to completion for the pod to be ready.

##### Init Containers That Just Sleeps (Too Slow)
Let's create a Pod with 2 Init Containers that just echos and sleeps .
``````sh
vi myInitPod-1.yaml
   
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: my-init-container-1
    image: busybox
    command: ['sh', '-c', 'echo my-init-container-1 start; sleep 2;echo my-init-container-1 complete;']
  - name: my-init-container-2
    image: busybox
    command: ['sh', '-c', 'echo my-init-container-2 start; sleep 2;echo my-init-container-2 complete;']
---
kubectl create -f myInitPod-1.yaml 
kubectl get po  
kubectl get po  # later on

``````
##### check the logs

```sh
kubectl logs pod/myapp-pod -c my-init-container-1 --timestamps=true
kubectl logs pod/myapp-pod -c my-init-container-2 --timestamps=true
kubectl logs pod/myapp-pod  --timestamps=true
kubectl describe pod/myapp-pod 
```

#####  Init Containers That Crash Forever
Init Containers must complete successfully before the main container can run.
Now we demo Init Container number 1 crashing : exit code = 1 ... and observe the effects
We specify imagePullPolicy: IfNotPresent in the Pod spec below.

That causes the Pod to ONLY go pull our busybox image from the Internet if it is not present on the local server.

 ``````sh
vi myInitPod-3.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  terminationGracePeriodSeconds: 0
  initContainers:
  - name: my-init-container-1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo my-init-container-1 start; exit 1 ;echo my-init-container-1 complete;']
  - name: my-init-container-2
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo my-init-container-2 start; sleep 2;echo my-init-container-2 complete;']  
---
kubectl create -f myInitPod-3.yaml
kubectl get po
kubectl describe pod/myapp-pod 
kubectl delete -f myInitPod-3.yaml --force --grace-period=0 

``````

##### Realistic Production Init Container Simulation
I removed terminationGracePeriodSeconds: 0 This simulates a live Pod: it must take the default 30 seconds to terminate gracefully.

``````sh
vi myInitPod-7.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo The app is running! && sleep 10']
  
  restartPolicy: Never
  
  initContainers:

  - name: my-init-container-1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo my-init-container-1 start; sleep 3 ;echo my-init-container-1 complete;']

  - name: my-init-container-2
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo my-init-container-2 start; sleep 3;echo my-init-container-2 complete;']  

  - name: my-init-container-3
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo my-init-container-3 start; sleep 3;echo my-init-container-3 complete;']  

  - name: my-init-container-4
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo my-init-container-4 start; sleep 3;echo my-init-container-4 complete;']  
--
kubectl apply -f myInitPod-7.yaml 
kubectl get po 
kubectl delete -f myInitPod-7.yaml

``````
``````sh


``````
``````sh


``````
``````sh


``````

