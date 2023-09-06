kubernetes.io > Documentation > Tasks > Run Jobs > https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

A CronJob definition includes the schedule of the job and the job template that specifies the container images, commands, and arguments to be executed.
Controller check: CronJob controller checks things (watching and syncing jobs) every 10 seconds.

##### CronJob Specifications

CronJobs also have several specifications that can be customized depending on the requirement of your workload.

successfulJobHistoryLimit: which specifies the number of completed jobs to keep in the history.
failedJobsHistoryLimit: which specifies the number of failed jobs to keep in the history.
ConcurrencyPolicy: specifies how the CronJob handles concurrent job execution.
- Allow which is the default, allows concurrent execution of the same job,
- Forbid prevents concurrent execution of the same job (where jobs are run sequentially), 
- Replace cancels the currently running job and replaces it with a new one.
Suspending a CronJob: can be done using kubectl apply or patch commands. This can be useful when you need to temporarily stop the execution of a CronJob without deleting it.
startingDeadlineSeconds:
The Starting Deadline is an optional field that specifies the deadline in seconds for starting the job if it misses "its scheduled time" for any reason. After the deadline, CronJob does not start the job. Jobs that do not meet their deadline in this way count as failed jobs.

It is important to note, that if the field startingDeadlineSeconds is set (not nil), it will count how many missed jobs occurred from the value of startingDeadlineSeconds till now. For example, if startingDeadlineSeconds = 200, It will count how many missed jobs occurred in the last 200 seconds.
Note that if the startingDeadlineSeconds is set to a value of less than 10 seconds, the CronJob may not be scheduled. This is because the CronJob controller checks things every 10 seconds.

##### Schedule syntax

# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *
Steps are also permitted after an asterisk, so if you want to say "every two hours", just use */2.

###### Ex1.
Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

``````sh
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" --dry-run=client -oyaml  -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'

kubectl get po   # copy the container just created
kubectl logs <container> # you will see the date and message 
kubectl delete cj busybox --force #cj stands for cronjob and --force to delete immediately 
``````

###### Ex 2 
Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).

``````sh
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 # add this line
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
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
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}

--
kubectl get cj
kubectl get po
kubectl logs container_name
kubectl delete cj/time-limited-job

``````
###### Ex 3.
Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.
``````sh
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      activeDeadlineSeconds: 12 # add this line
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
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}

``````
##### Create a Job from Cronjob

Create a job from cronjob.

``````sh
kubectl create job --from=cronjob/sample-cron-job sample-job

``````
###### Ex - Exams

``````sh


``````

