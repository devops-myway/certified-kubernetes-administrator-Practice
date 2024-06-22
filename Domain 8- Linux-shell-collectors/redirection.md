##### Shell I/O redirection

There are three redirectors to work with: >, >>, and <. The following information describes each one:

> character can be used to direct output into a file
>> This command will append output to file or add the text provided to the end of a file.

``````sh
ls -al > listings
cat listing

$ echo hello > world
$ echo hello > world
$ cat world
hello

$ echo "My Report" > report
$ date >> report
$ cat report
My Report
Sat Jul  8 11:49:48 AM EDT 2023
---
cat /dev/null > bigfile

--
$ echo "Enable Sysadmin" > myfile
$ cat myfile
Enable Sysadmin
----
$ cat myfile
/usr/bin/ls
$ echo "Enable Sysadmin" >> myfile
$ cat myfile
/usr/bin/ls
Enable Sysadmin
---

``````
##### Shell piping - will print the output as opposed to redirection operator
To "pipe" one command to another, separate the commands with this operator.
command1 | command2 | command3

Feed the grep input command with the output of the cat command:

The tee utility will have to run as root though, and tee -a will append data

``````sh
cat /etc/passwd | grep localuser
---
printf "Sysadmin\nEnable\n" > myfile | sort myfile
---
echo 'clock_hctosys="YES"' | sudo tee -a /etc/conf.d/hwclock >/dev/null
``````
 appends  >> , > always overwrites/destroys the previous content.
