

#### Exit Code

An exit code, or sometimes known as a return code, is the code returned to a parent process by an executable.
On POSIX systems the standard exit code is 0 for success and any number from 1 to 255 for anything else.

###### How to get the exit code of a command
``````sh

cat file.txt
hello world
echo $?

0

``````
``````sh
cat doesnotexist.txt
cat: doesnotexist.txt: No such file or directory
echo $?

1
``````
##### Using Exit code in Script
``````sh
#!/bin/bash
cat file.txt
if [ $? -eq 0 ]
then
  echo "The script ran ok"
  exit 0
else
  echo "The script failed" >&2
  exit 1
fi


``````
