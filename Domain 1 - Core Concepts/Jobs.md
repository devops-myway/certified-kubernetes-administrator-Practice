https://kubernetes.io/docs/concepts/workloads/controllers/job/

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

##### Kubernetes Jobs - Key Specifications
Jobs in Kubernetes has several specifications that can be customized to meet the specific requirements of your workload. These include completions, parallelism, backofflimit, and activeDeadlineSeconds.

Completions:
Completions is a job specification that specifies the desired number of successfully completed pods in a job sequentially (one after the other)
If the completions field is not specified, the default value is 1. The job is considered complete when the specified number of completions is reached, and any remaining pods are terminated.
 If a pod fails, Kubernetes will automatically create a new pod to replace it until the specified number of completions is reached.

Parallelism:                          
Parallelism specifies the maximum number of pods that can run in parallel to complete the job. This is useful for workloads that require multiple instances of a task to run concurrently.

Backofflimit
Backofflimit specifies the number of times Kubernetes should retry a failed container before giving up. This can be useful for workloads prone to failures or requiring several attempts to complete successfully.

ActiveDeadlineSeconds:
 specifies the maximum amount of time a job can run before it is terminated. This can be useful for workloads with strict time constraints or prevent a job from running indefinitely.

####  Ex 1. completions
``````sh
vi j1.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world
spec:
  completions: 2   #the desired number of completed pods that should exist when the job is considered complete.
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["echo",  "Hello World!"]
      restartPolicy: Never
-----
kubectl apply -f j1.yaml
kubectl get job
kubectl logs pod_name

``````
##### Ex 2 - parallelism
The parallelism field is set to 2, which means that at most 2 pods will be created to run the tasks of the job at any given time.
``````sh
vi j2.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world
spec:
  completions: 2
  parallelism: 2
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["echo",  "Hello World!"]
      restartPolicy: Never

``````
##### Ex 3. Backofflimit

The maximum number of retries the job controller should attempt before considering the Job as failed is 2.
``````sh
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world
spec:
  backoffLimit: 2
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["cat",  "job.yaml"]
      restartPolicy: Never

``````

##### Ex 4 - ActiveDeadlineSeconds - Job termination and cleanup
The activeDeadlineSeconds field is set to 15, which specifies that the job should be automatically terminated if it has been running for more than 15 seconds. Once a Job reaches activeDeadlineSeconds, all of its running Pods are terminated and the Job status will become type: Failed with reason: DeadlineExceeded.
Note that a Job's .spec.activeDeadlineSeconds takes precedence over its .spec.backoffLimit.
``````sh
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world
spec:
  activeDeadlineSeconds: 15
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sleep",  "45"]
      restartPolicy: Never

``````
###### Clean up finished jobs automatically - TTL Mechansm
Another way to clean up finished Jobs (either Complete or Failed) automatically is to use a TTL mechanism provided by a TTL controller for finished resources, by specifying the .spec.ttlSecondsAfterFinished field of the Job. delete its dependent objects, such as Pods, together with the Job.

``````sh
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never

``````
##### Questions: 1
Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute
``````sh
kubectl create job my-job --image=busybox --dry-run=client -oyaml -- "while true; do echo "hello world";sleep 10; done">j1.yaml

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}

--
kubectl apply -f jb1.yaml
kubectl get job
kubectl get po   # After deadline exceeded the pod will be terminated
kubectl logs po_name
``````
##### Ex 2.
Create the same job, make it run 5 times, one after the other. Verify its status and delete it
``````sh
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}

---
kubectl apply -f job.yaml
kubectl get job busybox -w # will take two and a half minutes
kubectl delete jobs busybox
``````
###### Ex3.
Create the same job, but make it run 5 parallel times
It will take some time for the parallel jobs to finish (>= 30 seconds)
``````sh
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
---
kubectl apply -f job.yaml
kubectl get jobs
kubectl delete job busybox
``````
``````sh


``````