##### Using &
The & character is used to run a command in the background, allowing the user to move onto other tasks while the command runs to completion.

``````sh
bigjob &

``````
##### Using && and ||
The && characters will ensure that the command on the right of it will be run if the command on the left of it succeeds and ensures that the second command isn’t run if the first command fails.

``````sh
$ ping 192.168.0.1 && echo router is reachable
router is reachable

``````
##### Using ||
<the || characters have the opposite effect. If the first command is successful, the second will not be run
``````sh
$ [ -d scripts ] || mkdir scripts
$ ls -ld scripts
drwxrwxr-x 2 shs shs 4096 Jul  8 12:24 scripts

``````
