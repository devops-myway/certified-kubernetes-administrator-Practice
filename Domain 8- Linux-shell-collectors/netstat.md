##### netstat
- Netstat is another powerful utility that is used to display network statistics about network connections, routing tables, and interfaces.
- This is pretty useful in multiple scenarios, let's say for troubleshooting network issues (for example, port or connection issues)
- or to monitor network traffic and to detect malicious activity.
``````sh
# To display all the active network connections
$ netstat -a

#To display all the listening ports:
$ netstat -l

# To display all TCP ports:
$ netstat -at

# To display all UDP ports:
$ netstat -au

#To display kernal IP routing table:
$ netstat -r

#To display the interface table:
$ netstat -i
``````
##### 

``````sh


``````
