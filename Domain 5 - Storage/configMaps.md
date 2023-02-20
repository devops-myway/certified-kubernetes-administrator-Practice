
#### Overview on Kubernetes ConfigMaps
- A ConfigMap allows us to define application-related data.
- It decouples the application data from the application so that the same application can be ported across different environments.
- It also provides a way to inject customized data into running services from the same container image.
- The key thing is that the ConfigMap is combined with the Pod right before it is run. This means that the container image and the Pod definition itself can be reused across many apps by just changing the ConfigMap that is used.

#### How to create ConfigMap
ConfigMaps can be created through a literal value or from a file or all the files in a directory.

with the command kubectl create configmap <map-name> <data-source>

<map-name> : is the name you want to assign to the ConfigMap
<data-source>: is the directory, file, or literal value to draw the data from. This corresponds to a key-value pair in the ConfigMap

Key: is the filename or the key you provided on the command line
Value: is the file content or the literal value you provided on the command line

#### Example-1: Create ConfigMap using file
Following is a configuration file which we want to place in our nginx container. So I have created a separate file with the content to be added:
``````sh
[root@controller ~]# cat nginx-custom-config.conf
server {
   listen       8888;
   server_name  localhost;
   location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
   }
}

``````
#### Example-1: Create ConfigMap using file
cm is a short abbrevation for configmap

``````sh
 kubectl create cm nginx-cm --from-file nginx-custom-config.conf

``````
#### Inspect ConfigMap content

``````sh
kubectl get cm
kubectl get cm nginx-cm -oyaml
``````
#### Create Pod using the ConfigMap
the next step is to mount it onto a container
Create a YAML file named nginx-cm.yml to be used as our Pod configuration using the following content:
``````sh
[root@controller ~]# cat nginx-cm.yml
-------
apiVersion: v1
kind: Pod
metadata:
  name: nginx-cm
  labels:
    role: web
spec:
  containers:
  - name: nginx-cm
    image: nginx
    volumeMounts:
    - name: conf
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: conf
    configMap:
      name: nginx-cm
      items:
      - key: nginx-custom-config.conf
        path: default.conf

---
kubectl create -f nginx-cm.yml
``````
#### Verify ConfigMap data inside Pod’s container
verify if our content was added into /etc/nginx/conf.d/default.conf
``````sh

kubectl exec -it nginx-cm -- cat /etc/nginx/conf.d/default.conf

``````
#### Example-2: Create Kubernetes ConfigMap using command line arguments
Here color is the KEY while red is the VALUE of the KEY
``````sh

kubectl create cm myconfig --from-literal=color=red

kubectl get cm

kubectl get cm myconfig -oyaml
``````
#### Create Pod using ConfigMap using env variable
create a YAML file named test-cm-pod.yml to create a Pod into which we will inject fields from our ConfigMap as an environment variable.
``````sh
[root@controller ~]# cat test-cm-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test
    image: nginx
    env:
    - name: COLOR   ## Define the name of the variable
      valueFrom:
        configMapKeyRef:
          name: myconfig    ## Search for "myconfig" configmap which must be installed before creating this Pod
          key: color   ## The value of COLOR variable will be the value of "color" key in "myconfig" configmap i.e. "red"
  restartPolicy: Never

``````
#### Verify environment variable inside the container

``````sh
kubectl exec -it test-pod -- env | grep COLOR

``````
#### Example-3: Use Kubernetes ConfigMap to declare environment variables

``````sh
 ~]# cat configmap-1.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: declare-colors
data:
  # property-like keys; each key maps to a simple value
  color1: "red"
  color2: "blue"
  color3: "green"
-----
kubectl create -f configmap-1.yaml

kubectl get cm

``````
##### Now we will use these KEYs inside our POD definition file. Here is a sample YAML file to create nginx pod
``````sh
~]# cat nginx-1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: VAR1  ## Define the name of the variable
      valueFrom:
        configMapKeyRef:
          name: declare-colors   ## This must match the name of the configmap
          key: color1  ## The key to fetch
    - name: VAR2  ## Define the name of the variable
      valueFrom:
        configMapKeyRef:
          name: declare-colors   ## This must match the name of the configmap
          key: color2  ## The key to fetch
    - name: VAR3  ## Define the name of the variable
      valueFrom:
        configMapKeyRef:
          name: declare-colors   ## This must match the name of the configmap
          key: color3  ## The key to fetch
    imagePullPolicy: Always
Create this Pod:
------
kubectl create -f nginx-1.yaml

kubectl get pods

kubectl exec -it nginx -- printenv | grep VAR
``````

#### Example-4: Kubernetes mount ConfigMap as file

``````sh
~]# cat configmap-2.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: declare-vars
data:
  variables.conf: |
    color1="red"
    color2="blue"
    color3="green"
---------
kubectl create -f configmap-2.yaml
kubectl get cm
``````
##### Create a Pod which will use this configmap

``````sh
~]# cat nginx-vars.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-vars
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: variables
      mountPath: "/dir1"  ## This will be created if not present
  volumes:
    # You set volumes at the Pod level, then mount them into containers inside that Pod
    - name: variables  ## This must be same as name used under volumeMounts
      configMap:
        # Provide the name of the ConfigMap you want to mount.
        name: declare-vars
        # An array of keys from the ConfigMap to create as files
        items:
        - key: "variables.conf"  ## Key to fetch
          path: "default.conf"   ## Mount the file using this name.
---------
kubectl create -f nginx-vars.yaml
 kubectl exec -it nginx-vars -- bash
 cat /dir1/default.conf
``````