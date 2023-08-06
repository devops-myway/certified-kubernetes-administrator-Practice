

##### Stages of Container Lifecycle:
A container is simply an isolated process running on your computer. A Docker container can be in one of several core states:

Created - The Docker container has been created, but not started (e.g. after using docker create)
Up - The Docker container is currently running. That is, the process inside the container is running. Can happen using docker start or docker run.
Paused - The Docker container has been paused, usually with the command docker pause.
Exited - The Docker container has exited, usually because the process inside the container has exited.

##### Why a Docker container terminates
The main process inside the container has ended successfully: This is the most common reason for a Docker container to stop! When the process running inside your container ends, the container will exit.

1 - You run a container, which runs a shell script to perform some tasks. When the shell script completes, the container will exit, because there’s nothing left for the container to run.
- You run a utility which is packaged as a Docker container, like the Busybox or alpine images. When the utility finishes, the container exits.

2- You’re running a shell in a container, but you haven’t assigned a terminal: If you’re running a container with a shell (like bash) as the default command, then the container will exit immediately if you haven’t attached an interactive terminal.
-  If there’s no terminal attached, then your shell process will exit, and so the container will exit. You can stop this by adding --interactive --tty (or just -it) to your docker run ... command, which will let you type commands into the shell.

##### To Keep a Pod Running
I have faced similar problems when I needed a POD just to run continuously without doing any useful operation. The following are the two ways those worked for me:
args: is an array

- Running sleep command while running the container.
- Running an infinite loop inside the container.

- 1. Sleep Command
- 2. Infinite Loop

kubectl apply -f <pod-yaml-file-name>.yaml

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  labels:
    app: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    ports:
    - containerPort: 80
    command: ["/bin/sh", "-c", "sleep 1000"]

------------------

kubectl run busy2 --image=busybox --port=8080 --dry-run=client -oyaml --command -- /bin/sh -c "while true; do echo '.'; sleep 3600; done" > busy2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: busybox
  labels:
    app: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    ports:
    - containerPort: 80
    command: ["/bin/sh", "-c", "while :; do echo '.'; sleep 5 ; done"]

``````
you can interact with Pod by running kubectl exec command.
second one passes "/bin/sh -c" , "sleep" & "3600" as arguments to Entrypoint of busybox image which is "/bin/sh" . So it will open a new shell to run "sleep 3600" inside the contain
command: ["/bin/bash"] and args: ["-c", "trap : TERM INT; sleep infinity & wait"]
command: ["/bin/sh", "-ec", "while :; do echo '.'; sleep 6 ; done"]

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