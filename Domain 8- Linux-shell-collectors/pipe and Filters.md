###### Pipe
You can connect two commands together so that the output from one program becomes the input of the next program.
To make a pipe, put a vertical bar (|) on the command line between two commands.

``````sh
command_1 | command_2 | command_3 | .... | command_N

``````
###### List all files and directories and give them as input to `grep` command using piping in Linux
List all files and directories and give them as input to `grep` command using piping in Linux
``````sh
ls -l | grep "Aug"
ls -l | grep -i "carol.*aug"
ifconfig | grep 192
ifconfig | grep UP
 ls | grep 'file' > geeks.txt
``````
###### How can I redirect the output of a piped command to file in Unix or Linux

``````sh
ls | grep file.txt
ls -l | more
ls -l | grep -i "carol.*aug"
``````
###### The sort Command
The sort command arranges lines of text alphabetically or numerically. The following example sorts the lines in the food file
-c: Prints only the count of matching lines.
+x: Ignores first x fields when sorting.
-r: Reverses the order of sort.
``````sh
sort food
ls -l | grep "Aug" | sort +4n
ls | wc -l

cat contacts.txt
cat contacts.txt | sort
cat contacts.txt | sort -r
``````
###### Echo command

``````sh

``````
