

##### kubectl explain

kubectl explain command is used to show documentation about Kubernetes resources like pod
The information displayed as output of this command is obtained from the OpenAPI Specification for the Pod [resource]

Kubectl explain pods.spec

#### Example to build securityContext for pod

kubectl explain pod.spec.securityContext | more
kubectl explain pod.spec.containers.securityContext | more