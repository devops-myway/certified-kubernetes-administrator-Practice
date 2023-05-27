
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
https://www.densify.com/kubernetes-autoscaling/kubernetes-hpa/


###### Horizontal Pod Autoscaler (HPA)
In Kubernetes, a HorizontalPodAutoscaler automatically updates a workload resource (such as a Deployment or StatefulSet), with the aim of automatically scaling the workload to match demand.

##### Scaling Out (Horizontal Scaling)
With horizontal scaling, the compute resource limitations from physical hardware are no longer the issue.
you can use any reasonable size of server as long as the server has enough resources to run the pods.

##### Scaling Up (Vertical Scaling)
Scaling up (or vertical scaling) is adding more resources—like CPU, memory, and disk—to increase more compute power and storage capacity.

#### Scaling Stateless Applications
Stateless applications do not store data in the application, so they can be used as short-term workers. Kubernetes manages stateless applications very well.
In Kubernetes, a HorizontalPodAutoscaler automatically updates a workload resource, such as a Deployment, with the aim of automatically scaling the workload to match demand
This means if the load of application pods increases, the HorizontalPodAutoscaler keeps increasing the number of pods until the load comes back to normal range.
The horizontal pod autoscaling controller, running within the Kubernetes control plane, periodically adjusts the desired scale of its target Deployment to match observed metrics such as average CPU utilization, average memory utilization, or any other custom metric you specify.

##### Scaling Stateful Applications
Unlike stateless applications, stateful applications such as databases need to store data in persistent volumes.
This makes it more difficult for Kubernetes to manage stateful applications. 

##### How to Use HPA
How HPA can be configured to auto-scale application pods based on target CPU utilization. There are two ways to create an HPA resource:
- The kubectl autoscale command
- The HPA YAML resource file

``````sh
kubectl create ns hpa-test
--------- # Create a deployment for HPA testing
cat example-app.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
   name: php-apache
   namespace: hpa-test
  spec:
   selector:
     matchLabels:
       run: php-apache
   replicas: 1
   template:
     metadata:
       labels:
         run: php-apache
     spec:
       containers:
       - name: php-apache
         image: k8s.gcr.io/hpa-example
         ports:
         - containerPort: 80
         resources:
           limits:
             cpu: 500m   #  55m/1000 is 5.5% of a CPU core being used. 500/1000 is 100% of 50% of cpu being used.
           requests:
             cpu: 200m  # 200m/1000m x 100% 20% cpu being used.
  ---
  apiVersion: v1
  kind: Service
  metadata:
   name: php-apache
   namespace: hpa-test
   labels:
     run: php-apache
  spec:
   ports:
   - port: 80
   selector:
     run: php-apache
---
kubectl create -f example-app.yaml
kubectl get deploy -n hpa-test

``````
After the deployment is up and running, create the HPA using kubectl autoscale command
This HPA will maintain minimum 1 and max 5 replica pods of the deployment to keep the overall CPU usage to 50%.
``````sh
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=5 -n hpa-test

``````
The declarative form of the same command would be to create the following Kubernetes resource
``````sh
apiVersion: autoscaling/v1
  kind: HorizontalPodAutoscaler
  metadata:
   name: php-apache
   namespace: hpa-test
  spec:
   scaleTargetRef:
     apiVersion: apps/v1
     kind: Deployment
     name: php-apache
   minReplicas: 1
   maxReplicas: 10
   targetCPUUtilizationPercentage: 50

``````
Examine the current state of HPA
``````sh
kubectl -n hpa-test get hpa
``````
Currently there is no load on the running application so the current and desired pods are equal to the initial number which is 1
``````sh
kubectl get hpa php-apache -o yaml -n hpa-test
``````
###### Increase the load
Stop the load by pressing CTRL-C and then get the hpa status again , this will show things get back to normal and one replica running
``````sh
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

kubectl -n hpa-test get hpa
``````
Clean up the resources
``````sh
kubectl delete ns hpa-test --cascade
``````
