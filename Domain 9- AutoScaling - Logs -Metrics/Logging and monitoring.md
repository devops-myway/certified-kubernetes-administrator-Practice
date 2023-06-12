

https://devopscube.com/kubernetes-logging-tutorial/


##### Kubernetes Logging and monitoring

Kubelet runs on all the nodes to ensure the containers on the node are healthy and running.
It is also responsible for running the static pods as well. If kubelet runs as a systemd service, it writes logs to journald daemon of systemd.

#### Kubernetes Pod Log Location
The journald daemon collects all logs from everywhere of the Linux operating systems and stores themes as binary data in files

/var/log/journal:    The journald stores the logs in binary format in linux OS.
/var/log/containers: All the container logs are present in a single location.
/var/log/pods/: Under this location, the container logs are organized in separate pod folders.
/var/log/pods/<namespace>_<pod_name>_<pod_id>/<container_name>/:  If you log in to any Kubernetes worker node and go to /var/log/containers the directory, you will find a log file for each container running on that node with this naming scheme.

##### Types for Kubernetes logs
Application logs: Logs from user deployed applications. Application logs help in understanding what is happening inside the application.
Kubernetes Cluster components: Logs from api-server, kube-scheduler, etcd, kube-proxy, etc. These logs help you troubleshoot Kubernetes cluster issues.
Kubernetes Audit logs: All logs related to API activity recorded by the API server. Primarily used for investigating suspicious API activity.

##### Kubernetes Logging Architecture
If we take the Kubernetes cluster as a whole, we would need to centralize the logs.
Let’s understand the three key components of logging.

- Logging Agent: A log agent that could run as a daemonset in all the Kubernetes nodes that streams the logs continuously to the centralized logging backend.
The logging agent could run as a sidecar container as well. For example, Fluentd.

- Logging Backend: A centralized system that is capable of storing, searching, and analyzing log data. A classic example is Elasticsearch.

- Log Visualization: A tool to visualize log data in the form of dashboards. For example, Kibana.

##### journalctl command
The journalctl command enables viewing and editing the systemd logs, making it a powerful tool for service and process debugging
The journalctl command queries and manipulates the journal data collected by the journald daemon.
``````sh
journalctl <options> <matches>

journalctl  # to display all journal entries

Exit the journal by pressing q.

``````
##### How to view journal entries for time zones

``````sh
journalctl --utc

``````
##### How to view only errors, warnings, etc in journal logs
To view emergency system messages use
 priority 2, 1 and 0
``````sh
journalctl -p 0
-----
0: emergency
1: alerts
2: critical
3: errors
4: warning
5: notice
6: info
7: debug
``````
##### How to view journal logs for a specific boot

``````sh
journalctl --list-boots

journalctl -b -45 #specific boot number you the first number or the boot ID 

``````
#####  view journal logs for a specific time, date duration
The journalctl is powerful enough to provide “english” like argument in the command itself for time and date manipulation.
You can use --since switch with a combination of “yesterday”, “today”, “tomorrow”, or “now”.
``````sh
journalctl --since "2020-12-04 06:00:00"

journalctl --since "2020-12-03" --until "2020-12-05 03:00:00"

journalctl --since yesterday

journalctl --since 09:00 --until "1 hour ago"

``````
##### How to see Kernel specific journal logs

``````sh
journalctl -k

journalctl -u NetworkManager.service  #How to see journal logs for a service, PID

systemctl list-units --type=service

journalctl --disk-usage
`````` 
##### How to view journal logs for an executable

``````sh
journalctl /usr/bin/gnome-shell --since today
``````
##### journalctl to debug Kubernetes
The command journalctl -x -f is used to view the logs of all units in the system
-x : option providing additional output that includes the unit's process ID and exit code
-f : option is used to "follow" the logs, so that new log entries will be displayed as they are generated.
-u: to filter by unit
-p: to filter by priority

``````sh
journalctl -u kubelet  #To view the logs for all pods in a specific namespace

journalctl -u kubelet -n kube-system -f | grep <pod-name>   #To view the logs for a specific pod in a namespace

journalctl -u kubelet -n kube-system -f | grep <container-name>   #To view the logs for a specific container in a pod:

journalctl -u kubelet -n kube-system -f | grep <node-name>      #To view the logs for a specific node:

journalctl -u kube-apiserver   # To see the logs of kube-apiserver:

journalctl -u kube-controller-manager  #To see the logs of kube-controller-manager:

journalctl -u kube-scheduler     #To see the logs of kube-scheduler:

``````
You can also filter the logs by pod,node,container name etc. using grep command.
``````sh
journalctl -u kubelet -f | grep <pod-name>

journalctl -xeu kubelet -f | grep kube-apiserver
``````
