


##### What is DNS
DNS ( Domain Name System) is mechanism to make internet human friendly. IP address map with host-name in DNS server.
When we enter a domain name into our browser like www.goole.com computer find our nearest DNS server and ask what is the correct IP address for www.google.com.

##### /etc/hosts/
/etc/hosts file is the most important file in Linux operating system. It is a text file for name resolution. It is just a static lookup method for resolution.
is used for administrative purposes, such as backend and internal functions, which is substantially more isolated in scope, as only the local server will reference it.
``````sh
cat /etc/hosts

127.0.0.1     localhost.localdomain localhost
10.2.3.4      myhost.domain.org myhost

``````

##### /etc/resolv.conf file
Domain names that must be appended to bare hostnames, and DNS servers that will be used for name resolution.
Lists nameservers that are used by your host for DNS resolution. If you are using DHCP, this file is automatically populated with DNS record issued by DHCP server.

``````sh
cat /etc/resolv.conf

search      domain1.org domain2.org
nameserver      192.168.3.3
nameserver      192.168.4.4

``````

##### /etc/nsswitch.conf
It defined order of resolution. Who should it consult first for resolution, a DNS or a host file? For example, if the file has following configuration hosts: files dns then /etc/hosts file will be checked first for resolution, if domain is still un-resolvable, DNS will then be consulted.

``````sh
cat /etc/nsswitch.conf | grep "hosts"

hosts:      files dns
``````
##### Troubleshooting DNS with command-line tools

### nslookup
By default, nslookup looks up the A record for a domain
Interpret the output from nslookup. For example, the following output shows information for example.com:
we can see that example.com is currently pointing to IP address 93.184.216.119. We can also see that DNS server resolver1.opendns.com was used for the query.

``````sh

nslookup example.com

nslookup example.com 208.67.222.222
----
Server:  resolver1.opendns.com
Address:  208.67.222.222

Name:    example.com
Address:  93.184.216.119
``````
### dig
Interpret the output from dig. For example, the following output shows the dig information for example.com:

Dig displays a QUESTION SECTION (the request) and an ANSWER SECTION (what the DNS server sends in response to the request). In this case, we used the default options for dig, which simply looks up the A record for a domain. From this, we can see that example.com currently points to IP address 93.184.216.119.

``````sh
dig example.com

----
dig example.com
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

``````sh

``````