
https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/

https://www.alibabacloud.com/help/en/container-service-for-kubernetes/latest/service-discovery-dns-dns-troubleshooting?spm=a2c63.p38356.0.0.65de26a0UUlwGB

###### DNS troubleshooting - Diagnostic procedure Terms

Internal domain name:
CoreDNS exposes services deployed in a cluster through an internal domain name that ends with .cluster.local by default. DNS queries for the internal domain name are resolved based on the DNS cache of CoreDNS instead of the upstream DNS servers.
External domain name:
Cluster external domain names are resolved by authoritative DNS provided by third-party DNS service providers, Cloud DNS. Cluster external domain names are resolved by the upstream DNS servers of CoreDNS. CoreDNS only forwards DNS queries to the upstream DNS servers.
Application pod:
Pods other than the pods of system components in a Kubernetes cluster.
Application pod that uses CoreDNS for DNS resolutions:
Application pods that use CoreDNS to process DNS queries.
Application pod that uses NodeLocal DNSCache for DNS resolutions:
After you install NodeLocal DNSCache in your cluster, you can configure DNS settings by injecting DNSConfig to application pods. This way, DNS queries of these pods are first sent to NodeLocal DNSCache. If NodeLocal DNSCache fails to process the queries, the queries are sent to the kube-dns Service of CoreDNS.


###### Diagnose the DNS configurations of application pods

``````sh
# Run the following command to query the YAML file of the foo pod. Then, check whether the dnsPolicy field in the YAML file is set to a proper value. 
kubectl get pod foo -oyaml

# If the dnsPolicy field is set to a proper value, check the DNS configuration file of the pod. 

# Run the following command to log on to the containers of the foo pod by using bash. If bash does not exist, use sh. 
kubectl exec -it foo bash

# Run the following command to query the DNS configuration file. Then, check the DNS server addresses in the nameserver field. 
cat /etc/resolv.conf

``````
##### Diagnose the status of the CoreDNS pod

``````sh
# Run the following command to query information about the CoreDNS pod:
kubectl -n kube-system get pod -o wide -l k8s-app=kube-dns

#Run the following command to query the real-time resource usage of the CoreDNS pod:
kubectl -n kube-system top pod -l k8s-app=kube-dns

``````
##### Inspecting a service’s A and SRV records in DNS
You start by inspecting the A and SRV records associated with your services.

###### Looking up a service’s A record
To determine the IP address of the quote service, you run the nslookup command in the shell running in the container of the dns-test pod like so:
``````sh
kubectl run -it --rm dns-test --image=giantswarm/tiny-tools  # This image contains the host, nslookup, and dig tools that you can use to examine DNS records

nslookup quote
--
Server:         10.96.0.10
Address:        10.96.0.10#53 //
Name:   quote.kiada.svc.cluster.local
Address: 10.96.161.97
``````
###### Looking up SRV records
``````sh
nslookup -query=SRV kiada

Server:         10.96.0.10
Address:        10.96.0.10#53 // //

``````

##### Diagnose the operational log of CoreDNS
Run the following command to query the operational log of CoreDNS
``````sh
kubectl -n kube-system logs -f --tail=500 --timestamps coredns-xxxxxxxxx-xxxxx
``````

##### Configure CoreDNS - CoreDNS ConfigMap options
In the kube-system namespace, you can find a CoreDNS ConfigMap. The CoreDNS server can be configured by maintaining a Corefile, which is the CoreDNS configuration file. You can modify the ConfigMap for the CoreDNS Corefile to change how DNS service discovery behaves for that cluster.
The CoreDNS Corefile is held in a ConfigMap named coredns. To edit it, use the command:

``````sh
  kubectl -n kube-system edit configmap coredns

``````

``````sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }    

``````

###### Scenario 1: Customize DNS servers for specified domain names - Stub-domain and upstream nameserver using CoreDNS
Example:
If a cluster operator has a Consul domain server located at "10.150.0.1", and all Consul names have the has a domain name suffix ".consul.local". To configure it in CoreDNS, the cluster administrator creates the following stanza in the CoreDNS ConfigMap.
``````sh
consul.local:53 {
    errors
    cache 30
    forward . 10.150.0.1
}
``````
The final ConfigMap along with the default Corefile configuration looks like:
``````sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . 172.16.0.1
        cache 30
        loop
        reload
        loadbalance
    }
    consul.local:53 {
        errors
        cache 30
        forward . 10.150.0.1
    }   

``````
##### Scenario 2: Customize DNS servers for external domain names
Do not modify the /etc/resolv.conf files on Elastic Compute Service (ECS) instances of the cluster. For example, if the IP addresses of the user-defined DNS servers are 10.10.0.10 and 10.10.0.20, you can modify the forward parameter.
``````sh
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 15s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
          ttl 30
        }
        prometheus :9153
        forward . 10.10.0.10 10.10.0.20{
          prefer_udp
        }
        cache 30
        loop
        reload
        loadbalance
    }

``````


Parameter                                       Description
--------------------------------------------------------------------------------------------------
errors:
Prints errors to standard output (stdout).

health:	
Generates health check reports for CoreDNS. The default listening port is 8080. This plug-in is used to evaluate the health status of CoreDNS. You can visit http://localhost:8080/health to view the health check report of CoreDNS.

ready:
	Reports the status of CoreDNS plug-ins. The default listening port is 8181. This plug-in is used to evaluate the readiness of CoreDNS plug-ins. You can visit http://localhost:8181/ready to view the readiness of the CoreDNS plug-ins. After all plug-ins are in the running state, a 200 response code is returned for the readiness of CoreDNS plug-ins.
kubernetes:
	The kubernetes plug-in of CoreDNS is used to provide DNS resolution for services in an ACK cluster.
prometheus:
	Exports CoreDNS metrics. You can visit http://localhost:9153/metrics to view CoreDNS metrics in Prometheus format.
  For getting CoreDNS metrics you should have Prometheus plugin enabled as part of the CoreDNS config.

forward or proxy:
	Forwards DNS queries to the predefined DNS server. By default, DNS queries of domain names beyond the cluster domain of Kubernetes are forwarded to the predefined DNS resolver (/etc/resolv.conf). The default configurations are based on the /etc/resolv.conf file on the host.
cache:
	Enables DNS caching.

loop:
	Performs loop detection. If a loop is detected, CoreDNS is suspended.

reload:
	Allows automatic reload of a changed Corefile. After you edit the ConfigMap, wait 2 minutes for the changes to take effect.

loadbalance:
	Works as a round-robin DNS load balancer to randomize the order of A, AAAA, and MX records in the answer.