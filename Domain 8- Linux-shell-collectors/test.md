#####  Diving Deeper into the ‘test’ Command
The ‘test’ command in Linux is a conditional expression evaluator. It evaluates the condition provided to it and returns a status code, which is zero (0) if the condition is true and non-zero if the condition is false

``````sh
test expression

test 10 -gt 5

``````
#####  The Role of the ‘test’ Command in Scripts
The ‘test’ command checks if the ‘/home/username’ directory exists
``````sh
#!/bin/bash

if test -d /home/username; then
    echo "The directory exists."
else
    echo "The directory does not exist."
fi
--------
#!/bin/bash

for file in /home/username/*; do
    if test -f "$file"; then
        echo "$file is a regular file."
    elif test -d "$file"; then
        echo "$file is a directory."
    else
        echo "$file is not a regular file or directory."
    fi
done
``````
#####  

``````sh

``````
