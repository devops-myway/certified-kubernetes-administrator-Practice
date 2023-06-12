
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands


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
##### Interacting With Pods and Namespaces
-A – List pods, services, daemonsets, deployments, replicasets, statefulsets, jobs, and CronJobs in all namespaces, not custom resource types.
Note the alias for --all-namespaces is -A
--namespace or -n 
-c argument if you have only one container running inside the pod
``````sh

``````
##### Cluster Management and Context - Kubectl config
``````sh
kubectl cluster-info                                                            #Display endpoint information about the master and services in the cluster.

kubectl version                                                                 #Display the Kubernetes version running on the client and server.

kubectl config view                                                            #Get the configuration of the cluster.

kubectl config use-context [context-name]
kubectl config get-contexts
kubectl config set-context [context-name] --cluster=[cluster-name] --user=[user-name]
kubectl config set-cluster [cluster-name] --server=[server-url]
kubectl config set-credentials [user-name] --username=[username] --password=[password]
kubectl config use-context [context-name] --namespace=[namespace]


``````

##### Events

``````sh
kubectl get events                                                                                      # ist recent events for all resources in the system.

kubectl get events --field-selector type=Warning                                                        # List Warnings only.

kubectl get events --sort-by=.metadata.creationTimestamp                                                # List events sorted by timestamp.

kubectl get events --field-selector involvedObject.kind!=Pod                                            # List events but exclude Pod events.

kubectl get events --field-selector involvedObject.kind=Node, involvedObject.name=<node_name>           #Pull events for a single node with a specific name.

kubectl get events --field-selector type!=Normal                                                        # Filter out normal events from a list of events 

``````

##### --sort-by
Pods have status, which you can use to find out startTime
``````sh
# List Services Sorted by Name
kubectl get services --sort-by=.metadata.name

# List pods Sorted by Restart Count
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# List PersistentVolumes sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage

# List Events sorted by timestamp
kubectl get events --sort-by=.metadata.creationTimestamp

``````


