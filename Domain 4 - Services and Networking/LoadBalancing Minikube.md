

##### Activate LoadBlancing in Minikube

The type is  LoadBalancer, but its  EXTERNAL-IPstatus is  Pending, since Minikube does not have a service with the LoadBalancer type, because it must be created at the infrastructure level – AWS, GCE, Azure, and then Kubernetes receives an IP or URL from them to route requests to this load balancer.

``````sh
kubectl get svc -n pritunl-local
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.103.96.24     <pending>     80:30046/TCP,443:31826/TCP   72s

``````
##### Minikube Tunnel - Recommended
use minikube tunnel – will create a tunnel between the host and the Service in Kubernetes
in another terminal run below:

``````sh
sudo minikube tunnel  # to avoid permission password issue

nginx-ingress-ingress-nginx-controller             LoadBalancer   10.103.96.24     127.0.0.1     80:30046/TCP,443:31826/TCP   3m33s
``````
##### Minikube service
Run  minikube service, specify a namespace and a name of the Service – Minicube will return the URL for connection to us:
``````sh
minikube service -n pritunl-local pritunl
|---------------|---------|--------------|-----------------------------|
|   NAMESPACE   |  NAME   | TARGET PORT  |             URL             |
|---------------|---------|--------------|-----------------------------|
| pritunl-local | pritunl | openvpn/1194 | http://192.168.59.108:32350 |
|---------------|---------|--------------|-----------------------------|
🎉  Opening service pritunl-local/pritunl in default browser...
``````

##### LoadBalancer externalIPs
``````sh
minikube ip

# Edit LoadBalancer:
kubectl -n pritunl-local edit svc pritunl
``````
``````sh

``````
``````sh

``````