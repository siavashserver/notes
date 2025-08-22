---
title: Kubernetes
---

## Introduction: The Role of Kubernetes in Modern Container Orchestration

Kubernetes, frequently abbreviated as **K8s**, has emerged as the de facto
standard for container orchestration in both enterprise and cloud-native
environments. It abstractly provides a robust, automated, scalable platform for
deploying, managing, and monitoring containerized applications.

### Why Container Orchestration?

The rapid rise of containerization also introduced operational challenges:
managing hundreds or thousands of ephemeral containers, handling networking,
storage, rolling upgrades, resource allocation, scaling, security, and more.
Manual or ad-hoc orchestration is not feasible at scale—this is the gap
Kubernetes was created to fill.

### Challenges Addressed by Kubernetes

Kubernetes brings the following **key solutions** to containerized environments:

- **Automated container scheduling and lifecycle management:** Pods
  automatically placed on optimal nodes; automated restart, repair, and scaling.
- **Declarative, versioned infrastructure:** Configuration is managed as code
  with YAML manifests and versioned APIs, promoting reproducibility.
- **Service discovery and load balancing:** Exposes stable endpoints for
  ephemeral workloads; dynamically balances traffic.
- **Self-healing and resiliency:** Failed containers are restarted
  automatically; workload is rescheduled on node failures.
- **Separation of concern:** Operators manage infrastructure, developers focus
  on application definitions.

These foundational strengths explain Kubernetes' broad adoption in diverse
environments, spanning cloud, edge, development, and production.

---

## Kubernetes Distributions: Minikube, K3s, and MicroK8s

While Kubernetes' architecture is standardized, many distributions make it
easier to deploy and test clusters suited for development, edge, or
resource-constrained scenarios. Here is a comparative overview of three popular
lightweight Kubernetes solutions:

| Distribution | CNCF Certified | Arch. Support            | Multi-node | Auto-HA | Updates | Add-ons | Target Use Cases                |
| ------------ | -------------- | ------------------------ | ---------- | ------- | ------- | ------- | ------------------------------- |
| Minikube     | Yes            | x86, ARM, ppc, s390x     | No         | No      | Manual  | Yes     | Local dev, testing, education   |
| K3s          | Yes            | x86, ARM64, ARMv7        | Yes        | Yes     | Yes     | No      | Edge, IoT, prod micro-clusters  |
| MicroK8s     | Yes            | x86, ARM64, s390x, POWER | Yes        | Yes     | Yes     | Yes     | Dev/prod clusters on any device |

---

## Kubernetes Architecture: Control Plane and Node Components

Kubernetes operates as a **distributed system** comprising a control plane
(cluster management) and worker nodes (container execution).

### High-Level Architecture

- **Control Plane:** Responsible for making global decisions (scheduling,
  scaling, health) and managing the cluster state. Can be highly-available
  (multi-node) for production.
- **Worker Nodes:** Machines (VMs/physical hosts) executing application Pods
  according to control plane directives.

#### Core Control Plane Components

| Component                | Responsibility                                                |
| ------------------------ | ------------------------------------------------------------- |
| kube-apiserver           | Exposes Kubernetes API, validates and updates resources       |
| etcd                     | Consistent, distributed key-value store for cluster state     |
| kube-scheduler           | Selects optimal node for unscheduled Pods                     |
| kube-controller-manager  | Runs controllers that drive actual state toward desired state |
| cloud-controller-manager | Integrates with cloud-provider APIs (optional)                |

#### Worker Node Components

| Component         | Role                                                                         |
| ----------------- | ---------------------------------------------------------------------------- |
| kubelet           | Agent on node, ensures containers are running and healthy on that node       |
| kube-proxy        | Maintains network rules for Service load balancing                           |
| Container runtime | Executes containers as directed by kubelet (e.g., containerd, CRI-O, Docker) |

In most clusters, each component can be horizontally scaled and flexibly
deployed.

### Key Control Plane Components Explained

#### API Server (kube-apiserver)

Acts as the **front end** for the control plane, exposing the RESTful Kubernetes
API. All operations on resources—pods, nodes, configmaps, etc.—pass through the
API server, which handles authentication, authorization, admission control, and
request validation. It persists all cluster state in etcd.

#### etcd

A strongly-consistent, distributed key-value store that holds all cluster data.
Only one control plane component (the API server) talks directly to etcd. Its
reliability is essential; if etcd is down, no cluster changes (even scheduling)
can occur, though existing pods may continue running.

#### Scheduler (kube-scheduler)

Detects Pods needing assignment and chooses the best worker node for each,
considering resource requirements, affinity/anti-affinity, taints/tolerations,
and placement policies. Schedulers support custom plugins for advanced placement
logic.

#### Controller Manager (kube-controller-manager)

Runs a suite of **controllers**—loops responsible for reconciling actual and
desired cluster state. Common controllers include those for nodes (health
monitoring), jobs, endpoints, serviceaccounts, ReplicaSets, and more. Runs as a
single binary but operates many logically separate controllers.

#### Kubelet

The node agent running on every worker node, responsible for maintaining
container health per the PodSpecs assigned by the scheduler. The kubelet
monitors resource consumption, mounts volumes, handles secrets, and reports
node/pod status to the API server.

#### Kube-proxy

Maintains network rules on the node. It allows network communication to/from
Pods inside or outside the cluster, implementing Service load balancing and
session affinity via tools like iptables, IPVS, or userspace proxies.

#### Kubectl and Kubeconfig

- **kubectl**: The primary CLI tool for cluster management, resource creation,
  querying, and debugging.
- **kubeconfig**: Configuration file holding cluster, user, and access
  credentials. kubeconfig allows seamless cross-cluster context switching and is
  essential for CI/CD pipelines and multi-tenant environments.

---

## Relationships: Clusters, Nodes, Namespaces, Pods, Containers

Understanding Kubernetes resource relationships is vital for efficient design
and troubleshooting.

### Cluster

The highest-level grouping, representing the entire Kubernetes installation
(control plane + nodes). A cluster manages all associated resources.

### Node

A VM or physical host within the cluster. Nodes are **workers** executing Pods.
Nodes register with the control plane, report health, and are managed by the
kubelet. Each cluster has at least one node.

### Namespace

A virtual cluster, helping organize resources and control access within a single
Kubernetes cluster. Namespaces support multi-tenancy, isolation, and resource
quotas. Namespaced resources include pods, deployments, services, configmaps,
and secrets.

### Pod

**Pods** are the fundamental unit of deployment—wrappers for one or more tightly
coupled containers that share storage, network, and lifecycle. Each Pod is
assigned a unique cluster-internal IP. **Containers within the same Pod can
reach one another via localhost and share mounted volumes**. Most workloads use
one container per Pod, but multi-container Pods (e.g., sidecar patterns) provide
additional architectural power.

### Container

**Containers** are individual processes bundled with their runtime environment.
In Kubernetes, containers are run within Pods, not directly on nodes. Each
container is isolated, lightweight (shares host OS kernel), and portable.
Containers run microservices, backends, job workers, and more.

**Comparison Table: Pods vs. Containers**

| Aspect     | Pods                                  | Containers                   |
| ---------- | ------------------------------------- | ---------------------------- |
| Definition | Unit of scheduling in Kubernetes      | Package of app code and deps |
| Scope      | May house ≥1 container(s)             | Runs app logic inside a Pod  |
| IP Address | One per Pod, shared by all containers | Shares Pod IP and localhost  |
| Storage    | May mount shared/exclusive volumes    | Accesses volumes via Pod     |
| Lifecycle  | Managed by Kubernetes via controllers | Managed within parent Pod    |

---

## Workload Controllers: Ensuring Availability and Declarative Management

Kubernetes controllers manage Pods' lifecycle, ensuring the desired state is
preserved despite failures or workload scaling demands.

| Controller  | Functionality                                              | Typical Use                 |
| ----------- | ---------------------------------------------------------- | --------------------------- |
| ReplicaSet  | Ensures a specified number of identical Pods run           | Single, stateless           |
| Deployment  | Manages ReplicaSets, supports rolling updates/rollback     | Stateless web apps          |
| StatefulSet | For stateful apps needing stable IDs/storage               | Databases, Kafka, etc.      |
| DaemonSet   | Runs a copy of a Pod on every (or select) node             | Log collectors, sys daemons |
| Job         | Runs Pods to completion for batch jobs                     | ETL, one-off tasks          |
| AutoBurst   | (Emerging) Handles automatic pod/replica burst for traffic | Traffic spikes              |

### ReplicaSet

**Purpose:** Maintains a stable set of replica Pods, primarily for stateless
deployments or as an implementation detail of Deployments. **Key config**:
`.spec.replicas`, `.spec.selector`, `.spec.template`. **YAML Example:**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
```

Scaling is managed by editing `replicas`. ReplicaSets are rarely created
directly; use Deployments instead for rollouts and rollback.

### Deployment

**Purpose:** High-level controller, providing declarative updates, rollbacks,
scaling, and auto-replacement. **YAML Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
```

Deployments are the workhorse for modern stateless applications, supporting
zero-downtime upgrades via rolling updates and version history for easy
rollback.

### StatefulSet

**Purpose:** Manages Pods with **stable, unique identifiers** and **stable
storage**. Critical for databases, queues, and legacy stateful applications.

- Provides ordered deployment and scale-out/down.
- Each Pod can have a persistent volume claim. **YAML Example:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mydb
spec:
  serviceName: "mydb"
  replicas: 3
  selector:
    matchLabels:
      app: mydb
  template:
    metadata:
      labels:
        app: mydb
    spec:
      containers:
        - name: mysql
          image: mysql:8
          volumeMounts:
            - name: www
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

StatefulSet ensures Pod names, storage, and network IDs persist through restarts
and rescheduling.

### DaemonSet

Installs a copy of a Pod on all or a specific subset of nodes (for example, with
node selectors or tolerations). Typical for log shippers, monitoring agents, and
system services.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
spec:
  selector:
    matchLabels:
      name: log-collector
  template:
    metadata:
      labels:
        name: log-collector
    spec:
      containers:
        - name: fluentd
          image: fluentd:latest
```

### Job

Controls **batch workloads**, running Pods to completion. Supports retry/backoff
logic, completions, and parallel execution.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-job
spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl
          command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

Jobs can be suspended, resumed, or made “indexed.” CronJobs are a variant for
periodic execution.

### AutoBurst

**AutoBurst** manages replicas in response to unpredictable bursts of demand,
allowing applications to rapidly scale up and down. This controller is
especially critical in modern, event-driven architectures and is implemented
using HPA (Horizontal Pod Autoscaler) and, in some environments, custom burst
control plugins.

---

## Service Types: ClusterIP, NodePort, LoadBalancer, Ingress, ExternalName

Services provide a stable, virtual endpoint for accessing Pods, abstracting
ephemeral Pod IPs and supporting service discovery, load balancing, and external
routing.

| Type         | Description                                    | YAML Field                           | Use Case                               |
| ------------ | ---------------------------------------------- | ------------------------------------ | -------------------------------------- |
| ClusterIP    | Default; exposes internally within the cluster | `type: ClusterIP`                    | Internal microservices                 |
| NodePort     | Exposes on static high port on each node       | `type: NodePort`, `nodePort: <port>` | Bare metal, dev, direct ext. access    |
| LoadBalancer | Uses ext. cloud LB to expose public IP         | `type: LoadBalancer`                 | Production ingress (cloud)             |
| Ingress      | HTTP/HTTPS routing, multi-path, domain-based   | `kind: Ingress`                      | Web apps, L7 routing, cert termination |
| ExternalName | Maps against external DNS/CNAME                | `type: ExternalName`                 | Legacy, hybrid, or multi-cluster conn. |

### **YAML & Usage Examples**

- **ClusterIP** (default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  type: ClusterIP
```

- **NodePort**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: np-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

- **LoadBalancer**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

- **ExternalName**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-database
spec:
  type: ExternalName
  externalName: db.example.com
```

To create: `kubectl create service externalname my-svc --external-name
external.address`

### **Ingress Resource**

Ingress exposes HTTP/HTTPS endpoints, acting as an L7 proxy with features like
SSL termination, path- and host-based routing, and can handle VirtualHosts and
fanout routing. Requires an active Ingress controller.

**Ingress YAML Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

See [Kubernetes docs on
Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) for
more on advanced host/path, TLS, and resource backends.

#### **Service Discovery and Cross-Namespace Communication**

- Pods access services in their own namespace via `<service-name>`.
- To connect across namespaces (e.g., from `frontend` in `ns-a` to `backend` in
  `ns-b`): use `backend.ns-b` or fully qualified DNS
  `backend.ns-b.svc.cluster.local`.
- Ingress/Gateway API can route traffic across namespaces for internal and
  shared gateway setups.

---

## Routing: Outside Cluster, Cross-Namespace, and Load Balancing

### How Does Traffic Flow from Outside to Containers?

1. **LoadBalancer Service**: Requests hit a cloud provider's external load
   balancer IP, which sends to designated NodePorts. From there, kube-proxy
   routes to target Pods via round-robin or session affinity.
2. **NodePort**: Requests made to `<nodeIP>:<nodePort>` are routed via
   kube-proxy to Service endpoints.
3. **Ingress**: An Ingress controller exposes (often on port 80/443) and proxies
   HTTP(S) traffic based on rules to appropriate backend Services, which then
   load-balance to Pods.
4. **Gateway API**: A more advanced, extensible L4–L7 programmable interface for
   traffic management, supporting cross-namespace routing, mTLS, split control,
   etc. Gateway listener can restrict allowed namespaces or routes for precise
   delegation.

### Load Balancing Methods in Kubernetes

Kubernetes supports:

- **Random or round-robin** kube-proxy load balancing across healthy Pod
  endpoints.
- **Session affinity** (ClientIP) where repeated connections from same source IP
  go to same Pod.
- **External (Cloud LB) routing** honors health checks and supports advanced
  algorithms.
- **Topology-aware routing**: Prefer cluster zone-local or node-local Pods to
  minimize latency on multi-zone clusters.

---

## Kubernetes Pod Patterns: Init Containers, Sidecar, Ambassador, and Adapter

Complex applications benefit from common patterns for multi-container Pods.

### **Init Containers**

Init containers run before main app containers—executing initialization scripts,
waiting for dependencies, or generating config files. They must complete
successfully for the main containers to start.

**YAML Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
    - name: wait-for-db
      image: busybox
      command: ["sh", "-c", "until nc -z db-service 5432; do sleep 2; done"]
  containers:
    - name: main
      image: my-app-image
```

Init containers help enforce ordering, environment readiness, and isolate
dependency installations.

### **Sidecar Pattern**

A sidecar is a container that runs alongside the main application, extending or
enhancing its functionality (e.g., log shipping, metrics, service mesh proxies).

**Example: Fluentd/Log Collector**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
    - name: app
      image: my-app
      volumeMounts:
        - mountPath: /var/log
          name: logs
    - name: log-collector
      image: fluentd
      volumeMounts:
        - mountPath: /var/log
          name: logs
  volumes:
    - name: logs
      emptyDir: {}
```

Sidecars share the Pod’s IP and volumes, allowing close, efficient interaction.

### **Ambassador Pattern**

An ambassador acts as a proxy between the main container and the external
network. Use cases include proxying to external APIs, terminating SSL, or
protocol translation.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-ambassador
spec:
  containers:
    - name: app
      image: my-app
    - name: ambassador
      image: envoy
      args: [...] # Configure Envoy to forward requests
```

### **Adapter Pattern**

Adapters transform or enrich output from the main app container into a different
format, often for metrics or protocol bridging.

Example: Nginx exposing its /nginx_status via a Prometheus exporter adapter for
metrics scraping.

---

## Volumes: emptyDir, hostPath, PersistentVolume, PersistentVolumeClaim

Kubernetes volumes handle container storage needs beyond ephemeral container
filesystems.

| Volume Type                 | Scope     | Description & Use Cases                                                             |
| --------------------------- | --------- | ----------------------------------------------------------------------------------- |
| emptyDir                    | Pod       | Temporary storage; deleted when Pod deleted. Shared by containers in Pod.           |
| hostPath                    | Node      | Mounts host filesystem dir/file into Pod. Node-affinity required. Use with caution. |
| PersistentVolume (PV)       | Cluster   | Abstracts storage implementation (NFS, EBS, etc.); lifecycle outside Pod.           |
| PersistentVolumeClaim (PVC) | Namespace | Request for PV resource. Pod consumes storage via PVC, not PV directly.             |

**emptyDir Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cache-pod
spec:
  containers:
    - name: app
      image: my-app
      volumeMounts:
        - name: cache
          mountPath: /cache
  volumes:
    - name: cache
      emptyDir:
        sizeLimit: 1Gi
```

**hostPath Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
    - name: app
      image: my-app
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      hostPath:
        path: /mnt/data
        type: Directory
```

**Security Note:** Avoid hostPath in production unless absolutely necessary; it
can break container isolation and pose privilege risks.

**PersistentVolume and PersistentVolumeClaim:**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 3Gi
```

Pods reference PVCs to bind storage:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: pvc1
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: data
```

---

## ConfigMaps and Secrets: Configuration and Secret Management

Decoupling configuration from code is a pillar of the Twelve-Factor App—and
Kubernetes provides **ConfigMaps** and **Secrets** for non-secret and secret
data respectively.

### **ConfigMap**

An API object to inject non-sensitive configuration data (env vars, CLI args,
files) into Pods. ConfigMaps can be mounted as volumes or consumed as
environment variables.

**Creation:**

```bash
kubectl create configmap game-config --from-file=game.properties
```

**YAML Example:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  database_url: "postgres://db.example.com"
```

**Usage in Pod:**

```yaml
env:
  - name: DATABASE_URL
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: database_url
```

### **Secret**

API object for storing sensitive data such as passwords, tokens, or keys
(base64-encoded). Mounted as volumes or injected as environment variables.
Supports image pull secrets and integration with secret management platforms.

**YAML Example:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  password: cGFzc3dvcmQ= # "password"
```

**Usage in Pod:**

```yaml
env:
  - name: PASSWORD
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: password
```

### **Best Practices**

- Use **RBAC** with least privileges for Secrets.
- Always enable **encryption at rest** for secrets in etcd.
- For production, consider external secret managers (HashiCorp Vault, cloud KMS)
  and use CSI drivers or controllers for integration.

---

## Health-Probes: Liveness, Readiness, and Startup Probes

Kubernetes offers **probes** for diagnostics and container orchestration:

| Probe Type | Purpose                  | Restart? | Typical Failure Response   |
| ---------- | ------------------------ | -------- | -------------------------- |
| Liveness   | Is the app stuck?        | Yes      | Container restarts         |
| Readiness  | Ready to serve requests? | No       | Unready, removed from svc  |
| Startup    | Is the app initialized?  | No       | Liveness/readiness blocked |

**Example: Liveness Probe**

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

**Startup probe** should be used for apps with slow initialization. Tune
thresholds and delays to match app behavior.

---

## Monitoring and Observability: Prometheus Integration

**Prometheus** is the most widely adopted monitoring solution for Kubernetes. It
scrapes metrics from endpoints annotated in pods/services, supports multilayer
scraping (node, cluster, app), and supports alerting via Alertmanager. Grafana
is typically used for dashboards.

**Prometheus in Kubernetes:**

- Deploy as Deployment/StatefulSet, with persistent storage.
- Scrapes `/metrics` endpoints.
- RBAC: Grant Prometheus service account access to API objects via roles and
  rolebindings.

**YAML Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  ...
```

Prometheus Operator automates installation/management, and Thanos or Cortex
provide multi-cluster and long-term storage.

**Exporters (e.g., node-exporter) and kube-state-metrics** enrich monitoring
capability.

---

## Interview Questions

### How are StatefulSets different from Deployments?

StatefulSets offer stable, predictable Pod names and persistent storage using
unique volume claims and ordered scheduling/terminations, essential for stateful
applications. Deployments focus on stateless, fungible Pods and do not provide
identity or sticky storage beyond Pod restarts.

### How can containers in different namespaces communicate?

Use the full service DNS: `<service>.<namespace>.svc.cluster.local`, or simply
`<service>.<namespace>`. By default, no network isolation exists between
namespaces unless network policies are enforced.

### Explain how to debug CrashLoopBackOff in a Pod.

Check pod events/logs with `kubectl describe pod <pod>` and `kubectl logs
<pod>`. Investigate failed readiness/liveness probes, resource limits, volume
mounts, image pulls, or missing configuration. Resolve root cause, patch, and
restart deployment.

### What is the difference between ClusterIP, NodePort, and LoadBalancer services?

- ClusterIP exposes the service within the cluster only.
- NodePort exposes via a fixed external port on each node.
- LoadBalancer provisions an external load balancer (in cloud), mapping to a
  NodePort and ClusterIP service.

### Describe the use of configMaps and secrets in CI/CD pipelines.

configMaps populate non-sensitive settings (URIs, feature flags), while secrets
supply credentials, tokens, or database passwords. Both are mounted or injected
into Pods at runtime, supporting different environments with the same deployment
manifest, plus encrypted and RBAC-limited access for secrets.

### How is persistent data managed in Kubernetes clusters?

PersistentVolume (PV) abstracts storage backend; PersistentVolumeClaim (PVC) is
how users request storage. StorageClasses define provisioning behaviors and
underlying technology. StatefulSets automate volume claims for per-pod
persistent data.

### How does Kubernetes perform service-level load balancing?

Kubernetes services use kube-proxy to set up iptables/IPVS rules, distributing
requests across healthy endpoints (Pods). LoadBalancer services rely on external
L4 LBs. Advanced algorithms (session affinity, topology-aware routing) can be
used.

### How does the Horizontal Pod Autoscaler (HPA) work?

HPA automatically scales pod replicas based on observed metrics (CPU, memory, or
custom) by comparing current values to your target. HPA controller periodically
queries metrics, computes desired replicas, and updates resource specs
accordingly.

---

## Fine-Grained Container Placement in Kubernetes

This capability is not delivered by default scheduling alone but rather by a
robust suite of mechanisms that enable fine-grained control over _which
containers (pods)_ run _on which nodes_ within the cluster.

## Kubernetes Scheduler Architecture and Scheduling Phases

### Overview of the kube-scheduler

Kubernetes scheduling is responsible for **deciding on which node an unscheduled
Pod will run**. The core entity is `kube-scheduler`, running as part of the
control plane.

The kube-scheduler executes a **two-phase process** for each pending Pod:

1. **Filtering phase:** The scheduler filters out nodes that do not meet the
   Pod's basic requirements. These requirements can include resource requests,
   node selectors, affinity/anti-affinity, taints/tolerations, etc. Only the
   nodes that pass all these constraints are considered feasible.
2. **Scoring phase:** Each feasible node is given a score, which is a composite
   of criteria such as resource availability, spread, locality, etc. The Pod is
   bound to the node with the highest score.

### Scheduling Sequence: Key Steps

1. **Pod creation:** A user or controller (e.g., Deployment) creates a Pod
   object without a specified node assignment.
2. **Pending state:** The API server marks the Pod as Pending for scheduling.
3. **Scheduler picks Pod:** kube-scheduler selects the unscheduled Pod from the
   queue.
4. **Filtering:** The feasible nodes are identified (filtering out those failing
   resource or policy checks).
5. **Scoring:** The remaining nodes are scored; affinity, anti-affinity, and
   spread constraints may influence this.
6. **Binding:** The scheduler "binds" the Pod to the chosen node (by updating
   `spec.nodeName`).
7. **Kubelet starts container:** On the assigned node, the kubelet pulls images
   and starts the containers.

**Constraints** like affinity, taints, and selectors participate in both
Filtering and Scoring phases.

---

## Node Selection Mechanisms: Theory and YAML Examples

### Node Selectors

#### What is a Node Selector?

The **nodeSelector** is the simplest form of node selection constraint. It is
specified as a map of key/value pairs that must exactly match the node's labels.

**YAML example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-worker
spec:
  containers:
    - name: workload
      image: nginx
  nodeSelector:
    disktype: ssd
```

To use this, you must first label the relevant node:

```bash
kubectl label nodes <node-name> disktype=ssd
```

**Behavior:**

- Pods will _only_ schedule onto a node matching _all_ the specified labels.
- No ordering or preference is specified; only a rigid match is accepted.
- If no suitable nodes exist, the Pod will remain in Pending.

**Real-World Use Cases:**

- Assign workloads to nodes with specific hardware (SSD, GPU).
- Separate environments (e.g., `env=production`, `env=dev`).
- Quickly dedicate nodes for specialized needs.

**Best Practices and Limitations:**

- Node selectors are simple and effective but inflexible; only strict matches
  are possible.
- Cannot specify logical operators, multiple alternatives, or soft preferences.
- Does not support spreading, distribution, or dynamic fallback.

---

### Node Affinity

#### What is Node Affinity?

**NodeAffinity** extends nodeSelector functionality with more expressive
configuration. It allows you to specify both "hard" (required) and "soft"
(preferred) placement constraints, supports operators (In, NotIn, Exists), and
is generally more flexible.

**Key types:**

- **requiredDuringSchedulingIgnoredDuringExecution**: _Hard_ affinity, must
  match at scheduling time.
- **preferredDuringSchedulingIgnoredDuringExecution**: _Soft_ preference,
  scheduler will try but fallback if needed.

**YAML example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-worker
spec:
  containers:
    - name: workload
      image: my-ml-task
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: gpu
                operator: In
                values:
                  - "true"
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 10
          preference:
            matchExpressions:
              - key: environment
                operator: In
                values: ["production"]
```

**Capabilities:**

- Operators: In, NotIn, Exists, DoesNotExist, Gt, Lt
- Multiple expressions (AND within a nodeSelectorTerm, OR across
  nodeSelectorTerms)
- Soft (weighted) and hard (mandatory) constraints
- Can build complex logic, e.g., `disk=ssd` AND `region IN [us-west, eu]`

**Real-World Use Cases:**

- Only schedule ML workloads to specific GPU-enabled, high-memory, or regional
  nodes.
- Prefer but do not require certain zones/regions for improved latency or cost.

**Best Practices:**

- Prefer nodeAffinity over nodeSelectors for future-proofing.
- Combine required and preferred settings for resiliency across environments.
- Use expressive operators for easier administration.

**Limitations:**

- Does not _repel_ workloads (unlike taints).
- Does not allow for dynamic adjustments after Pods are scheduled
  ("IgnoredDuringExecution").

---

### Pod Affinity and Anti-Affinity

#### What are Pod Affinity and Anti-Affinity?

**Pod affinity** allows pods to be scheduled on nodes where specific other pods
(identified by label) are already running. **Pod anti-affinity** works as a
repulsion, preventing placement on nodes hosting certain other pods.

These settings are particularly useful for **workload colocation (e.g., to
minimize latency between cooperating services) or high availability (e.g.,
prevent all replicas from landing on the same node/zone).**

**Types:**

- **requiredDuringSchedulingIgnoredDuringExecution**: Hard; must be satisfied.
- **preferredDuringSchedulingIgnoredDuringExecution**: Soft; try but fallback
  allowed.

**Key concepts:**

- `topologyKey`: The label key used to determine the scope of the affinity
  ("kubernetes.io/hostname" for node-level, AZ labels for zone-level).
- `labelSelector`: Matches target pods for affinity or anti-affinity.

**Pod Affinity YAML:**

Pod affinity to co-locate frontend and backend:

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - backend
        topologyKey: "kubernetes.io/hostname"
```

**Pod Anti-Affinity YAML:**

Distribute replicas across nodes:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - my-app
        topologyKey: "kubernetes.io/hostname"
```

**Advantages and Use Cases:**

- Enforce anti-colocation for higher-availability deployments (e.g., stateful
  apps).
- Colocate pods for high network throughput or low-latency interactions (e.g.,
  web app and cache).
- Distribute stateless replicas for load balancing, minimize "blast radius" of
  failures.

**Limitations and Considerations:**

- Pod affinity/anti-affinity can be expensive to compute in very large clusters
  (hundreds of nodes).
- All nodes must have the specified `topologyKey` label; a mislabelled node can
  result in unscheduled pods.
- Can combine both affinity and anti-affinity for sophisticated layouts.
- Pod anti-affinity can be more deterministic for spreading than podAffinity for
  grouping.

---

### Taints and Tolerations

#### What are Taints and Tolerations?

- **Taints** are applied to nodes and _repel pods_ unless the pods have a
  matching **toleration**. This is the inverse to node affinity, acting as a
  node's "keep out" filter.

**Taint Command Example:**

```bash
kubectl taint nodes node1 gpu=present:NoSchedule
```

- Key: `gpu`
- Value: `present`
- Effect: `NoSchedule` (others: `PreferNoSchedule`, `NoExecute`)

**Pod Toleration YAML Example:**

```yaml
tolerations:
  - key: "gpu"
    value: "present"
    effect: "NoSchedule"
    operator: "Equal"
```

**Behavior:**

- Without a toleration matching the node's taint, the Pod _cannot_ be scheduled
  to the node.
- With a matching toleration, the Pod can be scheduled.
- Does not _guarantee_ scheduling on that node—only _allows_ it.
- With `NoExecute`, the Pod will be evicted from the node upon violation.

**Common Use Cases:**

- Reserve nodes for specific teams, workloads, or hardware (e.g., GPU,
  high-memory).
- Perform maintenance by repelling generic workloads and ensuring only
  privileged pods or DaemonSets run.
- Protect control-plane nodes from general-purpose workloads (masters are
  typically tainted by default).

**Interaction and Combination:**

- Combine taints and tolerations with node affinity for stricter placement.
- Use together with node affinity to provide both attract and repel (see
  advanced patterns below).

**Best Practices:**

- Be intentional with taint usage to prevent orphaned workloads.
- Use tolerations sparingly to avoid accidental cross-placement.

---

### Topology Spread Constraints

#### What are Topology Spread Constraints?

**Pod topology spread constraints** allow you to express rules about how Pods of
a given workload should be distributed across a topology domain—such as nodes,
zones, or regions—to avoid imbalance and minimize risk of failure.

**Key Fields:**

- `maxSkew`: Maximum allowed difference in the number of matching pods between
  the most and least loaded topology domain.
- `topologyKey`: Node label to define failure domains (e.g.,
  `kubernetes.io/hostname`, `topology.kubernetes.io/zone`).
- `whenUnsatisfiable`: What to do if the constraint cannot be satisfied
  (`DoNotSchedule`, `ScheduleAnyway`).
- `labelSelector`: Which pods are considered in the spread computation.

**YAML Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spread-pod
spec:
  topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: kubernetes.io/hostname
      whenUnsatisfiable: DoNotSchedule
      labelSelector:
        matchLabels:
          app: frontend
  containers:
    - name: nginx
      image: nginx
```

**Real-World Use Cases:**

- Ensuring replicas of a workload are evenly spread by node, zone, or rack.
- Meeting specific SLOs for balancing workloads to prevent single-node or AZ
  overload.
- Achieving compliance or business continuity by spreading across failure
  domains.

**Comparison: Pod Anti-Affinity vs. Topology Spread:** While pod anti-affinity
can accomplish similar objectives (e.g., keeping replicas off the same node),
topology spread constraints are often more efficient and expressive for even
distribution and can work at higher topology levels (such as region/zone).

---

### Manual Pod Assignment: nodeName

#### What is `nodeName`?

By setting `.spec.nodeName` in a Pod, you are directly assigning the Pod to a
specific node—bypassing the scheduler entirely.

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-assigned-pod
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: node-foo
```

**Implications:**

- No filtering or placement logic is applied.
- Danger: Pod will _fail_ if the node does not exist or cannot accept the Pod
  (e.g., insufficient resources or taints).
- Most useful in special cases, such as debugging or with custom operator logic.

**Best Practices:**

- Avoid in general-purpose production workloads.
- Prefer affinity/selector mechanisms for sustainable, scalable placement.

---

### DaemonSets for Per-Node Pods

#### What is a DaemonSet?

A **DaemonSet** ensures that **one Pod (or a subset) runs on every (or every
matching) node in a cluster**.

**Typical Use Cases:**

- Node-local system agents (monitoring, logging, CNI).
- Storage or infrastructure tasks (e.g., persistent logging collectors).
- Security agents and node-level proxies.

**YAML Skeleton:**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-monitor
spec:
  selector:
    matchLabels:
      name: node-monitor
  template:
    metadata:
      labels:
        name: node-monitor
    spec:
      containers:
        - name: monitor
          image: datadog/agent
      nodeSelector:
        monitoring: enabled
```

**Targeting Specific Nodes:**

- Use `nodeSelector`, `nodeAffinity` and/or `tolerations` in the DaemonSet pod
  template to limit to a subset of nodes.
- DaemonSets can run on tainted nodes; often have necessary tolerations by
  default.

**Scheduling Behavior:**

- DaemonSet controller places pods directly; then default scheduler binds them
  to the intended node.
- DaemonSets tolerate node taints such as
  `node.kubernetes.io/not-ready:NoExecute` by default.

---

### Resource Requests and Limits: Scheduler Impact

**Resource requests** (`resources.requests`) define the _minimum_ amount of
CPU/memory a Pod requires, and the scheduler only considers nodes with enough
free resources for those requests.

**Resource limits** (`resources.limits`) define the _maximum_ a container can
use but are **not considered during scheduling**—only enforced at runtime.

**Example:**

```yaml
resources:
  requests:
    cpu: 500m
    memory: 256Mi
  limits:
    cpu: 1
    memory: 1Gi
```

**Scheduler Behavior:**

- Only requests are taken into account for bin-packing; limits are not.
- Pods will remain Pending if no node meets resource _requests_, even if limits
  would fit.

**Use Cases:**

- Control overcommit and guarantee resource isolation for critical workloads
- Prevent noisy-neighbor impact

**Caveats:**

- Always specifying realistic requests prevents unschedulability and inefficient
  resource fragmentation.
- Lack of limits can lead to "hogging" by misbehaving workloads.

---

### Priority Classes and Pod Preemption

**PriorityClass** resources let you assign a relative priority to pods,
influencing scheduling when resource contention occurs.

- **Pods with higher priorities can preempt (evict) lower-priority pods** when
  necessary to secure placement.
- Enables robust SLO/slo enforcement, critical-service protection, and batch
  workload optimization.

**YAML Example:**

Define a PriorityClass:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "Used for critical services"
```

Assign to a pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-service
spec:
  priorityClassName: high-priority
  ...
```

**Preemption Policy:** By default, `PreemptLowerPriority` is used; you may
specify `Never` to prevent preemption.

**Implications:**

- Maintaining cluster health requires careful assignment and monitoring of
  priorities, to avoid cascades of preemption or starvation of lower-priority
  workloads.
- Preemption respects PodDisruptionBudgets unless overridden for very
  high-priority pods.

---

### Custom Schedulers and schedulerName

Kubernetes supports running **multiple schedulers** in a cluster. By specifying
`schedulerName` in your Pod spec, you may direct scheduling to a custom (or
third-party) scheduler instance.

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: specialized-workload
spec:
  schedulerName: custom-scheduler
  containers:
    - name: app
      image: my-specialized-image
```

**Use Cases:**

- Specialized bin-packing, business logic, or hardware considerations outside
  the default scheduler's capabilities.
- Experimental or research-driven algorithm development (e.g., batch schedulers
  for HPC, job orchestrators).
- Integration with domain-specific resources or frameworks (e.g., Volcano for
  ML/HPC workloads).

**Cautions:**

- Custom schedulers must correctly manage leader election, Pod binding, and
  status reconciliation.
- Only one scheduler will schedule a Pod; others ignore Pods not labeled for
  them.
- Overlapping scheduler configurations can cause confusion if not
  well-documented.

---

## Combining Mechanisms for Fine-Grained Scheduling

### Matrix of Interactions

The mechanisms discussed above are **not mutually exclusive** and can (and often
should) be combined to express sophisticated scheduling policies and isolation.
The following table summarizes their core functions, overlap, and best-use
combinations.

| **Mechanism**        | **Key Purpose**                             | **Policy Type**       | **Interaction**                                      | **Best Use/Notes**                                                                        |
| -------------------- | ------------------------------------------- | --------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| nodeSelector         | Directly attract pod to specific nodes      | Hard constraint       | AND'd with affinity/taints                           | Simple static matching. Combine with taints/tolerations for single-tenant isolation.      |
| nodeAffinity         | Expressive, flexible node matching          | Hard/soft             | AND'd with other selectors and tolerations           | Use for hardware, region constraints. Prefer over nodeSelector for future flexibility.    |
| podAffinity/anti-    | Attract or repel based on pod presence      | Hard/soft             | AND'd with node constraints                          | Fine-tune colocation/spanning of related workloads; expensive in large clusters.          |
| taints & tolerations | Repel/allow pods to nodes                   | Repulsion (hard/soft) | AND'd with affinity                                  | Use to exclude groups of workloads from nodes. Combine with affinity for dedicated nodes. |
| topologySpread       | Even distribution across domains            | Hard/soft             | Evaluated after filtering, impacts scoring           | Modern alternative to podAntiAffinity for distribution or HA.                             |
| nodeName             | Override (static placement)                 | Always (hardest)      | Ignores scheduler; bypasses normal constraints       | For debugging or custom logic only. Use with caution in production.                       |
| DaemonSet            | Automate per-node deployment                | Controller logic      | Uses selectors, affinity, tolerations for targeting  | For per-node singleton pods (monitoring, logging, CNI, etc.)                              |
| PriorityClass        | Scheduling preference and eviction ordering | Global                | Can preempt lower-priority pods if schedule blocked  | Prevent starvation of critical workloads; caution with globalDefault                      |
| schedulerName        | Use custom scheduler                        | Controller/external   | Must be paired with custom scheduling implementation | For specialized workloads, batch scheduling, or research                                  |

**When combining mechanisms:**

- All constraints (selectors, affinity, tolerations) must be satisfied for
  placement.
- Taints act as negative filters, affinity/selectors as positive; combine for
  exclusive node pools.
- Use preferred rules where absolute assignment is undesirable to avoid
  scheduling starvation.
- Beware of conflicting rules—Pods may remain forever Pending if constraints are
  too restrictive.

---

## YAML Walkthrough: Combined Scenario

**Scenario:** Suppose you have nodes dedicated to GPU ML workloads, and you must
ensure only GPU jobs run there, spread evenly across nodes. Other workers must
not land on GPU nodes, and each replica should land on a distinct node.

- Label nodes: `hardware=gpu` (for GPU nodes), `hardware=general` (for others).
- Taint nodes: `kubectl taint nodes nodeX special=gpu:NoSchedule`
- Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ml-task
  labels:
    app: ml-worker
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: hardware
                operator: In
                values: ["gpu"]
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values: ["ml-worker"]
          topologyKey: "kubernetes.io/hostname"
  tolerations:
    - key: "special"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
  containers:
    - name: worker
      image: my-gpu-image
      resources:
        requests:
          cpu: 2
          memory: 8Gi
```

This:

- Attracts (only) to nodes labelled as GPU.
- Requires multi-node spreading (one per node).
- Only tolerates (can be scheduled onto) GPU-tainted nodes.
- Cannot accidentally run on non-GPU or general nodes, nor can general workloads
  run on tainted GPU nodes.

---

## Advanced Patterns and Best Practices

1. **Layer constraints for absolute control**: Combine taints/tolerations
   (exclude non-matching pods) with node affinity (attract correct pods) for
   isolation.
2. **Use topology spread constraints for resilient workloads**: Especially where
   zone outages are a concern or regulation requires distribution.
3. **Avoid over-constraining**: Excessive hard rules (required
   affinities/taints) can easily block scheduling; always test in staging.
4. **Monitor scheduling outcomes using `kubectl describe pod <pod>`**: Review
   `Events` for FailedScheduling and debug via node labels, resource reporting.
5. **Label and taint management automation**: Use tools (e.g., Cluster API,
   Kubeadm, cloud provider APIs) for consistency—mislabeling nodes is a common
   cause of scheduling errors.
6. **Document scheduling policies**: The complexity and interactions can trip up
   even experienced practitioners; maintain up-to-date design docs.

---

## Summarizing Table: Feature Comparison

| Feature                          | Placement Type         | Positive/Negative Matching | Hard/Soft | Expressiveness       | Maintenance Level | Primary Use Cases                                  |
| -------------------------------- | ---------------------- | -------------------------- | --------- | -------------------- | ----------------- | -------------------------------------------------- |
| nodeSelector                     | Node-only              | Positive                   | Hard      | Simple (exact match) | Easy              | Basic node targeting, quick setups                 |
| nodeAffinity                     | Node-only              | Positive                   | Both      | High                 | Medium            | Hardware, region/zone/label-based matching         |
| podAffinity/podAntiAffinity      | Node (via pods)        | Pos/Neg (colocate/spread)  | Both      | High                 | High              | Distributed HA, service colocation                 |
| taints/tolerations               | Node-only              | Negative                   | Both      | Medium               | Medium            | Isolation, maintenance, special nodes              |
| topologySpreadConstraints        | Domain (node/zone/etc) | Spread/Even-ness           | Both      | High                 | Medium            | Resilient HA, platform compliance, cost management |
| nodeName                         | Node-only              | Hard-overriding            | Hardest   | None (manual)        | Manual            | Debugging, admin override, static assignment       |
| DaemonSet                        | Node-specific          | Follows spec/labels        | Hard      | Controlled by spec   | Automatable       | Node agents, infra daemons, system pods            |
| PriorityClass/Preemption         | All                    | Scheduling precedence      | N/A       | N/A                  | Policy-side       | SLOs, resource triage, batch/critical separation   |
| schedulerName (custom scheduler) | All                    | Fully programmable         | N/A       | As coded             | High              | Specialized, research, strongly custom scheduling  |

---
