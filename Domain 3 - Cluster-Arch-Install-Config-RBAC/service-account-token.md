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
kubectl get sa -A  # show all ssa across all ns
kubectl get sa -A | grep default # show all the default sa on ns
kubectl get sa default -oyaml
--
# To test the default sa attached to the pod
kubectl run nginx --image=nginx 
kubectl describe po/nginx
kubectl describe sa/default -oyaml

kubectl exec -it po/nginx -- sh
ls -l /var/run/secrets/kubernetes.io/serviceaccount
----
lrwxrwxrwx 1 root root 13 Jun 24 14:05 ca.crt -> ..data/ca.crt
lrwxrwxrwx 1 root root 16 Jun 24 14:05 namespace -> ..data/namespace
lrwxrwxrwx 1 root root 12 Jun 24 14:05 token -> ..data/token
---
cat token
cat namespace
cat ca.crt
``````
##### Custom serviceaccount
##### ServiceAccount
``````sh
kubectl create sa demo-sa -n default --dry-run=client -oyaml> demo-sa.yaml
kubectl apply -f demo-sa.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
 name: demo-sa
``````
##### Role For Autorization
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
##### RoleBinding for Autorization
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
kubectl run pod-demosa --image=busybox --dry-run=client -oyaml -- sh -c "sleep 3600" > podsa.yaml
kubectl apply -f podsa.yaml
# manually add the serviceAccountName
kubectl explain pod.spec 

apiVersion: v1
kind: Pod
metadata:
 name: pod-demosa
spec:
 serviceAccountName: demo-sa
 containers:
 — name: busybox
   image: busybox
   command:
   — "sleep"
   — "10000"
``````
##### How can I check what the default service account is authorized to do
There isn't an easy way, but auth can-i may be helpful. You must be allowed to use impersonation for the global option "--as".
``````sh
# To check if our sa demo-sa in default ns can list services in the kube-system ns.
kubectl auth can-i --list --as=system:serviceaccount:<namespace>:<serviceaccount> -n <namespace>

kubectl auth can-i --list --as=system:serviceaccount:default:demo-sa -n kube-system
``````
