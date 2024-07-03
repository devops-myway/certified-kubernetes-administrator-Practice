##### One Liners
Sometimes we need to run multiple Bash commands in a single line in Linux. Such commands are called one-liners.
They are useful when we don’t want to wait until the execution of one command completes before issuing another.
##### Using &
The & character is used to run a command in the background, allowing the user to move onto other tasks while the command runs to completion.

``````sh
bigjob &
``````
##### Using &&
The && is a logical operator. It means that it takes into account the state of the previous command
if the first command fails (i.e., produces a non-zero exit code), then the second command won’t be executed.
``````sh
touch new_file && ls new_file
new_file

$ ping 192.168.0.1 && echo router is reachable
router is reachable

# Run on Error
non-existing-command
non-existing-command: command not found

``````
##### Using ;
the ; operator is simpler and doesn’t check the output of previous commands.
all the commands will be executed regardless of their status as long as the Bash syntax is correct.

``````sh
non-existing-command ; echo "Hello"
non-existing-command: command not found
Hello

``````

##### Using ||
the || operator is a logical OR operator. It works like the && one, but the other way around.
operator executes the second command only if the first command fails.

The second echo command didn’t run, because the first one was successful.
``````sh
echo "1" || echo "2"
1

``````
#####  ||
However, if we use the non-existing command like in the examples above, we’ll see another result:

``````sh
non-existing-command || echo "Hello"
non-existing-command: command not found
Hello
``````
##### The | Operator
The | is a pipe operator and allows us to direct an output of a first command into an input of a second command.
let’s use the wc -l command that counts the number of lines in the input:
``````sh
echo "Hello" | wc -l
1
``````
#####

``````sh

``````
