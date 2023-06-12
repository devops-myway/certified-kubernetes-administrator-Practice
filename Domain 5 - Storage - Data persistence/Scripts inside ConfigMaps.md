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

``````
To run this script by using the configmap, let’s take a look at an example Kubernetes Job below and walk through it.
Two things I would like to point out here:
- Inside the volume, we must specify a defaultMode of atleast 0500 (read+execute to the user)
- This defines the file permissions inside the container when the pod is run
- If we do not set this, we get permission denied errors
- This is because as a default the configmap will be mounted with 0644 and you will end up with an error like below
  Warning  Failed     7s    kubelet            Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/script/slim-shady.sh": permission denied: unknown 

``````sh
apiVersion: batch/v1
kind: Job
metadata:
  name: chicka-chicka-slim-shady
spec:
  template:
    spec:
      containers:
        - name: shady
          image: centos
          command: ["/script/slim-shady.sh"]
          volumeMounts:
            - name: script
              mountPath: "/script"
      volumes:
        - name: script
          configMap:
            name: slim-shady-configmap
            defaultMode: 0500
      restartPolicy: Never
----
kubectl logs -f chicka-chicka-slim-shady-h7j5m
``````
##### Create a ConfigMap
The key is "my-script.sh" and the value is a Bash script that prints "Hello, World!" to the console.
``````sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-script-configmap
data:
  my-script.sh: |
    #!/bin/bash
    echo "Hello, World!"
--
kubectl apply -f my-script-configmap.yaml
``````
##### Define a Volume Mount
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
    volumeMounts:
    - name: my-script-volume
      mountPath: /script
    command: ["/script/my-script.sh"]
  volumes:
  - name: my-script-volume
    configMap:
      name: my-script-configmap
      defaultMode: 0744
EOF

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