https://kubernetes.io/docs/concepts/policy/limit-range/

##### LimitRange

A LimitRange is a policy to constrain the resource allocations (limits and requests) that you can specify for each applicable object kind (such as Pod or PersistentVolumeClaim) in a namespace.

#### LimitRange for Memory
Kubernetes administrators can define RAM limits for their nodes.
These limits are enforced at higher priority over how much RAM your Pod declares and wants to use.
Let's define our first LimitRange : 25Mi RAM as min, 200Mi as max.

``````sh
cat << EOF| kubectl apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
    type: Container
EOF
``````
##### Pod That Requests RAM above and below Limits
Some namespaces or nodes may be dedicated to vast Pods. No tiny Pods allowed.
``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: example-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
EOF
# This will fail to schedule, along with a Pod that declares a CPU resource request of 700m, but not a limit:
# The Pod "example-conflict-with-limitrange-cpu" is invalid: spec.containers[0].resources.requests: Invalid value: "700m": must be less than or equal to cpu limit of 500m
--------

``````
##### If you set both request and limit, then that new Pod will be scheduled successfully even with the same LimitRange in place:

``````sh
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: example-no-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
      limits:
        cpu: 700m
EOF

``````

