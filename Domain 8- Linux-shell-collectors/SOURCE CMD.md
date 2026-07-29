
##### Source Command Syntax
source is a shell builtin that reads and executes commands from a file within the current shell process. This is different from running a script normally (e.g., ./script.sh), which spawns a subshell — any variables or aliases set there vanish when it exits.

#### What ~/.bashrc Is
~/.bashrc is a per-user configuration file for interactive, non-login Bash shells. It typically contains:
- Aliases (alias ll='ls -la')
- Functions
- Environment variables
- Shell options (shopt)
- Prompt customization (PS1)
- Path modifications

``````sh
source ~/.bashrc
# or the shorthand:
. ~/.bashrc

alias ll = 'ls -l'
sudo vim ~/.bashrc
alias ll = 'ls -l'

source ~/.bashrc

``````
##### Refresh the Current Shell Environment
- Bash opens ~/.bashrc
- Reads it line by line
- Executes each command in the current shell
- Variables, aliases, and functions persist after completion
``````sh


``````
