

###### Check running process in Linux

The procedure to monitor the running process in Linux using the command line is as follows:

The ps command is a traditional Linux command to lists running processes

The new categories of the expanded output include:

USER: The name of the user running the process.
%CPU: The CPU usage percentage.
%MEM: The memory usage percentage.
VSZ: Total virtual memory used by the process, in kilobytes.
RSS: Resident set size, the portion of RAM occupied by the process.
STAT: The current process state.
START: The time the process was started.

``````sh
ps -aux
sudo ps -a

``````
The process ID (PID) is essential to kill or control process on Linux, with example below
root – User name
1 – PID (Linux process ID)
19:10 – Process start time
/sbin/init splash – Actual process or command
``````sh
root         1  0.0  0.0 225868  9760 ?        Ss   19:10   0:13 /sbin/init splash
``````

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
The htop command is an interactive process viewer and recommended method for Linux users
One can see a list of top process that using the most memory or CPU or disk and more
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