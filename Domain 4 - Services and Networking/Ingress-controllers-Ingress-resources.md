
https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/
https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx
https://computingforgeeks.com/deploy-metallb-load-balancer-on-kubernetes/
https://computingforgeeks.com/deploy-nginx-ingress-controller-on-kubernetes-using-helm-chart/


####  Overview on Kubernetes Ingress

Kubernetes offers ingress which is an API object, that exposes HTTP and HTTPS routes from outside the cluster to services running within the cluster.An Ingress may be configured to: 

- Provide Services with externally-reachable URLs
- Load balance service/traffic coming into cluster services
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

##### Step 1: Deploy Nginx Ingress Controller in Kubernetes - Install without Helm
Option 1: Using Minikube Host

``````sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo ls
helm repo update
helm search repo ingress
helm install nginx-ingress ingress-nginx/ingress-nginx --dry-run
helm install nginx-ingress ingress-nginx/ingress-nginx
``````
``````sh
helm ls
kubectl get svc
### result below
NAME                                               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.103.96.24     <pending>     80:30046/TCP,443:31826/TCP   72s
``````
In another terminal run to create Loadblancer external IP
``````sh
sudo minikube tunnel
kubectl get svc
# Result for the external-Ip for loadbalancing activated
NAME                                               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ginx-ingress-ingress-nginx-controller             LoadBalancer   10.103.96.24     127.0.0.1     80:30046/TCP,443:31826/TCP   3m33s
``````

#### Deploy an Nginx instance and service to test the controller.

``````sh
kubectl create deployment nginx --image=nginx --port=80 --replicas=2 --dry-run=client -oyaml > dep.yaml
kubectl apply -f dep.yaml

expose deploy/nginx --port=8181 --target-port=80 --name=ngnx-svc --type=NodePort --dry-run=client -oyaml > svc1.yaml
kubectl apply -f svc1.yaml
``````
##### Deploy a simple Ingress Resource
``````sh
vi ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
spec:
  ingressClassName: nginx
  rules:
  - host: vishalvyas.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: nginx-svc
              port:
                number: 8181
--
kubectl get svc                            
kubectl get ing

curl http://vishalvyas.com/
``````
``````sh

``````

##### Upgrading Helm Release
I’ll set replica count of the controller Pods to 2
``````sh
vim values.yaml

kubectl get deploy -n ingress-nginx 
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

###### Example2: Deploy Services to test Nginx Ingress functionality
https://hub.docker.com/r/hashicorp/http-echo/
Create namespace, test Pods and Services YAML file
``````sh
kubectl create namespace demo

``````
``````sh
cat << EOF | kubectl apply -f -
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
EOF
---
cat << EOF | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - port: 5677 # Default port for image
EOF
---
cat << EOF | kubectl apply -f -
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
EOF
---
cat << EOF | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: banana-service
spec:
  selector:
    app: banana
  ports:
    - port: 5678 # Default port for image
EOF
----

``````
##### Test if it is working
``````sh
kubectl get pods -n demo
kubectl -n demo logs apple-app
``````
##### Create Ubuntu pod that will be used to test service connection.
``````sh
kubectl run testpod --image=ubuntu -- sh -c 'sleep 3600'
kubectl exec -it testpod -- sh
apt update && apt install curl -y
apt imnstall iputils-ping   # for ping ip of service
apt install dnsutils       # for dns service name checks nslookup 
apt instll curl

curl apple-service:5677
# apple
curl banana-service:5678
#banana
``````

##### Creating an ingress resrouce - with Simple fanout
A fanout configuration routes traffic from a single IP address to more than one Service, based on the HTTP URI being requested.
declare an Ingress to route requests to /apple to the first service, and requests to /banana to second service
``````sh
kubectl explain ingress
---
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-fanout
spec:
  ingressClassName: nginx
  rules:
  - host: vishalvyas.com
    http:
      paths:
      - path: /apple
        pathType: Prefix
        backend:
          service:
            name: apple-service
            port:
              number: 5677
      - path: /banana
        pathType: Prefix
        backend:
          service:
            name: banana-service
            port:
              number: 5678
EOF
----
kubectl -n web apply -f  webapp-app-ingress.yml
kubectl get ingress -n web
kubectl get pods -n ingress-nginx
curl http://vishalvyas.com/

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
