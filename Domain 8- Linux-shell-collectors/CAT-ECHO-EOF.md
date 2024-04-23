

##### Cat EOF in Bash Script

It can display a file, concatenate (combine) multiple files, takes standard input, echo text, and it can be used to create a new file.
cat<<eof uses “here-document”. A here-document is a code block in Linux. It passes a form of Input/Output redirection to commands like cat.
cat<<EOF” and “EOF” commands which tell the compiler to print the content from the “cat<<EOF” command at the end until “EOF” is reached
use <<- (followed by a dash) to disable leading tabs

Here's an example which will write the contents to a file at /tmp/yourfilehere
``````sh
echo "Dance, Dance" > cat_create #create a file
cat cat_create

#(Con)cat
echo "This is how we do it" > test1 #create 1st file
echo "*This is how we do it*" > test2 #create 2nd file
cat test1 test2

#Creating a File
echo "File 1 Contents" > file1
echo "File 2 Contents" > file2
echo "File 3 Contents" > file3
cat file1 file2 file3 > combined_file
cat combined_file

``````

``````sh
mkdir /tmp/yourfilehere

cat << EOF > /tmp/yourfilehere
These contents will be written to the file.
        This line is indented.
EOF
----
#!/usr/bin/env bash

if true ; then
    cat <<- EOF > /tmp/yourfilehere
    The leading tab is ignored.
    EOF
fi

``````
##### Multi-Line Bash Script
The cat <<EOF syntax is very useful when working with multi-line text in Bash, when assigning multi-line string to a shell variable, file or a pipe.

When root permissions are required for the destination file, use | sudo tee instead of >
``````sh

cat << 'EOF' | sudo tee /tmp/yourprotectedfilehere
The variable $FOO will *not* be interpreted.
EOF
----
#The print.sh file now contains:
cat <<EOF > print.sh
#!/bin/bash
echo \$PWD
echo $PWD
EOF
---
# The b.txt file contains bar and baz lines.
cat <<EOF | grep 'b' | tee b.txt
foo
bar
baz
EOF

``````
##### Multi-Line string for Deployments
``````sh

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args:
    - sleep
    - "1000000"
EOF

``````
###### Echo command
display a line of text, Echo the STRING(s) to standard output.
The output of the `echo` can be redirected to a file instead of displaying it on the terminal.
We can achive this by using the `>` or `>>` operators for output redirection.
If we want to append the output in an existing file, we use `>>` instead of `>`.
``````sh
#You can use echo to read the file.txt( not to redirect ) as follows:
echo [string]
echo "$(<file.txt)"
echo "Geeks for Geeks"
echo "Welcome GFG" > output.txt

#You can also pipe text to cat, in which case that text is echoed
echo "Piping fun" | cat
``````

