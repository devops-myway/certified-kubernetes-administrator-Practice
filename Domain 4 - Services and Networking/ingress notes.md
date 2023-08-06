https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/basic-configuration/

https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/

###### Using Annotations or configuration
Annotations applied to an Ingress resource allow you to use advanced NGINX features and customize/fine tune NGINX behavior for that Ingress resource.

Customization and fine-tuning is also available through the ConfigMap. Annotations take precedence over the ConfigMap.

Ingress works fine if I put the correct value of the ingress-class either to spec.ingressClass property or to metadata.annotation.kubernetes.io/ingress.class accordingly. It gives an error like the following if you try to put both values to the same Ingres object:

But you cannot use both:
The Ingress "test-ingress" is invalid: annotations.kubernetes.io/ingress.class: Invalid value: "nginx": can not be set when the class field is also set
``````sh
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx

``````
##### The ingressClassName field is now supported:
``````sh
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: cafe-ingress
  spec:
    ingressClassName: nginx
    tls:
    - hosts:
      - cafe.example.com
      secretName: cafe-secret
    rules:
    - host: cafe.example.com
  . . .
``````
###### Load balancing 
An Ingress controller is bootstrapped with some load balancing policy settings that it applies to all Ingress, such as the load balancing algorithm, backend weight scheme, and others.

``````sh

``````
``````sh

``````
``````sh

``````
``````sh

``````