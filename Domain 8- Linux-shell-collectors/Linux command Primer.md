

##### 

$ or dollar sign — you are logged in as a regular user
# or hashtag/pound symbol — you are logged in as a user with elevated privileges or root user.

Folder Interpretation:
--------
d	rwxr-xr-x	2	sammy	sammy	4.0K	Nov 13 18:06	files
-------

d or directory (or folder) — a type of file that can hold other files, useful for organizing a file system; if this were - instead, this would refer to a non-directory file
r or read — permission to open and read a file, or list the contents of a directory
w or write — permission to modify the content of a file; and to add, remove, rename files in a directory
x or execute — permission to run a file that is a program, or to enter and access files within a directory

-rwx, where the first hyphen signifies a non-directory file

Regardless of where we are in a filesystem, we can always use the command cd / to move to the primary directory.

The / directory is the main directory of a Linux server, referred to as the “root” directory.

A single . stands for the directory you are currently in, and a double .. stands for the parent directory

~ which stands for the home directory of your machine. Here, our home directory is called /home/sammy for the sammy user

cat command. Short for concatenate, the cat command is very useful for working with files. Among its functions is showing the contents of a file.

``````sh
pwd

mkdir files

ls
ls -l

cd files
cd /home/sammy/files
cd /
cd ..
cd ~

cd /home/sammy/files
touch ocean.txt
ls
echo Hello, World!
echo "Sammy the Shark" > sammy.txt
ls

cat sammy.txt
echo "Some people could look at a mud puddle and see an ocean with ships." > ocean.txt
cat ocean.txt


rm -r students
rm -rf text1.txt

cp sample.txt newsample.txt
mv newsample2.txt newdir


``````

###### Working with Files from the Web

With the terminal, you can also directly access cloud servers that you have credentials for, manage and orchestrate cloud infrastructure, build your own web apps, and more.

use the curl command to transfer data from the web to our personal interactive terminal on the browser

In order to save the text to a file, we’ll need to run curl with the -O flag, which enables us to output the text to a file, taking the same name of the remote file for our local copy.

to use a specific and alternate name of the file, you could do so with the -o flag and pass the name of the new file as an argument (in this case, jules.txt).

``````sh
curl https://assets.digitalocean.com/articles/command-line-intro/verne_twenty-thousand-leagues.txt


curl -O https://assets.digitalocean.com/articles/command-line-intro/verne_twenty-thousand-leagues.txt

curl -o jules.txt https://assets.digitalocean.com/articles/command-line-intro/verne_twenty-thousand-leagues.txt

``````
#### TAC

tac command in Linux is used to concatenate and print files in reverse.

tee command reads the standard input and writes it to both the standard output and one or more files

time command in Linux is used to execute a command and prints a summary of real-time, user CPU time and system CPU time spent by executing a command when it terminates.

until command in Linux used to execute a set of commands as long as the final command in the ‘until’ Commands has an exit status which is not zero.

watch command in Linux is used to execute a program periodically, showing output in fullscreen

date command is used to display the system date and time

Sleep command is used to delay for a fixed amount of time during the execution of any script.
You can set the delay amount by seconds (s), minutes (m), hours (h) and days (d).

``````sh
tac -b concat.txt tacexample.txt

wc -l file1.txt|tee -a file2.txt

time sleep 3

until COMMANDS; do COMMANDS; done

watch -d  free -m

date

now = $(date)
echo $now

ls && sleep 2 && pwd

---
#!/bin/bash
 
echo "Waiting for 2 seconds..."
sleep 2
echo "Task Completed"
``````