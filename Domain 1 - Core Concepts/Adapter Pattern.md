https://kubernetes.io/docs/concepts/workloads/pods/

###### Adapter - container + adapter(for checking other container's status. e.g. monitoring)

 A container that transform output of the main container. you’re running multiple applications within separate containers, but every application has a different way of outputting log files.

 ###### Example 1.
Then we cat the log file /var/log/out.log for the log-adapter, we see a Date prefixed to the output that was not in the original log /var/log/app.log

 ``````sh
apiVersion: v1
kind: Pod
metadata:
  name: adapter-pod
  labels:
    app: adapter-app
spec:
  volumes:
  - name: logs
    emptyDir: {}
  containers:
  - name: app-container
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> /var/log/app.log; sleep 2;done"]
    volumeMounts:
    - name: logs
      mountPath: /var/log
  - name: log-adapter
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", "tail -f /var/log/app.log|sed -e 's/^/Date /' > /var/log/out.log"]
    volumeMounts:
    - name: logs
      mountPath: /var/log

``````