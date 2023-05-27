https://kubernetes.io/docs/concepts/workloads/pods/

##### Ambassador Pattern - container + proxy(to networking outside)

A container that proxy the network connection to the main container. container + proxy(to networking outside)

Now, the official recommendation to handle these is to use Config Maps, but what if you have legacy code that is already using another way of connecting to the database. What if you want to communicate with localhost, and you can leave the rest to the admin? You can use the Ambassador pattern for these kinds of scenarios.
what we can do is create another container that can act as a TCP Proxy to the database, and you can connect to the proxy via localhost.


###### Ex2.
we will use a simple NGINX config that acts as a TCP proxy to example.com. That should also work for databases and other back ends

If you look carefully in the manifest YAML, you will find there are three containers. The app-container-poller continuously calls http://localhost:81 and sends the content to /usr/share/nginx/html/index.html

The app-container-server runs nginx and listens on port 80 to handle external requests. Both these containers share a common mountPath. That is similar to the sidecar approach

here is an ambassador-container running within the pod that listens on localhost:81 and proxies the connection to example.com, so when we curl the app-container-server endpoint on port 80, we get a response from example.com.

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-pod
  labels:
    app: ambassador-app
spec:
  volumes:
  - name: shared
    emptyDir: {}
  containers:
  - name: app-container-poller
    image: yauritux/busybox-curl
    command: ["/bin/sh"]
    args: ["-c", "while true; do curl 127.0.0.1:81 > /usr/share/nginx/html/index.html; sleep 10; done"]
    volumeMounts:
    - name: shared
      mountPath: /usr/share/nginx/html
  - name: app-container-server
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
    - name: shared
      mountPath: /usr/share/nginx/html
  - name: ambassador-container
    image: bharamicrosystems/nginx-forward-proxy
    ports:
      - containerPort: 81

``````