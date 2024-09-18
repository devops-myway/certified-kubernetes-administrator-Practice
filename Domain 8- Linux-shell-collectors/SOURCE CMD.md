
##### Source Command Syntax
source is a built-in shell command that reads and executes the file content in the current shell.
The source command also allows you to read variables from a file. 
Start by creating an example configuration file example_config.sh in the Home directory and adding the following content:

´´´´´sh
source [filename] [arguments]

example_config.sh
VAR1="a"
VAR2="b"
VAR3="c"

#!/usr/bin/env bash
source example_config.sh
echo "VAR1 is $VAR1"
echo "VAR2 is $VAR2"
echo "VAR3 is $VAR3"

source example_bash.sh
´´´´´
##### Refresh the Current Shell Environment
To make it permanent, open the bashrc file with: Refresh the current shell environment with the source command:
´´´´´sh
alias ll = 'ls -l'
sudo vim ~/.bashrc
alias ll = 'ls -l'

source ~/.bashrc
´´´´´
