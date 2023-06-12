
https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/
https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx
https://computingforgeeks.com/deploy-metallb-load-balancer-on-kubernetes/
https://computingforgeeks.com/deploy-nginx-ingress-controller-on-kubernetes-using-helm-chart/


####  Overview on Kubernetes Ingress

Kubernetes offers ingress which is an API object, that exposes HTTP and HTTPS routes from outside the cluster to services running within the cluster.An Ingress may be configured to: 

- Provide Services with externally-reachable URLs
- Load balance traffic coming into cluster services
- Terminate SSL / TLS traffic
- Provide name-based virtual hosting in Kubernetes

Ingresses do not work like other Services in Kubernetes. Just creating the Ingress itself will do nothing. You need two additional components:

1 - An Ingress controller: Ingress controllers do not come installed in a Kubernetes cluster by default, so you have to set one up yourself, built on tools such as Nginx or HAProxy. https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
The ingress controller is the software component that brings the Ingress object to life.

2 - Ingress Resource - contains a set of routing rules based on which traffic is routed to a service object.

3 - ClusterIP or NodePort Services objects or Resource for the intended routes.

##### Ingress Controller Configuration Categories
The NGINX ingress controller has additional configuration options that can be customized and configured to create a more dynamic application. Basically, this can be done in two ways.
- Annotations: this option can be used if you want a specific configuration for a particular ingress rule.
- ConfigMap: this option can be used when you need to set global configurations for the NGINX ingress controller.

Note: annotations take precedence over a ConfigMap.

##### Enable ingress controller add-on
 Kubernetes adopts a BYOS (Bring-Your-Own-Software) approach to most of its addons and it doesn’t provide a software that does Ingress functions out of the box.
 You can choose from the:
https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

``````sh
kubectl get pods -n kube-system

-- # check below - nginx-ingress-controller pod is up and running properly.
nginx-ingress-controller-6fc5bcc8c9-wnkfs   1/1     Running   0          111s
``````

##### Step 1: Deploy Nginx Ingress Controller in Kubernetes - Install without Helm
Option 1: Install without Helm
With this method you’ll manually download and run deployment manifests using kubectl command line tool.

``````sh
# Debian / Ubuntu
sudo apt update
sudo apt install wget curl git
``````
Step 2: Apply Nginx Ingress Controller manifest
https://kubernetes.github.io/ingress-nginx/deploy/#aws

Install ingress on Bare metal clusters:
Kubernetes clusters deployed on bare metal servers manually, using generic Linux distros (like CentOS, Ubuntu...)
``````sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/baremetal/deploy.yaml

``````
You can update your current context to use nginx ingress namespace:
``````sh
kubectl config set-context -h
kubectl config set-context --current --namespace=ingress-nginx  #add an entry for the namespace to our config file

# Once the ingress controller pods are running, you can cancel the command typing Ctrl+C.
kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --watch

# If you want to run multiple Nginx Ingress Pods
kubectl -n ingress-nginx scale deployment ingress-nginx-controller --replicas 2
kubectl get pods  -n ingress-nginx
``````

#### Option 2: Deploy Nginx Ingress Controller in Kubernetes - Install with Helm
Install Helm on Ubuntu or bare metal server
https://helm.sh/docs/intro/install/
``````sh
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
helm -h
helm version
``````
Step 2: Deploy Nginx Ingress Controller

use this repo: https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx
To use, add ingressClassName: nginx spec field or the kubernetes.io/ingress.class: nginx annotation to your Ingress resources.
``````sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install [RELEASE_NAME] ingress-nginx/ingress-nginx
helm uninstall [RELEASE_NAME]

``````
``````sh
kubectl create namespace ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx
kubectl get all -n ingress-nginx
kubectl get pods -n ingress-nginx
kubectl get pods --all-namespaces
``````
To check logs in the Pods use the commands.
To follow logs as they stream run
``````sh
kubectl -n ingress-nginx  logs deploy/ingress-nginx-controller
kubectl -n ingress-nginx  logs deploy/ingress-nginx-controller -f

``````

##### Upgrading Helm Release
I’ll set replica count of the controller Pods to 2
``````sh
vim values.yaml

kubectl -n ingress-nginx  get deploy
---
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           43m

helm upgrade -n ingress-nginx ingress-nginx
kubectl -n ingress-nginx  get deploy
``````
##### Uninstalling the Chart
``````sh
helm -n ingress-nginx uninstall ingress-nginx
``````
###### Step 2: Configure Nginx Ingress Controller on Kubernetes - Option 1: Using Load Balancer (Highly recommended)
Load balancer is used to expose an application running in Kubernetes cluster to the external network.
It provides a single IP address to route incoming requests to Ingress controller application.
Setting Nginx Ingress to use MetalLB - https://metallb.universe.tf/installation/

``````sh
# Check Nginx Ingress service.
kubectl get svc -n ingress-nginx
---
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.108.4.75      <none>        80:30084/TCP,443:30540/TCP   5m55s
ingress-nginx-controller-admission   ClusterIP   10.105.200.185   <none>        443/TCP                      5m55s
``````
Patch ingress-nginx-controller service by setting service type to LoadBalancer.
Confirm successful patch of the service.

``````sh
kubectl get service ingress-nginx-controller --namespace=ingress-nginx
kubectl -n ingress-nginx patch svc ingress-nginx-controller --type='json' -p '[{"op":"replace","path":"/spec/type","value":"LoadBalancer"}]'
service/ingress-nginx-controller patched

kubectl get service ingress-nginx-controller --namespace=ingress-nginx
----
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.108.4.75   192.168.1.30   80:30084/TCP,443:30540/TCP   10m
``````

##### 2. Mapping DNS name for Nginx Ingresses to LB IP
The mapping is *.k8s.example.com pointing to IP address 192.168.1.30 (Nginx Ingress LB IP).
``````sh

``````
###### Deploy Services to test Nginx Ingress functionality

Create namespace, test Pods and Services YAML file
``````sh
kubectl create namespace demo

``````
``````sh
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple"
---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - port: 5678 # Default port for image
---
kind: Pod
apiVersion: v1
metadata:
  name: banana-app
  labels:
    app: banana
spec:
  containers:
    - name: banana-app
      image: hashicorp/http-echo
      args:
        - "-text=banana"
---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
spec:
  selector:
    app: banana
  ports:
    - port: 5678 # Default port for image

----
kubectl apply -f demo-app.yml -n demo
``````
##### Test if it is working
``````sh
kubectl get pods -n demo
kubectl -n demo logs apple-app
``````
##### Create Ubuntu pod that will be used to test service connection.
``````sh
cat <<EOF | kubectl -n demo apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  containers:
  - name: ubuntu
    image: ubuntu:latest
    command: ["/bin/sleep", "3650"]
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
EOF

``````
``````sh
kubectl exec -it ubuntu -- bash -n demo
---
apt update && apt install curl -y
curl apple-service:5678
curl banana-service:5678

``````
##### Creating an ingress route
declare an Ingress to route requests to /apple to the first service, and requests to /banana to second service
``````sh
kubectl explain ingress
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: webapp.k8s.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: web-server-service
              port:
                number: 80
----
kubectl -n web apply -f  webapp-app-ingress.yml
kubectl get ingress -n web
kubectl get pods -n ingress-nginx
curl http://webapp.k8s.example.com/

``````

##### Create Ingress Rule

Using the kubernetes.io/ingress.class annotation (in deprecation): there are a few things to note below on the ingress manifest.

- Host: is specified as localhost, so the rule applies to the host. If a host is not specified, then all inbound HTTP traffic goes through the IP address.
- Path: can consist of multiple rules; in the example above, the path rule points to a Service within the Backend definition.
- Backend: is a combination of Service and port, as seen above.

Ingress YAML Explanation:
- ingressClassName: This field is used for the selection of ingress controller, similar to the selector field of services
https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/
- service:  denotes the name of the service on which the request has to be forwarded

``````sh

# $ cat nginx-ingress-rule.yml
apiVersion: networking.k8s.io/v1beta1kind: Ingress
metadata:
   name: nginx-ingress
   annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
spec:
   rules:
   - host: host.example.com
     http:
       paths:
       - path: /
         backend:
            serviceName: nginx
            servicePort: 80
--
kubectl apply -f nginx-ingress-rule.yml
kubectl get ingress

 kubectl get ing nginx-ingress -o yaml

``````
##### Verify the Kubernetes Ingress rule
``````sh

curl http://host.example.com/
 
``````
#####  Configure Kubernetes Ingress using Path
``````sh

kubectl create deployment web2 --image=nginx

kubectl scale deployment web2 --replicas=3

kubectl get pods

``````
#####  Expose the deployment pods to external network (Create a service)
``````sh

kubectl expose deployment web2 --type=NodePort --port=80

kubectl get svc

``````
#####  6.3 Configure ingress rule
We will modify the existing ingress rule and add one more path section as shown below:
So here we want to access the new web2 server using the same hostname i.e. host.example.com but at a different path i.e. /v2
``````sh
# $ cat nginx-ingress-rule.yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
   name: nginx-ingress
   annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
spec:
   rules:
   - host: host.example.com
     http:
       paths:
       - path: /
         backend:
            serviceName: nginx
            servicePort: 80
       - path: /v2
         backend:
            serviceName: web2
            servicePort: 80
--
kubectl apply -f nginx-ingress-rule.yml

kubectl get ing nginx-ingress -o yaml

-- # access and verify the kubernetes ingress rule

curl http://host.example.com/v2
``````
#####  Exposing a service through an Ingress

An Ingress object references one or more Service objects. Your first Ingress object exposes the kiada service.

The service exposes ports 80 and 443, the Ingress will forward traffic only to port 80.

``````sh
apiVersion: v1
kind: Service
metadata:
  name: kiada
spec:
  type: ClusterIP
  selector:
    app: kiada
  ports:
  - name: http
    port: 80
    targetPort: 8080

``````
#####  Creating the Ingress object

Ingress object exposing the kiada service at kiada.example.com

``````sh
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kiada-example-com
spec:
  rules:
  - host: kiada.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kiada
            port:
              number: 80

----
kubectl get ingresses

kubectl describe ing kiada-example-com

kubectl get ing kiada -o yaml

``````
#####  Path-based ingress traffic routing
Now you’ll create one for the quote and quiz services.

The Ingress object for these two services makes them available through the same host: api.example.com

- all requests with the path /quote are forwarded to the quote service
- all requests whose path starts with /questions are forwarded to the quiz service

``````sh
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-example-com
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /quote
        pathType: Exact
        backend:
          service:
            name: quote
            port:
              name: http
      - path: /questions
        pathType: Prefix
        backend:
          service:
            name: quiz
            port:
              name: http

----
kubectl get ingresses
kubectl describe ing kiada-example-com
kubectl get ing kiada -o yaml

-- # access the two services it exposes as follows (replace the IP with that of your ingress):
$ curl --resolve api.example.com:80:11.22.33.44 api.example.com/quote
$ curl --resolve api.example.com:80:11.22.33.44 api.example.com/questions/random

``````
###### Matching paths using the Exact path type
how matching works when pathType is set to Exact

Path in rule	       Matches request path	          Doesn’t match
/	                      /	                            /foo or /bar
/foo	                  /foo	                        /foo/ or /bar
/foo/	                 /foo/	                       /foo or /foo/bar or /bar
/FOO	                 /FOO	                           /foo

###### Matching paths using the Prefix path type
pathType is set to Prefix, things aren’t as you might expect. Matches based on a URL path prefix split by /


Path in rule	          Matches request paths	                                            Doesn’t match
/	                     All paths; for example: /, /foo, /foo/
/foo or /foo/             /foo, /foo/, /foo/bar  	                                         /foobar or /bar
/FOO	   

#####  Creating an Ingress object with multiple rules-name Based Virtual Hosting
Using annotation for the ingress class: https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/
If you're running multiple ingress controllers where one or more do not support IngressClasses, 
you must specify the annotation kubernetes.io/ingress.class: "nginx" in all ingresses that you would like ingress-nginx to claim.

``````sh
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kiada
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: kiada.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kiada
            port:
              name: http
  - host: api.example.com
    http:
      paths:
      - path: /quote
        pathType: Exact
        backend:
          service:
            name: quote
            port:
              name: http
      - path: /questions
        pathType: Prefix
        backend:
          service:
            name: quiz
            port:
              name: http
----
kubectl get ingresses

kubectl describe ing kiada-example-com

kubectl get ing kiada -o yaml

``````
##### Using wildcards in the host field

Host	                                      Matches request hosts	                                                 Doesn’t match
kiada.example.com	                          kiada.example.com	                                                     example.com, api.example.com, foo.kiada.example.com

*.example.com	                       kiada.example.com, api.example.com, foo.example.com,example.com                 foo.kiada.example.com

##### Specifying the default backend in an Ingress object

If the client request doesn’t match any rules defined in the Ingress object, the response 404 Not Found is normally returned.
you can also define a default backend to which the ingress should forward the request if no rules are matched. The default backend serves as a catch-all rule.

``````sh
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kiada
spec:
  defaultBackend:
    service:
      name: fun404
      port:
        name: http
  rules:
  ...
``````
##### Creating the service and pod for the default backend


``````sh
apiVersion: v1
kind: Pod
metadata:
  name: fun404
  labels:
    app: fun404
spec:
  containers:
  - name: server
    image: luksa/static-http-server
    args:
    - --listen-port=8080
    - --response-code=404
    - --text=This isn't the URL you're looking for.
    ports:
    - name: http
      containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: fun404
  labels:
    app: fun404
spec:
  selector:
    app: fun404
  ports:
  - name: http
    port: 80
    targetPort: http

----
curl api.example.com/unknown-path --resolve api.example.com:80:11.22.33.44

``````

##### Using annotations to enable cookie-based session affinity in Nginx ingresses
 because Ingresses operate at L7, they can support cookie-based session affinity
using Nginx-ingress-specific annotations to enable cookie-based session affinity and configure the session cookie name

``````sh

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kiada
  annotations:
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/session-cookie-name: SESSION_COOKIE
spec:
  ...
----
curl -I http://kiada.example.com --resolve kiada.example.com:80:11.22.33.44

``````
