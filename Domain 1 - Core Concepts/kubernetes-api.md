https://kubernetes.io/docs/reference/kubernetes-api/


##### What Is Kubernetes API Used For
The Kubernetes API is foundational to the Kubernetes control plane. We can create, modify, and delete primitive elements within our Kubernetes clusters through the API.
The API also enables different components within our clusters to communicate, and it facilitates interactions with external components.
the Kubernetes API server (or kube-apiserver) resides in the control plane, along with etcd, kube-scheduler, kube-controller-manager, and cloud-controller-manager.

Interacting with the Kubernetes API:
Most Kubernetes users interact with their clusters with tools like kubectl. The tool accepts a command followed by a TYPE and includes a NAME and various flags (as needed).

Let’s explore the master node using the get action and type of node:
``````sh
kubectl api-versions

kubectl get nodes

kubectl get node kube-master
``````
###### We can interact directly with the Kubernetes API in two different ways

- The first approach is by using the kubectl proxy and
- the second is by generating an authentication token and passing that as a header with each of your requests.
Other resources for which we can execute commands include services, pods, and deployments.
For a complete list of all resources available within the cluster, you can run:
``````sh
kubectl api-resources

kubectl proxy --port=8080 &

``````
###### Understanding Kubernetes API Watch

The Kubernetes API also supports the concept of watches, which you can set up on an API call to monitor changes to the state of the specified resources over time.
We’ll create a deployment on our cluster and scale it up. We’ll use v1 of the nginx image for this deployment.
we’ll update the image used by that deployment to v1.21
``````sh
kubectl create deployment nginx --image=nginx:1

kubectl autoscale deployment nginx --min=5 --max=10

kubectl set image deployment nginx nginx=nginx:1.21

kubectl get pods

# You’ll observe pods in various states of running, terminating, and creation
# Let’s roll back that latest deployment and then set up a watch so we can watch it happen in real-time without having to keep running the same command

kubectl rollout history deployment nginx

kubectl rollout undo deployment nginx --to-revision=1

# Another benefit of using the API is that the watch serves as a continuous feed of information based on parameters you pass in.

curl http: //localhost:8080/api/v1/namespaces/default/pods?watch=1&app=nginx

``````