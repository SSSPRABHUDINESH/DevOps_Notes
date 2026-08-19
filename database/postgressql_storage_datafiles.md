# 🐘 Chapter 3 — PostgreSQL Storage & Data Files

> **Learning Track:** AlloyDB Omni + Kubernetes + CSI + Enterprise Storage  
> **Chapter Status:** ✅ COMPLETE  
> **Depth:** Interview + project focused. Advanced PostgreSQL storage internals are intentionally deferred to the POC.

---

## 📚 Table of Contents

1. Chapter Objective
2. PostgreSQL Storage — Big Picture
3. Database → Tables → Pages
4. Tables and Indexes
5. PostgreSQL Data Files
6. Data Files vs WAL
7. PostgreSQL Write Path
8. PostgreSQL Read Path
9. Dirty Pages
10. What Is a Volume?
11. Filesystem → Volume → Storage
12. Containers and Persistent Storage
13. PersistentVolume and PersistentVolumeClaim
14. StorageClass
15. Where CSI Fits
16. Enterprise Storage
17. Why Enterprise Storage Matters for VLDB
18. Storage Snapshots
19. Why Snapshots Can Help VLDB
20. Storage Clone
21. Restore
22. Storage Replication
23. HA vs DR
24. Storage Performance
25. Benchmarking
26. pgBackRest vs Storage-Level Operations
27. How This Relates to Your Old Project
28. New Project Architecture
29. Old vs New Architecture
30. End-to-End Mental Model
31. Interview Questions
32. Revision Checklist
33. One-Minute Revision

---

# 1. 🎯 Chapter Objective

The purpose of this chapter is to understand:

- Where PostgreSQL data actually lives
- How tables and indexes consume storage
- What PostgreSQL pages are
- What data files represent
- The relationship between data and WAL
- How PostgreSQL reads and writes persistent data
- What a volume is
- Why persistent storage is important for containers
- PV and PVC concepts
- What CSI does
- Why enterprise storage platforms matter for VLDB databases
- Storage snapshots
- Storage clones
- Restore
- Storage replication
- HA vs DR
- Storage performance metrics
- What benchmarking means for this project
- Why this is relevant to the new AlloyDB Omni Kubernetes + CSI proposal

---

# 2. 🏗️ PostgreSQL Storage — Big Picture

At a high level:

```text
                         PostgreSQL
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
              Data Files                  WAL
                 │                         │
                 └────────────┬────────────┘
                              ▼
                         Filesystem
                              │
                              ▼
                         Storage Volume
                              │
                              ▼
                      Persistent Storage
```

### Core idea

> **PostgreSQL is database software running on compute, but its persistent database state ultimately needs durable storage.**

---

# 3. 📦 Database → Tables → Pages

Suppose we create:

```sql
CREATE TABLE customers (
    id INT,
    name TEXT,
    city TEXT
);
```

Conceptually:

```text
Database
   │
   ▼
customers table
   │
   ├── Page 1
   ├── Page 2
   ├── Page 3
   └── ...
```

PostgreSQL stores table and index data using pages.

> **A PostgreSQL page is typically 8 KB.**

You do not need to memorize the detailed internal page layout for this project.

### Key point

> PostgreSQL reads and writes database data in page-oriented storage units.

---

# 4. 🗃️ Tables and Indexes

Database storage is not only table data.

Example:

```sql
CREATE INDEX idx_customer_id
ON customers(id);
```

Now storage contains:

```text
Database
│
├── Table data
│
└── Index data
```

Conceptually:

```text
Database
   │
   ├── Tables
   │     └── Data pages
   │
   └── Indexes
         └── Index pages
```

For a VLDB, indexes can consume significant storage.

Therefore:

> **When discussing database size, think about both table data and indexes.**

---

# 5. 📁 PostgreSQL Data Files

PostgreSQL internally represents database objects through files on the filesystem.

Conceptually:

```text
PostgreSQL Data Directory
│
├── Database data
├── Table files
├── Index files
├── WAL
├── Configuration
├── Metadata
└── Other internal files
```

You do **not** need to memorize PostgreSQL's internal filename mapping for this project.

The important understanding is:

> **PostgreSQL eventually needs persistent storage underneath its data directory.**

---

# 6. 🧾 Data Files vs WAL

This distinction is extremely important.

## Data

Data files represent the current persistent database state.

```text
Data
├── Tables
├── Indexes
└── Other database structures
```

Example:

```text
Customers
Orders
Accounts
Transactions
```

## WAL

WAL = **Write-Ahead Log**

WAL records information about database changes and supports:

- Durability
- Crash recovery
- Replication
- Backup/PITR workflows

Conceptually:

```text
WAL
├── INSERT changes
├── UPDATE changes
├── DELETE changes
└── Other recovery-related information
```

### Critical distinction

> **WAL is not simply a backup copy of the database.**

Think:

```text
Data Files
    ↓
Current database state

WAL
    ↓
Change information used for durability,
recovery, and replication
```

---

# 7. ✍️ PostgreSQL Write Path

Consider:

```sql
UPDATE customers
SET city = 'Hyderabad'
WHERE id = 100;
```

Simplified flow:

```text
                    SQL UPDATE
                        │
                        ▼
                 PostgreSQL Backend
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
       Data Page Change        WAL Record
             │                     │
             ▼                     ▼
          Memory               WAL Buffer
             │                     │
             ▼                     ▼
        Dirty Page           Durable WAL
             │
             ▼
       Eventually written
       toward storage
```

### Important principle

> **PostgreSQL uses Write-Ahead Logging so the required WAL information is made durable before the corresponding data-page changes are considered safely persisted.**

This supports database durability and crash recovery.

---

# 8. 🔍 PostgreSQL Read Path

Example:

```sql
SELECT * FROM customers
WHERE id = 100;
```

Simplified:

```text
Application
    │
    ▼
PostgreSQL Backend
    │
    ▼
Shared Buffers
    │
    ├── Page available?
    │
    ├── YES ─────────► Use cached page
    │
    └── NO
         │
         ▼
    Persistent Storage
         │
         ▼
      Read page
         │
         ▼
    Shared Buffers
         │
         ▼
    Execute Query
         │
         ▼
       Result
```

### Key idea

If the required page is already in memory:

```text
Query
 ↓
Shared Buffers
 ↓
Found
 ↓
Avoid storage read
```

Otherwise:

```text
Query
 ↓
Shared Buffers
 ↓
Not found
 ↓
Storage read
 ↓
Page loaded into memory
```

---

# 9. 🟠 Dirty Pages

A **dirty page** is a database page in memory that has been modified and differs from the version currently persisted on storage.

Flow:

```text
Persistent Storage
       │
       ▼
    Read Page
       │
       ▼
     Memory
       │
       ▼
   Modify Page
       │
       ▼
   Dirty Page
```

Eventually:

```text
Dirty Page
    │
    ▼
Background Write / Checkpoint
    │
    ▼
Persistent Storage
```

### Remember

> **Dirty page = modified in-memory database page that has not yet been written back as the current persistent version.**

---

# 10. 💾 What Is a Volume?

Moving from PostgreSQL into infrastructure:

A **volume** is persistent storage presented to a compute workload.

Conceptually:

```text
Server / Pod
     │
     ▼
Filesystem
     │
     ▼
Volume
     │
     ▼
Storage System
```

Example:

```text
PostgreSQL
    │
    ▼
/data
    │
    ▼
Volume
    │
    ▼
Enterprise Storage
```

PostgreSQL generally interacts with a filesystem/storage path.

It does not need to know:

> "I am using NetApp."

The infrastructure/storage layer provides that storage.

---

# 11. 🧱 Filesystem → Volume → Storage

A useful layered model:

```text
                    PostgreSQL
                         │
                         ▼
                    Data Directory
                         │
                         ▼
                     Filesystem
                         │
                         ▼
                       Volume
                         │
                         ▼
                  Storage Backend
```

The storage backend might be:

- Local disks
- Cloud block storage
- Enterprise storage arrays
- Network-attached storage
- Other supported storage systems

For this project, the important relationship is:

```text
PostgreSQL
    ↓
Persistent Storage
```

and how Kubernetes/CSI manages that relationship.

---

# 12. 🐳 Containers and Persistent Storage

This is one of the most important differences between your old and new project.

## Old project

AlloyDB Omni was primarily deployed on RHEL 9 VMs:

```text
RHEL 9 VM
   │
   ├── Filesystem
   │
   └── AlloyDB Omni
```

The database workload lived in the VM environment.

## New project

The proposed architecture is Kubernetes-oriented:

```text
Kubernetes Node
      │
      ▼
     Pod
      │
      ▼
AlloyDB Omni Container
      │
      ▼
     PVC
      │
      ▼
Persistent Storage
```

### Why?

Containers/pods can be recreated or replaced.

We do **not** want:

```text
Pod deleted
    ↓
Database data deleted ❌
```

Instead:

```text
Pod deleted
    ↓
New Pod
    ↓
Same Persistent Storage
    ↓
Database data remains
```

Therefore:

> **Container lifecycle and database storage lifecycle are separated.**

---

# 13. ☸️ PersistentVolume and PersistentVolumeClaim

Kubernetes provides persistent storage abstractions.

Simplified relationship:

```text
Pod
 │
 ▼
PVC
 │
 ▼
PV
 │
 ▼
Storage
```

## PVC — PersistentVolumeClaim

A workload requests storage.

Conceptually:

```yaml
storage:
  size: 10Ti
```

The PVC represents:

> **"My workload needs persistent storage with these requirements."**

## PV — PersistentVolume

The PV represents a persistent storage resource available to Kubernetes.

Think:

```text
PVC
 ↓
Request for storage

PV
 ↓
Persistent storage resource
```

---

# 14. 🏷️ StorageClass

A **StorageClass** describes how Kubernetes should provision storage.

Conceptually:

```text
PVC
 │
 ▼
StorageClass
 │
 ▼
CSI Driver
 │
 ▼
Storage Backend
```

A StorageClass can describe characteristics such as:

- Storage provisioner
- Performance characteristics
- Replication behavior
- Other storage-specific parameters

Exact parameters depend on the CSI driver.

### Key point

> **StorageClass helps Kubernetes know which storage mechanism should satisfy a PVC request.**

---

# 15. 🔌 Where CSI Fits

CSI = **Container Storage Interface**

CSI provides a standardized interface between Kubernetes and storage systems.

Architecture:

```text
                         Kubernetes
                              │
                              ▼
                             PVC
                              │
                              ▼
                             PV
                              │
                              ▼
                        CSI Interface
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
            NetApp           Pure       Other Storage
            CSI Driver       CSI Driver
                │             │             │
                ▼             ▼             ▼
          Storage Backend / Storage Array
```

### Key definition

> **CSI is not the storage itself.**

It is the interface through which Kubernetes can request storage operations from a compatible storage system.

---

# 16. 🏢 Enterprise Storage

Enterprise storage platforms provide centralized storage infrastructure and advanced storage management capabilities.

Examples relevant to the proposal include:

```text
NetApp
Pure Storage / Portworx
```

These are **storage platforms**, not databases.

They can provide capabilities such as:

- Persistent volumes
- High-performance storage
- Snapshots
- Clones
- Replication
- Storage management
- Data protection features

The exact capabilities depend on the product, configuration, and CSI integration.

---

# 17. 🏦 Why Enterprise Storage Matters for VLDB

Imagine an enterprise with:

```text
Database
   │
   ▼
50 TB / 100 TB / larger
   │
   ▼
Existing Enterprise Storage
```

The organization may already have:

- High-performance storage
- Storage replication
- Snapshot infrastructure
- Disaster recovery
- Storage administrators
- Monitoring
- Existing operational procedures

They may not want to redesign all of this simply because they are deploying AlloyDB Omni.

Therefore, the project asks:

> **How can AlloyDB Omni running in Kubernetes integrate effectively with enterprise storage through CSI?**

Conceptually:

```text
Existing Enterprise Storage
          +
       Kubernetes
          +
      AlloyDB Omni
```

---

# 18. 📸 Storage Snapshots

A storage snapshot represents a point-in-time state of a volume.

Example:

```text
10:00 → Database State A
10:05 → Snapshot
10:10 → Database State B
10:15 → Database State C
```

The snapshot represents approximately:

```text
Snapshot
   ↓
State at 10:05
```

Conceptual flow:

```text
Persistent Volume
       │
       ▼
   Snapshot
       │
       ▼
Point-in-time state
```

### Important

Snapshot behavior and consistency guarantees depend on:

- Kubernetes
- CSI driver
- Storage backend
- Database/application coordination

Therefore:

> **A fast storage snapshot is not automatically equivalent to an application-consistent database backup.**

This is why the project requires validation and documentation.

---

# 19. ⚡ Why Snapshots Can Help VLDB

Suppose:

```text
Database = 20 TB
```

A traditional data-copy operation might conceptually involve:

```text
20 TB
 ↓
Read
 ↓
Process
 ↓
Compress
 ↓
Upload
 ↓
Backup Storage
```

This can take significant time and consume considerable I/O.

A storage snapshot may operate at the storage layer:

```text
20 TB Volume
      │
      ├──────────────► Snapshot
      │
      └── Database continues running
```

This can potentially provide a much faster point-in-time operation.

### But remember

We still need to validate:

- Database consistency
- Snapshot correctness
- Restore behavior
- Recovery time
- Application behavior
- CSI driver support

---

# 20. 🧬 Storage Clone

A clone creates another storage instance based on an existing point-in-time state.

Conceptually:

```text
Production Volume
       │
       ▼
    Snapshot
       │
       ▼
     Clone
       │
       ▼
Test / Dev / Recovery
```

Potential use cases:

- Development
- Testing
- Recovery validation
- Troubleshooting
- Migration
- Analytics

For a VLDB:

> **Fast cloning can avoid repeatedly copying a huge dataset.**

---

# 21. 🔄 Restore

Restore means recovering data from a backup/snapshot/recovery source.

Simplified:

```text
Snapshot / Backup
       │
       ▼
     Restore
       │
       ▼
Recovered Volume
       │
       ▼
AlloyDB Omni
       │
       ▼
Recovered Database
```

Example:

```text
Production
    │
    ▼
Snapshot at 10:00
    │
    ▼
Problem at 12:00
    │
    ▼
Restore
    │
    ▼
Recovery Environment
```

The actual AlloyDB Omni restore workflow must be validated against the supported operator and CSI capabilities.

---

# 22. 🌍 Storage Replication

Storage replication is different from a snapshot.

## Snapshot

```text
Volume
  │
  ▼
Point-in-time state
```

## Replication

```text
Primary Storage
      │
      │ Continuous / scheduled replication
      ▼
Secondary Storage
```

Example:

```text
                  PRIMARY SITE
                       │
                  Storage Array
                       │
                       │ Replication
                       ▼
                    DR SITE
                       │
                  Storage Array
```

Replication is useful for:

- HA designs
- DR designs
- Site-level recovery
- Data protection

The exact replication mechanism depends on the storage platform and integration.

---

# 23. 🛡️ HA vs DR

These concepts must be clearly separated.

## HA — High Availability

Goal:

> **Keep the service available when an individual component fails.**

Example:

```text
              HA Cluster
                  │
          ┌───────┴───────┐
          ▼               ▼
       Primary          Standby
          │
       Failure 💥
          │
          ▼
       Standby
       becomes
       active
```

## DR — Disaster Recovery

Goal:

> **Recover service after a major failure such as a site, region, or storage-domain failure.**

Example:

```text
Production Site
      │
      │ Replication
      ▼
DR Site
```

### Easy memory trick

```text
HA → Keep running
DR → Recover elsewhere
```

---

# 24. 🚀 Storage Performance

For VLDB databases, storage can become a major performance factor.

Three important metrics:

## IOPS

**Input/Output Operations Per Second**

Question:

> How many I/O operations can storage handle per second?

Example:

```text
100,000 IOPS
```

## Throughput

Question:

> How much data can storage transfer per second?

Examples:

```text
500 MB/s
1 GB/s
5 GB/s
```

## Latency

Question:

> How long does an I/O operation take?

Conceptually:

```text
Request
  │
  ▼
Storage
  │
  ▼
Response
```

Latency measures the time between request and response.

### Database perspective

Generally:

```text
Lower latency
     +
Higher suitable IOPS
     +
Higher suitable throughput
     ↓
Better potential database performance
```

But workload characteristics matter.

Random I/O, sequential I/O, read-heavy workloads, write-heavy workloads, and concurrency can behave very differently.

---

# 25. 🧪 Benchmarking

The new project explicitly requires:

> **Perform benchmarking tests.**

We need to determine how storage behaves under AlloyDB Omni workloads.

Potential metrics:

| Metric | What it tells us |
|---|---|
| Read IOPS | Read operation capability |
| Write IOPS | Write operation capability |
| Read throughput | Read bandwidth |
| Write throughput | Write bandwidth |
| Latency | Storage responsiveness |
| Snapshot time | Snapshot operation duration |
| Clone time | Clone operation duration |
| Restore time | Recovery operation duration |
| Backup time | Backup workflow duration |
| Recovery time | Practical RTO |
| Replication lag | Replication behavior |

The goal is not simply:

> "Storage A is faster."

Instead:

> **Which storage approach provides the required performance, reliability, and operational characteristics for the intended AlloyDB Omni workload?**

---

# 26. 🔥 pgBackRest vs Storage-Level Operations

This is an important interview topic.

The project is **not automatically saying:**

```text
pgBackRest ❌
      ↓
CSI Snapshot ✅
```

Instead, we are evaluating storage-integrated approaches.

There can be multiple mechanisms:

```text
                     Backup / Recovery
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
      PostgreSQL-aware            Storage-level
        mechanisms                mechanisms
              │                         │
         pgBackRest                  Snapshot
                                   Clone
                                   Replication
```

## pgBackRest

A PostgreSQL-aware backup solution.

Conceptually:

```text
PostgreSQL
    │
    ▼
pgBackRest
    │
    ▼
Backup Repository
    │
    ▼
Object / Backup Storage
```

It can support PostgreSQL backup/recovery workflows such as:

- Full backups
- Incremental backups
- Differential backups
- WAL archiving
- PITR

## Storage Snapshot

Operates at the storage layer:

```text
Database Volume
      │
      ▼
Storage Snapshot
```

This can be very useful for large datasets, but database consistency and recovery semantics must be validated.

### Interview answer

> **We are not simply replacing pgBackRest. The project evaluates how storage-native capabilities such as snapshots, clones, and replication can complement or potentially serve specific backup/recovery use cases, depending on AlloyDB Omni and storage integration requirements.**

---

# 27. 🏗️ How This Relates to Your Old Project

Your previous AlloyDB Omni project primarily focused on:

> **Deploying and orchestrating AlloyDB Omni on RHEL 9 VMs.**

Simplified:

```text
                  RHEL 9 VM(s)
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     AlloyDB Omni     etcd       HAProxy
     PostgreSQL     Cluster      Load Balancer
          │           State           │
          └────────────┼─────────────┘
                       ▼
                 Keepalived / VIP
                       │
                       ▼
                    Clients
```

Your Ansible collection handled:

```text
Ansible
  │
  ├── Install
  ├── Bootstrap
  ├── Add
  ├── Remove
  ├── Update
  ├── Backup
  ├── Restore
  └── Switchover
```

You also had a declarative control-plane concept:

```text
Deployment Spec
      +
Resource Spec
      │
      ▼
alloydbctl
      │
      ▼
gRPC Orchestrator
      │
      ▼
AlloyDB Omni
```

---

# 28. ☸️ New Project Architecture

The new project moves toward Kubernetes-native deployment.

Simplified:

```text
                         Kubernetes
                             │
                     AlloyDB Operator
                             │
                             ▼
                       AlloyDB Omni
                             │
                             ▼
                         PostgreSQL
                             │
                             ▼
                            PVC
                             │
                             ▼
                            CSI
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          NetApp            Pure          Other Storage
```

The major areas of work are:

- Kubernetes operator
- CSI integration
- Enterprise storage
- Persistent volumes
- Snapshots
- Restore
- Clone
- Storage replication
- HA
- DR
- Benchmarking
- Storage best practices

---

# 29. 🔄 Old vs New Architecture

## 🟦 Old Project — RHEL 9

```text
User
 │
 ▼
Ansible
 │
 ▼
RHEL 9 VM
 │
 ├── RPMs
 ├── Config files
 ├── Services
 ├── etcd
 ├── HAProxy
 ├── Keepalived
 ├── pgBackRest
 └── AlloyDB Omni
          │
          ▼
       Filesystem
          │
          ▼
       Storage
```

Emphasis:

> **Host-level infrastructure automation and database cluster orchestration.**

## 🟩 New Project — Kubernetes

```text
User / Operator
      │
      ▼
 Kubernetes
      │
      ▼
AlloyDB Operator
      │
      ▼
AlloyDB Omni Pod
      │
      ▼
     PVC
      │
      ▼
     PV
      │
      ▼
CSI Driver
      │
      ▼
Enterprise Storage
```

Emphasis:

> **Kubernetes-native database deployment and enterprise storage integration.**

---

# 30. 🧠 End-to-End Mental Model

This is the most important architecture to remember from Chapter 3:

```text
                         APPLICATION
                              │
                              │ SQL
                              ▼
                     ┌─────────────────┐
                     │  AlloyDB Omni   │
                     │   PostgreSQL    │
                     └────────┬────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
               Data Files              WAL
                    │                   │
                    └─────────┬─────────┘
                              ▼
                       Persistent Volume
                              │
                             PVC
                              │
                              ▼
                              PV
                              │
                              ▼
                             CSI
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           NetApp            Pure          Other
        Storage Platform   Storage       Storage
```

### The complete story

```text
Application
    ↓
AlloyDB Omni
    ↓
PostgreSQL
    ↓
Data + WAL
    ↓
Persistent Volume
    ↓
Kubernetes PVC/PV
    ↓
CSI Driver
    ↓
Enterprise Storage
```

---

# 31. 🎤 Interview Questions & Answers

### Q1. Where does PostgreSQL data live?

> PostgreSQL stores persistent database data in its data directory, which ultimately resides on a filesystem backed by persistent storage.

### Q2. What is a PostgreSQL page?

> A page is a basic unit of PostgreSQL database storage. PostgreSQL pages are typically 8 KB.

### Q3. Do tables and indexes both consume storage?

> Yes. Table data and index structures both require persistent storage.

### Q4. What is the difference between data files and WAL?

> Data files represent the current persistent database state, while WAL records database changes used for durability, crash recovery, and replication.

### Q5. Is WAL a backup?

> No. WAL is a write-ahead change log used by PostgreSQL for durability, recovery, and replication. It is an important component of backup/PITR workflows but is not itself simply a database backup.

### Q6. What is a dirty page?

> A dirty page is a database page in memory that has been modified and differs from the version currently persisted on storage.

### Q7. Why does PostgreSQL use WAL?

> WAL provides a durable record of changes that PostgreSQL can use to maintain durability and recover the database after a crash.

### Q8. What is a volume?

> A volume is persistent storage presented to a workload through an infrastructure/storage layer.

### Q9. Why is persistent storage important for containers?

> Containers and pods can be recreated. Persistent storage separates the database data lifecycle from the container lifecycle so that replacing a pod does not mean losing the database data.

### Q10. What is a PVC?

> A PersistentVolumeClaim is a Kubernetes request for persistent storage with specified requirements such as capacity and storage characteristics.

### Q11. What is a PV?

> A PersistentVolume is a Kubernetes representation of persistent storage that can satisfy a workload's storage claim.

### Q12. What is a StorageClass?

> A StorageClass describes how Kubernetes should provision storage and which provisioner or storage mechanism should satisfy storage requests.

### Q13. What is CSI?

> CSI, or Container Storage Interface, is a standardized interface that allows Kubernetes to interact with storage systems through CSI drivers.

### Q14. Is CSI itself a storage system?

> No. CSI is an interface. The actual storage is provided by a storage backend such as NetApp, Pure/Portworx, cloud storage, or another supported system.

### Q15. Why is CSI important for this AlloyDB Omni project?

> CSI allows Kubernetes-based AlloyDB Omni workloads to integrate with enterprise storage platforms using Kubernetes-native persistent storage and supported storage operations such as provisioning, snapshots, cloning, restore workflows, and replication-related capabilities.

### Q16. What is a storage snapshot?

> A storage snapshot is a point-in-time representation of a storage volume.

### Q17. Why can snapshots be useful for VLDBs?

> A storage snapshot can potentially create a point-in-time storage state much faster than copying an entire multi-terabyte database, depending on the storage platform and implementation.

### Q18. Is a storage snapshot automatically a database-consistent backup?

> No. Consistency and recoverability depend on the database, CSI driver, storage platform, and coordination of the snapshot operation. This must be validated.

### Q19. What is a clone?

> A clone creates another storage instance based on an existing point-in-time state, which can be useful for testing, development, recovery validation, and other workloads.

### Q20. What is storage replication?

> Storage replication maintains another copy of data on another storage system or location, depending on the storage platform's replication mechanism.

### Q21. Snapshot vs replication?

| Snapshot | Replication |
|---|---|
| Point-in-time state | Keeps another copy synchronized |
| Useful for backup/recovery workflows | Useful for HA/DR scenarios |
| Represents a particular state | Represents ongoing/scheduled data movement |
| Usually within a storage system/domain | Can target another storage system/site |

### Q22. HA vs DR?

> HA focuses on keeping service available when components fail. DR focuses on recovering service after major failures such as site or infrastructure loss.

### Q23. What are IOPS?

> Input/Output Operations Per Second. It measures how many storage I/O operations the system can handle per second.

### Q24. What is throughput?

> Throughput measures how much data can be transferred per unit of time, such as MB/s or GB/s.

### Q25. What is latency?

> Latency is the time required for a storage operation to receive a response.

### Q26. What will you benchmark in this project?

> We can benchmark read/write IOPS, read/write throughput, latency, snapshot duration, clone duration, restore duration, backup duration, recovery time, and replication behavior or lag where applicable.

### Q27. Are we replacing pgBackRest?

> Not automatically. The project is evaluating storage-integrated capabilities such as snapshots, clones, and replication. Whether they complement, replace, or serve specific backup/recovery use cases depends on AlloyDB Omni support, consistency requirements, storage capabilities, and the desired RPO/RTO.

### Q28. What was different in your previous project?

> My previous project focused on deploying and orchestrating AlloyDB Omni on RHEL 9 VMs using Ansible, including installation, HA, scaling, upgrades, backup/restore, failover, and diagnostics. The new project moves toward Kubernetes-native deployment using the AlloyDB Omni operator and focuses heavily on CSI-based enterprise storage integration.

### Q29. What is the biggest architectural change?

> The old architecture was host-centric, with RHEL VMs, RPMs, filesystems, services, and Ansible orchestration. The new architecture is Kubernetes-centric, with the operator managing the database workload and PVC/CSI abstracting the persistent storage layer.

---

# 32. 🧠 Quick Concept Map

```text
                    POSTGRESQL STORAGE
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
        Tables          Indexes             WAL
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                      Data Files
                           │
                           ▼
                       Filesystem
                           │
                           ▼
                        Volume
                           │
                           ▼
                           PV
                           │
                           ▼
                           PVC
                           │
                           ▼
                          CSI
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       NetApp             Pure          Other Storage
          │
          ├── Snapshot
          ├── Clone
          ├── Restore
          └── Replication
```

---

# 33. 🚦 Chapter 3 Revision Checklist

## PostgreSQL Storage

- [x] PostgreSQL storage architecture
- [x] Database → tables → pages
- [x] PostgreSQL pages
- [x] Tables consume storage
- [x] Indexes consume storage
- [x] PostgreSQL data files
- [x] Data files vs WAL
- [x] Read path
- [x] Write path
- [x] Dirty pages
- [x] WAL purpose

## Infrastructure Storage

- [x] Volume
- [x] Filesystem
- [x] Persistent storage
- [x] Container storage
- [x] Pod lifecycle vs storage lifecycle
- [x] PersistentVolume
- [x] PersistentVolumeClaim
- [x] StorageClass
- [x] CSI

## Enterprise Storage

- [x] Enterprise storage concept
- [x] Why VLDBs need strong storage
- [x] NetApp concept
- [x] Pure/Portworx concept
- [x] Storage snapshots
- [x] Storage clones
- [x] Restore
- [x] Storage replication
- [x] HA vs DR

## Performance

- [x] IOPS
- [x] Throughput
- [x] Latency
- [x] Benchmarking
- [x] Snapshot/clone/restore benchmarking
- [x] Recovery time
- [x] Replication behavior

## AlloyDB Omni

- [x] Old RHEL 9 architecture
- [x] New Kubernetes architecture
- [x] Operator concept
- [x] PVC/PV
- [x] CSI integration
- [x] Enterprise storage integration
- [x] pgBackRest vs storage-level operations

---

# 34. 🧩 One-Minute Revision

If you have only one minute before an interview, remember this:

```text
                         APPLICATION
                              │
                              ▼
                       ALLOYDB OMNI
                              │
                              ▼
                         POSTGRESQL
                              │
                  ┌───────────┴───────────┐
                  ▼                       ▼
             DATA FILES                  WAL
                  │                       │
                  └───────────┬───────────┘
                              ▼
                       PERSISTENT VOLUME
                              │
                             PVC
                              │
                              ▼
                              PV
                              │
                              ▼
                             CSI
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                 NetApp             Pure/Other
                    │
                    ▼
        ┌───────────────────────────┐
        │ Enterprise Storage        │
        │                           │
        │ Snapshot                  │
        │ Clone                     │
        │ Restore                   │
        │ Replication               │
        └───────────────────────────┘
```

### Remember these definitions

```text
Page
→ Basic PostgreSQL storage unit, typically 8 KB

Data
→ Current persistent database state

WAL
→ Change information used for durability/recovery/replication

Dirty Page
→ Modified in-memory page not yet persisted as the current version

PVC
→ Kubernetes request for persistent storage

PV
→ Kubernetes persistent storage resource

StorageClass
→ Defines how Kubernetes provisions storage

CSI
→ Standard interface between Kubernetes and storage systems

Snapshot
→ Point-in-time storage state

Clone
→ New storage instance based on an existing state

Replication
→ Maintaining another copy of data

HA
→ Keep service available

DR
→ Recover after major failure

IOPS
→ Number of I/O operations per second

Throughput
→ Amount of data transferred per second

Latency
→ Time taken for an I/O operation
```

---

# 🚀 Next Chapter

**Chapter 3 is complete.** ✅

We deliberately stopped before deep PostgreSQL storage internals because the next part of the preparation should shift toward the technology central to the new proposal:

```text
             PostgreSQL
                  ↓
             AlloyDB Omni
                  ↓
             Kubernetes
                  ↓
            Operator
                  ↓
              Pod/State
                  ↓
              PVC / PV
                  ↓
             StorageClass
                  ↓
                CSI
                  ↓
       ┌──────────┴──────────┐
       ↓                     ↓
    NetApp              Pure/Portworx
```

## ➡️ Next: Chapter 4 — Kubernetes Fundamentals for AlloyDB Omni ☸️

We will cover only what is actually needed:

**Cluster → Node → Pod → Container → Deployment → StatefulSet → Service → ConfigMap → Secret → PV → PVC → StorageClass → CSI → Operator → CRD → Reconciliation**

No unnecessary Kubernetes theory. Then we will move quickly into **CSI and the actual POC**, where the preparation becomes directly project-specific. 🚀
