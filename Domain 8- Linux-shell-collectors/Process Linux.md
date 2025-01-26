###### Processes in Linux/Unix
- A program/command when executed, a special instance is provided by the system to the process
- Whenever a command is issued in Unix/Linux, it creates/starts a new process. For example, pwd when issued, which is used to list the current directory location that the user is in, starts a process.
- Using a 5-digit ID number, Unix/Linux keeps track of processes, this number is called the process ID or PID. Each process in the system has a unique PID.
- 
###### Foreground Processes:
By default, every process that you start runs in the foreground. It gets its input from the keyboard and sends its output to the screen
``````sh
ls password

/home/geeksforgeeks/root
``````
###### Background Processes
A background process runs without being connected to your keyboard. If the background process requires any keyboard input, it waits.
``````sh
password &

``````
##### Monitoring of ongoing processes
- It is easy to see your own processes by running the ps (process status)
- One of the most commonly used flags for ps is the -f ( f for full) option

- Fields described by ps are described as: 

- UID : User ID that this process belongs to (the person running it)
- PID : Process ID
- PPID : Parent process ID (the ID of the process that started it)
- W : CPU utilization of process
- STIME : Process start time
- TTY : Terminal type associated with the process
- TEAM : CPU time is taken by the process
- CMD : The command that started this process

- There are other options which can be used along with ps command:

- -the : Shows information about all users
- -x : Shows information about processes without terminals
- -u : Shows additional information like -f option
- -and : Displays extended information

``````sh
# ps (Process Status) can be used to view/list all running processes.
ps

PID TTY TIME CMD
19 points/1 00:00:00 sh
24 points/1 00:00:00 ps

#For more information -f (full) can be used together with ps
ps –f

UID PID PPID C TIME TTY TIME CMD
52471 19 1 0 07:20 points/1 00:00:00f sh
52471 25 19 0 08:04 points/1 00:00:00 ps -f
``````
##### Stopping a process
- When running in foreground, hitting Ctrl + c (interrupt character) will exit the command.
- For processes running in background kill command can be used if it's pid is known
``````sh
$ ps –f

UID PID PPID C STIME TTY TIME CMD
52471 19 1 0 07:20 pts/1 00:00:00 sh
52471 25 19 0 08:04 pts/1 00:00:00 ps –f

$ kill 19 
Terminated
``````
##### Namespaces - Namespaces are fundamental blocks of Linux containers.
- Namespaces are a feature of the Linux kernel. A process can use a resource set not seen by another process. Namespaces isolate processes between them.
- Resource sets are kernel resources for example Process IDs, Hostnames, User IDs, Filenames, Networks, etc
- If the process changes the global resource(like PID) under a specific namespace, this change can be seen only by processes in the same namespace.
-  The first process of the PID namespace takes PID 1 and the subsequent process goes sequentially.

- There are several namespace types in the Linux kernel:
- User Namespace: It includes independent user and group IDs that may be given to processes. A process can escalate to the root user in its namespace.
- PID Namespace: PID namespaces isolate process IDs. Different processes under different PID namespaces can have the same process ID.
- Network Namespace: It is used for isolating a network. Processes in the network namespace can use custom/specific routing tables, IP addresses, network devices, and other network resources in an isolated manner from other network namespaces.
- Mount Namespace: The file system is mounted for processes under the namespace by not affecting the host file system
- IPC Namespace: Processes under this namespace can use IPC resources in an isolated manner. For example message queues, shared memory, and SystemV IPC objects.
- 
##### Control Group (cgroup)
- With control groups, CPU, disk, network, memory, and other system resources can be limited. We can create resource limits with cgroups.
- Prioritization: We can prioritize a process in the namespace with cgroup.
- Accounting: Resource limits are monitored and reported at a cgroup level.
- Control: Processes under the same cgroup can be managed by a single command.

``````sh
# Creating a new memory cgroup
sudo cgcreate -g memory:my-memory-limiter
ls -la /sys/fs/cgroup/memory/my-memory-limiter/

# Restricting memory:
sudo cgset -r memory.limit_in_bytes=50M my-memory-limiter
cat /sys/fs/cgroup/memory/my-memory-limiter/memory.limit_in_bytes

``````

``````sh

``````
