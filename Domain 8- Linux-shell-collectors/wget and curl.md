https://www.warp.dev/terminus/curl-vs-wget

##### Download Files With Curl
- cURL — Client URL — is a command line tool, used to transfer data to or from one server to another.
- It supports any of the below protocols: HTTP, FTP, IMAP, POP3, SCP, SFTP, SMTP, TFTP, TELNET, LDAP
- options:
- -O [downloads the file and saves it with the same name as in the URL]
- k [stands for "insecure" and is used to allow connections to SSL sites without certificates.curl will not verify the SSL/TLS certificate presented by the server, making the connection potentially insecure.
- -s [option in the curl command stands for "silent" or "quiet" mode. it suppresses the progress meter and other informational messages
-  -o  flag (short for --output ): Downloading and renaming files

``````sh
curl --version
Syntax: curl [options] [URL]

curl -O https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz
curl -kO https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz
curl -sO https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz
curl -o log.txt http://example.com/logs/20231003.txt
``````
##### Retrieving data from a server with cURL
- the term “curl” is used as a verb to describe requesting a URL with cURL. you can use cURL to retrieve any URL
- This is because the returned response to the cURL access will be the Google homepage in HTML.
- The HTML source code will be directly displayed in the command-line window without any formatting
``````sh
# Curling the following website
site="www.google.com"
curl "$site"

``````
##### Retrieving data with cURL and saving it locally
- The “-O” option (upper case o, not a zero) tells cURL to use the name of the file at the end of the URL
- -o” option (lower case o) to assign the file a name yourself:
``````sh
# An image from the English version of Wikipedia
file="https://en.wikipedia.org/static/images/project-logos/enwiki-2x.png"
# Retrieving the image and saving it locally under the same name
curl “$file" -O

# But what if the URL does not contain a file name? Test the following code

# Google homepage
homepage="www.google.com"
# Retrieving the homepage with the -O option
curl "$homepage" -O

# Assign a file name with -o

# Google homepage
homepage="www.google.com"
# Name for the file to be created
name="homepage-google.html"
# Retrieving the homepage and saving it locally under the chosen name
curl "$homepage" -o "$name"
``````
##### Using cURL to test whether a server is available
- cURL command checks whether and how a server responds:
``````sh
# Testing whether a web server is available
server="google.com"
curl -I "$server"


``````

##### Downloading in a specific directory
 current working directory, you can use the --output-dir
 --output-dir  flag can also be combined with the -o  flag.
``````sh
curl --output-dir <path> -O <url>
curl --output-dir /tmp/logs -O http://example.com/logs/20231003.txt

``````
##### Wget
- wget , on the other hand, is a simpler, more straightforward tool primarily designed for downloading files and mirroring websites efficiently.
- wget  saves downloaded contents to local files, whereas curl  outputs the content to the terminal by default.
  
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
