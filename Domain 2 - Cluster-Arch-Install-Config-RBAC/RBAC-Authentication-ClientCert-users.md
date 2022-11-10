##### Documentation Link:

https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#create-certificatesigningrequest

Creating a Subject using Client Certificate Authentication (Cluster Certificate of Authority-CA)
--------------
##### Read Below: RBAC High-Level Overview

All Kubernetes clusters have two categories of users: service accounts managed by Kubernetes, and normal users.
Any user that presents a valid certificate signed by the cluster's certificate authority (CA) is considered authenticated.
Enabling and configuring RBAC is mandatory for any organization with a strong emphasis on security.

RBAC helps with implementing a variety of use cases:
----------------------------------------------
- Establishing a system for users with different roles to access a set of Kubernetes resources
- Controlling processes running in a Pod and the operations they can perform via the Kubernetes API
- Limiting the visibility of certain resources per namespace

RBAC consists of three key building blocks which are below terminologies:
--------------------------------------------------------------------
- Subject
The user, groups, and serviceaccount that wants to access a resource. Calls to the API server with a user need to be authenticated.
Users and groups are not stored in etcd, the Kubernetes database, and are meant for processes running outside of the cluster.
Service accounts exists as objects in Kubernetes and are used by processes running inside of the cluster

- Resource
The Kubernetes API resource type (e.g., a Deployment,secrets,confgimaps,pods or node)

- Verb
The operation that can be executed on the resource (e.g., creating a Pod or deleting a Service)

When using kubeadm to install a cluster, the API server is configured with the option:

--client-ca-file=/etc/kubernetes/pki/ca.crt

##### Steps for Creating a Subject/User accounts and groups Authentication with Openssl with X.509 client certificate
First, this user e.g (dev-tom) must have a certificate issued by the Kubernetes cluster, and then present that certificate to the Kubernetes API server.

##### Step 0.0: Steps to creating a user/subject using client certificate to authenticate against our API server
--------------------
login to the control plane node/master node and create a temp. directory that would hold the generated private keys.

``````sh

mkdir cert && cd cert

``````
#### Step1: Generate a PKI private key (ca.key or dev-tom.key) with 2048bit:
----------------------------------
``````sh
# Create a private key
$ openssl genrsa -out myuser.key 2048
ls
cat ca.key

``````

##### Step2: Create a certificate sign request (CSR) in a file with the extension .csr.
--------------------------------------------------------------------------------------------------
It is important to set CN and O attribute of the CSR. The -subj option provides the
username (CN) and the group (O).
``````sh
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser/O=cka-study"
ls

``````
#### Step 2.1: Decode the csr
```sh
# add the csr content to your request section of the file below
cat myuser.csr | base64 | tr -d '\n'

```

#### step3: Generate the CertificateSigningRequest and get the request:
---------------
Create a CertificateSigningRequest and submit it to a Kubernetes Cluster via kubectl. using the below script
```sh
nano csr-requests.yaml

# copy the template into the csr-requests.yaml file

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser-csr
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth

# Some points to note:

- usages: has to be 'client auth'
- expirationSeconds: could be made longer (i.e. 864000 for ten days) or shorter (i.e. 3600 for one hour)
- request: is the base64 encoded value of the CSR file content. You can get the content using this command:
cat myuser.csr | base64 | tr -d "\n"

#### Apply the Signing Requests:

kubectl apply -f csr-requests.yaml

# You can optionally verify with:

kubectl get csr

```

#### step4: Approve certificate signing request
-----------------------
Use kubectl to create a CSR and approve it
```sh
kubectl certificate approve myuser-csr

# Get the list of CSRs:
kubectl get csr

```
##### step5: Get the certificate
-----------------------
Retrieve the certificate from the CSR.
The certificate value is in Base64-encoded format under status.certificate.

``````sh
kubectl get csr/myuser-csr -o yaml

``````
##### step 6: Export the issued certificate from the CertificateSigningRequest.
-----------------------------------------------
``````sh
kubectl get csr myuser-csr -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt

``````
#### Step 7: Create a new context - use kubectl config --help to Add the user into kubeconfig
```sh
kubectl config set-credentials myuser --client-key=myuser.key --client-certificate=myuser.crt
```
#### Step 8: Set new Context
```sh
kubectl config set-context myuser --cluster=kubernetes --user=myuser
```
#### Step 9: To test it, change the context to myuser
```sh
kubectl config use-context myuser
```

##### Create Role and RoleBinding
----------------------------
With the certificate created it is time to define the Role and RoleBinding for this user to access Kubernetes cluster resources.

```sh
# Create Role Imparatively

kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods

# RoleBinding
kubectl create rolebinding developer-binding-myuser --role=developer --user=myuser

```
#### Step 10: Create RBAC Role Allowing List PODS Operation

```sh
nano role.yaml
```
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```
```sh
kubectl apply -f role.yaml
```

#### Step 11: Create a New Role Binding
```sh
nano rolebinding.yaml
```
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: myuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```sh
kubectl apply -f rolebinding.yaml
```

#### Step 12: Verify Permissions
```sh
kubectl --context=myuser get pods
```
#### Step 13: Delete Resources Created in this Lab
```sh
kubectl delete -f role.yaml
kubectl delete -f rolebinding.yaml
```