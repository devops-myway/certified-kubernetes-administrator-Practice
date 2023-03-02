

#### Introducing kubectl configuration
kubectl uses this configuration file to determine where and how to issue requests to the API server.
By default, this file is located at $HOME/kube/config


##### Understanding kubeconfig
The kubectl configuration file compose of three top-level structures: users, clusters, and contexts.

Cluster: the URL for the API of a Kubernetes cluster. This URL identifies the cluster itself.

Users: credentials that identify a user connecting to the Kubernetes cluster.

Context: puts together a cluster (the API URL) and a user (who is connecting to that cluster). The context serves as the means by which kubectl connects and authenticates to an API server.

``````sh
$ export KUBECONFIG=/tmp/config

mv /path/to/kubeconfig ~/.kube

kubectl config option

$ kubectl config view

``````
##### Defining Clusters
The kubectl config set-cluster command allows you to create a new cluster connection by using the API URL

``````sh
$ kubectl config set-cluster my-cluster --server=127.0.0.1:8087

 kubectl config get-clusters

``````
##### Defining Users
use x509 client certificates to authenticate our user. Kubernetes has no User Objects.
User account consists of an authorized certificate that is completed with some authorization as defined in RBAC.
##### Step 1: Create User with private key
First we will create a normal user "user1" which will be part of wheel group
``````sh
openssl genrsa -out user1.key 2048

``````

##### Step 2: Certificate Signing Request 

``````sh
openssl req -new -key user1.key -out user1.csr -subj "/CN=user1/O=dev"

``````
##### Step 3: create the certificate

``````sh
openssl x509 -req -in user1.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out user1.crt -days 365

``````
##### Step 4:  Create namespace (optional)

``````sh
kubectl create namespace dev

``````
##### Step 5:  Update Kubernetes Config file with User Credentials
Since we are using x509 certificates for authentcating the user, the same needs to be added in the kubernetes config file.

``````sh
kubectl config view   # Following is the default kubeconfig file:

-----------
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.43.48:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

----------- #set user1 credentials to config file
kubectl config set-credentials user1 --client-certificate=user1.crt --client-key=user1.key
kubectl config get-users

 kubectl config view  # verify the kubernetes config file
 -----
 apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.43.48:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: user1
  user:
    client-certificate: /root/user1.crt
    client-key: /root/user1.key
``````
##### Step 6: Create security context for new user
A context defines which user to use with which cluster, but can also define the namespace that kubectl should use, when you don’t specify the namespace explicitly with the --namespace or -n option.
To create a new context user1-context that ties together the cluster and the user you created
``````sh
kubectl config set-context user1-context --cluster=kubernetes --namespace=dev --user=user1

kubectl config get-contexts

``````
##### Step 7: verify if you permission to list pods under this context

``````sh
kubectl --context=user1-context get pods

----
Error from server (Forbidden): pods is forbidden: User "user1" cannot list resource "pods" in API group "" in the namespace "dev"
``````
As expected user1 is not allowed to access user1-context at the moment as we have not yet assigned a role and rolebinding to this user.

A role is a set of abstract capabilities. For example, the appdev role might represent the ability to create Pods and services.
A role binding is an assignment of a role to one or more identities. Thus, binding the appdev role to the user identity alice indicates that Alice has the ability to create Pods and services.

##### Step 8: Create new role

``````sh
[root@controller ~]# cat dev-role.yml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
   namespace: dev
   name: dev
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["list", "get", "watch", "create", "update", "patch", "delete"]
--------------
kubectl create -f dev-role.yml
``````
##### Step 9: Bind a user to the newly created role

``````sh
[root@controller ~]# cat user1-rolebind.yml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
   name: dev-role-binding
   namespace: dev
subjects:
- kind: User
  name: user1
  apiGroup: ""
roleRef:
  kind: Role
  name: dev
  apiGroup: ""
--------------
 kubectl create -f user1-rolebind.yml
``````

##### Step 10: Verify the RBAC Policy

``````sh
kubectl --context=user1-context get pods

kubectl -n dev describe role dev

``````

##### Step 10: Create a new deployment resource under user1-context to verify if user1 has enough permission

``````sh
kubectl create deployment devnginx --image=nginx --context=user1-context

kubectl get pods

kubectl get pods -n dev
``````
##### Step 11: Testing Authorization with can-i

``````sh
 kubectl auth can-i create pods --context=user1-context

 kubectl auth can-i create service --context=user1-context
``````
##### Step 12: Deleting contexts

``````sh
kubectl config delete-context user1-context

kubectl config delete-context viewonly-context
``````
##### Step 13: Deleting Role and RoleBinding

``````sh
kubectl delete role dev -n dev

kubectl delete rolebinding dev-role-binding -n dev
``````
