

##### ENV variables in Kubernetes

ENV is an array. So every item under ENV property start with a dash (-) indicating an item in the array or list.
``````sh
env:
- name: VARIABLE1
  value: TEST

``````

However there are other ways of setting the environment variables such as using config maps and secrets.
The difference in this case is that instead of specifying the value, we say valueFrom and then specification.

##### Config Maps

We inject the config map when the POD is created. So the key-value pairs are available as environment variables.
There are two phases when configuring ConfigMap, so that the application deployed in the container inside the POD can consume them

Create the ConfigMap
Inject in into the POD.
##### Imperative way (command line)

``````sh
kubectl create configmap app-config --from-literal=VARIABLE1=foo --from-literal=VARIABLE2=bar

kubectl create configmap app-config --from-file=PATH TO THE FILE

``````
##### Declarative way (Definition file)
``````sh
apiVersion: v1
kind: ConfigMap
metadata:
   name:app-config
data:
  VARIABLE1: foo
  VARIABLE2: bar

``````
###### Inject ConfigMap into a  POD
 specify the config map under the containers section with envFrom
``````sh
envFrom:
  - configMapRef:
      name: app-config

----

env:
         - name: SPECIAL_LEVEL_KEY
           valueFrom:
             configMapKeyRef:
               name: special-config
               key: SPECIAL_LEVEL
         - name: SPECIAL_TYPE_KEY
           valueFrom:
             configMapKeyRef:
               name: special-config
               key: SPECIAL_TYPE
``````
##### Secrets
Secrets are designed to store sensitive data like usernames and passwords
There are two phases when configuring Secrets
Create the Secrets.
Inject in into the POD.

##### Imperative way (command line)
``````sh
kubectl create secret generic app-secret --from-literal=USER=foo --from-literal=PASSWORD=bar

kubectl create secret generic app-config --from-file=PATH TO THE FILE

``````
##### Declarative way (Definition file)
``````sh
apiVersion: v1
kind: Secret
metadata:
   name:app-secret
data:
  USER: Zm9v
  PASSWORD: YmFy

``````
##### How to refer it from POD
``````sh
envFrom:
  - secretRef:
        name: app-secret
``````
