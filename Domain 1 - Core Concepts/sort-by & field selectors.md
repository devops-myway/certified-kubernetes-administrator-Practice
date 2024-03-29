https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/

##### Field Selectors

Field selectors let you select Kubernetes objects based on the value of one or more resource fields. Here are some examples of field selector queries:


This kubectl command selects all Pods for which the value of the status.phase field is Running
```sh
kubectl run nginx --image=nginx --port=8181
kubectl run nginx2 --image=nginx --port=8282
kubectl get po -owide

kubectl label pod nginx app=dev
kubectl label pod nginx2 app=myapp
kubectl get po -owide --show-labels

kubectl get pods --field-selector metadata.namespace=default
kubectl get pods --field-selector metadata.label=app.myapp #this will throw an error as it doesnt support it

```
So, one can conclude that field-selector only works for metadata.name and for additional fields for some types, but it is a very select set.

###### Sort-by
used to filter by creationTimestamp and metadata objects

On OS/X like on many systems (BSDs, Solaris, AIX, IRIX...), the functionality of GNU tac is available in tail with the -r option. So no need to install GNU tac:

tail -r the-file
```sh
kubectl get pods -A 
kubectl get pods -A --sort-by={.metadata.creationTimestamp} # in descending order
kubectl get pods -A --sort-by={.metadata.creationTimestamp} | tac #Ascending order

```
##### Example2
List pods in all namespaces of the cluster and sort by ascending order.
move the file to directory /opt/pods_asc.sh and print to screen.
```sh
sudo ls -l /opt/
sudo cat << EOF > /opt/pods_asc.sh
> #! /bin/bash
> kubectl get pods -A --sort-by={.metadata.creationTimestamp} | tac
> EOF
sudo chmod +x /opt/pods_asc.sh
bash /opt/pods_asc.sh

```
Cat and TAC for OSX
I like to use tac to reverse the output of cat.
for mac you can tail instead of tac to reverse the output of cat.
##### Install Homebrew
brew install coreutils
To use tac utility