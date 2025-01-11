##### What is Journalctl
https://www.ionos.ca/digitalguide/server/tools/journalctl/
- journalctl is a powerful utility for querying and displaying event logs or logfiles under Linux.
- The name “journalctl” is a mix of “journal” (log) and “ctl” (control), which refers to the fact that the command is used to control and analyze logs.

#### How to adjust storage space for log files
- journalctl can be used to limit and configure storage space that log files occupy on the hard disk. This is done via the systemd-journald service settings
- The configuration settings are stored in the file /etc/systemd/journald.conf
- You can also use different units such as K (kilobytes), M (megabytes), G (gigabytes) or T (terrabytes).
- 
``````sh
[Journal] 
SystemMaxUse=50M 
RuntimeMaxUse=50M

#systemd-journald must be restarted 
sudo systemctl restart systemd-journald

``````
##### Assess disk space usage
- check how much storage space is currently occupied by the journal. To do this, use --disk-usage:
``````sh
journalctl --disk-usage
#output
Journals take up 8.0M on disk.

``````
##### Delete old log entries
- Use --vacuum-size to reduce the size of your journal by specifying the size
``````sh
sudo journalctl --vacuum-size=1G
sudo journalctl --vacuum-time=1years

``````
##### Filter by time window
- journalctl offers the options --since and --until, which can be used to restrict the entries to a specific period
``````sh
journalctl --since "2023-01-01 12:00:00" --until "2023-01-02 12:00:00"

journalctl --since 09:00 --until "1 hour ago"
journalctl --since "2023-11-16 15:25:00"
journalctl --since yesterday
``````
##### Filter by priority
- To filter logs with journalctl by message importance, you can use the priority categories for log entries.
- To do this, you can use either the priority name or its corresponding numerical value. The lower the number, the more important the message:

- 0: emerg (emergency)
- 1: alert (alarm)
- 2: crit (critical)
- 3: err (error)
- 4: warning (warning)
- 5: notice (note)
- 6: info (information)
- 7: debug (troubleshooting)

``````sh
journalctl -p err
journalctl -u apache2
journalctl -u apache2 --since today
journalctl -u nginx.service -u php-fpm.service --since today-u apache2 --since today

``````
##### Filter by process, user or group ID
- Journalctl can filter log entries by process, user or group ID. If you have the exact PID of the process you want to search for, you can use the _PID option
- you can use the _UID or _GID filters to display all entries logged by a particular user or group.
``````sh
journalctl _PID=8088

id -u www-data

journalctl _UID=33 --since today
journalctl -F _GID
``````
##### 

``````sh


``````

