


##### Systemd logs to find out about the problem
Systemd is a Linux service and system manager.One of the most powerful systemd functionalities is the logging features
Systemd provides a centralized solution for logging all kernel and user processes through logs known as journals..

Systemd has three basic components as follows –

- systemd: System and service manager for Linux operating systems.
- systemctl: Command to introspect and control the state of the systemd system and service manager.
- systemd-analyze: Provides system boot-up performance statistics and retrieve other state and tracing information from the system and service manager
- there are additional services that systemd provides such as – journald, logind, networkd, etc.

##### journald – systemd journal daemon
- systemd provides a centralized way of handling all operating system logs from processes, applications, etc. All these logging events are handled by journald daemon of systemd.
- The journald daemon collects all logs from everywhere of the Linux operating systems and stores themes as binary data in files.

##### The journald Configuration File
The configuration file of the journald is present in the below path, But I would recommend not to modify this file unless you know what you are doing.
``````sh
/etc/systemd/journald.conf

``````
##### Where journald stores the binary log files
The journald stores the logs in binary format. Do not use cat command or use nano or vi to open these files. They would not be displayed properly
``````sh
/var/log/journal
---
cd /var/log
cd journald
ls -lh
``````
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