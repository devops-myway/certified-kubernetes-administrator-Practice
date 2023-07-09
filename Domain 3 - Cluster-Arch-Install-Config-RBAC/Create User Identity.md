
https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user
https://kubernetes.io/docs/reference/access-authn-authz/authentication/#x509-client-certs

Link to the documentation: https://github.com/schoolofdevops/ultimate-kubernetes-bootcamp/blob/master/chapters/configuring_authentication_and_authorization.md

/etc/kubernetes/pki (kubeadm)

#### Creating Kubernetes private key
``````sh
openssl genrsa -out myuser.key 2048
``````
#### step 2: create a Certification Signing Request (CSR) for each of the user.
``````sh
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser/O=org1"
ls
cat myuser.csr

``````
#### Decode the csr
request: is the base64 encoded value of the CSR file content. You can get the content using this command:
``````sh
cat myuser.csr | base64 | tr -d "\n"

``````
#### Create a CertificateSigningRequest
usages: has to be 'client auth'
expirationSeconds: could be made longer (i.e. 864000 for ten days) or shorter (i.e. 3600 for one hour)

``````sh
vi csr-request.yaml
---
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: ADD-Decoded-csr-here
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
``````
#### Approve the CertificateSigningRequest and Get the certificate

``````sh
kubectl get csr
kubectl certificate approve myuser
kubectl get csr/myuser -o yaml
kubectl get csr/myuser -ojson
``````
#### Export the issued certificate from the CertificateSigningRequest
The certificate value is in Base64-encoded format under status.certificate.
Export the issued certificate from the CertificateSigningRequest.

``````sh
kubectl get csr/myuser -ojson  # use jsonpath to filter the .status.certificate
kubectl get csr myuser -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
``````
##### Create Role and RoleBinding for our newly created user
``````sh
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
kubectl get role
kubectl describe role/developer

kubectl create rolebinding developer-binding-myuser --role=developer --user=myuser
kubectl get rolebinding
kubectl describe rolebinding/developer-binding-myuser
``````
##### Test the role for the user using auth can-i global commands
``````sh
kubectl auth can-i delete pods --as=myuser -n default
kubectl auth can-i create pods --as=myuser -n default
kubectl auth can-i list pods --as=myuser -n default
``````

#### Using Set Context for kubeconfig file entries
``````sh
kubectl config -h

kubectl config set-credentials myuser --client-key=myuser.key --client-certificate=myuser.crt
kubectl config set-context myuser-context --cluster=kubernetes --user=myuser
kubectl config use-context myuser-context    # change the context to myuser
kubectl config current-context
``````

