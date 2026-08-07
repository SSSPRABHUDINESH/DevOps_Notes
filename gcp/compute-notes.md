# Compute Engine Master Notes

## 1. Machine Families

### Definition
Compute Engine machine families group VM shapes for different workload profiles, balancing cost, CPU, memory, and acceleration.

### Use Cases
Web apps, CI/CD, production APIs, memory-heavy databases, HPC, AI/ML.

### Examples
E2 for cost-optimized general workloads, N2 for balanced production, C2/C3 for CPU-intensive workloads, M-series for memory-heavy workloads, GPU VMs for AI/ML.

### Architecture
```text
Application workload
        │
        ├── Cost-sensitive → E2
        ├── Balanced production → N2
        ├── CPU bound → C2/C3
        ├── Memory bound → M-series
        └── AI/ML or graphics → GPU VM
```

### Important Points
Choose based on bottleneck, not habit. Monitor CPU, memory, disk, and network before resizing.

## 2. E2 Machine Family

### Definition
E2 is the cost-optimized general-purpose family for everyday workloads where peak consistency is not the primary goal.

### Use Cases
Dev/test, low-traffic web apps, internal tools, Jenkins, GitLab runners, bastion hosts.

### Examples
Small startup website, dev environment, CI runner fleet.

### Architecture
```text
Users
  │
  ▼
Load Balancer
  │
  ▼
E2 VM(s)
```

### Important Points
E2 is not "slow"; it is optimized for price/performance. Good default for many non-latency-sensitive workloads.

## 3. N2 Machine Family

### Definition
N2 is a balanced general-purpose family designed for consistent production performance.

### Use Cases
Production web apps, Java services, Kubernetes nodes, enterprise middleware, moderate databases.

### Examples
Payroll API, Spring Boot service, GKE node pool.

### Architecture
```text
Internet
  │
  ▼
External Load Balancer
  │
  ▼
N2 VM(s)
  │
  ├── Cloud SQL
  └── Persistent Disk
```

### Important Points
N2 is often the safest production choice when you need consistency and flexibility.

## 4. C2 Machine Family

### Definition
C2 is compute-optimized for CPU-intensive workloads requiring high per-core performance.

### Use Cases
Build servers, video encoding, scientific computing, gaming servers, CPU-heavy batch jobs.

### Examples
Maven builds, rendering jobs, high-frequency compute tasks.

### Architecture
```text
CPU-heavy job
  │
  ▼
Load Balancer or Batch Scheduler
  │
  ▼
C2 VM(s)
```

### Important Points
Use when CPU is the bottleneck; do not add RAM when CPU is saturated.

## 5. C3 Machine Family

### Definition
C3 is the newer compute-optimized family with improved CPU platform, networking, and storage performance.

### Use Cases
Modern build farms, rendering, HPC, high-throughput CPU workloads.

### Examples
Large-scale compilation, video processing pipeline, modern analytics workers.

### Architecture
```text
CPU-intensive workload
  │
  ▼
C3 VM(s)
  │
  ├── High network throughput
  └── High storage throughput
```

### Important Points
Choose C3 when the workload benefits from newer platform performance, not simply because it is newer.

## 6. Memory Optimized Machines

### Definition
Memory optimized families provide very large RAM for workloads where memory is the main bottleneck.

### Use Cases
SAP HANA, Redis, in-memory analytics, huge JVM heaps, large databases.

### Examples
SAP HANA cluster, Redis cache, in-memory reporting server.

### Architecture
```text
Memory-heavy application
  │
  ▼
Memory Optimized VM(s)
  │
  └── Large RAM footprint
```

### Important Points
If memory is at 90%+ and CPU is low, a memory optimized family is often the right direction.

## 7. GPU Machines

### Definition
GPU VMs pair CPU with accelerator hardware for massively parallel workloads.

### Use Cases
AI training, AI inference, graphics rendering, video processing, scientific computation.

### Examples
PyTorch training, TensorFlow inference, frame rendering.

### Architecture
```text
Dataset / model
  │
  ▼
GPU VM
  ├── CPU orchestrates
  └── GPU accelerates parallel compute
```

### Important Points
GPU does not replace CPU; it complements it. Use only when the workload can exploit parallelism.

## 8. Spot VMs and Preemptible VMs

### Definition
Spot VMs are discounted compute using spare capacity that can be reclaimed by Google at any time. Preemptible VMs are the legacy model, now replaced by Spot VMs.

### Use Cases
Batch jobs, CI runners, rendering, fault-tolerant ETL, checkpointed ML training.

### Examples
Nightly builds, queue-driven workers, distributed rendering jobs.

### Architecture
```text
Workload
  │
  ▼
Spot MIG or Spot Node Pool
  │
  ├── Runs while spare capacity exists
  └── Can be reclaimed with short notice
```

### Important Points
Use only when the workload can tolerate interruption and restart.

## 9. Shielded VMs

### Definition
Shielded VMs protect the VM boot chain using Secure Boot, vTPM, and Integrity Monitoring.

### Use Cases
Production VMs, regulated workloads, enterprise Linux hosts.

### Examples
Banking API host, database VM, security-sensitive Linux server.

### Architecture
```text
Power on
  │
  ▼
Secure Boot
  │
  ▼
vTPM measurements
  │
  ▼
Integrity Monitoring
  │
  ▼
VM boots only if trusted
```

### Important Points
Shielded VM protects boot integrity, not runtime memory.

## 10. Confidential VMs

### Pain point
1. Data inside cloud storage - Encrypted
2. Data during transit - Encrypted
3. Data inside RAM (While processing data) - No Encryption

### Definition
Confidential VMs protect data in use by encrypting memory with hardware confidential computing support.

### Use Cases
Banking, healthcare, government, sensitive analytics, regulated workloads.

### Examples
Payment processing host, patient record processing server.

### Architecture
```text
Application
  │
  ▼
Encrypted RAM / confidential compute
  │
  ▼
Hardware-backed protection
```

### Important Points
Confidential VM protects runtime memory, while Shielded VM protects boot integrity.

## 11. Sole-Tenant Nodes

### Definition
Sole-Tenant Nodes are dedicated physical servers reserved for one customer.

### Use Cases
Compliance, licensing, physical isolation.

### Examples
Oracle workloads, regulated bank environments, dedicated hardware requirements.

### Architecture
```text
Physical server reserved for one customer
  ├── VM1
  ├── VM2
  └── VM3
```

### Important Points
Sole-Tenant is about dedicated hardware, not automatic performance gains.

## 12. Live Migration

### Definition
Live Migration moves a running VM between hosts during planned maintenance with minimal interruption.

### Use Cases
Most standard production VMs.

### Examples
Web server host maintenance without downtime.

### Architecture
```text
Host A
  │
  ├── Running VM
  │
Google maintenance
  │
  ▼
Host B
  └── Same VM continues
```

### Important Points
Works for planned maintenance; not all hardware-dependent workloads support it.

## 13. Maintenance Policies

### Definition
Maintenance policy defines whether a VM should migrate or terminate when host maintenance occurs.

### Use Cases
Standard VMs, GPU jobs, hardware-dependent workloads.

### Examples
MIGRATE for web servers, TERMINATE for GPU training if migration is unsupported.

### Architecture
```text
Google maintenance
  │
  ├── Migrate supported VM
  └── Terminate unsupported VM
```

### Important Points
This applies to planned maintenance, not random hardware failure.

## 14. Instance Templates

### Definition
Instance templates are immutable blueprints used to create consistent VMs.

### Use Cases
Fleet deployments, MIGs, repeatable VM provisioning.

### Examples
Golden VM recipe for Nginx servers or CI runners.

### Architecture
```text
Instance Template
  │
  ▼
Managed Instance Group
  │
  ├── VM1
  ├── VM2
  └── VM3
```

### Important Points
Templates are immutable; create a new one for changes.

## 15. Managed Instance Groups (MIG)

### Definition
MIGs manage fleets of identical VMs for scaling, auto-healing, and rolling updates.

### Use Cases
Web tiers, runners, stateless services, regional HA.

### Examples
Regional web server fleet, Spot runner pool.

### Architecture
```text
Instance Template
  │
  ▼
MIG
  ├── Health checks
  ├── Auto-healing
  ├── Autoscaling
  └── Rolling updates
```

### Important Points
MIG manages runtime VM lifecycle; Terraform defines desired infrastructure.

## 16. Boot Disk

### Definition
The boot disk contains the OS and required boot files for a VM.

### Use Cases
Every VM needs one boot disk.

### Examples
Ubuntu boot disk, Windows boot disk.

### Architecture
```text
Image
  │
  ▼
Boot disk
  │
  ▼
VM boots from this disk
```

### Important Points
The boot disk is a role; it is often backed by Persistent Disk or Hyperdisk.

## 17. Persistent Disk

### Definition
Persistent Disk is durable, network-attached block storage for VMs.

### Use Cases
OS disks, application data, databases, logs, shared read-only data.

### Examples
PostgreSQL data disk, application data disk.

### Architecture
```text
VM
  │
  ▼
Google network
  │
  ▼
Persistent Disk service
```

### Important Points
Persistent Disk survives VM deletion if configured to do so and supports snapshots and resize.

## 18. Images

### Definition
Images are read-only templates used to create boot disks.

### Use Cases
Creating new VMs quickly with a standard OS.

### Examples
Ubuntu public image, RHEL image, custom organization image family.

### Architecture
```text
Image
  │
  ▼
Create boot disk
  │
  ▼
VM boots
```

### Important Points
Image is a template, not a running disk.

## 19. Snapshots

### Definition
Snapshots are point-in-time copies of Persistent Disks.

### Use Cases
Backups, disaster recovery, disk restore, cloning storage state.

### Examples
Nightly database snapshot, pre-upgrade snapshot.

### Architecture
```text
Persistent Disk
  │
  ▼
Snapshot
  │
  ▼
New Persistent Disk
  │
  ▼
Attach to VM
```

### Important Points
Snapshots are incremental after the first snapshot. Restoring a snapshot reconstructs the full disk automatically.

## 20. Custom Images

### Definition
Custom images are reusable images created from configured VMs or disks.

### Sources of Custom image:
1. **Persistent Disk (PD)**: An existing block storage volume (usually your VM's boot disk), even if it is currently attached to a running virtual machine.
2. **Snapshot**: A standard or archive snapshot that you previously captured from a persistent disk.
3. **Another Image**: An existing custom image within your current project, or an image shared with you from another Google Cloud project.
4. **Cloud Storage File**: A compressed virtual disk file (like a `.tar.gz` archive containing a raw disk image file) uploaded to a Google Cloud Storage bucket. This is commonly used when migrating on-premises virtual machines to the cloud.

### Use Cases
Golden images, standardized enterprise VM builds, rapid server provisioning.

### Examples
Company-hardened Ubuntu image with Docker, Git, Java, monitoring agents.

### Architecture
```text
Configured VM
  │
  ▼
Custom Image
  │
  ├── VM1
  ├── VM2
  └── VM3
```

### Important Points
Use custom images for provisioning future servers, not for backing up application data.

## 21. Startup Scripts

### Definition
Startup scripts run automatically when a VM boots.

### Use Cases
Install packages, configure services, pull code, register monitoring, bootstrap applications.

### Examples
Install Nginx, enable Docker, download app binaries.

### Architecture
```text
VM boot
  │
  ▼
Guest agent reads metadata
  │
  ▼
Startup script runs as root
```

### Important Points
Great for dynamic boot-time tasks; pair with custom images for static software baselines.

## 22. Metadata

### Definition
Metadata is key-value configuration provided to VMs via the metadata server.

### Use Cases
Startup scripts, environment flags, service account tokens, instance-specific configuration.

### Examples
`enable-oslogin=TRUE`, startup script, environment name.

### Architecture
```text
VM
  │
  ▼
Metadata server
  │
  ├── Project metadata
  └── Instance metadata
```

### Important Points
VMs pull metadata when needed; the metadata server is not a push channel.

## 23. Hyperdisk

### Definition
Hyperdisk is next-generation block storage with independent scaling of capacity, IOPS, and throughput.

### Use Cases
High-performance databases, enterprise storage, workload-specific tuning.

### Examples
Hyperdisk Extreme for high IOPS DBs, Hyperdisk Balanced for general workloads.

### Architecture
```text
VM
  │
  ▼
Hyperdisk volume
  ├── Capacity
  ├── IOPS
  └── Throughput
```

### Important Points
Some Hyperdisk types support multi-writer, but the application must be cluster-aware.

## 24. Local SSD

### Definition
Local SSD is very fast ephemeral storage physically attached to the host.

### Use Cases
Caches, scratch space, temporary processing, high-speed ephemeral workloads.

### Examples
Redis cache, temp ETL workspace, ML scratch space.

### Architecture
```text
Physical host
  ├── CPU
  ├── RAM
  └── Local SSD
```

### Important Points
Local SSD is not RAM. It is physical SSD storage attached to the host and data is lost when the host or VM is gone.

## 25. Reservations

### Definition
Reservations guarantee that specific Compute Engine capacity is available for your project when needed.

### Use Cases
Peak events, migrations, DR, scarce resources like GPUs.

### Examples
Black Friday capacity reservation, reserved GPU training nodes.

### Architecture
```text
Reserve capacity
  │
  ▼
Google holds CPU/RAM/GPU capacity
  │
  ▼
Create VM later
```

### Important Points
Reservations are charged while held, even if unused. They guarantee capacity, not discounts.

## 26. Placement Policies

### Definition
Placement policies influence physical placement of VMs for performance or availability.

### Use Cases
HPC clusters, low-latency AI training, fault-isolated production fleets.

### Examples
Compact placement for MPI jobs, spread placement for critical services.

### Architecture
```text
Placement policy
  ├── Compact
  └── Spread
```

### Important Points
Compact reduces latency by keeping VMs close; spread improves failure isolation.

## 27. OS Login

### Definition
OS Login uses IAM identities to manage SSH access to Linux VMs.

### Use Cases
Enterprise SSH management, large fleets, easier access revocation.

### Examples
Grant a user `roles/compute.osLogin` to allow login, `roles/compute.osAdminLogin` for sudo access.

### Architecture
```text
User Google identity
  │
  ▼
IAM role
  │
  ▼
OS Login
  │
  ▼
Linux VM access
```

### Important Points
Must be enabled via metadata (`enable-oslogin=TRUE`) at project or instance level. IAM role alone is not enough.

## 28. Serial Console

### Definition
Serial Console provides low-level console access to a VM for recovery and boot-time troubleshooting.

### Use Cases
Boot failures, SSH broken, kernel panics, corrupted configuration.

### Examples
Fix `/etc/fstab`, inspect GRUB errors, recover disabled networking.

### Architecture
```text
Boot issue
  │
  ├── SSH unavailable
  └── Serial Console used
```

### Important Points
Works even when SSH and networking fail; should be tightly controlled via IAM.

## 29. Troubleshooting

### Definition
Compute Engine troubleshooting means isolating the failing layer: application, OS, network, storage, or underlying infrastructure.

### Use Cases
VM won’t boot, SSH fails, app down, disk full, health checks failing, high CPU.

### Examples
Use Serial Console for boot failure, check firewall/IAM for SSH issues, verify health checks for LB issues.

### Architecture
```text
User complaint
  │
  ├── Application layer
  ├── OS layer
  ├── Compute layer
  ├── Network layer
  └── Storage layer
```

### Important Points
Troubleshoot layer by layer and avoid random restarts. Find the root cause first.

## Quick Revision Map

```text
Machine family → workload profile
Spot VM → cheap but reclaimable
Shielded VM → boot security
Confidential VM → memory protection
Sole-Tenant → dedicated hardware
Live Migration → planned maintenance continuity
Maintenance Policy → migrate or terminate
Instance Template → immutable blueprint
MIG → fleet manager
Image → boot disk source
Boot Disk → OS disk
Persistent Disk → durable storage
Snapshot → point-in-time disk backup
Custom Image → reusable configured OS template
Startup Script → boot-time automation
Metadata → VM config and identity channel
Hyperdisk → independent performance tuning
Local SSD → fastest ephemeral storage
Reservation → capacity guarantee
Placement Policy → physical placement preference
OS Login → IAM-based SSH
Serial Console → low-level recovery access
Troubleshooting → isolate the failing layer
```
