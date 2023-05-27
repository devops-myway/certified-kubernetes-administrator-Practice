

##### Container running on Kubernetes?
Containers are meant to run to completion. You need to provide your container with a task that will never finish.
A container exits when its main process exits. Doing something like. 
you should understand the basic concept here. Your docker container will be running till its main process is live. And it will be completed as soon as your main process will stop.

``````sh
docker run -itd debian
docker run -d debian sleep 300

------------------
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
spec:
  containers:
  - name: ubuntu
    image: ubuntu:latest
    # Just spin & wait forever
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "while true; do sleep 30; done" ]  #  semicolon. In programming standards, the ; signifies an end of statement,

``````

##### To Keep a Pod Running
In order to keep a POD running it should to be performing certain task, otherwise Kubernetes will find it unnecessary, therefore it stops. There are many ways to keep a POD running.
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
    command: ["/bin/sh", "-ec", "sleep 1000"]

------------------
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
    command: ["/bin/sh", "-ec", "while :; do echo '.'; sleep 5 ; done"]

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