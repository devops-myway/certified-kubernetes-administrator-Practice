
###### How Do I Use a While Loop in Bash Scripting
The structure of a while loop is simple: It begins with a condition, and as long as that condition is true, the loop will continue to execute the commands inside it.

``````sh
while IFS= read -r line
 do
    echo "$line"
done < myfile.txt

while true; do
    echo 'Infinite loop!'
done

``````
###### Nested While Loops
A nested while loop is a loop within another loop
``````sh
i=1
while [ $i -le 5 ]; do
    j=1
    while [ $j -le 5 ]; do
        echo -n "$((i*j)) "
        j=$((j+1))
    done
    echo
    i=$((i+1))
done
----
while IFS= read -r line; do
    if [[ $line == *'bash'* ]]; then
        echo "$line"
    fi
done < myfile.txt
----
i='hello'
while [ $i -le 5 ]; do
    echo $i
    i=$((i+1))
done
--
i=1
while [ $i -le 5 ]; do
    if [ $i -eq 3 ]; then
        break
    fi
    echo $i
    i=$((i+1))
done

``````
###### For Loops in Bash Scripting
A ‘for’ loop is a control flow statement that allows code to be executed repeatedly.

``````sh

for i in {1..5}; do
    echo $i
done
---


``````
###### Understanding Loops in Programming
In Bash scripting, conditions are expressed using test constructs like [ expression ] or [[ expression ]]
``````sh
while condition is true
    execute loop body
end loop

``````
##### Exploring Related Concepts

``````sh
fibonacci() {
    a=0
    b=1
    while [ $a -le $1 ]; do
        echo $a
        sum=$((a+b))
        a=$b
        b=$sum
    done
}

fibonacci 100
``````
``````sh


``````
