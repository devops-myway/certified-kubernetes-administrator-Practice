##### What is a Bash for loop
A for loop is classified as an iteration statement i.e. it is the repetition of a process within a bash script.

``````sh
#Numeric ranges for syntax is as follows:

for item in [LIST]
do
  [COMMANDS]
done

or 
for VARIABLE in 1 2 3 4 5 .. N
do
	command1
	command2
	commandN
done

OR

for VARIABLE in file1 file2 file3
do
	command1 on $VARIABLE
	command2
	commandN
done

``````
##### Example 1
``````sh
#!/bin/bash
for i in 1 2 3 4 5
do
   echo "Welcome $i times"
done

#!/bin/bash
for i in {1..5}
do
   echo "Welcome $i times"
done

``````
##### Loop over array elements or Range
You can also use the for loop to iterate over an array of elements.

``````sh
BOOKS=('In Search of Lost Time' 'Don Quixote' 'Ulysses' 'The Great Gatsby')

for book in "${BOOKS[@]}"; do
  echo "Book: $book"
done
----

# {START..END}
for i in {0..3}
do
  echo "Number: $i"
done
----
# {START..END..INCREMENT}
for i in {0..20..5}
do
  echo "Number: $i"
done
``````
##### break Statements

``````sh
for element in Hydrogen Helium Lithium Beryllium; do
  if [[ "$element" == 'Lithium' ]]; then
    break
  fi
  echo "Element: $element"
done

echo 'All Done!'

``````
##### continue Statement
The continue statement exits the current iteration of a loop and passes program control to the next iteration of the loop
``````sh
for i in {1..5}; do
  if [[ "$i" == '2' ]]; then
    continue
  fi
  echo "Number: $i"
done

``````
