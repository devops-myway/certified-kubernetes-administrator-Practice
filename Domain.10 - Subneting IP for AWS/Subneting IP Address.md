http://www.network-calculator.com/


#### Example of an IP Address format:
An IP Address is a 32-bit logical address that distinctively classifies a host of the network. The host can be a computer, Mobile handset or even a tablet.

192.168.1.64 (in decimal)
11000000.10101000.00000001.01000000 (in binary)

The range of an octet in binary is from 00000000 to 11111111 and in decimal from 0 to 255.
There are 8 bits and each bit has the value of 2 to the power n (2^n).

So the value of each bit is as follows:

2^7 2^6 2^5 2^4 2^3 2^2 2^1 2^0 (^ denotes the power)

Thus the result would be:

128+ 64+ 32+ 16+ 8+ 4+ 2+ 1

When all the bits are 1 then the values come out to be 255 (128+64+32+16+8+4+2+1= 255).

Suppose all the bits of an octet is not 1. Then see how we can calculate the IP address:

1 0 0 1 0 0 0 1, 128+0+0+16+0+0+0+1= 145.

##### Network Classes And Subnet Mask

The organization which governs the internet has divided the IP addresses into different classes of the network. Each class is identified by its subnet mask.

RFC1918 Subnets: The RFC1918 address space includes the following networks:

10.0.0.0 – 10.255.255.255  (10/8 prefix)  - 10/8) is typically used within large organizations that have thousands of hosts
172.16.0.0 – 172.31.255.255  (172.16/12 prefix) - The 172 and 192 subnets are more common within smaller organizations or home networks
192.168.0.0 – 192.168.255.255 (192.168/16 prefix) - The 172 and 192 subnets are more common within smaller organizations or home networks

The classification is shown with the help of the below table and figure.

Class	        Ist octet Decimal Range	            Network/Host ID                             Default subnet mask
A	            1 to 126                            N.H.H.H                                      255.0.0.0
B	            128 to 191	                        N.N.H.H	                                       255.255.0.0
C	            192 to 223	                        N.N.N.H                                         255.255.255.0
D	            224 to 239	                        Reserved for Multicasting	
E	            240 to 254	                        Experimental

##### Subnetting

developers came up with a way to split up an IP address into smaller networks called subnets. This process, called subnetting, uses the host section of the IP address to break it down into those smaller networks or subnets.
Variable Length Submasking (VLSM), which basically allowed network engineers to create subnets within subnets. And those subnets could be different sizes, so there would be fewer unused IP addresses.
Any switch, router or gateway that connects n networks has n unique Network ID and one subnet mask for each of the network it interconnects with.

 #### The formulae of subnetting is as follows: No of Subnets to use
 2^n >= requirement.

#### The formulae of a number of hosts per subnet is as follows:
2^n -2

#### Example 1: We have taken an example of Class C network ID with a default subnet mask.
Suppose Network ID/IP address is: 192.168.1.0/24
Default mask: 255.255.255.0 (in decimal) Thus the number of bits are 8+8+8+0= 24 bits
Default Subnet mask: 11111111.11111111.11111111.00000000 (in binary)

##### Therefore, to customize the subnet as per requirement:
we will borrow bits from the host portion of the subnet mask, while building our sub-networks and its cidr range.

We take a subnet mask of 255.255.255.248 (in decimal)
11111111.11111111.11111111.11111 000 (in binary). Borrow 3 bits from the host portion to customize our subnet and host per subnet
We can see that the last 3 bits of the last octet can be used for host ID addressing purpose.

Thus the number of subnets= 2^n = 2^5= 32 subnets (n=5)
Number of hosts per subnet= 2^h -2= 2^3 -2= 8-2= 6 Subnets i.e. usable Host IP
Subnet mask– 255.255.255.248 (binary)
11111111.11111111.11111111.11111(subnet bits) 000(host bits)
From the decimal notation we can calculate the number of bits having 1 in each octet: 8+8+8+5= 29
LIST OF SUBNETS
IP: 192.168.1.0/29
SUBNET MASK: 255.255.255.248 (29 bits)

The number of each sub-Network IP range: 0-31 = 32 numbers , 31 subnets X 8 hosts= 248:
192.168.1.0/29

NETWORK	                HOST RANGE	                    BROADCAST
192.168.0.0/29	    192.168.0.1 -- 192.168.0.6	        192.168.0.7
192.168.0.8/29	    192.168.0.9 -- 192.168.0.14	        192.168.0.15
192.168.0.16/29	    192.168.0.17 -- 192.168.0.22	    192.168.0.23
192.168.0.24/29	    192.168.0.25 -- 192.168.0.30	    192.168.0.31
192.168.0.32/29	    192.168.0.33 -- 192.168.0.38	    192.168.0.39
192.168.0.40/29	    192.168.0.41 -- 192.168.0.46	    192.168.0.47
.....
192.168.0.224/29	192.168.0.225 -- 192.168.0.230	    192.168.0.231
192.168.0.232/29	192.168.0.233 -- 192.168.0.238	    192.168.0.239
192.168.0.240/29	192.168.0.241 -- 192.168.0.246	    192.168.0.247
192.168.0.248/29	192.168.0.249 -- 192.168.0.254	    192.168.0.255

##### Example2: For a class C network with the network IP 190.164.24.0 and subnet mask 255,255.255.240 means /28 in CIDR notation.

Network Ip Cidr: 190.164.24.0 / 24
Mask: 255.255.255.0
HOST RANGE (254 hosts): 190.164.24.1 - 190.164.24.254

We will borrow the host IP from the last octet for the subnetting which is 11111111.11111111.11111111.1111(subnet bits) 0000 (host bits)

Here the no. of subnets are 2^n.subnet = 2^4 = 16 subnets maximum
Number of host per subnet is 2^host -2 = 2^4 -2 = 14  14 usable host IP

LIST OF SUBNETS
Subnet IP: 190.164.24.0/28
SUBNET MASK: 255.255.255.240 (28 bits)
we have 0-15 = 16 Subnets for our network, 15x16 = 240 Network IPs
e.g :

Network ID CIDR: 190.164.24.0/24
the Subnet mask can be denoted as 190.164.24.0/28

Network IP	    First Usable IP	    Last Usable IP	    Broadcast IP
190.164.24.0/28	    190.164.24.1 -- 190.164.24.14	190.164.24.15
190.164.24.16/28	190.164.24.17 -- 190.164.24.30	190.164.24.31
190.164.24.32/28	190.164.24.33 -- 190.164.24.46	190.164.24.47
190.164.24.48/28	190.164.24.49 -- 190.164.24.62	190.164.24.63
......
190.164.24.208/28	190.164.24.209 -- 190.164.24.222	190.164.24.223
190.164.24.224/28	190.164.24.225 -- 190.164.24.238	190.164.24.239
190.164.24.240/28	190.164.24.241 -- 190.164.24.254	190.164.24.255


##### Example3: For a class C network with the network IP 172.31.0.0/16 and subnet mask 255,255.0.0 means /16 in CIDR notation.

List the Network IP:
IP: 173.31.0.0 / 16
MASK: 255.255.0.0 (16 bits)
HOST RANGE: (65,534 hosts) 173.31.0.1 - 173.31.255.254

We will borrow the host IP from the 2 last octets for the subnetting which is 11111111.11111111.1111(subnet bits) 0000.0000 0000 (host bits)
New Subnet Cidr: 172.31.0.0/20
Here the no. of subnets are 2^n.subnet = 2^4 = 16 subnets
Number of host per subnet is 2^host -2 = 2^12 -2 = 4094 host per subnets means 4094 usable host IP
LIST OF SUBNETS Calculation
IP: 10.0.0.0/20
SUBNET MASK: 255.255.240.0 (20 bits)

simple comuputation:
the subnet ip range = 0-15 =16 subnets, 15 x 16 = 240

NETWORK	            HOST RANGE	                BROADCAST
173.31.0.0/20	173.31.0.1 -- 173.31.15.254	    173.31.15.255
173.31.16.0/20	173.31.16.1 -- 173.31.31.254	173.31.31.255
173.31.32.0/20	173.31.32.1 -- 173.31.47.254	173.31.47.255
173.31.48.0/20	173.31.48.1 -- 173.31.63.254	173.31.63.255
...
173.31.208.0/20	173.31.208.1 -- 173.31.223.254	173.31.223.255
173.31.224.0/20	173.31.224.1 -- 173.31.239.254	173.31.239.255
173.31.240.0/20	173.31.240.1 -- 173.31.255.254	173.31.255.255





