https://phoenixnap.com/kb/guide-linux-find-command

##### find
- The find command in Linux is a powerful tool used to search for files and directories within a specified path based on different criteria.
- it allows users to locate files by name, type, size, permissions, and more, making the tool essential for file management and system administration.

- find Command Options
- -name:	Searches for files or directories by name (case-sensitive).
- -prune<br>:	Follows symbolic links. The command operates on the files or directories that the links point to rather than the links themselves.
- -not:	Negates a condition.
- -empty:	Searches for empty files and directories.
- -mindepth	Starts searching only after a certain directory depth.
- -maxdepth<br>:	Limits the search to a specific depth.
- -print:	Displays the full file path of each match. This is the default action if no other action is specified.
- -exec:	Executes a command on each file found
- -perm:	Searches for files with specific permissions.
- -group:	Finds files owned by a specific group.
- -user<br>:	Delivers files owned by a specific user.
- -ctime:	Searches for files with their status changed within a certain number of days.
- -atime:	Delivers files accessed within a certain number of days.
- -mtime:	Searches for files modified within a certain number of days.
-size	Finds files of a specific size.
-type<br>:	Searches for files of a specific type (f for files, d for directories, l for symbolic links).
-iname:	Finds files or directories by name (case-insensitive).
-L:	Follows symbolic links. The command operates on the files or directories that the links point to, rather than the links themselves.
-H:	Follows symbolic links in the command itself but not the links encountered while searching.
-P:	Default behavior. Does not follow symbolic links. The search does not resolve or act on the files or directories the links point to.

##### Find Files by Name

``````sh
Syntax:
find [options] [path] [expression]

# This command searches the current user's directory (~) for any file named test1.txt
find ~ -name "test1.txt"
find ~ -name "test*.txt"
#find all .txt files within the /Documents directory with:
find /home/sara/Documents -name "*.txt"
``````
##### Find Files by Location

``````sh
# search for all files and directories across the entire hard drive
sudo find /
sudo find / | less
# To search the directory you're currently in, use a period (.).
sudo find . |less

#Use the tilde (~) character to search the current user's home directory:
sudo find ~ |less

#Search all user directories under /home for a file named test1.txt with
sudo find /home -name "test1.txt"

``````
