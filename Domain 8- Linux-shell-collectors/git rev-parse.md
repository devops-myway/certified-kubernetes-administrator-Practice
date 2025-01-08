#### Understanding Git Rev-Parse

- 'git rev-parse’ is a versatile command used to parse and query Git references
- The command git rev-parse can do a remarkable number of different things, so you'd need to go through 
- Git references can be branch names, commit IDs, tags, and more.
- The command takes one or more arguments and returns information about the references or objects specified

##### Retrieving the Commit ID
- One of the most straightforward uses of ‘git rev-parse’ is to obtain the commit ID of a given reference.
- Running this command will return the full SHA-1 hash of the latest commit in the current branch
- You can replace ‘HEAD’ with the name of any branch or tag to get the respective commit ID
``````sh
git rev-parse HEAD

# short commit hash
git rev-parse --short HEAD
git rev-parse --short=8 HEAD

# Finding the Branch Name: This command will return the name of the current branch
git rev-parse --abbrev-ref HEAD

# Resolving Abbreviated Commit IDs: You can use ‘git rev-parse’ to resolve these abbreviations to their full SHA-1 hashes
git rev-parse 34e45d
``````
#### 

``````sh

``````
#### 

``````sh

``````
