https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/?spm=a2c65.11461447.0.0.5f372c3epf2luv#configure-probes


###### Liveness Probes:
Many applications running for long periods of time eventually transition to broken states, this checks your containers are alive
Kubernetes assumes responsibility that your containers in your Pods are alive. If not, it restarts the containers that fail liveness probes.

- initialDelaySeconds: 2 waits 2 seconds after container got created before probing starts
- periodSeconds: 10 liveness probe probes every 10 seconds.

###### Readiness Probes:
checks your containers are able to do productive work
Kubernetes do not assume responsibility for your Pods to be ready
Restarting a container with a failing readiness probe will not fix it, so readiness failures receive no automatic reaction from Kubernetes

###### Define a liveness command
For the first 30 seconds of the container's life, there is a /tmp/healthy file. So during the first 30 seconds, the command cat /tmp/healthy returns a success code. After 30 seconds, cat /tmp/healthy returns a failure code.

Within 30 seconds, view the Pod events:
After 35 seconds, view the Pod events again:
``````sh
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
----
kubectl describe pod liveness-exec
kubectl describe pod liveness-exec

``````
##### Define a liveness HTTP request
The periodSeconds field specifies that the kubelet should perform a liveness probe every 3 seconds
 The initialDelaySeconds field tells the kubelet that it should wait 3 seconds before performing the first probe.
 httpGet livenessProbe: access port 8080 at path /healthz
 For the first 10 seconds that the container is alive, the /healthz handler returns a status of 200. After that, the handler returns status of 500.

``````sh
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
----
kubectl apply -f https://k8s.io/examples/pods/probe/http-liveness.yaml
kubectl describe pod liveness-http
``````

##### Define readiness probes
Readiness probes are configured similarly to liveness probes. The only difference is that you use the readinessProbe field instead of the livenessProbe field.
You have to determine exactly what to test to ensure a readiness probe tests readiness.

### Example2
``````sh
vi readiness-Pod.yaml

piVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-exec
spec:
  containers:
  - name: readiness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
----
kubectl apply -f myLiveness-Pod.yaml
kubectl get pod/readiness-exec
kubectl describe pod/readiness-exec  # after 30 seconds the probe fails
``````
