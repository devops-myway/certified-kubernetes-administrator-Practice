

##### Why does Kubernetes allow more than one container in a Pod
- Containers in a Pod runs on a "logical host": they use the same network namespace (same IP address and port space), they can use shared volumes
- using several containers for an application is simpler to use, more transparent, and allows decoupling software dependencies

##### Use Cases for Multi-Container Pods

The primary purpose of a multi-container Pod is to support co-located, co-managed helper processes for a main program

Sidecar containers:
"help" the main container. For example, log or data change watchers, monitoring adapters, and so on.
A log watcher, for example, can be built once by a different team and reused across different applications
Another example of a sidecar container is a file or data loader that generates data for the main container.

##### Communication Between Containers in a Pod

Shared volumes:
you can use a shared Kubernetes Volume as a simple and efficient way to share data between containers in a Pod.
Volumes enables data to survive container restarts. It has the same lifetime as a Pod.
it is sufficient to use a directory on the host that is shared with all containers within a Pod

- A standard use case for a multi-container Pod with shared Volume is when one container writes to the shared directory (logs or other files), and the other container reads from the shared directory
- The second container uses Debian image and has the shared volume mounted to the directory /html. The second container every second adds current date and time and into index.html that is located in the shared volume.
- Nginx servers reads this file and transfers it to the user for each HTTP request to the web server.

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: mc1
spec:
  volumes:
  - name: html
    emptyDir: {}
  containers:
  - name: 1st
    image: nginx
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: 2nd
    image: debian
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 1;
        done
----
kubectl exec mc1 -c 1st -- /bin/cat /usr/share/nginx/html/index.html

kubectl exec mc1 -c 2nd -- /bin/cat /html/index.html
``````
#### Network
Containers in a Pod are accessible via "localhost", they use the same network namespace

Step 1. Create a ConfigMap with nginx configuration file. Incoming HTTP requests to the port 80 will be forwarded to the port 5000 on localhost


``````sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: mc3-nginx-conf
data:
  nginx.conf: |-
    user  nginx;
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    events {
        worker_connections  1024;
    }

    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile        on;
        keepalive_timeout  65;

        upstream webapp {
            server 127.0.0.1:5000;
        }

        server {
            listen 80;

            location / {
                proxy_pass         http://webapp;
                proxy_redirect     off;
            }
        }
    }
``````
Step 2. Create a multi-container Pod with simple web app and nginx in separate containers. Note that for the Pod, we define only nginx port 80. Port 5000 will not be accessible outside of the Pod.

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: mc3
  labels:
    app: mc3
spec:
  containers:
  - name: webapp
    image: training/webapp
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nginx-proxy-config
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
  volumes:
  - name: nginx-proxy-config
    configMap:
      name: mc3-nginx-conf
      
``````
Step 3. Expose the Pod using the NodePort service:

``````sh
kubectl expose pod mc3 --type=NodePort --port=80
      
``````
Step 4. Identify port on the node that is forwarded to the Pod:

``````sh
kubectl describe service mc3
      
``````
##### The Side-car Pattern

``````sh
kubectl create namespace myproject
kubectl config set-context --current --namespace=myproject

----
cat > pod-multi-container.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: my-two-container-pod
  namespace: myproject
  labels:
    environment: dev
spec:
  containers:
    - name: server
      image: nginx:1.13-alpine
      ports:
        - containerPort: 80
          protocol: TCP
    - name: side-car
      image: alpine:latest
      command: ["/usr/bin/tail", "-f", "/dev/null"]
  restartPolicy: Never
EOF
---
kubectl create -f pod-multi-container.yaml
kubectl describe pod my-two-container-pod

#Execute a shell session inside the server container by specifying it with the `-c` argument: <command> <argument> -
kubectl exec -it my-two-container-pod -c server -- /bin/sh

--- # Run some commands inside the server container:
ip address
netstat -ntlp
hostname
ps -ef
exit
--- #Execute a shell session inside the side-car container:
kubectl exec -it my-two-container-pod -c side-car -- /bin/sh

---------------- # ip address, Each container within a pod runs its own cgroup
netstat -ntlp
hostname
ps -ef
exit
----

      
``````