https://kubernetes.io/docs/concepts/configuration/secret/
https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-a-container-environment-variable-with-data-from-a-single-secret
https://secrets-store-csi-driver.sigs.k8s.io/concepts.html#secretproviderclass


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

echo -n 'jsmith' | base64 | ./username.txt
echo -n 'mysecretpassword' | base64 | ./password.txt

---
kubectl create secret generic [secret-name] \
--from-file=[file1] \
--from-file=[file2]

kubectl create secret generic db-credentials --from-file=./username.txt --from-file=./password.txt
--
kubectl get secrets

``````
#### Create Secrets in a Configuration File
``````sh

apiVersion: v1
kind: Secret
metadata:
  name: secret-test
type: Opaque
data:
  username: dXNlcg==
  password: NTRmNDFkMTJlOGZh

---
kubectl apply -f secret.yml
``````

#### Mount a Secret as a volume to a pod
 the secret-test Secret that contains the username and password information is stored as a file under the /srt directory.
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: pod0
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: srt
      mountPath: "/srt "
      readOnly: true
  volumes:
  - name: srt
    secret:
      secretName: secret-test
-----
kubectl exec -it [pod] -- /bin/bash

kubectl exec -it test-pod -- /bin/bash
cd /srt

``````
#####  Expose a Secret as an environment variable for a pod - using env parameter with valueFrom field.
the username and password stored in the secret-test Secret are referenced in an environment variable of a pod.

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: redis
    image: redis
    env:
      - name: USERNAME
        valueFrom:
          secretKeyRef:
            name: secret-test
            key: username
      - name: PASSWORD
        valueFrom:
          secretKeyRef:
            name: secret-test
            key: password


``````
#### Mounting Secret as a file
It is possible to create Secret and pass it as a file or multiple files to Pods.
can see a sample Secret manifest file and Deployment that uses this Secret:
NOTE: I used subPath with Secrets and it works as expected.
``````sh
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  secret.file1: |
    c2VjcmV0RmlsZTEK
  secret.file2: |
    c2VjcmV0RmlsZTIK
---
apiVersion: apps/v1
kind: Deployment
metadata:
...
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: secrets-files
          mountPath: "/mnt/secret.file1"  # "secret.file1" file will be created in "/mnt" directory
          subPath: secret.file1
        - name: secrets-files
          mountPath: "/mnt/secret.file2"  # "secret.file2" file will be created in "/mnt" directory
          subPath: secret.file2
      volumes:
        - name: secrets-files
          secret:
            secretName: my-secret # name of the Secret
------
kubectl get secret,deploy,pod

kubectl exec nginx-7c67965687-ph7b8 -- ls /mnt

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
 some-file.conf with the provided data inside the file.
 Note: If you do not want to perform the base64 encoding, you can choose to use the stringData field instead.
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
kubectl apply -f secret-mount.yaml

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

#### Projected Volume

A projected volume maps many existing volume sources into an equivalent directory.
Using projected volume I mounted secret.file1, secret.file2 from Secret and config.file1 from ConfigMap as files into the Pod

``````sh
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  secret.file1: |
    c2VjcmV0RmlsZTEK
  secret.file2: |
    c2VjcmV0RmlsZTIK
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  config.file1: |
    configFile1  
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: all-in-one
      mountPath: "/config-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: my-secret
          items:
            - key: secret.file1
              path: secret-dir1/secret.file1
            - key: secret.file2
              path: secret-dir2/secret.file2
      - configMap:
          name: my-config
          items:
            - key: config.file1
              path: config-dir1/config.file1

---------
kubectl exec nginx -- ls /config-volume
kubectl exec nginx -- cat /config-volume/config-dir1/config.file1
kubectl exec nginx -- cat /config-volume/secret-dir1/secret.file1
kubectl exec nginx -- cat /config-volume/secret-dir2/secret.file2

``````