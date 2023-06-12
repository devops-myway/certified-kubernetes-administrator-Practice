

#### Linux Distros example

Size of Busybox image ~ 2 MB
Size of Alpine image ~ 5 MB
Size of Ubuntu image ~ 188 MB

Busybox: is a minimal set of tools typically present in a unix-like operating system. It does not contain a kernel and is not an operating system.

Alpine Linux: is a minimal Linux distribution that contains everything necessary to boot a kernel and initiate a session.

Ubuntu: is a consumer-focused Linux distribution with many tools, utilities, and applications needed for an end-user focused operating system.

##### busybox in Linux - default shell of busybox is shell (sh) that provides easy access to all of these commands
For debugging live, this command frequently helps me:
In the busybox image is a basic shell that contains useful utilities.
- Utils I often use are nslookup and wget.
- nslookup: is useful for testing DNS resolution in a pod.
- wget: is useful for trying to do network requests.

ls -alF /bin

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
apk update
apk upgrade
apk add curl

# Use cURL to Download files
curl -o  file-url
curl -o https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-standard-3.14.2-x86_64.iso

# In case the downloading of the file has been interrupted or canceled, it could be continued or resumed with the help of option -C.
curl -C - -o  https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-standard-3.14.2-x86_64.iso

``````
