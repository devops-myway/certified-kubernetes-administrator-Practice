#####  SSH tunnels and SSH port forwarding
- SSH tunnels, much like real tunnels, serve to connect two points. The first of these points is a computer that is usually located on an unsecured network.
- The target point is a server or web address that you can’t or don’t want to access from your network.
- SSH tunnels work as links between different servers and connect the TCP ports on two computers with each other.
- Any TCP port can be forwarded using SSH tunneling, which is why the process is also called SSH port forwarding or SSH forwarding.

##### What are SSH tunnels used for
- In most cases, SSH port forwarding is used to create an encrypted connection between a local computer (the local host) and a remote computer.
- If you’re transporting data from services that use an unencrypted protocol, you can use SSH forwarding to encrypt the data transfer.
- A SSH File Transfer Protocol, SFTP for short, will be used for this.
- SSH tunnels also offer increased security when you’re surfing on unfamiliar networks, for example in a hotel or coffee shop
-  SSH keys use asymmetric encryption and provide an even higher level of security.
-  It’s important to note that SSH tunneling is frequently used by hackers, who build backdoors in internal networks so that attackers can easily access internal data.

##### SSH local port forwarding
- ou can use any port number higher than 1024. Ports with smaller port numbers are privileged and can only be accessed by the root.
- Next, enter the IP address of the target server (remote_address) and your credentials (remote_port).
``````sh
ssh -L local_port: remote_address: remote_port username@server.com

ssh -L 8888: 123.234.1.111: 1234 Testuser@ssh.test.com
``````
#####  SSH remote port forwarding
- Remote port forwarding connects a port from the SSH server to a port on the client computer, which can then establish a connection to the target computer
``````sh
ssh -R remote_port: target_address: target_port user@ssh_server_address

ssh -R 8080: 127.0.0.1:3000 user@remote.host
remote.host:8080

``````
#####  SSH dynamic port forwarding
- A third way of using SSH tunnels involves dynamic port forwarding, which enables you to use a socket on your local computer that will function as a kind of SOCKS proxy
``````sh
ssh -D [local_ip_address:]local_port user@ssh_server_address
ssh -D 9090 -N -f user@remote.host

``````
#####  Reverse SSH tunnels

``````sh
ssh -Nf -R 2222:localhost:22 user@local.computer
ssh localhost -p 2222

``````
