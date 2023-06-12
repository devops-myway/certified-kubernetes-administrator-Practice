

#####  Aliases

# Needed
alias k='kubectl'

# Optional
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgc='kubectl get componentstatuses'
alias kctx='kubectl config current-context'
alias kcon='kubectl config use-context'
alias kgc='kubectl config get-context'

##### Creating Environmental Variables: to replace alias

``````sh
export do="--dry-run=client -o yaml"
export now="--grace-period 0 --force"
export kgp="kubectl get pods"

export NEW_VAR="Testing export"
printenv | grep NEW_VAR

# to use the variable in place of an alias
echo $NEW_VAR
echo $do
$do

----
kubectl $kgp --namespace <NAMESPACE>
kubectl delete pod <PODNAME> --grace-period=0 --force --namespace <NAMESPACE>
kubectl delete pod <PODNAME> $now --namespace <NAMESPACE>
``````

##### Print or list all environment variables
printenv command – Print all or part of environment.
env command – Display all exported environment or run a program in a modified environment.
set command – List the name and value of each shell variable.

printenv
printenv | less
printenv | more
printenv | grep foo