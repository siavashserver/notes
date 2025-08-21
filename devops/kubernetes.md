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
