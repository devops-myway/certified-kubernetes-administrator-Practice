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


``````

#### How to use Kubernetes CronJobs

``````sh
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: simple-cron-job
spec:
  schedule: "*/1 * * * *" # run every minute
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