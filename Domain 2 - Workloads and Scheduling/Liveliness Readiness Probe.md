https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/?spm=a2c65.11461447.0.0.5f372c3epf2luv#configure-probes


###### Liveness Probes:
Liveness probes indicate if a container is running. Meaning, has the application within the container started running and is it still running? If you’ve configured liveness probes for your containers, you’ve probably still seen them in action. When a container gets restarted, it’s generally because of a liveness probe failing. This can happen if your container couldn’t startup, or if the application within the container crashed. The Kubelet will restart the container because the liveness probe is failing in those circumstances. In some circumstances though, the application within the container is not working, but hasn’t crashed. In that case, the container won’t restart unless you provide additional information as a liveness probe

- initialDelaySeconds: 2 waits 2 seconds after container got created before probing starts
- periodSeconds: 10 liveness probe probes every 10 seconds.

###### Readiness Probes:
A readiness probe indicates if the application running inside the container is “ready” to serve requests. As an example, assume you have an application that starts but needs to check on other services like a backend database before finishing its configuration. Or an application that needs to download some data before it’s ready to handle requests. A readiness probe tells the Kubelet that the application can now perform its function and that the Kubelet can start sending it traffic.

###### Define a liveness command

There are three different ways these probes can be checked.

ExecAction: Execute a command within the container
TCPSocketAction: TCP check against the container’s IP/port
HTTPGetAction: An HTTP Get request against the container’s IP/Port

For the first 30 seconds of the container's life, there is a /tmp/healthy file. So during the first 30 seconds, the command cat /tmp/healthy returns a success code. After 30 seconds, cat /tmp/healthy returns a failure code.

Within 30 seconds, view the Pod events:
After 35 seconds, view the Pod events again:
``````sh
cat << EOF | kubectl apply -f -
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
EOF
----
kubectl describe pod liveness-exec

``````
##### Define a liveness HTTP request
The periodSeconds field specifies that the kubelet should perform a liveness probe every 3 seconds
 The initialDelaySeconds field tells the kubelet that it should wait 3 seconds before performing the first probe.
 httpGet livenessProbe: access port 8080 at path /healthz
 For the first 10 seconds that the container is alive, the /healthz handler returns a status of 200. After that, the handler returns status of 500.

``````sh
cat << EOF | kubectl apply -f -
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
EOF
----
kubectl describe pod liveness-http
``````

##### Define Readiness probes
Readiness probes are configured similarly to liveness probes. The only difference is that you use the readinessProbe field instead of the livenessProbe field.
You have to determine exactly what to test to ensure a readiness probe tests readiness.

### Example2
``````sh
vi readiness-Pod.yaml

cat << EOF | kubectl apply -f -
apiVersion: v1
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
EOF
----
kubectl get pod/readiness-exec
kubectl describe pod/readiness-exec  # after 30 seconds the probe fails
``````
