https://phoenixnap.com/kb/grep-command-linux-unix-examples

###### What Is grep
- Grep is a Linux command-line tool that allows users to search files for a specified textual pattern.
- When grep finds a match, it prints lines containing that pattern to the terminal.
- grep Options
- -i	The search ignores differences between uppercase and lowercase letters.
- -v	Display only lines that do not match the search pattern.
- -w	Match only whole words.
- -r or -R	Search through subdirectories for the pattern. -R also follows symbolic links.
- -x	Match only whole lines.
- -l	List files with the matching pattern only once.
- -L	List files that do not contain the pattern.
- -c	Count how many lines match the search pattern.
- -o	Display only the matching part of each line.
- -a	Search binary file as text.
- -m NUM	Stop searching a file after matching the following number (NUM) of lines.
- -A NUM	Show the following number (NUM) of lines after each matching line.
- -B NUM	Show the following number (NUM) of lines that precede each matching line.
- -C NUM	Show the following number (NUM) of lines around matching lines, combining the effects of -A and -B.
- -n	Show the line number for each matching line.
- -e or -E	Use extended regular expressions for complex patterns.
- -h	Don't show file names when searching multiple files.
- -q	Do not display results; only indicate if there were any matches.
- -s	Hide errors about files that can't be found or read.
``````sh
Syntax:
grep '[search_pattern]' [file_name]

``````
###### Ignore Case in Grep Searches
- grep commands are case-sensitive. Use the -i option to display both uppercase and lowercase results
``````sh
grep -i 'bare metal' example_file2.txt
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
###### Search All Files in Directory
- To search all files in the current directory, use an asterisk (*) instead of a filename at the end of a grep command.
``````sh
grep 'ransomware' *
``````
