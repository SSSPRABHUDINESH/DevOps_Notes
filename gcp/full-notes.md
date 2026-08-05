# 📘 GCP Full Notes

> Enterprise GCP source-of-truth notes.
>
> This handbook is designed for interview preparation and practical engineering. Each topic includes definition, purpose, use case, examples, important points, and interview nuggets.
>
> This version covers the GCP foundation topics discussed so far: why cloud, why GCP, global infrastructure, regions, zones, resource hierarchy, projects, billing, resource scope, networking foundations, VPC, subnets, auto mode vs custom mode, packet flow, firewall rules, source IP ranges, network tags vs labels vs service accounts, and routes.

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
When a company needs to launch quickly, scale on demand, reduce hardware operations, or improve availability and recovery.

### Example
```text
Need 100 servers -> request 100 VMs in minutes instead of buying hardware
```

### Important points
- Reduces capital expenditure because teams no longer need to purchase servers upfront.
- Improves agility because infrastructure can be provisioned in minutes instead of weeks.
- Improves scalability because capacity can be increased or reduced as demand changes.
- Adds managed services such as databases, messaging, monitoring, and Kubernetes.
- Helps with high availability and disaster recovery because workloads can be distributed across zones and regions.
- Shifts operational effort from hardware maintenance to application and platform management.

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
- Google’s private global network is a major differentiator and is often a reason for low-latency global traffic routing.
- GKE is one of Google Cloud’s flagship services because Google created Kubernetes.
- BigQuery and Vertex AI are strong for data and AI workloads.
- GCP’s global VPC model is often considered simpler for multi-region networking than some alternatives.
- Cloud choice is workload-driven, not absolute; the best cloud depends on business, compliance, ecosystem, and architecture requirements.
- Enterprises often select GCP when cloud-native compute, Kubernetes, analytics, and global networking are strategic priorities.

### Interview nuggets
- Cloud is about agility, scale, managed services, availability, and cost control.
- GCP is especially strong in global networking, Kubernetes, and data/AI.
- Never say one cloud is always better; say the right cloud depends on the workload.

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
- GCP is built on a global infrastructure model rather than a single central cloud region.
- Google’s private backbone improves performance, reliability, and routing predictability.
- Regions and zones are the core placement units for most resources.
- A global backbone allows many services to communicate across regions without relying purely on the public internet.
- Infrastructure design should consider user geography, compliance, and service placement together.

## 4. Region
A region is a geographic area that contains multiple zones and represents a broader failure and placement domain.

### Use case
Choose a region near users or where compliance and service availability requirements are met.

### Example
- `asia-south1` (Mumbai)
- `us-central1` (Iowa)
- `europe-west2` (London)

### Important points
- Region choice impacts latency, compliance, cost, service availability, and disaster recovery planning.
- A region contains multiple zones.
- Regional selection is both a business decision and an architecture decision.
- Choosing the nearest region is often good for latency, but not always sufficient because other factors matter too.
- Not all services are available in every region.
- For regulated workloads, region choice may be constrained by residency or compliance requirements.

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
- Multi-zone designs improve availability because a zone failure does not necessarily take down the whole application.
- Zones are used for placing VMs, disks, GPUs, and other zonal resources.
- For production systems, zone-aware design is a minimum expectation.

## 6. Resource Scope
GCP resources are designed with appropriate scope: global, regional, or zonal.

### Use case
Understand where resources naturally belong and how to design HA and DR.

### Example
- Global: VPC, routes,  rules, IAM
- Regional: subnet, Cloud NAT, Cloud Router, Cloud SQL
- Zonal: Compute Engine VM, GPU, TPU

### Important points
- Global resources are shared across the project’s geography.
- Regional resources belong to one region.
- Zonal resources belong to one zone.
- Scope is not arbitrary; it matches how the service is naturally designed and where the underlying infrastructure resides.
- Thinking about scope helps explain why some services can be shared globally while others are tied to a region or zone.
- Resource scope directly affects high availability, failure domains, and troubleshooting.

### Interview nuggets
- Region = geographic placement.
- Zone = failure domain within a region.
- Scope is one of the fastest ways to reason about a GCP service.
- If a resource feels “network-wide,” it is often global; if it is tied to IP allocation or routing for one area, it is often regional; if it is physical compute, it is usually zonal.

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
- Policies and IAM can be inherited down the hierarchy.
- Hierarchy is the basis for enterprise governance.

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
- Organization-level policies are powerful because they can define global guardrails for the entire company.

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
- Folders help platform teams apply policies to groups of projects without repeating the same configuration in every project.

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
- A production and development environment should usually not share the same project.
- Many enterprise designs use one project per workload/environment combination.

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
- Project ID is commonly used in Terraform, APIs, and `gcloud` commands.
- Project number is often used in backend/internal service identity patterns.

### Why both Project ID and Project Number?
- **Project ID** is the customer-facing identifier used by Terraform, APIs, CLI, and most user interaction.
- **Project Number** is the immutable Google-generated numeric identifier used internally by many Google services and some APIs.
- Having both separates human-friendly naming from backend system identity.
- Some Google-managed identities and service integration patterns use the project number because it is never renamed.

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
- Billing is usually managed by platform or finance teams, not individual developers.
- Billing and cost reporting should be designed together with labels and budgets.

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
- Quota increases are typically requested when an organization grows or its workload profile changes.
- Quotas protect platform capacity and prevent accidental resource explosions.

### How enterprises handle quota limits
- Clean up unused resources.
- Request quota increases with business justification.
- Plan capacity in advance for large rollouts.
- Monitor usage and alert before reaching hard limits.
- In some cases, switch regions if the needed capacity is not available.

### Interview nuggets
- Quota is not the same as billing plan.
- Larger companies usually request quota increases as part of onboarding or expansion.
- Some failures are due to quota limits, and others are due to actual regional capacity shortage.

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
- Troubleshooting connectivity usually requires checking multiple layers, not just firewall rules.

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
- Packet flow is the mental model for most networking troubleshooting.
- If a VM cannot reach a destination, the problem may be routing, firewall, DNS, NAT, load balancing, or the application itself.

### Interview nuggets
- Subnet owns IPs.
- Routes choose the path.
- Firewall decides the permission.
- Packet flow is the reason we do not jump straight to firewall every time.

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
- A VPC provides the base networking plane for compute and platform services.
- In enterprise design, a VPC is often the top-level network construct for a project or workload family.

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
- Global VPC design reduces the need for complex inter-VPC routing just to connect workloads across regions.

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
- Auto Mode is convenient for fast prototyping and learning, but it can create infrastructure that the enterprise does not actually need.
- Custom Mode is preferred when networking is centrally governed.

### Interview nuggets
- Auto Mode = Google creates a subnet in each supported region.
- Custom Mode = you create subnets yourself.
- Enterprises prefer Custom Mode for production control.
- Auto Mode is convenient, but Custom Mode is deliberate.

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
- Subnets create logical grouping for IP planning and placement.
- A VM’s private IP usually comes from the subnet it is attached to.

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
- Regional subnets allow a VPC to span regions while keeping IP allocation region-aware.
- IP planning should consider growth, future subnets, and hybrid connectivity.

### Interview nuggets
- VPC is global.
- Subnet is regional.
- Subnet owns the IP range.
- A VM receives its private IP from a subnet.

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

### Default firewall rules for default VPC
When configured by the default vpc, 4 specific ingress firewall rules are pre populated.
1. `default-allow-internal`: Allows incoming TCP (0-65535), UDP (0-65535), and ICMP traffic from any instance inside the same VPC network (10.128.0.0/9 range).
2. `default-allow-ssh`: Allows incoming TCP traffic on port 22 (SSH) from any source IP (0.0.0.0/0).
3. `default-allow-rdp`: Allows incoming TCP traffic on port 3389 (RDP) from any source IP (0.0.0.0/0).
4. `default-allow-icmp`: Allows incoming ICMP traffic (ping) from any source IP (0.0.0.0/0)

### Important points
- Firewall belongs to the VPC, not the VM.
- Firewalls are stateful.
- Lower priority number means higher priority.
- Firewall rules are evaluated against packets as they enter or leave instances.
- Centralized firewall design is easier to govern at scale than per-VM firewall management.

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
- For ingress traffic, source ranges define the allowed origin of the packet.
- A source range of `0.0.0.0/0` means the rule matches traffic from anywhere, so it must be used carefully.

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
- Tags are simple network markers attached to a VM.
- They are useful but can be changed more easily than identity-based controls.

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
- Labels help Finance and Platform teams understand ownership and cost.
- Labels are one of the most important enterprise metadata practices in GCP.

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
- Service accounts are central to IAM, workload identity, and secure firewall targeting.
- Production workloads often prefer service-account-based targeting over mutable tags.

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
- A VM can have tags and labels at the same time, but they serve different purposes.
- Enterprise design should avoid using labels for network policy.

### Interview nuggets
- Tags = networking identity.
- Labels = business metadata.
- Service accounts = workload identity.

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
- A route exists to identify the next hop for a destination network.
- Routes are a separate decision from firewall rules.

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
- When you create a subnet, the network automatically knows how to reach it.
- System routes make internal subnet-to-subnet communication possible without manual intervention.

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
- Custom routes are required when you want traffic to leave the default VPC path and follow enterprise-specific network paths.

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
- The default route gives a path toward the internet, but security and source IP behavior still matter.

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
- Static routes are ideal when the path is fixed and well known.
- Dynamic routes are better when on-prem or hybrid prefixes change over time.

### Interview nuggets
- Routes answer “where.”
- Firewall answers “allowed.”
- Longest prefix match selects the most specific route.
- Default route is not the same as internet access.

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
- Cloud Router is important for enterprise hybrid networks because it learns and advertises routes automatically.

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
- Cloud NAT is especially useful when you want private instances to patch packages, pull images, or reach external services without exposing public IPs.

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
- Load balancers are a front door for many internet-facing and internal applications.
- Healthy backend selection is key to load balancer behavior.

## 35. Internet Access Patterns
GCP workloads may use external IPs, Cloud NAT, load balancers, or private connectivity depending on the traffic direction and security design.

### Use case
Design outbound or inbound internet access for workloads.

### Example
- Public VM with external IP
- Private VM with Cloud NAT
- Public service behind load balancer

### Important points
- A route does not equal internet access.
- Internet access depends on the chosen design.
- Private instances often use Cloud NAT rather than public IPs.
- Ingress traffic is commonly handled by load balancers, not by direct exposure of every backend.

---

# Chapter 11: High-Level Enterprise Design

## 36. GCP Enterprise Design Thinking
Enterprise GCP design is about controlling scope, ownership, security, traffic flow, and operational clarity.

### Use case
Design production-ready multi-team GCP platforms.

### Example
```text
Organization -> folders -> projects -> VPC -> subnets -> firewall -> routes -> workloads
```

### Important points
- Separate environments into different projects when possible.
- Use custom VPCs in production.
- Prefer identity-based controls and clear metadata.
- Make networking and billing easier to operate at scale.
- Design for future growth instead of only the current workload.

## 37. Terraform Integration with GCP
Terraform is commonly used to provision GCP infrastructure such as projects, networks, subnets, firewall rules, routes, load balancers, and compute resources.

### Use case
Automate repeatable infrastructure creation and drift control.

### Example
- `google_compute_network`
- `google_compute_subnetwork`
- `google_compute_firewall`
- `google_compute_route`

### Important points
- Terraform is a strong fit for GCP because GCP resources are highly modelled and declarative.
- Provider aliases are important for multi-project designs.
- Imports and moved blocks matter during refactoring or adoption.

---

# Chapter 12: Interview Preparation Notes

## 38. Fast Revision of the Foundation
If you need to remember the most important foundation points quickly:

### Important points
- Cloud solves hardware, scaling, and operations problems.
- GCP is strong in networking, Kubernetes, data, and AI.
- Global infrastructure includes regions and zones.
- Organization -> Folder -> Project -> Resource is the enterprise hierarchy.
- Projects are the heart of GCP.
- Billing accounts pay for projects.
- Quotas control resource consumption.
- VPC is global, subnets are regional, and VMs are zonal.
- Packet flow is route first, then firewall, then destination.
- Firewalls belong to the VPC and are stateful.
- Source ranges define who can reach a firewall rule.
- Network tags are not labels.
- Service accounts are stronger identity-based targets.
- Routes answer where traffic goes.
- Cloud Router exchanges routes; Cloud NAT enables outbound internet for private instances.

## 39. Interview Nuggets
- Closer region is usually better for latency, but compliance and service availability can override that.
- Project ID is for APIs and automation; project number is Google’s internal identifier.
- Quotas are not just billing tier; they depend on capacity and request approval too.
- Auto Mode is simple; Custom Mode is enterprise-friendly.
- Labels help billing and reporting; tags help firewall targeting; service accounts help identity.
- Routes do not allow traffic; firewalls do.
- One VPC can contain many regional subnets.
- A subnet owns IP addresses for a region.
- VPC is global because Google wants one logical private network across regions.
- Firewall rules are stateful, so response traffic is allowed automatically for established connections.

---

# Chapter 13: Summary Tables

## 40. GCP Scope Summary
| Concept | Scope |
|---|---|
| VPC | Global |
| Routes | Global |
| Firewall Rules | Global (VPC-level) |
| Subnet | Regional |
| Cloud NAT | Regional |
| Cloud Router | Regional |
| Compute VM | Zonal |
| Persistent Disk | Usually Zonal unless using regional design |

## 41. Metadata / Identity / Networking Summary
| Feature | Labels | Network Tags | Service Accounts |
|---|---|---|---|
| Purpose | Metadata | Networking target | Workload identity |
| Firewall use | No | Yes | Yes |
| Billing use | Yes | No | No |
| IAM identity | No | No | Yes |
| Format | key=value | simple string | email identity |

## 42. Routing vs Firewall Summary
| Concept | Question Answered |
|---|---|
| Route | Where should traffic go? |
| Firewall | Is traffic allowed? |
| Subnet | Which IP range should be used? |
| VPC | Which private network does it belong to? |

---

# Chapter 14: Closing Notes for This Version
This version of the GCP full notes covers the foundation topics we completed so far. As we continue the GCP journey, additional chapters will be added for IAM, service accounts, impersonation, Shared VPC, VPC peering, Private Service Connect, Cloud DNS, Cloud Armor, GKE, Cloud SQL, Cloud Storage, Cloud Run, Cloud Build, logging, monitoring, security, and enterprise scenarios.

### Final interview reminder
- Think in terms of architecture, not only definitions.
- Always connect a service to its scope, use case, and traffic flow.
- In interviews, explain both the “what” and the “why.”
