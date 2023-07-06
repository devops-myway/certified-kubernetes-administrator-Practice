https://kubernetes.io/docs/concepts/policy/resource-quotas/


##### Resource Quotas
ResourceQuota is an object in Kubernetes that enables administrators to restrict cluster tenants’ resource usage per namespace.
A Kubernetes cluster has a limited amount of available hardware resources.

###### How Resource Quota Limits Work
we go through a demo example of how to create and define the CPU resource quota on a namespace with requests and limits.

``````sh
kubectl get nodes
kubectl create namespace quota-demo

``````
define and create the CPU quota on a namespace:
``````sh
cat cpu-quota.yaml
apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: test-cpu-quota
    namespace: quota-demo
  spec:
    hard:
      requests.cpu: "200m"  
      limits.cpu: "300m"
------
kubectl apply -f cpu-quota.yaml
kubectl describe resourcequota test-cpu-quota --namespace quota-demo
``````
you can create a test pod with requests and limits defined as shown below:

``````sh
cat <<EOF | kubectl apply -n -f-
  apiVersion: v1
  kind: Pod
  metadata:
    name: testpod1
  spec:
    containers:
    - name: quota-test
      image: busybox
      imagePullPolicy: IfNotPresent
      command: ["/bin/sh"]
      args: ["-c", "while true; do echo pod is Running; sleep 10;done"]
      resources:
        requests:
          cpu: "100m"
        limits:
          cpu: "200m"
    restartPolicy: Never
  EOF

``````
``````sh
kubectl describe resourcequota test-cpu-quota --namespace quota-demo

``````
Create another pod to test the ResourceRoutas
``````sh
kubectl create -n quota-demo -f- <<EOF
  apiVersion: v1
  kind: Pod
  metadata:
    name: testpod2
  spec:
    containers:
    - name: quota-test
      image: busybox
      imagePullPolicy: IfNotPresent
      command: ['sh', '-c', 'echo Pod is Running ; sleep 5000']
      resources:
        requests:
          cpu: "10m"
        limits:
          cpu: "20m"
    restartPolicy: Never
  EOF

``````
The new pod is now listed as having consumed more of the available quota limits for the total allocated CPU resources.
If we try to create a pod requesting more resources than what’s available in quota, we will now receive an error message that states we don’t have enough quota left to create the new pod:
``````sh
kubectl describe resourcequota test-cpu-quota --namespace quota-demo

``````
