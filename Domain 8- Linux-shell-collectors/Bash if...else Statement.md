###### if Statement
Bash if conditionals can have different forms. The most basic if statement takes the following form
The if statement starts with the if keyword followed by the conditional expression and the then keyword. The statement ends with the fi keyword.

If the TEST-COMMAND evaluates to True, the STATEMENTS are executed. If the TEST-COMMAND returns False, nothing happens; the STATEMENTS are ignored.

``````sh
if TEST-COMMAND
then
  STATEMENTS
fi

``````
###### If Example
Let’s look at the following example script that checks whether a given number is greater than 10:
``````sh
vi test.sh

#!/bin/bash

echo -n "Enter a number: "
read VAR

if [[ $VAR -gt 10 ]]
then
  echo "The variable is greater than 10."
fi

bash test.sh
``````
###### if...else Statement
The Bash if...else statement takes the following form
TEST-COMMAND evaluates to True, the STATEMENTS1 will be executed. Otherwise, if TEST-COMMAND returns False, 
the STATEMENTS2 will be executed. You can have only one else clause in the statement.

``````sh
if TEST-COMMAND
then
  STATEMENTS1
else
  STATEMENTS2
fi

``````
###### Example If...else
The while loop is used to performs a given set of commands an unknown number of times as long as the given condition evaluates to true.

``````sh
#!/bin/bash

echo -n "Enter a number: "
read VAR

if [[ $VAR -gt 10 ]]
then
  echo "The variable is greater than 10."
else
  echo "The variable is equal or less than 10."
fi

``````
##### if...elif...else Statement
The Bash if...elif...else statement takes the following form
``````sh
if TEST-COMMAND1
then
  STATEMENTS1
elif TEST-COMMAND2
then
  STATEMENTS2
else
  STATEMENTS3
fi

``````
##### Example of if...else
``````sh
#!/bin/bash

echo -n "Enter a number: "
read VAR

if [[ $VAR -gt 10 ]]
then
  echo "The variable is greater than 10."
elif [[ $VAR -eq 10 ]]
then
  echo "The variable is equal to 10."
else
  echo "The variable is less than 10."
fi

``````
##### Nested if Statements

``````sh
#!/bin/bash

echo -n "Enter the first number: "
read VAR1
echo -n "Enter the second number: "
read VAR2
echo -n "Enter the third number: "
read VAR3

if [[ $VAR1 -ge $VAR2 ]]
then
  if [[ $VAR1 -ge $VAR3 ]]
  then
    echo "$VAR1 is the largest number."
  else
    echo "$VAR3 is the largest number."
  fi
else
  if [[ $VAR2 -ge $VAR3 ]]
  then
    echo "$VAR2 is the largest number."
  else
    echo "$VAR3 is the largest number."
  fi
fi

``````
##### Multiple Conditions
Instead of the nested if statements, we’re using the logical AND (&&) operator.

``````sh
#!/bin/bash

echo -n "Enter the first number: "
read VAR1
echo -n "Enter the second number: "
read VAR2
echo -n "Enter the third number: "
read VAR3

if [[ $VAR1 -ge $VAR2 ]] && [[ $VAR1 -ge $VAR3 ]]
then
  echo "$VAR1 is the largest number."
elif [[ $VAR2 -ge $VAR1 ]] && [[ $VAR2 -ge $VAR3 ]]
then
  echo "$VAR2 is the largest number."
else
  echo "$VAR3 is the largest number."
fi

``````
``````sh


``````
