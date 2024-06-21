
###### Echo command
The echo command is a way to communicate with your Linux terminal. It allows you to send text, variables, and special characters to the standard output, which is usually the terminal screen.

##### Display Variable
The command echo “$variable” is used to print the value of a variable to the standard output
The $ symbol is used to access the value of a variable.
$USER is a predefined variable that holds the name of the current user

##### Print Special Characters
command echo “\n” is used to print a special character to the standard output

##### Writing to a File
> symbol is used to redirect the output of the echo command to a file instead of the terminal screen
>> symbol will append the output.

#### Writing to Both Terminal and File
you can use the echo “content” | tee file command to create a file with some content to the standard output.
The | symbol is used to pipe the output of the echo command to another command,
The tee command is used to write the input to both the standard output and a file or files

##### Delete Files
The rm -rf * command will delete all files and directories in the current working.

``````sh
echo rm -rf <prefix>*

``````
###### Execute Other Commands

``````sh
echo $(date)
$(date)
echo $varName  # not advisable unless you know what the variable contains
echo "$varName"
echo "message here"
echo "My home directory is $HOME. My shell is $SHELL."
echo -n "say something cool here"

``````
###### Count number of characters and printing variables

``````sh
echo "Hello" | wc -c

echo "Today is $(date)."
echo "The current working directory is $(pwd)"

``````
###### Generating Output With echo command

``````sh
#!/bin/bash
# Display welcome message, computer name and date
echo "*** Backup Shell Script ***"
echo
echo "*** Run time: $(date) @ $(hostname)"
echo

# Define variables
BACKUP="/nas05"
NOW=$(date +"%d-%m-%Y")
echo $(NOW)
 
# Let us start backup
echo "*** Dumping MySQL Database to $BACKUP/$(NOW...)"

# Just sleep for 3 secs
sleep 3

# And we are done...
echo
echo "*** Backup wrote to $BACKUP/$NOW/latest.tar.gz"
``````
##### Printing file names with echo

``````sh
cd /etc 
echo *.conf

``````
