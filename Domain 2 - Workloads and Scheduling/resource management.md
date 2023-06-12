
https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#organizing-resource-configurations

###### Organizing resource configurations
Management of multiple resources can be simplified by grouping them together in the same file (separated by --- in YAML)

``````sh

apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
----
kubectl apply -f https://k8s.io/examples/application/nginx-app.yaml

``````