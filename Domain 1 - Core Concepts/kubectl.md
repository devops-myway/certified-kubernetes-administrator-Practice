
##### kubectl run
kubectl run was updated to only create pods with Kubernetes 1.18 and it lost its deployment-specific options as well
kubectl run  <name> : command helps you to run container images on your Kubernetes pods.

``````sh
kubectl run nginx --image=nginx
``````
##### Using --Port and --dry-run
--port: You can control the port that the newly created instance will expose
--dry-run:  The dry-run option allows you to test the effects of a kubectl run call without actually running in on your live cluster - preview the manifest file

``````sh
kubectl run nginx --image-nginx --port=5701

kubectl run nginx --image=nginx --dry-run=client -oyaml

``````
##### --command vs --arguments
to run commands on your newly created pods.
- You can either append them to your kubectl run call like this:
- Or, you could pass them as commands
If you are looking to run a command, always prefer the --command option as it will override the EntryPoint and Cmd defined in your container and run your command directly

``````sh
kubectl run nginx --image=nginx -- 'echo hello;sleep 3600'

kubectl run nginx --image=nginx --command -- 'echo hello;sleep 3600'

``````
