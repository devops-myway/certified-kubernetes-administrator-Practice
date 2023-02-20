##### Command

Kubernetes has the same concept and uses the same underlying image structure, but its names are different:

Kubernetes command: overrides Docker ENTRYPOINT (and resets CMD)
Kubernetes args: overrides Docker CMD

- Some images use CMD to specify the main command to run and don't have ENTRYPOINT; in this case any of the forms you show will work.
- Some images use ENTRYPOINT to specify the main command to run (and depending on how they're constructed may completely ignore CMD)
- Some images use CMD to specify the main container command, and ENTRYPOINT as a wrapper script to do first-time setup

- Arguments are entered into the terminal or console after a command.  When we give a command to the shell, the words separated by space after the actual command are known as the arguments.
- Argument: input given to executable to process that input.
<command> <argument> 


``````sh
#  “echo” command which is used to print the output that it receives from the shell.
echo <typed-text> 
echo welcome to example

----

CMD the-main-program
# override with either Kubernetes command: or args:
--
ENTRYPOINT ["the-main-command"]
CMD ["--argument", "value"]
# override with Kubernetes command: and optionally args:
--
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["the-main-command"]
# override with Kubernetes args: only
---
kind: Pod
spec:
  containers:
  - name: kiada
    image: luksa/kiada:0.4
    args: ["--listen-port", "9090"]

------ # Setting a literal value to an environment variable
kind: Pod
metadata:
  name: kiada
spec:
  containers:
  - name: kiada
    image: luksa/kiada:0.4
    env:
    - name: POD_NAME
      value: kiada
    - name: INITIAL_STATUS_MESSAGE
      value: This status message is set in the pod spec.


``````
When configuring command line arguments for containers we might need to be able to use certain values that might be elsewhere like the name of the current namespace.
- To do so we'll have to use the $() syntax, pleace notice how it's using parentheses instead of curly braces that you might be more familar
- Then we just need to define the environment variable with whatever value we need. In this example we are using valueFrom to get the object's namespace

``````sh
------ # Using variable references in environment variable values

env:
- name: POD_NAME
  value: kiada
- name: INITIAL_STATUS_MESSAGE
  value: My name is $(POD_NAME). I run NodeJS version $(NODE_VERSION).

----- # Using variable references in the command and arguments

spec:
  containers:
  - name: kiada
    image: luksa/kiada:0.4
    args:
    - --listen-port
    - $(LISTEN_PORT)
    env:
    - name: LISTEN_PORT
      value: "8080"

---

kind: Deployment
apiVersion: apps/v1
metadata:
  name: test-env
  labels:
    app: test-env
spec:
  selector:
    matchLabels:
      app: test-env
  replicas: 1
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: test-env
    spec:
      containers:
      - name: test-env
        image: alpine:latest
        imagePullPolicy: Always
        command:  ["echo"]
        args:
        - namespace=$(CURRENT_NAMESPACE)
        env:
        - name: CURRENT_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
--
kubectl apply -f test_env.yaml
kubectl get pods
kubectl logs test-env-9499cdd8c-7jwsd

``````