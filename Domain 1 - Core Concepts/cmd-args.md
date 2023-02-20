#### Documentation Referenced in Video

https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

#### Create POD without any commands or arguments.
args in Kubernetes overrides CMD in the original docker image.
command in Kubernetes overrides ENTRYPOINT in the original docker image.

##### newpod.yaml

```sh
apiVersion: v1
kind: Pod
metadata:
  name: command
spec:
  containers:
  -  image: busybox
     name: count
```
```sh
kubectl apply -f commands.yaml
```
```sh
kubectl get pods
kubectl exec -it command -- bash
```

#### Create POD with Command

Modify the POD contents to the following one.

```sh
apiVersion: v1
kind: Pod
metadata:
  name: command2
spec:
  containers:
  -  image: busybox
     name: count
     command: ["sleep","3600"]
```
```sh
kubectl apply -f commands.yaml
```
```sh
kubectl get pods
kubectl exec -it command2 -- sh
```

#### Create POD with Command and Arguments

Modify the YAML file contents to the following one.

```sh
apiVersion: v1
kind: Pod
metadata:
  name: command3
spec:
  containers:
  -  image: busybox
     name: count
     command: ["sleep"]
     args: ["3600"]
```
```sh
kubectl apply -f commands.yaml
```
```sh
kubectl get pods
```

#### Create POD with Arguments

Modify the YAML file contents to the following one.

```sh
apiVersion: v1
kind: Pod
metadata:
  name: command4
spec:
  containers:
  -  image: busybox
     name: count
     args: ["sleep","3600"]
```
```sh
kubectl apply -f commands.yaml
```
```sh
kubectl get pods
kubectl exec -it command4 -- sh
```