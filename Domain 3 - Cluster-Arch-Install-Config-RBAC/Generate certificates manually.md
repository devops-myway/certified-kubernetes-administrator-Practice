https://kubernetes.io/docs/tasks/administer-cluster/certificates/

###### Openssl
openssl can manually generate certificates for your cluster.

Generate a ca.key with 2048bit and ca.crt
``````sh
openssl genrsa -out ca.key 2048

openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt

``````
Generate a ca.key with 2048bit and Create a config file for generating a Certificate Signing Request (CSR).
``````sh
openssl genrsa -out server.key 2048

openssl req -new -key server.key -out server.csr -config csr.conf

``````
Generate the server certificate using the ca.key, ca.crt and server.csr:
``````sh
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out server.crt -days 10000 \
    -extensions v3_ext -extfile csr.conf -sha256

``````
View the certificate signing request:
``````sh
openssl req  -noout -text -in ./server.csr

``````
View the certificate:
``````sh
openssl x509  -noout -text -in ./server.crt
``````