https://kubernetes.io/docs/concepts/policy/limit-range/

##### LimitRange

A LimitRange is a policy to constrain the resource allocations (limits and requests) that you can specify for each applicable object kind (such as Pod or PersistentVolumeClaim) in a namespace.

#### LimitRange for Memory
Kubernetes administrators can define RAM limits for their nodes.
These limits are enforced at higher priority over how much RAM your Pod declares and wants to use.
Let's define our first LimitRange : 25Mi RAM as min, 200Mi as max.

``````sh
vi myRAM-LimitRange.yaml

apiVersion: v1
kind: LimitRange
metadata:
  name: my-ram-limit
spec:
  limits:
  - max:
      memory: 200Mi
    min:
      memory: 25Mi
----
kubectl apply -f myRAM-LimitRange.yaml

``````
##### Pod That Requests RAM above and below Limits
Some namespaces or nodes may be dedicated to vast Pods. No tiny Pods allowed.
``````sh
nano myBench-Pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mybench-pod
spec:
  containers:
  - name: mybench-container
    image: mytutorials/centos:bench
    imagePullPolicy: IfNotPresent
    
    command: ['sh', '-c', 'echo The Bench Pod is Running ; sleep 3600']
    resources:
      limits:
        memory: "100Mi"
      requests:
        memory: "10Mi"
    
  restartPolicy: Never

--------
# Error from server (Forbidden): error when creating "myBench-Pod.yaml": pods "mybench-pod" is forbidden: minimum memory usage per Container is 25Mi, but request is 10Mi.
kubectl apply -f myBench-Pod.yaml
``````
##### Error easy to understand. Let's request too much RAM:
``````sh
vi myBench-Pod.yaml
   
apiVersion: v1
kind: Pod
metadata:
  name: mybench-pod
spec:
  containers:
  - name: mybench-container
    image: mytutorials/centos:bench
    imagePullPolicy: IfNotPresent
    
    command: ['sh', '-c', 'echo The Bench Pod is Running ; sleep 3600']
    resources:
      limits:
        memory: "300Mi"
      requests:
        memory: "30Mi"
    
  restartPolicy: Never
------
# Error from server (Forbidden): error when creating "myBench-Pod.yaml": pods "mybench-pod" is forbidden: maximum memory usage per Container is 250Mi, but limit is 300Mi.
kubectl apply -f myBench-Pod.yaml 

``````
##### Let's define a Pod within the allowed RAM ranges:
``````sh
vi myBench-Pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mybench-pod
spec:
  containers:
  - name: mybench-container
    image: mytutorials/centos:bench
    imagePullPolicy: IfNotPresent
    
    command: ['sh', '-c', 'echo The Bench Pod is Running ; sleep 3600']
    resources:
      limits:
        memory: "100Mi"
      requests:
        memory: "30Mi"    
  restartPolicy: Never
-----
kubectl apply -f myBench-Pod.yaml 
kubectl exec mybench-pod -it -- /bin/bash

stress --vm 1 --vm-bytes 50M --vm-hang 10
stress --vm 1 --vm-bytes 90M --vm-hang 10
stress --vm 1 --vm-bytes 120M --vm-hang 10

kubectl delete -f myBench-Pod.yaml --force --grace-period=0

``````

###### Define LimitRange Defaults and Limits
We can also use LimitRange to define default limits. These defaults are used when a Pod spec does not specify its own limits.
`````sh
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
    type: Container
----
kubectl apply -f cpu-resource-constraint.yaml
kubectl describe limits
------
#along with a Pod that declares a CPU resource request of 700m, but not a limit:
# Pod "example-conflict-with-limitrange-cpu" is invalid: spec.containers[0].resources.requests: Invalid value: "700m": must be less than or equal to cpu limit
apiVersion: v1
kind: Pod
metadata:
  name: example-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m

--------
# If you set both request and limit, then that new Pod will be scheduled successfully even with the same LimitRange in place:
apiVersion: v1
kind: Pod
metadata:
  name: example-no-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
      limits:
        cpu: 700m
``````
