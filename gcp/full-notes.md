# 📘 GCP Full Notes

> Enterprise GCP source-of-truth notes.
>
> This handbook is designed for interview preparation and practical engineering. Each topic includes definition, purpose, use case, examples, important points, and interview nuggets.

---

## How to use this file
- **Definition** = what the concept means.
- **Use case** = why teams use it.
- **Example** = real-world or conceptual example.
- **Important points** = what to remember in interviews.
- **Interview nuggets** = fast revision points.

---

# Chapter 1: Why Cloud, Why GCP

## 1. Why Cloud Computing?
Cloud computing solves the problems of buying, installing, maintaining, and scaling physical infrastructure. Instead of owning hardware, organizations consume infrastructure and managed services on demand.

### Use case
When a company needs to launch quickly, scale on demand, or reduce hardware operations.

### Example
```text
Need 100 servers -> request 100 VMs in minutes instead of buying hardware
```

### Important points
- Reduces capital expenditure.
- Improves agility and scalability.
- Adds managed services and operational simplicity.
- Supports high availability and disaster recovery.

## 2. Why GCP?
GCP is often chosen for its global private network, strong Kubernetes/GKE offering, analytics and AI capabilities, simple global networking model, enterprise security features, and strong managed services.

### Use case
Cloud-native workloads, container platforms, analytics-heavy systems, and enterprises needing strong global networking.

### Example
- GKE for Kubernetes
- BigQuery for analytics
- Vertex AI for ML/AI
- Shared VPC for enterprise networking

### Important points
- Google’s private global network is a major differentiator.
- GKE is a flagship service.
- BigQuery and Vertex AI are strong for data and AI workloads.
- Cloud choice is workload-driven, not absolute.

---

# Chapter 2: GCP Global Infrastructure

## 3. GCP Global Infrastructure
GCP operates a worldwide infrastructure made up of regions, zones, edge locations, and Google’s private backbone.

### Use case
Serving users globally with lower latency, strong reliability, and optimized routing.

### Example
```text
Users -> Google edge -> private backbone -> destination region
```

### Important points
- GCP is built on a global infrastructure model.
- The private backbone improves performance and reliability.
- Regions and zones are the core placement units.

## 4. Region
A region is a geographic area that contains multiple zones and represents a broader failure and placement domain.

### Use case
Choose a region near users or where compliance and service availability requirements are met.

### Example
- `asia-south1` (Mumbai)
- `us-central1` (Iowa)
- `europe-west2` (London)

### Important points
- Region choice impacts latency, compliance, cost, and service availability.
- A region contains multiple zones.
- Regional selection is a business and architecture decision.

## 5. Zone
A zone is an isolated failure domain within a region.

### Use case
Deploy workloads across multiple zones for high availability.

### Example
```text
asia-south1-a, asia-south1-b, asia-south1-c
```

### Important points
- A zone is not the same as a region.
- Single-zone deployments are more vulnerable to outages.
- Multi-zone designs improve availability.

## 6. Resource Scope
GCP resources are designed with appropriate scope: global, regional, or zonal.

### Use case
Understand where resources naturally belong and how to design HA and DR.

### Example
- Global: VPC, routes, firewall rules, IAM
- Regional: subnet, Cloud NAT, Cloud Router, Cloud SQL
- Zonal: Compute Engine VM, GPU, TPU

### Important points
- Global resources are shared across the project’s geography.
- Regional resources belong to one region.
- Zonal resources belong to one zone.

---

# Chapter 3: Resource Hierarchy

## 7. Resource Hierarchy
GCP organizes resources in a hierarchy: Organization -> Folder -> Project -> Resources.

### Use case
Manage ownership, billing, policies, and access control at scale.

### Example
```text
Organization
 ├── Folder
 │    └── Project
 │         └── Resources
```

### Important points
- Organization represents the company.
- Folders are optional groupings.
- Projects are the primary management boundary.

## 8. Organization
An Organization is the root resource for a company in GCP.

### Use case
Apply company-wide policies and manage all subordinate folders and projects.

### Example
```text
yourcompany.com
```

### Important points
- Usually linked to Cloud Identity or Google Workspace.
- Typically one organization per company.
- Enables centralized governance.

## 9. Folder
Folders help organize projects and can represent environments, teams, or business units.

### Use case
Separate dev, QA, prod, or line-of-business groupings.

### Example
- Production folder
- Development folder
- Finance folder

### Important points
- Optional but useful in enterprises.
- Helps with policy inheritance and organization.
- Supports cleaner governance.

## 10. Project
A project is the primary operational boundary in GCP. Most resources live inside a project.

### Use case
Separate environments, billing, API enablement, IAM, and quotas.

### Example
- `app-dev`
- `app-prod`
- `shared-network`

### Important points
- Every resource belongs to one project.
- Project is the heart of GCP.
- Projects isolate billing, IAM, APIs, and quotas.

## 11. Project Identity
A project has a project name, project ID, and project number.

### Use case
Different identifiers are used by humans, APIs, and internal Google systems.

### Example
- Project Name: `Production Application`
- Project ID: `epam-prod-001`
- Project Number: `847362918472`

### Important points
- Project name is human-friendly and changeable.
- Project ID is globally unique and permanent.
- Project number is Google-generated and immutable.

## 12. Billing Account
Billing accounts are linked to projects to pay for resources and manage cost controls.

### Use case
Centralize finance and cost management across multiple projects.

### Example
```text
Billing Account -> Project A -> resources
Billing Account -> Project B -> resources
```

### Important points
- One billing account can pay for many projects.
- One project can have one active billing account.
- Budgets and billing export are enterprise essentials.

## 13. Quotas
Quotas limit resource consumption and help with capacity management and abuse prevention.

### Use case
Control the number of VMs, CPUs, IPs, or other service-specific resources.

### Example
- 24 CPUs for a project by default
- Request increase to 5000 CPUs for large enterprises

### Important points
- Quotas are not purely based on billing tier.
- They are influenced by trust, request history, region capacity, and approval.
- Some quotas are regional, zonal, or global.

---

# Chapter 4: Networking Foundation

## 14. How Networking Works in Cloud
Every VM or compute resource needs an IP, routing, security filtering, and sometimes internet access.

### Use case
Understand the basic dependencies behind cloud networking.

### Example
```text
VM -> subnet -> route -> firewall -> destination
```

### Important points
- Networking is foundational for almost all GCP services.
- VPC provides the private network.
- Subnet provides IP allocation.
- Routes decide where traffic goes.
- Firewalls decide whether traffic is allowed.

## 15. Packet Flow
A packet in GCP moves through interfaces, subnet, route lookup, firewall evaluation, and then reaches the destination.

### Use case
Troubleshoot connectivity problems logically.

### Example
```text
Application -> NIC -> subnet -> route lookup -> firewall -> destination
```

### Important points
- A subnet does not route traffic.
- Routes answer “where to go.”
- Firewalls answer “allowed or denied.”

---

# Chapter 5: VPC

## 16. VPC
A Virtual Private Cloud is your private network inside GCP where cloud resources communicate securely.

### Use case
Private communication among VMs, databases, GKE nodes, and other services.

### Example
```text
VPC -> subnets -> VMs -> internal communication
```

### Important points
- VPC is global in GCP.
- A VPC is not a router or firewall device.
- It is the logical network container for resources.

## 17. Why VPC is Global
VPC is global because it allows one logical network to span multiple regions while keeping routing and policy centralized.

### Use case
Support workloads in Mumbai, Singapore, and Tokyo within one network design.

### Example
```text
One global VPC -> multiple regional subnets
```

### Important points
- One VPC can contain subnets in many regions.
- Simplifies enterprise multi-region networking.
- Avoids creating separate networks per region unnecessarily.

## 18. Auto Mode vs Custom Mode VPC
Auto Mode creates subnets automatically in supported regions; Custom Mode gives full control over subnet creation and CIDR planning.

### Use case
Auto Mode for quick labs; Custom Mode for enterprise production.

### Example
- Auto Mode: Google creates subnet ranges automatically
- Custom Mode: You define subnets and ranges manually

### Important points
- Enterprises usually choose Custom Mode.
- Custom Mode supports controlled IP planning and compliance.
- Auto -> Custom conversion is supported; Custom -> Auto is not.

---

# Chapter 6: Subnets

## 19. Subnet
A subnet is a regional IP range inside a VPC from which private IP addresses are allocated.

### Use case
Assign private IPs to resources in a specific region.

### Example
```text
Mumbai subnet -> 10.10.1.0/24
Singapore subnet -> 10.20.1.0/24
```

### Important points
- Subnets are regional.
- A subnet owns an IP range.
- VMs get their private IPs from subnets.

## 20. Why Subnets are Regional
Subnets are regional to support clear IP planning and regional placement of resources.

### Use case
Organize different regional ranges without ambiguity.

### Example
```text
asia-south1 -> 10.10.1.0/24
asia-southeast1 -> 10.20.1.0/24
```

### Important points
- One subnet belongs to one region.
- Multiple subnets can exist in one VPC.
- No overlapping CIDRs within the same network design.

---

# Chapter 7: Firewalls, Tags, Labels, and Identity

## 21. Firewall Rules
Firewall rules control whether traffic is allowed or denied between resources in a VPC.

### Use case
Protect VMs, databases, and internal services.

### Example
```text
Allow TCP 22 from office IPs only
```

### Important points
- Firewall belongs to the VPC, not the VM.
- Firewalls are stateful.
- Lower priority number means higher priority.

## 22. Source IP Ranges
Source IP ranges define which source CIDRs are allowed for ingress firewall rules.

### Use case
Allow SSH only from office IPs or app subnet only to database subnet.

### Example
```text
source_ranges = ["10.10.2.0/24"]
```

### Important points
- Source ranges answer “who is sending traffic?”
- Used mainly for ingress rules.
- CIDR notation allows single IPs, subnets, or broad ranges.

## 23. Network Tags
Network tags are simple strings attached to VMs to target firewall rules or routes.

### Use case
Apply firewall rules to web servers, database servers, or admin machines.

### Example
```text
VM tags = ["web", "production"]
```

### Important points
- Used for networking policy targeting.
- Tags are not labels.
- Tags are not identity.

## 24. Labels
Labels are key-value metadata attached to resources for organization, billing, and filtering.

### Use case
Cost allocation, search, reporting, and automation.

### Example
```text
env=prod, team=payments, owner=platform
```

### Important points
- Labels are not used for firewall rules.
- Labels are for metadata, not networking.
- Labels are useful for billing and reporting.

## 25. Service Accounts
Service accounts are workload identities and can be used as firewall targets and for IAM.

### Use case
Identity-based access and stronger enterprise security.

### Example
```text
web-sa@project.iam.gserviceaccount.com
```

### Important points
- More secure than simple tags for many enterprise use cases.
- A VM typically has one attached service account.
- A service account is an identity, not just a label.

## 26. Network Tags vs Labels vs Service Accounts
These three concepts are different: tags are for networking targets, labels are for metadata, and service accounts are for identity.

### Use case
Use the right mechanism for security, metadata, and billing separately.

### Example
- Firewall target: tag or service account
- Billing: labels
- Identity: service account

### Important points
- Labels do not affect firewall rules.
- Tags are networking-only.
- Service accounts are identity-based and preferred for production security.

---

# Chapter 8: Routing

## 27. Routes
Routes tell GCP where to send traffic based on the destination IP range.

### Use case
Move traffic between subnets, to on-prem networks, or to the internet.

### Example
```text
Destination 10.10.3.0/24 -> next hop local subnet
```

### Important points
- Routes answer “where should traffic go?”
- GCP uses longest prefix match.
- Routes do not allow or deny traffic.

## 28. System Routes
System routes are automatically created by Google, including routes for subnets and default connectivity.

### Use case
Enable communication inside the VPC without manual route creation.

### Example
```text
Subnet creation -> route for that subnet is automatically available
```

### Important points
- You do not manually manage most system routes.
- They are essential for internal connectivity.
- They support the VPC’s private network behavior.

## 29. Custom Routes
Custom routes let you add specific routing behavior for hybrid connectivity or special network paths.

### Use case
Route traffic to on-prem or special next hops.

### Example
```text
192.168.100.0/24 -> VPN gateway
```

### Important points
- Used for hybrid connectivity and custom traffic patterns.
- Should avoid overlapping with more specific routes.
- Can be static or dynamic.

## 30. Default Internet Route
The default route (`0.0.0.0/0`) is used when no more specific route matches and typically points to the internet gateway.

### Use case
Allow outbound internet access where appropriate.

### Example
```text
0.0.0.0/0 -> default internet gateway
```

### Important points
- The route alone does not guarantee internet access.
- External IPs or Cloud NAT may still be required.
- More specific routes override the default route.

## 31. Static vs Dynamic Routes
Static routes are manually created; dynamic routes are learned automatically through Cloud Router and BGP.

### Use case
Static for fixed paths, dynamic for hybrid networks.

### Example
- Static: manually route a CIDR to VPN
- Dynamic: learn on-prem routes automatically

### Important points
- Dynamic routing reduces manual overhead.
- Cloud Router enables dynamic route exchange.
- Route priorities matter.

---

# Chapter 9: Cloud Router and Cloud NAT

## 32. Cloud Router
Cloud Router is a regional service that exchanges dynamic routes with on-prem or other connected networks using BGP.

### Use case
Hybrid connectivity and dynamic route exchange.

### Example
```text
On-prem routes -> Cloud Router -> GCP
```

### Important points
- Cloud Router does not forward traffic like a traditional router.
- It participates in route exchange.
- Works with VPN and Interconnect.

## 33. Cloud NAT
Cloud NAT provides outbound internet access for private instances without external IPs.

### Use case
Private VMs or private GKE nodes need outbound internet access.

### Example
```text
Private VM -> Cloud NAT -> Internet
```

### Important points
- Outbound only.
- Regional service.
- Common in private clusters and secure production networks.

---

# Chapter 10: Load Balancing and Internet Access

## 34. Load Balancing
Load balancing distributes traffic across healthy backend instances and improves availability and scalability.

### Use case
Serve web traffic reliably and scale applications.

### Example
```text
Users -> Load Balancer -> healthy backends
```

### Important points
- Load balancing often includes health checks.
- Regional or global scope depends on the load balancer type.
- Used heavily in production architectures.

## 35. Internet Access Patterns
GCP workloads may use external IPs, Cloud NAT, or load balancers depending on the traffic direction and security model.

### Use case
Private egress, public ingress, or controlled exposure.

### Example
- Private VM outbound: Cloud NAT
- Public app ingress: Load balancer
- Public admin access: external IP with strong controls

### Important points
- Different patterns serve different use cases.
- Internet access is not automatic.
- Security and architecture must be considered together.

---

# Chapter 11: Shared VPC and Connectivity

## 36. Shared VPC
Shared VPC lets one project own the network while other projects consume it.

### Use case
Central network team manages networking; app teams use shared network services.

### Example
```text
Shared VPC project -> service projects
```

### Important points
- Strong enterprise pattern.
- Separates networking ownership from application ownership.
- Common in multi-team organizations.

## 37. VPC Peering
VPC peering connects two VPCs so they can communicate privately.

### Use case
Connect separate networks without using the public internet.

### Example
```text
VPC A <-> VPC B
```

### Important points
- No transitive routing by default.
- Useful for private network connectivity.
- Different from Shared VPC.

## 38. Private Service Connect
Private Service Connect enables private access to producer services over Google’s network.

### Use case
Access managed services privately without exposing them publicly.

### Example
```text
Consumer VPC -> PSC -> producer service
```

### Important points
- Supports private, controlled connectivity.
- Useful for enterprise-grade service exposure.
- Often used in security-conscious designs.

---

# Chapter 12: IAM and Security

## 39. IAM
IAM controls who can do what on which resource in GCP.

### Use case
Grant least-privilege access to users, groups, and service accounts.

### Example
```text
User -> role -> project or resource
```

### Important points
- One of the most important GCP security topics.
- Often inherited through hierarchy.
- Must be designed carefully in enterprise setups.

## 40. Organization Policies
Organization policies enforce security and compliance guardrails at scale.

### Use case
Prevent risky configurations across projects.

### Example
- Restrict public IPs
- Limit VM machine types
- Require uniform policies

### Important points
- Used at org/folder/project scopes.
- Great for guardrails.
- Complements IAM, not replaces it.

## 41. KMS
Cloud KMS manages encryption keys for sensitive data protection.

### Use case
Customer-managed encryption keys (CMEK) and encryption governance.

### Example
```text
Data -> encrypted with KMS-managed key
```

### Important points
- Important for compliance-heavy organizations.
- Works with many GCP services.
- Key lifecycle and access need governance.

## 42. Secret Manager
Secret Manager securely stores secrets such as API keys, passwords, and tokens.

### Use case
Centralized secret storage for applications and CI/CD.

### Example
```text
Application -> Secret Manager -> secret value
```

### Important points
- Better than storing secrets in code or plaintext files.
- Integrates with IAM.
- Frequently used in production pipelines.

## 43. Binary Authorization
Binary Authorization ensures only trusted, signed container images are allowed to deploy.

### Use case
Supply-chain security for containerized workloads.

### Example
```text
Signed image -> admission allowed
Unsigned image -> denied
```

### Important points
- Focuses on trust and signature verification.
- Common in GKE and regulated environments.
- Not the same as vulnerability scanning.

## 44. Workload Identity
Workload Identity allows workloads to securely authenticate to Google services without long-lived keys.

### Use case
GKE workloads accessing Google APIs securely.

### Example
```text
Kubernetes service account -> Google service account
```

### Important points
- Stronger than static service account keys.
- Important for modern GKE security.
- Reduces credential sprawl.

## 45. VPC Service Controls
VPC Service Controls help create service perimeters to reduce data exfiltration risks.

### Use case
Protect sensitive managed services and prevent data leakage.

### Example
```text
Perimeter -> allowed Google services and resources
```

### Important points
- Strong enterprise security feature.
- Often used in regulated industries.
- Protects against exfiltration paths.

---

# Chapter 13: Compute, GKE, and Containers

## 46. Compute Engine
Compute Engine provides virtual machines in GCP.

### Use case
Traditional VM workloads, lift-and-shift, custom server management.

### Example
```text
VM in a chosen region and zone
```

### Important points
- Zonal resource.
- Often combined with disks, firewalls, and load balancing.
- Core infrastructure service.

## 47. Managed Instance Groups
Managed Instance Groups manage and scale VMs across zones for availability and updates.

### Use case
High availability and scaling for VM-based applications.

### Example
```text
Regional MIG -> instances across zones
```

### Important points
- Great for HA and rolling updates.
- Common backend for load balancers.
- Supports autoscaling and autohealing.

## 48. GKE
Google Kubernetes Engine is Google’s managed Kubernetes platform.

### Use case
Container orchestration for microservices and cloud-native workloads.

### Example
```text
Cluster -> node pools -> workloads
```

### Important points
- Very important for enterprise interviews.
- Integrates with IAM, Workload Identity, Binary Authorization, Cloud NAT, and logging/monitoring.
- Standard and Autopilot modes exist.

## 49. Cloud Run
Cloud Run is a serverless container platform.

### Use case
Deploy stateless containers without managing servers.

### Example
```text
Container image -> Cloud Run service
```

### Important points
- Good for simple microservices and APIs.
- Scales quickly.
- Integrates with IAM and managed infrastructure.

---

# Chapter 14: Storage and Databases

## 50. Cloud Storage
Cloud Storage is object storage for files, backups, artifacts, and static content.

### Use case
Store backups, logs, artifacts, and application data objects.

### Example
```text
Bucket -> objects
```

### Important points
- Highly scalable object storage.
- Has storage classes and lifecycle policies.
- Often used with logs and backups.

## 51. Persistent Disk
Persistent Disk is block storage for Compute Engine VMs.

### Use case
VM boot disks, data disks, and attached storage.

### Example
```text
VM -> attached disk
```

### Important points
- Zonal or regional depending on the disk type.
- Common in VM-based workloads.
- Often paired with snapshots.

## 52. Cloud SQL
Cloud SQL is a managed relational database service.

### Use case
Managed MySQL, PostgreSQL, or SQL Server workloads.

### Example
```text
Application -> Cloud SQL
```

### Important points
- Common enterprise database service.
- Supports HA and backups.
- Networking and IAM are key design considerations.

---

# Chapter 15: DevOps, CI/CD, and Operations

## 53. Cloud Build
Cloud Build is GCP’s build and CI service.

### Use case
Build container images, run tests, and automate delivery steps.

### Example
```text
Git push -> build -> test -> push artifact
```

### Important points
- Common in CI/CD pipelines.
- Integrates with Artifact Registry and deployment workflows.
- Great to know for production engineering interviews.

## 54. Artifact Registry
Artifact Registry stores container images and other artifacts securely.

### Use case
Central artifact storage for CI/CD and deployment.

### Example
```text
Build pipeline -> Artifact Registry -> deploy
```

### Important points
- Replaces older image registry patterns in many modern setups.
- Integrates with Cloud Build and GKE.
- Important for secure supply chains.

## 55. Cloud Monitoring
Cloud Monitoring helps observe metrics, dashboards, and alerts.

### Use case
Track health, performance, and capacity.

### Example
```text
CPU -> dashboard -> alert
```

### Important points
- Essential for production observability.
- Alerts and dashboards are interview-relevant.
- Works with many GCP services.

## 56. Cloud Logging
Cloud Logging collects logs from workloads and managed services.

### Use case
Troubleshooting, auditing, and production diagnostics.

### Example
```text
Application logs -> Cloud Logging -> analysis
```

### Important points
- Very important for troubleshooting interviews.
- Often paired with Monitoring.
- Centralized logging is common in enterprises.

---

# Chapter 16: High Availability, Disaster Recovery, and Cost

## 57. High Availability
High availability is about designing systems to keep working during component or zone failures.

### Use case
Production applications that must stay online.

### Example
```text
Multi-zone deployment behind a load balancer
```

### Important points
- HA usually focuses on zone failures.
- Requires redundancy and health checks.
- Different from disaster recovery.

## 58. Disaster Recovery
Disaster recovery is about restoring service after a broader failure, often across regions.

### Use case
Region outage recovery with backups or cross-region architectures.

### Example
```text
Primary region -> DR region
```

### Important points
- Includes RPO and RTO.
- Needs backup and restore planning.
- Usually more expensive than HA.

## 59. Cost Optimization
Cost optimization helps control cloud spend through rightsizing, budgets, quotas, and efficient architecture.

### Use case
Keep cloud consumption within budget while meeting performance and reliability goals.

### Example
- Budgets and alerts
- Rightsizing VMs
- Using committed use discounts where appropriate

### Important points
- Labels help cost allocation.
- Billing export to BigQuery helps analysis.
- Cost management is an operational concern, not only finance.

---

# Chapter 17: Troubleshooting and Design Thinking

## 60. Troubleshooting Mindset
Troubleshooting in GCP should follow a systematic path: identity, network, routes, firewalls, DNS, service health, and logs.

### Use case
Connectivity issues, deployment failures, and performance problems.

### Example
```text
Check IP -> route -> firewall -> service -> logs
```

### Important points
- Don’t guess too early.
- Always check the full packet path.
- Logs and monitoring are essential.

## 61. Enterprise Architecture Mindset
Enterprise GCP design is about separation of concerns, security boundaries, shared services, observability, and operational safety.

### Use case
Design multi-project, multi-team, regulated production systems.

### Example
- Shared VPC project
- App projects
- Logging/monitoring projects
- Separate dev/prod projects

### Important points
- Architecture is more important than service definitions.
- Identity and network design are foundational.
- Use projects and folders intentionally.

---

# Chapter 18: Quick Interview Nuggets

- GCP region = geographic area.
- Zone = failure domain within a region.
- VPC = global private network.
- Subnet = regional IP range.
- Firewall = allow/deny traffic.
- Route = where traffic goes.
- Network tags = network targeting.
- Labels = metadata and billing.
- Service accounts = workload identity.
- Project = primary operational boundary.
- Billing account pays for projects.
- Quotas are controlled and adjustable.
- Custom Mode VPC is preferred for enterprise.
- Cloud NAT = outbound internet for private resources.
- Cloud Router = dynamic route exchange.
- Shared VPC = network ownership separated from app ownership.
- IAM and organization policies provide governance.
- Binary Authorization protects container supply chains.
- Workload Identity reduces key usage in GKE.
- Logs + metrics + alerts are essential for production.

---

# Chapter 19: First Revision Checklist
- Why cloud and why GCP
- Global infrastructure
- Regions, zones, resource scope
- Resource hierarchy
- Projects, billing, quotas
- Networking fundamentals
- VPC, subnets, auto vs custom mode
- Packet flow
- Firewalls, source ranges
- Tags vs labels vs service accounts
- Routes, static vs dynamic
- Cloud Router and Cloud NAT
- Shared VPC, peering, PSC
- IAM, org policies, KMS, Secret Manager
- Binary Authorization, Workload Identity, VPC Service Controls
- Compute, MIG, GKE, Cloud Run
- Cloud Storage, Persistent Disk, Cloud SQL
- Cloud Build, Artifact Registry
- Monitoring, Logging
- HA, DR, cost optimization
- Troubleshooting and enterprise architecture
