# 🚀 Helm — Complete Source of Truth Notes

> **LevelUp / DevOps Preparation**
>
> A practical, interview-ready reference for Helm 3: definitions, architecture, chart structure, templating, values, releases, upgrades, rollback, dependencies, packaging, CI/CD, troubleshooting, security, commands, and interview questions.

---

## 📚 Table of Contents

1. What is Helm?
2. Why Helm is needed
3. Helm vs Kubernetes
4. Helm architecture
5. Core terminology
6. Chart structure
7. Chart.yaml
8. values.yaml
9. Helm templates
10. The `.` context
11. `$` root context
12. `_helpers.tpl`
13. `define` and `include`
14. Values overriding and precedence
15. Multiple values files
16. `--set` and related options
17. Rendering and validation
18. Helm releases
19. `helm install`
20. `helm upgrade`
21. `helm upgrade --install`
22. Revision history
23. Rollback
24. `--wait`, `--timeout`, `--atomic`
25. `helm uninstall`
26. Dependencies and subcharts
27. Chart.lock
28. Subchart values
29. Helm repositories
30. Packaging and `.tgz`
31. Chart version vs app version
32. CI/CD workflow
33. Production architecture
34. Troubleshooting
35. Common failure scenarios
36. Security considerations
37. Helm/Kubernetes ownership
38. Command reference
39. Interview questions
40. Final mental model

---

# 1. 🧭 What is Helm?

**Helm is a package manager and release-management tool for Kubernetes.**

It packages Kubernetes resources into reusable **charts**, uses configurable values to render templates, and manages deployed instances as **releases**.

### Simple definition

> **Helm = Kubernetes application packaging + templating + release management.**

```text
                 📦 Helm Chart
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     Templates      Values       Metadata
        │             │             │
        └─────────────┼─────────────┘
                      ▼
               Rendered YAML
                      │
                      ▼
              Kubernetes API
                      │
                      ▼
                🟢 Resources
```

---

# 2. 🎯 Why Helm is needed

Without Helm, an application may require:

```text
deployment.yaml
service.yaml
configmap.yaml
secret.yaml
ingress.yaml
hpa.yaml
serviceaccount.yaml
```

DEV, QA and PROD often need different configurations, creating duplication.

Helm separates:

- **Structure** → templates
- **Configuration** → values
- **Metadata/dependencies** → Chart.yaml

```text
                 One Helm Chart
                       │
              ┌────────┴────────┐
              ▼                 ▼
          DEV values        PROD values
              │                 │
              ▼                 ▼
       DEV manifests      PROD manifests
```

### Benefits

| Benefit | Meaning |
|---|---|
| ♻️ Reusability | Same chart can deploy many environments |
| ⚙️ Configuration | Change values without duplicating templates |
| 📦 Packaging | Charts can be packaged and shared |
| 🔄 Release management | Track releases and revisions |
| ↩️ Rollback | Return to an earlier revision |
| 🧩 Dependencies | Use other charts as dependencies |
| 🚀 CI/CD | Easy pipeline integration |
| 🔍 Troubleshooting | Inspect rendering and releases |

---

# 3. ⚖️ Helm vs Kubernetes

Helm is **not a replacement for Kubernetes**.

```text
             👨‍💻 DevOps Engineer
                     │
                     ▼
                   Helm
                     │
          Render / install / upgrade
                     │
                     ▼
             Kubernetes API
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      Deployment   Service    ConfigMap
          │
          ▼
        Pods
```

> **Helm manages chart/release configuration; Kubernetes controllers manage actual workload behavior.**

For example:

```text
Helm
 │
 └── Deployment
       │
       └── Kubernetes Deployment Controller
               │
               └── ReplicaSet
                      │
                      └── Pods
```

---

# 4. 🏗️ Helm Architecture

Modern Helm 3 uses the Helm CLI to communicate with the Kubernetes API.

```text
┌───────────────────────────────┐
│        👨‍💻 DevOps User        │
└───────────────┬───────────────┘
                │
                ▼
        ┌───────────────┐
        │  Helm CLI     │
        │  install      │
        │  upgrade      │
        │  rollback     │
        └───────┬───────┘
                │
       Render / package / manage
                │
                ▼
        ┌───────────────┐
        │ Kubernetes API│
        │    Server     │
        └───────┬───────┘
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
 Deployment   Service   ConfigMap
      │
      ▼
 ReplicaSet
      │
      ▼
    Pods
```

### Helm 2 vs Helm 3

> **Helm 3 does not use the old Tiller server architecture.** Helm 3 uses the user's Kubernetes credentials/RBAC to interact with the API.

---

# 5. 📖 Core Helm Terminology

| Term | Definition |
|---|---|
| **Chart** | Package containing Kubernetes resource definitions/templates |
| **Template** | Kubernetes manifest containing Helm template expressions |
| **Values** | Configuration supplied to templates |
| **Release** | Deployed instance of a chart |
| **Revision** | Version/state of a release |
| **Repository** | Location from which charts can be discovered/downloaded |
| **Subchart** | Dependency chart used by a parent chart |
| **Chart.yaml** | Chart metadata and dependency declaration |
| **values.yaml** | Default chart configuration |
| **_helpers.tpl** | Reusable template helpers |
| **Chart.lock** | Locked dependency state |
| **Manifest** | Rendered Kubernetes YAML |
| **Namespace** | Kubernetes namespace where resources are deployed |

### ⭐ Most important

```text
Chart     = blueprint
Release   = deployed instance
Revision  = release version/state
```

---

# 6. 📁 Helm Chart Structure

Typical chart:

```text
myapp/
│
├── Chart.yaml
├── Chart.lock
├── values.yaml
├── values.schema.json
├── README.md
│
├── charts/
│   └── dependency-chart.tgz
│
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── configmap.yaml
    ├── serviceaccount.yaml
    ├── hpa.yaml
    ├── _helpers.tpl
    └── NOTES.txt
```

### Important files

**`Chart.yaml`** — metadata and dependencies.

**`values.yaml`** — default configuration.

**`templates/`** — Kubernetes templates.

**`_helpers.tpl`** — reusable template logic.

**`charts/`** — packaged dependencies.

**`Chart.lock`** — resolved dependency state.

**`values.schema.json`** — optional values validation schema.

**`NOTES.txt`** — optional post-install/upgrade notes.

---

# 7. 📋 Chart.yaml

Example:

```yaml
apiVersion: v2

name: customer-api

description: Helm chart for Customer API

type: application

version: 1.0.0

appVersion: "2.5.1"
```

### Important fields

| Field | Purpose |
|---|---|
| `apiVersion` | Chart API version |
| `name` | Chart name |
| `description` | Description |
| `type` | `application` or `library` |
| `version` | Chart version |
| `appVersion` | Application version metadata |
| `dependencies` | Dependent charts |

### Application chart

```yaml
type: application
```

Normal deployable chart.

### Library chart

```yaml
type: library
```

Provides reusable template helpers and is not normally installed as a standalone application.

---

# 8. ⚙️ values.yaml

Default configuration:

```yaml
replicaCount: 3

image:
  repository: company/customer-api
  tag: "2.5.1"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8080

resources:
  requests:
    cpu: 250m
    memory: 256Mi

  limits:
    cpu: 500m
    memory: 512Mi
```

Templates access values using:

```gotemplate
.Values
```

Examples:

```gotemplate
.Values.replicaCount
.Values.image.repository
.Values.image.tag
```

Mental model:

```text
values.yaml
     │
     ▼
  .Values
     │
     ▼
templates
     │
     ▼
Kubernetes YAML
```

---

# 9. 🧩 Helm Templates

Helm uses Go template syntax.

Example:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
```

With:

```yaml
replicaCount: 3
```

renders:

```yaml
spec:
  replicas: 3
```

Another:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

renders:

```yaml
image: "company/customer-api:2.5.1"
```

Common constructs:

```gotemplate
{{ .Values.replicaCount }}
```

Print a value.

```gotemplate
{{ if .Values.ingress.enabled }}
...
{{ end }}
```

Conditional rendering.

```gotemplate
{{ range .Values.environments }}
...
{{ end }}
```

Loop through values.

---

# 10. 🔵 The `.` Context

`.` means:

> **The current template context/data object.**

At the root:

```text
.
│
├── Values
├── Release
├── Chart
├── Files
└── ...
```

Therefore:

```gotemplate
.Values
```

means:

> Access `Values` from the current context.

And:

```gotemplate
.Values.replicaCount
```

means:

> Current context → Values → replicaCount.

### `.` can change

Inside:

```gotemplate
{{ range .Values.environments }}
```

`.` becomes the current item.

So if the current item is:

```yaml
name: dev
replicas: 2
```

then:

```gotemplate
.name
.replicas
```

refer to that current object.

### Mental model

```text
. → "Where am I / what data am I currently operating on?"
```

---

# 11. 🔵 `$` Root Context

`$` represents the root context.

Example:

```yaml
environments:
  - name: dev
    replicas: 2
  - name: prod
    replicas: 5
```

Template:

```gotemplate
{{ range .Values.environments }}
name: {{ .name }}
replicas: {{ .replicas }}
{{ end }}
```

Inside `range`:

```text
. → current environment
```

But:

```text
$ → original/root context
```

Therefore:

```gotemplate
{{ range .Values.environments }}
  name: {{ .name }}
  image: {{ $.Values.image.repository }}
{{ end }}
```

### Easy memory

```text
.  → Where am I currently?
$  → Where did I originally start?
```

---

# 12. 🛠️ `_helpers.tpl`

`_helpers.tpl` contains reusable Helm template definitions.

Example:

```text
templates/
├── deployment.yaml
├── service.yaml
└── _helpers.tpl
```

Helper:

```gotemplate
{{- define "myapp.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end }}
```

Use it:

```yaml
metadata:
  name: {{ include "myapp.fullname" . }}
```

Why?

Without helpers, the same naming/label logic can be duplicated in many templates.

```text
             _helpers.tpl
                   │
          "myapp.fullname"
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
 Deployment      Service     ConfigMap
```

---

# 13. `define` and `include`

### Define

```gotemplate
{{- define "myapp.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end }}
```

Creates reusable template logic.

### Include

```gotemplate
{{ include "myapp.fullname" . }}
```

Runs the helper and passes the current context.

```text
include
  │
  ├── "myapp.fullname" → which helper?
  │
  └── . → what context should it receive?
```

---

# 14. 🔀 Values Overriding and Precedence

Default:

```yaml
# values.yaml
replicaCount: 3
```

Override:

```yaml
# values-prod.yaml
replicaCount: 10
```

Command:

```bash
helm install myapp ./myapp -f values-prod.yaml
```

Final:

```text
replicaCount = 10
```

General order:

```text
                HIGHER
                  ▲
                  │
                --set
                  │
             -f file3.yaml
                  │
             -f file2.yaml
                  │
             -f file1.yaml
                  │
             values.yaml
                  │
                  ▼
                LOWER
```

Later overrides earlier.

---

# 15. 📚 Multiple Values Files

```bash
helm install myapp ./myapp   -f values-prod.yaml   -f values-india.yaml
```

Processing:

```text
values.yaml
     ↓
values-prod.yaml
     ↓
values-india.yaml
     ↓
Final values
```

If both override the same key, the later file wins.

---

# 16. 🎛️ `--set` and Related Options

Direct override:

```bash
helm install myapp ./myapp   --set replicaCount=10
```

Nested value:

```bash
helm install myapp ./myapp   --set image.tag=2.5.2
```

CI/CD example:

```bash
helm upgrade --install myapp ./myapp   -f values-prod.yaml   --set image.tag=$IMAGE_TAG
```

### `--set-string`

```bash
helm install myapp ./myapp   --set-string buildNumber=00123
```

Forces string semantics.

### `--set-file`

```bash
helm install myapp ./myapp   --set-file config=./config.txt
```

### `--set-json`

```bash
helm install myapp ./myapp   --set-json 'config={"enabled":true,"port":8080}'
```

---

# 17. 🔍 Rendering and Validation

### Lint

```bash
helm lint ./myapp
```

Checks common chart issues.

### Render

```bash
helm template myapp ./myapp
```

Production:

```bash
helm template myapp ./myapp   -f values-prod.yaml
```

Mental model:

```text
Chart + Values
      │
      ▼
Helm template engine
      │
      ▼
Rendered Kubernetes YAML
```

### Debug

```bash
helm template myapp ./myapp   -f values-prod.yaml   --debug
```

### Dry run

```bash
helm install myapp ./myapp   -f values-prod.yaml   --dry-run   --debug
```

---

# 18. 🟢 Helm Releases

A **release** is a deployed instance of a chart.

```bash
helm install prod-myapp ./myapp
```

Chart:

```text
myapp
```

Release:

```text
prod-myapp
```

One chart can have multiple releases:

```text
                📦 myapp chart
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       dev-myapp   qa-myapp   prod-myapp
```

A release is namespace-scoped.

```bash
helm install myapp ./myapp -n dev
helm install myapp ./myapp -n prod
```

These can coexist.

---

# 19. 📥 `helm install`

```bash
helm install myapp ./myapp
```

Meaning:

> Create a new release called `myapp` from the chart at `./myapp`.

With namespace:

```bash
helm install myapp ./myapp   -n production   --create-namespace
```

Flow:

```text
Chart
  │
  ▼
Values
  │
  ▼
Render templates
  │
  ▼
Kubernetes manifests
  │
  ▼
Kubernetes API
  │
  ▼
Release created
```

---

# 20. 🔄 `helm upgrade`

```bash
helm upgrade myapp ./myapp
```

Updates an existing release.

Example:

```text
Current:
image = 1.0

        │
        ▼
helm upgrade

        │
        ▼

New:
image = 2.0
```

Kubernetes then performs the workload rollout.

---

# 21. ⭐ `helm upgrade --install`

```bash
helm upgrade --install myapp ./myapp
```

Means:

```text
Does release exist?
       │
   ┌───┴───┐
  YES      NO
   │        │
   ▼        ▼
Upgrade   Install
```

Production-style:

```bash
helm upgrade --install customer-api ./customer-api   -n production   --create-namespace   -f values-prod.yaml   --set image.tag=2.5.2
```

This pattern is especially common in CI/CD.

---

# 22. 📜 Helm Revision History

Example:

```text
Revision 1 → app 1.0
Revision 2 → app 1.1
Revision 3 → app 2.0
```

Check:

```bash
helm history myapp
```

Example:

```text
REVISION   STATUS
1          superseded
2          superseded
3          deployed
```

---

# 23. ↩️ Rollback

```bash
helm rollback myapp 2
```

Example:

```text
Revision 1 → 1.0
Revision 2 → 1.1 ✅
Revision 3 → 2.0 ❌
```

Rollback to revision 2:

```text
1 → 2 → 3
        │
        ▼
   rollback to 2
        │
        ▼
1 → 2 → 3 → 4
             ↑
       new current revision
       based on revision 2
```

Rollback does **not** change Git history.

---

# 24. ⏳ `--wait`, `--timeout`, `--atomic`

### `--wait`

```bash
helm upgrade myapp ./myapp --wait
```

Waits for relevant resources to reach expected readiness, subject to timeout.

### `--timeout`

```bash
helm upgrade myapp ./myapp   --wait   --timeout 10m
```

Controls waiting timeout.

### `--atomic`

```bash
helm upgrade myapp ./myapp   --atomic   --wait
```

If the operation fails, Helm automatically rolls back the failed operation.

### Production pattern

```bash
helm upgrade --install myapp ./myapp   -n production   -f values-prod.yaml   --set image.tag=2.5.2   --wait   --atomic   --timeout 10m
```

Flow:

```text
                 Helm deployment
                       │
                       ▼
                    --wait
                       │
              ┌────────┴────────┐
              ▼                 ▼
           Healthy            Failure
              │                 │
              ▼                 ▼
           Success           --atomic
                                │
                                ▼
                             Rollback
```

⚠️ `--atomic` cannot understand business correctness. A running, ready application can still have a logical bug.

---

# 25. 🗑️ `helm uninstall`

```bash
helm uninstall myapp
```

or:

```bash
helm uninstall myapp -n production
```

Conceptually:

```text
Helm Release
     │
     ├── Deployment ──❌
     ├── Service ─────❌
     ├── ConfigMap ───❌
     └── Ingress ─────❌
```

Do not assume uninstall deletes absolutely every related object in every situation.

A resource can request retention:

```yaml
metadata:
  annotations:
    helm.sh/resource-policy: keep
```

### Storage warning ⚠️

For stateful workloads:

```bash
kubectl get pvc -n production
kubectl get pv
```

Understand PVC/PV lifecycle and reclaim policy before destructive operations.

---

# 26. 📦 Dependencies and Subcharts

A chart can depend on other charts.

Example:

```text
customer-api
    │
    ├── Redis
    └── PostgreSQL
```

Declare:

```yaml
dependencies:
  - name: redis
    version: 18.0.0
    repository: "https://example.com/charts"

  - name: postgresql
    version: 16.0.0
    repository: "https://example.com/charts"
```

A dependency can be a subchart or managed as a separate Helm release depending on architecture.

---

# 27. 🔒 Chart.lock

Dependency workflow:

```text
Chart.yaml
    │
    │ dependency requirements
    ▼
helm dependency update
    │
    ▼
Chart.lock
    │
    ▼
Resolved dependency state
```

Commands:

```bash
helm dependency update ./myapp
helm dependency build ./myapp
helm dependency list ./myapp
```

`Chart.lock` helps keep dependency resolution reproducible.

---

# 28. 🎛️ Subchart Values

Parent `values.yaml` can configure a subchart under its name:

```yaml
replicaCount: 3

redis:
  architecture: standalone

  auth:
    enabled: true
```

Conceptually:

```text
Parent values
│
├── replicaCount
│
└── redis
     │
     ├── architecture
     └── auth
```

### Subchart vs separate release

```text
Subchart:
customer-api release
      │
      └── Redis

Separate:
customer-api → release 1
Redis        → release 2
```

For shared infrastructure, separate releases can be preferable.

---

# 29. 🌐 Helm Repositories

A chart repository stores/distributes charts.

```bash
helm repo add myrepo https://example.com/charts
helm repo update
helm search repo myrepo
```

Install:

```bash
helm install myapp myrepo/myapp
```

Distinction:

```text
Repository → location containing charts
Chart      → packaged application
Release    → deployed chart instance
```

---

# 30. 📦 Packaging and `.tgz`

Package a chart:

```bash
helm package ./myapp
```

Output:

```text
myapp-1.0.0.tgz
```

Install directly:

```bash
helm install myapp ./myapp-1.0.0.tgz
```

You do **not** need to unpack it first.

To inspect/unpack:

```bash
tar -xzf myapp-1.0.0.tgz
```

Mental model:

```text
myapp/
   │
   │ helm package
   ▼
myapp-1.0.0.tgz
   │
   ├── share
   ├── store
   └── install directly
```

It is conceptually similar to a packaged Ansible Collection artifact, but Helm has its own chart/repository/release lifecycle.

---

# 31. 🔢 Chart Version vs App Version

Example:

```yaml
version: 1.4.0
appVersion: "2.8.3"
```

### `version`

Helm chart version.

Changes when the chart changes.

### `appVersion`

Application version metadata.

### Important

Changing `appVersion` does not automatically change the image.

The image is normally controlled through values/templates:

```yaml
image:
  repository: company/customer-api
  tag: "2.8.3"
```

---

# 32. 🔄 Helm and CI/CD

Typical workflow:

```text
Developer
    │
    ▼
Git push
    │
    ▼
CI/CD
    │
    ├── Tests
    ├── Docker build
    ├── Push image
    ├── helm lint
    ├── helm template
    │
    ▼
helm upgrade --install
    │
    ▼
Kubernetes/GKE
```

Example:

```bash
helm upgrade --install customer-api ./helm/customer-api   --namespace production   --create-namespace   --values ./helm/customer-api/values-prod.yaml   --set image.tag="${IMAGE_TAG}"   --wait   --atomic   --timeout 10m
```

---

# 33. 🏗️ Production Deployment Architecture

```text
                 👨‍💻 Developer
                      │
                      ▼
                 Git Repository
                      │
                      ▼
                   CI/CD
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
   Unit Tests      Docker Build   Helm Lint
                      │              │
                      ▼              ▼
               Container Registry  Helm Template
                      │              │
                      └──────┬───────┘
                             ▼
                  Helm Upgrade/Install
                             │
                             ▼
                        GKE Cluster
                             │
                 ┌───────────┼───────────┐
                 ▼           ▼           ▼
             Deployment    Service     Ingress
                 │
                 ▼
             ReplicaSet
                 │
                 ▼
                Pods
```

---

# 34. 🔎 Helm Troubleshooting

First distinguish:

```text
Helm/rendering problem
        VS
Kubernetes/runtime problem
```

### Helm side

```bash
helm status myapp -n production
helm history myapp -n production
helm get values myapp -n production --all
helm get manifest myapp -n production
helm lint ./myapp
helm template myapp ./myapp -f values-prod.yaml
```

### Kubernetes side

```bash
kubectl get pods -n production
kubectl describe pod <pod> -n production
kubectl logs <pod> -n production
kubectl logs <pod> -n production --previous
kubectl get deployment -n production
kubectl get svc -n production
kubectl get events -n production --sort-by='.lastTimestamp'
```

---

# 35. 🚨 Common Failure Scenarios

## Template syntax error

Use:

```bash
helm lint ./myapp
helm template myapp ./myapp --debug
```

Possible causes:

```text
Template syntax
Wrong variable
Missing end
Incorrect helper
```

## Wrong value

Check:

```bash
helm get values myapp -n production --all
helm template myapp ./myapp -f values-prod.yaml
```

Possible causes:

```text
Wrong values file
Wrong key
Wrong precedence
Wrong template reference
```

## ImagePullBackOff

```bash
kubectl describe pod <pod> -n production
```

Possible causes:

```text
Wrong image
Wrong tag
Private registry authentication
Registry connectivity
```

## CrashLoopBackOff

```bash
kubectl logs <pod> -n production
kubectl logs <pod> -n production --previous
kubectl describe pod <pod> -n production
```

Possible causes:

```text
Application crash
Bad configuration
Missing environment variable
Database unavailable
Wrong command/arguments
```

## Pending Pod

```bash
kubectl describe pod <pod> -n production
```

Possible causes:

```text
Insufficient CPU/memory
Node selector mismatch
Taints/tolerations
PVC pending
Scheduling constraints
```

## `--wait` timeout

```bash
helm status myapp
kubectl get pods -n production
kubectl describe pod <pod> -n production
kubectl get events -n production --sort-by='.lastTimestamp'
```

Likely flow:

```text
Helm waits
   ↓
Resource never becomes Ready
   ↓
Timeout
```

## Service not working

```bash
kubectl get svc -n production
kubectl describe svc myapp -n production
kubectl get endpoints -n production
kubectl get endpointslices -n production
```

Common cause:

```text
Service selector ≠ Pod labels
```

---

# 36. 🔐 Security Considerations

Helm does not bypass Kubernetes RBAC.

```text
Developer
   │
   ▼
Helm CLI
   │
   ▼
Kubernetes API
   │
   ▼
RBAC authorization
```

If the identity cannot create Deployments in a namespace, Helm cannot magically bypass that.

### Secrets

Avoid casually committing credentials into:

```text
values.yaml
```

Use appropriate secret-management mechanisms such as:

- Kubernetes Secrets with appropriate controls
- External secret managers
- Cloud secret-management integrations
- CI/CD secret stores

Remember:

> Kubernetes Secrets should not automatically be treated as a complete enterprise secret-management solution.

---

# 37. 🔗 Helm and Kubernetes Object Ownership

If Helm wants:

```text
Deployment/myapp
```

but a manually-created object with the same identity already exists, Helm can encounter ownership/conflict errors.

Conceptually:

```text
Helm chart wants:
Deployment/myapp

Cluster already has:
Deployment/myapp
created outside the release

             ↓

       ownership conflict
```

Operational lesson:

> Avoid manually creating resources that a Helm release is expected to own unless you deliberately understand Helm's ownership/adoption behavior.

---

# 38. 📋 Useful Command Reference

```bash
# Create
helm create myapp

# Validate
helm lint ./myapp

# Render
helm template myapp ./myapp

# Dry run
helm install myapp ./myapp --dry-run --debug

# Install
helm install myapp ./myapp

# Namespace
helm install myapp ./myapp -n production --create-namespace

# Upgrade
helm upgrade myapp ./myapp

# Install or upgrade
helm upgrade --install myapp ./myapp

# List
helm list
helm list -A

# Status
helm status myapp

# History
helm history myapp

# Values
helm get values myapp
helm get values myapp --all

# Manifest
helm get manifest myapp

# Rollback
helm rollback myapp 2

# Uninstall
helm uninstall myapp

# Package
helm package ./myapp

# Dependencies
helm dependency update ./myapp
helm dependency build ./myapp
helm dependency list ./myapp

# Repository
helm repo add myrepo https://example.com/charts
helm repo update
helm search repo myrepo
```

---

# 39. 🎯 Interview Questions

### Q1. What is Helm?

> Helm is a Kubernetes package and release-management tool that packages Kubernetes resource templates, supports configurable values, and manages deployed releases and revisions.

### Q2. What is a Helm chart?

> A chart is a package containing Kubernetes resource definitions/templates, metadata, default values, and optionally dependencies.

### Q3. Chart vs release?

> A chart is the blueprint/package. A release is a deployed instance of that chart.

### Q4. What is `values.yaml`?

> It contains default configuration values consumed by Helm templates.

### Q5. What is `.`?

> `.` represents the current template context. At root it gives access to objects such as `.Values`, `.Release`, and `.Chart`; it can change inside `range` and `with`.

### Q6. What is `_helpers.tpl`?

> A file containing reusable Helm template definitions, commonly for naming, labels, selectors, and other repeated logic.

### Q7. Why pass `.` to `include`?

For:

```gotemplate
{{ include "myapp.fullname" . }}
```

> The final `.` passes the current context to the helper so it can access `.Release`, `.Chart`, `.Values`, etc.

### Q8. What is `helm upgrade --install`?

> It upgrades an existing release or installs it when the release does not exist. It is common in CI/CD.

### Q9. What is a revision?

> A version/state of a Helm release across its lifecycle.

### Q10. How do you rollback?

```bash
helm rollback myapp 2
```

### Q11. What does `--atomic` do?

> It automatically rolls back a failed Helm installation or upgrade.

### Q12. Why use `--wait`?

> It makes Helm wait for relevant resources to become ready, subject to timeout.

### Q13. How do you troubleshoot Helm?

```text
helm status
     ↓
helm history
     ↓
helm get values --all
     ↓
helm get manifest
     ↓
helm lint
     ↓
helm template
     ↓
kubectl get
     ↓
kubectl describe
     ↓
kubectl logs
     ↓
kubectl events
```

### Q14. What is a dependency?

> Another Helm chart required by a parent chart.

### Q15. Where are dependencies declared?

```text
Chart.yaml
```

### Q16. What is Chart.lock?

> A lock file recording resolved dependency state for more reproducible dependency builds.

### Q17. Can a chart be packaged?

Yes:

```bash
helm package ./myapp
```

### Q18. Can you install a `.tgz` directly?

Yes:

```bash
helm install myapp ./myapp-1.0.0.tgz
```

### Q19. `version` vs `appVersion`?

> `version` is the Helm chart version. `appVersion` is application version metadata.

### Q20. Is Helm idempotent?

A nuanced answer:

> Helm supports declarative desired-state-style deployment and `helm upgrade --install` is commonly used repeatedly in automation, but Helm should not be described as an absolute idempotency guarantee for every chart or template side effect. The Kubernetes resources themselves are reconciled by Kubernetes controllers.

---

# 40. 🧠 Final Mental Model

If you remember one diagram, remember this:

```text
                         📦 HELM CHART
                              │
              ┌───────────────┼────────────────┐
              │               │                │
              ▼               ▼                ▼
         Chart.yaml       values.yaml      templates/
         metadata         configuration     Kubernetes
              │               │              templates
              │               │                │
              └───────────────┼────────────────┘
                              ▼
                      Helm Template Engine
                              │
                              ▼
                     Rendered Kubernetes YAML
                              │
                              ▼
                     Kubernetes API Server
                              │
                              ▼
                       🟢 HELM RELEASE
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
           Deployment       Service       ConfigMap
                │
                ▼
             ReplicaSet
                │
                ▼
               Pods
```

### Release lifecycle

```text
                    📦 CHART
                       │
                       ▼
                  helm install
                       │
                       ▼
                 🟢 Revision 1
                       │
                  helm upgrade
                       │
                       ▼
                 🟢 Revision 2
                       │
                  helm upgrade
                       │
                       ▼
                 🟢 Revision 3
                       │
                    ❌ Problem
                       │
                       ▼
                helm rollback 2
                       │
                       ▼
                 🟢 New revision
                  based on Rev 2
                       │
                       ▼
                 helm uninstall
                       │
                       ▼
                 🔴 Release removed
```

---

# 🏆 Helm LevelUp Checklist

## Fundamentals
- [x] What Helm is
- [x] Why Helm is used
- [x] Helm vs Kubernetes
- [x] Helm 3 architecture
- [x] Chart vs release vs revision
- [x] Chart structure

## Chart
- [x] `Chart.yaml`
- [x] `values.yaml`
- [x] `templates/`
- [x] `_helpers.tpl`
- [x] `charts/`
- [x] `Chart.lock`
- [x] `NOTES.txt`
- [x] `values.schema.json`

## Templating
- [x] Go templates
- [x] `.Values`
- [x] `.Release`
- [x] `.Chart`
- [x] `.` current context
- [x] `$` root context
- [x] `range`
- [x] `if`
- [x] `define`
- [x] `include`

## Configuration
- [x] Default values
- [x] `-f`
- [x] Multiple values files
- [x] `--set`
- [x] `--set-string`
- [x] `--set-file`
- [x] `--set-json`
- [x] Precedence

## Lifecycle
- [x] `helm install`
- [x] `helm upgrade`
- [x] `helm upgrade --install`
- [x] `helm list`
- [x] `helm status`
- [x] `helm history`
- [x] `helm rollback`
- [x] `helm uninstall`

## Reliability
- [x] `--wait`
- [x] `--timeout`
- [x] `--atomic`
- [x] Revision history
- [x] Rollback behavior

## Dependencies
- [x] Chart dependencies
- [x] Subcharts
- [x] `helm dependency update`
- [x] `helm dependency build`
- [x] `helm dependency list`
- [x] Parent/subchart values
- [x] Chart.lock

## Packaging
- [x] Helm repositories
- [x] `helm package`
- [x] `.tgz`
- [x] Installing packaged charts

## Troubleshooting
- [x] `helm lint`
- [x] `helm template`
- [x] `--dry-run`
- [x] `--debug`
- [x] `helm get values`
- [x] `helm get manifest`
- [x] `kubectl get`
- [x] `kubectl describe`
- [x] `kubectl logs`
- [x] `kubectl logs --previous`
- [x] Kubernetes events
- [x] Helm vs Kubernetes troubleshooting

---

# 🎓 Final Interview Summary

If asked **"Explain Helm"**:

> **Helm is a Kubernetes package and release-management tool. A Helm chart packages Kubernetes resource templates, metadata, default values, and optionally dependencies. Environment-specific configuration can be supplied through values files or command-line overrides. Helm renders those templates into Kubernetes manifests and deploys them through the Kubernetes API. Each deployment is tracked as a release with revisions, allowing us to inspect history, upgrade releases, and roll back to previous revisions. In CI/CD, `helm upgrade --install` is commonly combined with `--wait` and `--atomic` for reliable deployments. Helm also provides dependency management, chart packaging, and troubleshooting commands.**

## 🔥 One-line mental model

> **Chart = blueprint → Values = configuration → Templates = rendering logic → Release = deployed instance → Revision = release history → Kubernetes = actual workload orchestration.**

---

# 📌 Quick Cheat Sheet

```bash
helm create myapp
helm lint ./myapp
helm template myapp ./myapp
helm install myapp ./myapp --dry-run --debug

helm install myapp ./myapp
helm upgrade myapp ./myapp
helm upgrade --install myapp ./myapp

helm list -A
helm status myapp
helm history myapp
helm get values myapp --all
helm get manifest myapp

helm rollback myapp 2
helm uninstall myapp

helm package ./myapp

helm dependency update ./myapp
helm dependency build ./myapp
helm dependency list ./myapp

helm repo add myrepo https://example.com/charts
helm repo update
helm search repo myrepo
```

---

> 🧠 **LevelUp focus:** You do not need to memorize every Helm templating function. Prioritize architecture, chart structure, values, template context, release lifecycle, upgrade/rollback, dependencies, CI/CD integration, and troubleshooting. These are the concepts you should be able to explain and apply as a Senior System Engineer.
