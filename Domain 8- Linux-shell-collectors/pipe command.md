###### Pipe
A pipe is a form of redirection (transfer of standard output to some other destination)
This purpose is why the most popular use for pipes involves the commands grep and sort

``````sh
command_1 | command_2 | command_3 | .... | command_N

``````
###### List all files and directories and give them as input to `grep` command using piping in Linux
List all files and directories and give them as input to `grep` command using piping in Linux
``````sh
ifconfig | grep 192
ifconfig | grep UP
 ls | grep 'file' > geeks.txt
``````
###### How can I redirect the output of a piped command to file in Unix or Linux

``````sh
ls | grep file.txt
ls -l | more

``````
###### Counting files & Sorting results

``````sh
ls | wc -l

cat contacts.txt
cat contacts.txt | sort
cat contacts.txt | sort -r
``````
###### Echo command

``````sh

``````
