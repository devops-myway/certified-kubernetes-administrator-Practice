##### Know your Linux Directories

``````sh
~ means the home directory of the logged on user whereas, The tilde (~) is a Linux "shortcut" to denote a user's home directory.
~/ means the path to the beginning of a directory
.bashrc` file is typically located in the user’s home directory (~/.bashrc)
cd ~
vi ~/.bashrc and rc means read command.
alias abc='ls -l'
#Reload the .bashrc File
source ~/.bashrc

``````

##### Variables in Linux
Variables are declared and assigned without $ and without {}. to assign. In order to read from the variable
(in other words, expand the variable), you must use $.

``````sh
var=10

$var      # use the variable
${var}    # same as above
${var}bar # expand var, and append "bar" too or concatenate.
$varbar  ## same as ${varbar}, i.e expand a variable called varbar, if it exists.

dir=(*)           # store the contents of the directory into an array
echo "${dir[0]}"  # get the first entry.
echo "$dir[0]"    # incorrect

${var.variable_name}  # in terraform it is same style.

``````
