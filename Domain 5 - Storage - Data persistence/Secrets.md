- https://kubernetes.io/docs/concepts/configuration/secret/
- https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data


#### Kubernetes Secrets
- Kubernetes Secrets are confidential credentials, stored by default in etcd and available for authorized entities to use in authentication throughout the system. Secrets are classified into the following types:

- Service account: A service account is automatically created by Kubernetes and automatically mounted to the /run/secrets/kubernetes.io/serviceaccount directory of a pod. The service account provides an identity for the pod to interact with the API server.
- Opaque: This type of Secret is encoded in Base64 and used to store sensitive information, such as passwords and certificates.

#### How to create Kubernetes Secrets

The command syntax to create a Secret has the following format:
kubectl create secret <map-name> <data-source>

the name you want to assign to the Secret and is the directory, file, or literal value to draw the data from:
Key: is the filename or the key you provided on the command line
Value: is the file content or the literal value you provided on the command line

##### Using Kubernetes Secrets
- Use kubelet, and the imagePullSecrets field.
- Mount the secret as a file in a volume available to any number of containers in a pod.
- Import the secret as an environment variable to a container.

##### Create a Secret Manifest File
To start creating a secret with kubectl, first create the files to store the sensitive information:
``````sh
echo -n '[username]' > [file1]
echo -n '[password]' > [file2]

touch secret1.txt
echo -n 'username1' | base64
echo -n 'password1' | base64
cat << EOF > secret1.txt
username= dXNlcm5hbWUx    
password = cGFzc3dvcmQx    
EOF
---
kubectl create secret generic -h
kubectl create secret generic test-secret --from-file=./secret1.txt --dry-run=client -oyaml > sec1.yaml #output the secret file
kubectl apply -f sec1.yaml
--
kubectl get secrets/test-secret
kubectl describe secret/test-secret

``````
#### Mount a Secret as a volume to a pod
``````sh
kubectl explain pod.spec  # to understand the properties of containers, volumes, volumeMounts, env and envFrom
kubectl run nginx --image=nginx --dry-run=client -oyaml > pod1.yaml # manually add the voolumes and use volumeMount

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: srt
      mountPath: "/opt"
      readOnly: true
  volumes:
  - name: srt
    secret:
      secretName: test-secret
-----
kubectl apply -f pod1.yaml

kubectl exec -it test-pod -- /bin/bash
cd /data/srt
cat test-secret

``````
#####  Define container environment variables using Secret data
the username and password stored in the secret-test Secret are referenced in an environment variable of a pod.

``````sh
kubectl create secret generic backend-user --from-literal=backend-username='backend-admin' --dry-run=client -oyaml > secret2.yaml
kubectl apply -f secret2.yaml

----
kubectl run nginx --image=nginx --dry-run=client -oyaml > test-sec1.yaml
vi test-sec1.yaml

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
    name: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: backend-user
          key: backend-username
---
kubectl exec -it po/nginx -- sh -c 'echo $SECRET_USERNAME'
backend-admin

``````

#### Define container environment variables with data from multiple Secrets 

``````sh
kubectl create secret generic backend-user --from-literal=backend-username='backend-admin'
kubectl create secret generic db-user --from-literal=db-username='db-admin'

kubectl get secret
kubectl describe secret/backend-user
kubectl describe secret/db-user

``````

``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: secpod
  name: secpod
spec:
  containers:
  - image: nginx
    name: secpod
    env:
    - name: BACKEND_USERNAME
      valueFrom:
        secretKeyRef:
          name: backend-user
          key: backend-username
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-user
          key: db-username
EOF
--
kubectl apply -f secpod.yaml
kubectl exec -it secpod -- sh -c 'env | grep _USERNAME'
DB_USERNAME=db-admin
BACKEND_USERNAME=backend-admin
``````
### Configmap and secrets with deployments
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  db_host: mysql-service
EOF
---
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  username: dXNlcm5hbWU=
  password: cGFzc3dvcmQ=
EOF
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: busybox:1.28
        command: ['sh', '-c', "while true; do env | grep MYSQL; sleep 3600; done"]
        env:
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: password
        - name: MYSQL_SERVER
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: db_host
---
kubectl apply podsec.yaml
kubectl get all
kubectl exec -it my-app-5564479c67-nnl52 -- sh -c 'env | grep MYSQL'
kubectl logs my-app-7d54f6bcd7-ksncn
--
MYSQL_SERVER=mysql-service
MYSQL_PASSWORD=password
MYSQL_USER=username
``````
