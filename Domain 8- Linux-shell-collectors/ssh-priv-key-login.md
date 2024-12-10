#### Note
https://unix.stackexchange.com/questions/23291/how-to-ssh-to-remote-server-using-a-private-key

SSH (Secure Shell) is a secure protocol provides a safe and secure way of executing commands, making changes, and configuring services remotely.

#### How To Log Into SSH with Keys

it is faster and more secure to set up key-based authentication.
Key-based authentication works by creating a pair of keys: a private key and a public key.
- The private key is located on the client’s machine and is secured and kept secret.
- The public key can be given to anyone or placed on any server or remote host you wish to access.

The client computer then sends the appropriate response back to the server, which will tell the server that the client is legitimate.

# How To Create SSH Keys
Your keys will be created at ~/.ssh/id_rsa.pub and ~/.ssh/id_rsa.
Change into the .ssh directory by typing:
``````sh
ssh-keygen -t rsa
ssh-keygen -t rsa -b 4096 -C "major.sampson@beqom.com"
cd ~/.ssh
ls -l

``````
##### Generate an SSH Key Pair with a Larger Number of Bits
SSH keys are 2048 bits by default.  include the -b argument with the number of bits you would like
``````sh
ssh-keygen -b 4096

``````
##### Removing or Changing the Passphrase on a Private Key
If you have generated a passphrase for your private key and wish to change or remove it, you can do so easily.
``````sh
ssh-keygen -p

``````

##### Copying your Public SSH Key to a Server with SSH-Copy-ID
To copy your public key to a server, allowing you to authenticate without a password, a number of approaches can be taken
- Use ssh-copy-id on Server 1, assuming you have the key pair (generated with ssh-keygen):
- Now you should be able to ssh into Server 2 with ssh using the private key

``````sh
ssh-copy-id username@remote_host
ssh-copy-id -i ~/.ssh/id_rsa user@server2_hostname

ssh -i ~/.ssh/id_rsa user@server2_hostname

``````
##### Copying your Public SSH Key to a Server Manually
If you do not have password-based SSH access available, you will have to add your public key to the remote server manually

``````sh
cat ~/.ssh/id_rsa.pub
# On the remote server, create the ~/.ssh directory if it does not already exist:
mkdir -p ~/.ssh
# Afterwards, you can create or append the ~/.ssh/authorized_keys file by typing
echo public_key_string >> ~/.ssh/authorized_keys

``````

##### Connecting to a Remote Server
To connect to a remote server and open a shell session there, be sure to give access to the key directory
chmod 400 file_name
 - ~/.ssh/id_rsa
     Contains the private key for authentication.  These files contain
     sensitive data and should be readable by the user but not acces-
     sible by others (read/write/execute

- ~/.ssh/id_rsa.pub
     Contains the public key for authentication.  These files are not
     sensitive and can (but need not) be readable by anyone
``````sh
mkdir -p ~/.ssh && cd ~/.ssh && ssh-keygen -b 2048 -C "major.sampson@devopsmyway.com"
chmod 400 file_name_or_~/.ssh
ssh -i '/path/to/Private_keyfile' username@remote_server_ip

ssh -i ~/.ssh/id_rsa username@public_ip_of_remote_host

``````
# Command and Option Summary
- -p “Change the passphrase
- -i "Input" to the ~/.ssh/private_key_file, 
- -f "File" Specifies name of the file in which to store the created key.
- -c "Comment" Changes the comment for a keyfile.
- -l "Fingerprint" Print the fingerprint of the specified public key.
- -t “Type” This option specifies the type of key to be created. Commonly used values are: - rsa for RSA keys - dsa for DSA keys - ecdsa for elliptic curve DSA keys


