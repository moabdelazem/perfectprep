# Container Orchestration (Kubernetes)

[← Back to Index](../../README.md)

This section covers Kubernetes architecture, core concepts, workloads, networking, storage, RBAC, and troubleshooting.

---

<details>
<summary><strong>Q1: What is Kubernetes?</strong></summary>

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It provides a way to manage and run containerized applications in a distributed environment, making it easier to build, deploy, and manage containerized applications at scale.

</details>

<details>
<summary><strong>Q2: Explain the Kubernetes architecture</strong></summary>

Kubernetes follows master-worker architecture.

**Master architecture (Control Plane):**

- `API Server`: The API server is the entry point for the control plane.
- `etcd`: The etcd is the key-value store that stores the state of the cluster.
- `Controller Manager`: The controller manager is the component that manages the controllers.
- `Scheduler`: The scheduler is the component that assigns pods to nodes.

**Worker architecture (Data Plane):**

- `Kubelet`: The kubelet is the agent that runs on each node and manages the containers.
- `Container Runtime`: The container runtime is the runtime that runs the containers.
- `Kube Proxy`: The kube proxy is the network proxy that runs on each node and manages the network policies.

</details>

<details>
<summary><strong>Q3: What is the difference between a Pod and a container?</strong></summary>

A Pod is a group of containers that are scheduled together on the same node. A container is an instance of an image that is running in a Pod.

</details>

<details>
<summary><strong>Q4: Why do we need Services if pod can be reached by IP address?</strong></summary>

Pods are ephemeral; their IPs change when they restart. A Service provides a stable IP and DNS name that load-balances traffic across a set of Pods.

</details>

<details>
<summary><strong>Q5: Explain the ReplicaSet and Deployment</strong></summary>

A `Deployment` is the high-level object you interact with. It manages the `ReplicaSet`, which in turn ensures the exact number of Pods are running. Deployments allow for rolling updates and rollbacks.

</details>

<details>
<summary><strong>Q6: What are Namespaces, and why do we need them?</strong></summary>

They are virtual clusters within a physical cluster. They provide logical isolation for different teams, environments (Prod/Dev), or projects and help apply RBAC and resource quotas.

</details>

<details>
<summary><strong>Q7: Explain the difference between ConfigMap and Secret</strong></summary>

Both store configuration data, but Secret are specifically for sensitive data (passwords, tokens, etc), Secrets are only Base64 encoded by default and should be used with encryption-at-rest or external providers like HashiCorp Vault, Sealed Secrets, etc.

</details>

<details>
<summary><strong>Q8: What is difference between Deployment and StatefulSet?</strong></summary>

Deployment is for stateless applications, while StatefulSet is for stateful applications (Databases, etc).

</details>

<details>
<summary><strong>Q9: What is Headless Service?</strong></summary>

A Service with clusterIP: None. It doesn't load-balance; instead, it returns the individual IP addresses of the Pods via DNS. Often used for StatefulSets (Databases) where Pods need to talk to each other directly.

</details>

<details>
<summary><strong>Q10: Explain PV, PVC, and StorageClass</strong></summary>

These three components work together to manage persistent storage in Kubernetes:

- **PersistentVolume (PV)**: A piece of storage in the cluster that has been provisioned by an administrator or dynamically using a StorageClass. Think of it as the actual disk or storage resource (e.g., an AWS EBS volume, NFS share, or local disk). PVs have a lifecycle independent of any Pod.

- **PersistentVolumeClaim (PVC)**: A request for storage by a user. It's like a "ticket" that says "I need 10GB of fast storage." Kubernetes matches the PVC to an available PV that meets the requirements. Pods use PVCs to access storage.

- **StorageClass**: Defines the "type" of storage available (e.g., fast SSD, slow HDD, replicated, etc.). It enables **dynamic provisioning** - when a PVC is created, Kubernetes automatically creates a matching PV using the StorageClass.

**Quick Summary:**
| Component | Description |
|-----------|-------------|
| `StorageClass` | The menu of storage options (SSD, HDD, etc.) |
| `PVC` | Your order ("I want 10GB of SSD storage") |
| `PV` | The actual storage you receive |

</details>

<details>
<summary><strong>Q11: What are PVC Access Modes?</strong></summary>

PersistentVolumeClaims (PVCs) have access modes that define how a volume can be mounted by pods.

**Access Modes:**

| Mode | Abbreviation | Description |
|------|--------------|-------------|
| **ReadWriteOnce** | RWO | Volume can be mounted as read-write by a single node |
| **ReadOnlyMany** | ROX | Volume can be mounted as read-only by many nodes |
| **ReadWriteMany** | RWX | Volume can be mounted as read-write by many nodes |
| **ReadWriteOncePod** | RWOP | Volume can be mounted as read-write by a single pod (K8s 1.22+) |

**Storage Provider Support:**

| Storage Type | RWO | ROX | RWX |
|--------------|-----|-----|-----|
| AWS EBS | Yes | No | No |
| Azure Disk | Yes | No | No |
| GCE Persistent Disk | Yes | No | No |
| NFS | Yes | Yes | Yes |
| CephFS | Yes | Yes | Yes |
| HostPath | Yes | No | No |

**Example PVC:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

**When to use each mode:**

- **RWO**: Single-instance databases (PostgreSQL, MySQL), stateful applications
- **ROX**: Shared configuration files, static content served by multiple pods
- **RWX**: Shared file storage, content management systems, build artifacts
- **RWOP**: Strict single-pod access (ensures data integrity)

**Important Notes:**
- A volume can have multiple access modes, but can only be mounted using one mode at a time
- Not all storage providers support all access modes
- RWX typically requires network-attached storage (NFS, CephFS, etc.)

</details>

<details>
<summary><strong>Q12: What is Ingress?</strong></summary>

Ingress is a Kubernetes API object that manages external HTTP/HTTPS access to services within a cluster. It acts as a smart router that sits at the edge of your cluster.

**Why use Ingress instead of LoadBalancer Services?**

- A LoadBalancer Service creates one external IP per service (expensive!)
- Ingress provides a single entry point for multiple services with path-based or host-based routing

**Key Components:**

- **Ingress Resource**: The YAML configuration that defines routing rules (e.g., `/api` goes to service A, `/web` goes to service B)
- **Ingress Controller**: The actual implementation that makes Ingress work (e.g., NGINX, Traefik, HAProxy). Without a controller, Ingress resources do nothing.

**What Ingress provides:**

- Path-based routing (`example.com/api` → api-service)
- Host-based routing (`api.example.com` → api-service)
- SSL/TLS termination
- Load balancing

</details>

<details>
<summary><strong>Q12: How do Liveness, Readiness, and Startup probes differ?</strong></summary>

| Probe | Purpose |
|-------|---------|
| **Startup** | Runs first; disables other probes until the app finishes starting up |
| **Readiness** | Tells K8s when the app is ready to receive traffic (added to Service endpoints) |
| **Liveness** | Tells K8s if the app is alive. If it fails, K8s restarts the container |

</details>

<details>
<summary><strong>Q13: What are the Kubernetes services?</strong></summary>

| Service Type | Description |
|--------------|-------------|
| **ClusterIP** | Exposes the service on a cluster-internal IP address |
| **NodePort** | Exposes the service on a static port on each node's IP address |
| **LoadBalancer** | Exposes the service using a cloud provider's load balancer |

</details>

<details>
<summary><strong>Q14: What are Taints and Tolerations?</strong></summary>

Taints and Tolerations work together to **repel pods from nodes** unless they explicitly allow it.

**Taints** (applied to Nodes):
A taint marks a node as "restricted." Pods won't be scheduled on tainted nodes unless they tolerate the taint.

```bash
# Add a taint to a node
kubectl taint nodes node1 key=value:NoSchedule
```

**Taint Effects:**

| Effect | Description |
|--------|-------------|
| `NoSchedule` | Pods won't be scheduled on this node (existing pods stay) |
| `PreferNoSchedule` | Scheduler will _try_ to avoid this node, but may still schedule if needed |
| `NoExecute` | Existing pods are evicted, new pods won't be scheduled |

**Tolerations** (applied to Pods):
A toleration allows a pod to be scheduled on nodes with matching taints.

```yaml
tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

**Use Cases:**

- Dedicated nodes for specific workloads (GPU nodes, high-memory nodes)
- Preventing regular workloads from running on master nodes
- Evicting pods during node maintenance

</details>

<details>
<summary><strong>Q15: What is a DaemonSet?</strong></summary>

A DaemonSet ensures that a copy of a Pod runs on **all (or some) nodes** in the cluster. When nodes are added, Pods are automatically added to them.

**Use Cases:**

- Log collectors (Fluentd, Filebeat)
- Monitoring agents (Prometheus Node Exporter, Datadog)
- Cluster storage daemons (Ceph, GlusterFS)
- Network plugins (Calico, Weave)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
```

</details>

<details>
<summary><strong>Q16: What is the difference between Job and CronJob?</strong></summary>

| Type | Purpose |
|------|---------|
| **Job** | Runs a Pod to completion once (e.g., batch processing, database migration) |
| **CronJob** | Runs Jobs on a schedule (like cron in Linux) |

**Job Example:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      containers:
      - name: migration
        image: my-app:latest
        command: ["./migrate.sh"]
      restartPolicy: Never
  backoffLimit: 3
```

**CronJob Example:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

</details>

<details>
<summary><strong>Q17: Explain Resource Requests and Limits</strong></summary>

**Requests**: The minimum resources guaranteed to a container. Used by the scheduler to decide which node to place the Pod on.

**Limits**: The maximum resources a container can use. If exceeded, CPU is throttled; memory causes OOMKill.

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"      # 0.25 CPU cores
  limits:
    memory: "512Mi"
    cpu: "500m"      # 0.5 CPU cores
```

**Best Practices:**

- Always set requests for production workloads
- Set limits to prevent runaway processes
- `requests` = what you need; `limits` = maximum allowed

</details>

<details>
<summary><strong>Q18: What is RBAC in Kubernetes?</strong></summary>

RBAC (Role-Based Access Control) regulates access to Kubernetes resources based on the roles of users or service accounts.

**Key Components:**

| Component | Scope | Description |
|-----------|-------|-------------|
| **Role** | Namespace | Defines permissions within a namespace |
| **ClusterRole** | Cluster-wide | Defines permissions across the entire cluster |
| **RoleBinding** | Namespace | Binds a Role to users/service accounts |
| **ClusterRoleBinding** | Cluster-wide | Binds a ClusterRole to users/service accounts |

**Example:**
```yaml
# Role: Allow read-only access to pods
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
# RoleBinding: Bind role to a user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

</details>

<details>
<summary><strong>Q19: What are Network Policies?</strong></summary>

Network Policies control traffic flow between Pods (East-West traffic) and from external sources (North-South traffic). By default, all Pods can communicate with each other.

**Example: Deny all ingress traffic except from specific Pods**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

> **Note:** Requires a CNI plugin that supports Network Policies (Calico, Cilium, Weave).

</details>

<details>
<summary><strong>Q20: What is Horizontal Pod Autoscaler (HPA)?</strong></summary>

HPA automatically scales the number of Pod replicas based on observed metrics (CPU, memory, or custom metrics).

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Key Points:**

- Requires Metrics Server to be installed
- Scales based on average utilization across all Pods
- HPA takes precedence over Deployment's replica count (don't set both!)

</details>

<details>
<summary><strong>Q21: What are Init Containers?</strong></summary>

Init Containers run **before** the main application containers start. They run to completion sequentially, and the main containers only start after all init containers succeed.

**Use Cases:**

- Wait for a dependency (database, API) to be ready
- Perform setup tasks (clone a git repo, run migrations)
- Security setup (download secrets)

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 2; done']
  - name: init-data
    image: my-app:latest
    command: ['./setup.sh']
  containers:
  - name: app
    image: my-app:latest
```

</details>

<details>
<summary><strong>Q22: How do Rolling Updates work in Kubernetes?</strong></summary>

Rolling updates gradually replace old Pods with new ones, ensuring zero downtime.

**Deployment Strategy:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max extra Pods during update
      maxUnavailable: 0  # Min Pods always available
```

**How it works:**

1. Create new Pod with updated spec
2. Wait for new Pod to become Ready
3. Remove old Pod
4. Repeat until all Pods are updated

**Useful Commands:**
```bash
# Check rollout status
kubectl rollout status deployment/my-app

# View rollout history
kubectl rollout history deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to specific revision
kubectl rollout undo deployment/my-app --to-revision=2
```

</details>

<details>
<summary><strong>Q23: What is Helm and why use it?</strong></summary>

Helm is the **package manager for Kubernetes**. It packages Kubernetes manifests into reusable, versioned units called **Charts**.

**Why Helm?**

| Problem | Helm Solution |
|---------|---------------|
| Managing many YAML files | Bundle into a single Chart |
| Environment-specific configs | Use `values.yaml` for templating |
| Version control for deployments | Charts are versioned |
| Sharing configurations | Public/private Chart repositories |

**Key Concepts:**

- **Chart**: A package of Kubernetes resources
- **Release**: An instance of a Chart running in the cluster
- **Repository**: Where Charts are stored and shared

**Common Commands:**
```bash
# Install a chart
helm install my-release bitnami/nginx

# Upgrade a release
helm upgrade my-release bitnami/nginx --set replicaCount=3

# List releases
helm list

# Rollback
helm rollback my-release 1
```

</details>

<details>
<summary><strong>Q24: What are CRDs and Operators?</strong></summary>

**Custom Resource Definitions (CRDs)** extend Kubernetes with new resource types beyond built-in ones (Pods, Services, etc.).

**Operators** are applications that use CRDs to manage complex, stateful applications automatically. They encode operational knowledge (backup, upgrade, scaling) into software.

**Example: A PostgreSQL Operator might:**

- Define a `PostgresCluster` CRD
- Automatically handle replication setup
- Perform automated backups
- Handle failover and recovery

**Popular Operators:**

| Operator | Purpose |
|----------|---------|
| Prometheus Operator | Manages Prometheus instances |
| Cert-Manager | Automates TLS certificates |
| Strimzi | Manages Kafka clusters |
| Zalando Postgres Operator | Manages PostgreSQL clusters |

```yaml
# Example: Using a PostgreSQL CRD
apiVersion: acid.zalan.do/v1
kind: postgresql
metadata:
  name: my-postgres-cluster
spec:
  teamId: "myteam"
  numberOfInstances: 3
  postgresql:
    version: "15"
```

</details>

<details>
<summary><strong>Q25: What is Node Affinity and Anti-Affinity?</strong></summary>

Node Affinity controls **which nodes Pods can be scheduled on** based on node labels. It's more expressive than `nodeSelector`.

**Types of Node Affinity:**

| Type | Behavior |
|------|----------|
| `requiredDuringSchedulingIgnoredDuringExecution` | **Hard requirement** - Pod won't schedule if no matching node |
| `preferredDuringSchedulingIgnoredDuringExecution` | **Soft preference** - Scheduler tries to match but can ignore |

**Example: Required Node Affinity**
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

**Example: Preferred Node Affinity**
```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east-1a
```

**Pod Anti-Affinity** (schedule Pods away from each other):
```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: redis
        topologyKey: kubernetes.io/hostname
```

**Use Cases:**

- Run Pods on specific hardware (GPU nodes, SSD nodes)
- Spread replicas across availability zones
- Co-locate Pods for low latency (Pod Affinity)
- Separate competing workloads (Pod Anti-Affinity)

**Node Affinity vs Taints/Tolerations:**

| Feature | Node Affinity | Taints/Tolerations |
|---------|---------------|-------------------|
| Direction | Pod attracts to Node | Node repels Pod |
| Use case | "Run me on these nodes" | "Don't run anything except X" |

</details>

<details>
<summary><strong>Q26: What happens behind the scenes when you run `kubectl apply -f deployment.yaml`?</strong></summary>

Here's the complete flow from command to running Pods:

**1. Client Side (kubectl)**
```
kubectl → reads YAML → validates syntax → sends HTTP request to API Server
```

**2. API Server Processing**
```
Request → Authentication → Authorization (RBAC) → Admission Controllers → etcd
```

- **Authentication**: Verifies identity (certificates, tokens, etc.)
- **Authorization**: Checks if user can perform the action (RBAC)
- **Admission Controllers**: Mutating (modify request) → Validating (reject if invalid)
- **Persistence**: Object stored in etcd

**3. Controller Manager (Deployment Controller)**
```
Watches for Deployment changes → Creates/Updates ReplicaSet
```

**4. Controller Manager (ReplicaSet Controller)**
```
Watches for ReplicaSet changes → Creates Pod objects (status: Pending)
```

**5. Scheduler**
```
Watches for Pods with no node assigned → Selects best node → Updates Pod.spec.nodeName
```

Scheduling considers:
- Resource requests (CPU, memory)
- Node affinity/anti-affinity
- Taints and tolerations
- Pod topology spread constraints

**6. Kubelet (on the selected node)**
```
Watches for Pods assigned to its node → Pulls images → Creates containers via CRI
```

**Complete Flow Diagram:**
```
┌─────────┐     ┌────────────┐     ┌───────┐
│ kubectl │────▶│ API Server │────▶│ etcd  │
└─────────┘     └─────┬──────┘     └───────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
┌──────────────┐ ┌───────────┐ ┌─────────┐
│ Deployment   │ │ ReplicaSet│ │Scheduler│
│ Controller   │ │ Controller│ │         │
└──────┬───────┘ └─────┬─────┘ └────┬────┘
       │               │            │
       ▼               ▼            ▼
   Creates         Creates      Assigns
   ReplicaSet      Pods         Node
                                   │
                      ┌────────────┘
                      ▼
               ┌──────────┐
               │  Kubelet │──▶ Creates Container
               └──────────┘
```

</details>

<details>
<summary><strong>Q27: How does the Kubernetes Scheduler decide which node to place a Pod on?</strong></summary>

The scheduler uses a **two-phase process**: Filtering and Scoring.

**Phase 1: Filtering (Predicates)**

Eliminates nodes that cannot run the Pod:

| Filter | What it checks |
|--------|----------------|
| `PodFitsResources` | Does node have enough CPU/memory? |
| `PodFitsHostPorts` | Are required host ports available? |
| `NodeSelector` | Does node match pod's nodeSelector? |
| `NodeAffinity` | Does node satisfy affinity rules? |
| `TaintToleration` | Can pod tolerate node's taints? |
| `CheckVolumeBinding` | Can required volumes be bound? |

**Phase 2: Scoring (Priorities)**

Ranks remaining nodes (0-100 score each):

| Priority | What it favors |
|----------|----------------|
| `LeastRequestedPriority` | Nodes with most available resources |
| `BalancedResourceAllocation` | Even CPU/memory utilization |
| `NodeAffinityPriority` | Preferred affinity matches |
| `ImageLocalityPriority` | Nodes that already have container images |
| `PodTopologySpread` | Even distribution across zones/nodes |

**Final Selection:**
```
Final Score = Σ (Priority Score × Weight)
Winner = Node with highest total score
```

**Example Scenario:**
```
Pod requests: 500m CPU, 256Mi memory

Node A: 2000m free, 4Gi free → Score: 85
Node B: 800m free, 1Gi free  → Score: 45
Node C: 100m free, 512Mi free → FILTERED OUT (insufficient CPU)

Result: Pod scheduled on Node A
```

</details>

<details>
<summary><strong>Q28: How does DNS work inside a Kubernetes cluster?</strong></summary>

Kubernetes runs **CoreDNS** (or kube-dns) as a cluster add-on to provide DNS resolution.

**DNS Records Created:**

| Resource | DNS Format | Example |
|----------|------------|---------|
| Service | `<svc>.<namespace>.svc.cluster.local` | `nginx.default.svc.cluster.local` |
| Pod | `<pod-ip-dashed>.<namespace>.pod.cluster.local` | `10-244-1-5.default.pod.cluster.local` |
| Headless Service | `<pod-name>.<svc>.<namespace>.svc.cluster.local` | `web-0.nginx.default.svc.cluster.local` |

**How Pods Resolve DNS:**

1. Pod's `/etc/resolv.conf` points to CoreDNS Service IP (usually `10.96.0.10`)
2. Pod makes DNS query → CoreDNS Pod
3. CoreDNS resolves and returns IP

**Pod's /etc/resolv.conf:**
```
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

**DNS Resolution Flow:**
```
┌─────────┐  DNS Query   ┌─────────┐   Lookup    ┌───────────┐
│   Pod   │─────────────▶│ CoreDNS │────────────▶│ API Server│
│         │◀─────────────│   Pod   │◀────────────│  (etcd)   │
└─────────┘  IP Address  └─────────┘   Service   └───────────┘
                                        Endpoints
```

**Headless Services (clusterIP: None):**
- Returns Pod IPs directly instead of Service IP
- Essential for StatefulSets where each Pod needs a stable DNS name

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  clusterIP: None  # Headless
  selector:
    app: nginx
```

</details>

<details>
<summary><strong>Q29: How do you debug a Pod that keeps crashing (CrashLoopBackOff)?</strong></summary>

**Step-by-step debugging approach:**

**1. Check Pod Status and Events**
```bash
kubectl describe pod <pod-name>
# Look at: Events, State, Last State, Restart Count
```

**2. Check Container Logs**
```bash
# Current logs
kubectl logs <pod-name>

# Previous container (if restarted)
kubectl logs <pod-name> --previous

# Specific container in multi-container pod
kubectl logs <pod-name> -c <container-name>
```

**3. Common Causes and Solutions**

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| `OOMKilled` | Out of memory | Increase memory limits |
| Exit code `1` | Application error | Check logs, fix app |
| Exit code `137` | SIGKILL (OOM or timeout) | Increase resources/timeout |
| Exit code `143` | SIGTERM | Graceful shutdown issue |
| `ImagePullBackOff` | Can't pull image | Check image name, registry auth |
| Probe failures | Liveness/readiness failing | Adjust probe timing or endpoint |

**4. Interactive Debugging**
```bash
# Exec into running container
kubectl exec -it <pod-name> -- /bin/sh

# If container keeps crashing, override entrypoint
kubectl run debug --image=<image> --command -- sleep infinity
kubectl exec -it debug -- /bin/sh
```

**5. Check Resource Constraints**
```bash
kubectl top pod <pod-name>
kubectl describe node <node-name> | grep -A5 "Allocated resources"
```

**6. Debugging Checklist:**
- [ ] Image exists and is pullable?
- [ ] Entrypoint/CMD correct?
- [ ] Environment variables set correctly?
- [ ] ConfigMaps/Secrets mounted?
- [ ] Resource limits sufficient?
- [ ] Probes configured correctly?
- [ ] Dependent services available?

</details>

<details>
<summary><strong>Q30: What is etcd and why is it critical for Kubernetes?</strong></summary>

**etcd** is a distributed, consistent key-value store that serves as Kubernetes' **single source of truth**.

**What etcd Stores:**
- All cluster state (Pods, Services, ConfigMaps, Secrets, etc.)
- Cluster configuration
- Service discovery data
- Current and desired state of all resources

**Why etcd is Critical:**

| Aspect | Impact |
|--------|--------|
| **All state lives here** | If etcd fails, cluster can't function |
| **All API calls go here** | Every kubectl command reads/writes etcd |
| **Distributed consensus** | Uses Raft algorithm for consistency |
| **Watch mechanism** | Controllers use watches to react to changes |

**Architecture:**
```
┌────────────┐    ┌────────────┐    ┌────────────┐
│   etcd-1   │◀──▶│   etcd-2   │◀──▶│   etcd-3   │
│  (Leader)  │    │ (Follower) │    │ (Follower) │
└─────┬──────┘    └────────────┘    └────────────┘
      │
      ▼
┌────────────┐
│ API Server │ (only talks to etcd)
└────────────┘
```

**Key Characteristics:**

- **Strongly consistent**: All reads return latest write
- **Highly available**: Tolerates (n-1)/2 failures in n-node cluster
- **3 nodes minimum** for production HA
- **5 nodes** for large clusters (tolerates 2 failures)

**etcd Operations:**
```bash
# Check etcd health
etcdctl endpoint health

# List all keys
etcdctl get / --prefix --keys-only

# Backup etcd
etcdctl snapshot save backup.db

# Restore from backup
etcdctl snapshot restore backup.db
```

**Best Practices:**
- Run on dedicated nodes (not with workloads)
- Use SSD storage for low latency
- Regular automated backups
- Monitor etcd metrics closely

</details>

