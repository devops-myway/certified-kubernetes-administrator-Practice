https://kubernetes.io/docs/concepts/workloads/controllers/job/

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

##### Kubernetes Jobs

Jobs are used to run one or more containers to completion, and they're often used for running batch processing tasks, database migrations, or resource-intensive jobs.
Jobs are useful when you want to run a one-time task, or when you want to run a task multiple times until it succeeds, with a specified number of retries.

#### Kubernetes CronJobs
Cronjobs are used to run a task on a specified schedule, such as running a backup every day at 5 pm.
CronJobs are useful when you want to run a task regularly at a specific interval, such as running a daily report or cleaning up old logs.

#### Understand the YAML manifest for Kubernetes Jobs
``````sh
kubectl create job echo-job --image=busybox --dry-run=client -oyaml -- sh -c "echo Hello K8s" > job1.yaml
vi job1.yaml
-----
apiVersion: batch/v1
kind: Job
metadata:
  name: echo-job
spec:
  template:
    metadata:
      name: echo
    spec:
      containers:
      - name: echo
        image: busybox
        command: ['echo', 'Hello Kubernetes Jobs!']
      restartPolicy: Never
  backoffLimit: 4
-----
kubectl get job
kubectl describe job/echo-job
kubectl logs jobs/echo-job

``````

#### How to use Kubernetes CronJobs
https://crontab.guru/

# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *

5 4 4 10 *  # Example 
2023-10-04 04:05:00


``````sh
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: simple-cron-job
spec:
  schedule: "* * * * *" # run every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: simple-cron-job
            image: busybox
            command:
            - "echo"
            - "This is a simple cron job."
          restartPolicy: OnFailure
  concurrencyPolicy: Allow

``````