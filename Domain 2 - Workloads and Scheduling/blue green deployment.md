
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

###### Blue/ Green (or Red / Black) deployments
In a blue/green deployment strategy (sometimes referred to as red/black) the old version of the application (blue) and the new version (green) get deployed at the same time. When both of these are deployed, users only have access to the blue; whereas, the green is available to your QA team for test automation on a separate service or via direct port-forwarding.

###### Example 1. Create Blue deployment
with pod label `app: blue`
with an busybox initContainer
``````sh
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deploy
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
    spec:
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
EOF

``````
###### Create a service to interact with the blue deployment
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: blue
  name: blue-svc
spec:
  ports:
  - name: blue-svc
    port: 8282
    protocol: TCP
    targetPort: 80
  selector:
    app: blue
  type: NodePort
status:
  loadBalancer: {}
EOF
------
kubectl apply -f blue-deploy.yaml
kubectl get svc/blue-svc
kubectl get nodes -owide
``````
##### Test the Blue Deployment for output
``````sh
kubectl port-forward svc/blue-svc 8383:8282
``````

##### Create Green deployment
Green deployment is same as blue except for label `app=green` and command echoing `green` instead of `blue

``````sh
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-deploy
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
EOF
``````
##### Create and validate green deployment
``````sh
kubectl apply -f green-deploy.yaml
kubectl get deploy/green-deploy -owide
kubectl get po

``````
##### Update the selector of service nginx
``````sh
kubectl create svc nodeport green-svc --tcp=8181:80 --dry-run=client -oyaml > green-svc.yaml

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: green
  name: green-svc
spec:
  ports:
  - name: green-svc
    port: 8181
    protocol: TCP
    targetPort: 80
  selector:
    app: green
  type: NodePort
status:
  loadBalancer: {}
EOF
--
kubectl apply -f green-svc.yaml
kubectl get svc/green-svc -owide
kubectl get nodes -owide
``````
###### Test the deployment
``````sh
kubectl port-forward svc/green-svc 8181:8181
``````
