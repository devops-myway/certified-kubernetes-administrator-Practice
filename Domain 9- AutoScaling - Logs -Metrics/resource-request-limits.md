

###### Note:Resource requests and limits for Pod and Container

Limits and Requests are important settings when working with Kubernetes. will focus on the two most important ones: CPU and memory.

- Kubernetes defines Limits as the maximum amount of a resource to be used by a container. This means that the container can never consume more than the memory amount or CPU amount indicated.
- Requests, are the minimum guaranteed amount of a resource that is reserved for a container.

###### Understanding resource units
There is a different unit which us used in Kubernetes to measure CPU and Memory:

###### CPU limit
- Limits and requests for CPU resources are measured in cpu units.
- 1 vCPU/Core for cloud providers and 1 hyperthread on bare-metal Intel processors.
- It is also possible to configure requests or limits with a millicore value. As there are 1,000 millicores to a single core, we could specify half a CPU as 500 m.
- The smallest amount of CPU that can be specified is 1 m or 0.001.

##### Defining Memory limit
kilobyte	1000	  K	kibibyte	   1024    Ki
megabyte	1000*2	M	mebibyte	   1024*2	 Mi
gigabyte	1000*3	G	gibibyte	   1024*3	 Gi

#### How pods with resource limits are managed
- If memory limits are reached, the container runtime will kill the container (and it might be restarted) with OOM.
- When the Kubelet starts a container, the CPU and memory limits are passed to the container runtime, which is then responsible for managing the resource usage of that container.
- If a container is using more memory than the requested amount, it becomes a candidate for eviction if and when the node begins to run low on memory
- This typically means assigning a certain number of slices to a cgroup. If the processes in one cgroup are idle and don't use their allocated CPU time, these shares will become available to be used by processes in other cgroups.

##### Here is an example of a container being killed with OOM:
``````sh

Nov 28 23:27:36 worker-1.example.com kernel: Memory cgroup out of memory: Kill process 1331 (mysqld) score 2250 or sacrifice child
Nov 28 23:27:36 worker-1.example.com kernel: Killed process 1331 (mysqld) total-vm:1517000kB, anon-rss:126500kB, file-rss:42740kB, shmem-rss:0kB

``````

##### Example: Define CPU and Memory limit for containers
It is always a good idea to use a separate namespace when defining resource limits so that the resources you create in this exercise are isolated from the rest of your cluster.

``````sh
kubectl create namespace cpu-limit
kubectl get ns
--
[root@controller ~]# cat pod-resource-limit.yml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  namespace: cpu-limit
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
--
kubectl create -f pod-resource-limit.yml
kubectl get pods  -n cpu-limit -o wide
kubectl describe pods frontend

``````
##### Deleting Pod

``````sh
kubectl delete pods -n cpu-limit frontend

``````

##### Hands-on example
deployment, where we are setting up limits and requests for two different containers on both CPU and memory
example, we are running a cluster of 4 cores and 16GB RAM nodes. We can extract a lot of information:

- Pod effective request is 400 MiB of memory and 600 millicores of CPU. You need a node with enough free allocatable space to schedule the pod.
- CPU shares for the redis container will be 512, and 102 for the busybox container. Kubernetes always assign 1024 shares to every core, so redis: 1024 * 0.5 cores ≅ 512 and busybox: 1024 * 0.1cores ≅ 102
- Redis container will be OOM killed if it tries to allocate more than 600MB of RAM, most likely making the pod fail.

``````sh

kind: Deployment
apiVersion: extensions/v1beta1
…
template:
  spec:
    containers:
      - name: redis
        image: redis:5.0.3-alpine
        resources:
          limits:
            memory: 600Mi
            cpu: 1
          requests:
            memory: 300Mi
            cpu: 500m
      - name: busybox
        image: busybox:1.28
        resources:
          limits:
            memory: 200Mi
            cpu: 300m
          requests:
            memory: 100Mi
            cpu: 100m

``````

##### Example: Define CPU quota for a namespace

``````sh

kubectl create namespace quota-example
---
[root@controller ~]# cat ns-quota-limit.yml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: 2
    requests.memory: 1Gi
    limits.cpu: 3
    limits.memory: 2Gi
---
kubectl apply -f resourcequota.yaml --namespace=mynamespace

kubectl get resourcequota -n mynamespace


``````

##### Now we will create a simple example pod with nginx image and assign a CPU resource limit of 500m
So with this we should be allowed to create upto two Pods only because after that that CPU quota limit would be reached.

``````sh

[root@controller ~]# cat pod-nginx-lab-1.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
  namespace: quota-example
spec:
  selector:
    matchLabels:
      app: example
  template:
    metadata:
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            cpu: 1100m
-----
kubectl create -f pod-nginx-lab-1.yml
kubectl get pods -n quota-example -o wide
--
kubectl -n quota-example scale deployment/example --replicas=5
kubectl get pods -n quota-example -o wide

kubectl -n quota-example get events

kubectl delete deployments -n quota-example example

kubectl delete ns quota-example
---------
