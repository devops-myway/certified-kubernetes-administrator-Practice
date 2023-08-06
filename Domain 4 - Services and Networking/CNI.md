https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/

###### CNI

CNI (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins. CNI concerns itself only with network connectivity of containers and removing allocated resources when the container is deleted. Because of this focus, CNI has a wide range of support and the specification is simple to implement.

###### Installation
A Container Runtime, in the networking context, is a daemon on a node configured to provide CRI Services for kubelet. In particular, the Container Runtime must be configured to load the CNI plugins required to implement the Kubernetes network model.

##### CNI configuration Directories
CNI configuration file:  default /etc/cni/net.d
CNI binary is included in your CNI bin dir default: /opt/cni/bin

``````sh
ls -l /etc/cni/net.d

cat << EOF | tee /etc/cni/net.d/10-containerd-net.conflist
{
 "cniVersion": "1.0.0",
 "name": "containerd-net",
 "plugins": [
   {
     "type": "bridge",
     "bridge": "cni0",
     "isGateway": true,
     "ipMasq": true,
     "promiscMode": true,
     "ipam": {
       "type": "host-local",
       "ranges": [
         [{
           "subnet": "10.88.0.0/16"
         }],
         [{
           "subnet": "2001:db8:4860::/64"
         }]
       ],
       "routes": [
         { "dst": "0.0.0.0/0" },
         { "dst": "::/0" }
       ]
     }
   },
   {
     "type": "portmap",
     "capabilities": {"portMappings": true},
     "externalSetMarkChain": "KUBE-MARK-MASQ"
   }
 ]
}
EOF
---
ls -l /opt/cni/bin   # default directory
``````
##### Find out what container runtime endpoint you use
The container runtime talks to the kubelet over a Unix socket using the CRI protocol, which is based on the gRPC framework. The kubelet acts as a client, and the runtime acts as the server. 

look for the --container-runtime flag and the --container-runtime-endpoint flag.
``````sh
kubectl get nodes -o wide   # The container runtime you used on node

ps -ex | grep kubelet
ps -ex | grep kubelet | grep container-runtime-endpoint
ps -ex | grep kubelet | grep --color container-runtime-endpoint    # container runtime endpoint of the kubelet service
``````
default directory of the kubelet
``````sh
ls -l /opt/cni/bin

-rwxr-xr-x 1 root root  3859475 Jan 16  2023 bandwidth
-rwxr-xr-x 1 root root  4299004 Jan 16  2023 bridge
-rwxr-xr-x 1 root root 10167415 Jan 16  2023 dhcp
-rwxr-xr-x 1 root root  3986082 Jan 16  2023 dummy
-rwxr-xr-x 1 root root  4385098 Jan 16  2023 firewall
-rwxr-xr-x 1 root root  2474798 Aug  2 08:08 flannel
-rwxr-xr-x 1 root root  3870731 Jan 16  2023 host-device
-rwxr-xr-x 1 root root  3287319 Jan 16  2023 host-local
-rwxr-xr-x 1 root root  3999593 Jan 16  2023 ipvlan
-rwxr-xr-x 1 root root  3353028 Jan 16  2023 loopback
-rwxr-xr-x 1 root root  4029261 Jan 16  2023 macvlan
-rwxr-xr-x 1 root root  3746163 Jan 16  2023 portmap
-rwxr-xr-x 1 root root  4161070 Jan 16  2023 ptp
-rwxr-xr-x 1 root root  3550152 Jan 16  2023 sbr
-rwxr-xr-x 1 root root  2845685 Jan 16  2023 static
-rwxr-xr-x 1 root root  3437180 Jan 16  2023 tuning
-rwxr-xr-x 1 root root  3993252 Jan 16  2023 vlan
-rwxr-xr-x 1 root root  3586502 Jan 16  2023 vrf

----
CNI plugin configured to be used on this kubernetes cluster

cat /etc/cni/net.d/10-flannel.conflist
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}

``````