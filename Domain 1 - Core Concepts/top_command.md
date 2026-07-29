#### top Command & kubectl top Command
- Launches an interactive, real-time view of system processes.
```sh
top

### top — Screen Layout Explained
top - 10:32:01 up 5 days,  3:21,  2 users,  load average: 0.52, 0.58, 0.60
Tasks: 213 total,   1 running, 212 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  1.3 sy,  0.0 ni, 92.8 id,  0.4 wa,  0.0 hi,  0.3 si
MiB Mem :  15949.8 total,   4321.2 free,   8012.4 used,   3616.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   7200.1 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1234 root      20   0  512000  45000  12000 R  25.0   0.3   1:02.34 nginx
 5678 postgres  20   0 1024000 200000  50000 S   5.2   1.3  10:22.11 postgres
 9012 claude    20   0  256000  32000   8000 S   1.0   0.2   0:15.00 python

### Column Headers Decoded

PID     → Process ID
USER    → Owner of the process
PR      → Priority (lower = higher priority)
NI      → Nice value (-20 highest priority, +19 lowest)
VIRT    → Virtual memory used (total claimed)
RES     → Resident memory (actual RAM used) ← most important
SHR     → Shared memory
S       → Status: R=Running S=Sleeping Z=Zombie D=Disk wait
%CPU    → CPU percentage used
%MEM    → RAM percentage used
TIME+   → Total CPU time consumed since start
COMMAND → Process name
```
#### Top Flags

```sh
# Run top for specific user
top -u nginx

# Run top with custom refresh interval (every 1 second)
top -d 1

# Run top for N iterations then exit (good for scripting)
top -b -n 5          # batch mode, 5 snapshots

# Save top output to file
top -b -n 1 > top_output.txt

# Monitor specific PID
top -p 1234

# Monitor multiple PIDs
top -p 1234,5678,9012

# Show threads instead of processes
top -H

# Show threads of a specific process
top -H -p 1234

```
#### htop — Enhanced Alternative to top

```sh
# Install
sudo apt install htop       # Ubuntu/Debian
sudo yum install htop       # RHEL/CentOS

# Run
htop

# Advantages over top:
# - Color coded, easier to read
# - Mouse support
# - Horizontal/vertical scroll
# - Tree view of processes (F5)
# - Easier process killing (F9)

```
#### kubectl top Command
- Prerequisites — Metrics Server must be installed
```sh
# Check if metrics server is running
kubectl get pods -n kube-system | grep metrics-server

# Install metrics server (if not present)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify it's working (wait ~1 min after install)
kubectl top nodes
```
#### kubectl top nodes
```sh
# Basic node resource usage
kubectl top nodes

# Output:
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node-1          250m         12%    1800Mi          45%
node-2          800m         40%    3200Mi          80%
master-node     100m         5%     900Mi           22%

# What it means:
# CPU(cores) → millicores used  (1000m = 1 full CPU core)
# CPU%       → % of node's total CPU
# MEMORY     → actual RAM used in MiB
# MEMORY%    → % of node's total RAM
```
#### Linux Bash
```sh
# Sort nodes by CPU
kubectl top nodes --sort-by=cpu

# Sort nodes by Memory
kubectl top nodes --sort-by=memory

# Show raw values (no human-readable units)
kubectl top nodes --no-headers

# Use label selector
kubectl top nodes -l node-role.kubernetes.io/worker=true
```
#### kubectl top pods
```sh
# Top pods in current namespace
kubectl top pods

# Output:
NAME                        CPU(cores)   MEMORY(bytes)
nginx-7d6b8c9f4-xk2pq       5m           32Mi
postgres-5c8b7d6f9-mn3rt     200m         512Mi
redis-6f9b8c7d4-pq1wx        10m          64Mi

# Top pods in specific namespace
kubectl top pods -n production

# Top pods across ALL namespaces
kubectl top pods -A
kubectl top pods --all-namespaces

# Top for a specific pod
kubectl top pod nginx-7d6b8c9f4-xk2pq

# Sort by CPU
kubectl top pods --sort-by=cpu

# Sort by Memory
kubectl top pods --sort-by=memory

# Show pod labels in output
kubectl top pods --show-labels

# Top pods with label selector
kubectl top pods -l app=nginx
kubectl top pods -l env=production,tier=frontend
```
#### kubectl top — Container Level
```sh
# Show individual container usage within pods
kubectl top pods --containers

# Output:
NAME                        CONTAINER     CPU(cores)   MEMORY(bytes)
nginx-7d6b8c9f4-xk2pq       nginx         5m           30Mi
nginx-7d6b8c9f4-xk2pq       log-sidecar   1m           2Mi
postgres-5c8b7d6f9-mn3rt     postgres      200m         500Mi
postgres-5c8b7d6f9-mn3rt     pgexporter    10m          12Mi

# Very useful for multi-container pods!
```
#### Practical kubectl top Scenarios
```sh
# Scenario 1: Find which node is under pressure
kubectl top nodes --sort-by=memory

# Scenario 2: Find resource-hungry pods in production
kubectl top pods -n production --sort-by=cpu

# Scenario 3: Which container in a pod uses most memory?
kubectl top pods --containers -n production | grep my-app

# Scenario 4: Full cluster resource snapshot
echo "=== NODES ===" && \
kubectl top nodes && \
echo "=== PODS (all ns) ===" && \
kubectl top pods -A --sort-by=memory

# Scenario 5: Watch pod resources live (refresh every 2s)
watch -n 2 kubectl top pods -n production

# Scenario 6: Find pods near their resource LIMITS
kubectl top pods -A --sort-by=cpu | head -20

# Scenario 7: Compare requests vs actual usage
kubectl top pods -n production
kubectl describe pod <pod-name> -n production | grep -A4 "Requests\|Limits"
```

---

### Understanding millicores (m)
```
1000m  = 1 full CPU core
500m   = 0.5 CPU core (half)
250m   = 0.25 CPU core (quarter)
100m   = 0.1 CPU core
10m    = 0.01 CPU core (very light)

# Example in a Deployment spec:
resources:
  requests:
    cpu: "250m"       # needs at least 0.25 core
    memory: "128Mi"   # needs at least 128 MB RAM
  limits:
    cpu: "500m"       # cannot exceed 0.5 core
    memory: "256Mi"   # cannot exceed 256 MB RAM
```

---

### top vs kubectl top — Side by Side
```
Feature               linux top          kubectl top
─────────────────────────────────────────────────────
Scope               single node         whole cluster
Unit of view        processes           pods/nodes
CPU unit            percentage          millicores (m)
Memory unit         KiB/MiB             MiB
Real-time           yes (live)          snapshot only
Refresh             automatic           manual/watch
Kill process        yes (k key)         no
Requires extra      no                  metrics-server
Filter by           user, PID           labels, namespace
Container level     threads (-H)        --containers flag
```