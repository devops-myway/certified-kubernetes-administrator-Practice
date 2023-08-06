https://kubernetes.io/docs/setup/best-practices/certificates/
https://kubernetes.io/docs/tasks/administer-cluster/certificates/

###### Where certificates are stored
most certificates are stored in /etc/kubernetes/pki

Kubectl suddenly stops responding to your commands. Check it out! Someone recently modified the /etc/kubernetes/manifests/etcd.yaml file
You are asked to investigate and fix the issue. Once you fix the issue wait for sometime for kubectl to respond. Check the logs of the ETCD container.

first verify the pki certificate if it matches with the etcd client server certificates

The kube-api server stopped again! Check it out. Inspect the kube-api server logs and identify the root cause and fix the issue.
If we inspect the kube-apiserver container on the controlplane, we can see that it is frequently exiting.
This indicates an issue with the ETCD CA certificate used by the kube-apiserver. Correct it to use the file /etc/kubernetes/pki/etcd/ca.crt.

Once the YAML file has been saved, wait for the kube-apiserver pod to be Ready. This can take a couple of minutes.

##### View the certificate

``````sh
openssl x509  -noout -text -in ./server.crt

openssl x509  -noout -text -in /etc/kubernetes/pki/apiserver.crt

Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 258912386982333139 (0x397d771b607cad3)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Jul 24 22:25:27 2023 GMT
            Not After : Jul 23 22:25:27 2024 GMT
        Subject: CN = kube-apiserver
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:be:1b:98:9c:35:a5:09:f2:e6:9d:63:ea:b4:0e:
                    4a:d2:56:3c:68:bf:1c:a5:e3:71:57:8f:ae:06:26:
                    9f:04:6d:ea:6c:3e:81:42:5a:46:22:60:7b:fd:1b:
                    fa:ea:1a:7b:08:b4:4c:2b:f0:70:b3:64:af:83:be:
                    28:41:ca:b7:80:7b:c6:b4:c0:56:96:fb:4e:9a:72:
                    52:bd:01:b1:18:f3:89:2b:ce:86:38:07:1f:0c:76:
                    3f:b2:d6:79:80:52:e0:04:eb:85:0f:dd:8a:3c:d1:
                    be:13:fe:55:53:f1:48:fb:02:88:ce:39:c8:a7:b7:
                    c9:d4:90:8a:81:ee:07:e8:78:6d:cf:65:f7:85:18:
                    8d:81:68:5e:ca:c5:71:38:29:fa:2c:48:2f:a8:30:
                    d7:c8:c2:4b:67:24:45:e3:81:41:20:86:c5:db:20:
                    59:fc:5c:d3:24:68:f8:6f:3a:84:b2:6a:2c:06:59:
                    50:5a:e7:45:98:b8:21:23:9b:b2:31:f0:37:6b:2f:
                    d3:a2:07:1b:57:cf:9d:16:24:4b:9f:29:80:30:5e:
                    8b:34:95:3c:93:0a:f1:28:30:97:6d:84:b2:bc:4d:
                    ca:e3:5c:b6:63:15:bf:eb:ea:e2:37:78:1f:c6:7a:
                    4d:4a:e9:9e:5b:74:a1:ce:7a:2f:0b:1d:7c:9b:76:
                    49:a1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                keyid:FC:82:04:9E:A3:D1:30:9E:4F:45:DC:F2:AD:61:15:0F:C5:72:52:D3

``````
openssl x509  -noout -text -in /etc/kubernetes/pki/etcd/server.crt


            X509v3 Subject Alternative Name: 
                DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:10.96.0.1, IP Address:192.0.253.3
    Signature Algorithm: sha256WithRSAEncryption
         bb:ab:b8:35:5d:55:c2:b3:97:5d:e4:f0:72:8b:73:c5:54:5d:
         39:25:3a:1c:8e:86:a1:52:5e:6f:2f:d9:cf:20:63:f0:20:6c:
         ae:3c:3d:ca:46:50:6c:21:1c:00:a4:5e:9b:41:01:34:a4:b4:
         e6:8b:cc:06:40:26:3e:59:9d:35:63:7f:af:2e:61:75:84:ff:
         38:03:98:f4:78:f2:7c:65:ba:50:46:7c:80:02:88:54:e5:0a:
         48:be:a2:51:13:31:f8:98:06:e2:a6:9a:a3:db:2e:2f:6d:72:
         a0:25:1f:81:61:07:e6:a1:40:b5:29:0a:08:67:db:b8:a4:71:
         74:0e:3b:a9:5f:98:33:a0:6e:5b:3b:b5:3e:2b:53:13:1e:54:
         86:3f:4f:6c:b3:6c:d7:33:d9:72:e7:98:c4:67:8c:40:80:fd:
         60:80:65:93:54:e2:7c:00:7a:09:76:79:83:cc:16:8f:b2:58:
         4c:ef:67:b0:0d:77:32:f9:9d:9c:f2:1c:7c:f3:4e:77:07:0c:
         d3:a6:3f:38:63:48:0f:de:c6:83:ad:5e:80:ff:f0:c8:0f:f1:
         98:7f:db:e6:4b:d1:d9:d1:4f:0d:89:73:c6:c1:f9:6c:8c:0d:
         3c:ba:45:b9:2b:c4:c3:c1:cf:0f:50:e5:23:1b:56:a2:31:4a:
         6f:c9:73:ee

``````sh

``````

``````sh

``````

``````sh

``````
