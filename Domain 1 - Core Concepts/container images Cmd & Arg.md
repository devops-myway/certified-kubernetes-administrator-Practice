https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

##### Stages of Container Lifecycle:
A container is simply an isolated process running on your computer. A Docker container can be in one of several core states:

- Created - The Docker container has been created, but not started (e.g. after using docker create)
- Up - The Docker container is currently running. That is, the process inside the container is running. Can happen using docker start or docker run.
- Paused - The Docker container has been paused, usually with the command docker pause.
- Exited - The Docker container has exited, usually because the process inside the container has exited.

##### Why a Docker container terminates
The main process inside the container has ended successfully: This is the most common reason for a Docker container to stop! When the process running inside your container ends, the container will exit.

##### To Keep a Pod Running
I have faced similar problems when I needed a POD just to run continuously without doing any useful operation. The following are the two ways those worked for me:
- args: is an array

- Running sleep command while running the container.
- Running an infinite loop inside the container.

- 1. Sleep Command
- 2. Infinite Loop

kubectl apply -f <pod-yaml-file-name>.yaml

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: ubuntu
    env:
    - name: Message
      value: "hello world"
    command: ["/bin/bash","-c"]
    args: ["while true; do echo $(Message); sleep 3600;done"]

------------------

apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
  restartPolicy: OnFailure

``````
kubectl logs command-demo

``````sh
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- sleep 3600
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml -- /bin/sh -c "sleep 3600"
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600;echo boo"
kubectl run ubuntu --image=ubuntu --restart=Never --command sleep infinity

------------------
kubectl exec ubuntu -it -- bash

----
apiVersion: v1
kind: Pod
metadata: 
  name: busybox-infinity
spec: 
  containers: 
    - 
      command: 
        - /bin/sh
        - "-c"
        - "ls /etc/config/"
        - "sleep infinity"
      image: busybox
      name: busybox-infinity

``````
you can use different variations of while loops, tailing and etc etc. That only rely on your imagination and needs.
That is because busybox runs the command and exits. You can solve it by updating your command in the containers section
Note that sh -c's argument is a single word. Part of the question is "which shell"; the natural answer would be a POSIX Bourne shell in the container's /bin/sh.


``````sh
command:
  - /bin/sh
  - -c
  - some_program 'a file; with punctuation'
------
["sh", "-c", "tail -f /dev/null"]

---
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
---
command: [ "/bin/bash", "-c", "--" ]
args: [ "while true; do sleep 30; done;" ]
---
 [ "/bin/sh", "-c", "ls /etc/config/", "sleep 3600"]

``````
