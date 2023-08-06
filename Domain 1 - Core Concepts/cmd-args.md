#### Documentation Referenced in Video

https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

#### Create POD without any commands or arguments.
args in Kubernetes overrides CMD in the original docker image.
command in Kubernetes overrides ENTRYPOINT in the original docker image.


```sh
command: ["/bin/sh","-c"]
args: ["command one; command two && command three"]
```
The command ["/bin/sh", "-c"] or "-- sh -c": says "run a shell, and execute the following instructions"
The args are then passed as commands to the shell
In shell scripting a semicolon ; separates commands, and && conditionally runs the following command if the first succeed.

##### Referring to environment variables that aren’t in the manifest
you can only use the $(VAR_NAME) syntax in the command and args fields to reference variables that are defined in the pod manifest
If you are using the bash shell, you can do this by referring to the variable using the syntax $VAR_NAME or ${VAR_NAME} instead of $(VAR_NAME)
```sh
kubectl run alpine1 --image=alpine --port=8080 --dry-run=client -oyaml --command -- sh -c "echo "hostname is $HOSTNAME."; sleep infinity" > alpine1.yaml

containers:
- name: main
  image: alpine
  command:
  - sh
  - -c
  - 'echo "Hostname is $HOSTNAME."; sleep infinity'
```

#### Multi-Line Script
If you want to avoid concatenating all commands into a single command with ; or && you can also get true multi-line scripts using a heredoc.
```sh

command: 
 - sh
 - "-c"
 - |
   /bin/bash <<'EOF'

   echo "Hello world"
   ls -l
   exit 123

   EOF
```
```sh
readinessProbe:
  exec:
    command:
    - sh
    - -c
    - |
      command1
      command2 && command3

```

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
##### example Pod 2 with Multi-Line Commandss
```sh
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - |
      echo "running below scripts"
      i=0; 
      while true; 
      do 
        echo "$i: $(date)"; 
        i=$((i+1)); 
        sleep 1; 
      done
    name: busybox
    image: busybox
```
```sh
apiVersion: batch/v1
kind: Job
metadata:
  name: multiline
spec:
  template:
    spec:
      containers:
      - command:
        - /bin/bash
        - -exc
        - |
          set +x
          echo "running below scripts"
          if [[ -f "if-condition.sh" ]]; then
            echo "Running if success"
          else
            echo "Running if failed"
          fi
        name: ubuntu
        image: ubuntu
      restartPolicy: Never
  backoffLimit: 1
```
#### using secret example
```sh
apiVersion: v1
kind: Secret 
metadata:
  name: secret-script
type: Opaque
data:
  script_text: <<your script in b64>>

```
```sh
....
containers:
    - name: container-name
      image: image-name
      command: ["/bin/bash", "/your_script.sh"]
      volumeMounts:
        - name: vsecret-script
          mountPath: /your_script.sh
          subPath: script_text
....
  volumes:
    - name: vsecret-script
      secret:
        secretName: secret-script

```