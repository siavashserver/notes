---
title: 12-Factor App
---

## Introduction: Origin, Purpose, and Enduring Relevance of the 12-Factor Method

In today’s rapidly evolving landscape of web and cloud-based software delivery,
creating applications that are **scalable, resilient, maintainable, and
portable** has become paramount. The **12-Factor App methodology**, originally
published by Adam Wiggins and the Heroku team in 2011, was conceived out of this
necessity. Drawing upon firsthand experience developing, deploying, and
maintaining hundreds of SaaS (Software-as-a-Service) projects, the authors
distilled twelve essential principles for building modern web applications,
especially those intended for deployment on cloud platforms like Heroku, AWS,
Azure, and beyond.

---

## I. Codebase: One Codebase Tracked in Revision Control, Many Deploys

**Essential Principle**: Maintain a single codebase per application, tracked in
version control, and deploy that codebase to multiple environments (development,
staging, production, etc.).

A **codebase** is often defined as the _entire set of files, scripts, and
resources that make up an application_, typically stored in a source control
repository like Git, Subversion, or Mercurial. The codebase’s singularity
ensures it is the _single source of truth_ for all environments.

**Key Best Practices**:

- Store all application assets (including infrastructure-as-code, automation
  scripts, and configuration files) in a repository, accessible to all
  stakeholders.
- Prohibit multiple codebases for what you consider a single app. If you have
  multiple codebases, you have a distributed system—a violation of the
  principle. Shared code should be abstracted as libraries managed via
  dependency managers, not through multiple app codebases.
- Each deploy, whether for development, QA, staging, or production, represents
  an _instance_ (“deploy”) of the same app, tied to a particular snapshot
  (“commit”) of the codebase.

**Benefits**:

- **Consistency**: All environments derive from an identical codebase, reducing
  “it works on my machine” issues.
- **Traceability**: Changes can be associated with individual commits,
  supporting debugging, rollback, and accountability.
- **CI/CD Support**: Automation tools can reliably build, test, and deploy the
  app across environments using the same codebase.

**Modern Context**: With the increased adoption of microservices, the guidance
is that each microservice should maintain its own dedicated codebase, even if
some functionality is shared. That shared logic should be extracted as versioned
libraries, not duplicated across services.

---

## II. Dependencies: Explicitly Declare and Isolate Dependencies

**Essential Principle**: Applications must _explicitly_ declare and _isolate_
all dependencies.

Relying on the implicit presence of libraries or system tools in your deployment
environment is a source of unexpected bugs and inconsistencies. Instead, use
dependency managers and package manifests (e.g., `package.json` in Node.js,
`requirements.txt` in Python, `pom.xml` in Java) to:

- **Declare all dependencies** (third-party packages, libraries, tools),
  including exact versions.
- **Isolate dependencies** so that the app does **not leak or inherit unwanted
  packages from the system**. This enables _deterministic builds_ and ensures
  the app runs the same everywhere.

**Best Practices**:

- Never bundle external dependencies directly with the codebase; always
  reference via manifest files.
- Use virtual environments, containers (Docker), or language-specific isolation
  tools (virtualenv for Python, rbenv for Ruby, etc.) to ensure complete
  dependency isolation.
- Refrain from relying on system-level binaries (e.g., `curl`, `ffmpeg`) unless
  they are included as explicit dependencies (via scripting, Dockerfiles, or
  vendoring).

**Benefits**:

- **Onboarding**: New developers need only the language runtime and the
  dependency manager; they can bring up the environment consistently via a
  build/install command.
- **Deployment consistency**: Minimize “works on my box” problems arising from
  environmental drift or missing libraries.

**Modern Context**: Tools like Docker, Poetry (Python), Bundler (Ruby), and
Maven (Java) embody this principle. Cloud build systems also enforce strict
dependency isolation for reliable deployments.

---

## III. Config: Store Configuration in the Environment

**Essential Principle**: _Store all configuration that varies between
environments outside the codebase, preferably in environment variables_.

- **Config** is anything that can vary between deployments: database URIs,
  credentials, API endpoints, secret keys, etc.
- **Never hardcode configuration or secrets in the source code**. This allows
  you to open-source code or transfer responsibility without risking security
  breaches.

**Best Practices**:

- Inject configurations at runtime using environment variables or external
  config files (not tracked in source control).
- Tools like Docker Compose, Kubernetes ConfigMaps/Secrets, and cloud
  environment variables make this process seamless.
- Avoid grouping config into named profiles (“dev”, “prod”); each deploy should
  be configurable independently with granular, orthogonal variables.

**Example**:

```sh
export DATABASE_URL="postgres://user:pass@host:5432/db"
export SMTP_SERVER="smtp.example.com"
```

In application code:

```python
import os
db_url = os.environ['DATABASE_URL']
```

**Benefits**:

- **Security**: Sensitive values are not accidentally checked into repositories.
- **Flexibility**: Switch environments, credentials, or endpoints without code
  changes.
- **Portability**: The same codebase runs unchanged in every environment; only
  the config differs.

**Real-World Tip**: Use secret management tools (e.g., HashiCorp Vault, AWS
Secrets Manager) for sensitive configs and inject them as environment variables
at launch.

---

## IV. Backing Services: Treat Backing Services as Attached Resources

**Essential Principle**: **All services the app consumes over the network are
“attached resources”**—treat all databases, caches, queues, email systems, or
external APIs as interchangeable backing services.

Applications should make no distinction between local and third-party backing
services. This abstraction enables powerful changeability and flexibility:

- _A local MySQL database can be swapped out for Amazon RDS with no code
  changes—only the resource handle in config needs to change_.
- _A local Redis could be replaced by a managed Redis service by changing a
  single environment variable_.

**Best Practices**:

- Configure all access credentials and endpoints for backing services via
  environment variables/config; never hardcode within codebase.
- Swapping, adding, or detaching services should not require code changes—just
  config changes.
- Treat every service—database, cache, messaging, logging, even third-party
  APIs—as loosely coupled and external.

**Benefits**:

- **Resilience and Portability**: Deploys can be moved, scaled, or restored by
  attaching/detaching resources freely.
- **Testability**: Easier to simulate outages or failovers by swapping service
  endpoints.

**Modern Cloud Practice**: Managed services (such as AWS S3, Google Cloud SQL,
or SendGrid) fit perfectly into this paradigm, and IaC (Infrastructure as Code)
tools like Terraform help define resources declaratively.

---

## V. Build, Release, Run: Strictly Separate Build and Run Stages

**Essential Principle**: **Separate the build, release, and run stages in your
app’s lifecycle**. Each deployable instance is created via these distinct
stages:

- **Build**: Converts codebase into an executable bundle (build artifacts) by
  fetching dependencies, compiling, and packaging.
- **Release**: Combines build artifacts with configuration, forming an immutable
  release ready for deployment.
- **Run**: Executes the application’s code in the execution environment using
  the specified configuration.

**Best Practices**:

- Releases should be uniquely identified (timestamp, semantic version, etc.) and
  immutable; changes should always create a new release.
- Never allow code changes at runtime—ensure changes go through the build stage.
- Deploy automation (CI/CD pipelines) commonly embodies this principle, e.g.,
  Jenkins, GitHub Actions, GitLab CI.

**Benefits**:

- **Consistency and Repeatability**: The same release artifact can be deployed
  to any environment, simplifying rollback and troubleshooting.
- **Separation of Concerns**: Different teams or stages in your pipeline can
  inspect and approve builds, releases, and runs independently.

**Modern Examples**:

- **Docker** images encapsulate builds; deployments select which tagged build to
  run.
- Cloud platforms (Heroku, AWS CodePipeline) encapsulate these phases for easy
  management.

---

## VI. Processes: Execute the App as One or More Stateless Processes

**Essential Principle**: **Apps should run as one or more stateless
processes**—no persistent state is stored in-process or on the application’s
local disk.

- Every execution of your app—be it a web request, batch job, or background
  task—is handled by a _stateless_ process.
- **Stateful data** (session, user uploads, persistent jobs) must be placed in
  external backing services or storage (databases, object stores, caches).

**Best Practices**:

- Do not use “sticky sessions” (assigning a user to a specific app instance);
  session information should be in distributed stores like Redis.
- In-memory or disk-local caches should be used only for temporary, not
  persistent, data.
- Processes can be freely restarted, scaled up or down, or replaced without loss
  of critical data.

**Benefits**:

- **Scalability and Fault Tolerance**: Stateless processes can be scaled
  horizontally; failures do not cause data loss.
- **Maintainability**: Rolling deployments, dynamic scaling, and blue/green
  deployments become easier.

**Modern Implementation**: Statelessness is enforced by serverless platforms
(AWS Lambda), containers (Docker), and orchestrators (Kubernetes). Session
stores and databases are externalized.

---

## VII. Port Binding: Export Services via Port Binding

**Essential Principle**: **App processes should export services via port
binding**, making them fully self-contained and independent of an external web
server in the runtime environment.

- Rather than running behind a preconfigured web server (e.g., Apache hosting
  PHP scripts), an app should embed its own web server or networking component,
  listening on a port defined by the environment (commonly via `PORT` env var).
- This practice enables modern routing layers (load balancers, reverse proxies)
  to direct external traffic as needed.

**Best Practices**:

- Use environment variables to assign the listening port (do not hardcode port
  numbers in code).
- Ensure your app runs the same way in all environments—development, staging,
  and production should all rely on port binding.
- Your service is composable: it can serve as a backing service for others if
  needed (service chaining).

**Benefits**:

- **Portability**: Any platform where the language runtime is available can run
  your app.
- **Simplicity**: Less dependency on platform-specific server management.
- **Integration**: Facilitates container orchestration and service discovery
  systems, crucial for microservices.

**Example (Node.js/Express):**

```javascript
const port = process.env.PORT || 3000;
app.listen(port, () => console.log(`App listening on port ${port}`));
```

---

## VIII. Concurrency: Scale Out via the Process Model

**Essential Principle**: **Applications scale by starting more process
instances**, not by increasing resources for a single process (“vertical
scaling”).

- Different types of work (web requests, background jobs, scheduled tasks) are
  managed by separate process types, not via threads inside a monolithic
  application.
- The “process formation” specifies what types and how many processes should be
  running for the current workload.

**Best Practices**:

- Compose your app of multiple process types (web, worker, scheduler, etc.),
  each able to run multiple instances as needed.
- Use operating-system or platform-level process managers (such as systemd,
  Kubernetes, PM2, or Foreman) to orchestrate process scaling.
- Avoid daemonizing or writing PID files within application code; delegate
  management to the execution environment.

**Benefits**:

- **Flexible Scaling**: Scale components independently as load changes (add more
  workers for job queues, more web processes under heavy traffic).
- **Reliability**: If a process type is overwhelmed, scale only that process and
  not the entire app.

**Modern Context**: Tools like Kubernetes, Docker Swarm, and cloud platforms
orchestrate these process formations, providing elasticity at the level of
independent containers or pods.

---

## IX. Disposability: Maximize Robustness with Fast Startup and Graceful Shutdown

**Essential Principle**: **Application processes must be disposable—easy and
quick to start or stop, and resilient to unexpected crashes**.

- **Fast Startup**: Processes should start in seconds, not minutes. This enables
  rapid scaling or redeployment.
- **Graceful Shutdown**: On receiving termination signals (SIGTERM), processes
  should cease new work, finish what’s in progress, and then exit cleanly.
  Workers should safely return tasks to queues, web processes should finish
  requests, etc.
- **Crash-Only Philosophy**: Apps should handle crashes as normally as clean
  shutdowns; process failures are part of cloud-native operation.

**Best Practices**:

- Use robust queueing backends to handle unfinished jobs if a process dies
  unexpectedly.
- Release resource locks, close DB connections, and flush logs during shutdown.
- Design jobs to be idempotent, so repeat execution after a crash does not cause
  inconsistent state.

**Benefits**:

- **Agility**: Ability to rapidly deploy or roll back with minimal downtime.
- **Robust Recovery**: Improved resilience to hardware, orchestration, or
  software failures.

**Modern Extensions**: Kubernetes liveness and readiness probes, health checks,
and orchestration readiness hooks automate disposability. Most serverless
platforms (like AWS Lambda) enforce fast startup (cold start) requirements.

---

## X. Dev/Prod Parity: Keep Development, Staging, and Production as Similar as Possible

**Essential Principle**: **Maintain minimal divergence between development,
staging, and production environments**.

- Historically, large gaps in deployment time, personnel, and tool versions have
  led to bugs and operational headaches (the infamous “works on my machine”
  problem).
- Enforced parity prevents bugs by surfacing them in development, not
  production.

**Best Practices**:

- Reduce the _time gap_: Enable continuous integration/continuous deployment so
  code is deployed within minutes or hours, not weeks.
- Reduce the _personnel gap_: Developers should be closely involved in
  deployment; operations engineering should not be a separate silo.
- Reduce the _tools gap_: Use containerization (Docker), VMs, or scripts to
  ensure dependencies and config match across all environments.
- All environments should use the same backing services (e.g., if production
  uses PostgreSQL, so should local and staging environments).

**Benefits**:

- **Predictability**: Fewer surprises when moving code to production.
- **Confidence for Continuous Delivery**: Automated tests and parity encourage
  rapid, reliable releases.
- **Simplified Debugging**: Issues found in development mimic those found in
  live environments.

**Modern Implementations**: Containerization technologies (Docker, Kubernetes)
and config management (Ansible, Chef, Puppet) are now central in maintaining
parity. The rise of IaC and CI/CD pipelines automate and enforce this across
teams.

---

## XI. Logs: Treat Logs as Event Streams

**Essential Principle**: **Treat logs as time-ordered event streams, written to
stdout/stderr, not as files managed by the application**.

- The app does _not_ attempt to manage log files, archival, or routing.
- Instead, the execution environment (container platform, PaaS, or server OS)
  captures the output and routes it to log processing tools (aggregators,
  dashboards, alerting systems).

**Best Practices**:

- Standardize your application’s logging output as unbuffered lines to
  stdout/stderr.
- Use log aggregation services like ELK Stack, Splunk, Graylog, or cloud-native
  offerings like AWS CloudWatch Logs or Google Stackdriver to collect, index,
  and analyze logs.
- Employ structured log formats (JSON, key-value pairs) where possible for
  better parsing.

**Benefits**:

- **Observability**: Real-time tracking and searchability of logs across all
  processes.
- **Scalability and Resilience**: Logs are not lost if a process dies;
  aggregation systems retain history for analysis.
- **Decoupling**: The app is agnostic to log storage, processing, and alerting.

---

## XII. Admin Processes: Run Admin/Management Tasks as One-Off Processes

**Essential Principle**: **Run administrative or maintenance tasks as ephemeral,
one-off processes using the same code and configuration as your app’s main
processes**.

- Typical admin tasks: running database migrations, data imports, scripts, or
  analytics jobs.
- These tasks must be version-controlled, executed in the same environment as
  the app, and isolated from regular app operations.

**Best Practices**:

- Include admin scripts and utilities in the app repo.
- Run these tasks as ad-hoc commands or jobs in the exact same
  release/environment as all other processes.
- Ensure dependency isolation and that side effects are safely handled.

**Benefits**:

- **Change Auditability**: All scripts are subject to the same review and
  versioning as mainline code.
- **Consistency and Reliability**: Reduces risks of “it worked in the admin
  environment but not in prod.”

**Modern Implementation**: In CI/CD, database migrations often run as a distinct
container or job in the pipeline; in PaaS (like Heroku), “one-off” dynos can be
launched to run admin scripts with the current release.

---

## Summary Table: The 12 Factors of the 12-Factor App

| Factor              | Key Practice                                | Benefit/Goal                       |
| ------------------- | ------------------------------------------- | ---------------------------------- |
| Codebase            | Single codebase, many deploys               | Consistency, traceability          |
| Dependencies        | Explicit, isolated dependencies             | Portable, deterministic builds     |
| Config              | Store config in environment                 | Flexibility, security, reusability |
| Backing Services    | Treat as attached resources                 | Swap-ability, resilience           |
| Build, Release, Run | Strict separation of stages                 | Consistency, easier rollback       |
| Processes           | Stateless, share-nothing processes          | Scalability, robustness            |
| Port Binding        | Export via port—self-contained services     | Portability, integration ease      |
| Concurrency         | Scale by process type                       | Scalable, flexible resource usage  |
| Disposability       | Fast startup/shutdown, crash resilience     | Agility, failover                  |
| Dev/prod Parity     | Minimize gaps between all environments      | Predictability, rapid deployment   |
| Logs                | Event streams to stdout/stderr              | Observability, decoupling          |
| Admin Processes     | Run tasks as one-off, first-class processes | Consistency, maintainability       |
