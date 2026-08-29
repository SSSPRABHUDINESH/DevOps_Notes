# 🌐 GCP Networking Notes

> Enterprise-focused networking handbook for interviews and production design.
>
> This file covers the networking topics studied so far with a clean structure: definition, use case, examples, architecture, differences, important points, and interview nuggets.

---

## How to use this file
- **Definition** = what the concept means.
- **Use case** = why teams use it.
- **Example** = a practical illustration or architecture.
- **Architecture** = how the service or flow fits together.
- **Difference tables** = fast comparison for interviews.
- **Important points** = the key facts to remember.
- **Interview nuggets** = short revision bullets.

---

# Chapter 1 — Networking Foundations

## 1. Why networking matters in cloud
Every cloud workload depends on networking because compute, databases, load balancing, security, and hybrid connectivity all need a way to communicate.

### Use case
Designing any production system in GCP, from a single VM to a multi-region platform.

### Example
```text
VM -> subnet -> route -> firewall -> destination
```

### Architecture
```text
Application
  -> Network Interface
  -> Subnet
  -> Route Lookup
  -> Firewall Evaluation
  -> Destination
```

### Important points
- Networking is the foundation for almost every GCP service.
- If networking is broken, compute and platform services appear broken too.
- Packet flow is the mental model used for troubleshooting.

## 2. Packet flow
Packet flow describes how traffic moves from a source to a destination through the network stack.

### Use case
Troubleshooting connectivity issues logically instead of guessing.

### Example
```text
Application -> NIC -> VPC -> route lookup -> firewall -> destination
```

### Architecture
```text
Source VM
  -> NIC
  -> Subnet
  -> VPC Route Table
  -> Firewall Rules
  -> Destination VM / Service
```

### Important points
- A subnet does not route traffic.
- Routes answer “where should the packet go?”
- Firewalls answer “is this traffic allowed?”
- Network troubleshooting should check route, firewall, DNS, NAT, and application state.

### Interview nuggets
- Packet flow is one of the best troubleshooting frameworks.
- Do not blame firewall first for every network issue.

---

# Chapter 2 — VPC and Subnets

## 3. VPC
A Virtual Private Cloud is the private logical network inside GCP where resources communicate securely.

### Use case
Private internal communication between VMs, databases, GKE nodes, and other services.

### Example
```text
Global VPC -> subnets -> compute -> internal traffic
```

### Architecture
```text
VPC
  ├── regional subnet A
  ├── regional subnet B
  └── route/firewall policies
```

### Important points
- In GCP, VPC is global.
- A VPC is not a router or firewall device.
- It is the top-level logical network boundary.

## 4. Why VPC is global
Google made VPC global so one logical network can span multiple regions without creating separate networks for every region.

### Use case
Multi-region applications that should remain on one network design.

### Example
```text
One global VPC
  ├── Mumbai subnet
  ├── Singapore subnet
  └── Tokyo subnet
```

### Important points
- One VPC can have subnets in many regions.
- It reduces complexity when applications expand geographically.
- It helps centralize network policy.

## 5. Subnet
A subnet is a regional IP range within a VPC from which private IP addresses are allocated.

### Use case
Give private IPs to resources in a specific region.

### Example
```text
Mumbai subnet -> 10.10.1.0/24
Singapore subnet -> 10.20.1.0/24
```

### Architecture
```text
Global VPC
  ├── Mumbai subnet (regional)
  └── Singapore subnet (regional)
```

### Important points
- Subnets are regional.
- A subnet owns an IP range.
- VMs get their private IPs from subnets.
- No overlapping CIDRs in the same network design.

### Interview nuggets
- VPC = global.
- Subnet = regional.
- VM private IP = from subnet.

## 6. Why subnets are regional
Subnets are regional because IP allocation and placement should be tied to a region, not to the whole world.

### Use case
Maintain clear IP planning and regional separation.

### Example
```text
asia-south1 -> 10.10.1.0/24
asia-southeast1 -> 10.20.1.0/24
```

### Important points
- One subnet belongs to one region.
- Multiple subnets can exist inside one VPC.
- Regional subnets help with scalable address planning.

## 7. Auto Mode vs Custom Mode VPC
Auto Mode creates subnets automatically in supported regions; Custom Mode lets you define everything yourself.

### Use case
Auto Mode for labs and quick demos; Custom Mode for enterprise production.

### Example
- Auto Mode: Google creates subnet ranges for supported regions.
- Custom Mode: You define subnet regions and CIDRs.

### Architecture
```text
Auto Mode
  VPC -> auto-created subnets

Custom Mode
  VPC -> manually created subnets
```

### Important points
- Enterprises usually choose Custom Mode.
- Auto -> Custom conversion is supported.
- Custom -> Auto is not supported.
- Custom Mode gives better IP planning and governance.

### Difference table
| Feature | Auto Mode | Custom Mode |
|---|---|---|
| Who creates subnets | Google | You |
| Control | Low | High |
| Production fit | Low | High |
| IP planning | Automatic | Manual |
| Enterprise governance | Weak | Strong |

### Interview nuggets
- Auto Mode is convenience.
- Custom Mode is control.
- Production usually needs control.

---

# Chapter 3 — Routing and Firewall Flow

## 8. Routes
Routes decide where traffic should go based on the destination IP range.

### Use case
Move traffic between subnets, to on-prem networks, or to the Internet.

### Example
```text
Destination 10.10.3.0/24 -> local subnet path
```

### Architecture
```text
Packet
  -> route table
  -> best destination match
  -> next hop
```

### Important points
- Routes answer “where?”
- Routes do not allow or deny traffic.
- Longest prefix match wins.
- System routes are created automatically for subnets.

## 9. System routes
System routes are automatically created by Google so subnets and default network paths work without manual route creation.

### Use case
Internal subnet-to-subnet and default connectivity.

### Example
```text
Subnet creation -> automatic route for that subnet
```

### Important points
- You do not manually manage system routes.
- They are essential for internal connectivity.
- They make same-VPC communication work naturally.

## 10. Default route
The default route (`0.0.0.0/0`) is used when no more specific destination route matches.

### Use case
Outbound internet routing when the instance has a valid external path.

### Example
```text
0.0.0.0/0 -> internet gateway path
```

### Important points
- A default route does not by itself guarantee internet access.
- External IPs or Cloud NAT may still be needed.
- More specific routes override the default route.

## 11. Static vs Dynamic routes
Static routes are manually configured; dynamic routes are learned through Cloud Router using BGP.

### Use case
Static for fixed paths, dynamic for hybrid networks.

### Example
- Static: manually route `192.168.100.0/24` to a VPN gateway
- Dynamic: learn on-prem prefixes automatically through BGP

### Difference table
| Feature | Static Route | Dynamic Route |
|---|---|---|
| Created by | Manual admin action | BGP / Cloud Router |
| Changes automatically | No | Yes |
| Best for | Small fixed networks | Hybrid / changing networks |

### Important points
- Dynamic routing reduces operational overhead.
- Cloud Router is the key GCP service for dynamic route exchange.

## 12. Firewall rules
Firewall rules control whether traffic is allowed or denied between network resources in a VPC.

### Use case
Protect VMs, databases, internal services, and workloads.

### Example
```text
Allow TCP 22 from office IPs only
```

### Architecture
```text
Packet -> route decision -> firewall decision -> destination
```

### Default firewall rules for default VPC
When configured by the default vpc, 4 specific ingress firewall rules are pre populated.
1. `default-allow-internal`: Allows incoming TCP (0-65535), UDP (0-65535), and ICMP traffic from any instance inside the same VPC network (10.128.0.0/9 range).
2. `default-allow-ssh`: Allows incoming TCP traffic on port 22 (SSH) from any source IP (0.0.0.0/0).
3. `default-allow-rdp`: Allows incoming TCP traffic on port 3389 (RDP) from any source IP (0.0.0.0/0).
4. `default-allow-icmp`: Allows incoming ICMP traffic (ping) from any source IP (0.0.0.0/0)

### Important points
- Firewall is attached to the VPC, not to individual VMs.
- Firewall is stateful.
- Lower priority number = higher priority.
- First matching rule wins.

### Interview nuggets
- Firewall answers “allowed or denied?”
- Routes answer “where?”
- Firewall does not decide routing.

## 13. Source IP ranges
Source IP ranges specify which source CIDRs are allowed for ingress firewall rules.

### Use case
Allow SSH only from office networks or allow app subnet traffic to database subnet.

### Example
```hcl
source_ranges = ["10.10.2.0/24"]
```

### Architecture
```text
Source IP CIDR
  -> firewall rule match
  -> allow / deny
```

### Important points
- `source_ranges` is about who is sending traffic.
- Primarily used in ingress rules.
- CIDR notation can represent one IP, a subnet, or a broad range.

## 14. Network tags vs labels vs service accounts
These three concepts are different: tags are for network targeting, labels are for metadata, and service accounts are for workload identity.

### Use case
Use the right mechanism for networking, reporting, and identity separately.

### Example
- Firewall target: tag or service account
- Billing / cost allocation: labels
- Identity: service account

### Difference table
| Feature | Network Tags | Labels | Service Accounts |
|---|---|---|---|
| Purpose | Networking target | Metadata | Identity |
| Format | String | Key=value | Email identity |
| Used by firewall | Yes | No | Yes |
| Used for billing | No | Yes | No |
| Security identity | No | No | Yes |

### Important points
- Labels do not affect firewall rules.
- Tags are networking only.
- Service accounts are identity-based and preferred for production security.

### Interview nuggets
- Tags = networking targeting.
- Labels = metadata and cost allocation.
- Service account = identity.

---

# Chapter 4 — Cloud Router, NAT, and Hybrid Connectivity

## 15. Cloud Router
Cloud Router is a regional managed service that exchanges routes dynamically between GCP and connected networks using BGP.

### Use case
Hybrid connectivity and automatic route advertisement/learning.

### Example
```text
On-prem router <-> Cloud Router <-> VPC routes
```

### Architecture
```text
On-prem / peer router
  -- BGP --> Cloud Router
  -- routes --> VPC route table
```

### Important points
- Cloud Router is not a packet-forwarding router.
- It does not create the physical link.
- It exchanges routing information.
- It is regional.
- It is commonly used with Cloud VPN and Cloud Interconnect.

### Difference table
| Component | Responsibility |
|---|---|
| Cloud Router | Route exchange using BGP |
| Routes | Packet destination path |
| Firewall | Allow / deny traffic |

### Interview nuggets
- Cloud Router learns and advertises prefixes.
- It exchanges network prefixes, not individual VM details.

## 16. Cloud NAT
Cloud NAT allows private VMs to initiate outbound Internet connections without assigning public IPs to those VMs.

### Use case
Private application servers need package downloads or outbound API access but should not be directly exposed to the Internet.

### Example
```text
Private VM -> Cloud NAT -> Internet
```

### Architecture
```text
Private VM
  -> route
  -> Cloud NAT
  -> public egress IP
  -> Internet
```

### Important points
- Cloud NAT is outbound only.
- It translates the source IP.
- The VM keeps its private IP.
- Cloud NAT is regional.
- Cloud NAT is configured through a Cloud Router association, but Cloud Router does not do NAT.

### Difference table
| Feature | External IP on VM | Cloud NAT |
|---|---|---|
| VM public exposure | Yes | No |
| Outbound internet | Yes | Yes |
| Inbound internet | Possible | No |
| Security posture | Lower | Higher |

### Interview nuggets
- Cloud NAT is for outbound internet from private instances.
- Cloud Router manages routing; Cloud NAT performs translation.

## 17. Cloud VPN
Cloud VPN creates an encrypted IPSec tunnel between on-premises networks (or another cloud) and a GCP VPC over the public Internet.

### Use case
Secure site-to-site connectivity between on-prem and GCP.

### Example
```text
On-prem network <encrypted tunnel> GCP VPC
```

### Architecture
```text
On-prem router
  -> IPSec tunnel
  -> Cloud VPN gateway
  -> Cloud Router (optional/dynamic)
  -> VPC
```

### Important points
- Traffic is encrypted over the public Internet.
- Cloud VPN is for network-to-network connectivity.
- IPSec provides encryption, integrity, and authentication.

### Difference table
| Feature | TLS | IPSec / Cloud VPN |
|---|---|---|
| Protects | Application session | Network tunnel |
| Layer | Higher | Network layer |
| Scope | One app/session | Entire tunnel traffic |

### Interview nuggets
- HTTPS/TLS is not the same as VPN/IPSec.
- VPN encrypts network traffic; TLS encrypts application sessions.

## 18. HA VPN vs Classic VPN
Classic VPN has a simpler design with higher failure risk; HA VPN uses redundant tunnels and gateways for production resiliency.

### Use case
Use HA VPN for production-grade hybrid connectivity.

### Example
```text
Classic VPN -> single tunnel / possible SPOF
HA VPN -> redundant tunnels / failover
```

### Architecture
```text
On-prem router
  -> Tunnel 1 + Tunnel 2
  -> HA VPN gateway
  -> Cloud Router
  -> VPC
```

### Difference table
| Feature | Classic VPN | HA VPN |
|---|---|---|
| Redundancy | Low | High |
| Failover | Manual or limited | Automatic |
| Production fit | Lower | Higher |

### Important points
- HA VPN is designed to eliminate single points of failure.
- Cloud Router + BGP is the normal enterprise pairing.

## 19. Cloud Interconnect
Cloud Interconnect provides private, dedicated connectivity between on-premises networks and Google Cloud without using the public Internet.

### Use case
High-bandwidth, low-latency enterprise connectivity.

### Example
```text
On-prem -> dedicated link -> Google edge -> VPC
```

### Architecture
```text
On-prem router
  -> dedicated connection or partner network
  -> Google edge
  -> Cloud Router
  -> VPC
```

### Difference table
| Feature | Cloud VPN | Cloud Interconnect |
|---|---|---|
| Internet used | Yes | No |
| Security model | Encrypted tunnel | Private link |
| Bandwidth | Lower | Higher |
| Latency | Internet dependent | More predictable |

### Important points
- Interconnect is for high-scale enterprise use.
- Cloud Router is commonly used with it for dynamic routing.

## 20. VPC Peering
VPC Peering connects two GCP VPC networks privately so traffic can flow between them without using public IPs.

### Use case
Private GCP-to-GCP connectivity between separate teams or projects.

### Example
```text
VPC A <-> VPC B
```

### Architecture
```text
VPC A route table <-> peering link <-> VPC B route table
```

### Important points
- VPC Peering does not use public IPs.
- It is private networking, not Internet routing.
- It connects networks, not individual services.
- It is different from PSC.

### Difference table
| Feature | VPC Peering | PSC |
|---|---|---|
| What it connects | Whole VPCs | Specific services |
| Scope | Network-level | Service-level |
| Uses public IPs | No | No |
| Best for | Broad internal connectivity | Controlled service exposure |

### Interview nuggets
- VPC Peering is private and does not traverse public IPs.
- PSC is still needed when you want service-level access rather than entire network access.

## 21. Private Service Connect (PSC)
Private Service Connect allows private access to a specific service without exposing or peering the entire VPC.

### Use case
Expose one internal service privately or consume Google APIs privately.

### Example
```text
Consumer VM -> private endpoint -> PSC -> producer service
```

### Architecture
```text
Consumer VPC
  -> PSC endpoint
  -> Producer service attachment / Google API
```

### Important points
- PSC connects services, not entire networks.
- Consumer and producer are different roles.
- PSC can provide private access to selected Google APIs.
- PSC is ideal when you want a single service, not full network access.

### Difference table
| Feature | VPC Peering | PSC |
|---|---|---|
| Connects | VPC to VPC | Consumer to service |
| Scope | Broad network | Specific service |
| Network merging | Yes | No |
| Best for | Internal network connectivity | Controlled private service access |

### Simple example
- Without PSC: private VM calls a public Google API endpoint.
- With PSC: private VM uses a private endpoint inside the VPC to reach the service privately.

### Interview nuggets
- PSC means private service sharing, not private network sharing.
- PSC is service-level control.
- PSC is excellent for shared services and Google API private access.

## 22. VPC Service Controls (VPC SC)
VPC Service Controls create a security perimeter around supported Google Cloud services to help reduce data exfiltration risk.

### Use case
Keep sensitive data inside trusted projects and prevent it from leaving the organization boundary.

### Example
```text
Perimeter: Project A + Project B + Cloud Storage + BigQuery
```

### Architecture
```text
Trusted perimeter
  -> protected services
  -> ingress/egress policies
```

### Important points
- VPC SC protects data movement, not network packets.
- IAM answers “who are you?”
- VPC SC answers “where can data go?”
- It helps reduce unauthorized copying of sensitive data to untrusted projects or environments.

### Difference table
| Feature | IAM | Firewall | VPC SC |
|---|---|---|---|
| Controls | Who can access | Network traffic | Data boundary |
| Layer | Identity | Network | Service/data perimeter |
| Main problem | Authorization | Packet control | Exfiltration prevention |

### Interview nuggets
- VPC SC is a service perimeter.
- It complements IAM, firewall, Cloud Armor, and PSC.
- It is especially important for data governance and regulated industries.

---

# Chapter 5 — Load Balancing and Traffic Distribution

## 23. Load Balancers
Load balancers distribute incoming client requests across multiple backend resources to improve availability, scalability, and performance.

### Use case
Serve large traffic volumes and avoid a single server becoming a bottleneck.

### Example
```text
Internet -> load balancer -> backend VM1 / VM2 / VM3
```

### Architecture
```text
Client
  -> DNS
  -> Load Balancer IP
  -> backend selection
  -> healthy instance
```

### Important points
- Load balancer gives one stable entry point.
- It helps with scaling and failover.
- Health checks are critical.
- Clients do not need to know backend instance IPs.

## 24. Load balancer internal components
GCP load balancing is modular: forwarding rule, target proxy, URL map, backend service, health check, and managed instance group work together.

### Use case
Explain what happens after a client types a URL.

### Example
```text
DNS -> forwarding rule -> target proxy -> URL map -> backend service -> health check -> MIG -> VM
```

### Architecture
```text
Client
  -> DNS
  -> Forwarding Rule
  -> Target Proxy
  -> URL Map
  -> Backend Service
  -> Health Check
  -> MIG
  -> VM
```

### Important points
- Forwarding rule listens on IP/port.
- Target proxy handles protocol.
- URL map does host/path routing.
- Backend service manages traffic policy.
- Health checks determine healthy backends.
- MIG manages the VM group.

### Responsibilities table
| Component | Responsibility |
|---|---|
| Forwarding Rule | Listen on IP/port |
| Target Proxy | Handle protocol |
| URL Map | Route by host/path |
| Backend Service | Traffic policy manager |
| Health Check | Backend health detection |
| MIG | VM lifecycle management |

## 25. Layer 4 vs Layer 7 load balancing
Layer 4 load balancing works with IP, TCP, and UDP, while Layer 7 understands HTTP/HTTPS details like hostnames and URL paths.

### Use case
L4 for databases and raw TCP/UDP; L7 for web apps and APIs.

### Example
- L4: PostgreSQL on TCP 5432
- L7: `/api`, `/admin`, `/images`

### Difference table
| Feature | Layer 4 | Layer 7 |
|---|---|---|
| Understands | IP, TCP, UDP | HTTP, HTTPS, headers, URL path |
| Routing granularity | Port-based | Host/path-based |
| Common use | DB, SSH, raw TCP | Web apps, APIs |

### Important points
- Layer 4 does not understand URLs.
- Layer 7 is more intelligent for web traffic.
- Choose the layer based on protocol, not preference.

## 26. Health checks, backend services, and MIG
Health checks detect unhealthy instances, backend services decide traffic routing, and MIGs manage identical VMs as one unit.

### Use case
Production web farms and autoscaling platforms.

### Example
```text
Load balancer -> backend service -> health check -> MIG -> VM
```

### Architecture
```text
Client -> LB -> backend service -> health check -> MIG -> healthy VM
```

### Important points
- Health check only reports health.
- Backend service routes traffic and enforces policy.
- MIG creates and manages VMs from a template.
- Autoscaler adds/removes instances based on demand.

### Responsibilities table
| Component | Responsibility |
|---|---|
| Health Check | Detect healthy backends |
| Backend Service | Choose routing policy |
| MIG | Manage VM group |
| Instance Template | VM blueprint |
| Autoscaler | Adjust capacity |

## 27. Global vs Regional load balancers
Global load balancers provide one global entry point for multiple regions; regional load balancers keep traffic inside one region.

### Use case
Global user base vs regional/internal workloads.

### Example
- Global: India, USA, Europe users
- Regional: one-region app in Mumbai

### Architecture
```text
Global LB -> multiple regions
Regional LB -> one region only
```

### Difference table
| Feature | Global | Regional |
|---|---|---|
| Scope | Multi-region | Single region |
| Best for | Global user-facing apps | Regional or internal apps |
| Failover | Built for cross-region routing | Requires regional DR design |

### Important points
- Global load balancers improve latency and resilience.
- Regional load balancers are simpler for single-region systems.
- Global LB uses Google’s global edge and backbone more heavily.

## 28. Cloud Armor
Cloud Armor is Google’s WAF and DDoS protection service for applications behind supported load balancers.

### Use case
Block malicious HTTP requests, DDoS, abuse, and unwanted geography.

### Example
```text
Internet -> Cloud Armor -> load balancer -> app
```

### Architecture
```text
Internet
  -> Cloud Armor policy
  -> load balancer
  -> backend service
```

### Important points
- Cloud Armor is Layer 7 protection.
- It complements VPC firewall rules.
- It can rate-limit, block IPs, block countries, and apply WAF rules.
- It protects applications, not raw VMs.

### Difference table
| Feature | Firewall | Cloud Armor |
|---|---|---|
| Layer | L3/L4 | L7 |
| Protects | Network access | Web application requests |
| Can block SQL injection | No | Yes |
| Can rate limit requests | No | Yes |

### Interview nuggets
- Firewall protects packets.
- Cloud Armor protects web requests.
- Cloud Armor sits in front of the web application.

---

# Chapter 6 — Security Perimeter and Private Access

## 29. PSC for Google APIs
Private Service Connect can provide private endpoints to supported Google APIs so workloads can access them without using public endpoints.

### Use case
Private VMs need to use services like Cloud Storage or Secret Manager without public IP exposure.

### Example
```text
Private VM -> private endpoint -> PSC -> Google API
```

### Architecture
```text
Private VM
  -> private IP endpoint
  -> PSC
  -> Google API service
```

### Important points
- The VM uses a private endpoint inside the VPC.
- The service remains Google-managed, but the access path is private.
- This is about private endpoint access, not Internet routing.

## 30. PSC vs VPC Peering vs VPC SC
PSC connects to a specific service privately, VPC Peering connects whole VPCs privately, and VPC SC creates a service perimeter for data protection.

### Difference table
| Feature | VPC Peering | PSC | VPC SC |
|---|---|---|---|
| Scope | Whole VPC | One service | Service perimeter |
| Main goal | Network connectivity | Private service access | Data exfiltration control |
| Public IP needed | No | No | No |
| Best for | Internal network sharing | Controlled service publishing/consuming | Sensitive data governance |

### Important points
- Peering is network sharing.
- PSC is service sharing.
- VPC SC is data boundary control.

---

# Chapter 7 — Console Locations and Interview Framing

## 31. Where PSC and VPC SC live in the console
PSC is generally part of Networking, while VPC Service Controls is a Security service.

### Use case
Know where to look in the console and how to explain configuration ownership.

### Example
- PSC: VPC Network -> Private Service Connect
- VPC SC: Security -> VPC Service Controls

### Important points
- PSC is a connectivity feature.
- VPC SC is a security policy feature.
- The console placement reflects the problem each service solves.

### Important table
| Service | Console area | Why there |
|---|---|---|
| PSC | Networking | Connectivity / private endpoints |
| VPC SC | Security | Data protection / service perimeter |
| Cloud Armor | Security | Web application protection |
| VPC / Subnets / Routes / NAT / VPN | Networking | Packet movement and connectivity |

## Configuring Private Service Connect (PSC) in the GCP Console depends on whether you are **publishing a service** (Producer) or **connecting to a service / Google API** (Consumer).

---

### Scenario 1: Connect to Google APIs or Published Services (Consumer Side)

Use this workflow to give private workloads access to Google APIs (like Cloud Storage or BigQuery) or a third-party/internal producer service using a private IP.

1. **Go to Private Service Connect:**
   * Open the **GCP Console**.
   * Navigate to **Network Services** -> **Private Service Connect**.
2. **Connect an Endpoint:**
   * Under the **Connected Endpoints** tab, click **Connect Endpoint**.
3. **Choose Target Type:**
   * Select **Google APIs** or **Published service** (depending on what you are connecting to).
4. **Configure Endpoint Details:**
   * **Target / Service Attachment URL:** If connecting to a custom/third-party service, paste the URI string provided by the producer.
   * **Network:** Select your VPC network.
   * **Subnet:** Choose the subnet where the endpoint IP should be created.
   * **IP Address:** Create or assign a static internal IP address in that subnet.
5. **Set up Service Directory & DNS (Optional):**
   * Link the endpoint to **Service Directory** and configure a private DNS zone so your applications can access the service using a friendly domain name instead of the IP address.
6. **Click Add Endpoint.**

---

### Scenario 2: Publish Your Own Service (Producer Side)

Use this workflow to share an internal application with other VPCs or external consumer projects without setting up full VPC Peering.

1. **Prerequisite:** Ensure your service is already sitting behind a supported regional **Internal Load Balancer** (ILB) in your VPC.
2. **Go to Private Service Connect:**
   * Open **Network Services** -> **Private Service Connect**.
3. **Publish Service:**
   * Switch to the **Published Services** tab and click **Publish Service**.
4. **Configure the Service Attachment:**
   * **Load Balancer Type:** Select your internal load balancer type (e.g., Regional internal Application Load Balancer or Passthrough ILB).
   * **Internal Load Balancer:** Pick your target load balancer.
   * **Subnets:** Select a dedicated subnetwork with the purpose set to *Private Service Connect* (used for NAT IP allocation to incoming consumer traffic).
   * **Connection Preference:**
     * *Automatically accept connections:* Allow any consumer with the attachment URI to connect.
     * *Accept connections for selected projects:* Require manual approval or specify allowed GCP Project IDs.
5. **Click Add Service.**
6. **Share the Service Attachment URI:** Copy the generated Service Attachment URI string and share it with your consumers so they can set up their endpoint.

---

# Chapter 8 — Interview Decision Trees

## 32. Connectivity decision tree
Choose the service based on what you are connecting and what you want to protect.

### Use case
Fast architecture decisions during interviews.

### Example
```text
VPC to VPC? -> VPC Peering
On-prem to GCP? -> Cloud VPN or Cloud Interconnect
Private VM to Internet? -> Cloud NAT
Single service exposure? -> PSC
Dynamic route exchange? -> Cloud Router
Public web protection? -> Cloud Armor
```

### Architecture
```text
Need connectivity?
  -> network level -> peering / VPN / interconnect
  -> service level -> PSC
  -> outbound internet -> Cloud NAT
  -> routing exchange -> Cloud Router
  -> web protection -> Cloud Armor
```

### Important points
- Select services by the problem they solve.
- Avoid using the biggest connectivity tool when a smaller one is enough.
- Layered security is normal in enterprise GCP.

### Interview nuggets
- Network sharing = peering.
- Service sharing = PSC.
- Data boundary = VPC SC.
- Web request protection = Cloud Armor.
- Outbound internet for private VMs = Cloud NAT.

---

# Chapter 9 — Enterprise architecture examples

## 33. Secure web application example
A typical secure enterprise web application uses Cloud Armor, global or regional load balancing, private backend VMs, health checks, MIG, and Cloud NAT for outbound updates.

### Use case
Public web apps with private compute.

### Example
```text
Internet -> Cloud Armor -> Load Balancer -> MIG -> private VMs -> Cloud NAT for outbound updates
```

### Architecture
```text
Internet
  -> Cloud Armor
  -> Load Balancer
  -> Backend Service
  -> Health Check
  -> MIG
  -> Private VMs
  -> Cloud NAT
  -> Internet egress
```

### Important points
- Public traffic enters through the load balancer.
- Private VMs remain hidden behind the balancer.
- Cloud NAT handles outbound internet use.

## 34. Hybrid enterprise example
Large organizations often use Cloud VPN or Cloud Interconnect with Cloud Router to connect on-premises data centers to GCP.

### Use case
Private enterprise integrations, mainframes, LDAP, AD, file servers, shared datasets.

### Example
```text
On-prem -> VPN/Interconnect -> Cloud Router -> VPC -> workloads
```

### Architecture
```text
On-prem network
  -> Cloud VPN / Cloud Interconnect
  -> Cloud Router
  -> VPC route table
  -> private subnets
```

### Important points
- Cloud VPN = encrypted over the Internet.
- Cloud Interconnect = private dedicated connectivity.
- Cloud Router = route exchange.

## 35. Service sharing example
PSC is useful when one team wants to share only one service with another team without exposing the whole VPC.

### Use case
Payment API, internal APIs, shared platform services.

### Example
```text
Producer VPC -> PSC service attachment -> Consumer VPC endpoint
```

### Important points
- This is service-level sharing.
- It is safer than broad peering when only one service is needed.

## 36. Data protection example
VPC Service Controls is useful when sensitive data should not move outside trusted projects.

### Use case
Banks, healthcare, regulated workloads, production analytics.

### Example
```text
Project A + Project B inside one perimeter; copy to personal project denied
```

### Important points
- IAM alone is not enough for exfiltration protection.
- VPC SC is for data governance and perimeter control.

---

# Chapter 10 — Quick revision summary

## 37. One-line recall table
This is the fastest interview revision sheet for networking.

### Table
| Topic | Remember |
|---|---|
| VPC | Global private network |
| Subnet | Regional IP range |
| Route | Where traffic goes |
| Firewall | Whether traffic is allowed |
| Network Tags | Networking target strings |
| Labels | Metadata / billing tags |
| Service Accounts | Workload identity |
| Cloud Router | BGP route exchange |
| Cloud NAT | Outbound internet for private VMs |
| Cloud VPN | Encrypted tunnel over Internet |
| HA VPN | Redundant tunnels / failover |
| Cloud Interconnect | Dedicated private link |
| VPC Peering | Private VPC-to-VPC connectivity |
| PSC | Private service access |
| VPC SC | Service perimeter / exfiltration control |
| Cloud Armor | WAF and DDoS protection |
| Load Balancer | Traffic distribution and entry point |
| Health Check | Backend health detection |
| MIG | Manage identical VM group |

### Interview nuggets
- Know the job of each service, not just the name.
- Most senior interviews test architecture, not memorization.
- The right answer usually begins with “what problem are we solving?”
