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
This example defines a simple Pod that has two init containers. The first waits for myservice, and the second waits for mydb. Once both init containers complete, the Pod runs the app container from its spec section.
``````sh
vi myapp-init.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app.kubernetes.io/name: myApp
spec:
  containers:
  - name: myapp
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]

--
kubectl apply -f myapp-init.yaml 
kubectl get pod
kubectl describe myapp-init.yaml
``````
To see logs for the init containers in this Pod, run:

``````sh
kubectl logs myapp-pod -c init-myservice # Inspect the first init container
kubectl logs myapp-pod -c init-mydb      # Inspect the second init container

``````
At this point, those init containers will be waiting to discover Services named mydb and myservice.
You'll then see that those init containers complete, and that the myapp-pod Pod moves into the Running state:
``````sh
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  type: clusterip
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  type: clusterip
  - protocol: TCP
    port: 80
    targetPort: 9377
----
kubectl apply -f services.yaml

``````
``````sh


``````

