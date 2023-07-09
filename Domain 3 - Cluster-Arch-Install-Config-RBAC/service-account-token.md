https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
https://kubernetes.io/docs/reference/kubectl/jsonpath/

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
kubectl describe sa/default

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
kubectl api-resources
kubectl api-resources | grep v1
kubectl create role list-pods --namespace=default --verb=list --resource=pods."" --dry-run=client -oyaml > role1.yaml

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
kubectl create rolebinding -h
kubectl create rolebinding listpods-rolebinding --role=list-pods --serviceaccount=default:demo-sa --dry-run=client -oyaml > rolebinding1.yaml

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
kubectl auth can-i -h
kubectl auth can-i list,get resources --as=system:serviceaccount:<namespace>:<serviceaccount> -n <namespace>

kubectl auth can-i list pods --as=system:serviceaccount:default:demo-sa -n kube-system
kubectl auth can-i list pods --as=system:serviceaccount:default:demo-sa
kubectl auth can-i create pods --as=system:serviceaccount:default:demo-sa --all-namespaces #No
``````
##### Manually create an API token for a ServiceAccount and secret

In Kubernetes 1.24, ServiceAccount token secrets are no longer automatically generated.
Use the TokenRequest API to acquire service account tokens.

``````sh
kubectl create sa demo-sa
kubectl create token demo-sa
``````
##### Manually create a long-lived API token for a ServiceAccount

You can still manually create a service account token Secret; for example, if you need a token that never expires. However, using the TokenRequest subresource to obtain a token to access the API is recommended instead.
If you want to obtain an API token for a ServiceAccount, you create a new Secret with a special annotation, kubernetes.io/service-account.name.


``````sh
kubectl create sa demo-sa 
kubectl describe sa/demo-sa
kubectl get sa/demo-sa -oyaml

----
vi demo-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: demo-sa-secret
  annotations:
    kubernetes.io/service-account.name: demo-sa
type: kubernetes.io/service-account-token
--
kubectl apply -f demo-secret.yaml
kubectl get sa/demo-sa -oyaml
kubectl get secret demo-sa-secret --namespace default -o json | jq -r '.data["ca.crt"]' | base64 -d
kubectl get secret demo-sa-secret --namespace default -o json | jq -r '.data["token"]' | base64 -d 
kubectl get secret demo-sa-secret --namespace default -o json | jq -r '.data["namespace"]' | base64 -d
----
kubectl get secret demo-sa-secret -o jsonpath='{.data.token}'| base64 -d > token.txt
kubectl get secret demo-sa-secret -o jsonpath='{.data.ca\.crt}'| base64 -d > ca-demo.key
cat token.txt
cat ca-demo.key

``````

#### Using Set Context for kubeconfig file entries
``````sh
kubectl config -h

kubectl config set-credentials demo-sa --client-key=ca-dmo.key --token=token.txt
kubectl config set-context demo-sa-context --cluster=kubernetes --user=demo-sa --namespace=default
kubectl config current-context
kubectl config get-contexts
kubectl config use-context demo-sa-context
kubectl config view

``````