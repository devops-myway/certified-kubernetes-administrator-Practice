
###### How Do I Use a While Loop in Bash Scripting
The while loop is used to performs a given set of commands an unknown number of times as long as the given condition evaluates to true.

``````sh
while [CONDITION]
do
  [COMMANDS]
done

``````
###### Iteration
on each iteration, the current value of the variable i is printed and incremented by one.
``````sh
i=0

while [ $i -le 2 ]
do
  echo Number: $i
  ((i++))
done

``````
###### Infinite while Loop
An infinite loop is a loop that repeats indefinitely and never terminates
we are using the built-in command : to create an infinite loop. : always returns true.
You can terminate the loop by pressing CTRL+C
``````sh

while :
do
  echo "Press <CTRL+C> to exit."
  sleep 1
done

# single equivalent
while :; do echo 'Press <CTRL+C> to exit.'; sleep 1; done

``````
###### Read a File Line By Line 
One of the most common usages of the while loop is to read a file, data stream, or variable line by line.
Here is an example that reads the /etc/passwd file line by line and prints each line.
 always use read with the -r option to prevent backslash from acting as an escape character.
we are using input redirection (< "$file") to pass a file to the read command.
Use the IFS= option before read to prevent this behavior:
``````sh
file=/etc/passwd

while read -r line; do
  echo $line
done < "$file"
----
file=/etc/passwd

while IFS= read -r line; do
  echo $line
done < "$file"

``````
##### break Statement
The break statement terminates the current loop and passes program control to the command that follows the terminated loop.

``````sh
i=0
while [ $i -lt 5 ]
do
  echo "Number: $i"
  ((i++))
  if [[ "$i" == '2' ]]; then
    break
  fi
done

echo 'All Done!'

``````
##### continue Statement
The continue statement exits the current iteration of a loop and passes program control to the next iteration of the loop.
``````sh
i=0

while [ $i -lt 5 ]
do
  ((i++))
  if [[ "$i" == '2' ]]; then
    continue
  fi
  echo "Number: $i"
done

echo 'All Done!'

``````
