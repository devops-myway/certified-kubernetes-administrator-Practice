https://kubernetes.io/docs/concepts/configuration/configmap/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/


#### Overview on Kubernetes ConfigMaps
- ConfigMap is a Kubernetes API object that allows you to store non-sensitive information.
- store non-confidential data: working with Kubernetes is by actually storing these scripts as configmaps for automation.

#### How to create ConfigMap

kubectl create configmap [map-name] [data-source] [arguments]
map-name - refers to the name of the ConfigMap
data-source - can be the path to a file or directory containing the ConfigMap
arguments - specifies whether to use files or directories (--from-file), environmental files (--from-env-file), or literal values (--from-literal)

Key: is the filename or the key you provided on the command line
Value: is the file content or the literal value you provided on the command line

#### Example-1: Configmap using ENV
cm is a short abbrevation for configmap

``````sh
kubectl create configmap fromliterals --from-literal=pizza=cheese --from-literal=drink=coke
---
kubectl get cm
kubectl get cm fromliterals -oyaml
kubectl describe cm fromliterals
--
Name:         fromliterals
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
pizza:
----
cheese

drink:
----
coke
Events:  <none>
``````

#### Create Pod using the ConfigMap
the next step is to mount it onto a container
use the configmap fromliterals in POD manifest and accessing the key/value pair using keys and passing the values to environment variables.
FO.
FOOD env variable should get the value of pizza key and DRINK env variable should get the value of drink key
``````sh

------- # configmap/literal-pod.yaml
cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: literal-pod
   spec:
     containers:
       - name: literal-container
         image: k8s.gcr.io/busybox
         command: [ "/bin/sh", "-c", "env" ]
         env:
           # Define the environment variable
           - name: FOOD
             valueFrom:
               configMapKeyRef:
                 # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
                 name: fromliterals
                 # Specify the key associated with the value
                 key: pizza
           - name: DRINK
             valueFrom: 
               configMapKeyRef: 
                 name: fromliterals
                 key: drink     
     restartPolicy: Never
EOF

--- # Verify from the POD logs that environment variables are set to right values
kubectl logs literal-pod

``````
#### Create a ConfigMap with manifest file

``````sh
apiVersion: v1
kind: ConfigMap
metadata:
   name: special-config
   namespace: default
data:
   SPECIAL_LEVEL: very
   SPECIAL_TYPE: charm

``````
##### Use a ConfigMap to define environment variables for a pod

#### Use the a single key-value pairs of a ConfigMap to define environment variables for a pod using -  env parameter with valueFrom field
To define an environment variable for a pod, you can use the valueFrom field to reference a single value of SPECIAL_LEVEL.
``````sh
apiVersion: v1
kind: Pod
metadata:
   name: config-pod-1
spec:
   containers:
     - name: test-container
       image: busybox
       command: [ "/bin/sh", "-c", "env" ]
       env:
         - name: SPECIAL_LEVEL_KEY
           valueFrom:              ##Use valueFrom to denote that env references the value of a ConfigMap. 
             configMapKeyRef:
               name: special-config               ##The name of the referenced ConfigMap. 
               key: SPECIAL_LEVEL                 ##The key of the referenced key-value pair. 
   restartPolicy: Never
``````
#### Use all key-value pairs of a ConfigMap to define multiple environment variables for a pod - envFrom parameter
To use all key-value pairs of a ConfigMap to define multiple environment variables for a pod, you can use the envFrom parameter. The keys of the ConfigMap are used as the names of the environment variables.
``````sh
apiVersion: v1
kind: Pod
metadata:
   name: config-pod-2
spec:
   containers:
     - name: test-container
       image: busybox
       command: [ "/bin/sh", "-c", "env" ]
       envFrom:                ##All key-value pairs of the special-config ConfigMap are referenced. 
       - configMapRef:
           name: special-config
   restartPolicy: Never

``````
#### Use a ConfigMap to set command line parameters
You can use ConfigMaps to define the commands or parameter values for a container by using the environment variable replacement syntax $(VAR_NAME)

``````sh
apiVersion: v1
kind: Pod
metadata:
   name: config-pod-3
spec:
   containers:
     - name: test-container
       image: busybox
       command: [ "/bin/sh", "-c", "echo $(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]
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
   restartPolicy: Never

``````
#### Use a ConfigMap in a volume
To mount a ConfigMap as a volume, specify the name of the ConfigMap in the volumes section. This saves the key-value pairs of the ConfigMap to the directory specified in the mountPath field. In this example, the directory is /etc/config. Configuration files that are named after the keys of the ConfigMap are generated. The values of the ConfigMap are stored in the related files.

``````sh
apiVersion: v1
kind: Pod
metadata:
   name: config-pod-4
spec:
   containers:
     - name: test-container
       image: busybox
       command: [ "/bin/sh", "-c", "ls /etc/config/" ]   ##List the files in the directory. 
       volumeMounts:
       - name: config-volume
         mountPath: /etc/config
   volumes:
     - name: config-volume
       configMap:
         name: special-config
   restartPolicy: Never

---- # After you run the pod, the following output is returned:
SPECIAL_TYPE
SPECIAL_LEVEL

``````