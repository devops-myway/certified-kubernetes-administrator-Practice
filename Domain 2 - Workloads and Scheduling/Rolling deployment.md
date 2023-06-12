https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

##### Rolling Strategy Deployment
The rolling deployment is the standard default deployment to Kubernetes. It works by slowly, one by one, replacing pods of the previous version of your application with pods of the new version without any cluster downtime.

A rolling update waits for new pods to become ready via your readiness probe before it starts scaling down the old ones
If there is a problem, the rolling update or deployment can be aborted without bringing the whole cluster down.

``````sh
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: awesomeapp
spec:
  replicas: 3
  template:
        metadata:
           labels:
             app: awesomeapp
        spec:
          containers:
            - name: awesomeapp
              image: imagerepo-user/awesomeapp:new
              ports:
                - containerPort: 8080

``````
Rolling updates can be further refined by adjusting the parameters in the manifest file:
``````sh
spec:
  replicas: 3
  strategy:
        type: RollingUpdate
        rollingUpdate:
           maxSurge: 25%
           maxUnavailable: 25%  
  template:
  ...

``````
##### Recreate
In this type of very simple deployment, all of the old pods are killed all at once and get replaced all at once with the new ones.

``````sh
spec:
  replicas: 3
  strategy:
        type: Recreate
  template:
  ...

``````
