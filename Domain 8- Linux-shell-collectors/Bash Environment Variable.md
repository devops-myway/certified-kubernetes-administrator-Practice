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
##### The Difference between Login, Non-Login, Interactive, and Non-Interactive Shell Sessions
One distinction between different sessions is whether the shell is being spawned as a login or non-login session.

A login shell is a shell session that begins by authenticating the user. If you are signing into a terminal session or through SSH and authenticate, your shell session will be set as a login shell.
 session started as a login session will read configuration details from the /etc/profile file first.
 It reads the first file that it can find out of ~/.bash_profile, ~/.bash_login, and ~/.profile and does not read any further files.
 
``````sh


``````
