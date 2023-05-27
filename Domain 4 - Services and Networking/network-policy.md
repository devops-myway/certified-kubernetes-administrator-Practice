
https://kubernetes.io/docs/concepts/services-networking/network-policies/
https://snyk.io/blog/kubernetes-network-policy-best-practices/
https://dev.to/loft/kubernetes-network-policies-a-practitioner-s-guide-1bfg

##### Network Policies Scope
Using the NetworkPolicy resource, you can control the traffic flow for your applications in the cluster, at the IP address level or port level (OSI layer 3 or 4). With the NetworkPolicy you can define how your Pod can communicate with various network entities in the cluster.

###### Why We Need Network Policies

It is of paramount importance to secure the traffic in our clusters. By default, all pods can talk to all pods with no restriction.
NetworkPolicy resource allows us to restrict the ingress and egress traffic to/from pods. This way, we can secure our resources so that only legitimate traffic is allowed to/from the applications.
For example, it provides the means to restrict the ingress traffic of a database pod to only backend pods, and it can restrict ingress traffic of a backend pod's traffic to a frontend application only.

##### Install CNI Plugin
The functionality of controlling traffic is typically achieved in networks by using firewalls (software or hardware). Kubernetes provides networking functionality by using network plugins. If you don't install a network plugin, the policies won't have any effect.
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
These network plugins provide Network Policy implementation and more, such as advanced monitoring, L7 filtering, integration to cloud networks, etc.
If you are using cloud providers for your Kubernetes setup (such as AWS, Azure, GCP), they might already have deployed a network plugin that supports network policies. use kubectl get po -n kube-system to check the cni plugin for up and running.

##### Writing & Applying Network Policies
In a Kubernetes cluster, by default, all pods are non-isolated, meaning all ingress and egress traffic is allowed.
Once a network policy is applied and has a matching selector, the pod becomes isolated, meaning the pod will reject all traffic that is not permitted by the aggregate of the network policies applied.

##### Network Policy syntax

The spec of the resource mainly consists of three parts:
- podSelector: Use labels to select the group of pods for which the rules will be applied. using app=hello applies the policy to all Pods with that label.
- policyTypes — Decide if the policy applies for incoming (ingress) traffic, outgoing (egress) traffic, or both.

- Ingress Rules:
Defines the rules that will be applied to the ingress traffic of the selected pod(s). If it is empty, it matches all the ingress traffic. If it is absent, it doesn't affect ingress traffic.
Fields:
from: an array of NetworkPolicyPeer (ipBlock, namespaceSelector, podSelector)
ports: an array of NetworkPolicyPort (port, endport, protocol)

- Egress Rules:
Defines the rules that will be applied to the egress traffic of the selected pod(s). If it is empty, it matches all the egress traffic. If it is absent, it doesn't affect egress traffic.
Fields:
ports: an array of NetworkPolicyPort (port, endport, protocol)
to: an array of NetworkPolicyPeer (ipBlock, namespaceSelector, podSelector)

#### Walkthrough

``````sh
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sample-network-policy
  namespace: default
``````
###### Here is a sample NetworkPolicy:
The podSelector tells us that the policy applies to all Pods in the default namespace that have the app: hello label set.
PolicyType: We are defining policy for both ingress and egress traffic.
The ingress Rule policy below:
Pods policy applies to can be made from any IP within the CIDR block 172.17.0.0/16 (that's 65536 IP addresses, from 172.17.0.0 to 172.17.255.255), except for Pods whose IP falls within the CIDR block 172.17.1.0/24 (256 IP addresses, from 172.17.1.0 to 172.17.1.255) to the port 8080. Additionally, the calls to the Pods policy applies to can be coming from any Pod in the namespace(s) with the label owner: ricky and any Pod from the default namespace, labeled version: v2

The egress Rule policy below:
specifies that Pods with the label app: hello in the default namespace can make calls to any IP within 10.0.0.0/24 (256 IP addresses, from 10.0.0.0, 10.0.0.255), but only to the port 5000.

``````sh
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: hello
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          owner: ricky 
    - podSelector:
        matchLabels:
          version: v2
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5000

``````
Let's look at an example that demonstrates how to disable egress traffic from the Pods.
Once the Pod is running, let's try calling google.com using curl
The call completes successfully
``````sh
apiVersion: v1
kind: Pod
metadata:
  name: no-egress-pod
  labels:
    app.kubernetes.io/name: hello
spec:
  containers:
    - name: container
      image: radial/busyboxplus:curl
      command: ["sh", "-c", "sleep 3600"]
-------
kubectl exec -it no-egress-pod -- curl -I -L google.com

``````
Let's define a network policy that will prevent egress for Pods with the label app.kubernetes.io/name: hello:
If you run the same command this time, curl won't be able to resolve the host.
``````sh
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-egress
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: hello
  policyTypes:
    - Egress
-----------------
kubectl exec -it no-egress-pod -- curl -I -L google.com
``````
###### Common Network Policies

https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-traffic


