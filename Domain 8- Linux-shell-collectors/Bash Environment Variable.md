##### How To Read and Set Environmental and Shell Variables on Linux
When interacting with your server through a shell session, there are many pieces of information that your shell compiles to determine its behavior and access to resources. Some of these settings are contained within configuration settings and others are determined by user input.
 
One way that the shell keeps track of all of these settings and details is through an area it maintains called the environment. The environment is an area that the shell builds every time that it starts a session that contains variables that define system properties.

##### How the Environment and Environmental Variables Work

The keys in these scenarios are variables. They can be one of two types, environmental variables or shell variables.
Environmental variables:
are variables that are defined for the current shell and are inherited by any child shells or processes.
Environmental variables are used to pass information into processes that are spawned from the shell.

Shell variables:
are variables that are contained exclusively within the shell in which they were set or defined.
They are often used to keep track of ephemeral data, like the current working directory.

``````sh
KEY=value1:value2:...

KEY="value with spaces"

``````
##### Printing Shell and Environmental Variables
We can see a list of all of our environmental variables by using the env or printenv commands.
``````sh
env
env | grep any_env_variable
env | grep PATH

printenv
printenv PATH

``````
##### Creating Shell Variables
we only need to specify a name and a value

``````sh
TEST_VAR='Hello World!'
set | grep TEST_VAR
printenv | grep TEST_VAR
env | grep TEST_VAR
echo $TEST_VAR

``````
##### How do we identify a Bash script
``````sh
export NEW_VAR="Testing export"
printenv | grep NEW_VAR
echo $NEW_VAR

``````
##### Environment Variable
let’s use grep to display $PATH from the env command:
#!/usr/bin/env bash is also a shebang line used in script files to execute commands with the Bash shell.
It uses the env command to display the environment variables present in the system and then execute commands with the defined interpreter.

``````sh
which env
env
env | grep PATH

``````
##### test Sample Bash
sample.sh
Running script files through /usr/bin/env has the advantage of automatically searching for the default version of the interpreter in the current environment. 

#!/usr/bin/bash: Offers more security	
#!/usr/bin/env bash: Offers more portability

it’s recommended to use #!/usr/bin/bash when security is a priority and #!/usr/bin/env bash if we’re looking for portability.

``````sh
!/usr/bin/env bash
./sample.sh
Baeldung is Awesome!


``````
``````sh


``````
``````sh


``````
