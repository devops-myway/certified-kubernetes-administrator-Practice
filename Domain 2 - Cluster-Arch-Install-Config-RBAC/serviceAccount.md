

#### Overview on Kubernetes Service Accounts- Notes

Interaction with a Kubernetes cluster involves communication with the API server. This communication is in the form of requests sent to the API server. When a request is received by the API server it first tries to authenticate the request, if this authentication fails the request is treated as an anonymous request. This means that every process inside or outside the cluster, from a human typing kubectl on his workstation to the kubelet process running on a node must authenticate when making a request to the API server.

Kubernetes has two types of users, normal users and service account users. Service accounts are maintained by the Kubernetes cluster. Service accounts are meant to represent the processes running in pods in the cluster.
Service accounts on the other hand are tied to a specific namespace. These are created automatically by the API server or manually through API calls. This ensures namespace isolation.
Whenever we create a pod in a cluster, Kubernetes assigns that pod a service account. This service account identifies the pod process as it interacts with the API server.
However, assigning a service account to a pod or deployment is optional. This means that if our manifest file doesn’t specify a service account, our pod will be deployed and it will still be able to communicate with the API server.

The reason behind the above behaviour is that whenever we create a new namespace in our cluster, Kubernetes creates a default service account for us in that namespace.
When a pod has no service account in its manifest and is deployed to that namespace, that pod will be assigned the default service account.

###### What is a service account?

To understand what a service account is? we will examine the default service account created in the default namespace of a Kubernetes cluster.
To get the service account in the default namespace run the command and To see more details about this service account run the command:
``````sh
kubectl get serviceaccount -n default

kubectl get serviceaccount default -n default -o yaml
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

``````
The above shows that this service account has a set of credentials mounted in a secret volume.
In this particular case, the secret is called default-token-srs4v.

To check the content of this secret run this command: We can see that the secret includes a ca.crt file and a token . The ca.crt is the public CA of the API server and the token is a signed JSON Web Token (JWT).
These secrets are mounted into the pod for in-cluster access to the API server.
``````sh
kubectl get secret default-token-srs4v -o yaml

``````


##### Understanding ServiceAccount resource

- ServiceAccounts (db, argocd, sqs, http clients, web servers etc) are resources just like Pods, Secrets, ConfigMaps, and so on, and are scoped to individual namespaces. a pod can only use a ServiceAccount from the same namespace.
- A default ServiceAccount is automatically created for each namespace (that’s the one your pods have used all along).
- Every Pod uses the default ServiceAccount to contact the API server.
- This default ServiceAccount allows a resource to get information from the API server. The API server obtains this information from the system-wide authorization plugin configured by the cluster administrator. One of the available authorization plugins is the role-based access control (RBAC) plugin.
- Service accounts come with a secret which contains the API credentials

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
[root@controller ~]# vim service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: user2

kubectl create -f service-account.yaml

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
After you have created ServiceAccount, you can start assigning them to respective pods.
You can use spec.serviceAccountName field in the pod definition to assign a ServiceAccount. Here I am creating a simple nginx pod using our user1 ServiceAccount.
To confirm that the custom ServiceAccount’s token is mounted into the two containers, you can compare the content of token from /var/run/secrets/kubernetes.io/serviceaccount/token within the Pod and the secret token part of user1 ServiceAccount i.e. user1-token-jg85r

``````sh
[root@controller ~]# cat nginx.yaml
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

kubectl create -f nginx.yaml

----
kubectl exec -it nginx -- awk 1 /var/run/secrets/kubernetes.io/serviceaccount/token

------ # You can get the secret token id for your ServiceAccount using:
kubectl describe sa user1 | grep "Tokens:"

``````
#### Access API server using ServiceAccount within Pod
We will create a busybox pod and assign user2 ServiceAccount which we created earlier in this tutorial.

``````sh
[root@controller ~]# cat nginx.yaml
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

kubectl create -f nginx.yaml

----
kubectl exec -it nginx -- awk 1 /var/run/secrets/kubernetes.io/serviceaccount/token

------ # You can get the secret token id for your ServiceAccount using:
kubectl describe sa user1 | grep "Tokens:"

``````
#### Access API server using ServiceAccount within Pod
We will create a busybox pod and assign user2 ServiceAccount which we created earlier in this tutorial.

``````sh
[root@controller ~]# cat busybox.yaml 
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
[root@controller ~]# cat list-pods.yaml 
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
kubectl create -f list-pods.yaml 
kubectl get roles
``````
#### Create RoleBinding
A Role defines what actions can be performed, but it doesn’t specify who can perform them.
To do that, you must bind the Role to a subject, which can be a user(real users), a Service-Account(applications, argo-cd, db, web servers, agents etc), or a group (of users or ServiceAccounts).

``````sh
[root@controller ~]# cat list-pods-binding.yaml 
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
kubectl create -f list-pods-binding.yaml
kubectl get rolebindings
``````
#### Create a Pod
We will create a new pod using user3 ServiceAccount

``````sh
[root@controller ~]# cat user3-busybox.yaml 
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
kubectl create -f user3-busybox.yaml

``````

#### Verify API server access from the Pod
Next we will install the curl package:

``````sh
kubectl exec -it busybox -- opkg-install curl

kubectl exec -it busybox -- /bin/sh
---
kubectl create -f user3-busybox.yaml

``````