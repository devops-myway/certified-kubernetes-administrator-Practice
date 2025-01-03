##### SCP – Secure Copy Protocol
- SCP – Secure Copy Protocol. While it’s a separate tool, it leverages the underlying SSH protocol to securely transfer files in an encrypted manner, in a way that can’t be intercepted by anyone en route.
- SSH is a way to log into a remote server from your local machine, it’s not a way to transfer a file from your local computer to the remote server.

###### Transferring Files from Local to Remote via SCP

To make the transfer, you’ll need the following pieces of information:

- Full path to the file on the local computer
- Your login credentials on the remote server
- The IP address or the domain name of the remote server
- The path to the folder on the remote destination server

-r      Recursively copy entire directories. this is not needed during copy process
``````sh
scp Test-File.txt [Username]@[IP Address]:[Destination Directory]
scp /path/to/local/file.txt username@remote_host:/path/on/remote/system/

# You need to run the scp command from the local machine, not on the remote
scp -P 2222 file.ext username@domain:~/

#If you want to transfer a folder, just zip it first, we can unzip it later on.
# Here, root is your account, and 10.145.198.100 is the remote server's IP address. We're going to copy the_file to ~/ folder in the remote.

scp your_path_to_the_file/the_file root@10.145.198.100:~/

#Unzip file:
unzip the_zip_file.zip -d destination_folder


``````

##### Copy a File from a Remote System to the Local System

``````sh
ssh -i /path/to/private_key.ppk username@hostname  # the input is for ssh access to the remote host
scp -i /path/to/private_key.ppk remote-hostname@remote_ip:/path/to/file ./path/to/localhost_dir
scp -i ./bqm-weu-nethub-snd-glb-linux-vm.pem vmadmin@40.118.24.255:/home/vmadmin/state-setup.yaml ./state-setup.yaml

``````


