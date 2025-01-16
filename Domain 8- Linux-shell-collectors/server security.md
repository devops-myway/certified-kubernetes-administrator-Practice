##### Change the SSH ports
- For server access via SSH, port 22 is provided by default
- By simply picking a different port for encrypted remote connections, you minimize the risk of unwanted access
- all you have to do is open the SSH configuration file sshd_config with the text editor of your choice
- Look for the appropriate line and replace port number 22 with a number of choice
``````sh
vi /etc/ssh/sshd_config

``````
##### Disable the SSH login for Root users
- To secure your server, it is also recommended to disable the direct SSH log-in for the root account
- If not, an attacker who possesses the password could use the remote account to remotely access the server, and block the SSH direct log-in for the root account.
- you must have at least one additional user account that can be connected to the server, in addition to the root user.
- With this, you can create the user account user1, which is also added to the users group (-g).
- As well as that, the home directory /home/user1 is assigned to this user profile (-d). The command specifies the bash shell as the default shell (-s /bin/bash)
- 
``````sh
useradd -g users -d /home/nutzer1 -m -s /bin/bash user1

# define a secure password for this new log-in account:
passwd user1
``````
##### E-mail notification for SSH log-ins
- monitoring these activities means that you are warned at an early stage by a good monitoring of the established
- connections in case of unauthorized access and can take appropriate countermeasures

``````sh
#!/bin/bash
echo "Log in to $(hostname) on $(date +%Y-%m-%d) at $(date +%H:%M)"
echo "User: $USER"
echo
finger

# Then add the following line to the /etc/profile profile file:
/opt/shell-login.sh | mailx -s "SSH-Log-in to YOUR-HOSTNAME" mailadresse@example.com

``````
``````sh


``````
