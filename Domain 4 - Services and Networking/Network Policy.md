
https://kubernetes.io/docs/concepts/services-networking/network-policies/
https://snyk.io/blog/kubernetes-network-policy-best-practices/
https://banzaicloud.com/blog/network-policy/

##### Anatomy of a network policy
- podSelector: This selects particular Pods in the same namespace as the NetworkPolicy which should be allowed as ingress sources or egress destinations.

- namespaceSelector: This selects particular namespaces for which all Pods should be allowed as ingress sources or egress destinations.

- namespaceSelector and podSelector: A single to/from entry that specifies both namespaceSelector and podSelector selects particular Pods within particular namespaces

##### Single Element for Ingress, pods in same 
This policy contains a single from element allowing connections from Pods with the label role=client in namespaces with the label user=alice
- any pod in a namespace with the label user=alice
``````sh
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
``````
##### 2 Elements in the array Ingress
It contains two elements in the from array, and allows connections from Pods in the local Namespace with the label role=client, or from any Pod in any namespace with the label user=alice.
``````sh
ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client

``````
#### Adding Egree Rules
You can add egress rules to allow connections from any pod labeled role=db in namespace default

``````sh
egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
-----        
     ports:
        - protocol: TCP
          port: 5978
``````

##### Install CNI Plugin
The functionality of controlling traffic is typically achieved in networks by using firewalls (software or hardware). Kubernetes provides networking functionality by using network plugins. If you don't install a network plugin, the policies won't have any effect.
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/

##### Network Policy syntax

- podSelector: selects a group of pods for which the policy applies (the policy target) - An empty podSelector: {} selects all pods in the namespace.
- policyTypes: defines the type of traffic to be restricted (inbound, outbound, both), ingress and/or egress - this is optional but I recommend to always specify it explicitly.
- ingress whitelist rules: allowed inbound traffic to the target pods whitelist rules - ingress:  - {} - Allow all ingress traffic
- egress whitelist rules: allowed outbound traffic from the target pods whitelist rules - egress:  - {} - Allow all egress traffic
- Pods policy applies to can be made from any IP within the CIDR block 172.17.0.0/16 (that's 65536 IP addresses, from 172.17.0.0 to 172.17.255.255)
- empty selector: An empty selector means everything. If PodSelector:{} is mentioned, it will select all the pods in the namespace
- Null selector: If the policy contains a Null selector [], means it is not selecting anything (so all traffic is blocked).

#### Deny all traffic in the default namespace 
Since this resource defines both policyTypes (ingress and egress), but doesn’t define any whitelist rules, it blocks all the pods in the default namespace from communicating with each other.

``````sh
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}    #all pods created in this default namespace are target
  policyTypes:
  - Ingress
  - Egress
EOF
---- ##### Test pods in deafult namespace
kubectl run box1 --image=busybox --dry-run=client -oyaml --labels="app=box1" --command -- sh -c "sleep 3600" > box1.yaml
kubectl run box2 --image=busybox --dry-run=client -oyaml --labels="app=box2"  --command -- sh -c "sleep 3600" > box2.yaml

kubectl exec -it box1 -- sh
ping 192.168.1.4  # notice traffic behavior
``````
#### Allow Traffic in default namespace
this resource defines both ingress and egress whitelist rules for all traffic, the pods in the default namespace can now communicate with each other.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-both
spec:
  podSelector: {}
  ingress:
  - {}
  egress:
  - {}
  policyTypes:
  - Egress
  - Ingress
EOF
---- # Test Pods for traffic behavior
kubectl run test1 --image=busybox --dry-run=client -oyaml --command -- sh -c "sleep 3600" --labels="app=box1" > box1.yaml
kubectl run test2 --image=busybox --dry-run=client -oyaml --command -- sh -c "sleep 3600" --labels="app=box2" > box2.yaml

kubectl get pods -owide --show-labels
kubectl exec -it box1 -- sh
curl 192.168.1.4
``````
#### Example 2 - Create an allow-out-to-in policy, and add labels to pods
To isolate all pods in namespace default:

allow inbound traffic from pods in namespace default with the label test=out
allow outbound traffic to pods in namespace default with the label test=in

``````sh
kubectl run test1 --image=nginx --labels='role=db' -n default
kubectl run frontend --image=nginx --labels='role=frontend' -n default
kubectl create ns myproject
kubectl run test2 --image=ubuntu -- sh -c 'sleep 3600' -n myproject

kubectl get po -owide --show-labels #copy ips and add on the ipblocks for default ns
kubectl get po -owide -n myproject --show-labels


cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 10.0.0.0/16
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/16
EOF
``````
#### Test Pods

``````sh
kubectl exec -it testpod -n myproject -- sh
apt update && upgrade
apt install ipuntils-ping
ping 10.244.0.53   #ip of role-db
``````

###### A more realistic example To Test Network Policy
We will create 3 pods, 1 for the front-end, back-end and database application that should have initially abilities to communicate.

YAML file contains the 3 services which is pointing to 3 pods as below -
frontend service => frontend pod
backend service => backend pod
db service => mysql pod

2- Create pods and services -vi  network-policy-demo.yaml

``````sh
vi pod1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    role: frontend
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
---
vi svc1.yaml

apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    role: frontend
spec:
  selector:
    role: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
vi pod2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    role: backend
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
---
vi svc2.yaml

apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    role: backend
spec:
  selector:
    role: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
vi svcdb.yaml

apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    name: mysql
spec:
  selector:
    name: mysql
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
---
vi pod3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  containers:
    - name: mysql
      image: mysql:latest
      env:
        - name: "MYSQL_USER"
          value: "mysql"
        - name: "MYSQL_PASSWORD"
          value: "mysql"
        - name: "MYSQL_DATABASE"
          value: "testdb"
        - name: "MYSQL_ROOT_PASSWORD"
          value: "shhhitssecret"
      ports:
        - name: http
          containerPort: 3306
          protocol: TCP
---------------
kubectl apply -f all-files
kubectl get all

``````
Test: the communication from frontend pod to the backend pod using curl. We are calling the backend service with the port 80

``````sh
kubectl exec -it frontend -- bash
curl backend:80

``````
Test call database pod from the frontend pod and backend pod. We will verify that database pod with port number 3306 is accessible from frontend pod using telnet utility.
It’s connected. Frontend pod is able to connect to database
``````sh
apt update && apt install telnet -y
telnet db 3306
``````
##### Network Policy can be used to prevent that communication
For the security purposes, Frontend pod should not communicate to the database directly.
Let’s allow the communication only to the backend pod using the Network policy.
 We want to create the policy for the ingress traffic to the database pod. All the incoming traffic on database pod should be blocked except the backend pod.

``````sh

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-netpol
spec:
  podSelector:
    matchLabels:
      name: mysql
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 3306
  policyTypes:
  - Ingress

----
kubectl apply -f db-netpol.yaml
``````
For testing the netpol connection to th db from both pods:
``````sh
kubectl exec -it backend -- bash
apt update && apt install telnet -y
telnet db 3306

kubectl exec -it frontend -- bash
apt update && apt install telnet -y
teclnet db 3306

``````
##### Example question
Create a network policy to allow traffic from the Internal application only to the payroll-service and db-service.

Policy Name: internal-policy
Policy Type: Egress
Egress Allow: payroll
Payroll Port: 8080
Egress Allow: mysql
MySQL Port: 3306

``````sh

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: Internal
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              name: payroll
      ports:
        - protocol: TCP
          port: 8080
          
    - to:
        - podSelector:
            matchLabels:
              name: mysql
      ports:
        - protocol: TCP
          port: 3306


``````
