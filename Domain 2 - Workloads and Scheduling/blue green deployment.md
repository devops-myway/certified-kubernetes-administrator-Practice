
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/


##### Kubernetes Deployment Strategies
Kubernetes deployment strategies are the way that pods are updated, downgraded or created into new versions of applications.
Kubernetes deployment strategies work by replacing pods of previous versions of your application with pods of the new version.
The main benefits of these Kubernetes deployment strategies are that it mitigates the risk of disruptions and downtime of services.


###### Blue/ Green (or Red / Black) deployments
In a blue/green deployment strategy (sometimes referred to as red/black) the old version of the application (blue) and the new version (green) get deployed at the same time. When both of these are deployed, users only have access to the blue; whereas, the green is available to your QA team for test automation on a separate service or via direct port-forwarding.

###### Example 1. Create Blue deployment
with pod label `app: blue`
with an busybox initContainer
``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-1.10
spec:
  replicas: 3
  selector:
    matchLabels:
      name: nginx
      app: blue
  template:
    metadata:
      labels:
        name: nginx
        app: blue
    spec:Create Blue deployment


      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo blue > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      containers: 
        - name: nginx
          image: nginx:1.10
          ports:
          - name: http
            containerPort: 80
          volumeMounts:
          - name: workdir
            mountPath: /usr/share/nginx/html
      volumes:
      - name: workdir
        emptyDir: {}

``````
###### Create a service to interact with the blue deployment
``````sh
apiVersion: v1
kind: Service
metadata: 
  name: nginx
  labels: 
    name: nginx
spec:
  ports:
    - name: http
      port: 80
      targetPort: 80
  selector: 
    name: nginx
    app: blue
------
kubectl apply -f blue-deploy.yaml
kubectl apply -f blue-green-svc.yaml
``````
##### Validate both the objects
``````sh
kubectl get deploy
kubectl get svc
kubectl get po
``````
###### Test the deployment

``````sh
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox --command -- wget -qO- nginx

``````
##### Create Green deployment
Green deployment is same as blue except for label `app=green` and command echoing `green` instead of `blue

``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-1.10
spec:
  replicas: 3
  selector:
    matchLabels:
      name: nginx
      app: green
  template:
    metadata:
      labels:
        name: nginx
        app: green
    spec:
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo green > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      containers: 
        - name: nginx
          image: nginx:1.10
          ports:
          - name: http
            containerPort: 80
          volumeMounts:
          - name: workdir
            mountPath: /usr/share/nginx/html
      volumes:
      - name: workdir
        emptyDir: {}

``````
##### Create and validate green deployment
``````sh
kubectl apply -f green-deploy.yaml

kubectl get deployments
kubectl get po

``````
##### Update the selector of service nginx
``````sh
apiVersion: v1
kind: Service
metadata: 
  name: nginx
  labels: 
    name: nginx
spec:
  ports:
    - name: http
      port: 80
      targetPort: 80
  selector: 
    name: nginx
    app: green
--
kubectl apply -f blue-green-svc.yaml
``````
###### Test the deployment
``````sh
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox --command -- wget -qO- nginx

``````
