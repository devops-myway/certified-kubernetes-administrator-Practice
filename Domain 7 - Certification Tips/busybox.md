

##### busybox pod
For debugging live, this command frequently helps me:
In the busybox image is a basic shell that contains useful utilities.
- Utils I often use are nslookup and wget.
- nslookup: is useful for testing DNS resolution in a pod.
- wget: is useful for trying to do network requests.

``````sh
kubectl create deployment bb --image busybox --restart=Never -it --rm -- /bin/sh

kubectl run -it --rm debug --image=busybox --restart=Never -- /bin/sh

kubectl run myshell --image=busybox --command -- sh -c "sleep 3600"

``````
##### Modified Busybox
Because busybox does not have package manager like: yum, apk, or apt-get ..
Either use a modified busybox: https://github.com/progrium/busybox


``````sh
kubectl exec -it busybox -- opkg-install curl

``````
##### minimal image, you can use alpine
``````sh
kubectl exec -it alpine -- apk --update add curl

``````