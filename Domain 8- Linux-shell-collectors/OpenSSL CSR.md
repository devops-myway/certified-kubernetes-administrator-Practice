https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/


#### OPENSSL
OpenSSL is a software library for applications that secure communications over computer networks. secure TLS connection between an identity and an API.
OpenSSL contains an open-source implementation of the SSL and TLS protocols.

##### Privacy-Enhanced Mail (PEM):
is a de facto file format for storing and sending cryptographic keys, certificates, and other data.
The PEM format solves this problem by encoding the binary data using base64.
PEM also defines a one-line header, consisting of -----BEGIN, a label, and -----, and a one-line footer, consisting of -----END, a label, and -----.
Common labels include CERTIFICATE, CERTIFICATE REQUEST, PRIVATE KEY and X509 CRL.

PEM data is commonly stored in files with a

".pem" suffix, a ".cer"

or ".crt" suffix (for certificates),

or a ".key" suffix (for public or private keys).

##### Public Key Certificates
public key certificate, also known as a digital certificate or identity certificate, is an electronic document used to prove the ownership of a public key.
The certificate includes information about the key, information about the identity of its owner (called the subject), and the digital signature of an entity that has verified the certificate’s contents (called the issuer).
In a typical public-key infrastructure (PKI) scheme, the certificate issuer is a certificate authority (CA), usually a company that charges customers to issue certificates for them.
The most common format for public key certificates is defined by X.509.

##### X.509 Public Key Certificates

In cryptography, X.509 is a standard defining the format of public key certificates
X.509 certificates are used in many Internet protocols, including TLS/SSL, which is the basis for HTTPS, the secure protocol for browsing the web.
X.509 certificates are digital documents that represent a user, computer, service, or device.
They are issued by a certification authority (CA), subordinate CA, or registration authority or self-signed and contain the public key of the certificate subject.

##### Certificate Formats
- .pem – (Privacy-enhanced Electronic Mail) Base64 encoded DER certificate, enclosed between "-----BEGIN CERTIFICATE-----" and "-----END CERTIFICATE-----"
- .cer, .crt, .der – usually in binary DER form, but Base64-encoded certificates are common too

#### Creating a Private Key - RSA and ECDSA keys
A private key helps to enable encryption, and is the most important component of our certificate. In the commands below, replace [bits] with the key size (For example, 2048, 4096, 8192).
``````sh
mkdir priv-pub-keyfile
cd priv-pub-keyfile

openssl genrsa -out example.key [bits]
openssl genrsa -out domain.pem 2048
ls
cat private-key.pem

``````
##### Print public key or modulus only:
``````sh
openssl rsa -in private-key.pem -pubout -out public-key.pem
ls
cat public-key.pem

``````

#### Create certificate signing requests (CSR)
If you would like to obtain an SSL certificate from a commercial certificate authority (CA), you must generate a certificate signing request (CSR).
Whenever you generate a CSR, you will be prompted to provide information regarding the certificate. This information is known as a Distinguished Name (DN). An important field in the DN is the Common Name (CN), which should be the exact Fully Qualified Domain Name (FQDN) of the host that you intend to use the certificate with. CSRs can be used to request SSL certificates from a certificate authority.

If you want to non-interactively answer the CSR information prompt, you can do so by adding the -subj option to any OpenSSL commands that request CSR information.
``````sh
-subj "/O=Example Brooklyn Company/CN=examplebrooklyn.com"
------
openssl req -nodes -newkey rsa:[bits] -keyout example.key -out example.csr -subj "/C=UA/ST=Kharkov/L=Kharkov/O=Super Secure Company/OU=IT Department/CN=example.com"

openssl req -newkey rsa:2048 -keyout example.key -out example.csr -subj "Company/OU=IT Department/CN=example.com"

``````
##### Generate a CSR from an Existing Private Key

``````sh
openssl req \
       -key domain.key \
       -new -out domain.csr

``````
##### Generate a CSR from an Existing Certificate and Private Key

``````sh
openssl x509 \
       -in domain.crt \
       -signkey domain.key \
       -x509toreq -out domain.csr

``````

#### Creating a CA-Signed Certificate With Our Own CA
We can be our own certificate authority (CA) by creating a self-signed root CA certificate, and then installing it as a trusted certificate in the local browser

Create a Self-Signed Root CA:  create a private key (rootCA.key) and a self-signed root CA certificate (rootCA.crt) from the command line
``````sh
# Create a self signed certificate using existing CSR and private key
openssl x509 -req -in example.csr -signkey example.key -out example.crt -days 365  

# Create self-signed certificate and new private key from scratch:
openssl req -nodes -newkey rsa:2048 -keyout example.key -out example.crt -x509 -days 365

``````

##### View Certificates

``````sh
# Verify a CSR signature:
openssl req -in example.csr -verify

# Verify that private key matches a certificate and CSR:
openssl rsa -noout -modulus -in example.key | openssl sha256
openssl x509 -noout -modulus -in example.crt | openssl sha256
openssl req -noout -modulus -in example.csr | openssl sha256

# Verify certificate, provided that you have root
openssl verify example.crt

``````