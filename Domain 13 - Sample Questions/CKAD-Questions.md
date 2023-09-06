
###### Questions - Network Policy
- Task 1-
Update the Pod ckad00018-newpod in the ckad00018 namespace to use a NetworkPolicy allowing the Pod to send and receive traffic only to and from the pods web and db.
Note:
All required network policies have already been created. You must not create, modify or delete any network policy while working on this task.
you may only use the existing network policies.

``````sh

kubectl create ns ckad00018
kubectl run ckad00018-newpod --image=nginx -n ckad00018
kubectl run web --image=nginx -n ckad00018
kubectl run db --image=redis -n ckad00018

kubectl get po --show-labels -n ckad00018
kubectl expose po/ckad00018-newpod --port=80 --name=ckad00018-newpod-svc --type=ClusterIP -n ckad00018
kubectl get svc -n ckad00018

#Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: ckad00018
spec:
  podSelector:
    matchLabels:
      run: ckad00018-newpod
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              run: web
        - podSelector:
            matchLabels:
              run: web
      ports:
        - protocol: TCP
          port: 80
  egress:
    - to:
        - podSelector:
            matchLabels:
              run: db
        - podSelector:
            matchLabels:
              run: web
      ports:
        - protocol: TCP
          port: 80
--
kubectl apply -f nt1.yaml
kubectl describe netpol/test-network-policy -n ckad00018
## Test the netpol
kubectl run testpod --image=busybox -n ckad00018 --labels=run=web -- sh -c "sleep 3600"
kubectl run busybox --image=busybox -n ckad00018 --labels=run=db -- sh -c "sleep 3600"
kubectl exec -it busybox -n ckad00018 -- sh
# Run the svc object to test connection
wget -O- ckad00018-newpod-svc:80

``````
##### Question - Network Policy
Task 2-
You have rolled out a new pod to your infrastructure and now you need to allow it to communicate with the web and storage pods but nothing else. Given the running pod kdsn00201-newpod edit it to use a network policy that will allow it to send and receive traffic only to and from the web and storage pods.
``````sh
kubectl run web --image=nginx
kubectl run storage --image=redis
kubectl get po --show-labels

kubectl get po/kdsn00201-newpod --show-labels

vi net1.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: kdsn00201-newpod
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              run: web
        - podSelector:
            matchLabels:
              run: storage
  egress:
    - to:
        - podSelector:
            matchLabels:
              run: web
        - podSelector:
            matchLabels:
              run: storage
---
kubectl apply -f nt1.yaml
kubectl describe netpol/test-network-policy
## Test the netpol
kubectl expose kdsn00201-newpod --port=80 --name=svc2 --type=ClusterIP
kubectl run testpod --image=busybox --labels=run-web -- sh -c "sleep 3600"
kubectl exec -it testpod -- sh
wget -O- svc2:80

``````
###### Network Policy
- Task -
Create a YAML file with the name our-net-pol that runs two pods and a networkpolicy.
The first pod should run a nginx image with default settings
The second pod should run a busybox image with sleep 3600 command
Control the traffic between these two pods, such that access to the nginx server is only allowed from busybox pod and busybox pod is free to access or to be accessed from any where.
``````sh
kubectl run nginx --image=nginx --labels=app=nginx
kubectl run busybox --image=busybox --labels=access=allowed -- sh -c "sleep 3600"
kubectl get po --show-labels
kubectl get ns --show-labels

vi our-net-pol.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: allowed

kubectl apply -f our-net-pol.yaml
### Test connectivity
kubectl expose po/nginx --port=80 --name=svc2 --type=clusterIP
kubectl get svc -owide
kubectl run test --image=busybox --labels=access=allowed -- sh -c "sleep 3600"
kubectl exec -it test -- sh
wget -O- svc2:80
``````

###### Questions- network Policy
Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it.
``````sh
kubectl create nginx --image=nginx --replicas=2
kubectl expose deploy/nginx --port=80 --name=nginx-svc --type=ClusterIP

kubectl get deploy --show-labels
kubectl get svc --show-labels

### Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: granted
      ports:
        - protocol: TCP
          port: 6379

kubectl apply -f nt1.yaml
## Test the netpol
kubectl run testpod -k-image=busybox --labels=access=granted -- sh -c "sleep 3600"
kubectl exec -it testpod -- sh
wget -O- nginx-svc:80
``````

###### Questions - ConfigMaps

You are tasked to create a ConfigMap and consume the ConfigMap in a pod using a volume mount.
Task
Please complete the following:
* Create a ConfigMap named another-config containing the key/value pair: key4/value3
* start a pod named nginx-configmap containing a single container using the nginx image, and mount the key you just created into the pod under directory /also/a/path

``````sh
kubectl create cm another-config --from-literal=key4=value3

vi pod1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap 
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: vol
      mountPath: "/also/a/path"
      readOnly: true
  volumes:
  - name: vol
    configMap:
      name: another-config

kubectl apply -f pod1.yaml
``````
##### Questions - Secrets

Context You are tasked to create a secret and consume the secret in a pod using environment variables as follow: 
- Task 1.
- Create a secret named another-secret with a key/value pair; key1/value4
- Start an nginx pod named nginx-secret using container image nginx, and add an environment variable exposing the
value of the secret key key1, using COOL_VARIABLE as the name for the environment variable inside the pod

``````sh
# Task 1
kubectl create secret generic another-secret --from-literal=key1=value4

vi pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secret
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: COOL_VARIABLE
      valueFrom:
        secretKeyRef:
          name: another-secret
          key: key1

kubectl apply -f pod1.yaml


``````
- Q2 - Task 2-
1- Create a secret named app-secret in the default namespace containing the following single keyvalue pair: Key3: value1
2- Create a Pod named ngnix-secret in the default namespace. Specify a single container using the nginx:stable image.
Add an environment variable named BEST_VARIABLE consuming the value of the secret key3.
``````sh
--- # Task 2
kubectl create secret generic app-secret --from-literal=Key3=value1 -n default

vi pod2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: ngnix-secret
  namespace: default
spec:
  containers:
  - name: nginx:stable
    image: nginx:stable
    env:
    - name: BEST_VARIABLE 
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: key3

``````
##### Questions - Volumes
Context
A project that you are working on has a requirement for persistent data to be available.
Task
To facilitate this, perform the following tasks:
- Task 1.
Create a file on node sk8s-node-0 at /opt/KDSP00101/data/index.html with the content Acct=Finance
- Task 2.
Create a PersistentVolume named task-pv-volume using hostPath and allocate 1Gi to it, specifying that the volume is at
/opt/KDSP00101/data on the cluster's node. The configuration should specify the access mode of ReadWriteOnce. It should define the StorageClass name exam for the PersistentVolume, which will be used to bind PersistentVolumeClaim requests to this PersistenetVolume
- Task 1.
Create a PersissentVolumeClaim named task-pv-claim that requests a volume of at least 100Mi and specifies an access
mode of ReadWriteOnce
- Task 2.
Create a pod that uses the PersistentVolmeClaim as a volume with a label app: my-storage-app mounting the resulting
volume to a mountPath /usr/share/nginx/html inside the pod
``````sh
ssh sk8s-node-0
mkdir -p /opt/KDSP00101/data/
touch /opt/KDSP00101/data/index.html
echo "Acct=Finance" > /opt/KDSP00101/data/index.html

vi pv1.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  storageClassName: slow
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: /opt/KDSP00101/data
    type: Directory

kubectl apply -f pv1.yaml

vi pvc1.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 100Mi
  storageClassName: slow

kubectl apply -f pvc1.yaml

vi pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: my-storage-app
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: task-pv-claim

kubectl apply -f pod1.yaml
``````
##### Questions - Canary Deployment
Context
You are asked to prepare a Canary deployment for testing a new application release.
Task:
A Service named krill-Service in the goshark namespace points to 5 pod created by the Deployment named current-krill-deployment
1- Create an identical Deployment named canary-kill-deployment, in the same namespace.
2- Modify the Deployment so that:
- A maximum number of 10 pods run in the goshark namespace.
- 40% of the krill-service ‘s traffic goes to the canary-krill-deployment pod(s)
The service to exposed on Nodeport 30000 to test its load-balancing run.
curl http://k8s-master-0:30000/
- Side Note: 
Use only one label as selector for the service object e.g app: canary

``````sh
kubectl create ns goshark
kubectl create deploy current-krill-deployment --replicas=5 --image=nginx -n goshark --dry-run=client -oyaml> dep11.yaml
--
vi dep11.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: current-krill-deployment
    upgrade: canary
  name: current-krill-deployment
  namespace: goshark
spec:
  replicas: 5
  selector:
    matchLabels:
      app: current-krill-deployment
      upgrade: canary
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: current-krill-deployment
        upgrade: canary
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

kubectl get deploy -owide -n goshark
--
kubectl expose deploy/current-krill-deployment --port=80 --name=krill-service --type=ClusterIP -n goshark
 kubectl get svc -n goshark -owide

## Canary Deploy
vi dep11.yaml
kubectl apply -f dep11.yaml  # manually scale deployment to 6 pods = 60% of the pods in the namespace

kubectl create deploy canary-kill-deployment --replicas=4 --image=nginx -n goshark   #40%
vi dep22.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: canary-kill-deployment
    upgrade: canary
  name: canary-kill-deployment
  namespace: goshark
spec:
  replicas: 4
  selector:
    matchLabels:
      app: canary-kill-deployment
      upgrade: canary
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: canary-kill-deployment
        upgrade: canary
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
--
kubectl get deploy -n goshark

##### Modify the service object
kubectl expose deploy canary-kill-deployment --port=80 --name=krill-service --type=ClusterIP --dry-run=client -oyaml -n goshark>svc33.yaml
vi svc33.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    upgrade: canary
  name: krill-service
  namespace: goshark
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30000
  selector:
    upgrade: canary
  type: NodePort
status:
  loadBalancer: {}
~                    
kubectl apply -f svc33.yaml
kubectl describe svc/krill-service -n goshark
#### Sln
Endpoints:         10.244.35.156:80,10.244.35.157:80,10.244.35.158:80 + 7 more...

curl http://k8s-master-0:30000/
curl nopeip:nodeport
##### Test using busybox
kubectl get nodes -woide
kubectl run test --image=nginx:alpine -n goshark
kubectl exec -it test -n goshark -- sh
curl http://k8s-master-0:30000/ or curl node_ip:node_port
``````

##### Questions - DockerFile
A Dockerfile is simply a text-based file with no file extension that contains a script of instructions. Docker uses this script to build a container image.
docker build  -t ImageName:TagName dir   # build image from dockerfile
To name a container, we just need to pass the --name flag to the run command.
docker run -d -p 8000:8000 --name rest-server node-docker  # run a image as container
When exporting, you save the filesystem of the container in a .tar archive 
docker export container-id > arithmetic.tar

https://adamtheautomator.com/docker-save-image/

- Task:
A Dockerfile has been prepared at  ~/human-stork/build/Dockerfile
1- Using the prepared Dockerfile, build a container image with the name macque and tag 3.0. You may install and use the tool of your choice.
2- Using the tool of your choice export the built container image in OC-format and store it at ~/human stork/macque 3.0.tar.

``````sh
ssh cluster
sudo -i
cd ~/human-stork/build/
# Dockerfile-----Example
FROM busybox
CMD echo $(((20*5)/10))
# ------
ls -l
docker build --tag macque:3.0 .
docker images
docker run -d macque:3.0
docker ps # Running container
docker container ls --all
docker export container-id > ~/human-stork/macque-3.0.tar
cat ~/human-stork/macque-3.0.tar

#### Importing Docker Containers
docker import arithmetic.tar put_any_name_here:latest
docker images
docker run -it put_any_name_here:latest sh
echo $(((20*5)/10))
### Saving Images via Docker Save Image
docker save arithmetic > arm_image.tar 
``````

###### Questions - DockerFile2

Create a Docker file and save the file in directory /opt/ldh/Question12 . The Docker file run an alpine image with the command "echo hello linuxdatahub" as the default command. Build the image, and export it in OCI format to a file file with the name "linuxdocker" and tag should be 9.8. Use sudo wherever required
Note:
Docker images are by default OCI complaint
Save one or more images to a tar archive (streamed to STDOUT by default): docker save --output busybox.tar busybox

``````sh
vi Dockerfile
FROM alpine
CMD ["echo hello linuxdatahub"]

docker build --tag linuxdocker:9.8 .
docker images
docker run -d linuxdocker:9.8
docker container -a
docker export container_id> linuxdocker.tar

``````
##### Questions - ResourceQoutas
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/

Create ResourceQuota in namespace one with hard requests cpu=1, memory=1Gi and hard limits cpu=2, memory=2Gi.
Attempt to create a pod with resource requests cpu=2, memory=3Gi and limits cpu=3, memory=4Gi in namespace one. # will throw error
Create a pod with resource requests cpu=0.5, memory=1Gi and limits cpu=1, memory=2Gi in namespace one
``````sh
kubectl create ns one

vi pp1.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
  namespace: one
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi

kubectl apply -f pp1.yaml
kubectl get resourcequota mem-cpu-demo -n one -oyaml

vi pod22.yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
  namespace: one
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "4Gi"
        cpu: "3"
      requests:
        memory: "3Gi"
        cpu: "2"

kubectl apply -f pod22.yaml
# Error Throws out
rror from server (Forbidden): error when creating "pod22.yaml": pods "quota-mem-cpu-demo" is forbidden: exceeded quota: mem-cpu-demo, requested: limits.cpu=3,limits.memory=4Gi,requests.cpu=2,requests.memory=3Gi, used: limits.cpu=0,limits.memory=0,requests.cpu=0,requests.memory=0, limited: limits.cpu=2,limits.memory=2Gi,requests.cpu=1,requests.memory=1Gi

----
vi pod33.yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
  namespace: one
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "2Gi"
        cpu: "1"
      requests:
        memory: "1Gi"
        cpu: "0.5"

kubectl apply -f pod33.yaml
kubectl get po -n one
kubectl get resourcequota/mem-cpu-demo -n one -oyaml
``````

##### Questions - Deployments & Resource Managment - Must Do
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

- Task 1- 
The pod for the Deployment named nosql in the crayfish namespace fails to start because its container runs out of resources.
Update the nosql Deployment so that the Pod:
1- Request 128M of memory for its Container
2- Limits the memory to half the maximum memory constraint set for the crayfish namespace.
Note:
the nosql deployments manifest file can be found at
~/chief-cardinal/nosql.yaml

``````sh
kubectl create ns crayfish
kubectl get ns

vi lr1.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: set-limit-range
  namespace: crayfish
spec:
  limits:
  - max:
      memory: "800Mi"
    min:
      memory: "200Mi"
    type: Container

kubectl apply -f 1.yaml
kubectl get limitrange/set-limit-range -n crayfish
kubectl describe limitrange set-limit-range -n crayfish -oyaml

# In the above screenshot, it can be seen that we have created a limit range that defines memory as "Min=200Mi" and "Max=800Mi".
# This means that pods in the crayfish namespace cannot have memory "less than Min=200Mi" and "more than Max=800Mi".

vi lr1-edit.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: set-limit-range-new
  namespace: crayfish
spec:
  limits:
  - max:
      memory: "256Mi"
    min:
      memory: "128Mi"
    type: Container

---
vi ~/chief-cardinal/nosql.yaml
vi dp44.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: crayfish
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: nosql
      app.kubernetes.io/name: backend
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nosql
        app.kubernetes.io/name: backend
    spec:
      containers:
      - name: mongo
        image: mongo:4.2
        args:
        - --bind_ip
        - 0.0.0.0
        ports:
        - containerPort: 27017
        resources:
          requests:
            memory: "128Mi"

kubectl apply -f dp44.yaml
 kubectl get deployment.apps/nginx-deployment -n crayfish
kubectl get limitrange/mem-limit-range -n crayfish -oyaml
``````

- Task 2-
Create a Pod named nginx-resources in the existing pod-resources namespace.
Specify a single container using nginx:stable image.
Specify a resource request of 300m cpus and 1G1 of memory for the Pod's container.

``````sh
vi pod22.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-resources
  namespace: pod-resources
spec:
  containers:
  - name: nginx
    image: nginx:stable
    resources:
      requests:
        memory: "1Gi"
        cpu: "3000m"

kubectl apply -f pod22.yaml
``````

##### Questions - Deployments - RollingUpdate strategy
https://kubernetes.io/blog/2022/05/27/maxunavailable-for-statefulset/
- Task 1-
As a Kubernetes application developer you will often find yourself needing to update a running application.
Task
Please complete the following:
* Update the app deployment in the kdpd00202 namespace with a maxSurge of 5% and a maxUnavailable of 2%
* Perform a rolling update of the web1 deployment, changing the Ifccncf/nginx image version to 1.13
* Roll back the app deployment to the previous version

- Task 2-
Task:
1- Update the Propertunel scaling configuration of the Deployment web1 in the ckad00015 namespace setting maxSurge to 2 and maxUnavailable to 5%
2- Update the web1 Deployment to use version tag 1.13.7 for the Ifconf/nginx container image.
3- Perform a rollback of the web1 Deployment to its previous version

``````sh
kubectl create ns kdpd00202
kubectl create deploy nginx --image=nginx --replicas=1 --dry-run=client -oyaml -n kdpd00202> dep31.yaml
vi dep31.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web1
  namespace: kdpd00202
spec:
  replicas: 3
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 5%
      maxUnavailable: 2%
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
        image: Ifccncf/nginx:1.13.7
        ports:
        - containerPort: 80

$ka -f dep2.yaml
kubectl rollout status deploy/web1 -n kdpd00202
kubectl rollout history deploy/web1 -n kdpd00202
kubectl rollout undo deploy/web1 --to-revision=1 -n kdpd00202

``````
##### Questions - Deployments - Configuration

- Task 1-
A deployment is falling on the cluster due to an incorrect image being specified. Locate the deployment, and fix the problem.

``````sh
kubectl get deploy --all-namespaces
kubectl edit deploy/deploy_name
#manually update the image: Update deployment image to nginx:1.17.4: 
$ksi deploy/hello-deploy nginx=nginx:1.17.4
$kg deploy

``````
##### Questions:  CronJobs - Jobs
https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

Context
Developers occasionally need to submit pods that run periodically.
- Task
Follow the steps below to create a pod that will start at a predetermined time and which runs to completion only once each time it is started:
- Create a YAML formatted Kubernetes manifest ~/opt/KDPD00301/periodic.yaml that runs the following shell command: date in a single busybox container. The command should run every minute and must complete within 22 seconds or be terminated by Kubernetes.
The Cronjob name and container name should both be hello• Create the resource in the above manifest and verify that the job executes successfully at least once.
- Task -
Create a job from cronjob and name it sample-job.

``````sh
vi /opt/KDPD00301/periodic.yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: hello
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: hello
    spec:
      activeDeadlineSeconds: 22 # strict time constraint
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: hello
            resources: {}
          restartPolicy: Never
  schedule: '*/1 * * * *'
status: {}
--
kubectl apply -f ~/opt/KDPD00301/periodic.yaml
kubectl get cj
kubectl get po
kubectl logs hello-28217792-v8q5w
Sat Aug 26 16:32:01 UTC 2023
Hello from the Kubernetes cluster
kubectl delete cj/

--

kubectl create job --from=cronjob/sample-cron-job sample-job
``````

- Task 2-
Create a cronjob with the given name. Image, Schedule was given in the question.
Keep 3 success and 2 Failed history configuration
Terminate the pod after 8 seconds.
Run a job from the cronjob we created.

``````sh
-- # Task 2
vi cron1.yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: hello
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: hello
    spec:
      activeDeadlineSeconds: 8   # strict deadline constraint with time
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: hello
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 2
status: {}
--
kubectl apply -f cj2.yaml
kubectl get cj
kubectl get po  #after the duration set at 8 seconds
kubectl logs hello-28217801-z7kzc
``````
###### Questions - Cronjob

Create a Cronjob name show-date that runs every minute and executes the shell command
echo "current date: $(date)".
- Watch the jobs as they are being scheduled
- Identify one of the pods that ran the cronjob and render the logs
- determine the number of successful executions the cronjob will keep in its history 
Delete the Job.

``````sh
kubectl create cronjob show-date --image=busybox --schedule="*/1 * * * *" -- sh -c 'echo "current date: $(date)"'
or
kubectl create cronjob show-date --image=busybox --dry-run=client -oyaml --schedule="*/1 * * * *" -- sh -c 'while true;do echo "current date: $(date)";sleep 30;done'

apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: show-date
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: show-date
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - sh
            - -c
            - 'while true;do echo "current date: $(date)";sleep 30;done'
            image: busybox
            name: show-date
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'

kubectl get cj -w
kubectl get po
kubectl logs container-name
kubectl delete cj/name_cj
``````

###### Service Accounts

- Task 1-
Your application's namespace requires a specific service account to be used.
Task
Update the app-a deployment in the production namespace to run as the restrictedservice service account. The service account has already been created.

- Task 2-
Update the Deployment app-1 in the frontend namespace to use the existing ServiceAccount app.


``````sh
kubectl create ns test1
kubectl create sa restrictedservice -n test1

kubectl create deploy app-a --image=nginx --replicas=1 --dry-run=client -oyaml -n test1>dd1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-a
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      serviceAccountName: restrictedservice
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl apply -f dep2.yaml
kubectl get deployment.apps/app-a -n test1
 kubectl describe deployment.apps/app-a -n test1
``````

###### Question - Deployment - Exposed Service 
Task:
Create a Deployment named expose in the existing ckad00014 namespace running 6 replicas of a Pod.
Specify a single container using the ifccncf/nginx: 1.13.7 image Add an environment variable named NGINX_PORT with the value 8001 to the container then expose port 8001

``````sh
kubectl create ns ckad00014
kubectl create deployment expose --image=ifccncf/nginx:1.13.7 --replicas=6 -n ckad00014 --dry-run=client -oyaml>dp1.yaml

vi dp1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: expose
  labels:
    app: expose
spec:
  replicas: 6
  selector:
    matchLabels:
      app: expose
  template:
    metadata:
      labels:
        app: expose
    spec:
      containers:
      - name: nginx
        image: ifccncf/nginx:1.13.7
        ports:
        - containerPort: 8001
        env:
        - name: NGINX_PORT
          value: "8001"

kubectl get deployment.apps/expose -n ckad00014

``````
###### Questions - Security Context
Task:
Modify the existing Deployment named broker-deployment running in namespace quetzal so that its containers.
1- Run with user ID 30000 and
2- Privilege escalation is forbidden
The broker-deployment is manifest file can be found at:
``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broker-deployment
  namespace: quetzal
  labels:
    app: test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: test
        image: busybox
        ports:
        - containerPort: 80
        command: [ "sh", "-c", "sleep 3600" ]
      securityContext:
        runAsUser: 3000
        allowPrivilegeEscalation: false

kubectl apply -f l1.yaml
kubectl get deployment.apps/test-deployment -n quetzal
kubectl get po -n quetzal
kubectl exec -it test-deployment-bd5cf86cd-cchl8 -n quetzal -- sh
ps aux
id
``````

##### Questions - Logs
- Task 1-
You sometimes need to observe a pod's logs, and write those logs to a file for further analysis.
Task
Please complete the following;
* Deploy the counter pod to the cluster using the provided YAMLspec file at /opt/KDOB00201/counter.yaml
* Retrieve all currently available application logs from the running pod and store them in the file /opt/KDOB0020l/log_Output.txt, which has already been created

- Task 2-
Task:
A pod within the Deployment named buffale-deployment and in namespace gorilla is logging errors.
1-  Look at the logs identify errors messages.
Find errors, including User “system:serviceaccount:gorilla:default” cannot list resource “deployment” […] in the namespace “gorilla” 
2- Update the Deployment buffalo-deployment to resolve the errors in the logs of the Pod.
The buffalo-deployment manifest can be found at - /prompt/escargot/buffalo-deployment.yaml
``````sh
cat /opt/KDOB00201/counter.yaml
kubectl apply -f /opt/KDOB00201/counter.yaml
kubectl get po
kubectl logs counter
kubectl logs couter > /opt/KDOB0020l/log_Output.txt
cat /opt/KDOB0020l/log_Output.txt

-- # Task 2

kubectl create ns gorilla
kubectl create sa default -n gorilla
kubectl auth can-i list deployments --as=system:serviceaccount:gorilla:default -n gorilla
kubectl create role list-deploy --namespace=gorilla --verb=list --resource=deployments
kubectl create rolebinding list-deploy-rbinding --role=list-deploy --serviceaccount=gorilla:default -n gorilla
--
vi /prompt/escargot/buffalo-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test-deploy
  namespace: gorilla
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      serviceAccount: default
      containers:
      - image: nginx
        name: test
        resources: {}
status: 

kubectl apply -f /prompt/escargot/buffalo-deployment.yaml
kubectl auth can-i list deployments --as=system:serviceaccount:gorilla:default -n gorilla
kubectl logs test-deploy-7cff6cdb89-ch2kd -n gorilla
``````

###### Questions - Service and Deployment

- Task 1-

Task:
1- First update the Deployment cka00017-deployment in the ckad00017 namespace:
*To run 2 replicas of the pod
*Add the following label on the pod: Role userUI
2- Next, Create a NodePort Service named cherry in the ckad00017 nmespace exposing the ckad00017-deployment Deployment on TCP port 8888

``````sh
# Task 1
kubectl get ns 
kubectl get deploy -n ckad00017
kubectl edit cka00017-deployment -n ckad00017

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cka00017-deployment
  namespace: ckad00017
  labels:
    Role: userUI
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl get deploy cka00017-deployment -n ckad00017
kubectl expose deploy cka00017-deployment --port=8888 --name=cherry --type=NodePort -n ckad00017
kubectl get svc cherry -n ckad00017

``````
- Task 3- 
Task
Create a new deployment for running nginx with the following parameters;
* Run the deployment in the kdpd00201 namespace. The namespace has already been created
* Name the deployment frontend and configure with 4 replicas
* Configure the pod with a container image of lfccncf/nginx:1.13.7
* Set an environment variable of NGINX__PORT=8080 and also expose that port for the container above
``````sh
vi dep1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: kdpd00201
  labels:
    app: nginx
spec:
  replicas: 4
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
        image: lfccncf/nginx:1.13.7
        env:
        - name: NGINX__PORT
          value: "8080"

kubectl apply -f dep1.yaml
kubectl get deploy frontend
kubectl logs pod/frontend_pods

``````
- Task 3- 
Context
A container within the poller pod is hard-coded to connect the nginxsvc service on port 90 . As this port changes to 5050 an additional container needs to be added to the poller pod which adapts the container to connect to this new port. This should be realized as an ambassador container within the pod.
Task
* Update the nginxsvc service to serve on port 5050.
* Add an HAproxy container named haproxy bound to port 90 to the poller pod and deploy the enhanced pod. Use the image haproxy and inject the configuration located at /opt/KDMC00101/haproxy.cfg, with a ConfigMap named haproxy-config, mounted into the container so that haproxy.cfg is available at /usr/local/etc/haproxy/haproxy.cfg.
Ensure that you update the args of the poller container to connect to localhost instead of nginxsvc so that the connection is correctly proxied to the new service endpoint. You must not modify the port of the endpoint in poller's args . The spec file used to create the initial poller pod is available in /opt/KDMC00101/poller.yaml
``````sh
vi /opt/KDMC00101/haproxy.cfg

apiVersion: v1
kind: ConfigMap
metadata:
  name: haproxy-config
data:
  haproxy.cfg: |-
  "global"
  "daemon"

vi pod-service.yaml

apiVersion: v1
kind: Pod
metadata:
  name: poller
  labels:
    app.kubernetes.io/name: proxy
spec:
  volumes:
  - name: config-vol
    configMap:
      name: haproxy-config
  containers:
  - name: haproxy
    image: haproxy
    ports:
      - containerPort: 90
    args: ["/bin/sh", "-c", "while true; do curl 127.0.0.1:5050; sleep 3600; done"]
    volumeMounts:
    - name: config-vol
      mountPath: "/usr/local/etc/haproxy/haproxy.cfg"
      readOnly: true
  - name: ambassador-container
    image: nginx
    ports:
      - containerPort: 5050
    - 
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 5050
    targetPort: 90

---
kubectl logs po/poller -c ambassador-container
``````
##### Question - Multiple Containers - sidecar 
https://kubernetes.io/docs/concepts/cluster-administration/logging/

Given a container that writes a log file in format A and a container that converts log files from format A to format B, create a deployment that runs both containers such that the log files from the first container are converted by the second container, emitting logs in format B.
Task:
* Create a deployment named deployment-xyz in the default namespace, that:
* Includes a primary lfccncf/busybox:1 container, named logger-dev
* includes a sidecar Ifccncf/fluentd:v0.12 container, named adapter-zen
* Mounts a shared volume /tmp/log on both containers, which does not persist when the pod is deleted
* Instructs the logger-dev
container to run the command: while true; do echo "i lov cncf" >> /tmp/log/input.log;sleep 10; done
which should output logs to /tmp/log/input.log in plain text format, with example values:
i luv cncf, i luv cncf,i luv cncf.
* The adapter-zen sidecar container should read /tmp/log/input.log and output the data to /tmp/log/output.log in Fluentd JSON format.
Note:
 that no knowledge of Fluentd is required to complete this task: all you will need to achieve this is to create the ConfigMap from the spec file provided at /opt/KDMC00102/fluentd-configmap.yaml , and mount that ConfigMap to /fluentd/etc in the adapter-zen sidecar container
``````sh
vi /opt/KDMC00102/fluentd-configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluentd.conf: |
    source>
      type tail
      format none
      path /tmp/log/output.log
--
kubectl create deploy deployment-xyz --image=lfccncf/busybox:1 --replicas=1 --dry-run=client -oyaml> dep2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-xyz
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: vol
        emptyDir: {}
      - name: config-vol
        configMap:
          name: fluentd-config
      containers:
      - name: logger-dev
        image: lfccncf/busybox:1
        args:
        - /bin/sh
        - -c
        - "while true; do echo "i lov cncf" >> /tmp/log/input.log;sleep 10; done"
        volumeMounts:
        - name: vol
          mountPath: /tmp/log
      - name: adapter-zen
        image: Ifccncf/fluentd:v0.12
        args: [/bin/sh, -c, 'cat /tmp/log/input.log > /tmp/log/output.log' ]
        volumeMounts:
        - name: vol
          mountPath: /tmp/log
        - name: config-vol
          mountPath: /etc/fluentd-config

``````

##### Questions: Readiness Probe & Liveness Probe
Task
A Deployment named backend-deployment in namespace staging runs a web application on port 8081.
The deployment manifest can be found ~/spicy-pikachu/backend-deployment.yaml
mnodify the deployment specifying a readiness probe using path /healthz
set initialDelaySeconds to 8 and periodSeconds to 5

``````sh
vi ~/spicy-pikachu/backend-deployment.yaml
kubectl get deploy -n staging

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: staging
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 8081
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
        initialDelaySeconds: 8
        periodSeconds: 5
kubectl apply -f ~/spicy-pikachu/backend-deployment.yaml

``````

- Task 2- 
A user has reported an aopticauon is unreachable due to a failing livenessProbe .
Task
Perform the following tasks:
• Find the broken pod and store its name and namespace to /opt/KDOB00401/broken.txt in the format:
The output file has already been created
• Store the associated error events to a file /opt/KDOB00401/error.txt, The output file has already been created. You will need to use the -o wide output specifier with your command
• Fix the issue.
Note:
the associated deployment could be running in any of the following namespaces: qa, test, production, alan

``````sh
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3


kubectl get po -owide
liveness-http   0/1     CrashLoopBackOff    5 (50s ago)   3m11s   # The logs should be same 
kubectl get po liveness-http -n default > /opt/KDOB00401/broken.txt 
kubectl events liveness-http -n default > /opt/KDOB00401/error.txt

``````
##### API Troubleshooting Issue
- Task 1:
1) Fix any API depreciation issues in the manifest file ~/credible-mite/www.yaml so that this application can be deployed on cluster K8s.
2) Deploy the application specified in the updated manifest file ~/credible-mite/www.yaml in namespace cobra

``````sh
vi /credible-mite/www.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: cobra
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: vol
        emptyDir: {}
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
        env:
        - name: NGINX_ENTRYPOINT_LOGS
          value: "1"

kubectl apply -f ~/credible-mite/www.yaml
kubectl get po -n cobra

``````

##### Questions- Resource Limits & Requests

you are required to create a pod that requests a certain amount of CPU and memory, so it gets scheduled to a node that has those resources available.
• Create a pod named nginx-resources in the pod-resources namespace that requests a minimum of 200m CPU and 1Gi memory for its container
• The pod should use the nginx image
• The pod-resources namespace has already been created

``````sh
kubctl get ns

vi pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-resources
  namespace: pod-resources
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: "200m"

kubectl appky -f pod2.yaml
kubectl get po -n pod-resources

``````
###### Questions - Sort-by
It is always useful to look at the resources your applications are consuming in a cluster.
Task
• From the pods running in namespace cpu-stress , write the name only of the pod that is consuming the most CPU to file /opt/KDOBG030l/pod.txt, which has already been created.

``````sh
kubectl top pods
kubectl top pod POD_NAME --sort-by=cpu 
kubectl top pod nginx-ingress-ingress-nginx-controller-7c6d45df75-s5kln --sort-by=cpu > /opt/KDOBG030l/pod.txt
``````
###### Questions - Pod Creation
Context
Anytime a team needs to run a container on Kubernetes they will need to define a pod within which to run the container.
Task
Please complete the following:
• Create a YAML formatted pod manifest
/opt/KDPD00101/podl.yml to create a pod named app1 that runs a container named app1cont using image Ifccncf/arg-output with these command line arguments: -lines 56 -F
• Create the pod with the kubectl command using the YAML file created in the previous step
• When the pod is running display summary data about the pod in JSON format using the kubectl command and redirect the output to a file named /opt/KDPD00101/out1.json
• All of the files you need to work with have been created, empty, for your convenience
Note:
when creating pod, you do not need to specify command, only args
``````sh
vi /opt/KDPD00101/podl.yml

apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
  - name: app1cont
    image: Ifccncf/arg-output
    args:
    - /bin/sh
    - -c
    -  -lines 56 -F

kubectl apply -f /opt/KDPD00101/podl.yml
kubectl get po/app1 -ojson >  /opt/KDPD00101/out1.json
cat /opt/KDPD00101/out1.json
``````

###### Questions - Service and Deployments

You have been tasked with scaling an existing deployment for availability, and creating a service to expose the deployment within your infrastructure.
- Task -
Start with the deployment named kdsn00101-deployment which has already been deployed to the namespace kdsn00101 . 
Edit it to:
• Add the func=webFrontEnd key/value label to the pod template metadata to identify the pod for the service definition
• Have 4 replicas
Next, create and deploy in namespace kdsn00l01 a service that accomplishes the following:
• Exposes the service on TCP port 8080
• is mapped to the pods defined by the specification of kdsn00l01-deployment
• Is of type NodePort
• Has a name of cherry

``````sh
### Assume this is the original deployment in the namespace.
kubectl get deploy -n kdsn00101

apiVersion: apps/v1
kind: Deployment
metadata:
  name: kdsn00101-deployment
  namespace: kdsn00101
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80

#### Edit the above original deployment - for example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kdsn00101-deployment
  namespace: kdsn00101
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        func: webFrontEnd
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl expose deploy kdsn00101-deployment --port=8080 --name=cherry --type=NodePort -n kdsn00101
kubectl describe svc/cherry -n kdsn00101
``````

###### Questions - InitContainers

Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

``````sh
vi in1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  volumes:
  - name: vol
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: vol
      mountPath: /usr/share/nginx/html
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c',"wget -O /work-dir/index.html http://neverssl.com/online"]
    volumeMounts:
    - name: vol
      mountPath: /work-dir

kubectl apply -f in1.yaml
kubectl get po -owide
kubectl run test --image=busybox -- sh -c 'sleep 3600'
kubectl exec -it test -- sh
wget -O- ip_of_po
``````
###### Question - Ingress and Deployment - Must 
Verify that the Ingress Controller is running.
- Task- 
Create a new Deployment named web that controls a single replica running the image bmuschko/nodejs-hello-world:1.0.0 on port 3000.
Expose the Deployment with a Service named web of type NodePort. The Service routes traffic to the Pods controlled by the Deployment web.
Make a request to the endpoint of the application on the context path /. You should see the message "Hello World".
Create an Ingress that exposes the path / for the host hello-world.exposed. The traffic should be routed to the Service created earlier.
List the Ingress object.
Add an entry in /etc/hosts that maps the virtual node IP address to the host hello-world.exposed.
Make a request to http://hello-world.exposed. You should see the message "Hello World".

``````sh
kubectl get svc

kubectl create deploy web --image=bmuschko/nodejs-hello-world:1.0.0 --replicas=1 --port=3000
kubectl expose deploy web --port=3000 --name=web --type=NodePort
kubectl get deploy,svc
kubectl get nodes -owide
### Test the svc
kubectl run testbox --image=busybox -- sh -c 'sleep 3600'
kubectl exec -it testbox -- sh
wget -O- node_ip:node_port
wget -O- 192.168.49.2:32510
#### create Ingress Resource
vi in3.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - host: hello-world.exposed
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 3000
kubectl apply -f in3.yaml
kubectl describe ing/web-ingress
echo 'hello-world.exposed 192.168.49.2' | sudo tee /etc/hosts or 
sudo vi /etc/hosts #manually input the ip to the hosts hello-world.exposed 192.168.49.2
sudo cat /etc/hosts
curl hello-world.exposed

``````

``````sh


``````