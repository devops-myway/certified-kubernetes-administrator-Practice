#### What is DNS
DNS (Domain Name System) is a mechanism that makes the internet human-friendly. It maps human-readable hostnames to machine-readable IP addresses. When you enter a domain name into your browser — for example, www.google.com — your computer queries the nearest DNS server to find the correct IP address for that domain.

#### /etc/hosts
/etc/hosts is a local text file used for static hostname-to-IP resolution. It is typically used for administrative purposes, such as backend and internal functions, because it is isolated in scope — only the local server references it.

``````sh
cat /etc/hosts

127.0.0.1     localhost.localdomain localhost
10.2.3.4      myhost.domain.org myhost

``````
#### /etc/resolv.conf
This file defines which DNS servers your host uses for name resolution, and which domain names are appended to bare hostnames (via the search directive). If you are using DHCP, this file is automatically populated with DNS records issued by the DHCP server.
``````sh
cat /etc/resolv.conf

search      domain1.org domain2.org
nameserver  192.168.3.3
nameserver  192.168.4.4

``````
#### /etc/nsswitch.conf
This file defines the order of name resolution — for example, whether the system should consult /etc/hosts or a DNS server first. If the file contains the following configuration:
then /etc/hosts is checked first. If the domain remains unresolved, the configured DNS server is consulted next.
``````sh
hosts: files dns

cat /etc/nsswitch.conf | grep "hosts"

hosts:      files dns
``````
#### Troubleshooting DNS with Command-Line Tools
##### nslookup
By default, nslookup looks up the A record for a domain. For example:
``````sh
nslookup example.com
#####
Server:  resolver1.opendns.com
Address:  208.67.222.222

Name:    example.com
Address:  93.184.216.119
``````
From this output, you can see that example.com points to IP address 93.184.216.119, and that the query was resolved by resolver1.opendns.com.
You can also specify a custom DNS server to use for the query:

``````sh
nslookup example.com 208.67.222.222

``````
#### dig
dig displays a QUESTION SECTION (the request) and an ANSWER SECTION (the DNS server's response). Using the default options, dig looks up the A record for a domain.
``````sh
dig example.com

######
; <<>> DiG 9.8.4-rpz2+rl005.12-P1 <<>> example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46803
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;example.com.   IN A

;; ANSWER SECTION:
example.com.  2424 IN A 93.184.216.119

;; Query time: 12 msec
;; SERVER: 192.168.0.1#53(192.168.0.1)
;; WHEN: Thu Jan  9 16:07:09 2014
;; MSG SIZE  rcvd: 45

``````