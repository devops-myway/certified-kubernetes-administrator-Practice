https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/
##### tar.gz
Tar files are often compressed after being created, giving it the .tar.gz file extension
Short for tape archiver; nothing but a computer file format that combines and compresses multiple files.
Creating a tar file is just as easy. Just replace the -x with a -c to "Create," though I find it easier to remember by "Compress," even though that's -z's job.

-c: To "Create
-x: Extract
-v: Enables verbose mode, showing the progress of the command
-z: Uses gzip, omit this if you just have a .tar
-f: specifies file input, rather than STDIN

A tarball (or tar.gz or tar.bz2)
``````sh
tar --help

tar -czvf file.tar.gz directory
tar -czvf filename.tar.gz /path/to/dir1
tar -czvf filename.tar.gz /path/to/dir1 dir2 file1 file2

``````
##### How to create tar.gz file in Linux using command line
``````sh
tar -ztvf projects.tar.gz
ls | grep test
tar -czf test.tar.gz test
ls | grep test

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
