

###### set Command

We use the set command to change the values of shell options and display variables in Bash scripts.
We can also use it to debug Bash scripts, export values from shell scripts, terminate programs when they fail, and handle exceptions.

set [option] [argument]

###### When we run the set command without any arguments, it returns a list of all system settings.
The list  includes the names and values of all shell variables and functions:

 set

##### The -e Option
When a query returns a non-zero status, the -e flag stops the script. It also detects errors in the currently executing script.

``````sh
!/bin/bash 
set -e 
mkdir newfolder 
cat filenotindirectory 
echo 'The file is not in our directory!'

# We run the executable file to test out the -e flag:

chmod u+x eflag.sh
./eflag.sh
cat: can't open 'filenotindirectory': No such file or directory

``````
##### The -C Option
Using the -C flag ensures that we cannot overwrite an existing file with the same name:
``````sh
echo 'An existing file' > myfile
set -C
echo 'Editing an existing file' > myfile
sh: can't create myfile: File exists

``````
##### The -f Option
we can easily search for files using wildcard characters such as ?, *, or []. 
This method is similar to regex, where we try to find similar texts using patterns.
``````sh
ls *.txt
files.txt
--------- # let’s run the set -f command and then search for files with the .txt extension using a wildcard:
set -f
ls *.txt
ls: *.txt: No such file or directory

``````
##### The -x Option
We use the -x parameter when debugging our scripts to determine the output of individual commands.
We can see that the code executes one line at a time, displaying results before moving to the following line
``````sh
!/bin/bash
set -x
n=3
while [ $n -gt 0 ]; do
    n=$[ $n-1 ]
    echo $n
    sleep 1
done
----- # Then, we execute the script:

bash debugging.sh
+ n=3
+ '[' 3 -gt 0 ']'
+ n=2
+ echo 2
2
+ sleep 1
+ '[' 2 -gt 0 ']'
+ n=1
+ echo 1
1
+ sleep 1
+ '[' 1 -gt 0 ']'
+ n=0
+ echo 0
0
+ sleep 1
+ '[' 0 -gt 0 ']'
`````` 
#### The -a Option
We can export variables or functions with this flag.
making them reusable in subshells or script.
We can see above that the script outputs the created variables, as determined by the echo command.
``````sh
set -a 
name='May' 
age=22
----
!/bin/bash 
echo $name $age
---
bash export.sh 
May 22

``````
``````sh
name='May' 
age=22

---
!/bin/bash 
echo $name $age
``````
