
#####  Execute Shell commands into a POD

execute bash or any shell commands using kubectl and exec any command into a container or pod.

##### Kubectl exec command syntax
 double dash (--), It is intentionally kept to separate the arguments you want to pass to the command from the kubectl arguments:

kubectl exec (POD | TYPE/NAME) [-c CONTAINER] [flags] – COMMAND [args...] [options]
Kubectl exec into pod - Executing commands inside POD

As we have already mentioned If it is a single container pod, you do not have to mention the container name with -c
If it is a multi-container pod. you need to mention which container, the command should be executed using -c

tomcatinfra-7f58bf9cb8-bk654 - Single Container POD
tomcat-nginx-78d457fd5d-446wx - Multi Container POD

``````sh
kubectl exec -it po/sidecar-pod -c app-container -- sh -c 'cat /var/log/app.log'
kubectl exec -it po/sidecar-pod -c log-exporter-sidecar -- sh -c 'ls -l /usr/share/nginx

``````
To get SSH or Terminal access to the container on the POD using kubectl exec.
You need to use the option -i  and -t

-i  represents that we want kubectl exec to run this interactive session
-t  represents that kubectl exec should get a terminal ID allotted
``````sh
kubectl exec tomcat-nginx-78d457fd5d-446wx -n test-ns -c tomcat8 -it – bash

``````
