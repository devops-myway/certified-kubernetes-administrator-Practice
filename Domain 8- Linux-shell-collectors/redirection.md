##### Shell I/O redirection

There are three redirectors to work with: >, >>, and <. The following information describes each one:

- Redirection with >
command > file: Sends standard output to <file>

- Append with >>
command >> file: Appends standard output to a file
command >> file 2>&1: Appends standard output and error output to a file

Redirect with <
command < input: Feeds a command input from <input>

cmd1 > out1
cmd2 < out1 > out2
cmd3 < out2

``````sh
ls -al > listings
cat listing
---
cat music.mp3 > /dev/audio

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