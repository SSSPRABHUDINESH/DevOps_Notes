# ☁️ GKE — LevelUp Source of Truth

> 🎯 **Complete GKE reference for LevelUp preparation**
>
> 📚 Concepts • 🏗️ Architectures • 💻 Commands • 📝 YAML • 🔐 IAM • 📈
> Scaling • 🛡️ Security

------------------------------------------------------------------------

# GKE — Complete LevelUp Source of Truth

------------------------------------------------------------------------

> **Scope:** This document follows the exact GKE roadmap from our study
> plan. It is deliberately organized around the topics in the roadmap
> image, not around a different Kubernetes syllabus.
>
> **GKE topics covered:** 1. What GKE is 2. GKE Standard vs Autopilot 3.
> Cluster creation 4. Node pools 5. Workloads 6. Services 7. Load
> balancing 8. GKE networking 9. GKE IAM integration 10. Workload
> Identity 11. GKE logging/monitoring 12. GKE autoscaling 13. Cluster
> upgrades 14. GKE security basics
>
> Kubernetes fundamentals are assumed because they were covered before
> this GKE level. The purpose here is to map those fundamentals to GKE
> and make the document useful for LevelUp interviews and real DevOps
> work.

------------------------------------------------------------------------

# Table of Contents

------------------------------------------------------------------------

1.  [What GKE Is](#1-what-gke-is)
2.  [GKE Standard vs Autopilot](#2-gke-standard-vs-autopilot)
3.  [Cluster Creation](#3-cluster-creation)
4.  [Node Pools](#4-node-pools)
5.  [Workloads](#5-workloads)
6.  [Services](#6-services)
7.  [Load Balancing](#7-load-balancing)
8.  [GKE Networking](#8-gke-networking)
9.  [GKE IAM Integration](#9-gke-iam-integration)
10. [Workload Identity](#10-workload-identity)
11. [GKE Logging and Monitoring](#11-gke-logging-and-monitoring)
12. [GKE Autoscaling](#12-gke-autoscaling)
13. [Cluster Upgrades](#13-cluster-upgrades)
14. [GKE Security Basics](#14-gke-security-basics)
15. [End-to-End Architecture](#15-end-to-end-architecture)
16. [Troubleshooting Playbook](#16-troubleshooting-playbook)
17. [LevelUp Interview Questions](#17-levelup-interview-questions)
18. [Final Revision Sheet](#18-final-revision-sheet)

------------------------------------------------------------------------

# ☁️ 1. What GKE Is

------------------------------------------------------------------------

## 1.1 Definition

------------------------------------------------------------------------

**Google Kubernetes Engine (GKE)** is Google Cloud’s managed Kubernetes
service.

Kubernetes provides the orchestration model. GKE provides the Google
Cloud-managed platform around that Kubernetes cluster, including
integration with Google Cloud networking, IAM, load balancing,
monitoring, logging, upgrades and other services.

### Simple mental model

``` text
                         Google Cloud
                              |
                              v
                    +-------------------+
                    |       GKE         |
                    |      Cluster      |
                    +-------------------+
                       /             \
                      /               \
                     v                 v
          +----------------+    +------------------+
          | Control Plane  |    |   Node Pools     |
          |                |    |                  |
          | API Server     |    | Node             |
          | Scheduler      |    | Node             |
          | Controllers    |    | Node             |
          | etcd           |    |                  |
          +----------------+    +--------+---------+
                                         |
                                         v
                                       Pods
                                         |
                                         v
                                    Containers
```

The key hierarchy is:

``` text
GKE Cluster
   |
   +-- Control Plane
   |
   +-- Node Pools
         |
         +-- Nodes
               |
               +-- Pods
                     |
                     +-- Containers
```

## 1.2 What problem GKE solves

------------------------------------------------------------------------

Without a managed Kubernetes service, an organization has to operate
many parts of the Kubernetes platform itself:

- Control-plane components
- Kubernetes upgrades
- Node infrastructure
- Networking
- Cluster availability
- Integration with cloud services
- Monitoring and logging
- Security configuration

GKE reduces this operational burden while retaining Kubernetes as the
workload orchestration layer.

## 1.3 GKE control plane

------------------------------------------------------------------------

The Kubernetes control plane is responsible for managing the cluster
state.

Important concepts:

| Component   | High-level responsibility                       |
|-------------|-------------------------------------------------|
| API server  | Entry point for Kubernetes API requests         |
| Scheduler   | Decides where Pods should run                   |
| Controllers | Continuously reconcile desired and actual state |
| etcd        | Stores Kubernetes cluster state                 |

In GKE, the control plane is managed by Google. You interact with it
mainly through:

``` text
kubectl
      |
      v
Kubernetes API
      |
      v
GKE control plane
```

## 1.4 GKE cluster locations

------------------------------------------------------------------------

### Zonal cluster

A zonal cluster is associated primarily with one zone.

``` text
Region
 |
 +-- Zone A
      |
      +-- GKE cluster
           +-- Nodes
```

### Regional cluster

A regional cluster provides control-plane availability across multiple
zones in a region and can distribute worker capacity across zones.

``` text
Region
 |
 +-- Zone A ---> Nodes
 |
 +-- Zone B ---> Nodes
 |
 +-- Zone C ---> Nodes
```

For production systems requiring high availability, regional
architecture is commonly preferred.

## 1.5 Important point

------------------------------------------------------------------------

Do not say:

> “GKE means Google manages everything.”

That is too broad.

A better answer is:

> “GKE is a managed Kubernetes service. Google manages the Kubernetes
> control plane and provides managed integrations and capabilities,
> while the amount of worker infrastructure I manage depends on whether
> I use Standard or Autopilot.”

------------------------------------------------------------------------

# ⚖️ 2. GKE Standard vs Autopilot

------------------------------------------------------------------------

This is a high-value interview topic.

## 2.1 GKE Standard

------------------------------------------------------------------------

Standard gives you more control over the worker infrastructure.

Typical areas you can manage include:

- Node pools
- Machine types
- Node labels
- Taints and tolerations
- Node-level configuration
- Specialized compute
- Scaling configuration

Mental model:

``` text
You
 |
 +-- Workloads
 +-- Node pools
 +-- Node configuration
 +-- Scaling choices
 |
Google
 |
 +-- Managed control plane
```

## 2.2 GKE Autopilot

------------------------------------------------------------------------

Autopilot is a more managed operating mode. You focus primarily on
Kubernetes workloads and resource requirements while GKE handles more of
the underlying worker infrastructure.

Mental model:

``` text
You
 |
 +-- Kubernetes manifests
 +-- CPU / memory requirements
 +-- Workload behavior
 |
GKE
 |
 +-- Worker infrastructure management
 +-- Node provisioning/management
 +-- Operational optimization
```

## 2.3 Comparison

------------------------------------------------------------------------

| Area                          | Standard               | Autopilot                |
|-------------------------------|------------------------|--------------------------|
| Control plane                 | Managed                | Managed                  |
| Worker infrastructure control | More                   | Less                     |
| Node pool management          | Explicit               | More abstracted          |
| Node customization            | More flexibility       | More restrictions        |
| Operational overhead          | Higher                 | Lower                    |
| Specialized infrastructure    | More control           | Policy/feature dependent |
| Best fit                      | Infrastructure control | Reduced operations       |

## 2.4 When would you choose Standard?

------------------------------------------------------------------------

Use Standard when you need deeper control, for example:

- Specialized node types
- Specific node-pool architecture
- Custom scheduling requirements
- Infrastructure-level tuning
- Workloads that do not fit Autopilot constraints

## 2.5 When would you choose Autopilot?

------------------------------------------------------------------------

Use Autopilot when:

- You want less node-management work.
- Workloads fit Autopilot’s supported model.
- You want to focus more on applications than worker infrastructure.

## 2.6 LevelUp answer

------------------------------------------------------------------------

> “Standard gives me more control over nodes and node pools, while
> Autopilot is more managed and lets me focus on workloads. I would
> select Standard when I have infrastructure-level requirements and
> Autopilot when reducing operational overhead is more important and the
> workload fits its constraints.”

------------------------------------------------------------------------

# 🏗️ 3. Cluster Creation

------------------------------------------------------------------------

A GKE cluster can be created through:

1.  Google Cloud Console
2.  `gcloud`
3.  Terraform

## 3.1 Before creating a production cluster

------------------------------------------------------------------------

Decide:

``` text
Cluster design
 |
 +-- Region / zone
 +-- Standard / Autopilot
 +-- VPC / subnet
 +-- Pod and Service networking
 +-- Private/public access requirements
 +-- Node pools
 +-- Autoscaling
 +-- IAM / Workload Identity
 +-- Logging / monitoring
 +-- Upgrade strategy
```

## 3.2 Console flow

------------------------------------------------------------------------

The exact UI wording can change, but the conceptual flow is:

``` text
Google Cloud Console
        |
        v
Kubernetes Engine
        |
        v
Clusters
        |
        v
Create
        |
        +-- Standard / Autopilot
        +-- Region / Zone
        +-- Networking
        +-- Security
        +-- Node configuration (Standard)
        +-- Observability
        |
        v
Create cluster
```

## 3.3 Standard cluster using gcloud

------------------------------------------------------------------------

Example regional cluster:

``` bash
gcloud container clusters create my-gke-cluster \\
  --region=asia-south1 \\
  --machine-type=e2-standard-4 \\
  --num-nodes=3
```

Check clusters:

``` bash
gcloud container clusters list
```

Get Kubernetes credentials:

``` bash
gcloud container clusters get-credentials my-gke-cluster \\
  --region=asia-south1
```

Verify:

``` bash
kubectl get nodes
```

## 🧪 3.4 Autopilot example

------------------------------------------------------------------------

``` bash
gcloud container clusters create-auto my-autopilot-cluster \\
  --region=asia-south1
```

Then:

``` bash
gcloud container clusters get-credentials my-autopilot-cluster \\
  --region=asia-south1
```

## 🧪 3.5 Terraform example

------------------------------------------------------------------------

A simplified example:

``` hcl
resource "google_container_cluster" "primary" {
  name     = "levelup-gke"
  location = "asia-south1"

  deletion_protection = false

  initial_node_count = 1

  node_config {
    machine_type = "e2-standard-4"
  }
}
```

In production, use a deliberately designed network, node-pool, security
and upgrade configuration rather than relying on a minimal example.

## 3.6 Important cluster-creation concepts

------------------------------------------------------------------------

### Region

Determines the geographic location and latency/failure domain.

### VPC

Provides Google Cloud networking.

### Subnet

Provides node/network placement and IP allocation.

### Pod range

In VPC-native GKE, Pods can use secondary IP ranges.

### Service range

Services can use a secondary range for stable virtual IPs.

### 🖥️ Node pools

Define worker capacity and configuration in Standard mode.

------------------------------------------------------------------------

# 🖥️ 4. Node Pools

------------------------------------------------------------------------

## 4.1 Definition

------------------------------------------------------------------------

A **node pool** is a group of nodes in a GKE cluster that have a common
configuration.

``` text
GKE Cluster
 |
 +-- general-pool
 |     +-- node-1
 |     +-- node-2
 |     +-- node-3
 |
 +-- memory-pool
       +-- node-4
       +-- node-5
```

## 🖥️ 4.2 Why use multiple node pools?

------------------------------------------------------------------------

Different workloads have different requirements.

Example:

``` text
                  GKE Cluster
                       |
            +----------+----------+
            |                     |
            v                     v
      General Pool          Memory Pool
            |                     |
       web / API             analytics
```

Another common pattern:

``` text
general-pool       -> normal applications
gpu-pool           -> ML/GPU workloads
memory-pool        -> memory-intensive workloads
system-pool        -> selected platform/system workloads
```

## 4.3 Labels

------------------------------------------------------------------------

Labels help target workloads to suitable nodes.

Example node label:

``` text
workload=memory-intensive
```

Pod:

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-app
spec:
  nodeSelector:
    workload: memory-intensive
  containers:
    - name: app
      image: nginx:1.27
```

## 4.4 Taints and tolerations

------------------------------------------------------------------------

A **taint** says:

> Do not schedule normal Pods here unless they explicitly tolerate this
> condition.

Example:

``` text
Node
 |
 +-- taint: workload=gpu:NoSchedule
```

Pod:

``` yaml
spec:
  tolerations:
    - key: workload
      operator: Equal
      value: gpu
      effect: NoSchedule
```

Mental model:

``` text
Node taint
   |
   | blocks normal scheduling
   v
Special node
   ^
   |
Pod toleration
   |
   +-- permits scheduling
```

## 📈 4.5 Node-pool autoscaling

------------------------------------------------------------------------

Node-pool autoscaling changes infrastructure capacity.

``` text
2 nodes
   |
   | more unscheduled Pods
   v
4 nodes
   |
   | more workload
   v
8 nodes
```

This is different from HPA.

``` text
HPA                    Node-pool autoscaling
 |                              |
 v                              v
Pods                            Nodes
```

------------------------------------------------------------------------

# 📦 5. Workloads

------------------------------------------------------------------------

The workload controller determines how Pods are created and maintained.

The main workload types relevant here are:

- Deployment
- StatefulSet
- DaemonSet
- Job
- CronJob

## 5.1 Deployment

------------------------------------------------------------------------

Use Deployment primarily for stateless applications.

Architecture:

``` text
Deployment
    |
    v
ReplicaSet
    |
    +---- Pod
    +---- Pod
    +---- Pod
```

Example:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
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
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Commands:

``` bash
kubectl apply -f deployment.yaml
kubectl get deployment
kubectl get rs
kubectl get pods
```

## 5.2 StatefulSet

------------------------------------------------------------------------

StatefulSet is intended for workloads needing stable identity and/or
persistent storage.

Example identity:

``` text
postgres-0
postgres-1
postgres-2
```

Unlike ordinary Deployment replicas, the Pods have stable ordinal
identities.

Example:

``` yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: database
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
        - name: database
          image: postgres:16
          env:
            - name: POSTGRES_PASSWORD
              value: example
```

## 5.3 DaemonSet

------------------------------------------------------------------------

DaemonSet is useful when a Pod should run on nodes according to the
DaemonSet’s scheduling rules.

Common use cases:

- Node-level monitoring agent
- Logging agent
- Security agent

Architecture:

``` text
Node 1 ---> Agent Pod
Node 2 ---> Agent Pod
Node 3 ---> Agent Pod
```

Example:

``` yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-agent
spec:
  selector:
    matchLabels:
      app: node-agent
  template:
    metadata:
      labels:
        app: node-agent
    spec:
      containers:
        - name: agent
          image: nginx:1.27
```

## 5.4 Job

------------------------------------------------------------------------

A Job runs a finite task until successful completion.

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: migration
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migration
          image: my-migration-image:1.0
          command: ["/bin/sh", "-c", "./migrate.sh"]
```

## 5.5 CronJob

------------------------------------------------------------------------

CronJob creates Jobs according to a schedule.

``` yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-task
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: task
              image: busybox:1.36
              command: ["/bin/sh", "-c", "echo daily task"]
```

## 5.6 Deployment vs StatefulSet

------------------------------------------------------------------------

| Deployment                       | StatefulSet                                     |
|----------------------------------|-------------------------------------------------|
| Usually stateless                | Usually stateful                                |
| Replica identity is not stable   | Stable ordinal identity                         |
| Common for web/API               | Common for databases/distributed state          |
| Usually interchangeable replicas | Replicas can have identity/storage requirements |

------------------------------------------------------------------------

# 🔌 6. Services

------------------------------------------------------------------------

## 6.1 Why a Service is required

------------------------------------------------------------------------

Pods are ephemeral. Their IP addresses can change.

A Service provides a stable access point and selects Pods using labels.

``` text
                 Service
                    |
          +---------+---------+
          |         |         |
         Pod       Pod       Pod
```

Example:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

The important relationship is:

``` text
Service selector
       |
       v
Pod labels
```

If the labels do not match, the Service can have no usable endpoints.

## 6.2 ClusterIP

------------------------------------------------------------------------

Default Service type.

Used primarily for internal cluster communication.

``` text
Frontend Pod
     |
     v
ClusterIP Service
     |
  +--+--+
  |  |  |
 Pod Pod Pod
```

## 6.3 NodePort

------------------------------------------------------------------------

Exposes a Service through a port on nodes.

``` text
External client
      |
      v
Node IP : NodePort
      |
      v
Service
      |
      v
Pods
```

Example:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

## 6.4 LoadBalancer

------------------------------------------------------------------------

A LoadBalancer Service requests external load-balancing integration from
the cloud provider.

In GKE, this integrates with Google Cloud load-balancing infrastructure.

``` text
Internet
   |
   v
Google Cloud Load Balancer
   |
   v
GKE Service
   |
   +---- Pod
   +---- Pod
   +---- Pod
```

Example:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

## 6.5 Important correction: LoadBalancer, NodePort and ClusterIP

------------------------------------------------------------------------

Do not memorize:

> “LoadBalancer always creates ClusterIP and NodePort, therefore all
> three are always separate objects.”

The correct mental model is that **ClusterIP, NodePort and LoadBalancer
are Service types/behaviors**, not three independent Service objects
that you must manually create.

Historically, a LoadBalancer Service often relied on NodePort for
traffic delivery. Modern Kubernetes/GKE configurations can optimize or
alter this behavior, so the implementation should not be reduced to a
universal rule.

What you should confidently say in an interview:

> “A LoadBalancer Service exposes the application externally through
> cloud load-balancing integration. I should not assume the exact
> underlying packet path without considering the GKE networking mode and
> Service configuration.”

------------------------------------------------------------------------

# 🌐 7. Load Balancing

------------------------------------------------------------------------

## 🌐 7.1 Why load balancing?

------------------------------------------------------------------------

If an application has multiple Pods:

``` text
            +-- Pod 1
            |
Client ---> Service / LB
            |
            +-- Pod 2
            |
            +-- Pod 3
```

Traffic can be distributed among healthy backends according to the
relevant load-balancing mechanism.

## 7.2 L4 vs L7

------------------------------------------------------------------------

### L4

Works at transport/network connection level, such as TCP/UDP.

``` text
Client
  |
  v
L4 Load Balancer
  |
  v
Backend
```

### L7

Understands application-layer protocols such as HTTP/HTTPS and can make
decisions using host/path information.

``` text
Client
  |
  v
HTTP(S) Load Balancer
  |
  +-- /api ---> API
  |
  +-- /web ---> Frontend
```

## 7.3 Ingress

------------------------------------------------------------------------

Ingress is a Kubernetes API object for HTTP/HTTPS routing.

Example:

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 80
```

Architecture:

``` text
Internet
   |
   v
Google Cloud HTTP(S) Load Balancer
   |
   v
Ingress
   |
   +------ /api ------> backend Service ---> Pods
   |
   +------ /web ------> frontend Service --> Pods
```

## 7.4 Service vs Ingress

------------------------------------------------------------------------

``` text
Service
 |
 +-- stable access to a set of Pods

Ingress
 |
 +-- HTTP/HTTPS routing to Services
```

A useful interview statement:

> “A Service provides stable networking to Pods. Ingress provides
> application-layer routing, typically HTTP/HTTPS, to one or more
> Services.”

------------------------------------------------------------------------

# 🌐 8. GKE Networking

------------------------------------------------------------------------

This is one of the most important GKE-specific areas.

## 8.1 VPC-native networking

------------------------------------------------------------------------

GKE can integrate Pod and Service networking directly with the Google
Cloud VPC using secondary IP ranges.

Mental model:

``` text
                     Google Cloud VPC
                            |
                    +-------+-------+
                    |               |
                  Subnet       Secondary ranges
                    |               |
                Node IPs       +----+----+
                                |         |
                              Pod IPs   Service IPs
```

## 8.2 Important IP concepts

------------------------------------------------------------------------

### Node IP

IP assigned to the node/network interface.

### Pod IP

IP used by a Pod.

### Service IP

Virtual/stable IP used to reach a Kubernetes Service.

Keep this mental model:

``` text
Node -> compute/network identity
Pod  -> workload network identity
Service -> stable virtual access point
```

## 8.3 Pod-to-Pod

------------------------------------------------------------------------

``` text
Pod A
  |
  | cluster network
  v
Pod B
```

NetworkPolicy can restrict whether the traffic is allowed.

## 8.4 Pod-to-Service

------------------------------------------------------------------------

``` text
Pod A
  |
  v
Service
  |
  +----> Pod B
  +----> Pod C
```

The Service selects Pods based on labels.

## 8.5 NetworkPolicy

------------------------------------------------------------------------

NetworkPolicy controls allowed Pod network traffic.

Example requirement:

> Only frontend Pods may initiate ingress traffic to backend Pods.

Policy:

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-ingress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

Architecture:

``` text
frontend Pod
    |
    | ALLOWED
    v
backend Pod

random Pod
    |
    | DENIED by policy
    X
backend Pod
```

Important:

> A Service selector does not by itself create a security boundary that
> says only frontend Pods can connect. A NetworkPolicy is used when
> explicit Pod-to-Pod restrictions are required.

## 8.6 VPC firewall vs NetworkPolicy

------------------------------------------------------------------------

| Control       | Scope                               |
|---------------|-------------------------------------|
| VPC firewall  | Google Cloud VPC/network traffic    |
| NetworkPolicy | Kubernetes Pod traffic              |
| IAM           | Google Cloud resource authorization |
| RBAC          | Kubernetes API authorization        |

These solve different problems.

## 8.7 Private cluster mental model

------------------------------------------------------------------------

A private GKE design can keep worker nodes from having public IP
addresses.

Conceptually:

``` text
Internet
   X
   |
   | no direct public node access
   |
Private GKE nodes
   |
   +-- Pods
```

Application exposure can still happen through an appropriate Google
Cloud load balancer.

------------------------------------------------------------------------

# 🔐 9. GKE IAM Integration

------------------------------------------------------------------------

GKE uses both Google Cloud IAM and Kubernetes authorization mechanisms.

The easiest mental model is:

``` text
Google Cloud resource access
        |
        v
       IAM

Kubernetes API access
        |
        v
       RBAC
```

## 9.1 IAM

------------------------------------------------------------------------

IAM answers:

> Who can perform what action on which Google Cloud resource?

Examples:

- Cloud Storage
- Cloud KMS
- BigQuery
- Secret Manager
- Pub/Sub
- Compute Engine

Example:

``` text
GSA
 |
 +-- roles/storage.objectViewer
 |
 v
Cloud Storage bucket
```

## 9.2 Kubernetes RBAC

------------------------------------------------------------------------

RBAC answers:

> What can this identity do through the Kubernetes API?

Examples:

- List Pods
- Create Deployments
- Read Services
- Update ConfigMaps

## 9.3 KSA vs GSA

------------------------------------------------------------------------

### Kubernetes Service Account (KSA)

A Kubernetes identity used by Pods.

``` yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
```

### Google Cloud Service Account (GSA)

A Google Cloud identity.

``` text
app-sa@PROJECT_ID.iam.gserviceaccount.com
```

Do not confuse them.

## 9.4 User accessing GKE

------------------------------------------------------------------------

A human user may use Google Cloud identity to authenticate and then
Kubernetes authorization controls what that user can do through the
cluster API.

Conceptually:

``` text
User
 |
 v
Google identity / IAM authentication
 |
 v
GKE / Kubernetes API
 |
 v
Kubernetes RBAC
 |
 v
Pod / Deployment / Service etc.
```

------------------------------------------------------------------------

# 🪪 10. Workload Identity

------------------------------------------------------------------------

## 10.1 What problem does it solve?

------------------------------------------------------------------------

A Pod may need to access Google Cloud APIs.

Bad pattern:

``` text
Pod
 |
 +-- service-account.json
 |
 v
Google API
```

This introduces a long-lived credential that can be copied or leaked.

Better:

``` text
Pod
 |
 v
Kubernetes Service Account
 |
 v
Workload Identity
 |
 v
Google Cloud Service Account
 |
 v
Google Cloud API
```

## 🏗️ 10.2 Complete architecture

------------------------------------------------------------------------

``` text
+---------------------------+
| Kubernetes Pod            |
|                           |
| serviceAccountName:       |
|   gcs-reader              |
+-------------+-------------+
              |
              v
+---------------------------+
| Kubernetes ServiceAccount |
| gcs-reader                |
+-------------+-------------+
              |
              | Workload Identity
              v
+---------------------------+
| Google Service Account    |
| gcs-reader@project...     |
+-------------+-------------+
              |
              | IAM role
              v
+---------------------------+
| Cloud Storage             |
+---------------------------+
```

## 🧪 10.3 Example: Cloud Storage reader

------------------------------------------------------------------------

### Step 1 — Create GSA

``` bash
gcloud iam service-accounts create gcs-reader \\
  --project=PROJECT_ID
```

### Step 2 — Grant required Google Cloud permission

``` bash
gcloud projects add-iam-policy-binding PROJECT_ID \\
  --member="serviceAccount:gcs-reader@PROJECT_ID.iam.gserviceaccount.com" \\
  --role="roles/storage.objectViewer"
```

The exact scope can be project, bucket or another supported resource
scope depending on the access requirement. Prefer the narrowest
practical scope.

### Step 3 — Create KSA

``` bash
kubectl create serviceaccount gcs-reader
```

### Step 4 — Allow KSA to use the GSA

A common binding is:

``` bash
gcloud iam service-accounts add-iam-policy-binding \\
  gcs-reader@PROJECT_ID.iam.gserviceaccount.com \\
  --role="roles/iam.workloadIdentityUser" \\
  --member="serviceAccount:PROJECT_ID.svc.id.goog[default/gcs-reader]"
```

This is an important distinction:

``` text
roles/iam.workloadIdentityUser
        |
        +-- allows KSA identity to use/impersonate the GSA

roles/storage.objectViewer
        |
        +-- allows the GSA to read Cloud Storage objects
```

They solve different problems.

### Step 5 — Associate KSA with GSA

``` bash
kubectl annotate serviceaccount gcs-reader \\
  --namespace default \\
  iam.gke.io/gcp-service-account=gcs-reader@PROJECT_ID.iam.gserviceaccount.com
```

### Step 6 — Use KSA in the Pod

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: storage-reader
spec:
  replicas: 1
  selector:
    matchLabels:
      app: storage-reader
  template:
    metadata:
      labels:
        app: storage-reader
    spec:
      serviceAccountName: gcs-reader
      containers:
        - name: app
          image: google/cloud-sdk:slim
          command:
            - /bin/sh
            - -c
            - |
              gcloud storage ls gs://MY_BUCKET
              sleep 3600
```

## 10.4 Why the KSA annotation matters

------------------------------------------------------------------------

The Pod uses:

``` yaml
serviceAccountName: gcs-reader
```

That tells Kubernetes which KSA the workload runs as.

The annotation identifies the Google service account associated with
that KSA.

The IAM `workloadIdentityUser` relationship allows the KSA identity to
use the GSA.

The GSA’s own IAM roles determine what Google Cloud resources it can
access.

Think of it as three separate questions:

``` text
1. Which Kubernetes identity is the Pod using?
        |
        v
      KSA

2. Which Google identity does that KSA map to/use?
        |
        v
      GSA

3. What can that Google identity access?
        |
        v
   GSA IAM roles
```

## 10.5 Do I need another user service account?

------------------------------------------------------------------------

No. A workload does not require a `user@gmail.com` identity just because
it uses Workload Identity.

For example:

``` text
Terraform / CI identity
        |
        v
Can impersonate GSA if configured

GKE Pod
        |
        v
KSA -> GSA through Workload Identity
```

Human user identities and workload identities are separate concepts.

## 10.6 Service account impersonation

------------------------------------------------------------------------

Service account impersonation is useful when one identity needs to act
as a service account.

Example:

``` text
User / CI identity
       |
       | Service Account Token Creator
       v
Target GSA
       |
       v
Google Cloud resources
```

A common permission is:

``` text
roles/iam.serviceAccountTokenCreator
```

Example:

``` bash
gcloud iam service-accounts add-iam-policy-binding \\
  terraform-sa@PROJECT_ID.iam.gserviceaccount.com \\
  --member="user:user@example.com" \\
  --role="roles/iam.serviceAccountTokenCreator"
```

Then authenticate with impersonation:

``` bash
gcloud auth application-default login \\
  --impersonate-service-account=terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

Or for many `gcloud` commands:

``` bash
gcloud config set auth/impersonate_service_account \\
  terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

Terraform can also be configured to impersonate a service account, for
example:

``` hcl
provider "google" {
  project                     = var.project_id
  region                      = var.region
  impersonate_service_account = "terraform-sa@PROJECT_ID.iam.gserviceaccount.com"
}
```

Important distinction:

``` text
Workload Identity
   = Pod/KSA -> Google Cloud identity

Service account impersonation
   = one identity -> acts as another service account
```

------------------------------------------------------------------------

# 📋 11. GKE Logging and Monitoring

------------------------------------------------------------------------

Observability answers three broad questions:

``` text
Logs    -> What happened?
Metrics -> How is the system behaving?
Traces  -> Where did a request spend time?
```

## 📋 11.1 Logging

------------------------------------------------------------------------

Typical GKE logs include:

- Container stdout/stderr
- Application logs
- Kubernetes/system logs
- Events and operational information

Mental model:

``` text
Application
   |
   v
Container stdout/stderr
   |
   v
GKE logging integration
   |
   v
Cloud Logging
```

## 📊 11.2 Monitoring

------------------------------------------------------------------------

Metrics include:

- CPU utilization
- Memory utilization
- Pod count
- Node health
- Request rate
- Error rate
- Latency
- Resource saturation

Architecture:

``` text
GKE
 |
 +-- Nodes
 +-- Pods
 +-- Workloads
 |
 v
Metrics
 |
 v
Cloud Monitoring
 |
 +-- Dashboards
 +-- Alerts
 +-- SLOs
```

## 11.3 Logs vs metrics

------------------------------------------------------------------------

| Logs                  | Metrics                      |
|-----------------------|------------------------------|
| Event/detail oriented | Numeric/time-series oriented |
| Good for debugging    | Good for trends and alerts   |
| Example: stack trace  | Example: CPU = 82%           |

## 💻 11.4 Useful kubectl commands

------------------------------------------------------------------------

``` bash
kubectl logs POD_NAME
```

Follow logs:

``` bash
kubectl logs -f POD_NAME
```

Previous container instance:

``` bash
kubectl logs POD_NAME --previous
```

Pod details/events:

``` bash
kubectl describe pod POD_NAME
```

Cluster events:

``` bash
kubectl get events --sort-by=.lastTimestamp
```

## 📊 11.5 Monitoring architecture for an application

------------------------------------------------------------------------

``` text
Users
  |
  v
Load Balancer
  |
  v
GKE Pods
  |
  +------------------+
  |                  |
  v                  v
Logs              Metrics
  |                  |
  v                  v
Cloud Logging    Cloud Monitoring
  |                  |
  +--------+---------+
           |
           v
       Dashboards
       Alerts
```

------------------------------------------------------------------------

# 📈 12. GKE Autoscaling

------------------------------------------------------------------------

There are different autoscaling layers. Do not mix them.

``` text
Application workload
       |
       +---- HPA ------> number of Pods
       |
       +---- VPA ------> Pod resource recommendations/requests
       |
       +---- Cluster/node autoscaling --> number of Nodes
```

## 12.1 HPA

------------------------------------------------------------------------

Horizontal Pod Autoscaler changes the number of replicas.

``` text
CPU high
   |
   v
HPA
   |
   v
2 Pods -> 5 Pods
```

Example:

``` yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
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

The Deployment must define meaningful resource requests for CPU-based
utilization calculations.

## 📈 12.2 Cluster Autoscaler / node-pool autoscaling

------------------------------------------------------------------------

Node autoscaling changes infrastructure capacity.

``` text
Pod cannot be scheduled
       |
       v
Node autoscaling
       |
       v
Add node
       |
       v
Pod gets scheduled
```

## 📈 12.3 HPA + node autoscaling

------------------------------------------------------------------------

This is the important combined scenario:

``` text
Traffic increases
       |
       v
HPA increases replicas
       |
       v
New Pods cannot fit
       |
       v
Node autoscaler adds capacity
       |
       v
Scheduler places Pods
```

## 12.4 VPA

------------------------------------------------------------------------

Vertical Pod Autoscaler works with resource sizing rather than replica
count.

Mental model:

``` text
Observed workload usage
          |
          v
VPA recommendation
          |
          v
CPU / memory sizing
```

Do not casually configure HPA and VPA to fight over the same resource
signal. Understand the scaling objective first.

## 🎯 12.5 Interview distinction

------------------------------------------------------------------------

> **HPA scales Pods. Node/cluster autoscaling scales Nodes. VPA
> changes/recommends Pod resource sizing.**

------------------------------------------------------------------------

# 🔄 13. Cluster Upgrades

------------------------------------------------------------------------

A production GKE cluster must remain on supported Kubernetes/GKE
versions.

Think about two major areas:

``` text
GKE cluster
 |
 +-- Control plane version
 |
 +-- Node pool version
```

## 13.1 Why upgrade?

------------------------------------------------------------------------

- Security fixes
- Bug fixes
- New Kubernetes/GKE capabilities
- Supported-version compliance
- Removal of obsolete/deprecated APIs

## 13.2 Upgrade planning

------------------------------------------------------------------------

Before production upgrade:

``` text
Check release notes
       |
       v
Check deprecated APIs
       |
       v
Test in lower environment
       |
       v
Check PDBs / replicas / capacity
       |
       v
Upgrade production
       |
       v
Validate workloads
```

## 13.3 Cordon

------------------------------------------------------------------------

Cordon marks a node unschedulable for new Pods.

``` bash
kubectl cordon NODE_NAME
```

Existing Pods are not automatically removed just because the node is
cordoned.

## 13.4 Drain

------------------------------------------------------------------------

Drain evicts eligible Pods so the node can be maintained.

``` bash
kubectl drain NODE_NAME --ignore-daemonsets
```

Use carefully. Production workloads, PDBs, local storage and special
scheduling rules can affect the result.

## 13.5 Uncordon

------------------------------------------------------------------------

``` bash
kubectl uncordon NODE_NAME
```

The node becomes schedulable again.

## 13.6 PodDisruptionBudget

------------------------------------------------------------------------

PDB helps protect availability during voluntary disruptions.

Example:

``` yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```

If a workload has five replicas and `minAvailable: 4`, the PDB expresses
that at least four should remain available during covered voluntary
disruptions.

Important:

> PDB is not a guarantee against every possible failure. It is primarily
> a disruption-availability control.

## 🎯 13.7 Upgrade interview answer

------------------------------------------------------------------------

> “Before a production GKE upgrade, I would check supported upgrade
> paths and release notes, identify deprecated APIs, validate workload
> compatibility, review PodDisruptionBudgets and capacity, test in
> staging, then upgrade in a controlled way and validate the application
> afterward.”

------------------------------------------------------------------------

# 🛡️ 14. GKE Security Basics

------------------------------------------------------------------------

GKE security is layered.

``` text
                    GKE Security
                         |
       +-----------------+-----------------+
       |                 |                 |
      IAM               RBAC        Network controls
       |                 |                 |
 Google resources   Kubernetes API   VPC / NetworkPolicy
       |
 Workload Identity
       |
       +-------------------------------+
                                       |
                               Container security
                                       |
                              Pod security / secrets
```

## 14.1 Least privilege

------------------------------------------------------------------------

Always ask:

> What is the minimum permission this identity needs?

Bad:

``` text
Application
   |
   +-- broad project Editor role
```

Better:

``` text
Application
   |
   +-- narrowly scoped role
```

Example:

``` text
Need: read Cloud Storage objects

Prefer:
roles/storage.objectViewer

rather than a broad administrative role.
```

## 🪪 14.2 Workload Identity instead of long-lived keys

------------------------------------------------------------------------

Avoid:

``` text
Pod
 |
 +-- service-account.json
```

Prefer:

``` text
Pod
 |
 v
KSA
 |
 v
Workload Identity
 |
 v
GSA
 |
 v
Google API
```

## 14.3 Kubernetes RBAC

------------------------------------------------------------------------

RBAC protects the Kubernetes API.

Example Role:

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

Binding:

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
  - kind: User
    name: user@example.com
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## 14.4 Role vs ClusterRole

------------------------------------------------------------------------

``` text
Role
 |
 +-- namespace-scoped permission definition

ClusterRole
 |
 +-- cluster-scoped permission definition
```

A ClusterRole can also be bound within a namespace through a
RoleBinding.

## 14.5 NetworkPolicy

------------------------------------------------------------------------

Use NetworkPolicy when you need Pod-to-Pod traffic restrictions.

Example:

``` text
frontend -> backend       ALLOW
payment  -> backend       DENY
unknown  -> backend       DENY
```

## 🛡️ 14.6 SecurityContext

------------------------------------------------------------------------

Use container/pod security settings to reduce privilege.

Example:

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: nginx:1.27
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
```

Important concepts:

- Avoid root where possible.
- Disable privilege escalation where possible.
- Use a read-only root filesystem where the application permits it.
- Drop unnecessary Linux capabilities.
- Use a suitable seccomp profile.
- Use trusted and patched images.

## 14.7 Secrets

------------------------------------------------------------------------

Do not bake secrets into images.

Bad:

``` dockerfile
ENV DB_PASSWORD=my-password
```

Also avoid putting credentials directly into Git.

Kubernetes Secrets can hold sensitive configuration, but production
architectures may use Google Secret Manager and an appropriate
integration instead.

Example Kubernetes Secret:

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: app
  password: example-password
```

## 🛡️ 14.8 Security layers

------------------------------------------------------------------------

A good interview architecture is:

``` text
User
 |
 v
Cloud Load Balancer
 |
 v
GKE Ingress / Service
 |
 v
NetworkPolicy
 |
 v
Pod
 |
 +-- SecurityContext
 +-- non-root
 +-- trusted image
 |
 v
KSA
 |
 v
Workload Identity
 |
 v
GSA
 |
 v
Least-privilege IAM
 |
 v
Google Cloud resource
```

------------------------------------------------------------------------

# 🏗️ 15. End-to-End Architecture

------------------------------------------------------------------------

This combines the completed GKE topics.

``` text
                                Users
                                  |
                                  v
                     +--------------------------+
                     | Google Cloud Load        |
                     | Balancer                 |
                     +------------+-------------+
                                  |
                                  v
                            GKE Ingress
                                  |
                       +----------+----------+
                       |                     |
                    /web                    /api
                       |                     |
                       v                     v
               frontend Service       backend Service
                       |                     |
                 +-----+-----+         +-----+-----+
                 |     |     |         |     |     |
                Pod   Pod   Pod       Pod   Pod   Pod
                                             |
                                             |
                                     Workload Identity
                                             |
                                             v
                                   Google Service Account
                                             |
                              +--------------+--------------+
                              |              |              |
                              v              v              v
                       Cloud Storage   Secret Manager    Pub/Sub
```

Infrastructure view:

``` text
                       GKE Regional Cluster
                                |
                +---------------+---------------+
                |                               |
          general-pool                     special-pool
                |                               |
             Nodes                           Nodes
                |                               |
              Pods                            Pods
                |
        HPA / scheduling
                |
        Node autoscaling
```

Security view:

``` text
Google Cloud IAM
       |
       +-- GSA permissions
       |
       +-- Workload Identity

Kubernetes RBAC
       |
       +-- Kubernetes API access

NetworkPolicy
       |
       +-- Pod-to-Pod restrictions

VPC Firewall
       |
       +-- VPC/network controls

SecurityContext
       |
       +-- Container/process security
```

Observability view:

``` text
Pods / Nodes / GKE
       |
       +------ Logs ------> Cloud Logging
       |
       +---- Metrics -----> Cloud Monitoring
       |
       +---- Alerts ------> Operations response
```

------------------------------------------------------------------------

# 🔎 16. Troubleshooting Playbook

------------------------------------------------------------------------

The important LevelUp skill is not just knowing commands; it is knowing
the order in which to investigate.

## 16.1 Pod is Pending

------------------------------------------------------------------------

``` text
Pod Pending
    |
    v
kubectl describe pod
    |
    v
Check Events
    |
    +-- insufficient CPU/memory?
    |
    +-- nodeSelector / affinity?
    |
    +-- taint/toleration?
    |
    +-- PVC / dependency issue?
    |
    +-- node autoscaling possible?
```

Commands:

``` bash
kubectl get pods -o wide
kubectl describe pod POD_NAME
kubectl get events --sort-by=.lastTimestamp
kubectl get nodes
```

## 16.2 Pod is CrashLoopBackOff

------------------------------------------------------------------------

``` text
CrashLoopBackOff
      |
      v
kubectl logs POD
      |
      v
kubectl logs POD --previous
      |
      v
kubectl describe pod POD
      |
      +-- app crash?
      +-- bad configuration?
      +-- missing Secret/ConfigMap?
      +-- permissions?
      +-- probe failure?
      +-- resource problem?
```

## 16.3 Service has no traffic

------------------------------------------------------------------------

``` text
Client
  |
  v
Service
  |
  +-- selector correct?
  |
  +-- Pod labels correct?
  |
  +-- endpoints exist?
  |
  +-- targetPort correct?
  |
  +-- Pods Ready?
  |
  +-- NetworkPolicy?
  |
  +-- Load Balancer / Ingress?
```

Commands:

``` bash
kubectl get svc
kubectl describe svc SERVICE_NAME
kubectl get endpoints SERVICE_NAME
kubectl get pods --show-labels
```

## 16.4 Workload cannot access Cloud Storage

------------------------------------------------------------------------

``` text
Pod
 |
 +-- serviceAccountName correct?
 |
 +-- KSA exists?
 |
 +-- KSA/GSA association correct?
 |
 +-- workloadIdentityUser binding?
 |
 +-- GSA has required storage role?
 |
 +-- correct bucket/project?
```

## 16.5 LoadBalancer not working

------------------------------------------------------------------------

Check:

``` bash
kubectl get svc
kubectl describe svc SERVICE_NAME
kubectl get pods -o wide
kubectl get endpoints SERVICE_NAME
```

Then investigate:

- Service selector
- Pod readiness
- Port/targetPort
- GKE load-balancer provisioning
- Firewall/network controls
- Health-check behavior
- Ingress configuration if using Ingress

------------------------------------------------------------------------

# 🎯 17. LevelUp Interview Questions

------------------------------------------------------------------------

## Q1. What is GKE?

------------------------------------------------------------------------

**Answer:**

GKE is Google Cloud’s managed Kubernetes service. It provides a managed
Kubernetes control plane and integrates Kubernetes workloads with Google
Cloud networking, IAM, load balancing, logging, monitoring and other
services.

## ⚖️ Q2. Standard vs Autopilot?

------------------------------------------------------------------------

**Answer:**

Standard provides more control over worker infrastructure and node
pools. Autopilot abstracts more of the worker infrastructure and reduces
operational overhead. I would choose Standard for workloads needing
deeper infrastructure control and Autopilot when the workload fits its
constraints and I want more managed operations.

## 🖥️ Q3. Why do we use node pools?

------------------------------------------------------------------------

**Answer:**

Node pools group nodes with common configuration. They allow different
workload classes to use different machine types, labels, taints or
capacity profiles.

## Q4. What is the difference between a node and a node pool?

------------------------------------------------------------------------

**Answer:**

A node is an individual worker machine. A node pool is a group of nodes
with a common configuration.

## Q5. Why do we need a Service?

------------------------------------------------------------------------

**Answer:**

Pods are ephemeral and their IP addresses can change. A Service provides
stable access and selects backend Pods through labels.

## Q6. ClusterIP vs LoadBalancer?

------------------------------------------------------------------------

**Answer:**

ClusterIP provides internal Service access. LoadBalancer exposes a
Service through cloud load-balancing integration and is commonly used
for external access.

## Q7. Does a LoadBalancer Service always mean I manually create NodePort and ClusterIP objects?

------------------------------------------------------------------------

**Answer:**

No. These are Service types/behaviors, not separate Services that I need
to create manually. The underlying implementation can depend on
Kubernetes/GKE configuration.

## Q8. What is the difference between Service and Ingress?

------------------------------------------------------------------------

**Answer:**

A Service gives stable access to a set of Pods. Ingress provides
HTTP/HTTPS routing to Services, often using host and path rules.

## Q9. What is VPC-native GKE?

------------------------------------------------------------------------

**Answer:**

VPC-native GKE integrates Pod and Service networking with the Google
Cloud VPC using secondary IP ranges, making GKE networking a first-class
part of the VPC design.

## Q10. NetworkPolicy vs VPC firewall?

------------------------------------------------------------------------

**Answer:**

VPC firewall controls traffic at the Google Cloud VPC/network layer.
NetworkPolicy controls allowed traffic between Kubernetes Pods according
to policy.

## Q11. IAM vs RBAC?

------------------------------------------------------------------------

**Answer:**

IAM controls authorization to Google Cloud resources. Kubernetes RBAC
controls authorization through the Kubernetes API.

## Q12. KSA vs GSA?

------------------------------------------------------------------------

**Answer:**

A KSA is a Kubernetes identity used by a workload. A GSA is a Google
Cloud identity used to access Google Cloud resources.

## 🪪 Q13. Why Workload Identity?

------------------------------------------------------------------------

**Answer:**

It allows GKE workloads to access Google Cloud APIs without storing
long-lived service-account JSON keys in Pods.

## Q14. Why do we need `roles/iam.workloadIdentityUser`?

------------------------------------------------------------------------

**Answer:**

It establishes the authorization for the Kubernetes workload identity to
use the target Google Cloud service account. It is separate from the
GSA’s resource permissions such as `roles/storage.objectViewer`.

## Q15. What if HPA creates more Pods but no node has enough capacity?

------------------------------------------------------------------------

**Answer:**

The new Pods can remain Pending. If node autoscaling is configured and
the workload is schedulable on additional nodes, the node autoscaler can
add capacity and the scheduler can then place the Pods.

## Q16. HPA vs node autoscaler?

------------------------------------------------------------------------

**Answer:**

HPA changes Pod replica count. Node autoscaling changes node capacity.

## 🔎 Q17. How would you troubleshoot a Pending Pod?

------------------------------------------------------------------------

**Answer:**

I would use `kubectl describe pod` and inspect events first. Then I
would check resource requests, node capacity, taints/tolerations,
selectors/affinity, storage/dependencies and whether node autoscaling
can add capacity.

## 🔎 Q18. How would you troubleshoot a Service that is not reaching its Pods?

------------------------------------------------------------------------

**Answer:**

I would verify the Service selector against Pod labels, inspect Service
endpoints, validate ports and targetPort, confirm Pods are Ready, and
then check NetworkPolicy and the external load-balancing path if
applicable.

## Q19. Why use a regional cluster?

------------------------------------------------------------------------

**Answer:**

A regional cluster provides better resilience across zones in a region
and is generally more suitable for production workloads that require
higher availability.

## Q20. What happens during node maintenance?

------------------------------------------------------------------------

**Answer:**

Nodes can be made unschedulable, workloads can be drained according to
Kubernetes disruption rules, the node can be upgraded/replaced, and
workloads are scheduled elsewhere if sufficient capacity exists.
PodDisruptionBudgets can help preserve availability during covered
voluntary disruptions.

------------------------------------------------------------------------

# 18. Final Revision Sheet

------------------------------------------------------------------------

## GKE hierarchy

------------------------------------------------------------------------

``` text
GKE Cluster
 |
 +-- Control Plane
 |
 +-- Node Pools
       |
       +-- Nodes
             |
             +-- Pods
                   |
                   +-- Containers
```

## 📦 Workloads

------------------------------------------------------------------------

``` text
Deployment  -> stateless replicas
StatefulSet -> stable identity/stateful workloads
DaemonSet   -> node-oriented workloads
Job         -> finite task
CronJob     -> scheduled task
```

## Networking

------------------------------------------------------------------------

``` text
Pod IP       -> workload address
Service IP   -> stable Service address
Node IP      -> node/network address

Service      -> stable access to Pods
Ingress      -> HTTP/HTTPS routing
LoadBalancer -> cloud external load-balancing integration
NetworkPolicy -> Pod traffic restrictions
VPC firewall  -> VPC/network traffic controls
```

## Identity

------------------------------------------------------------------------

``` text
KSA -> Kubernetes workload identity
GSA -> Google Cloud workload identity

Workload Identity:
KSA -> GSA -> Google Cloud APIs

IAM -> Google Cloud authorization
RBAC -> Kubernetes API authorization
```

## 📈 Autoscaling

------------------------------------------------------------------------

``` text
HPA                  -> Pods
VPA                  -> Pod resource sizing
Node/cluster scaling -> Nodes
```

## Observability

------------------------------------------------------------------------

``` text
Logs    -> what happened
Metrics -> how the system behaves
```

## 🛡️ Security

------------------------------------------------------------------------

``` text
Least privilege
Workload Identity
IAM
RBAC
NetworkPolicy
VPC firewall
SecurityContext
Trusted images
Secret management
```

## Upgrades

------------------------------------------------------------------------

``` text
Plan
 |
 v
Check compatibility / deprecated APIs
 |
 v
Test staging
 |
 v
Check PDB / capacity
 |
 v
Upgrade
 |
 v
Validate
```

------------------------------------------------------------------------

# ✅ 19. Final LevelUp Checklist

------------------------------------------------------------------------

You should be able to explain each item without looking at notes:

- [ ] What GKE is
- [ ] GKE control plane and worker architecture
- [ ] Zonal vs regional cluster
- [ ] Standard vs Autopilot
- [ ] Cluster creation through Console, gcloud and Terraform
- [ ] Node pools
- [ ] Node labels
- [ ] Taints and tolerations
- [ ] Deployment
- [ ] StatefulSet
- [ ] DaemonSet
- [ ] Job
- [ ] CronJob
- [ ] ClusterIP
- [ ] NodePort
- [ ] LoadBalancer
- [ ] Service selectors
- [ ] Ingress
- [ ] L4 vs L7
- [ ] VPC-native networking
- [ ] Node IP vs Pod IP vs Service IP
- [ ] NetworkPolicy
- [ ] VPC firewall vs NetworkPolicy
- [ ] IAM integration
- [ ] IAM vs RBAC
- [ ] KSA vs GSA
- [ ] Workload Identity
- [ ] `roles/iam.workloadIdentityUser`
- [ ] Service account impersonation
- [ ] `roles/iam.serviceAccountTokenCreator`
- [ ] Logging
- [ ] Monitoring
- [ ] Kubernetes events
- [ ] HPA
- [ ] VPA
- [ ] Node/cluster autoscaling
- [ ] HPA + node autoscaling interaction
- [ ] Cluster upgrades
- [ ] Cordon
- [ ] Drain
- [ ] Uncordon
- [ ] PodDisruptionBudget
- [ ] Least privilege
- [ ] SecurityContext
- [ ] Secrets
- [ ] GKE troubleshooting

------------------------------------------------------------------------

# Final Mental Model

------------------------------------------------------------------------

If asked to explain GKE from scratch, use this flow:

``` text
Google Cloud
     |
     v
    GKE
     |
     +-- Control Plane
     |
     +-- Node Pools
           |
           +-- Nodes
                 |
                 +-- Pods
                       |
                       +-- Workloads
                              |
                              +-- Deployment
                              +-- StatefulSet
                              +-- DaemonSet
                              +-- Job / CronJob

Users
  |
  v
Load Balancer / Ingress
  |
  v
Services
  |
  v
Pods

Pods
 |
 +-- KSA
      |
      v
 Workload Identity
      |
      v
     GSA
      |
      v
Google Cloud APIs

Operations:
Logs -> Cloud Logging
Metrics -> Cloud Monitoring
HPA -> Pods
Node autoscaling -> Nodes
Upgrades -> supported cluster/node versions

Security:
IAM -> Google Cloud
RBAC -> Kubernetes API
NetworkPolicy -> Pod traffic
VPC Firewall -> VPC traffic
SecurityContext -> container/process security
Least privilege -> everywhere
```

> **LevelUp rule:** For every topic, be ready to explain **what it is,
> why it exists, how it works, how you configure it, and how you
> troubleshoot it.**

------------------------------------------------------------------------

# End of GKE Notes

------------------------------------------------------------------------
