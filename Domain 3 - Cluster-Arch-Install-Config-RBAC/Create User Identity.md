
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
``````
#### Export the issued certificate from the CertificateSigningRequest

``````sh
kubectl get csr myuser -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
``````

#### Using Set Context for kubeconfig file entries.

``````sh
kubectl config -h

kubectl config set-credentials --
kubectl config set-cluster --
kubectl config set-context --

kubectl config set-credentials myuser --client-key=myuser.key --client-certificate=myuser.crt
kubectl config set-context myuser-context --cluster=kubernetes --user=myuser
kubectl config use-context myuser-context    # change the context to myuser

kubectl --context=myuser-context get pods  # Use Context to Verify
``````

