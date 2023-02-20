#### How to run curl in Kubernetes (for troubleshooting)

If you’re trying to diagnose an issue with your app in Kubernetes, you’ll probably want to look at curl. It’s a handy tool for testing APIs and network requests.

#### Run curl in another Pod
A ‘clean’ way to run curl inside Kubernetes is to run it in a separate Pod.
You can create a Pod running curl, do the troubleshooting work you need to do, and then terminate the Pod.

#### What about Busybox
Busybox is one of the other well-known “box of tools” type images. It doesn’t contain curl, but it does contain wget!

#### Create a new Pod that runs curl or Either use a modified busybox
To do that, I use the kubectl run command, which creates a single Pod
Kubernetes will now pull the curlimages/curl image, start the Pod, and drop you into a terminal session.

``````sh
image: curlimages/curl

kubectl run mycurlpod --image=curlimages/curl -it -- sh

kubectl run busybox --image=busybox --restart=Never --sleep 3600
or 
image: progrium/busybox
kubectl exec -it busybox -- opkg-install curl

or
image: alpine:3.12
kubectl exec -it alpine -- apk --update add curl

kubectl delete pod mycurlpod

``````

