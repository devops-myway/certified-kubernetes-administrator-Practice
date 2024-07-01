##### Wildcards
*  : Matches any characters
?  : Matches any single character
[characters]  : matches any character that is a member of the set characters
[!characters]  : Matches any character that is not a member of the set characters
[[:class:]]  : Matches any character that is a memeber of the specified class 

``````sh
cp * /target/directory   # copy all files in the current directory to the /target/directory directory
# move all files in the current directory that have names that begin with a lowercase letter and end with the “.jpg” extension to the /target/directory directory
mv [a-z]*.jpg /target/directory  
ls *.txt   #where
*         #matches any character and
*.txt     #means any file that ends with .txt.
ls a*[!abc].txt   #(e.g. “a1.txt” or “ax.txt”)
ls a[a-z]*b?c.txt   # (e.g. “aabbc.txt” or “aaxb1c.txt”)
``````
##### Match any single character as wildcard using
 ? character is a wildcard that is used to match any single character in a file name or path.
 
``````sh
ls ???.txt   #(e.g. “abc.txt” or “123.txt”)
grep 'ERROR' log-file?.log  # log-file1.log” or “log-fileA.log
rm file??.txt              # (e.g. “file01.txt” or “fileXX.txt”)

``````
##### 

``````sh


``````
