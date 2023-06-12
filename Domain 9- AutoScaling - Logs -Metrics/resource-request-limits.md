
https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/


###### Note:Resource requests and limits for Pod and Container

Limits and Requests are important settings when working with Kubernetes. will focus on the two most important ones: CPU and memory.

- Kubernetes defines Limits as the maximum amount of a resource to be used by a container. This means that the container can never consume more than the memory amount or CPU amount indicated.
- Requests, are the minimum guaranteed amount of a resource that is reserved for a container.

###### Understanding resource units
There is a different unit which us used in Kubernetes to measure CPU and Memory:

###### CPU limit
- Limits and requests for CPU resources are measured in cpu units.
- 1 vCPU/Core for cloud providers and 1 hyperthread on bare-metal Intel processors.
- It is also possible to configure requests or limits with a millicore value. As there are 1,000 millicores to a single core, we could specify half a CPU as 500m.
-  CPU units less than 1.0 or 1000m using the milliCPU form; for example, 5m rather than 0.005

##### Defining Memory limit
kilobyte	1000	  K	kibibyte	   1024    Ki
megabyte	1000*2	M	mebibyte	   1024*2	 Mi
gigabyte	1000*3	G	gibibyte	   1024*3	 Gi

#### How pods with resource limits are managed
- If memory limits are reached, the container runtime will kill the container (and it might be restarted) with OOM.
- When the Kubelet starts a container, the CPU and memory limits are passed to the container runtime, which is then responsible for managing the resource usage of that container.
- If a container is using more memory than the requested amount, it becomes a candidate for eviction if and when the node begins to run low on memory

##### Here is an example of a container being killed with OOM:
``````sh

Nov 28 23:27:36 worker-1.example.com kernel: Memory cgroup out of memory: Kill process 1331 (mysqld) score 2250 or sacrifice child
Nov 28 23:27:36 worker-1.example.com kernel: Killed process 1331 (mysqld) total-vm:1517000kB, anon-rss:126500kB, file-rss:42740kB, shmem-rss:0kB

``````

##### Example: Pod with Memory Request and Limit
50 Mi memory requested
100 Mi memory limit - the max RAM we declare our Pod will ever need

How Kubernetes deals with Pods exceeding those limits. We use an image: mytutorials/centos:bench, that I created and uploaded to the docker hub.
It contains a simple CentOS 7 base operating system. It also includes stress, a benchmark and stress test application.
Syntax:
stress --vm 1 --vm-bytes 50M --vm-hang 10
--vm 1 ... start one virtual worker thread
--vm-bytes ... allocate 50MB of RAM
--vm-hang ... randomly waits 10 seconds and re-allocate around 50 MB of ram

If you do not use --vm-hang the Pod will continuously use 100% CPU since it will continuously re-allocate around 50 MB of ram with no pause between re-allocations.
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
        memory: "50Mi"
    
  restartPolicy: Never
--
kubectl apply -f myBench-Pod.yaml
# We use the command kubectl exec to ssh into the Pod. Here we can run commands at the Linux shell.
kubectl exec mybench-pod -it -- /bin/bash
----
stress --vm 1 --vm-bytes 50M --vm-hang 10
stress --vm 1 --vm-bytes 90M --vm-hang 10
stress --vm 1 --vm-bytes 95M --vm-hang 10   # Container can easily handle allocation of 95MB RAM.
stress --vm 1 --vm-bytes 97M --vm-hang 10   # Allocating 97MB RAM fails - our CentOS container needs around 3 MB to run, We went over our 100Mi RAM limit.

kubectl delete -f myBench-Pod.yaml --force --grace-period=0
``````
##### Example2: Pod with 2 Containers: Each with Memory Request and Limit

``````sh
vi myBench-Pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mybench-pod
spec:
  containers:
  - name: mybench-container-1
    image: mytutorials/centos:bench
    imagePullPolicy: IfNotPresent
    
    command: ['sh', '-c', 'echo mybench-container-1 is Running ; sleep 3600']
    resources:
      limits:
        memory: "100Mi"
      requests:
        memory: "50Mi"
    
  - name: mybench-container-2
    image: mytutorials/centos:bench
    imagePullPolicy: IfNotPresent
    
    command: ['sh', '-c', 'echo mybench-container-2 is Running ; sleep 3600']
    resources:
      limits:
        memory: "100Mi"
      requests:
        memory: "50Mi"

  restartPolicy: Never
-------
kubectl apply -f myBench-Pod.yaml
kubectl exec mybench-pod -c mybench-container-2 -it -- /bin/bash
stress --vm 1 --vm-bytes 88M --vm-hang 10
-----
kubectl exec mybench-pod -c mybench-container-1 -it -- /bin/bash
stress --vm 1 --vm-bytes 95M --vm-hang 10
stress --vm 1 --vm-bytes 120M --vm-hang 10

---
kubectl delete -f myBench-Pod.yaml --force --grace-period=0
``````
##### Cpu and Mem Reuest and Limits
Both containers are defined with a request for 0.25 CPU and 64MiB (226 bytes) of memory. Each container has a limit of 0.5 CPU and 128MiB of memory

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

``````

##### Example: Cpu Management policy
You can specify CPU requests and limits using whole integers or Millicores.
One CPU core comprises 1000m = 1000 millicores.
A 4 core node has a CPU capacity of 4000m
4 cores * 1000m = 4000m

``````sh
vi myguaranteed.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myguaranteed-pod
spec:
  containers:
  - name: myram-container-1
    image: mytutorials/centos:bench
    imagePullPolicy: IfNotPresent    
    command: ['sh', '-c', 'stress --vm 1 --vm-bytes 5M --vm-hang 3000 -t 3600']    
    resources:
      limits:
        memory: "10Mi"
        cpu: 1
      requests:
        memory: "10Mi"
        cpu: 1    
  restartPolicy: Never
  terminationGracePeriodSeconds: 0
----
kubectl apply -f myguaranteed.yaml
kubectl delete -f myguaranteed.yaml
``````

##### Now we will create a simple example pod with nginx image and assign a CPU resource limit of 500m
So with this we should be allowed to create upto two Pods only because after that that CPU quota limit would be reached.

``````sh

``````