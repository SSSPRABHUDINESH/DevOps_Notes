# 🐘 Chapter 2 — PostgreSQL Architecture

> **Learning Track:** AlloyDB Omni + Kubernetes + CSI Storage  
> **Chapter Status:** ✅ COMPLETE  
> **Purpose:** Build the PostgreSQL architecture foundation needed for AlloyDB Omni, HA/DR, Kubernetes storage, CSI, and enterprise storage integration.

---

## 📚 Table of Contents

1. Chapter Objective
2. PostgreSQL High-Level Architecture
3. Client–Server Model
4. Connection Lifecycle
5. Authentication vs Authorization
6. PostgreSQL Processes
7. Backend Processes
8. PostgreSQL Memory Architecture
9. Shared Buffers
10. WAL Buffers
11. Per-Process Memory
12. PostgreSQL Pages
13. Data Directory
14. Data Files vs WAL
15. Read Path
16. Write Path
17. Dirty Pages
18. Write-Ahead Logging
19. Checkpoints
20. Crash Recovery
21. RPO and RTO
22. Tablespaces
23. PostgreSQL Configuration
24. Important Parameters
25. VACUUM — Basic Understanding
26. PostgreSQL Database Cluster vs HA Cluster
27. Old AlloyDB Omni Architecture
28. New Project Architecture
29. The Big Architectural Change
30. Where CSI Fits
31. End-to-End Mental Model
32. Interview Questions
33. Revision Checklist

---

# 1. 🎯 Chapter Objective

By the end of this chapter you should understand:

- How a client connects to PostgreSQL
- Backend processes
- PostgreSQL memory
- Shared buffers
- WAL buffers
- PostgreSQL pages
- Data directory
- Data files and WAL
- Read and write paths
- Dirty pages
- Checkpoints
- Crash recovery
- RPO and RTO
- Tablespaces
- Basic PostgreSQL configuration
- How this maps to AlloyDB Omni
- How your old RHEL 9 architecture differs from the new Kubernetes + CSI architecture

---

# 2. 🏗️ PostgreSQL High-Level Architecture

```text
                    Client / Application
                           │
                           │ SQL
                           ▼
                ┌─────────────────────┐
                │   PostgreSQL Server │
                │                     │
                │  Backend Processes  │
                │  Shared Memory      │
                │  Background Workers │
                └──────────┬──────────┘
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
            Data Files                WAL
                │                     │
                └──────────┬──────────┘
                           ▼
                    Persistent Storage
```

### Core idea

> **PostgreSQL is database software running on compute, but its persistent database state ultimately needs durable storage.**

---

# 3. 🔌 Client–Server Model

PostgreSQL follows a client/server model.

```text
        Client
          │
          │ SQL request
          ▼
   PostgreSQL Server
          │
          │ Result
          ▼
        Client
```

Clients can include:

- Applications
- `psql`
- Monitoring tools
- Other services
- Administrative tools

Example:

```bash
psql -h database-host -U postgres -d mydatabase
```

Then:

```sql
SELECT * FROM customers;
```

The client sends SQL to PostgreSQL and receives the result.

---

# 4. 🔄 Connection Lifecycle

Simplified connection flow:

```text
Application
    │
    │ Connection request
    ▼
PostgreSQL Server
    │
    ▼
Authentication
    │
    ▼
Authorization / access checks
    │
    ▼
Database session
    │
    ▼
Backend process handles requests
    │
    ▼
SQL execution
```

Example:

```text
psql
 │
 ├── Connect
 ├── Authenticate
 ├── Establish session
 └── Execute SQL
```

---

# 5. 🔐 Authentication vs Authorization

## Authentication

> **Who are you?**

Example:

```text
Username: dinesh
Password: ********
```

## Authorization

> **What are you allowed to do?**

```text
User: dinesh

SELECT  → ✅
INSERT  → ✅
UPDATE  → ✅
DELETE  → ❌
DROP    → ❌
```

Mental model:

```text
Connection
    ↓
Authentication
    ↓
Authorization
    ↓
Database Session
```

---

# 6. ⚙️ PostgreSQL Processes

PostgreSQL is **not one single process doing everything**.

It uses multiple processes/background workers for different responsibilities.

```text
                    PostgreSQL
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
     Backend        Background      WAL / I/O
     Processes       Processes       Processes
          │
          ▼
    Client requests
```

Conceptually, PostgreSQL has components responsible for:

- Client connections
- WAL processing
- Writing database pages
- Checkpoints
- Autovacuum
- Statistics/monitoring
- Other maintenance tasks

> For this project, understand the responsibilities conceptually; do not spend time memorizing every process name.

---

# 7. 👤 Backend Processes

A PostgreSQL backend process handles database work for a client session.

```text
Client 1 ─────► Backend Process 1
Client 2 ─────► Backend Process 2
Client 3 ─────► Backend Process 3
```

Simplified query flow:

```text
SQL
 ↓
Parse
 ↓
Plan
 ↓
Execute
 ↓
Return Result
```

---

# 8. 🧠 PostgreSQL Memory Architecture

```text
                    PostgreSQL
                         │
              ┌──────────┴──────────┐
              │                     │
        Shared Memory         Per-Process Memory
              │                     │
       ┌──────┴──────┐              │
       ▼             ▼              ▼
Shared Buffers   WAL Buffers   Query/session work
```

Two broad categories:

### Shared memory

Shared across PostgreSQL processes.

Examples:

- Shared buffers
- WAL-related buffers

### Per-process memory

Used by individual processes for operations such as:

- Sorting
- Hash operations
- Query processing
- Temporary work

---

# 9. 💾 Shared Buffers

**Shared buffers** are PostgreSQL's main shared memory area for caching database pages.

Without caching:

```text
Query
  ↓
Storage
  ↓
Read page
```

With shared buffers:

```text
Query
  ↓
Shared Buffers
  ↓
Page already available?
```

If yes:

```text
Query
 ↓
Shared Buffers
 ↓
Found ✅
```

If no:

```text
Query
 ↓
Shared Buffers
 ↓
Not found
 ↓
Persistent Storage
 ↓
Read page
 ↓
Shared Buffers
```

### Why it matters

Memory is much faster than persistent storage. Caching frequently accessed pages can reduce storage I/O.

For a VLDB:

```text
10 TB Database
      │
      ▼
Persistent Storage
      │
      │ Frequently accessed pages
      ▼
Shared Buffers
      │
      ▼
CPU / Query Execution
```

The entire database cannot fit in RAM, so storage performance still matters.

Important storage characteristics include:

- Latency
- IOPS
- Throughput

---

# 10. 📝 WAL Buffers

PostgreSQL also uses memory for WAL records before they are written to durable WAL storage.

```text
Transaction
    │
    ▼
WAL Record
    │
    ▼
WAL Buffer
    │
    ▼
Persistent WAL
```

Remember:

> **Shared buffers primarily relate to database data pages; WAL buffers relate to WAL records.**

WAL becomes important for:

- Replication
- Crash recovery
- Backup
- PITR
- HA/DR

---

# 11. 🧮 Per-Process Memory

Individual PostgreSQL processes may need their own memory:

```text
Backend Process
      │
      ├── Query processing
      ├── Sort
      ├── Hash operations
      └── Other temporary work
```

Mental model:

```text
PostgreSQL Memory
│
├── Shared Memory
│   ├── Shared Buffers
│   └── WAL-related buffers
│
└── Per-Process Memory
    └── Query / Session Operations
```

---

# 12. 📄 PostgreSQL Pages

PostgreSQL works with smaller units of database storage called **pages**.

A PostgreSQL page is typically:

> **8 KB**

```text
Table
│
├── Page 1
├── Page 2
├── Page 3
├── Page 4
└── ...
```

A page can contain multiple rows/tuples depending on their size.

PostgreSQL therefore does not need to read an entire multi-terabyte database to retrieve one row.

---

# 13. 📁 PostgreSQL Data Directory

PostgreSQL maintains a **data directory** containing persistent files required by its database cluster.

```text
PostgreSQL Data Directory
│
├── Database data
├── WAL
├── Configuration
├── Metadata
└── Other internal files
```

Typical installations may use paths such as:

```text
/var/lib/postgresql/...
```

but packaging and deployment can change the exact path.

> **For AlloyDB Omni, do not assume a specific host path. Understand the persistent data-directory concept instead.**

---

# 14. 🗃️ Data Files vs WAL

## Data Files

Represent the current persistent database state.

```text
Data Files
├── Tables
├── Indexes
└── Other database structures
```

## WAL

WAL = **Write-Ahead Log**

It records database changes used for durability and recovery.

```text
WAL
├── Change records
├── Transaction-related information
└── Recovery information
```

### Simplified mental model

> **Data files = current database state**

> **WAL = durable change history used for recovery/replication**

This is simplified but very useful for interviews.

---

# 15. 🔍 Read Path

Example:

```sql
SELECT * FROM customers WHERE id = 100;
```

Simplified flow:

```text
Application
    │
    ▼
PostgreSQL Backend
    │
    ▼
Shared Buffers
    │
    ├── Page exists?
    │       │
    │       └── YES → Use page
    │
    └── NO
        │
        ▼
    Persistent Storage
        │
        ▼
    Read database page
        │
        ▼
    Shared Buffers
        │
        ▼
    Execute query
        │
        ▼
      Result
```

A storage read can be avoided when the required page is already cached.

---

# 16. ✍️ Write Path

Example:

```sql
UPDATE customers
SET city = 'Hyderabad'
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
Modify Data
     │
     ├─────────────────┐
     ▼                 ▼
Data Page          WAL Record
     │                 │
     ▼                 ▼
Memory              WAL Buffer
     │                 │
     │                 ▼
     │          Persistent WAL
     │
     ▼
Eventually written
toward persistent storage
```

### Important principle

> **PostgreSQL uses Write-Ahead Logging so that required WAL information is made durable before the corresponding database-page changes are considered safely persisted.**

This supports durability and crash recovery.

---

# 17. 🟠 Dirty Pages

A database page in memory that has been modified and differs from the version currently persisted on storage is called a **dirty page**.

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
Background write / checkpoint
    │
    ▼
Persistent Storage
```

---

# 18. 🧾 Write-Ahead Logging — WAL

### Definition

> **Write-Ahead Logging means PostgreSQL records the necessary information about a database change in WAL before the corresponding changed data page is considered safely persisted.**

Simplified:

```text
Transaction
    │
    ├──────────────┐
    ▼              ▼
Data change      WAL record
    │              │
    ▼              ▼
Memory          Durable WAL
    │
    ▼
Data page eventually persisted
```

### Why?

If a change exists only in memory:

```text
Database
   ↓
Memory
   ↓
Data changed
   ↓
💥 Server crash
```

The in-memory change could be lost.

WAL provides a durable record PostgreSQL can use during recovery.

---

# 19. ⏱️ Checkpoints

A **checkpoint** establishes a known recovery point and causes PostgreSQL to flush modified data pages toward persistent storage.

```text
Changes
   ↓
Memory
   ↓
Dirty Pages
   ↓
Checkpoint
   ↓
Data written toward storage
```

Why?

After a crash, PostgreSQL can use:

```text
Checkpoint
    +
WAL after checkpoint
    ↓
Crash Recovery
```

Checkpoints influence:

- Recovery behavior
- Storage I/O patterns
- Write activity
- Database performance

---

# 20. 💥 Crash Recovery

Imagine:

```text
PostgreSQL
    │
    ├── Data pages in memory
    └── WAL persisted
```

Then:

```text
💥 Server crashes
```

On restart:

```text
PostgreSQL Startup
       │
       ▼
Crash Recovery
       │
       ▼
Read recovery state
       │
       ▼
Read required WAL
       │
       ▼
Replay required changes
       │
       ▼
Database reaches consistent state
```

### Important distinction

PostgreSQL's WAL is designed to protect committed transactions under the configured durability assumptions.

Work can still be lost if it was:

- Not committed
- Still only in memory
- Not durably persisted
- Affected by storage or replication failure

---

# 21. 🎯 RPO and RTO

## RPO — Recovery Point Objective

> **The maximum amount of data loss a system is designed to tolerate after a failure, normally expressed as a time period.**

Example: **RPO = 5 minutes**

```text
10:00 ───── 10:05 ───── 10:10
                         💥 Failure
                  ↑
             Recovery point
```

The business accepts that, in the worst case, approximately the latest 5 minutes of data may not be recoverable.

### RPO = 0

The requirement is effectively:

> No committed data loss within the defined failure model.

The achievable RPO depends on the durability and replication design.

---

## WAL vs RPO

Do not confuse them.

### WAL

A PostgreSQL mechanism:

```text
Database Changes
      ↓
WAL
      ↓
Durability / Recovery / Replication
```

### RPO

A system/business requirement:

```text
RPO
 ↓
How much data/time can the business afford to lose?
```

> **RPO is not simply "how much WAL was lost."**

---

## Replication and RPO

### Asynchronous replication

```text
Primary
   │
   │ Change
   ▼
Commit
   │
   │ Network delay
   ▼
Replica
```

If the primary fails before the replica receives the latest changes:

```text
Primary:   A B C D E F
Replica:   A B C D E
                    ↑
              Potential loss
```

This can produce a non-zero RPO.

### Synchronous concept

```text
Primary
   │
   │ Change
   ▼
Replica confirms
   │
   ▼
Commit
```

This can provide much stronger data-loss guarantees, depending on the exact implementation and configuration.

---

## RTO — Recovery Time Objective

> **RTO is the maximum acceptable time required to restore service after a failure.**

Example:

```text
RPO = 5 minutes
RTO = 30 minutes
```

Means:

```text
Failure 💥
   │
   ├── Data loss tolerance → ~5 minutes
   │
   └── Service recovery → within 30 minutes
```

### Easy memory trick

| Concept | Question |
|---|---|
| **RPO** | How much data can we lose? |
| **RTO** | How quickly must we recover? |

---

# 22. 🗂️ Tablespaces

A **tablespace** allows PostgreSQL database objects to be associated with a different physical storage location.

Without a custom tablespace:

```text
PostgreSQL
    │
    ▼
Default Storage
```

With tablespaces:

```text
PostgreSQL
    │
    ├── Default tablespace → Storage A
    │
    └── Custom tablespace  → Storage B
```

Example:

```text
Database
 │
 ├── Main data      → SSD storage
 └── Special data   → Different storage
```

For this project, understand the concept rather than advanced administration.

---

# 23. ⚙️ PostgreSQL Configuration

Two important configuration files:

```text
postgresql.conf
pg_hba.conf
```

## `postgresql.conf`

Controls many server settings:

- Memory
- Logging
- WAL
- Connections
- Checkpoints
- Query behavior

Mental model:

```text
postgresql.conf
       ↓
How PostgreSQL behaves
```

## `pg_hba.conf`

Controls client authentication/access rules.

```text
Client
  │
  ▼
pg_hba.conf
  │
  ├── Allowed → Continue
  │
  └── Not allowed → Reject
```

---

# 24. 🔧 Important PostgreSQL Parameters

You do not need to memorize hundreds of parameters.

## `max_connections`

Maximum concurrent client connections.

```text
Applications
   │
   ├── Connection 1
   ├── Connection 2
   ├── Connection 3
   └── ...
          ↓
   max_connections
          ↓
     PostgreSQL
```

Too many connections can consume significant resources.

This is one reason connection pooling can help.

---

## `shared_buffers`

Controls PostgreSQL shared buffer cache.

```text
Storage
   │
   ▼
Shared Buffers
   │
   ▼
Query
```

The correct value depends on RAM, workload, OS cache, and other memory requirements.

---

## `work_mem`

Memory available for certain individual query operations such as:

- Sorts
- Hash operations
- Other query processing

```text
Query
 │
 ├── Sort → work_mem
 │
 └── Hash → work_mem
```

Important:

> `work_mem` is not one global pool. Multiple operations/sessions can use it.

---

## `maintenance_work_mem`

Used for certain maintenance operations such as:

- `VACUUM`
- `CREATE INDEX`
- Other maintenance activities

```text
Normal query
    ↓
work_mem

Maintenance
    ↓
maintenance_work_mem
```

---

## WAL-related parameters

You may encounter:

```text
wal_level
max_wal_size
min_wal_size
```

At this stage focus on:

```text
Database Changes
       ↓
      WAL
       ↓
Durability
       ↓
Crash Recovery
       ↓
Replication
       ↓
Backup / PITR
```

Detailed WAL configuration belongs in later chapters.

---

## Checkpoint-related settings

Checkpoint settings influence:

- Recovery
- Storage I/O
- Write patterns
- Database performance

```text
Database Changes
       ↓
Dirty Pages
       ↓
Checkpoint
       ↓
Storage I/O
```

---

# 25. 🧹 VACUUM — Basic Understanding

PostgreSQL uses **MVCC (Multi-Version Concurrency Control)**.

At a high level, MVCC lets transactions work concurrently while maintaining consistent views.

Updates/deletes can create obsolete row versions.

Simplified:

```text
UPDATE row
    ↓
New row version
    ↓
Old version becomes obsolete
    ↓
VACUUM
    ↓
Cleanup / space management
```

Why storage matters:

> Database storage is not simply "write once and never touch it."

There is ongoing:

- Data modification
- Cleanup
- Index maintenance
- WAL activity
- Checkpoint activity
- Background I/O

This becomes relevant when benchmarking storage.

---

# 26. 🏷️ PostgreSQL Database Cluster vs HA Cluster

## PostgreSQL terminology

A PostgreSQL **database cluster** is a collection of databases managed by one PostgreSQL server instance/data directory.

```text
PostgreSQL Database Cluster
│
├── Database A
├── Database B
└── Database C
```

## HA terminology

An HA cluster may contain multiple database nodes:

```text
HA Cluster
│
├── Primary
├── Standby
└── Standby
```

> **PostgreSQL database cluster ≠ multi-node HA cluster**

---

# 27. 🏗️ Old AlloyDB Omni Project Architecture

Your previous project primarily deployed **AlloyDB Omni on RHEL 9 VMs**.

Simplified:

```text
                         RHEL 9 VM(s)
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        AlloyDB Omni        etcd           HAProxy
        PostgreSQL        Cluster          Load Balancing
              │            State               │
              │               │                 │
              └───────────────┼─────────────────┘
                              ▼
                        Keepalived / VIP
                              │
                              ▼
                       Database Clients
```

Your broader stack included:

- AlloyDB Omni engine
- PostgreSQL
- etcd
- Cluster Manager
- Node Manager
- HAProxy
- Keepalived
- pgBackRest
- pgbouncer
- Certificates
- RHEL 9
- RPM packages

Your Ansible collection automated lifecycle operations:

```text
Ansible
   │
   ├── Install
   ├── Bootstrap
   ├── Add node
   ├── Remove node
   ├── Update / rolling upgrade
   ├── Backup
   ├── Restore
   └── Switchover
```

You also had a declarative-style control plane:

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

This used Kubernetes-style declarative reconciliation without requiring a full Kubernetes cluster.

---

# 28. ☸️ New Project Architecture

The new proposal moves AlloyDB Omni toward a Kubernetes-native model.

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
          NetApp            Pure         Other Storage
```

The new project focuses on:

- Kubernetes
- Google-provided AlloyDB Omni operator
- CSI
- Enterprise storage
- Persistent Volumes
- Snapshots
- Restore
- Clone
- Storage replication
- HA
- DR
- Benchmarking
- Storage best practices

---

# 29. 🔄 The Big Architectural Change

## Old model

> **Infrastructure + OS + database software + cluster orchestration**

```text
VM
 │
 ▼
RHEL 9
 │
 ▼
RPM Packages
 │
 ▼
AlloyDB Omni
 │
 ▼
Ansible Orchestration
 │
 ▼
Database Operations
```

Your automation handled host-level operations.

## New model

More infrastructure management moves into Kubernetes:

```text
Kubernetes
 │
 ▼
AlloyDB Operator
 │
 ▼
AlloyDB Omni Pods
 │
 ▼
PVC
 │
 ▼
CSI
 │
 ▼
Enterprise Storage
```

### Key shift

> **From host-centric infrastructure automation toward Kubernetes-native application and storage orchestration.**

---

# 30. 🔌 Where CSI Fits

CSI = **Container Storage Interface**

It provides a standardized interface between Kubernetes and storage systems.

```text
                     Kubernetes
                          │
                          ▼
                    CSI Interface
                          │
            ┌─────────────┼─────────────┐
            ▼             ▼             ▼
       NetApp CSI     Portworx CSI   Other CSI
            │             │             │
            ▼             ▼             ▼
        Storage       Storage         Storage
```

CSI allows Kubernetes to request capabilities such as:

- Persistent volume provisioning
- Volume attachment
- Mounting
- Expansion
- Snapshots
- Cloning
- Restore-related workflows
- Other operations supported by the driver/backend

The exact capabilities depend on:

> **Kubernetes + CSI driver + storage backend + vendor implementation**

---

# 31. 🧠 End-to-End Mental Model

The most important architecture diagram from this chapter:

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
                             CSI
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           NetApp            Pure          Other
        Storage Platform   Storage       Storage
```

### The whole story

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
Kubernetes PVC
    ↓
CSI Driver
    ↓
Enterprise Storage
```

---

# 32. 🎤 Interview Questions

### Q1. What is PostgreSQL?

> PostgreSQL is an open-source relational database management system. AlloyDB Omni is built around PostgreSQL technology and extends it for enterprise and hybrid/on-premises deployment scenarios.

### Q2. How does a client interact with PostgreSQL?

> A client establishes a database connection, authenticates, creates a session, sends SQL requests, and receives query results. PostgreSQL uses backend processes to handle the session's database work.

### Q3. Is PostgreSQL a single process?

> No. PostgreSQL uses multiple processes and background workers for client handling, WAL, writing, checkpoints, autovacuum, and other database operations.

### Q4. What are shared buffers?

> Shared buffers are PostgreSQL memory used to cache database pages. If a required page is already in shared buffers, PostgreSQL can avoid reading that page from persistent storage.

### Q5. Why use memory instead of directly reading storage?

> Memory access is much faster than persistent storage access. Caching frequently accessed database pages can reduce storage I/O and improve query performance.

### Q6. What is a PostgreSQL page?

> A page is a basic unit of PostgreSQL storage. PostgreSQL pages are typically 8 KB.

### Q7. What is WAL?

> WAL, or Write-Ahead Log, records database changes in a durable log before corresponding data-page changes are considered safely persisted. WAL supports durability, crash recovery, and replication.

### Q8. What is a dirty page?

> A dirty page is a database page in memory that has been modified and differs from the version currently persisted on storage.

### Q9. What is a checkpoint?

> A checkpoint establishes a known recovery point and causes modified database pages to be flushed toward persistent storage. It helps reduce the amount of WAL PostgreSQL needs to process during crash recovery.

### Q10. What happens during crash recovery?

> PostgreSQL uses its persisted database state and WAL to bring the database back to a consistent state by replaying required WAL information.

### Q11. What is `postgresql.conf`?

> The main PostgreSQL server configuration file. It controls settings related to memory, connections, logging, WAL, checkpoints, and other database behavior.

### Q12. What is `pg_hba.conf`?

> A configuration file controlling client authentication and access rules.

### Q13. What is a tablespace?

> A PostgreSQL mechanism that allows database objects to be associated with a different physical storage location.

### Q14. What is RPO?

> RPO, or Recovery Point Objective, defines the maximum amount of data loss a system is designed to tolerate after a failure, normally expressed as a time period.

### Q15. What is RTO?

> RTO, or Recovery Time Objective, defines the maximum acceptable time required to restore service after a failure.

### Q16. What is the difference between RPO and RTO?

| Concept | Question |
|---|---|
| **RPO** | How much data can we lose? |
| **RTO** | How quickly must we recover? |

### Q17. Is RPO the same as WAL?

> No. WAL is a PostgreSQL mechanism used for durability and recovery, while RPO is a system/business requirement defining acceptable data loss.

### Q18. How does replication affect RPO?

> Synchronous replication can provide stronger data-loss guarantees depending on configuration. Asynchronous replication can have lag, which can result in a non-zero RPO if the primary fails before the latest changes reach the replica.

### Q19. What is the PostgreSQL data directory?

> The persistent filesystem area containing the PostgreSQL database cluster's data and supporting files.

### Q20. What is the difference between a PostgreSQL database cluster and an HA cluster?

> A PostgreSQL database cluster is a collection of databases managed by one PostgreSQL server instance/data directory. An HA cluster generally refers to multiple database nodes working together for availability or replication.

### Q21. How was your old AlloyDB Omni project different from this project?

> My previous project focused on deploying and orchestrating AlloyDB Omni on RHEL 9 VMs. We managed RPM installation, configuration, HA components, backups, upgrades, scaling, failover, and diagnostics using Ansible and a declarative orchestration layer. The new project moves toward Kubernetes-native deployment using the AlloyDB Omni operator and focuses heavily on integrating persistent storage through CSI with enterprise storage platforms.

### Q22. What is the major architectural change?

> The old model was more host-centric: RHEL VMs, RPMs, filesystems, and Ansible-managed infrastructure. The new model is Kubernetes-centric: the operator manages the database workload, Kubernetes manages container lifecycle, and PVC/CSI abstracts the persistent storage layer.

### Q23. What is CSI?

> CSI, or Container Storage Interface, is a standard interface that allows Kubernetes to interact with different storage systems through CSI drivers.

### Q24. Why is CSI important for the new AlloyDB Omni project?

> CSI allows AlloyDB Omni running in Kubernetes to use enterprise storage platforms through Kubernetes-native persistent volumes and storage operations such as provisioning, snapshots, cloning, restore workflows, and potentially replication capabilities supported by the storage driver.

### Q25. Does CSI itself provide the storage?

> No. CSI is an interface. The actual storage is provided by the storage backend, such as NetApp or Pure/Portworx. The CSI driver translates Kubernetes storage requests into operations understood by that storage platform.

---

# 33. ✅ Chapter 2 Revision Checklist

## PostgreSQL Architecture

- [x] Client/server model
- [x] Connection lifecycle
- [x] Authentication
- [x] Authorization
- [x] Backend processes
- [x] Background processes
- [x] Shared memory
- [x] Per-process memory
- [x] Shared buffers
- [x] WAL buffers
- [x] PostgreSQL pages
- [x] Data directory
- [x] Data files
- [x] WAL
- [x] Dirty pages
- [x] Checkpoints
- [x] Crash recovery
- [x] Read path
- [x] Write path
- [x] Tablespaces
- [x] PostgreSQL configuration
- [x] `postgresql.conf`
- [x] `pg_hba.conf`
- [x] `max_connections`
- [x] `shared_buffers`
- [x] `work_mem`
- [x] `maintenance_work_mem`
- [x] WAL configuration concepts
- [x] Checkpoint configuration concepts
- [x] Basic VACUUM/MVCC concept
- [x] PostgreSQL database cluster terminology
- [x] HA cluster terminology

## Recovery Concepts

- [x] WAL and durability
- [x] Crash recovery
- [x] RPO
- [x] RTO
- [x] Synchronous vs asynchronous replication concept
- [x] WAL vs RPO distinction

## AlloyDB Omni Context

- [x] Old RHEL 9 architecture
- [x] Ansible-based orchestration
- [x] Declarative AlloyDB control-plane concept
- [x] New Kubernetes-based architecture
- [x] AlloyDB Omni operator concept
- [x] PVC concept
- [x] CSI concept
- [x] Enterprise storage integration concept
- [x] Old vs new architectural difference

---

# 🧩 One-Minute Revision

If you have only one minute before an interview:

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
             DATA PAGES                  WAL
                  │                       │
                  └───────────┬───────────┘
                              ▼
                       PERSISTENT VOLUME
                              │
                             PVC
                              │
                             CSI
                              │
                    ENTERPRISE STORAGE
```

### Remember:

```text
READ:
Storage → Shared Buffers → Query → Client

WRITE:
Query → Memory → WAL → Durable Storage
              ↓
          Dirty Pages
              ↓
      Checkpoint / Background Write
              ↓
       Persistent Storage

CRASH:
Crash → Recovery → WAL Replay → Consistent Database

RPO:
How much data can we lose?

RTO:
How quickly must we recover?

OLD PROJECT:
RHEL 9 + RPMs + Ansible + Host-level orchestration

NEW PROJECT:
Kubernetes + Operator + Pods + PVC + CSI + Enterprise Storage
```

---

# 🚀 Next Chapter

**Chapter 2 is complete.** ✅

## Chapter 3 — PostgreSQL Storage & Data Files 💾

We will connect:

```text
PostgreSQL Data
      ↓
Pages
      ↓
Tables / Indexes
      ↓
Data Files
      ↓
Filesystem
      ↓
Block Storage
      ↓
Volumes
      ↓
Persistent Volumes
      ↓
Kubernetes PVC
      ↓
CSI
      ↓
NetApp / Pure / Other Storage
```

This chapter is where PostgreSQL fundamentals start directly connecting to the **enterprise storage + CSI problem statement**.
