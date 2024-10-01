##### How To Read and Set Environmental and Shell Variables on Linux
- Variables usually are meant to convey information from one system to another. you can set them by defining a variable name, and then setting its value.
- You can see the results for yourself with the echo command, recalling your variable by prepending it with a dollar sign ($)
- To ensure that the variable is read exactly as you defined it, you can also wrap it in braces ${Variable_key} and quotes.

``````sh
KEY=value1:value2:...
KEY="value with spaces"

env | grep any_env_variable
env
env | grep any_env_variable
env | grep PATH
printenv
printenv PATH

FOO="/home/seth/Documents"
echo "${FOO}"

``````

##### How to clear a variable using unset command
You can clear a variable with the unset command:

``````sh
unset FOO
echo $FOO 
``````
##### Prepending variables
You can prepend any number of variables before running a command.
``````sh
FOO=123 bash
echo $FOO

``````
##### Exporting variables
Another way to make variables available to a child process is the export keyword.

``````sh
FOO=123
export FOO
echo $FOO

export NEW_VAR="Testing export"
printenv | grep NEW_VAR
echo $NEW_VAR
``````

##### Use env. to set commands in Bash
- env FOO=BAR command. Note that the environment variables will be restored/unchanged again when command finishes executing.

``````sh
$ export FOO=BAR
$ env FOO=FUBAR bash -c 'echo $FOO'
FUBAR
$ echo $FOO
BAR

``````

``````
``````sh


``````
``````sh


``````
