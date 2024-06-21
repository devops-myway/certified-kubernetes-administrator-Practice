###### grep
The grep command is primarily used to streamline and automate text processing and data extraction tasks.
Grep is a Linux command-line tool that allows users to search files for a specified textual pattern
``````sh
grep '[search_pattern]' [file_name]
grep 'cloud computing' example_file2.txt

``````
###### Search a File

``````sh
grep 'bare metal' example_file2.txt
grep 'ransomware' *

``````
###### Use grep with Pipes
grep can be combined with other commands through piping (|) for more complex searches or data processing.
``````sh
cat example_file2.txt | grep 'server'
ls -l | grep -i xyz

# Sends output to STDOUT. Output from cat file is sent via STDOUT to grep's STDIN via the pipe
cat file
cat file | grep 5

``````
###### Echo command

``````sh

``````
