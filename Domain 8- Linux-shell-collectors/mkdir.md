
##### -p, --parents
 I want to create directories hello/goodbye but none exist
 mkdir [-switch] foldername
 -p is a switch, which is optional. It will create a subfolder and a parent folder as well

 ``````sh
$ mkdir hello/goodbye
mkdir:cannot create directory 'hello/goodbye': No such file or directory
$ mkdir -p hello/goodbye
$
---
mkdir -p storage/framework/{sessions,views,cache}

``````