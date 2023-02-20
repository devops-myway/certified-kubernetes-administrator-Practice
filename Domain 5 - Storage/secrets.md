#### Kubernetes Secrets to pass sensitive data to containers
- While ConfigMaps are great for most configuration data, there is certain data that is extra-sensitive. This can include passwords, security tokens, or other types of private keys. Collectively, we call this type of data “secrets.”
- Secrets enable container images to be created without bundling sensitive data. This allows containers to remain portable across environments.
- A Secret is base64-encoded, so we cannot treat it as secure and It can also store binary data such as a public or private key.
- Kubernetes ensures that Secrets are passed only to the nodes that are running the Pods that need the respective Secrets.

#### How to create Kubernetes Secrets
Secrets can be created through

- literal value or hardcorded values : literal value maybe something like a password to your internal database and it would be categorized as a generic Secret:
- from a file or all the files in a directory
- Manually defining the key value in YAML file format

The command syntax to create a Secret has the following format:

kubectl create secret <map-name> <data-source>

the name you want to assign to the Secret and is the directory, file, or literal value to draw the data from:
Key: is the filename or the key you provided on the command line
Value: is the file content or the literal value you provided on the command line

#### Method-1: Declare Kubernetes Secrets as Environment Variables
``````sh
kubectl create secret generic db-secret --from-literal=db_user=deepak --from-literal=db_password=test1234

kubectl get secret db-secret -oyaml
--
apiVersion: v1
data:
  db_password: dGVzdDEyMzQ=
  db_user: ZGJfZGVlcGFr
kind: Secret

``````
``````sh
echo dGVzdDEyMzQ= | base64 --decode

``````
##### Declare secrets as environment variables
I will create a busybox pod secret-busybox.yml 

use secretKeyRef under env to fetch the secret value from the provided key and set it as environment variable.

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: secret-busybox
spec:
  containers:
  - name: secret-busybox
    image: busybox
    command:
     - sleep
     - "3600"
    env:
      - name: DB_USER  ## Provide the variable name to map the value
        valueFrom:
          secretKeyRef:
            name: db-secret  ## Name of the secret created
            key: db_user  ## The key to fetch
      - name: DB_PWD  ## Provide the variable name to map the value
        valueFrom:
          secretKeyRef:
            name: db-secret  ## Name of the secret created
            key: db_password  ## The key to fetch
-------

kubectl create -f secret-busybox.yaml
kubectl exec -it secret-busybox -- printenv | grep DB
``````
#### Example-2: Declare Kubernetes Secrets manually using YAML file
You can encode any string using echo <string> | base64 and then place the encoded string in above file.

``````sh
apiVersion: v1
kind: Secret
metadata:
  name: some-secret-1
type: Opaque
data:
  secret1: dGVzdDEyMwo=
  secret2: ZHVtbXkxMjMK

---
kubectl create -f secret-1.yaml
kubectl get secret some-secret -o yaml

``````
#### Declare Kubernetes Secrets as environment variables

``````sh
[root@controller ~]# cat secret-busybox-1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-busybox-1
spec:
  containers:
  - name: secret-busybox-1
    image: busybox
    command:
     - sleep
     - "3600"
    env:
      - name: VAR1  ## Provide the variable name to map the value
        valueFrom:
          secretKeyRef:
            name: some-secret  ## Name of the secret created
            key: secret1  ## The key to fetch
      - name: VAR2  ## Provide the variable name to map the value
        valueFrom:
          secretKeyRef:
            name: some-secret  ## Name of the secret created
            key: secret2  ## The key to fetch
----
kubectl create -f secret-busybox-1.yaml
kubectl exec -it secret-busybox-1 -- printenv | grep VAR

``````
#### Method-2: Mount Kubernetes Secrets as a file
 Declare Kubernetes Secrets using certificates and mount as a file
``````sh
[root@controller ~]# ls server.*
server.crt  server.csr  server.key

``````
Create Kubernetes Secrets from multiple files
``````sh
kubectl create secret generic secret-tls --from-file=server.crt --from-file=server.key

kubectl describe secret secret-tls

``````
#### Method-2: Mount Secrets as a file inside Pod’s container
 
``````sh
[root@controller ~]# cat secret-tls-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: secret-tls-pod
spec:
  containers:
    - name: secret-tls-pod
      image: busybox
      command:
      - sleep
      - "3600"
      volumeMounts:
      - name: tls-certs  ## Use this name inside volumes to define mount point
        mountPath: "/tls"  ## This will be created if not present
  volumes:
    - name: tls-certs  ## This must match the volumeMount name
      secret:
        secretName: secret-tls  ## This must match the secret name which we created earlier
--
kubectl get pods secret-tls-pod
kubectl exec -it secret-tls-pod -- /bin/sh

``````
#### Create Kubernetes Secret as a file
 some-file.conf with the provided data inside the file
``````sh
~]# cat secret-mount.yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-mount
type: Opaque
stringData:
  some-file.conf: |
          VAR1=value-1
          VAR2=value-2
--
kubectl create -f secret-mount.yaml

kubectl get secret secret-mount -o yaml

``````
#### Mount the Kubernetes Secret as a file

Now we will mount our some-file.conf under /dir1 on the Pod as read-only
``````sh
~]# cat secret-busybox-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-mount
spec:
  containers:
    - name: secret-mount
      image: busybox
      command:
      - sleep
      - "3600"
      volumeMounts:
      - name: check-mount  ## Use this name inside volumes to define mount point
        mountPath: "/dir1"  ## This will be created if not present
  volumes:
    - name: check-mount  ## This must match the volumeMount name
      secret:
        secretName: secret-mount  ## This must match the secret name which we created earlier
        items:
        - key: some-file.conf  ## Existing filename used in the secret
          path: new-file-name.conf  ## New file name using which it will be mounted on the Pod
--
kubectl create -f secret-busybox-2.yaml

kubectl exec -it secret-mount -- sh
``````