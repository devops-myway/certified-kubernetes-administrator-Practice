
##### What Are Environment Variables
Environment variables are the variables specific to a certain environment.
Here are some examples of environment variables in Linux:

USER – This points to the currently logged-in user.
HOME – This shows the home directory of the current user.
SHELL – This stores the path of the current user’s shell, such as bash or zsh.
LANG – This variable points to the current language/locales settings.
MAIL – This shows the location of where the current user’s mail is stored.

##### Use Alias Command in Linux
Linux users often need to use one command over and over again. Aliases are like custom shortcuts that represent a command (or set of commands) that can be executed with or without custom options.

- What you need to do is type the word alias then use the name you wish to use to execute a command followed by "=" sign and quote the command you wish to alias.
``````sh
$ alias

alias shortName="your custom command here"
alias wr=”cd /var/www/html”  # You can then use "wr" shortcut to go to the webroot directory

``````

##### Creating Permanent Aliases in Linux
To keep aliases between sessions, you can save them in your user’s shell configuration profile file. This can be:
- Bash – ~/.bashrc
- ZSH – ~/.zshrc
- Fish – ~/.config/fish/config.fish
``````sh
ls -l -A  # to view all hidden files
cat ~/.bashrc  # to save alias files
vim ~/.bashrc
alias home=”ssh -i ~/.ssh/mykep.pem tecmint@192.168.0.100”
alias ll="ls -alF"

source ~/.bashrc  #If you want to use the newly defined alias in the current session, issue the following command:

 unalias alias_name  # To remove alias
``````

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
