
https://kubernetes.io/docs/concepts/services-networking/service/#headless-services

https://medium.com/@bubu.tripathy/headless-k8s-service-924c689607a7


##### Headless Service
A headless service is a service with a service IP but instead of load-balancing it will return the IPs of our associated Pods.
This allows us to interact directly with the Pods instead of a proxy.  It's as simple as specifying None for .spec.clusterIP and can be utilized with or without selectors.
A headless service doesn't provide any sort of proxy or load balancing -- it simply provides a mechanism by which clients can look up the ip address of pods.
This means that when they connect to your service, they're connecting directly to the pods; there's no intervening proxy.
By removing the proxy from the equation, clients are aware of the actual pod ips, which may be important for some application.

Once you’ve created a headless service, you can access each pod associated with the service through DNS. The DNS record for each pod will be in the format <pod-name>.<headless-service-name>.<namespace>.svc.cluster.local.

##### Example1: Create a deployment with five pods.
``````sh
cat <<EOF | kubectl create --filename -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: api
spec:
  replicas: 5
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: eddiehale/hellonodeapi
        ports:
        - containerPort: 3000
EOF

``````
##### Create a regular service and And a headless service
``````sh
cat <<EOF | kubectl create --filename -
apiVersion: v1
kind: Service
metadata:
  name: normal-service
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
EOF
---------
cat <<EOF | kubectl create --filename -
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None # <-- Don't forget!!
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
EOF
----
kubectl get all
----------
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/headless-service   ClusterIP   None             <none>        80/TCP    5s
service/kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP   45h
service/normal-service     ClusterIP   10.109.192.226   <none>        80/TCP    5s

``````
##### Now that its all running, deploy a Pod and execute a few commands to test
Lets run nslookup on each service to see what DNS entries exist.
If we nslookup normal-service one DNS entry and IP is returned
where nslookup headless-service returns the list of associated Pod IPs with the service DNS

``````sh
kubectl run --generator=run-pod/v1 --rm utils -it --image eddiehale/utils bash
nslookup normal-service
nslookup headless-service
exit
kubectl delete svc headless-service normal-service && kubectl delete deployment api-deployment
``````

##### Example2: Use Cases for Headless Services
StatefulSets:
When you create a StatefulSet, Kubernetes automatically creates a headless service to enable direct communication with each pod.
This allows you to access each pod by its unique network identity, which can be useful for stateful applications.

###### create a stateful service
``````sh
cat <<EOF | kubectl create --filename -
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  serviceName: my-statefulset-headless-service
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image
  volumeClaimTemplates:
  - metadata:
      name: my-pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
EOF
``````
