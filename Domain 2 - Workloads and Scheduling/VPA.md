

##### Vertical Pod Autoscaler (VPA)
The Vertical Pod Autoscaler or VPA functionality provided in the Kubernetes platform aims to address the below problem.
To summarize:

If you over-allocate resources: you add unnecessary workers, waste unused resources, and increase your monthly bill.
If you under-allocate resources: resources will get used up quickly, application performance will suffer, and the kubelet may start killing pods until resource utilization drops.

##### How Does Horizontal Pod Autoscaler (HPA) Relate to VPA?
HPA scales the application pods horizontally by running more copies of the same pod (assuming that the hosted application supports horizontal scaling via replication). 
VPA is still relevant when horizontally scaling as it will compute and recommend the correct values for the requested resources of the pods managed by HPA.

##### How VPA works
The VPA controller observes the resource usage of an application. Then, using that usage information as a baseline, VPA recommends a lower bound, an upper bound, and target values for resource requests for those application pods.
Depending on how you configure VPA, it can either:

Apply the recommendations directly by updating/recreating the pods (updateMode = auto).
Store the recommended values for reference (updateMode = off).
Apply the recommended values to newly created pods only (updateMode = initial)

##### How to use VPA
Here is a sample Kubernetes Deployment that uses VPA for resource recommendations.
``````sh
apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.7.8
          ports:
          - containerPort: 80

``````
Then, create the VPA resources using the following manifest:
Note that the update mode is set to off. This will just get the recommendations, but not auto-apply them
``````sh
apiVersion: autoscaling.k8s.io/v1beta1
  kind: VerticalPodAutoscaler
  metadata:
    name: nginx-deployment-vpa
  spec:
    targetRef:
      apiVersion: "apps/v1"
      kind:       Deployment
      name:       nginx-deployment
    updatePolicy:
      updateMode: "Off"

``````
Once the configuration is applied, get the VPA recommendations by using the:
kubectl describe vpa nginx-deployment-vpa
``````sh
recommendation:
  containerRecommendations:
  - containerName: nginx
    lowerBound:
      cpu: 40m
      memory: 3100k
    target:
      cpu: 60m
      memory: 3500k
    upperBound:
      cpu: 831m
      memory: 8000k

``````