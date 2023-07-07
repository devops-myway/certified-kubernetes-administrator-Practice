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

#### Create a ConfigMap with manifest file

``````sh
kubectl create configmap testcm --from-literal=SPECIAL_LEVEL=very --from-literal=SPECIAL_TYPE=charm --dry-run=client -oyaml > cmtest.yaml
kubectl apply -f cmtest.yaml
kubectl describe configmap/testcm
--
apiVersion: v1
kind: ConfigMap
metadata:
   name: special-config
   namespace: default
data:
   SPECIAL_LEVEL: very
   SPECIAL_TYPE: charm

``````

#### Use a ConfigMap to set command line parameters
You can use ConfigMaps to define the commands or parameter values for a container by using the environment variable replacement syntax $(VAR_NAME)

``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: cm-demo-pod
spec:
  containers:
    - name: cm-demo-pod
      image: busybox
      command: ["/bin/sh", "-c", "while true; do echo $(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY); sleep 3600 ; done"]
      env:
        - name: SPECIAL_LEVEL_KEY  
          valueFrom:
            configMapKeyRef:
              name: testcm           
              key: SPECIAL_LEVEL
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: testcm
              key: SPECIAL_TYPE
EOF
--

kubectl exec -it cm-demo-pod -- sh -c 'env | grep _KEY'
SPECIAL_TYPE_KEY=charm
SPECIAL_LEVEL_KEY=very
``````
#### Use a ConfigMap in a volume

``````sh
vi cm1.yaml
kubectl apply -f cm1.yaml
kubectl describe configmap/game-demo

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"
  
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
EOF
---
vi pod1.yaml
kubectl apply -f pod1.yaml

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
          valueFrom:
            configMapKeyRef:
              name: game-demo           # The ConfigMap this value comes from.
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
  - name: config
    configMap:
      name: game-demo
      items:
      - key: "user-interface.properties"
        path: "user-interface.properties"
EOF

---- # After you run the pod, the following output is returned:
kubectl exec -it configmap-demo-pod -- sh -c 'env | grep _LIVES'
kubectl exec -it configmap-demo-pod -- sh -c 'env | grep _NAME'
kubectl exec -it configmap-demo-pod -- sh -c 'cat /config/user-interface.properties'
color.good=purple
color.bad=yellow
allow.textmode=true 

``````
###### Example cm and secret combine in deployment
``````sh

apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  mysql.conf: |
    [mysqld]
    port=3306
    socket=/tmp/mysql.sock
    key_buffer_size=16M
    max_allowed_packet=128M
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  secret.file: |
    c29tZXN1cGVyc2VjcmV0IGZpbGUgY29udGVudHMgbm9ib2R5IHNob3VsZCBzZWU=
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-db
  labels:
    app: my-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
      - name: my-db
        image: busybox:1.28
        command: ['sh', '-c', "while true; do cat /mysql/db-config && cat /mysql/db-secret; sleep 3600; done"]
        volumeMounts:
        - name: db-config
          mountPath: /mysql/db-config
        - name: db-secret
          mountPath: /mysql/db-secret
          readOnly: true
      volumes:
        - name: db-config
          configMap:
            name: mysql-config
        - name: db-secret
          secret:
            secretName: mysql-secret
--

kubectl apply -f pod2.yaml
kubectl get cm
kubectl get secret
kubectl get deploy
kubectl get po
kubectl exec -it my-db-66695bc6bd-hqqd4 -- sh -c "ls -l /mysql/db-config; ls -l /mysql/db-secret"
kubectl exec -it my-db-66695bc6bd-hqqd4 -- sh -c "ls -l /mysql/db-config/mysql.conf; cat /mysql/db-secret/secret.file"
``````