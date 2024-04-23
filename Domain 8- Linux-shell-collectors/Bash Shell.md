https://www.cyberciti.biz/faq/bash-while-loop/

###### Introduction to the Bash Shell

A bash script is a series of commands written in a file. These are read and executed by the bash program. File extension of .sh
When a shell is used interactively, it displays a $ when it is waiting for a command from the user.  shell prompt==  [username@host ~]$
If shell is running as root, the prompt is changed to #. The superuser shell prompt looks like this: [root@host ~]#

Scripts start with a bash bang: Shebang is a combination of bash # and bang ! : #! /bin/bash

##### Execution rights
An execution right is represented by x. In the example below, my user has the rwx (read, write, execute) rights for the file test_script.sh

##### Create Your First Bash Script

``````sh
touch hello_world.sh
--
which bash
/usr/bin/bash
---
#! usr/bin/bash
echo "Hello World"   # Quoting “a string”

``````
##### Write the command.

``````sh
#! usr/bin/bash
echo "Hello World"

---- # execution right to a user only
chmod u+x hello_world.sh
chmod 100 hello_world.sh
--- #Run the script
./hello_world.sh
bash hello_world.sh

Using #!/usr/bin/env NAME makes the shell search for the first match of NAME in the $PATH environment variable. It can be useful if you aren't aware of the absolute path or don't want to search for it.
#!/usr/bin/env bash #lends you some flexibility on different systems
#!/usr/bin/bash     #gives you explicit control on a given system of what executable is called
``````
##### Copy a file to the root directory
cp is copy. -p means to preserve the file owner and permissions.
``````sh
sudo cp -p Desktop/FILE /

``````

##### How to define variables
variable by using the syntax variable_name=value. To get the value of the variable, add $ before the variable
``````sh
#!/bin/bash
# A simple variable example
greeting=Hello
name=Tux
echo $greeting $name
``````
##### Export Command
$ export : the command will generate or display all exported variables. Below is an example of the expected output
``````sh
#Viewing all exported variables on current shell
$ export
$ export -p

#You can also assign a value before exporting a function as shown
$ export name[=value]
#For example, you can define a variable before exporting it as shown
$ student=Divya
$ export students
$ printenv students
#The above can be achieved in 2 simple steps by declaring and exporting the variable in one line as shown
$ export student=Divya
$ printenv student
``````
##### Arithmetic Expressions
+	addition
-	subtraction
*	multiplication
/	division
**	exponentiation
%	modulus

Numerical expressions can also be calculated and stored in a variable using the syntax below: var=$((expression))
``````sh
#!/bin/bash
var=$((3+9))
echo $var

``````
##### How to read user input
In bash, we can take user input using the read command.
To prompt the user with a custom message, use the -p flag
read variable_name
read -p "Enter your age" variable_name
``````sh
#!/bin/bash

echo "Enter a numner"
read a

echo "Enter a numner"
read b

var=$((a+b))
echo $var

``````
##### Numeric Comparison logical operators
Comparison is used to check if statements evaluate to true or false

Equality	num1 -eq num2	is num1 equal to num2
Greater than equal to	num1 -ge num2	is num1 greater than equal to num2
Greater than	num1 -gt num2	is num1 greater than num2
Less than equal to	num1 -le num2	is num1 less than equal to num2
Less than	num1 -lt num2	is num1 less than num2
Not Equal to	num1 -ne num2	is num1 not equal to num2
``````sh
if [ conditions ]
    then
         commands
fi
--------
echo -n "Enter Number: "
read x

if [ $((x%2)) == 0 ]; then
  echo "Number is Even"
fi
----
read x
read y

if [ $x -gt $y ]
then
echo X is greater than Y
elif [ $x -lt $y ]
then
echo X is less than Y
elif [ $x -eq $y ]
then
echo X is equal to Y
fi
``````
##### Conditional Statements (Decision Making)
The structure of conditional statements is as follows:

if...then...fi statements
if...then...else...fi statements
if..elif..else..fi
if..then..else..if..then..fi..fi.. (Nested Conditionals)The structure of conditional statements is as follows:

``````sh
if [[ condition ]]
then
	statement
elif [[ condition ]]; then
	statement 
else
	do this by default
fi
--------
read a
read b
read c

if [ $a == $b -a $b == $c -a $a == $c ]
then
echo EQUILATERAL

elif [ $a == $b -o $b == $c -o $a == $c ]
then 
echo ISOSCELES
else
echo SCALENE

fi

``````
##### Looping with numbers:
the loop will iterate 5 times.
``````sh
#!/bin/bash

for i in {1..5}
do
    echo $i
done

``````
##### Looping with strings:
We can loop through strings as well.
``````sh
#!/bin/bash

for X in cyan magenta yellow  
do
	echo $X
done

``````
##### While loop
While loops check for a condition and loop until the condition remains true.
(( i += 1 )) is the counter statement that increments the value of i


``````sh
while [ condition ]; do commands; done
while control-command; do COMMANDS; done

while [ condition ]
do
   command1
   command2
   command3
done

---
#!/bin/bash
x=1
while [ $x -le 5 ]
do
  echo "Welcome $x times"
  x=$(( $x + 1 ))
done
---
x=1; while  [ $x -le 5 ]; do echo "Welcome $x times" $(( x++ )); done

``````
