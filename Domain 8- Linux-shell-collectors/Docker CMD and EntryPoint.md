#####  CMD directive
To define an executable with its arguments that the container should run. In this case, we can omit the ENTRYPOINT directive.

##### The Exec Form of the CMD Directive
The exec form invokes only the defined executable, without a shell
The exec form is parsed as a JSON array, which means that you must use double-quotes (") around words not single-quotes (').
``````sh
CMD "executable","param1","param2"
CMD "param1","param2"
CMD ["executable", "param1", "param2"]

``````
##### The Shell Form of the CMD Directive
Unless we set a shell, like /bin/sh or /bin/bash, to be the executable, the container doesn’t start a shell. 
``````sh
CMD command param1 param2 (shell form)
/bin/sh -c executable param1 param2

``````
#####  Run Multiple Commands With Dockerfile
The sh -c command accepts a list of commands and executes them. Moreover, the commands can be separated with the known shell operators:

- the semicolon (;) operator
- the ambersand (&) operator
- the AND (&&) operator
- the OR (||) operator
- the pipe (|)

###### Run Multiple Commands With the Shell Form

``````sh
FROM ubuntu:latest
CMD echo FIRST COMMAND;echo SECOND COMMAND
---
sudo docker build -t example1 .
sudo docker run -it example1
FIRST COMMAND
SECOND COMMAND
``````
#####  Run Multiple Commands With the Exec Form
The exec form doesn’t start a shell on its own, we can set the /bin/sh -c or the /bin/bash -c commands in the CMD executable position, and invoke the shell

``````sh
FROM ubuntu:latest
CMD ["/bin/bash", "-c", "echo FIRST COMMAND;echo SECOND COMMAND"]
---
sudo docker build -t example2 .
sudo docker run -it example2
FIRST COMMAND
SECOND COMMAND
``````
#####  Run Multiple Commands Without Shell Invocation

``````sh
sudo docker run -it example2  echo -e "\ttest1";echo test2
        test1
test2
---

$ sudo docker run -it example2  /bin/bash -c "echo -e '\ttest1';echo \$HOME"
        test1
/root
``````
#####  Run Multiple Commands With a Shell Script
we can create a shell script that contains all the necessary logic, and copy it to the container’s filesystem
We can use the Dockerfile COPY directive to copy the shell script file to our container’s filesystem.
``````sh
vi startup.sh

! /bin/bash
echo $HOME
date
--- 
vi Dockerfile

FROM ubuntu:latest
COPY startup.sh .
CMD ["/bin/bash","-c","./startup.sh"]
----
docker build -t example3 .
docker run -it example3

``````
#####  Example 3- Now CMD using the Exec form:

``````sh
vi Dockerfile

FROM alpine:3.8
CMD ["/bin/echo","hello from CMD - using exec form"]
--

docker build --tag tutorial:demo --file Dockerfile  .
docker run --name tutorial tutorial:demo

# expected Output
hello from CMD - using exec form
``````
#####  Now CMD using the shell form:

``````sh
FROM alpine:3.8
CMD /bin/echo "hello from CMD using shell form"

---
docker build --tag tutorial:demo --file Dockerfile  .
docker stop -t 0 tutorial ; docker container prune -f;docker ps -a
docker run --name tutorial tutorial:demo

---
# Expected Output
hello from CMD using shell form
``````
#####  

``````sh
FROM alpine:3.8
CMD /bin/echo $HOME

docker build --tag tutorial:demo --file Dockerfile  .
docker stop -t 0 tutorial
docker container prune -f
docker ps -a
docker run --name tutorial tutorial:demo

--
Expected Output
/root
``````
