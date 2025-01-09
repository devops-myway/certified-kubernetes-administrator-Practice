https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/
##### tar.gz
The tar command on Linux is used to create and extract TAR archive files.

tar -czvf (archive name).tar.gz (pathtofile)

- -c: To create an archive
- -x: Extract an archive
- -v: Enables verbose mode, showing the progress of the command
- -z: Uses gzip, omit this if you just have a .tar
- -f: specifies file input, rather than STDIN or specify output archive name.
- -j: to use .tar.bz2 format

##### Compress an Entire Directory or a Single File
Use the following command to compress an entire directory or a single file on Linux.

``````sh
tar --help

tar -czvf name-of-archive.tar.gz /path/to/directory-or-file

 #you have a directory named "stuff" in the current directory and you want to save it to a file named archive.tar.gz.
tar -czvf archive.tar.gz stuff
tar -czvf archive.tar.gz /usr/local/something

# compress multiple directories
tar -czvf archive.tar.gz /home/ubuntu/Downloads /usr/local/stuff /home/ubuntu/Documents/notes.txt

# using -g format option to convert txt to tar.gz file
tar -czvf sample1.tar.gz random1.txt

# using -f format option
tar -cjf sample.tar.bz2 sample.txt
``````
##### How to create tar.gz file in Linux using command line
``````sh
tar -ztvf projects.tar.gz
ls | grep test
tar -czf test.tar.gz test
ls | grep test

``````
##### Exclude Directories and Files
You can do so by appending an --exclude switch for each directory or file you want to exclude.
``````sh
tar -czvf archive.tar.gz /home/ubuntu --exclude=/home/ubuntu/Downloads --exclude=/home/ubuntu/.cache
tar -czvf archive.tar.gz /home/ubuntu --exclude=*.mp4

``````
##### How to extract a tar.gz or Untar a compressed archive on Linux
``````sh
tar -xzf tarfile
tar -xvf file.tar.gz
tar -xzvf file.tar.gz
tar -xzvf projects.tar.gz

``````
##### Download, extract and move a tar.gz file
``````sh
cd /home/username/public_html/
wget http://wordpress.org/latest.tar.gz
tar xvfz latest.tar.gz
mv /home/username/public_html/wordpress/* /home/username/public_html


``````
