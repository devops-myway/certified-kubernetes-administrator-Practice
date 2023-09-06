https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/?spm=a2c65.11461447.0.0.5f372c3epf2luv#configure-probes


###### Liveness Probes:
The liveness probe is used to check whether a pod is alive.

###### Readiness Probes:
The readiness probe is used to check whether a pod is ready. Only pods in the ready state provide external services and receive traffic from the access layer. When a pod is not in the ready state, the access layer bypasses traffic from the pod.

###### Application Health Status - 

The liveness and readiness probes support three check methods.

1- HTTP GET: Sends an HTTP GET request to check the health of an application. The application is considered healthy when the return code ranges from 200 to 399.
2- exec: Checks whether a service is normal by running a command in the container. The container is healthy when 0 is returned for the command.
3- TCP Socket: Performs a TCP health check on a container by checking the IP address and port of the container. The container is healthy if a TCP connection is established.

###### Check Results
Check results are divided into three states:

- Succeeded: Indicates that the container passed the health check and is considered normal by the liveness probe or readiness probe.
- Failed: Indicates that the container fails to pass the health check. When the readiness probe determines that a pod is abnormal, the pod is removed from the service layer. When the liveness probe determines that a pod is abnormal, the pod is restarted or deleted.
- Unknown: Indicates that the current mechanism is not fully executed probably because the mechanism times out or no response is returned promptly after script execution. In this case, the readiness probe or liveness probe waits for the following check mechanism, without performing any operations.

##### Ex 1
Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status.
Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.
``````sh
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe:
      initialDelaySeconds: 5 # add this line
      periodSeconds: 5 # add this line as well
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
----
kubectl
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
Readiness probes are configured similarly to liveness probes. The only difference is that you use the readinessProbe field instead of the livenessProbe field. You have to determine exactly what to test to ensure a readiness probe tests readiness.
###### Question-
Create an busybox pod (that includes port 80) with an HTTP readinessProbe on path /tmp/healthy on port 80. Again, run it, check the readinessProbe, delete it.

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
    ports:
    - containerPorts: 80
      name: http
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
kubectl describe po nginx | grep -i readiness
    Readiness:      http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3
``````
