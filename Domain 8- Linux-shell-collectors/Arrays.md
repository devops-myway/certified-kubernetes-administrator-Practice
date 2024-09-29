https://www.makeuseof.com/bash-script-array-usage/

##### Arrays
There are two types of bash arrays:

- Indexed – the array is referred via integers or numbers starting from 0.
- Associative – the array is referred via strings or a set of characters and words. Store elements in key-value pair

#####  standard bash array 
You’ll need to prepend the dollar sign ($) to the variable name
You’ll also need to use braces ({}) to make the variable name unambiguous
By default, bash will treat $city[2] as a variable named city. Add braces to tell bash to evaluate the brackets and index number too.
``````sh
city=(London Paris Milan "New York")
city[2]

${variable_name[index]}

#!/bin/bash
city=(London Paris Milan "New York")
echo ${city[3]}

# New York

#!/bin/bash
city[0]=London
city[1]=Paris
city[2]=Milan
city[3]="New York"
echo ${city[3]}

``````
#####  More Uses for Bash Arrays
Arrays are perfect for storing related data
``````sh
#!/bin/bash
    
sqrt[1]=1
sqrt[4]=2
sqrt[9]=3
sqrt[16]=4
sqrt[25]=5
    
echo ${sqrt[$1]}

``````

##### Accessing Array Elements Individually

``````sh
assoc_array[element1]="Hello World"
echo ${assoc_array[element1]}

index_array=(1 2 3 4 5 6)
echo ${index_array[0]}

``````
##### Reading Array Elements Sequentially
noticed the index_array[@] and if you’re wondering what the @ symbol is for

``````sh
#!/bin/bash
index_array=(1 2 3 4 5 6 7 8 9 0)

for i in ${index_array[@]}
do
        echo $i
done

``````
#####  Access All Elements of an Array
[@] symbol: But if we want to print all the elements at the same time or work with all the elements
- "#" symbol:  which can be prefixed to an array name to provide us the count of the elements stored in the array
``````sh
echo ${assoc_array[@]}
echo ${#index_array[@]}

``````
#####  

``````sh

``````
