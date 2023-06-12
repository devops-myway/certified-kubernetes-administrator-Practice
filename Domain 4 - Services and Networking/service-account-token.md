https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

###### Using service account tokens to connect with the API server
A ServiceAccount is used by containers running in a Pod, to communicate with the API server of the Kubernetes cluster.

- A default service account is automatically created for each namespace.
-  Each pod is associated with exactly one service account but multiple pods can use the same service account.
- By default, the default service account in a namespace has no permissions other than those of an unauthenticated user
- The default permissions for a service account don't allow it to list or modify any resources. The default service account isn't allowed to view cluster state let alone modify it in any way.
- Therefore pods by default can’t even view cluster state. Its up to you to grant them appropriate permissions to do that.

##### Obtaining the service account token from the pod
A long-running service account is mounted in the /var/run/secrets/kubernetes.io/serviceaccount directory
The following three files are stored in this mounted directory:
- ca.crt - the certificate file that is needed for HTTPS access.
- namespace - the namespace scope of the associated token
- token - the service account token that is used for authentication

##### Using the service account token with kubectl

``````sh
kubectl get sa --all-namespaces | grep default
kubectl get sa default -o yaml
kubectl get secret default-token-dffkj -o yaml
``````
##### Custom serviceaccount
##### ServiceAccount
``````sh
apiVersion: v1
kind: ServiceAccount
metadata:
 name: demo-sa
``````
##### Role
``````sh
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
 name: list-pods
 namespace: default
rules:
 — apiGroups:
   — ''
 resources:
   — pods
 verbs:
   — list

``````
##### RoleBinding
``````sh
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
 name: list-pods_demo-sa
 namespace: default
roleRef:
 kind: Role
 name: list-pods
 apiGroup: rbac.authorization.k8s.io
subjects:
 — kind: ServiceAccount
   name: demo-sa
   namespace: default
``````
##### ServiceAccount within a Pod
The serviceAccountName key is specified and contains the name of the ServiceAccount used by that Pod, demo-sa
if the serviceAccountName is not specified in the Pod specification, the default ServiceAccount of the namespace is used.
``````sh
apiVersion: v1
kind: Pod
metadata:
 name: pod-demo-sa
spec:
 serviceAccountName: demo-sa
 containers:
 — name: alpine
   image: alpine:3.9
   command:
   — "sleep"
   — "10000"
``````
##### How can I check what the default service account is authorized to do
There isn't an easy way, but auth can-i may be helpful.
``````sh
kubectl auth can-i --list --as=system:serviceaccount:<namespace>:<serviceaccount> -n <namespace>

kubectl auth can-i --list --as=system:serviceaccount:default:demo-sa -n default
``````