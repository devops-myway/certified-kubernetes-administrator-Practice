

###### useful network commands to query Host
The netstat command is a CLI tool for network statistics. It gives an overview of network activities and displays which ports are open or have established connections.

``````sh
netstat help
netstat -a   # List All Ports and Connections
netstat -at   #List All TCP Ports
netstat -l    # List only listening ports
netstat -lt   # List TCP Listening Ports
netstat -n    # Display Numerical Addresses
netstat -nlpt  # Display all the network properties and ports for our objects
netstat -nlpt | grep magic   # show the ports attach to resource magic only
``````
``````sh
netstat -i     # List network interfaces
netstat -r    # Display Kernel IP Routing Table
``````

###### List Network Interfaces Using ip Command on Linux

Ip command: It is used to show or manipulate routing, devices, policy routing and tunnels
ifconfig command – It is used to display or configure a network interface
``````sh
ip link show   # show the interfaces and routing networks
ip link show | grep cni0  # get the state of the cni0 interface object
ip r    # how to see the routing table
ip addr    # view the ip address
ifconfig -a  # show all interfaces and network connections - deprecated

``````
