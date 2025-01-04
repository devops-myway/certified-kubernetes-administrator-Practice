https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
https://stackoverflow.com/questions/10175812/how-to-generate-a-self-signed-ssl-certificate-using-openssl


#### OPENSSL
OpenSSL is a software library for applications that secure communications over computer networks. secure TLS connection between an identity and an API.
OpenSSL contains an open-source implementation of the SSL and TLS protocols.

##### Privacy-Enhanced Mail (PEM):
is a de facto file format for storing and sending cryptographic keys, certificates, and other data.
- The PEM format solves this problem by encoding the binary data using base64.
- PEM also defines a one-line header, consisting of -----BEGIN, a label, and -----, and a one-line footer, consisting of -----END, a label, and -----.
- Common labels include CERTIFICATE, CERTIFICATE REQUEST, PRIVATE KEY and X509 CRL.

PEM data is commonly stored in files with a

- ".pem" suffix, a ".cer"
- or ".crt" suffix (for certificates),
- or a ".key" suffix (for public or private keys).

##### Public Key Certificates
- public key certificate, also known as a digital certificate or identity certificate, is an electronic document used to prove the ownership of a public key.
- The certificate includes information about the key, information about the identity of its owner (called the subject), and the digital signature of an entity that has verified the certificate’s contents (called the issuer).
- In a typical public-key infrastructure (PKI) scheme, the certificate issuer is a certificate authority (CA), usually a company that charges customers to issue certificates for them.
- The most common format for public key certificates is defined by X.509.

##### X.509 Public Key Certificates

In cryptography, X.509 is a standard defining the format of public key certificates
- X.509 certificates are used in many Internet protocols, including TLS/SSL, which is the basis for HTTPS, the secure protocol for browsing the web.
- X.509 certificates are digital documents that represent a user, computer, service, or device.
- They are issued by a certification authority (CA), subordinate CA, or registration authority or self-signed and contain the public key of the certificate subject.

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
openssl genrsa -out server.key 2048
ls
cat private-key.pem

``````
##### Print public key:
``````sh
openssl rsa -in private-key.pem -pubout -out public-key.pem
openssl rsa -in server.key -out server.key
ls
cat public-key.pem

``````

#### Create certificate signing requests (CSR)
- If you would like to obtain an SSL certificate from a commercial certificate authority (CA), you must generate a certificate signing request (CSR).
- Whenever you generate a CSR, you will be prompted to provide information regarding the certificate. This information is known as a Distinguished Name (DN).
- An important field in the DN is the Common Name (CN), which should be the exact Fully Qualified Domain Name (FQDN) of the host that you intend to use the certificate with.
- CSRs can be used to request SSL certificates from a certificate authority.
- Replace 'localhost' with whatever domain you require. You will need to run the first two commands one by one as OpenSSL will prompt for a passphrase.
``````sh
-subj "/O=Example Brooklyn Company/CN=examplebrooklyn.com"
------
openssl req -nodes -newkey rsa:[bits] -keyout example.key -out example.csr -subj "/C=UA/ST=Kharkov/L=Kharkov/O=Super Secure Company/OU=IT Department/CN=example.com"

openssl req -newkey rsa:2048 -keyout example.key -out example.csr -subj "Company/OU=IT Department/CN=example.com"
openssl req -sha256 -newkey server.key -out server.csr -subj '/CN=localhost'
or 
### View the certificate:
openssl x509  -noout -text -in ./server.crt
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text

``````
#### Creating a CA-Signed Certificate With Our Own CA
We can be our own certificate authority (CA) by creating a self-signed root CA certificate, and then installing it as a trusted certificate in the local browser

- Create a Self-Signed Root CA:  create a private key (rootCA.key) and a self-signed root CA certificate (rootCA.crt) from the command line
- https://security.stackexchange.com/questions/91913/why-is-it-fine-for-certificates-above-the-end-entity-certificate-to-be-sha-1-bas
- using SHA-2 does not add any security to a self-signed certificate.
- But I still recommend using it as a good habit of not using outdated / insecure cryptographic hash functions
``````sh
# Create a self signed certificate using existing CSR and private key
openssl req -x509 -sha256 -newkey rsa:2048 -keyout key.pem -out cert.pem -days XXX

openssl x509 -req -in example.csr -signkey example.key -out example.crt -days 365
openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt

# To combine the two into a .pem file:
cat server.crt server.key > cert.pem

``````
for the flag -subj | -subject empty values are permitted -subj "/C=/ST=/L=/O=/OU=web/CN=www.server.com", but you can sets more details as you like:

- C - Country Name (2 letter code)
- ST - State
- L - Locality Name (eg, city)
- O - Organization Name
- OU - Organizational Unit Name
- CN - Common Name - required!

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
##### 
As of 2023 with OpenSSL ≥ 1.1.1, the following command serves all your needs, including: The following files are generated:

- Private key: example.com.key
- Certificate: example.com.crt

Each of the below commands creates a certificate that is:

- valid for the domain example.com (SAN),
- also valid for the wildcard domain *.example.com (SAN),
- also valid for the IP address 10.0.0.1 (SAN),
- relatively strong (as of 2023) and
- valid for 3650 days (~10 years).

``````sh
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
  -nodes -keyout example.com.key -out example.com.crt -subj "/CN=example.com" \
  -addext "subjectAltName=DNS:example.com,DNS:*.example.com,IP:10.0.0.1"
``````
##### Parameters
-  Remark #1: Crypto parameters
- Since the certificate is self-signed and needs to be accepted by users manually, it doesn't make sense to use a short expiration or weak cryptography.
- In the future, you might want to use more than 4096 bits for the RSA key and a hash algorithm stronger than sha256, but as of 2023 these are sane values
- Remark #2: Parameter "-nodes"
- Theoretically you could leave out the -nodes parameter (which means "no DES encryption"), in which case example.key would be encrypted with a password. However, this is almost never useful for a server installation, because you would either have to store the password on the server as well, or you'd have to enter it manually on each reboot.


