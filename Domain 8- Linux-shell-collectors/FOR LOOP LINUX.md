#### Bash Arrays
If the index number is @ or *, all members of an array are referenced
The difference is subtle; "${LIST[*]}" (like "$*") creates one argument, 
while "${LIST[@]}" (like "$@") will expand each item into separate arguments, so:

Using [@] will create a list.
``````sh
for variable in list;
do
    # Commands to execute for each item in the list
done

LIST=(1 2 3)
for i in "${LIST[@]}"; do
  echo "test.$i"
done


LIST=(1 2 3)
for i in "${LIST[*]}"; do
    echo "test.$i"
done

#!/bin/bash
for file in /path/to/files/*; do
    echo "Processing file: $file"
    # Add your custom commands here
done

``````

``````sh
for filename in file1.txt file2.txt file3.txt;
do
    if [ "$filename" == "file2.txt" ]; then
        echo "Found file2.txt! Exiting loop."
        break
    fi
    echo "Processing $filename"
done


#!/bin/bash
for i in {1..5}; do
    echo "Number: $i"
done
``````
