##### Role-Based Access Control - RBAC API Primitives : Documentation Link

https://kubernetes.io/docs/reference/access-authn-authz/rbac/


##### Note:

- Role:
A collection of one or more rules defining the actions that users can take e.g: "create", "delete", "list, to be performed on API resources e.g pod, node etc.
A Role resources are namespaced and represent capabilities within a single namespace. You cannot use namespaced roles for nonnamespaced resources e.g CRDs.

- Rolebinding
An association between a role and one or more users or service accounts, called subjects. Each subject targeted by the binding can perform the actions covered by the role within the role namespace.

- ClusterRole and ClusterRoleBinding:
These are variants of Role and RoleBinding that exist at the cluster level. ClusterRoles and ClusterRoleBindings do not belong to a namespace and grant access across the entire cluster.
ClusterRoles only describe a role which can be reused across namespaces. It doesn't grant any permissions without a binding. I believe the following rule setup does exactly what you're describing:

To view all the api groups: Also note that below:
Continued deprecation of extensions/v1beta1, apps/v1beta1, and apps/v1beta2 APIs; these extensions will be retired in 1.16! Migrate to the apps/v1 API, available since v1.9.

```sh
kubectl api-resources -o wide
kubectl api-resources -o wide | grep apps/v1  # To find resources in the actual api-groups
kubectl api-resources -o wide | grep -i deployment

kubectl proxy --port=8181
curl localhost:8181
```
##### Role Resource and Verbs as List/arrays

The Kubernetes "" empty API group is a special group that refers to the built-in objects or core Api groups.

```sh
kubectl create -h
kubectl create role -h
kubectl create role NAME --verb=verb --resource=resource.group/subresource  # with specific api group

kubectl create role foo --verb=get,list,watch --resource=replicasets.apps

```
In Kubernetes, a collection of resources and verbs is called a Rule, and you can group rules into a list:
A collection of rules has a specific name in Kubernetes, and it is called a Role.
Each rule contains the apiGroups, resources and verbs that you just learned:
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: viewer
rules: # Authorization rules
  - apiGroups: # 1st API group
      - '' # An empty string designates the core API group.
    resources:
      - pods
      - pods/logs
      - serviceaccounts
    verbs:
      - get
      - list
      - watch
```
#### Make RBAC policies more concise
the above configurations can be re-written using the following format:
```sh
- apiGroups: ['']
  resources: ['services', 'endpoints', 'namespaces']
  verbs: ['get', 'list', 'watch', 'create', 'delete']
```
##### Checking API Access
To determine if the current user can perform a given action
Administrators can combine this with user impersonation to determine what action other users can perform.
to check whether a ServiceAccount named dev-sa in Namespace dev can list Pods in the Namespace target
```sh
kubectl auth can-i create deployments --namespace dev
kubectl auth can-i create deployments --namespace prod

kubectl auth can-i list secrets --namespace dev --as daves

kubectl auth can-i list pods --namespace target --as system:serviceaccount:dev:dev-sa
--as=system:serviceaccount:{namespace}:{service-account-name}

```

##### Making sense of Roles, RoleBindings, ClusterRoles, and ClusterBindings
At a high level, Roles and RoleBindings are placed inside and grant access to a specific namespace, while ClusterRoles and ClusterRoleBindings do not belong to a namespace and grant access across the entire cluster.
However, it is possible to mix these two types of resources.

#### Ex1: To start, create four namespaces
```sh
kubectl create namespace test
kubectl create namespace test2
kubectl create namespace test3
kubectl create namespace test4
```
##### create a Service Account in the test namespace

```sh
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myaccount
  namespace: test

kubectl apply -f service-account.yaml
```
##### Scenario 1: Role and RoleBinding in the same namespace
Let's start with creating a Role and a RoleBinding to grant the Service Account access to the test namespace:
All resources (the Service Account, Role, and RoleBinding) are in the test namespace.

```sh

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: testadmin
  namespace: test
rules:
  - apiGroups: ['*']
    resources: ['*']
    verbs: ['*']
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: testadminbinding
  namespace: test
subjects:
  - kind: ServiceAccount
    name: myaccount
    namespace: test
roleRef:
  kind: Role
  name: testadmin
  apiGroup: rbac.authorization.k8s.io

---
kubectl apply -f scenario1.yaml

kubectl auth can-i <verb> <resource>
kubectl auth can-i get pods -n test --as=system:serviceaccount:test:myaccount
yes
```
##### Scenario 2: Role and RoleBinding in a different namespace
Let's test if the Service Account located in test has access to the resources in test2
```sh
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: test2
  name: testadmin
rules:
  - apiGroups: ['*']
    resources: ['*']
    verbs: ['*']
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: testadminbinding
  namespace: test2
subjects:
  - kind: ServiceAccount
    name: myaccount
    namespace: test
roleRef:
  kind: Role
  name: testadmin
  apiGroup: rbac.authorization.k8s.io

-----
kubectl apply -f scenario2.yaml

kubectl auth can-i get pods -n test2 --as=system:serviceaccount:test:myaccount
yes
```
##### Scenario 3: Using a ClusterRole with a RoleBinding
ClusterRoles do not belong to a namespace. However, when a ClusterRole is linked to a Service Account via a RoleBinding, the ClusterRole permissions only apply to the namespace in which the RoleBinding was created.

```sh
kind: Cluster
apiVersion: rbac.authorization.k8s.io/v1alpha1
metadata:
  name: jenkins-deployer
rules:
- apiGroups: ["extensions"]
  resources: ["deployments"]
  verbs: ["*"]
  nonResourceURLs: ["/version"]
---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1alpha1
metadata:
  name: deploy-myapp
  namespace: default
subjects:
- kind: ServiceAccount
  name: jenkins-deployer
  namespace: default
roleRef:
  kind: ClusterRole
  name: jenkins-deployer

---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1alpha1
metadata:
  name: deploy-myapp
  namespace: myapp
subjects:
- kind: ServiceAccount
  name: jenkins-deployer
  namespace: default
roleRef:
  kind: ClusterRole
  name: jenkins-deployer

kubectl apply -f scenario3.yaml
kubectl auth can-i get pods -n test3 --as=system:serviceaccount:test:myaccount
#yes
kubectl auth can-i get pods -n test4 --as=system:serviceaccount:test:myaccount
#no
kubectl auth can-i get pods --as=system:serviceaccount:test:myaccount
#no
```
##### Create a ClusterRole and bind it to multiple namespaces using a RoleBinding.
We would like to create a ServiceAccount which is allowed to manage cronjobs in two namespaces, namespace1 and namespace2.

ServiceAccount to grant permissions to:
```sh
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa
  namespace: default

```
Now we create a ClusterRole
```sh
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: job-master
rules:
- apiGroups:
  - batch
  resources:
  - cronjobs
  verbs:
  - create
  - delete
  - deletecollection
  - get
  - list
  - patch
  - update
  - watch
```
We will now create two RoleBinding objects (not ClusterRoleBinding) which will use the ClusterRole job-master from above to allow the ServiceAccount sa to perform various actions (verbs) on cronjobs in two namespaces
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: job-master-1
  namespace: namespace1
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: job-master
subjects:
- kind: ServiceAccount
  name: sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: job-master-2
  namespace: namespace2
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: job-master
subjects:
- kind: ServiceAccount
  name: sa
  namespace: default
```
test
```sh
kubectl auth can-i get cronjobs -n namespace1 --as system:serviceaccount:default:sa
#yes
kubectl auth can-i delete cronjobs -n namespace2 --as system:serviceaccount:default:sa
#yes

kubectl auth can-i get cronjobs -n namespace3 --as system:serviceaccount:default:sa
#no

kubectl auth can-i get pod -n namespace2 --as system:serviceaccount:default:sa
#no
```
##### Scenario 4: Granting cluster-wide access with ClusterRole and ClusterRoleBinding
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: testadminclusterbinding
subjects:
  - kind: ServiceAccount
    name: myaccount
    namespace: test
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
-----
kubectl apply -f scenario3.yaml

kubectl auth can-i get pods -n test4 --as=system:serviceaccount:test:myaccount
#yes
kubectl auth can-i get namespaces --as=system:serviceaccount:test:myaccount
Warning: resource 'namespaces' is not namespace scoped
#yes
```

##### Assign clusterRole to a user
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: cluster-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: zeal
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io

--
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: User
  name: zeal
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io

```

##### Command-line utilities - kubectl create and role kubectl create clusterrole
```sh

kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
kubectl create role pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod
kubectl create role foo --verb=get,list,watch --resource=replicasets.apps
kubectl create role foo --verb=get,list,watch --resource=pods,pods/status
kubectl create role my-component-lease-holder --verb=get,list,watch,update --resource=lease --resource-name=my-component

kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
kubectl create clusterrole pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod
kubectl create clusterrole foo --verb=get,list,watch --resource=replicasets.apps

```
##### Command-line utilities - kubectl create rolebinding and kubectl create clusterrolebinding
```sh

kubectl create rolebinding bob-admin-binding --clusterrole=admin --user=bob --namespace=acme
kubectl create rolebinding myapp-view-binding --clusterrole=view --serviceaccount=acme:myapp --namespace=acme
kubectl create rolebinding myappnamespace-myapp-view-binding --clusterrole=view --serviceaccount=myappnamespace:myapp --namespace=acme

kubectl create clusterrolebinding root-cluster-admin-binding --clusterrole=cluster-admin --user=root
kubectl create clusterrolebinding kube-proxy-binding --clusterrole=system:node-proxier --user=system:kube-proxy
kubectl create clusterrolebinding myapp-view-binding --clusterrole=view --serviceaccount=acme:myapp

```
