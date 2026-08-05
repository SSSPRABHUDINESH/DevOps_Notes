# GCP Networking Architectures

> Text-based networking architecture diagrams and flow summaries.
> This file focuses on structural logic, request paths, and component interactions.

---

## 1. Basic GCP Networking Flow

```text
Application
  │
  ▼
NIC
  │
  ▼
Subnet
  │
  ▼
VPC
  │
  ▼
Route Lookup
  │
  ▼
Firewall Evaluation
  │
  ▼
Destination
```

### Key architecture points
- Subnet provides private IP allocation.
- Routes decide where packets go.
- Firewall decides whether packets are allowed.

---

## 2. VPC and Subnet Layout

```text
Global VPC
├── Mumbai Region
│   ├── Web Subnet
│   ├── App Subnet
│   └── DB Subnet
├── Singapore Region
│   ├── Web Subnet
│   ├── App Subnet
│   └── DB Subnet
└── Tokyo Region
    ├── Web Subnet
    ├── App Subnet
    └── DB Subnet
```

### Key architecture points
- One VPC can span multiple regions.
- Subnets are regional.
- Tiered subnet design helps with segmentation.

---

## 3. Firewall Targeting Architecture

```text
Firewall Rule
  │
  ├── source_ranges
  ├── target_tags
  └── target_service_accounts
       │
       ▼
      VM
```

### Example targeting patterns

```text
Source subnet -> Database subnet on TCP 5432
Web tag -> HTTP firewall rule
DB service account -> PostgreSQL firewall rule
```

### Key architecture points
- Source ranges define who can connect.
- Target tags and service accounts define what receives the rule.
- Labels are not used in firewall targeting.

---

## 4. Route Decision Path

```text
Packet Destination
  │
  ▼
Most Specific Match
  │
  ├── Local Route
  ├── Custom Route
  ├── VPN Route
  └── Default Route 0.0.0.0/0
```

### Key architecture points
- Longest prefix match wins.
- Default route is used only when no more specific route matches.
- Routes do not allow or deny traffic.

---

## 5. Cloud Router Hybrid Connectivity

```text
On-Prem Router
  │
  ▼
BGP Session
  │
  ▼
Cloud VPN or Cloud Interconnect
  │
  ▼
Cloud Router
  │
  ▼
VPC Route Table
  │
  ▼
GCP Resources
```

### Key architecture points
- Cloud Router exchanges routes.
- It does not forward packets.
- BGP learns and advertises prefixes.

---

## 6. Cloud NAT Outbound Flow

```text
Private VM
  │
  ▼
Private Subnet
  │
  ▼
Cloud NAT
  │
  ▼
Public Internet
```

### Response flow

```text
Public Internet
  │
  ▼
Cloud NAT
  │
  ▼
Private VM
```

### Key architecture points
- VM has no public IP.
- NAT translates source address for outbound traffic.
- Cloud NAT is regional.
- Cloud Router is used as the associated control plane resource.

---

## 7. Internet Exposure vs Private Egress

### External HTTP(S) Load Balancer for inbound traffic

```text
Internet User
  │
  ▼
External Load Balancer (Static Public IP)
  │
  ▼
Backend Service
  │
  ▼
Private Web VMs
```

### Cloud NAT for outbound traffic

```text
Private Web VM
  │
  ▼
Cloud NAT
  │
  ▼
Internet
```

### Key architecture points
- Load balancer handles inbound client traffic.
- Cloud NAT handles outbound Internet access from private instances.
- The load balancer public IP does not give backend VMs outbound Internet access.

---

## 8. Cloud VPN Architecture

```text
On-Prem Network
  │
  ▼
IPSec Tunnel
  │
  ▼
Cloud VPN Gateway
  │
  ▼
Cloud Router
  │
  ▼
GCP VPC
```

### Key architecture points
- Cloud VPN creates encrypted connectivity over public Internet.
- IPSec protects traffic in transit.
- Cloud Router is commonly used for dynamic route exchange.

---

## 9. HA VPN Architecture

```text
On-Prem Router
   ║          ║
Tunnel 1    Tunnel 2
   ║          ║
   ║          ║
HA VPN Gateway
   ║          ║
Cloud Router (BGP)
   ║
GCP VPC
```

### Key architecture points
- Multiple tunnels reduce single points of failure.
- BGP detects path loss and supports failover.
- HA VPN is the preferred production design.

---

## 10. Cloud Interconnect Architecture

```text
On-Prem Data Center
  │
  ▼
Dedicated or Partner Connectivity
  │
  ▼
Google Edge / Interconnect
  │
  ▼
Cloud Router
  │
  ▼
GCP VPC
```

### Key architecture points
- Private dedicated connectivity.
- Higher bandwidth and lower latency than VPN.
- Cloud Router handles dynamic route exchange.

---

## 11. VPC Peering Architecture

```text
VPC A
  │
  ▼
VPC Peering
  │
  ▼
VPC B
```

### Key architecture points
- Connects GCP VPC to GCP VPC.
- Does not use Cloud Router.
- Used for private network-level communication between projects or teams.

---

## 12. Private Service Connect Architecture

### Consumer side

```text
Consumer VM
  │
  ▼
PSC Endpoint
  │
  ▼
Private Service Connect
  │
  ▼
Producer Service
```

### Producer side

```text
Producer VPC
  │
  ▼
Internal Load Balancer
  │
  ▼
Service Attachment
  │
  ▼
Consumer Endpoint
```

### Key architecture points
- Connects to a specific service, not an entire VPC.
- Helps avoid broad network exposure.
- Works well for multi-team and multi-project service sharing.
- Can be used for private access to supported Google APIs.

---

## 13. VPC Service Controls Architecture

```text
Trusted Perimeter
+--------------------------------------+
| Project A                            |
| Project B                            |
| Cloud Storage                        |
| BigQuery                             |
| Secret Manager                       |
+--------------------------------------+
```

### Request path

```text
User / Workload
  │
  ▼
IAM Check
  │
  ▼
VPC SC Perimeter Check
  │
  ├── Allowed inside perimeter
  └── Denied if outside perimeter
```

### Key architecture points
- Protects data from leaving trusted boundaries.
- Works at service perimeter level.
- Complements IAM and firewall, not replace them.

---

## 14. Load Balancer Request Lifecycle

```text
DNS
  │
  ▼
Forwarding Rule
  │
  ▼
Target Proxy
  │
  ▼
URL Map
  │
  ▼
Backend Service
  │
  ▼
Health Check
  │
  ▼
Managed Instance Group
  │
  ▼
VM
```

### Key architecture points
- Forwarding rule listens on IP and port.
- Target proxy handles protocol-specific logic.
- URL map chooses backend by host/path.
- Backend service controls traffic policy.
- MIG owns VM lifecycle.

---

## 15. Layer 4 vs Layer 7 Load Balancing

### Layer 4

```text
Client
  │
  ▼
IP / TCP / UDP
  │
  ▼
Backend
```

### Layer 7

```text
Client
  │
  ▼
HTTP / HTTPS
  │
  ▼
Forwarding Rule
  │
  ▼
Target Proxy
  │
  ▼
URL Map
  │
  ▼
Backend Service
  │
  ▼
Backend
```

### Key architecture points
- Layer 4 uses IP and port only.
- Layer 7 uses host, path, headers, and HTTP semantics.
- Layer 7 is used for web and API routing.
- Layer 4 is used for TCP/UDP services such as databases.

---

## 16. Health Check + Backend Service + MIG Flow

```text
Client
  │
  ▼
Load Balancer
  │
  ▼
Backend Service
  │
  ▼
Health Check
  │
  ├── Healthy -> send traffic
  └── Unhealthy -> skip backend
  │
  ▼
Managed Instance Group
  │
  ▼
Replace / scale / heal VM
```

### Key architecture points
- Health check reports health.
- Backend service routes traffic.
- MIG creates and manages identical VMs.
- Autoscaler adjusts instance count.

---

## 17. Global vs Regional Load Balancers

### Global

```text
User -> Global Load Balancer -> Closest healthy region
```

### Regional

```text
User -> Regional Load Balancer -> Region-local backends
```

### Key architecture points
- Global LB is ideal for multi-region and world-facing apps.
- Regional LB is ideal for regional or internal applications.
- Google Front Ends help route traffic early into Google’s private backbone.

---

## 18. Cloud Armor Security Placement

```text
Internet
  │
  ▼
Cloud Armor
  │
  ▼
Load Balancer
  │
  ▼
Backend Service
  │
  ▼
Private VMs
```

### Key architecture points
- Cloud Armor protects web applications at L7.
- Firewall protects network traffic at L3/L4.
- Cloud Armor can filter malicious HTTP requests before they reach backends.

---

## 19. Consolidated Enterprise Edge Architecture

```text
Internet
  │
  ▼
Cloud Armor
  │
  ▼
External Load Balancer
  │
  ▼
Backend Service
  │
  ▼
Health Check
  │
  ▼
Managed Instance Group
  │
  ▼
Private Web VMs
  │
  ▼
Cloud NAT
  │
  ▼
Internet
```

### Key architecture points
- Public entry is protected by Cloud Armor.
- Backends remain private.
- Outbound Internet is handled by Cloud NAT.
- This is a common enterprise web application pattern.

---

## 20. Secure Data Boundary Architecture

```text
External Users
  │
  ▼
Cloud Armor
  │
  ▼
Load Balancer / Web Tier
  │
  ▼
IAM Controlled Apps
  │
  ▼
Google Managed Services
  │
  ▼
VPC Service Controls Perimeter
  │
  ▼
No unauthorized data egress
```

### Key architecture points
- IAM controls identity.
- Cloud Armor protects app traffic.
- VPC SC protects data movement.
- PSC can provide private service access where needed.

---

## 21. Quick Decision Tree

```text
Need to expose an app to the Internet?
  └── External Load Balancer

Need outbound Internet for private VMs?
  └── Cloud NAT

Need on-prem to GCP connectivity?
  ├── Cloud VPN
  └── Cloud Interconnect

Need redundancy for VPN?
  └── HA VPN

Need to connect only one service privately?
  └── Private Service Connect

Need to connect GCP VPCs privately?
  └── VPC Peering

Need to protect Google managed service data boundaries?
  └── VPC Service Controls
```

---

## 22. Console Navigation Reference

### Networking area
- VPC Network
- VPC
- Subnets
- Firewall
- Routes
- Cloud Router
- Cloud NAT
- Cloud VPN
- Load Balancing
- Private Service Connect

### Security area
- IAM
- Cloud Armor
- VPC Service Controls
- Organization Policies

---

## 23. Exam-Style Architecture Prompts

### Prompt 1
Design internet access for private VMs without public IPs.

```text
Private VM -> Cloud NAT -> Internet
```

### Prompt 2
Design secure site-to-site connectivity to on-prem.

```text
On-Prem -> Cloud VPN / Interconnect -> Cloud Router -> GCP VPC
```

### Prompt 3
Design public web access with private backends.

```text
Internet -> Cloud Armor -> Load Balancer -> MIG -> Private VMs
```

### Prompt 4
Design private service consumption between teams.

```text
Consumer VPC -> PSC Endpoint -> Producer Service
```

### Prompt 5
Design data protection for sensitive Google-managed services.

```text
Projects -> VPC SC Perimeter -> Google managed services
```

---

## 24. Summary of Ownership

| Component | Responsibility |
|---|---|
| VPC | Private network boundary |
| Subnet | Regional IP allocation |
| Route | Packet destination selection |
| Firewall | Allow or deny network traffic |
| Cloud Router | Dynamic route exchange |
| Cloud NAT | Outbound IP translation |
| Cloud VPN | Encrypted connectivity over Internet |
| HA VPN | Redundant encrypted connectivity |
| Cloud Interconnect | Private dedicated connectivity |
| Load Balancer | Request distribution |
| Health Check | Backend health detection |
| MIG | VM lifecycle management |
| Cloud Armor | Web application protection |
| PSC | Private service consumption/sharing |
| VPC SC | Data exfiltration boundary |
