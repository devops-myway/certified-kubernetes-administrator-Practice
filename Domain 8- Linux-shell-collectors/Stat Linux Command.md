
##### What is the ‘stat’ command in Linux
stat' command in Linux is a powerful tool used to display detailed information about a file or file system.
-c	Specifies a format like ‘ls -l’.	stat -c '%A %h %U %G %s %y %n' myfile.txt
-f	Displays file system status instead of file status.	stat -f /home
-L	Follows links.	stat -L symlink
-t	Displays info in terse form.	

``````sh
stat [options] [file.txt or /path/to/directory]

stat report.pdf
ls -l report.pdf

``````
##### Using the ‘-c’ Flag with the Stat Command
%A: Access rights in human-readable form
%h: Number of hard links
%U: User name of owner
%G: Group name of owner
%s: Total size, in bytes
%y: Time of last modification, human-readable
%n: File name
``````sh
stat -c '%A %h %U %G %s %y %n' myfile.txt

``````
##### Dealing with Permission Errors

``````sh
stat /root/example.txt

stat non_existent_file.txt

``````
##### Understanding File Permissions in Linux
Read (r): Users can read the contents of the file or list the contents of the directory.
Write (w): Users can write or modify the file or directory.
Execute (x): Users can run the file or lookup a specific file within a directory.
``````sh
ls -l report.pdf

# Check the current permissions of a file
stat -c '%A' myfile.txt

# Output:
# -rw-rw-r--

# Change the permissions of the file
chmod 755 myfile.txt

# Check the permissions again
stat -c '%A' myfile.txt

# Output:
# -rwxr-xr-x

``````
