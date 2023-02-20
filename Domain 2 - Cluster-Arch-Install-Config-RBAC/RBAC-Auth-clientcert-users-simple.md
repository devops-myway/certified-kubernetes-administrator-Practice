Link to the documentation: https://github.com/schoolofdevops/ultimate-kubernetes-bootcamp/blob/master/chapters/configuring_authentication_and_authorization.md

#### Role Based Access Control (RBAC) - Creating Kubernetes Users and Groups
``````sh
# Generate the user's private key
mkdir priv-keyfile
cd priv-keyfile

openssl genrsa -out maya.key 2048
ls
cat maya.key

``````
#### step 2: create a Certification Signing Request (CSR) for each of the user.
``````sh
# CN: This will be set as username or common name and O: Org name or group

openssl req -new -key maya.key -out maya.csr -subj "/CN=maya/O=ops/O=example.org"
ls
cat maya.csr

``````
#### step 3: In order to be deemed authentic, these CSRs need to be signed by the Certification Authority (CA) which in this case is Kubernetes Master
You need access to the folllwing files on kubernetes master.
Certificate : ca.crt (kubeadm)
Private Key : ca.key (kubeadm)
You would typically find it at one of the following paths:
/etc/kubernetes/pki (kubeadm)

``````sh
openssl x509 -req -CA ca.pem -CAkey ca-key.pem -CAcreateserial -days 730 -in maya.csr -out maya.crt
ls
cat maya.crt

``````
#### step 4: Setting up User configs with kubectl
Add credentials in the configurations
Set context to login as a user to a cluster
Switch context in order to assume the user's identity while working with the cluster

``````sh
kubectl config set-credentials maya --client-certificate=/absolute/path/to/maya.crt --client-key=/absolute/path/to/maya.key

kubectl config get-contexts

kubectl config set-context maya-prod --cluster=prod  --user=maya --namespace=instavote

kubectl config get-contexts

kubectl config view

kubectl config use-context yono-prod

kubectl get pods

``````

#### step 5: Define authorisation rules with Roles and ClusterRoles
Role is limited to a namespace (Projects/Orgs/Env)
ClusterRole is Global

``````sh
#  interns-role.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  namespace: instavote
  name: interns
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]

------------ # interns-rolebinding.yml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: interns
  namespace: instavote
subjects:
- kind: Group
  name: interns
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: interns
  apiGroup: rbac.authorization.k8s.io

``````
#### step 6: Apply the files and test it 
``````sh
kubectl create -f interns-role.yml

kubectl create -f interns-rolebinding.yml

----- To gt information about the objects created above,
kubectl get roles -n instavote
kubectl get roles,rolebindings -n instavote

kubectl describe role interns
kubectl describe rolebinding interns
---------To validate the access,
kubectl config use-context yono-prod
kubectl get pods

----------To switch back to admin
kubectl config use-context admin-prod

``````