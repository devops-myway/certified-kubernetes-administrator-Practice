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

touch secret1.txt
echo -n 'jsmith' | base64 | ./username.txt
echo -n 'mysecretpassword' | base64 | ./password.txt
cat << EOF > secret1.txt
username= username encoded
password = password encoded
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
      mountPath: "/data/srt "
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
#####  Expose a Secret as an environment variable for a pod - using env parameter with valueFrom field.
the username and password stored in the secret-test Secret are referenced in an environment variable of a pod.

``````sh
# crete a simple txt file to store the secrets
touch test-secret.txt 
echo -n 'user1'| base64
dXNlcjE=
echo -n 'pass1'| base64
cGFzczE=
cat << EOF > ./test-secret.txt
username=dXNlcjE=
password=cGFzczE=
EOF
cat test-secret.txt

kubectl create secret generic test-secret --from-file=./test-secret.txt --dry-run=client -oyaml > test-secret.yaml
kubectl apply -f test-secret.yaml
kubectl get secret/test-secret -oyaml

----
kubectl run nginx --image=nginx --dry-run=client -oyaml > test-sec1.yaml
vi test-sec1.yaml

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
