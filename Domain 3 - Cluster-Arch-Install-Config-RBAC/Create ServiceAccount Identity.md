https://kubernetes.io/docs/reference/access-authn-authz/authentication/#users-in-kubernetes


###### What is a service account?
Service accounts are usually created automatically by the API server and associated with pods running in the cluster through the ServiceAccount Admission Controller.
Bearer tokens are mounted into pods at well known locations, and allow in cluster processes to talk to the API server.
Accounts may be explicitly associated with pods using the serviceAccountName field of a PodSpec.

``````sh
kubectl get serviceaccount -n default

kubectl get serviceaccount default -n default -oyaml
----
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2021-01-26T09:12:15Z"
  name: default
  namespace: default
  resourceVersion: "422"
  uid: 8843bb73-e090-4d48-9cd6-6d8717740af1
secrets:
- name: default-token-srs4v
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
spec:
  replicas: 3
  template:
    metadata:
    # ...
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
      serviceAccountName: bob-the-bot
---
# To test if this service account has permissions
kubectl auth can-i get pods --as=system:serviceaccount:default:demo-sa -n default #No
``````
The above shows that this service account has a set of credentials mounted in a secret volume.
In this particular case, the secret is called default-token-srs4v.

To check the content of this secret run this command: We can see that the secret includes a ca.crt file and a token . The ca.crt is the public CA of the API server and the token is a signed JSON Web Token (JWT).
``````sh
kubectl get secret default-token-srs4v -o yaml

``````

#### Creating ServiceAccount resource
A default ServiceAccount is automatically created for each namespace. You can list ServiceAccounts like you do other resources:
serviceaccount is sa for short
``````sh
kubectl get sa

kubectl get sa --all-namespaces
``````
#### Using kubectl command

``````sh
kubectl create serviceaccount user1

kubectl get sa user1 -oyaml

``````
#### Using YAML file for declarative

``````sh
vim service-account.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: user2

kubectl apply -f service-account.yaml
kubectl get sa user2 -o yaml

``````
#### Add ImagePullSecrets to a service account
If your images are available in a private registry (like, for example, the Docker Hub, Quay.io or a self-hosted registry), you will need to configure your Kubernetes cluster so that it is authorized to actually access these images.
You can now use this newly created Secret object when creating a new Pod, by adding an imagePullSecrets attribute to the Pod specification:
``````sh
kubectl create secret docker-registry my-private-registry --docker-server https://index.docker.io/v1/ --docker-username <your-username> --docker-password <your-password> --docker-email <your-email>

----
apiVersion: v1
kind: Pod
metadata:
  name: example-from-private-registry
spec:
  containers:
  - name: secret
    image: quay.io/devs-private-registry/secret-application:v1.2.3
  imagePullSecrets:
  - name: my-private-registry

``````
#### Assign ServiceAccount to a Pod
You can use spec.serviceAccountName field in the pod definition to assign a ServiceAccount. Here I am creating a simple nginx pod using our user1 ServiceAccount.

``````sh
 cat nginx.yaml
--- 
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
spec:
  serviceAccountName: user1
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

kubectl apply -f nginx.yaml

----
kubectl exec -it nginx -- awk 1 /var/run/secrets/kubernetes.io/serviceaccount/token

------ # You can get the secret token id for your ServiceAccount using:
kubectl describe sa user1 | grep "Tokens:"

``````
#### Access API server using ServiceAccount within Pod
We will create a busybox pod and assign user2 ServiceAccount which we created earlier in this tutorial.

``````sh
cat nginx.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
spec:
  serviceAccount: user1
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

kubectl apply -f nginx.yaml

----
kubectl exec -it nginx -- awk 1 /var/run/secrets/kubernetes.io/serviceaccount/token
kubectl describe sa user1 | grep "Tokens:"

``````
#### Access API server using ServiceAccount within Pod
We will create a busybox pod and assign user2 ServiceAccount which we created earlier in this tutorial.

``````sh
vi busybox.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  serviceAccountname: user2
  containers:
  - name: busybox
    image: progrium/busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always

----
kubectl create -f busbybox.yaml
kubectl get pods
kubectl exec -it busybox -- opkg-install curl
kubectl exec -it busybox -- /bin/sh

# and make sure that you are able to resolve kubernetes.default using CoreDNS.
 nslookup kubernetes.default

------- #Now let us try to access our API server using curl
curl --insecure   https://kubernetes.default.svc/
``````
#### Assign Role and RoleBinding for ServiceAccount
It is possible in some cases the ServiceAccount may not be able to access the API server due to RBAC restrictions.

#### Create ServiceAccount
``````sh
kubectl create sa user3

``````
#### Create Role
A Role resource defines what actions can be taken on which resources. I will create a separate role which allows to list the Pods in default namespace:
``````sh
vi list-pods.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: list-pods
  namespace: default
rules:
  - apiGroups:
    - ''
    resources:
    - pods
    verbs:
    - list

---
kubectl apply -f list-pods.yaml 
kubectl get roles
``````
#### Create RoleBinding
A Role defines what actions can be performed, but it doesn’t specify who can perform them.
To do that, you must bind the Role to a subject, which can be a user(real users), a Service-Account(applications, argo-cd, db, web servers, agents etc), or a group (of users or ServiceAccounts).

``````sh
cat list-pods-binding.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: list-pods-user3-binding
  namespace: default
roleRef:
  kind: Role
  name: list-pods
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: user3
    namespace: default

---
kubectl apply -f list-pods-binding.yaml
kubectl get rolebindings
``````
#### Create a Pod
We will create a new pod using user3 ServiceAccount

``````sh
cat user3-busybox.yaml 
---
apiVersion: v1
kind: Pod
metadata:
  name: user3-busybox
  namespace: default
spec:
  serviceAccountname: user3
  containers:
  - name: busybox
    image: progrium/busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always

---
kubectl apply -f user3-busybox.yaml

``````

#### Verify API server access from the Pod
Next we will install the curl package:

``````sh
kubectl exec -it busybox -- opkg-install curl
kubectl exec -it busybox -- /bin/sh
kubectl apply -f user3-busybox.yaml

``````
