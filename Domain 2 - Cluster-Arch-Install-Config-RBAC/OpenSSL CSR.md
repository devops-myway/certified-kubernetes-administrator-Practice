https://www.golinuxcloud.com/things-to-consider-when-creating-csr-openssl/

#### Root CA Certificates and the Certificate Chain
SSL certificates operate on a structure called the certificate chain — a network of certificates starting back at the issuing company of the certificate, also known as a certificate authority (CA).
These certificates consist of root certificates, intermediate certificates, and leaf (server) certificates.

Root CA certificates, these are certificates that are self-signed by their respective CA
Every valid SSL certificate is under a Root CA certificate, as these are trusted parties.

When you purchase a security certificate (typically, an SSL certificate), your certificate authority is supposed to send you the certificate – which is nothing but a bunch of files that includes a CA server certificate, intermediate certificate, and the private key.

PEM is a container format for digital certificates and keys, most notably used by Apache and other web server platforms.
A PEM file is often used for X.509 certificates, and it’s a text file that consists of Base64 encoding of the certificate text, a plain-text header, and footer marking the beginning and end of the certificate.

#### create openssl public and private keys pairs
``````sh
mkdir priv-pub-keyfile
cd priv-pub-keyfile

openssl genrsa -out private-key.pem 2048
ls
cat private-key.pem

``````

``````sh
# Create a public key from private key and with openssl command, openssl rsa -help commands

openssl rsa -in private-key.pem -pubout -out public-key.pem
ls
cat public-key.pem

``````

#### Problems which you can face with incorrect CSR

- Writing a CSR is the most crucial part of generating a certificate. If your CSR is not proper then:
- RootCA may fail to sign the certificate
- Your MTLS authentication will not work with TCP handshake error
- You will end up creating multiple certificates for each host if you are not familiar with SAN
- Your X.509 extensions will not be properly added

#### Important points to consider when creating CSR
- The openssl command will by default consider /etc/pki/tls/openssl.cnf as the configuration file unless you specify your own configuration file using -config
- The req_distinguished_name field is used to get the details which will be asked while generating the CSR. You can alter this section inside the openssl.cnf and add the default values, modify the conditions such as min and max allowed characters etc
- There are different policy sections available in the openssl.cnf. The policy_anything is normally used for self-signed certificates where all the fields except commonName are optional.
- commonName is used for MTLS communications. The commonName must match the HOSTNAME or FQDN of the server on the server certificate and client on the client certificate.
- The Common Name is considered as the most important field of any Certificate and must be filled cautiously, especially when generating server or client certificate
- To consider High Availability and Load balancing, in IT organizations we use single FQDN mapped to multiple IP Addresses so in such case we prefer to use SAN certificates

#### Generate my RootCA certificate or Create Certificate Authority Certificate.

Generate a private key for the rootCA certificate, we will use this private key to create Certificate Authority certificate

``````sh
 openssl genrsa -out ca.key 4096

  openssl genrsa -out cakey.pem 4096

 openssl req -new -x509 -days 365 -key cakey.pem -out cacert.pem

 ls -l cacert.pem

``````
#### Generate and sign certificate using openssl x509 command
We now generate a Certificate Signing Request which contains some of the info that we want to be included in the certificate. To prove ownership of the private key, the CSR is signed with the subject's private key server.key. Think carefully when inputting a Common Name (CN) as you generate the .csr file below. This should match the DNS name, or the IP address you specify in your Apache configuration.

``````sh
openssl genrsa -out server.key.pem 4096

openssl req -new -key server.key.pem -out server.csr

 ls -l cacert.pem

``````
#### OpenSSL verify Certificate Signing Request (CSR)
``````sh
openssl req -noout -text -in server.csr

``````

####  Sign a certificate with CA
In this command we will issue this certificate server.crt, signed by the CA root certificate ca.cert.pem and CA key ca.key which we created in the previous command.

Openssl takes your signing request (csr) and makes a one-year valid signed server certificate (crt) out of it. In doing so, we need to tell it which Certificate Authority (CA) to use, which CA key to use, and which Server key to sign. We set the serial number using CAcreateserial, and output the signed key in the file named server.crt.

to connect to our web server using the client certificates. Use --key to define the client key file, --cert to define the client certificate and --cacert to define the CA certificate we used to sign the certificates followed by the web server address.

--key client.key.pem
--cert client.cert.pem

``````sh
openssl x509 -req -in server.csr  -CA /root/tls/certs/cacert.pem -CAkey /root/tls/private/cakey.pem -out server.cert.pem  -CAcreateserial -CAserial serial -days 365 -sha256

openssl x509 -req -days 365 -in server.csr -CA ca.cert.pem -CAkey ca.key -CAcreateserial -out server.crt

``````
#### OpenSSL verify server certificate

``````sh
openssl x509 -noout -text -in server.crt

``````