#### Note:
During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the kubectl run command can help in generating a YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all.

run: Create a new Pod to run a Container.
expose: Create a new Service object to load balance traffic across Pods.
autoscale: Create a new Autoscaler object to automatically horizontally scale a controller, such as a Deployment.

##### Create an NGINX Pod

kubectl run nginx --image=nginx

##### Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

kubectl run nginx --image=nginx --dry-run=client -o yaml

##### Create a deployment

kubectl create deployment --image=nginx nginx

##### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

##### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

##### Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.

kubectl create -f nginx-deployment.yaml

OR

##### In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

#### How to create objects - declaratively

kubectl apply -f <directory>/

##### Executing Commands Within an Existing Pod

With the kubectl exec command, you can execute commands inside existing pods
following command will execute ls within the running pod named mypod

The -- separates the parts of the command intended for Kubernetes itself from the command that should be passed to and executed within the container.

kubectl exec mypod -- /bin/ls

##### Connecting a Shell to an Existing Pod

A common use of kubectl exec is to open a new shell within a running pod

kubectl exec --stdin --tty mypod -- /bin/bash

or 

kubectl exec --it mypod -- /bin/bash

##### Define aliases for the most frequently used kubectl commands

alias k=kubectl
alias kgp="k get pod"
alias kgd="k get deploy"
alias kgs="k get svc"
alias kgn="k get nodes"
alias kd="k describe"
alias kge="k get events --sort-by='.metadata.creationTimestamp' |tail -8"

##### Create a temporary pod for testing
 kubectl -it  run busybox --rm --image=busybox -- sh

 # Start a busybox pod and keep it in the foreground, don't restart it if it exits.
  kubectl run -i -t busybox --image=busybox --restart=Never

##### VIM (Editor)

Enter edit mode or insert: I
Enter command mode: Esc
Save and leave vi : wq
Move the cursor to the last line: G
Move the cursor to the first line: gg
Moves the cursor to aspecified line: nG (n is the line number)