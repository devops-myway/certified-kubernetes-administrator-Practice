###### Note:
Kubernetes namespaces allow multiple clusters to share a single physical cluster.

They allow you to easily divide clusters into sub-clusters,that can still communicate with one another.
By remaining isolated within a namespace, a team’s infrastructure is better protected against outages and malicious threats.

- Namespaces are also useful for defining specialized sets of permissions—especially those pertaining to Role-Based Access Control (RBAC).
- Developers can use namespaces to separate test and production environments.

- A Kubernetes administrator may creatively use namespaces to form groups with shared permissions and roles

- You may also use namespaces for smart resource management. Admins can define quotas for resource usage and distribution.

- namespaces are useful for boosting performance across your clusters.\

Kube-system:
is the namespace for objects and service accounts with high-level privileges within Kubernetes.
Kubernetes itself owns this namespace, which specifically contains vital components like kube-dns, kube-proxy, kubernetes-dashboard, and more. don’t poke around heavily in kube-system.

kube-public:
Contains information deemed insensitive and therefore accessible to all inquiring users

##### How to Manually Generate a Namespace With Kubectl

``````sh
kubectl get namespaces

kubectl create namespace <namespace name>

kubectl apply -f namespace.yml

kubectl delete namespace

``````
##### Namespace Tips and Best Practices
``````sh
kubectl apply -f pod.yaml --namespace=examplenamespace

kubectl get pods --namespace=examplenamespace

``````
##### Namespace Communication
Namespaces are isolated from one another. This means one namespace isn’t discoverable to another.
there’s a high possibility that services within different namespaces need to communicate,
for example, if you have two namespaces that contain different users and RBAC policies.

Whatever the use case, using DNS service discovery can expose your endpoints as required.

This allows services to talk effectively by pointing apps toward service names. A service’s DNS pattern might look like this.

``````sh

ExampleService.ExampleNamespace.svc.cluster.local

#To access a service in another namespace

LoggingUtility.examplename
``````