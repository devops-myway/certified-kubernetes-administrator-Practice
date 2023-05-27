
https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/
https://loft.sh/blog/kubernetes-nginx-ingress-10-useful-configuration-options/


####  Overview on Kubernetes Ingress

Kubernetes offers an ingress resource and controller that is designed to expose Kubernetes services to the outside world. It can do the following:

- Provide an externally visible URL to your service
- Load balance traffic
- Terminate SSL
- Provide name-based virtual hosting

Ingresses do not work like other Services in Kubernetes. Just creating the Ingress itself will do nothing. You need two additional components:

- An Ingress controller: Ingress controllers do not come installed in a Kubernetes cluster by default, so you have to set one up yourself, built on tools such as Nginx or HAProxy. https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
The ingress controller is the software component that brings the Ingress object to life.

- Ingress Resource - contains a set of routing rules based on which traffic is routed to a service.

- ClusterIP or NodePort Services or Resource for the intended routes.

##### Ingress Controller Configuration Categories
The NGINX ingress controller has additional configuration options that can be customized and configured to create a more dynamic application. Basically, this can be done in two waysç
- Annotations: this option can be used if you want a specific configuration for a particular ingress rule.
- ConfigMap: this option can be used when you need to set global configurations for the NGINX ingress controller.

Note: annotations take precedence over a ConfigMap.

##### Enable ingress controller add-on

When you want to expose a set of services externally, you create an Ingress object and reference the Service objects in it.

Once the add-on is enabled, you can verify the status of the Pod:

``````sh
kubectl get pods -n kube-system

-- # check below - nginx-ingress-controller pod is up and running properly.
nginx-ingress-controller-6fc5bcc8c9-wnkfs   1/1     Running   0          111s
``````

##### 5. Configure Kubernetes Ingress using Host
``````sh
kubectl create deployment nginx --image=nginx

kubectl scale deployment nginx --replicas=3
--
kubectl get deployments

kubectl get pods
``````
##### 5.2 Expose the deployment (Create a service)
``````sh
kubectl expose deployment nginx --type=NodePort --port=80

kubectl get service

``````
##### Access the container using external network
``````sh
ip a

So my interface IP is 172.17.0.34 which means I can access my nginx server at http://172.17.0.34:30745.

curl http://172.17.0.34:30745

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
pathType is set to Prefix, things aren’t as you might expect


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
##### Ingress Controllers
Ingress controllers in Kubernetes are resources that accept traffic from the internet and load balance it to applications (usually in the form of running pods).
An Ingress controller is a daemon running in a Pod that watches the /ingresses endpoint on the API server. When a new endpoint is created, the daemon uses the configured set of rules to allow traffic into a service.

There are many Ingress controllers and NGINX Ingress controllers as NGINX-based controllers seem to be the most common.
NGINX is a general-purpose implementation compatible with most Kubernetes deployments.

##### How to Set Up an NGINX Ingress Controller:
To run your own NGINX Ingress Controller, you can use the process documented in this:
https://kubernetes.github.io/ingress-nginx/deploy/
https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/
