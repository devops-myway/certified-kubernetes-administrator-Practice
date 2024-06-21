https://www.warp.dev/terminus/curl-vs-wget

##### Download Files With curl And wget
curl  is a flexible, feature-rich, and multi-purpose tool designed for user authentication, proxy support, cookie handling, file uploading, and more.
wget , on the other hand, is a simpler, more straightforward tool primarily designed for downloading files and mirroring websites efficiently.

wget  saves downloaded contents to local files, whereas curl  outputs the content to the terminal by default.
curl  supports a wide range of protocols, such as HTTP, HTTPS, FTP, FTPS, SCP, SFTP, TFTP, etc, whereas wget  only supports HTTP, HTTPS, and FTP.

``````sh
# -O  flag (short for --remote-name )

curl -O <url>
curl -O http://example.com/logs/20231003.txt

``````

##### Downloading and renaming files
you can use the -o  flag (short for --output )
``````sh
curl -o <filename> <url>
curl -o log.txt http://example.com/logs/20231003.txt

``````
##### Downloading in a specific directory
 current working directory, you can use the --output-dir
 --output-dir  flag can also be combined with the -o  flag.
``````sh
curl --output-dir <path> -O <url>
curl --output-dir /tmp/logs -O http://example.com/logs/20231003.txt

``````
##### Downloading a single file with with wget
``````sh
wget <url>

wget http://example.com/logs/20231003.txt

``````
##### Downloading and renaming files
you can use the wget  command with the -O  flag (short for --output-document
``````sh
wget -O <filename> <url>
wget -O log.txt http://example.com/logs/20231003.txt

``````
##### Downloading in a specific directory
you can use the -P  flag (short for --directory-prefix

``````sh
wget -P <path> <url>
wget -P /tmp/logs http://example.com/logs/20231003.txt

``````
##### Downloading multiple files with
curl 
To download multiple files at once with curl , you can repeat the -o , -O , and --output-dir  flags several times, for each URL:

``````sh
$ curl \
-O http://example.com/logs/20231003.txt \
-O http://example.com/logs/20231004.txt

curl \
-o logs_03.txt http://example.com/logs/20231003.txt \
-o logs_04.txt http://example.com/logs/20231004.txt
``````
