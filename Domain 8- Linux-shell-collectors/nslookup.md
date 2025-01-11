##### nslookup
- nslookup is a simple but very practical command-line tool,
- which is principally used to find the IP address that corresponds to a host, or the domain name that corresponds to an IP address (a process called “Reverse DNS Lookup”)

##### Retrieving specific addresses out of the resource records
- The “resource record type” corresponds to the desired type of query, some of the options for which are included in the following chart:
- nslookup parameter values	Type of query
- A	: IPv4 address
- AAAA	:IPv6 address
- MX	:Domain name mailserver (mail exchanger)
- NS	: Domain name name server
- PTR	: “Pointer” entry (pinpoints the host name that corresponds to an IP address)
- SOA	: “Start-of-Authority” entry (details about the administration of the DNS zone)

``````sh
syntax:
set type= <RESOURCE RECORD TYPE>

$ nslookup
> google.com
Server: 127.0.0.53
Address: 127.0.0.53#53


Non-authoritative answer:
Name: google.com
Address: 142.250.192.46
Name: google.com
Address: 2404:6800:4009:828::200e
>

``````
##### If you are looking out for a simple DNS lookup, you can just fire nslookup DOMAIN_NAME

``````sh
$ nslookup google.com
Server: 127.0.0.53
Address: 127.0.0.53#53


Non-authoritative answer:
Name: google.com
Address: 142.250.192.46
Name: google.com
Address: 2404:6800:4009:828::200e

# If you want to do a reverse DNS lookup, simply type nslookup IP
$ nslookup 142.250.192.46
46.192.250.142.in-addr.arpa name = bom12s15-in-f14.1e100.net.
``````
##### Record Type
To find mail exchange servers, use nslookup -type=mx DOMAIN_NAME; honestly, this -type flag is really interesting
``````sh
syntax:
set type= <RESOURCE RECORD TYPE>

$ nslookup -type=mx google.com
Server: 127.0.0.53
Address: 127.0.0.53#53

Non-authoritative answer:
google.com mail exchanger = 10 smtp.google.com.

# If you are looking for a list of name servers
$ nslookup -type=ns google.com
Server: 127.0.0.53
Address: 127.0.0.53#53

Non-authoritative answer:
google.com nameserver = ns1.google.com.
google.com nameserver = ns2.google.com.

# type of any
$ nslookup -type=any google.com
Server: 127.0.0.53
Address: 127.0.0.53#53

# type = soa
$ nslookup -type=soa google.com
Server: 127.0.0.53
Address: 127.0.0.53#53

``````
##### 

``````sh


``````
