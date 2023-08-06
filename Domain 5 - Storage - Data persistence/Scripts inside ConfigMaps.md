https://kubernetes.io/docs/concepts/configuration/configmap/

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

https://stackoverflow.com/questions/73365727/mounting-a-configmap-as-a-volume-in-kubernetes-how-do-i-calculate-the-value-of


##### Using scripts inside configmaps

Injecting a script to a pod to debug issues is a common need of Kubernetes administrators
ConfigMaps are Kubernetes objects that allow you to store configuration data separately from the application code.
This separation of concerns makes it easier to manage and update configuration data without modifying the application code.

#### Executing a simple Task with Bash
configmap we are creating a bash script called “slim-shady.sh” that echos the lyrics “My Name Is” by Eminem
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: slim-shady-configmap
data:
  slim-shady.sh: |
    #!/bin/bash
    echo "Hi!"
    echo "My name is"
    echo "What?"
    echo "My name is"
    echo "Who?"
    echo "My name is"
    echo "Chika-chika"
    echo "Slim Shady"
EOF
``````
Inject the script into a pod with ubuntu image.
``````sh
vi testpod.sh

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: testpod
  name: testpod
spec:
  containers:
  - args:
    - sh
    - -c
    - sleep 3600
    image: ubuntu
    name: testpod
    volumeMounts:
    - name: script
      mountPath: /opt
  volumes:
  - name: script
    configMap:
      name: slim-shady-configmap
      defaultMode: 0500
      items:
      - key: slim-shady.sh
        path: slim-shady.sh
----
kubectl apply -f testpod.yaml
kubectl describe pod/testpod
kubectl exec -it pod/testpod -- sh
# use bash to run the script
bash /opt/slim-shady.sh
``````
##### Create a ConfigMap
The key is "my-script.sh" and the value is a Bash script that prints "Hello, World!" to the console.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-script-configmap
data:
  my-script.sh: |
    #! /bin/bash
    echo "Hello, World!"
    echo "Wahala no de finish"
EOF

``````
##### Define a Volume Mount
``````sh
vi testpod2.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: testpod2
  name: testpod2
spec:
  containers:
  - args:
    - sh
    - -c
    - sleep 3600
    image: ubuntu
    name: testpod2
    volumeMounts:
    - name: script2
      mountPath: /opt
  volumes:
  - name: script2
    configMap:
      name: my-script-configmap
      defaultMode: 0500
      items:
      - key: my-script.sh
        path: my-script.sh

---
kubectl describe po/testpod2
kubectl exec -it testpod2 -- sh
ls -l /opt
cat /opt/my-script.sh 
``````

##### stored as an env variable and call that variable to run the script.
After running the command, you should see the message "Hello, World!" in the console output.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: alpine
    env:
    - name: MY_SCRIPT
      valueFrom:
        configMapKeyRef:
          name: my-script-configmap
          key: my-script.sh
    command: ["/bin/sh"]
    args: ["-c", "$(MY_SCRIPT)"]
EOF
--- # Test the injection 
kubectl logs -f my-pod -c my-container

``````