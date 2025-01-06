

###### Check running process in Linux

The procedure to monitor the running process in Linux using the command line is as follows:

The ps command is a traditional Linux command to lists running processes.

The new categories of the expanded output include:

- USER: The name of the user running the process.
- %CPU: The CPU usage percentage.
- %MEM: The memory usage percentage.
- VSZ: Total virtual memory used by the process, in kilobytes.
- RSS: Resident set size, the portion of RAM occupied by the process.
- STAT: The current process state.
- START: The time the process was started.

###### Foreground Processes:
By default, every process that you start runs in the foreground. It gets its input from the keyboard and sends its output to the screen
``````sh
ls ch*.doc
``````
###### Background Processes
A background process runs without being connected to your keyboard. If the background process requires any keyboard input, it waits.
``````sh
ls ch*.doc &
``````
##### Listing Running Processes
It is easy to see your own processes by running the ps (process status)
One of the most commonly used flags for ps is the -f ( f for full) option

- UID: User ID that this process belongs to (the person running it), root, reguslar user or service account or system user.
- PID: Process ID
- TTY: Terminal type associated with the process
- TIME: CPU time taken by the process
- C: CPU utilization of process
- CMD: The command that started this process
- STIME: Process start time
- PPID: Parent process ID (the ID of the process that started it)

``````sh
ps
ps -f
``````
##### Process Status
Process Status”. ps command is used to list the currently running processes and their PIDs along with some other information depends on different options.
``````sh
sudo ps -e
ps -aux | more
sudo ps -aux | less
``````
##### Linux top command
The top command is another highly recommended method to see your Linux servers resource usage

``````sh
sudo top
sudo top [options]

``````
##### Linux htop command to check running process in Linux
- The htop command is an interactive process viewer and recommended method for Linux users
- One can see a list of top process that using the most memory or CPU or disk and more
``````sh
sudo htop [options]
htop
``````
##### Linux kill command
Want to kill a process
``````sh
kill pid
kill 16750
``````
``````sh

``````
``````sh

``````
