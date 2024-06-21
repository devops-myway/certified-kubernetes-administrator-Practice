
###### Cat Command
it concatenate and let you create, merge or print files in the standard output screen or to another file and much more

##### Using the cat Command to Create a File
Using the cat command you can quickly create a file and put text into it.
use the > redirect operator to redirect the text in the file.
``````sh
cat –help
cat [OPTION] [FILE]

cat > filename.txt
ls -l

``````
###### Using the cat Command to View the Content of a File
This is one of the most basic usages of the cat command.
``````sh
cat filename.txt
cat filename.txt | more
cat *.txt

``````
###### Using the cat Command to Redirect Content
To append the contents of the destination file, use the >> option along with the cat command:
To redirect the output to another file using the option >

``````sh

cat source.txt > destination.txt
cat source.txt >> destination.txt

``````
###### Using the cat Command to Concatenate Files
concatenate multiple files into a single one.
``````sh
cat source1.txt source2.txt > destination.txt
``````
###### Cat && EOF
as "heredoc", is a special type of redirection in Bash scripting that allows you to pass multiline strings of text directly into a command.
this delimiter is often represented as EOF (stands for "End of File").
The <<EOF  initiates the here document, and the final EOF indicates its end

``````sh
touch demo.sh
vi demo.sh  #write this simple bash script below and grant permissions.

#!/bin/bash
cat <<EOF
Hello world!
This is a heredoc.
EOF

bash demo.sh  # to run the script

``````

`````sh
vi demo.sh

#!/bin/bash

WELCOME="Hello world!"
cat <<EOF
${WELCOME}
This is a heredoc.
Today is $(date).
EOF

bash demo.sh
``````

`````sh


``````
