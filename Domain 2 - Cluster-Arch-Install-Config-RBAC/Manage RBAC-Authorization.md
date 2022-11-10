##### Role-Based Access Control - RBAC API Primitives : Documentation Link

https://kubernetes.io/docs/reference/access-authn-authz/authorization/

--------------------------

##### Note:

Role and ClusterRole: An RBAC Role or ClusterRole contains rules that represent a set of permissions
----------------------
- Role:
A collection of one or more rules defining the actions that users can take e.g: "create", "delete", "list, to be performed on API resources e.g pod, node etc.
A Role always sets permissions within a particular namespace; when you create a Role, you have to specify the namespace it belongs in.

- Rolebinding
An association between a role and one or more users or service accounts, called subjects. Each subject targeted by the binding can perform the actions covered by the role.

- ClusterRole and ClusterRoleBinding:
These are variants of Role and RoleBinding that exist at the cluster level. They’re used to create rules that target cluster-level resources such as nodes.

##### Implementing RBAC In Your Kubernetes Cluster
Run the below commands to see if RBAC is enabled.
```sh
kubectl api-versions
kubectl api-versions | grep rbac.authorization.k8s

```
##### create role and assign the role to test user
-----------
```sh
# imaparatively creating role

kubectl create role read-only --verb=list,get,watch --resource=pods,deployments,services

kubectl get roles

kubectl describe role read-only

```
```sh
# create a role declaratively with nano pod-reader.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

```
##### create RoleBinding
To bind a Role to the RoleBinding, use the --role command-line option. The subject
type can be assigned by declaring the options --user, --group, or --serviceaccount.

```sh
# imaparatively - create rolebinding
kubectl create rolebinding read-only-binding --role=read-only --user=johndoe

# create a RoleBinding examples

nano role-to-rolebinding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: read-only-binding
roleRef:
apiGroup: rbac.authorization.k8s.io
kind: Role
name: read-only
subjects:
- apiGroup: rbac.authorization.k8s.io
kind: User
name: johndoe

```
```sh
kubectl apply -f role-to-rolebinding.yaml

kubectl get rolebindings

kubectl describe rolebinding read-only-binding

```
##### Seeing the RBAC Rules in Effect
we’ll create a new Deployment with the cluster-admin credentials to test the RBAC rules.
```sh
kubectl config current-context

kubectl create deployment myapp --image=nginx --port=80 --replicas=2

kubectl config use-context johndoe-context

kubectl get deployments

kubectl get replicasets  # error here

```

##### create ClusterRole
  
ClusterRole example
------------------
ClusterRoles are cluster-scoped, you can also use them to grant access to:

cluster-scoped resources (like nodes)

non-resource endpoints (like /healthz)

namespaced resources (like Pods), across all namespaces

##### Step 2: Create a Cluster Role
```sh
nano cluster-role.yaml
```
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```
```sh
kubectl apply -f cluster-role.yaml
```
```sh
kubectl get clusterrole
kubectl describe clusterrole pod-reader
```

#### Step 3: Create a Cluster Role Binding
```sh
nano cluster-role-binding.yaml
```
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: list-pods-global
subjects:
- kind: User
  name: system:serviceaccount:default:external
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```sh
kubectl cluster-role-binding.yaml
```
```sh
kubectl get clusterrolebinding
kubectl describe clusterrolebinding list-pods-global
```

##### Step 4: Verify Permissions
```sh
curl -k <K8S-SERVER-URL>/api/v1/namespaces/default/pods --header "Authorization: Bearer <TOKEN-HERE>"
```
##### Step 5: Create a new Namespace
```sh
kubectl create namespace external
```

#### Checking API Access:
-------------------
kubectl auth can-i create deployments --namespace dev

kubectl auth can-i create deployments --namespace prod

kubectl auth can-i list secrets --namespace dev --as dave









