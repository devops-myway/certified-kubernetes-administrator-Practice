https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

###### Debugging DNS Resolution - Create a simple Pod to use as a test environment

``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "infinity"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
EOF
--------
kubectl get pods dnsutils
kubectl exec -it dnsutils -- nslookup kubernetes.default
--
Server:    10.0.0.10
Address 1: 10.0.0.10
Name:      kubernetes.default
Address 1: 10.0.0.1
``````
###### Check the local DNS configuration first
Take a look inside the resolv.conf file if the nslookup dns search path is failing
tp resolve.
``````sh
kubectl exec -i -t dnsutils -- nslookup kubernetes.default
kubectl exec -it dnsutils -- cat /etc/resolv.conf

------ # the search path and name server are set up like the following
Server:    10.0.0.10
Address 1: 10.0.0.10

nslookup: can't resolve 'kubernetes.default'
``````

##### Check if the DNS pod is running

``````sh
kubectl get po -n kube-system -owide
kubectl get pods --namespace=kube-system -l k8s-app=kube-dns
-------
coredns-7b96bf9f76-5hsxb   1/1       Running   0           1h
coredns-7b96bf9f76-mvmmt   1/1       Running   0           1h
``````
###### Check for errors in the DNS pod
See if there are any suspicious or unexpected messages in the logs.
``````sh
kubectl logs -n kube-system -l k8s-app=kube-dns
---- # Here is an example of a healthy CoreDNS log: 
.:53
2018/08/15 14:37:17 [INFO] CoreDNS-1.2.2
2018/08/15 14:37:17 [INFO] linux/amd64, go1.10.3, 2e322f6
CoreDNS-1.2.2
linux/amd64, go1.10.3, 2e322f6
2018/08/15 14:37:17 [INFO] plugin/reload: Running configuration MD5 = 24e6c59e83ce706f07bcc82c31b1ea1c

``````
##### Is DNS service up
Verify that the DNS service is up by using the kubectl get service command
``````sh
kubectl get svc --namespace=kube-system

``````
#### Are DNS endpoints exposed
You can verify that DNS endpoints are exposed by using the kubectl get endpoints command
``````sh
kubectl get endpoints kube-dns --namespace=kube-system
``````
##### Are DNS queries being received/processed - Adding Log Plugins
You can verify if queries are being received by CoreDNS by adding the log plugin to the CoreDNS configuration (aka Corefile)
Corefile is held in a ConfigMap named coredns. To edit it, use the command

``````sh
kubectl edit cm coredns -n kube-system
``````
Then add log in the Corefile section per the example below:
``````sh
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        log
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }    
``````
If CoreDNS pods are receiving the queries, you should see them in the logs. Here is an example of a query in the log:
``````sh
:53
2018/08/15 14:37:15 [INFO] CoreDNS-1.2.0
2018/08/15 14:37:15 [INFO] linux/amd64, go1.10.3, 2e322f6
CoreDNS-1.2.0
linux/amd64, go1.10.3, 2e322f6
2018/09/07 15:29:04 [INFO] plugin/reload: Running configuration MD5 = 162475cdf272d8aa601e6fe67a6ad42f
2018/09/07 15:29:04 [INFO] Reloading complete
172.17.0.18:41675 - [07/Sep/2018:15:29:11 +0000] 59925 "A IN kubernetes.default.svc.cluster.local. udp 54 false 512" NOERROR qr,aa,rd,ra 106 0.000066649s
``````
##### Does CoreDNS have sufficient permissions
CoreDNS must be able to list service and endpoint related resources to properly resolve service names.
Sample error message:
``````sh
2022-03-18T07:12:15.699431183Z [INFO] 10.96.144.227:52299 - 3686 "A IN serverproxy.contoso.net.cluster.local. udp 52 false 512" SERVFAIL qr,aa,rd 145 0.000091221s
``````
First, get the current ClusterRole of system:coredns
``````sh
kubectl describe clusterrole system:coredns -n kube-system
----
Expected output:

PolicyRule:
  Resources                        Non-Resource URLs  Resource Names  Verbs
  ---------                        -----------------  --------------  -----
  endpoints                        []                 []              [list watch]
  namespaces                       []                 []              [list watch]
  pods                             []                 []              [list watch]
  services                         []                 []              [list watch]
  endpointslices.discovery.k8s.io  []                 []              [list watch]
``````
If any permissions are missing, edit the ClusterRole to add them:
``````sh
kubectl edit clusterrole system:coredns -n kube-system
----
# Example insertion of EndpointSlices permissions:
- apiGroups:
  - discovery.k8s.io
  resources:
  - endpointslices
  verbs:
  - list
  - watch
``````
##### Are you in the right namespace for the service
DNS queries that don't specify a namespace are limited to the pod's namespace.
``````sh
kubectl exec -it dnsutils -- nslookup <service-name>
``````
This query specifies the namespace:
``````sh
kubectl exec -it ubuntu -- nslookup <service-name>.<namespace>
``````