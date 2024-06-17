https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/
##### tar
Short for tape archiver; nothing but a computer file format that combines and compresses multiple files
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

``````
##### How to extract a tar.gz compressed archive on Linux
``````sh
tar -xvf file.tar.gz
tar -xzvf file.tar.gz
tar -xzvf projects.tar.gz

tar -xzvf projects.tar.gz -C /tmp/   #One can extract files in a given directory such as /tmp/:

``````
