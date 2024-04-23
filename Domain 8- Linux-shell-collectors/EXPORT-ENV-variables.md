
##### What Are Environment Variables
Environment variables are the variables specific to a certain environment.
Here are some examples of environment variables in Linux:

USER – This points to the currently logged-in user.
HOME – This shows the home directory of the current user.
SHELL – This stores the path of the current user’s shell, such as bash or zsh.
LANG – This variable points to the current language/locales settings.
MAIL – This shows the location of where the current user’s mail is stored.

##### Set Environment Variables in Linux

``````sh
export VARIABLE_NAME=value
export JAVA_HOME=/usr/bin/java
--- # Verify by listing it
env
--- # Print its value:
echo $Key_Name
echo $JAVA_HOME
``````
Creating Environmental Variables: to replace alias

``````sh
export do="--dry-run=client -o yaml"
export now="--grace-period 0 --force"

export NEW_VAR="Testing export"
printenv | grep NEW_VAR

# to use the variable in place of an alias
echo $NEW_VAR
echo $do
$do

----
kubectl delete pod <PODNAME> --grace-period=0 --force --namespace <NAMESPACE>
kubectl delete pod <PODNAME> $now --namespace <NAMESPACE>
``````
