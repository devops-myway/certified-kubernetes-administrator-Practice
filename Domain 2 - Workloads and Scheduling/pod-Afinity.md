
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

##### inter-pod affinity and anti-affinity

Similar to node affinity are two types of Pod affinity and anti-affinity as follows:

requiredDuringSchedulingIgnoredDuringExecution - 
The Pod affinity rule uses the "hard", to tell the scheduler to co-locate Pods of two services in the same cloud provider zone because they communicate with each other a lot

preferredDuringSchedulingIgnoredDuringExecution - 
anti-affinity rule uses the "soft" , anti-affinity to spread Pods from a service across multiple cloud provider zones.

``````sh
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: registry.k8s.io/pause:2.0


``````

``````sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine

``````