##### If statements (and, closely related, case statements) allow us to make decisions in our Bash scripts
Anything between then and fi (if backwards) will be executed only if the test (between the square brackets) is true.
``````sh
if [ <some test> ]
then
<commands> anything between will be executed!
fi

``````
``````sh
if [ $1 -gt 100 ]
then
echo Hey that\'s a large number.
pwd
fi


if [ $1 -gt 100 ]
then
echo Hey that\'s a large number.
if (( $1 % 2 == 0 ))
then
echo And is also an even number.
fi
fi

``````
##### IF Else
``````sh
if [ <some test> ]
then
<commands> execute command here
else
<other commands> do another command
fi

``````
``````sh
if [ $# -eq 1 ]
then
nl $1
else
nl /dev/stdin
fi

``````
##### If Elif Else
``````sh
if [ <some test> ]
then
<commands> do some exec
elif [ <some test> ] exec
then
<different commands> exec
else
<other commands> exec
fi

``````
``````sh
if [ $1 -ge 18 ]
then
echo You may go to the party.
elif [ $2 == 'yes' ]
then
echo You may go to the party but be back before midnight.
else
echo You may not go to the party.
fi

``````
##### Boolean Operators
``````sh
and - &&  Perform the and operation.
or - ||  Perform the or operation.

``````
``````sh
if [ -r $1 ] && [ -s $1 ]
then
echo This file is useful.
fi

if [ $USER == 'bob' ] || [ $USER == 'andy' ]
then
ls -alh
else
ls
fi

``````
