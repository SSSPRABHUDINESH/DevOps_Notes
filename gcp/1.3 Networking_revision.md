# ⚡ GCP Revision Cheat Sheet

> Short, easy-to-revise notes for interview prep.
>
> Focus: what it is, what scope it has, and what interviewers usually care about.

---

## Foundation

| Topic | Remember |
|---|---|
| GCP global infrastructure | Google’s worldwide cloud network built around regions, zones, and edge locations |
| Region | Geographic location with multiple zones |
| Zone | Independent failure domain inside a region |
| Resource hierarchy | Organization → Folder → Project → Resource |
| Project | Main operational boundary in GCP |
| Billing account | Pays for one or more projects |
| Resource scope | Global / Regional / Zonal |

### Quick points
- Use the nearest region when latency, compliance, and service availability allow it.
- Projects isolate IAM, billing, APIs, quotas, logs, and monitoring.
- Project ID is human/API facing; Project Number is Google’s internal immutable identifier.

---

## Networking

| Topic | Remember |
|---|---|
| VPC | Global private network in GCP |
| Subnet | Regional IP range inside a VPC |
| Auto Mode VPC | Google creates subnets automatically |
| Custom Mode VPC | You create subnets manually |
| Packet flow | Route decides where; firewall decides whether |
| Firewall | Attached to VPC, not VM |
| Network tags | Simple strings used for firewall targeting |
| Service accounts | Identity-based firewall targeting |
| Routes | Decide packet destination |

### Quick points
- VPC is global; subnets are regional; VMs are zonal.
- Firewall is stateful.
- Lower firewall priority number wins.
- `source_ranges` is for allowed source CIDRs in ingress rules.
- Labels are not for networking; they are metadata.

---

## Networking design choices

| Question | Short answer |
|---|---|
| Auto Mode or Custom Mode? | Custom for enterprise, Auto for labs |
| Tags or Service Accounts? | Service Accounts for stronger enterprise security |
| Labels or Tags? | Labels for metadata/billing, Tags for firewall targeting |

### Quick points
- Auto → Custom conversion is supported.
- Custom → Auto conversion is not supported.
- One VPC can have multiple subnets in multiple regions.

---

## Billing and quotas

| Topic | Remember |
|---|---|
| Quotas | Resource limits per project/region/service |
| Billing export | Useful for cost allocation and dashboards |
| Budgets | Alerts when spend crosses thresholds |

### Quick points
- Quota increases are usually requested, not bought as a simple tier upgrade.
- Quota ≠ capacity; a region can still run out of capacity even if quota is available.
- Labels are very important for chargeback/showback.

---

## Security

| Topic | Remember |
|---|---|
| IAM | Identity and access management |
| Service accounts | Workload identity |
| Impersonation | Let one identity act as another |
| Binary Authorization | Allow only trusted/signed container images |
| KMS | Key management |
| Secret Manager | Store secrets securely |
| VPC Service Controls | Data exfiltration protection |
| Organization Policies | Company-wide guardrails |

### Quick points
- Binary Authorization is about trust, not vulnerability scanning.
- Workload Identity is a major GKE security feature.
- Least privilege is the default mindset.

---

## Compute and containers

| Topic | Remember |
|---|---|
| Compute Engine | Virtual machines |
| MIG | Managed Instance Group for scalable VMs |
| GKE | Managed Kubernetes |
| Cloud Run | Serverless containers |
| Cloud Functions | Event-driven serverless functions |
| Artifact Registry | Container image and package storage |

### Quick points
- GKE often needs VPC, subnets, IAM, service accounts, Artifact Registry, logging, and monitoring.
- GKE security topics to remember: Workload Identity, Binary Authorization, private clusters.
- MIGs are used for HA and autoscaling of VMs.

---

## Storage and databases

| Topic | Remember |
|---|---|
| Cloud Storage | Object storage |
| Persistent Disk | VM block storage |
| Filestore | Managed NFS |
| Cloud SQL | Managed relational database |
| Spanner | Globally distributed relational database |
| Memorystore | Managed Redis/Memcached |

### Quick points
- Choose the storage type based on access pattern, performance, and consistency needs.
- Cloud SQL HA is regional; DR is usually cross-region planning.

---

## DevOps and operations

| Topic | Remember |
|---|---|
| Cloud Build | Build and CI service |
| Cloud Deploy | Deployment orchestration |
| Logging | Centralized logs |
| Monitoring | Metrics, alerts, dashboards |
| Operations Suite | Logging + Monitoring + tracing ecosystem |

### Quick points
- Production interviews often ask about dashboards, alerts, and metrics you actually monitored.
- Cloud Build + Artifact Registry is a common image pipeline pattern.

---

## High availability and DR

| Topic | Remember |
|---|---|
| High availability | Survive zone failure |
| Disaster recovery | Survive region failure |
| Multi-zone | Standard HA pattern |
| Multi-region | DR or global service pattern |

### Quick points
- Zone failure is an HA design problem.
- Region failure is a DR design problem.
- You usually start with multi-zone first, then add DR.

---

## Enterprise architecture

| Topic | Remember |
|---|---|
| Shared VPC | Central networking project for multiple service projects |
| Load balancing | Global or regional entry point for traffic |
| Private access | Keep traffic off the public internet where possible |
| Production design | Separate Dev/QA/Stage/Prod projects |

### Quick points
- Interviewers love asking how you would design enterprise landing zones.
- Shared VPC separates network ownership from application ownership.
- Think in terms of ownership, security, and failure domains.

---

## Fast recall

- Region = geographic area.
- Zone = failure domain.
- Project = main boundary.
- VPC = global network.
- Subnet = regional IP pool.
- Firewall = VPC-level policy.
- Routes = where traffic goes.
- Tags = networking labels.
- Labels = metadata for billing and organization.
- Service accounts = workload identity.
- Binary Authorization = only trusted images deploy.
- Shared VPC = centralized networking for many projects.
