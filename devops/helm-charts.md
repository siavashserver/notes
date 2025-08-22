---
title: Helm Charts
---

## Introduction

Helm charts are **packaged collections of Kubernetes resources** bundled with
templating and configuration files. Think of them as the Kubernetes equivalent
of Debian’s DEB or RedHat’s RPM packages, but specially designed to create,
deploy, and manage complex app stacks, including their configuration,
dependencies, and versioning.

**Key concepts:**

- **Chart**: The main packaging unit, comprising templates, default config, and
  chart metadata.
- **Release**: A running instance of a chart, deployed to a cluster (each
  install creates a release).
- **Repository**: A central location or registry for publishing and discovering
  charts (e.g., Artifact Hub).
- **Values**: Parameters injected into templates at install time, driving
  reusability and environment-specific deployments.

**High-level overview:**

- **Reusable**: Templates allow parameterization, so a single chart is usable
  across dev, staging, and prod.
- **Versioned**: Both the chart and each release are versioned, supporting
  rollback and traceability.
- **Composed**: Charts can depend on other charts, supporting modular
  microservice architectures.

---

## Problems Solved by Helm Charts

Kubernetes’ native configuration model relies on static and often duplicated
YAML manifests. In real-world environments, this leads to several operational
headaches:

### 1. Managing Large-Scale, Dynamic Configurations

Without Helm:

- Teams copy and edit hundreds of nearly identical YAML files for different
  environments and services.
- Subtle differences cause **configuration drift**, resulting in environment
  inconsistencies.

With Helm:

- Centralized **values files** allow easy overrides, reducing duplication and
  errors.
- **Templates** ensure environment differences are explicit and manageable.

### 2. Consistency and Portability

Without Helm:

- Manual changes are error-prone.
- Deploying the same app to multiple clusters or environments requires bespoke
  manifests.

With Helm:

- A chart encapsulates application logic and makes it portable across clusters.
- Rollout and changes are scripted and tracked via Helm commands.

### 3. Dependency and Lifecycle Management

Without Helm:

- Managing service dependencies (like a database for your app) requires manual
  coordination.
- Rollbacks need hunting down old manifests.

With Helm:

- **Dependencies** are encoded in Chart.yaml, and Helm automatically
  pulls/install dependencies.
- **Rollback** and **release history** are built in (e.g., `helm rollback
myrelease 2`).

### 4. GitOps and CI/CD Integration

Helm supports modern workflows where infrastructure and app configs are managed
via version control systems like Git. Its declarative, idempotent commands fit
seamlessly into CI/CD pipelines and GitOps strategies.

---

## Helm Architecture and Components

Modern Helm (v3+) adopts a **client-only architecture**:

- **Helm CLI (`helm`)**: The command-line interface for all user actions:
  install, upgrade, rollback, lint, etc.
- **Helm Library**: A Go library to render charts, merge values, and interact
  with the Kubernetes API.

**Architecture improvements in Helm 3:**

- Removed Tiller (the now obsolete server-side component used in Helm 2),
  greatly improving cluster security and operational simplicity.
- Release data is now stored as secrets in the cluster, in the same namespace as
  the release, supporting namespace isolation and RBAC.
- Leverages Kubernetes-native role-based access control (RBAC), supporting least
  privilege security models out-of-the-box.

**Deployment workflow:**

1. User runs `helm install` with a chart and values.
2. Helm renders the templates with the given values, producing Kubernetes
   objects.
3. The generated manifests are applied to the cluster using the Kubernetes API.
4. Helm tracks the release and its revision for future upgrades or rollback.

---

## Key Components of a Helm Chart

A well-formed Helm chart includes the following structure:

```plaintext
my-chart/
├── Chart.yaml             # Metadata about the chart
├── values.yaml            # Default parameter values
├── charts/                # Dependencies (subcharts)
├── templates/             # Kubernetes manifests with templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl       # Helper template functions
│   └── NOTES.txt          # Post-install instructions
├── .helmignore            # Patterns for files/directories to exclude
├── README.md              # Documentation (optional but recommended)
└── crds/                  # Custom Resource Definitions (optional)
```

Each component is described below in detail.

---

### 1. Chart.yaml

The **Chart.yaml** file defines the chart’s metadata, including:

- `apiVersion`: Chart API version (`v2` for Helm 3+)
- `name`: Chart name
- `description`: Short description (optional but recommended)
- `version`: The chart’s version (semver, e.g., `1.2.0`)
- `appVersion`: Upstream application version (e.g., Docker image tag)
- `dependencies`: List of subcharts/dependencies
- `maintainers`, `icon`, `sources`: Optional metadata for documentation and
  repository indexing

**Example:**

```yaml
apiVersion: v2
name: my-api
description: A Helm chart for MyAPI microservice
version: 1.0.0
appVersion: "2.1.3"
dependencies:
  - name: postgresql
    version: 10.2.0
    repository: https://charts.bitnami.com/bitnami
maintainers:
  - name: Alice DevOps
    email: alice@example.com
icon: https://example.com/icon.png
sources:
  - https://github.com/example/my-api
```

---

### 2. values.yaml

This is the **default configuration file**—parameterizing templates for
customization and safe overrides.

**Intended usage:**

- Chart authors fill in common, non-sensitive defaults (e.g., image name,
  resource limits).
- At installation or upgrade, users can override these values via the command
  line (`--set key=value`) or with custom YAML files (`-f values-prod.yaml`).

**Example snippet:**

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.19"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: "200m"
    memory: "256Mi"
  requests:
    cpu: "100m"
    memory: "128Mi"

env:
  LOG_LEVEL: info
```

---

### 3. templates/

All **Kubernetes resources** (Deployment, Service, Ingress, ConfigMaps, etc.)
are defined here as template files. These files make extensive use of the Helm
Go templating engine, supporting variable expansion, conditionals, loops, and
helper functions.

**Template syntax example (`deployment.yaml`):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: { { include "my-api.fullname" . } }
  labels: { { - include "my-api.labels" . | nindent 4 } }
spec:
  replicas: { { .Values.replicaCount } }
  selector:
    matchLabels: { { - include "my-api.selectorLabels" . | nindent 6 } }
  template:
    metadata:
      labels: { { - include "my-api.selectorLabels" . | nindent 8 } }
    spec:
      containers:
        - name: { { .Chart.Name } }
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: { { .Values.image.pullPolicy } }
          resources: { { - toYaml .Values.resources | nindent 12 } }
          env:
            - name: LOG_LEVEL
              value: "{{ .Values.env.LOG_LEVEL }}"
```

**Key points:**

- Variables use the double-brace `{{ ... }}` syntax.
- Functions (`include`, `toYaml`, `nindent`, etc.) come from Helm’s Go template,
  Sprig library, or are user-defined.
- The `include` function is commonly used to reuse named templates from
  `_helpers.tpl`.

---

### 4. charts/

This directory houses dependencies or **subcharts**—charts bundled with or
referenced by the parent chart. For instance, a web app might bundle a Redis or
PostgreSQL subchart.

- Dependencies are declared in Chart.yaml.
- When packaging or installing, `helm dependency update` will pull these as
  needed.

---

### 5. \_helpers.tpl

A special template file (usually `_helpers.tpl`) for DRY (Don’t Repeat Yourself)
helper snippets, such as label selectors and resource naming conventions.

**Example helper function:**

```yaml
{{- define "my-api.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" -}}
{{- end }}
```

**Usage in another template:**

```yaml
metadata:
  name: { { include "my-api.fullname" . } }
```

---

### 6. NOTES.txt

Any **NOTES.txt** in the templates/ directory is rendered (using Helm templates)
and displayed to the user after a successful release. It’s perfect for
post-install instructions, tips, or URLs.

---

### 7. .helmignore

Analogous to .gitignore, this file lists files or patterns to exclude from
packed charts (e.g., test data, local settings).

**Example:**

```
*.tgz
*.bak
.DS_Store
```

---

### 8. crds/

A dedicated folder for **Custom Resource Definitions** needed prior to rendering
or installing custom resources.

---

## Understanding YAML Syntax and Helm Templating

### Helm Template Syntax Tour

Helm leverages the Go template engine. Here are the fundamental template
features:

#### Variable Substitution

```yaml
replicas: {{ .Values.replicaCount }}
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

#### Default Values and Functions

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}"
```

#### Helpers and Includes

```yaml
metadata:
  name: { { include "mychart.fullname" . } }
```

#### Conditionals

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
...
{{- end }}
```

#### Loops

```yaml
env:
{{- range $key, $value := .Values.env }}
  - name: {{ $key }}
    value: {{ $value | quote }}
{{- end }}
```

#### Pipelines and Type Conversion

```yaml
resources: { { - toYaml .Values.resources | nindent 8 } }
```

#### Quoting and YAML Folding

- Use `quote` for strings with potential special characters: `value: {{
.Values.token | quote }}` - Multiline strings: `def: |`, `summary: >`

#### Comments in Templates

```yaml
# This is a comment in YAML
{ { /* This is a template comment and will NOT appear in the output */ } }
```

---

## Helm Chart Usage Scenarios

Charts are not just for deploying monolithic applications. They shine
particularly in these common scenarios:

### 1. Deploying Applications

**Use case:** Single-command deployment of web apps, databases, and monitoring
stacks.

**Example:** Install Bitnami NGINX with default or custom config:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-nginx bitnami/nginx
helm install custom-nginx bitnami/nginx -f custom-values.yaml
```

### 2. Managing Configuration

Charts centralize configuration and enable parameterization:

- **Per-environment values:** `-f values-prod.yaml`, `-f values-dev.yaml`
- **Override specific values:** `--set image.tag=v1.2.3`
- **Dynamic template variables:** Custom resource composition, e.g., cycling
  through a list of secrets.

**Advantage:** Single chart, N environments, zero drift.

### 3. Versioning, Upgrade, and Rollback

**Lifecycle operations:**

- Upgrade: `helm upgrade myapp ./mychart -f values-new.yaml`
- View history: `helm history myapp`
- Roll back to version N: `helm rollback myapp N`
- Uninstall: `helm uninstall myapp`

**Benefit:** Safe, reversible deploys—critical for production reliability.

### 4. Dependency Management and Modular Apps

- **Umbrella charts:** Compose multiple services as subcharts (e.g., web app +
  Redis + DB).
- **Share standard components:** Internal library charts for org-wide patterns.

**Automated:** `helm dependency update` fetches and packages dependencies.

### 5. GitOps and CI/CD

Helm is natively scriptable and integrates well with Argo CD, FluxCD, and
workflow tools such as Jenkins and GitHub Actions. Declarative configurations
(charts and values) are tracked in Git and deployed automatically.

---

## Common Helm Chart Usage Scenarios

| Scenario                         | Example Command(s)                                   | Description/Benefit                             |
| -------------------------------- | ---------------------------------------------------- | ----------------------------------------------- |
| Deploying default app            | `helm install myapp ./mychart`                       | Single command deploy; uses values.yaml         |
| Deploy with environment override | `helm install myapp ./mychart -f values-prod.yaml`   | Environment-specific values                     |
| On-demand config change          | `helm upgrade myapp ./mychart --set image.tag=1.2.5` | Zero-downtime rolling update                    |
| Rollback broken deployment       | `helm rollback myapp 2`                              | Instantly revert to revision 2                  |
| Fetch dependency updates         | `helm dependency update ./mychart`                   | Downloads/upgrades subchart versions            |
| Render manifests only            | `helm template myapp ./mychart`                      | Dry-run, output manifests to std. out           |
| Test pre-install logic           | `helm lint ./mychart`                                | Lints templates and values; IDs warnings/errors |
| Share/publish chart              | `helm package ./mychart` then push to repo           | Makes charts reusable across teams & orgs       |

---

## Managing Configuration, Versioning, and Rollback with Helm

Helm is engineered for safe, consistent, and automated application lifecycle
management on Kubernetes.

### Deployments

- **Install:** `helm install myrelease ./mychart`
- **Upgrade:** `helm upgrade myrelease ./mychart -f new-values.yaml`
- **Dry-run:** `helm install --dry-run --debug ...` for manifest preview/testing

### Configuration

- **Override per deployment:** `helm install ... -f env-specific.yaml` or `--set
key=value`
- **Multiple environments:** Store separate values files (values-dev.yaml,
  values-prod.yaml, etc.)

### Versioning and Rollback

1. Each `upgrade` or install creates a new **revision**.
2. **View history:** `helm history myrelease`
3. **Rollback:** `helm rollback myrelease <revision>` — reverts manifest and
   parameters to previous revision, incrementing release revision for repeatable
   audit and recovery.

**Advantage:**

- Fast, atomic rollback in case of broken updates.
- Full deployment history and audit trace.

---

## Interview Questions

### What is the role of values.yaml in a Helm Chart?

The `values.yaml` file provides default configuration parameters for chart
templates. When a chart is deployed, Helm reads this file and exposes the values
as `.Values` in each template, enabling parameter substitution and dynamic
configuration. Users can override values with custom files (`-f`) or CLI flags
(`--set`). This supports scalability, reuse, and environment-specific config.

### Write a Helm template snippet to generate a Deployment resource with values substitution.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: { { .Values.deployment.name } }
spec:
  replicas: { { .Values.deployment.replicas } }
  template:
    spec:
      containers:
        - name: { { .Values.deployment.name } }
          image: { { .Values.deployment.image } }
          ports:
            - containerPort: { { .Values.deployment.port } }
```

### How do you manage dependencies in a Helm chart?

Dependencies are defined in the `dependencies` field of `Chart.yaml`. You can
specify subchart name, version, and repository. Upon running `helm dependency
update`, Helm fetches the subcharts and populates the `charts/` directory.
Parameter overrides for subcharts are set via `values.subchartname.key` in the
parent’s values.yaml.

### How would you perform a rollback in Helm?

Use `helm rollback <release> <revision>`. First, inspect release history with
`helm history <release>`. Pick the desired revision, then execute the rollback
command. Helm re-applies the manifests and values from that revision and
increments the release revision for traceability (i.e., rollbacks are versioned
too).

### How do you override default chart values at install time?

You can pass a custom values file with `-f <filename.yaml>`, or override
specific values directly with `--set key=value`. These have higher precedence
than values.yaml: `--set` > custom file(s) > values.yaml.

### What is a Helm hook and why is it used?

Helm hooks are annotations added to manifest templates to run specific resources
at key points in the Helm release lifecycle, e.g., pre-install, post-upgrade,
pre-delete. They’re used for database migrations, backup jobs, or custom actions
needed before/after install/upgrade. Hooks are declared using annotations like
`helm.sh/hook: pre-install` in Kubernetes manifests.

### Difference between `helm install` and `helm upgrade`.

`helm install` deploys a new release (first-time install). `helm upgrade`
updates an existing release with new chart version or config; if the release
does not exist, use `helm upgrade --install` for idempotent CI/CD workflows.

### Can Helm charts be used as libraries?

Yes. In Helm 3, a chart’s `type` can be set to `library`, meaning it provides
helper templates but cannot be installed on its own. Application charts can then
`depend` on library charts for reusable functions and blocks.

### How does Helm ensure consistent, reproducible deployments?

By versioning every release, locking dependencies (`Chart.lock`), using
declarative templating, and supporting overrides, Helm ensures that any
deployment or rollback is repeatable and auditable.

### How do you manage secrets or sensitive data in Helm charts?

Do **not** store secrets in plain-text values.yaml. Use Kubernetes Secrets, or
integrate with Helm plugins like `helm-secrets` or operators such as Bitnami
Sealed Secrets. Values can reference secrets for injection into pods.

### What is the purpose of `_helpers.tpl`?

`_helpers.tpl` is a template file for DRY, reusable snippet functions. These can
be imported into any resource template via `include`, enabling consistent
labels, naming, and templating conventions.
