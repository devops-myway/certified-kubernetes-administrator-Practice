#### Note
https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys

SSH (Secure Shell) is a secure protocol provides a safe and secure way of executing commands, making changes, and configuring services remotely. It provides a text-based interface by spawning a remote shell. After connecting, all commands you type in your local terminal are sent to the remote server and executed there. The SSH connection is implemented using a client-server model.

When you connect through SSH, you will be dropped into a shell session, which is a text-based interface where you can interact with your server.

#### How SSH Authenticates Users

The public key must be copied to a file within the user’s home directory at ~/.ssh/authorized_keys
The default location is in the top level directory of the user running the terminal session: ~/.ssh/

ls ~/.ssh/

OpenSSH version of ssh-keygen:
Ssh-keygen is a tool for creating new authentication key pairs for SSH. Such key pairs are used for automating logins, single sign-on, and for authenticating hosts. The below openSSH key format:

~/.ssh/id_rsa: The private key. DO NOT SHARE THIS FILE!
~/.ssh/id_rsa.pub: The associated public key. This can be shared freely without consequence.
*.pub = Public Key
known_hosts = machines that you trust or remote machine or ec2 instances or servers

# Generating an SSH Key Pair

``````sh
ssh-keygen --help     # Generating public/private rsa key pair.

ssh-keygen -t rsa -C "azure-vm-key"    #Generating public/private rsa key pair, located in the .ssh hidden directory within your user’s home directory

-t : flag specifies the type of key to be generated

-C : allows you to provide a comment for the key

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

##### Displaying the SSH Key Fingerprint
Each SSH key pair share a single cryptographic “fingerprint” which can be used to uniquely identify the keys
``````sh
ssh-keygen -l

# To see the fingerprint and comment for your keys you can run

ssh-keygen -l -f ~/user/home/.ssh/id_rsa
``````

##### Copying your Public SSH Key to a Server with SSH-Copy-ID
To copy your public key to a server, allowing you to authenticate without a password, a number of approaches can be taken
If you currently have password-based SSH access configured to your server, and you have the ssh-copy-id utility installed.

``````sh
ssh-copy-id username@remote_host

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
To connect to a remote server and open a shell session there, be sure to give access to the key dir
chmod 400 file_name

``````sh
ssh -i ~/.ssh/id_rsa username@public_ip_of_remote_host

``````
# Command and Option Summary
-p “Change the passphrase
-i "Input" When ssh-keygen is required to access an existing key, this option designates the file.
-f "File" Specifies name of the file in which to store the created key.
-c "Comment" Changes the comment for a keyfile.
-l "Fingerprint" Print the fingerprint of the specified public key.
-t “Type” This option specifies the type of key to be created. Commonly used values are: - rsa for RSA keys - dsa for DSA keys - ecdsa for elliptic curve DSA keys


