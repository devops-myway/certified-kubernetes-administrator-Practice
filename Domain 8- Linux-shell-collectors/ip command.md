https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/

###### IP Command Utlility

The ip command is used to assign an address to a network interface and/or configure network interface parameters on Linux operating systems.
This command replaces old good and now deprecated ifconfig command on modern Linux distributions.

##### Understanding ip command OBJECTS syntax
OBJECTS can be any one of the following and may be written in full or abbreviated form:

Object	    Abbreviated form	    Purpose

link	    l	                    Network device.
address	    a, addr	                Protocol (IP or IPv6) address on a device.
addrlabel	addrl	                Label configuration for protocol address selection.

neighbour	n , neigh	            ARP or NDISC cache entry.
route	    r	                        Routing table entry.
rule	    ru	                    Rule in routing policy database.
maddress	m, maddr            	Multicast address.
mroute	        mr  	            Multicast routing cache entry.
tunnel	    t	                Tunnel over IP.
xfrm	    x	                Framework for IPsec protocol.

###### How understand (the output of) ifconfig or ip addr show 
eth0: is the interface name. It can be any string:

mtu 1500: maximum transmission unit = 1500 bytes, this is the largest size that a frame sent over this interface can be. This number is usually limited by the Ethernet protocol's cap of 1500. If you send a larger packet and it arrives at an Ethernet interface, then the frame will get fragmented and its payload transmitted in two or more packets. Not really any benefit to that, so it's best to follow standards.

qdisc: pfifo_fast queuing discipline = three pipes of first in first out. This determines how an interface chooses which packet to transmit next, when it's being overloaded.

group default: Interface groups give a single interface to clients by combining the capabilities of the aggregated interfaces on them.

qlen 1000: transmission queue length = 1000 packets. The 1000th packet will be queued, the 1001st will be dropped.

link/ether: means the link layer protocol is Ethernet:

brd: means broadcast. This is the address that the device will set as the destination when it sends a broadcast. An interface sees all traffic on the wire it's sitting on, but is polite enough to only read data addressed to it. The way you address an interface is by using its specific address, or the broadcast address.

inet: means the network layer protocol is internet (ipv4). inet6 would mean IPv6.

lft: stands for lifetime. If you get this address through DHCP, then you'll have a valid lifetime for your lease on the IP address. And just to make handoffs a little bit easier, a (probably) shorter preferred lifetime.

``````sh
ip OBJECT help
ip OBJECT h
ip a help
ip r help
``````
``````sh
ip a
ip addr

# specify and list particular interface TCP/IP details
ip a show eth0
ip a list eth0
ip a show dev eth0

# Only show running interfaces
ip link show up

``````
###### ip route: Routing table management commands
``````sh
ip r
ip r list
ip route list

``````
``````sh

``````
``````sh

``````