##### Copy a File from a Remote System to the Local System

``````sh
ssh -i /path/to/private_key.ppk username@hostname  # the input is for ssh access to the remote host
scp -i /path/to/private_key.ppk remote-hostname@remote_ip:/path/to/file ./path/to/localhost_dir
scp -i ./bqm-weu-nethub-snd-glb-linux-vm.pem vmadmin@40.118.24.255:/home/vmadmin/state-setup.yaml ./state-setup.yaml

``````
##### Copy a Local File to a Remote System
``````sh
scp /path/to/local/file.txt username@remote_host:/path/on/remote/system/

``````

