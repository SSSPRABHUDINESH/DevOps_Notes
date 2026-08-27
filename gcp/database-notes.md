# ☁️ GCP Chapter 7 — Databases & Data Services



```text
Chapter 7 — GCP Databases & Data Services
│
├── 7.1 GCP Database Landscape
├── 7.2 Cloud SQL ⭐⭐⭐⭐⭐
├── 7.3 Cloud SQL HA, Backup & Replication ⭐⭐⭐⭐⭐
├── 7.4 Cloud SQL Connectivity & Security ⭐⭐⭐⭐⭐
├── 7.5 AlloyDB ⭐⭐⭐
├── 7.6 Memorystore ⭐⭐⭐⭐
├── 7.7 Firestore & Bigtable ⭐⭐
├── 7.8 BigQuery ⭐⭐⭐
└── 7.9 Database Selection + Interview Scenarios ⭐⭐⭐⭐⭐
```

Let's begin.

---

# ☁️ GCP Database Architecture: Beginner-Friendly Master Notes

---

## 📊 Quick Comparison: All GCP Databases at a Glance

| Database Service | Data Model | Primary Workload | Storage Medium | Read/Write Latency | Scalability Model | Primary Use Cases |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Cloud SQL** | Relational (RDBMS) | OLTP (Transactional) | Persistent Disk (SSD / HDD) | Single-digit ms | Vertical (Read replicas for read scaling) | Standard web apps, ERP, CRM, transactional backends |
| **AlloyDB** | Relational (PostgreSQL) | Demanding OLTP & HTAP | Disaggregated distributed block storage | Sub-millisecond to low ms | Vertical compute, auto-scaling reads, decoupled storage | Enterprise-scale PostgreSQL, high-throughput transactions |
| **Memorystore** | In-Memory NoSQL (Key-Value) | Low-latency Cache / Store | RAM (System Memory) | Sub-millisecond (Microseconds) | Vertical RAM sizing / Cluster sharding | Session storage, application caching, real-time counters |
| **Firestore** | NoSQL (Document Store) | OLTP / Real-time Sync | Distributed SSD multi-region storage | Single-digit ms | Fully horizontal (Automatic autoscaling) | Mobile/web backends, user profiles, collaborative apps |
| **Cloud Bigtable** | NoSQL (Wide-Column) | Heavy OLTP / Ingestion | Colossus (Distributed File System - SSD/HDD) | Sub-10ms (Down to <1ms) | Horizontal (Node additions without storage rebalance) | IoT telemetry, time-series, financial tick feeds, AdTech |
| **BigQuery** | Relational / Columnar | OLAP (Analytical) | Colossus (Capacitor columnar format) | Seconds to minutes (Scan-heavy) | Serverless dynamic compute slots | Enterprise data warehousing, BI dashboards, petabyte SQL |

---

| Database | Multi-Writer (Active-Active) Support? | How Writes Work |
| :--- | :--- | :--- |
| Cloud SQL | ❌ No | It uses a single primary architecture. All INSERT, UPDATE, and DELETE commands must go to the one primary node. Replicas are strictly for reading. |
| AlloyDB | ❌ No | Like Cloud SQL, it uses a single primary node for all writes. It scales massively for reads using read pools, but writes are still centralized. |
| Memorystore | ❌ No | Uses a standard Redis/Valkey Primary-Replica architecture. Writes go to the primary node. |
| Firestore | ✅ Yes | It is a globally distributed NoSQL database. In a multi-region setup, you can write data simultaneously from different regions, and Google handles the synchronization. |
| Cloud Bigtable | ✅ Yes | It fully supports multi-primary (active-active) replication. You can have clusters in different regions, and every cluster can accept reads and writes simultaneously. |

## ☁️ GCP Databases

```text
                       HOW TO CHOOSE YOUR DATABASE:
                                    │
           ┌────────────────────────┴────────────────────────────┐
           ▼                                                     ▼
   Transactional (OLTP)                                  Analytical (OLAP)
(Daily app clicks, orders, updates)                    (Big reports, 5-year trends)
           │                                                     │
   ┌───────┴───────────────────────────┐                         ▼
   ▼                                   ▼                      BigQuery
Relational (Tables & Rows)?         NoSQL?                 (Data Warehouse)
   │                                   │
   ├───────────────────┐               ├─────────────────────────────┐
   ▼                   ▼               ▼                             ▼
Standard Workload  Heavy Enterprise    Documents (JSON)?     Huge Stream / IoT?
Cloud SQL          AlloyDB               Firestore                 Bigtable
(Postgres/MySQL)   (Super Postgres)    (Web & Mobile Apps)      (Time-Series & Telemetry)
                                   │
                                   ▼
                           Need Extreme Speed?
                                   │
                                   ▼
                              Memorystore
                             (RAM Caching)

```

---

## 1. 🗄️ Cloud SQL

### 1.1 Overview & Core Purpose

* **What is it?** Cloud SQL is Google's managed version of standard relational databases: **PostgreSQL, MySQL, and SQL Server**.


* **What does it do?** It handles daily transactions (OLTP)—like creating user accounts, updating balances, or placing orders—using standard SQL tables with rows and columns.


* **Why use it instead of your own VM?** Google takes care of server maintenance, OS updates, security patches, automated backups, and failure recovery so you only manage your data and queries.

---

### Architecture diagram with read replica:
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Google Cloud (VPC)                                │
│                                                                             │
│  [ Application / App Engine / GKE ] ───── (Heavy Read Traffic) ─────────┐   │
│           │                                                             │   │
│           ▼ (Writes & Standard Reads)                                   ▼   │
│  [ Primary DB Endpoint (Internal IP) ]                     [ Replica IP ]   │
│           │                                                             │   │
│ ══════════╪═════════════════════════════════════════════════════════════╪══ │
│           │                  Google Managed Network                     │   │
│           │                                                             │   │
│      [ Zone A ]                     [ Zone B ]                [ Zone C ]    │
│     ┌───────────┐                 ┌───────────┐             ┌───────────┐   │
│     │  Primary  │                 │  Standby  │             │   Read    │   │
│ ────► Instance  │                 │ Instance  │             │  Replica  │   │
│     └─────┬─────┘                 └─────┬─────┘             └─────▲─────┘   │
│           │                             │                         │         │
│           ├── (Synchronous Sync) ───────┤                         │         │
│           │                             │                         │         │
│           └──────────────── (Asynchronous Replication) ───────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
```
---
### 1.2 Caching Strategy & Access Pattern

* **Source of Truth:** Cloud SQL holds your real, permanent data on disk.


* **How Caching Works (Cache-Aside):** If thousands of users ask for the exact same data (e.g., a product page), querying Cloud SQL every time will slow it down. Your app checks a fast RAM cache (Memorystore) first; if missing, it grabs it from Cloud SQL and saves a copy in the cache for next time.


* **Read Replicas:** If your app does 90% reading and 10% writing, you can create "copycat" read-only instances (Read Replicas) to handle read traffic so your main database stays fast.



```text
[ User App ] ──── (1. Check Cache First) ────► [ Memorystore (Redis) ]
     │                                                │
(2. Miss: Fetch from DB)                              │ (Found: Instant reply)
     ▼                                                ▼
[ Cloud SQL (Main DB) ] ──── (Copy Data) ────► [ Read Replicas (Reads Only) ]

```
---

### Commands for connecting:

**Connecting for Writes (Primary Instance)**
All write operations (`INSERT`, `UPDATE`, `DELETE`) and critical real-time reads must be routed to the Primary instance.

* **Via Direct IP (using `psql`):**
`psql -h <PRIMARY_INSTANCE_IP> -U postgres -d your_database`
* **Via Cloud SQL Auth Proxy:**
Start the proxy: `./cloud-sql-proxy <PROJECT_ID>:<REGION>:<PRIMARY_INSTANCE_NAME>`
Connect app to proxy: `psql -h 127.0.0.1 -U postgres -d your_database`

**Connecting for Reads (Read Replica)**
All heavy analytical or reporting `SELECT` queries should be routed to the Read Replica instance to avoid burdening the primary database.

* **Via Direct IP (using `psql`):**
`psql -h <READ_REPLICA_IP> -U postgres -d your_database`
* **Via Cloud SQL Auth Proxy:**
Start the proxy: `./cloud-sql-proxy <PROJECT_ID>:<REGION>:<READ_REPLICA_INSTANCE_NAME>`
Connect app to proxy: `psql -h 127.0.0.1 -U postgres -d your_database`

---

### 1.3 Storage Hierarchy & Comparisons

* **Data Organization:** Rigid, structured tables linked together (Foreign Keys and Primary Keys).


* **Storage Type:** Built on Google's Persistent Disks (SSDs or HDDs).

| Feature | Cloud SQL | Running DB on Compute Engine VM | AlloyDB |
| :--- | :--- | :--- | :--- |
| **Management** | Google manages OS, backups, updates | You do everything manually | Google manages + extra AI tuning |
| **Storage** | Normal Cloud Persistent Disk | Standard Attached Disk | Separate, high-speed shared storage |
| **Read Scaling** | Add Read Replicas | Manually configure replication | Auto-scaling groups of read pools |
### 1.4 Under the Hood: Persistence & Durability

* **Auto Storage Growth:** If your database starts running out of disk space, Google will automatically expand the disk size without shutting down the database.


* **Write-Ahead Log (WAL):** Every change is written safely to a transaction log before it is officially committed, ensuring no data is lost during a crash.


* **High Availability (HA):** Google keeps a Primary instance in Zone A and a Standby instance in Zone B. If Zone A fails, the Standby in Zone B takes over automatically in under 60 seconds.


* **Point-in-Time Recovery (PITR):** Did an engineer accidentally delete a table at 2:05 PM? You can roll back your database state to exactly 2:04 PM.



### 1.5 GCP Architecture & Provisioning

* **Private IP (PSA):** Cloud SQL sits inside Google's private network and talks directly to your VPC using Private Services Access (PSA). No public internet needed.


* **Cloud SQL Auth Proxy:** A small helper program you run next to your app that handles secure SSL/TLS connections and IAM authentication automatically.
      
  *  Looks similar as Identity Aware proxy (IAP) for loadbalancer - share a similar security philosophy: Enforcing IAM-based `authorization` and `encryption` without exposing resources directly to the public intern
      
  *  Creates a secure TLS 1.3 tunnel from the proxy client directly to the Cloud SQL backend.



```text
┌──────────────────────────────────────┐          ┌──────────────────────────────────────┐
│          Your VPC Network            │          │      Google's Managed Network        │
│                                      │          │                                      │
│  [ Your App / GKE Pod ]              │          │         Zone A        Zone B         │
│         │                            │          │       ┌─────────┐   ┌─────────┐      │
│         ▼                            │          │       │ Primary │──►│ Standby │      │
│  [ Cloud SQL Auth Proxy ]            │          │       └────┬────┘   └─────────┘      │
│         │                            │          │            │ (Mirrored Disks)        │
│         └───( Private IP Peering )───┼──────────┼────────────┘                         │
└──────────────────────────────────────┘          └──────────────────────────────────────┘

```

### 1.6 Sample Structure & Example

```sql
-- Creating structured relational tables
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(user_id),
    total_amount NUMERIC(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING'
);

```

# 7.1.3 ⚡ Transactional vs Analytical

This distinction is **very important**.

## Data Processing Systems or Database Workloads.

### ⚡ OLTP vs 📊 OLAP Comparison

| 🌟 Feature | ⚡ OLTP (Transactional) | 📊 OLAP (Analytical) |
| --- | --- | --- |
| 📖 **Meaning** | **O**nline **T**ransaction **P**rocessing | **O**nline **A**nalytical **P**rocessing |
| 🎯 **Primary Purpose** | Daily operations & real-time transactions | Analyzing massive historical data (BI) |
| ⚙️ **Operations** | Fast `INSERT`, `UPDATE`, `DELETE`, `SELECT` | Complex, read-heavy queries across billions of rows |
| 📅 **Data Scope** | Current, real-time data | Historical, aggregated data |
| 🏢 **Use Cases** | Banking apps, E-commerce, User portals | "What were our 5-year sales trends across regions?" |
| ☁️ **GCP Services** | Cloud SQL, AlloyDB, Cloud Spanner | BigQuery |
| 🔄 **Data Flow** | App ➔ Transactional DB | Transactional DB ➔ ETL ➔ Data Warehouse ➔ Analytics |

---

### 📦 Data Ingestion Scenarios


* 🚀 **1KB per Transaction (10,000 times) ➔ OLTP**
* **Why:** Fast, tiny, highly concurrent operations. This mimics a live web app (like an e-commerce site) processing thousands of simultaneous user actions.


* 🛑 **100GB in a Single Transaction ➔ OLAP**
* **Why:** This is a massive bulk load. Analytical DBs (like BigQuery) are built to swallow giant chunks at once. Trying this on an OLTP database will crash it, lock all tables, or exhaust the transaction log.

---

### 🚫 The 100GB Limit on Cloud Spanner & AlloyDB

**Can they handle a 100GB single transaction? No.**
Both are governed by OLTP mechanics and expect data in smaller, faster chunks.

* 🌐 **Cloud Spanner:** Enforces strict hard limits to maintain global consistency and millisecond latency. A single commit is limited to a maximum of **100 MB** (or 20,000 mutations). It will physically block a 100GB transaction from starting.
* 🐘 **AlloyDB:** While highly optimized, a single 100GB `INSERT` would overwhelm PostgreSQL's Write-Ahead Log (WAL), exhaust memory limits, create massive locking issues, and eventually crash or time out.

### ✅ The Solution: How to Load 100GB

* **Batching & Bulk Loading:** You never load 100GB in a single transaction.
* **The Tools:** Use processing services like **Google Cloud Dataflow** or **Dataproc**.
* **The Method:** Break the 100GB file into thousands of small, manageable transactions (e.g., 50MB at a time) and stream them into the database concurrently.


### Transactional — OLTP

Application performs:

```text
INSERT
UPDATE
DELETE
SELECT
```

Example:

```text
Banking application
Order management
Employee application
```

Think:

> **Cloud SQL / AlloyDB**

---

### Analytical — OLAP

You have huge amounts of data and want questions such as:

> "What were our sales across all regions during the last five years?"

Think:

> **BigQuery**

Architecture:

```text
Applications
    │
    ▼
Transactional DB
    │
    │ ETL / ELT
    ▼
Data Warehouse
    │
    ▼
BigQuery
    │
    ▼
Analytics / BI
```

---

## 2. 🧬 AlloyDB for PostgreSQL

### 2.1 Overview & Core Purpose

* **What is it?** AlloyDB is a supercharged, enterprise version of PostgreSQL built by Google.


* **What does it do?** It runs standard PostgreSQL code with zero changes, but runs transactional queries up to **4x faster** and analytical queries up to **100x faster**.


* **When to use it?** When standard Cloud SQL PostgreSQL is struggling with massive scale, heavy loads, or large analytical queries on transactional data.



### 2.2 Caching Strategy & Access Pattern

* **Two-Layer Memory Cache:** AlloyDB uses normal RAM (Tier 1) plus ultra-fast local NVMe flash drives (Tier 2) to keep frequently accessed data ready in sub-milliseconds.


* **Columnar Engine:** It automatically creates an in-memory column-based copy of your data so you can run heavy reporting queries instantly without slowing down normal user transactions.

A **columnar engine** stores database records by column rather than by row. This design is specifically optimized for analytical processing (OLAP) and heavy reporting.

**How It Works**

Example: 
**The Logical Table (What you see)**

| ID | Name | Date | Amount |
| --- | --- | --- | --- |
| 1 | Alice | Jan-01 | $10 |
| 2 | Bob | Jan-02 | $20 |
| 3 | Charlie | Jan-03 | $30 |

**Row-Based Storage (Traditional OLTP like Cloud SQL)**

The database writes the data to the physical hard drive sequentially, record by record.

* **Disk Layout:** `[1, Alice, Jan-01, $10]` `[2, Bob, Jan-02, $20]` `[3, Charlie, Jan-03, $30]`
* **The Query:** `SELECT SUM(Amount) FROM table;`
* **How it executes:** The database engine must load the entire first row into memory just to see the `$10`. Then it loads the entire second row to see the `$20`, and so on. If the table has a billion rows with 50 columns, the database is forced to read massive amounts of irrelevant data (Names, Dates) from the disk into memory just to find the Amounts.

**Column-Based Storage (Analytical like BigQuery)**

The database writes the data to the physical hard drive sequentially, column by column.

* **Disk Layout:** `[1, 2, 3]` `[Alice, Bob, Charlie]` `[Jan-01, Jan-02, Jan-03]` `[$10, $20, $30]`
* **The Query:** `SELECT SUM(Amount) FROM table;`
* **How it executes:** The database engine ignores the ID block, ignores the Name block, and ignores the Date block. It jumps directly to the physical location of the Amount block on the disk and reads `[$10, $20, $30]` in one continuous, lightning-fast swoop. It reads almost zero irrelevant data, saving massive amounts of time and computing power.

**Key Benefits**

* **Blazing Fast Aggregations:** Mathematical operations across millions of records (like `COUNT`, `SUM`, or `AVG`) execute instantly because the engine only scans the relevant data.
* **High Compression:** Since data within a single column is of the exact same type and often highly repetitive (e.g., dates or country codes), it can be compressed incredibly tightly.
* **Reduced I/O:** Reading fewer columns means moving significantly less data from disk to memory, which drastically speeds up query times and lowers computing costs.


### 2.3 Storage Hierarchy & Comparisons

* **Decoupled Architecture:** The compute processors (CPUs) and the storage system are completely separated. Storage expands automatically up to 128TB.



| Feature | Cloud SQL for PostgreSQL | AlloyDB for PostgreSQL |
| :--- | :--- | :--- |
| **Storage System** | Single attached persistent disk | Smart, multi-zone distributed shared storage |
| **Analytical Speed** | Normal PostgreSQL speed | Up to 100x faster (Built-in Columnar Engine) |
| **Failover Time (HA)** | Under 60 seconds | Under 10 seconds (Nearly instantaneous) |
### 2.4 Under the Hood: Persistence & Durability

* **Log-Based Storage:** The database compute engine only sends light transaction logs (WAL) over the network. The storage layer processes and saves them independently, eliminating I/O bottlenecks.


* **Zero-Impact Backups:** Backups are handled directly by the storage layer with zero performance slowdown on your live database.



### 2.5 GCP Architecture & Provisioning

* **Cluster Model:** You create one **AlloyDB Cluster** which manages your storage, one active **Primary Instance** for writes, and auto-scaling **Read Pools** for read traffic.



```text
┌────────────────────────────────────────────────────────────────────────┐
│                        AlloyDB Cluster (Regional)                      │
│                                                                        │
│  ┌──────────────────────────┐          ┌────────────────────────────┐  │
│  │ Primary Node (Writes)    │          │ Read Pool Nodes (Reads)    │  │
│  └─────────────┬────────────┘          └─────────────┬──────────────┘  │
│                │ (Tiny Log Changes)                  │                 │
│                ▼                                     ▼                 │
│  ════════════════════════════════════════════════════════════════════  │
│          Smart Distributed Storage Layer (Replicated Multi-Zone)       │
│  ════════════════════════════════════════════════════════════════════  │
└────────────────────────────────────────────────────────────────────────┘

```

### 2.6 Sample Structure & Example

```sql
-- Turn on the fast analytical Columnar Engine
ALTER TABLE user_payments SET (google_columnar_engine.enabled = true);

-- Query millions of rows in milliseconds
SELECT 
    payment_method,
    COUNT(*) AS total_transactions,
    SUM(amount) AS total_settled
FROM user_payments
GROUP BY payment_method;

```

---

## 3. ⚡ Memorystore (Redis & Valkey)

### 3.1 Overview & Core Purpose

* **What is it?** Memorystore is a fully managed in-memory data store running open-source **Redis** or **Valkey**.


* **What does it do?** It stores all data directly in computer **RAM** (not on hard drives) for microsecond (sub-millisecond) response times.


* **Core Rule:** **Memory = Speed**. It is used as a cache or session store, not as your primary permanent database.



### 3.2 Caching Strategy & Access Pattern

* **Cache Hit vs. Cache Miss:**
* **Hit:** App asks Memorystore for data $\rightarrow$ Found in RAM $\rightarrow$ Returns in microseconds.


* **Miss:** App asks Memorystore $\rightarrow$ Not found $\rightarrow$ Fetches from Cloud SQL $\rightarrow$ Saves a clone into Memorystore for next time.




* **TTL (Time to Live):** Keys can be set to automatically expire and delete after a given time (e.g., 1 hour for login sessions).



```text
               Is the data in Memorystore (RAM)?
                               │
                ┌──────────────┴──────────────┐
                ▼                             ▼
             [ YES ]                       [ NO ]
         (Cache HIT ⚡)                 (Cache MISS 🐢)
                │                             │
        Return immediately             Fetch from Cloud SQL
     (Sub-millisecond speed)                  │
                                              ▼
                                    Save a copy in RAM for next time

```

### 3.3 Storage Hierarchy & Comparisons

* **Data Model:** NoSQL Key-Value dictionary. Stores Strings, Lists, Sets, and Hashes.

| Feature | Memorystore (Redis/Valkey) | Cloud SQL | Firestore |
| :--- | :--- | :--- | :--- |
| **Where Data Lives** | **RAM (Fast, Volatile)** | Disk (Permanent) | Distributed SSD (Permanent) |
| **How You Look Up Data** | By exact Key name | Complex SQL Queries | Field filters in JSON documents |
| **Primary Job** | Speed Booster / Sessions | Permanent Source of Truth | Permanent App Document Store |

### 3.4 Under the Hood: Persistence & Durability

* **Why Disks are used:** RAM gets wiped clean if power is lost. Memorystore can write background copies to disk via **RDB Snapshots** (periodic full backups) and **AOF logs** (sequential transaction log of all writes) to restore data on restart.


* **High Availability (Standard Tier):** Automatically keeps a Primary RAM node and a Replica RAM node in separate zones with automatic failover.



### 3.5 GCP Architecture & Provisioning

* **Basic Tier:** Single RAM node (ideal for non-critical development caching).


* **Standard Tier:** High Availability setup with cross-zone replication and failover.


* **Private IP:** Gives you an internal IP endpoint (e.g., `10.0.0.15:6379`) accessible only inside your VPC network.



```text
┌──────────────────────────────────────┐          ┌──────────────────────────────────────┐
│          Your VPC Network            │          │      Google's Managed Network        │
│                                      │          │                                      │
│  [ Your App / Cloud Run / GKE ]      │          │   [ Primary RAM ]   [ Replica RAM ]  │
│                 │                    │          │        (Zone A)          (Zone B)    │
│                 └───( VPC Peering )──┼──────────┼───────────┴─────────────────┘        │
│                                      │          │     Private IP: 10.0.0.15:6379       │
└──────────────────────────────────────┘          └──────────────────────────────────────┘

```

### 3.6 Sample Structure & Example

```bash
# 1. Store a user login session that expires in 3600 seconds (1 hour)
SET session:user_101 "{\"auth\": true, \"role\": \"admin\"}" EX 3600

# 2. Increment a real-time page visit counter
INCR page:home:views

# 3. Add game scores to a live leaderboard
ZADD leaderboard:game 1500 "PlayerOne"
ZADD leaderboard:game 2400 "PlayerTwo"

```

---

## 4. 📄 Firestore

### 4.1 Overview & Core Purpose

* **What is it?** Firestore is a serverless NoSQL document database built for mobile, web, and serverless apps.


* **What does it do?** It stores data as flexible, JSON-like objects (Documents) organized into folders (Collections).


* **Key Superpower:** Real-time data sync (when data changes on the server, mobile screens update instantly over WebSockets) and offline data support.



### 4.2 Caching Strategy & Access Pattern

* **Built-in Offline Cache:** The mobile/web SDK automatically keeps a copy of data on the user's phone. Users can still use the app with no internet; changes sync once back online.


* **Automatic Indexing:** Every single field in every document is indexed automatically, so lookups are fast without extra setup.



### 4.3 Storage Hierarchy & Comparisons

* **Hierarchy:** **Collection** (Folder) $\rightarrow$ **Document** (JSON Object) $\rightarrow$ **Fields / Subcollections**.



```text
📁 Collection: "users"
    │
    ├── 📄 Document: "user_101"
    │     ├── name: "Dinesh"
    │     ├── city: "Hyderabad"
    │     └── 📁 Subcollection: "orders" ──► 📄 Document: "order_abc"
    │
    └── 📄 Document: "user_102"
          ├── name: "Alex"
          └── role: "Architect"

```

| Feature | Firestore | Memorystore | Cloud SQL |
| :--- | :--- | :--- | :--- |
| **Data Layout** | Flexible JSON Documents | Key-Value Pairs | Strict Tables & Rows |
| **Schema** | Schema-less (Each doc can have different fields) | Opaque Strings/Objects | Fixed schema (All rows share same columns) |
| **Real-Time Sync** | Built-in live listeners | Redis Pub/Sub | Not supported natively |

### 4.4 Under the Hood: Persistence & Durability

* **Zero Infrastructure to Manage:** Serverless—there are no VMs, RAM limits, or storage disks to size. It scales up and down automatically.


* **Strong Multi-Zone Replication:** Built on Google's Spanner engine. Every write is saved synchronously across multiple zones before confirming success.

> Doubt: But cloud spanner is relational db then how it is used in noSQL system ?

Cloud Spanner *is* a strongly consistent, relational database.

It seems contradictory that a NoSQL document database like Firestore runs on top of a relational SQL database, but it comes down to how Google separates the developer interface from the underlying storage engine.

**The Two Layers of Firestore**

* **The Developer Interface (NoSQL):** When you interact with Firestore, you use its NoSQL API. You send flexible, JSON-like documents and organize them into collections. You never write SQL or manage rigid tables.
* **The Storage Engine (Relational):** Behind the scenes, Google's internal systems take your NoSQL documents and translate them into a format that can be stored within Spanner's massive, highly structured relational tables.

**Why Google Built It This Way**
Google wanted mobile and web developers to have the speed, flexibility, and real-time syncing of a NoSQL database, but they needed the backend to be virtually indestructible. By using Spanner as the underlying engine, Firestore inherits Spanner's enterprise-grade superpowers without forcing you to write complex SQL:

* **ACID Transactions:** Ensuring that complex, multi-document writes (like deducting money from one account and adding it to another) succeed or fail together perfectly.
* **Global Consistency:** Synchronous data copying across multiple geographic zones so that no matter where a user is, they always read the most up-to-date data.


### 4.5 GCP Architecture & Provisioning

* **Location Scope:** Choose **Regional** (lower cost/latency) or **Multi-Regional** (maximum uptime, 99.999% SLA).


* **Security Rules:** Access is controlled directly from mobile clients using built-in declarative Firebase Security Rules instead of backend database passwords.

```text
[ Mobile App / Browser / Cloud Run ]
                 │
                 ▼ (Direct HTTPS / WebSocket Listeners)
┌────────────────────────────────────────────────────────────────────────┐
│                        Firestore Service                               │
│     [ Zone A Copy ] ◄───► [ Zone B Copy ] ◄───► [ Zone C Copy ]        │
└────────────────────────────────────────────────────────────────────────┘

```

### 4.6 Sample Structure & Example

```json
// Document Path: users/user_101
{
  "name": "Dinesh Kumar",
  "city": "Hyderabad",
  "age": 25,
  "skills": ["GCP", "Kubernetes", "PostgreSQL"],
  "is_active": true,
  "registered_date": "2026-08-27"
}

```

---

## 5. 🧱 Cloud Bigtable

### 5.1 Overview & Core Purpose

* **What is it?** Bigtable is Google's massive, distributed wide-column NoSQL database engine (the same tech powering Google Search and Maps).
* **What does it do?** It handles huge streams of incoming data with single-digit millisecond response times.


* **When to use it?** When you have terabytes or petabytes of streaming data—like IoT sensor logs, GPS trackers, stock ticker streams, or AdTech click logs.



### 5.2 Caching Strategy & Access Pattern

* **Row Key is Everything:** Data is sorted alphabetically by a single **Row Key**.


* **Access Rule:** You look up data by exact Row Key or read a consecutive range of Row Keys. You cannot run complex ad-hoc searches without the row key.



```text
Best Pattern:  Get sensor data for "SENSOR#101" between "10:00 AM" and "10:05 AM"
Bad Pattern:   "Find all sensors where temperature is greater than 100°F" (Requires slow full table scan)

```

### 5.3 Storage Hierarchy & Comparisons

* **Data Model:** Rows with Column Families containing individual timestamped columns.



| Feature | Cloud Bigtable | Firestore | BigQuery |
| :--- | :--- | :--- | :--- |
| **Best Used For** | High-speed, heavy write streams (Petabytes) | Mobile/Web user documents | Business analytics & reporting |
| **Lookups** | By Row Key & Key Ranges only | Filter by any JSON property | Full SQL Queries & Table Joins |
| **Write Capacity** | Millions of writes per second | Moderate write speed per document | Bulk ingest / Streaming buffers |


**No, Cloud Bigtable does not store data in JSON documents, and it is not a document database.**

While both Firestore and Bigtable fall under the NoSQL umbrella, they belong to completely different NoSQL categories and handle data structure differently.

---

**1. How Bigtable Stores Data vs. Firestore**

* **Firestore (Document Store):** Stores data as JSON-like documents with named fields (e.g., `{"name": "Dinesh", "city": "Hyderabad", "age": 25}`). It parses and understands the JSON structure natively.
* **Cloud Bigtable (Wide-Column Store):** Stores data as raw **byte arrays** inside a multi-dimensional, sorted map. Data is organized into **Row Key $\rightarrow$ Column Family $\rightarrow$ Column Qualifier $\rightarrow$ Timestamp $\rightarrow$ Value**. It does not interpret your data as JSON objects; to Bigtable, every column value is just a sequence of raw bytes (`bytes[]`).

---

**2. Why the Row Key is Used in Bigtable**

In Bigtable, the **Row Key is the only index in the entire database**.

* **Lexicographical Sorting:** Bigtable sorts every record alphabetically (lexicographically) by its Row Key and splits them into chunks called *tablets* across Google's storage layer.
* **Predictable Single-Digit Latency:** Because there are no secondary indexes, Bigtable knows the exact physical location of a record instantly if you provide the Row Key or a consecutive range of Row Keys (e.g., `CAR_101#2026-08-01` to `CAR_101#2026-08-31`).

---

**3. How a Bigtable Row Key Differs from Passing Data in Firestore**

| Feature | [Firestore](https://github.com/SSSPRABHUDINESH/DevOps_Notes/blob/main/gcp/database-notes.md#4--firestore) | [Cloud Bigtable](https://github.com/SSSPRABHUDINESH/DevOps_Notes/blob/main/gcp/database-notes.md#5--cloud-bigtable) |
| --- | --- | --- |
| **Data Model** | JSON Documents inside Collections | Wide-column rows sorted by Row Key |
| **Indexing** | **Every field** inside the document is automatically indexed. | **Only the Row Key** is indexed. Column values are unindexed bytes. |
| **Querying Capabilities** | Query by any field: `WHERE city == 'Hyderabad' AND age > 20`. | Query strictly by **Row Key** lookup or **Row Key range scan**. |
| **Design Pattern** | Pass natural JSON payloads; query fields freely. | You must pack query criteria directly into the Row Key string (e.g., `DEVICE_ID#TIMESTAMP`). |

In Firestore, you can drop in a user profile document and ask the database to find users by `city` or `age`. In Bigtable, if you want to find telemetry from a specific device at a specific time, you must construct a composite Row Key like `SENSOR_99#20260827_190000` because the database cannot query the values inside the columns without scanning petabytes of raw data.

### 5.4 Under the Hood: Persistence & Durability

* **Separation of Compute and Storage:** Bigtable compute nodes don't hold the data. The data lives on Google's massive shared file system (Colossus).


* **Instant Scaling:** When you add more compute nodes, Bigtable updates pointer addresses instantly without having to copy or move physical files around.



### 5.5 GCP Architecture & Provisioning

* **Storage Type:** Choose **SSD** for low-latency production applications, or **HDD** for low-cost archives over 10TB.



```text
┌────────────────────────────────────────────────────────────────────────┐
│                        Bigtable Instance                               │
│                                                                        │
│   [ Compute Node 1 ]   [ Compute Node 2 ]   [ Compute Node 3 ]         │
│          │                    │                    │                   │
│   ═══════╪════════════════════╪════════════════════╪════════════════   │
│          ▼                    ▼                    ▼                   │
│   [ Data Tablet A ]    [ Data Tablet B ]    [ Data Tablet C ]          │
│         (Stored across Google's Colossus Distributed Disks)            │
└────────────────────────────────────────────────────────────────────────┘

```

### 5.6 Sample Structure & Example

```text
Row Key: "CAR_102#TIMESTAMP_1774684800"
-------------------------------------------------------------------------
Column Family: sensor_readings
  sensor_readings:speed       -> 65 MPH   (Timestamp: 1774684800)
  sensor_readings:tire_psi    -> 32 PSI   (Timestamp: 1774684800)
-------------------------------------------------------------------------
Column Family: vehicle_info
  vehicle_info:driver_id      -> "D_4401" (Timestamp: 1774684800)

```

---

## 6. 📊 BigQuery

### 6.1 Overview & Core Purpose

* **What is it?** BigQuery is Google's fully managed, serverless cloud data warehouse for analytical reporting (OLAP).


* **What does it do?** It allows you to run SQL queries across massive datasets (gigabytes to petabytes) in seconds to generate business insights, dashboards, and reports.


* **Core Rule:** BigQuery is for analysis, not for daily transactional application buttons.



### 6.2 Caching Strategy & Access Pattern

* **Query Cache:** If you re-run the exact same query within 24 hours, BigQuery returns the results instantly for free from its cache.


* **BI Engine:** A built-in RAM acceleration layer that makes business dashboards (like Looker or Tableau) load sub-second.


* **Column Scans:** BigQuery only scans and bills for the columns you ask for in your `SELECT` query.

### Architecture diagram:

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 1. DATA SOURCES & INGESTION (ETL)                                      │
│                                                                                                        │
│  [ OLTP DBs / Cloud Storage / Streams / APIs ] ──► [ Cloud Dataflow / Dataproc (Transform) ]           │
└────────────────────────────────────────┬───────────────────────────────────────────────────────────────┘
                                         │
                               (Batch Load / Storage API)
                                         │
                                         ▼ (Data is loaded directly into storage)
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                               2. BIGQUERY STORAGE LAYER (Persistent)                                   │
│                                                                                                        │
│    [ Colossus Distributed File System ]  ──► (Stores data in highly compressed Capacitor columns)      │
└────────────────────────────────────────┬───────────────────────────────────────────────────────────────┘
                                         │
                                         ▲ (Data is accessed ONLY when a query is executed)
                                         │
═════════════════════════════════════════╪════════════════════════════════════════════════════════════════
                 JUPITER PETABIT NETWORK │ (Lightning-fast bridge between Storage and Compute)
═════════════════════════════════════════╪════════════════════════════════════════════════════════════════
                                         │
                                         ▼
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                     3. BIGQUERY COMPUTE LAYER (Activates on User Request)                              │
│                                                                                                        │
│    [ Leaf Nodes (Slots) ]       ◄── (Massive parallel workers read columns from Colossus)              │
│              ▲                                                                                         │
│              │                                                                                         │
│    [ Intermediate Nodes ]       ◄── (Aggregates the sub-query results)                                 │
│              ▲                                                                                         │
│              │                                                                                         │
│    [ Dremel Root Node ]         ◄── (Receives query, builds execution tree, returns final answer)      │
└──────────────▲─────────────────────────────────────────────────────────────────────────────────────────┘
               │
               │ (Action: Submits SQL Query)
               │
[ 👤 User / Data Analyst / BI Dashboard ]

```

### 6.3 Storage Hierarchy & Comparisons

* **Data Format:** Columnar (Capacitor format)—stores table data column by column instead of row by row.


| Feature | BigQuery (OLAP) | Cloud SQL / AlloyDB (OLTP) | Bigtable (NoSQL) |
| :--- | :--- | :--- | :--- |
| **How Data is Stored** | **Columns** (Great for math/aggregations) | **Rows** (Great for single user records) | **Wide Rows** (Sorted by key) |
| **Main Action** | Scan millions of rows for big trends | Quick Insert, Update, Delete | Super-fast streaming write ingestion |
| **Scale** | Petabytes / Exabytes | Gigabytes to Terabytes | Petabytes |

### 6.4 Under the Hood: Persistence & Durability

* **Google's Jupiter Petabit Network:** Jupiter is Google's internal data center network infrastructure, capable of delivering over 1 Petabit per second of total bandwidth. In BigQuery, it acts as the lightning-fast bridge that connects the compute servers with the storage servers, allowing massive amounts of data to move between them instantly without network bottlenecks.

    *  Google's Jupiter Petabit Network relies on massive, physical fiber-optic cables and specialized hardware running inside and between Google’s data centers.

```
    +------------------------+                     +-----------------------+
    
    |  STORAGE SERVERS       |   ================> |    COMPUTE SERVERS    |
    | (Colossus File System) |    JUPITER NETWORK  |   (Dremel Engine)     |
    | Holds your CSV/tables  |   [Physical Fiber]  | Runs your SQL queries |
    +------------------------+                     +-----------------------+
```

* **Colossus Storage vs. Persistent Disk (PD):** [Colossus](https://github.com/SSSPRABHUDINESH/DevOps_Notes/blob/main/gcp/database-notes.md#64-under-the-hood-persistence--durability) is Google's global, distributed file system (the successor to the Google File System). 
     *  It is **not** normal Persistent Disk (PD) storage. PDs are standard attached block storage used for virtual machines and transactional databases (like Cloud SQL). 
     *  Colossus is a massive, cluster-level storage system that spreads data in chunks across thousands of disks in Google's data centers, providing built-in replication, extreme fault tolerance, and petabyte scale.
     *  It is a distributed file system.

## Google Distributed File systems:

## 📂 Google File System (GFS) Architecture
The original Google File System (GFS) relied on a centralized topology. A single Master Node managed all of the namespace metadata and mapped file requests to various Chunk Servers, which stored the physical data replicas across raw Linux file systems.

<img src="data:image/jpeg;base64,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"/><img width="435" height="348" alt="image" src="https://github.com/user-attachments/assets/c60bf45d-740a-416a-bacd-21c9ebd0f76f" />

------------------------------
## 🚀 Google Colossus Architecture
In contrast, Colossus decouples and distributes the metadata layer entirely to eliminate the master bottleneck. 
   -  It uses a distributed ring of front-end servers called Curators that read and write metadata structures directly into a scalable [Google Bigtable](https://cloud.google.com/bigtable) database. 
   -  The actual file blocks are dynamically written across massive pools of network-attached Data Servers (D-Servers).

<img src="https://www.systemdesignhandbook.com/wp-content/uploads/2025/03/architecture-of-colossus.svg" alt="Exploring Distributed File Systems"/>



* **Dremel Execution Engine:** Dremel is BigQuery's massive, distributed query engine. While Spanner and Cloud SQL have engines optimized for single-record transactions, Dremel is exclusively built for complex analytical queries across massive datasets. When you run a query, Dremel builds an execution tree and divides the work into thousands of tiny subtasks. These tasks are assigned to worker units called "Slots" (leaf nodes) that read the data in parallel, apply filters, and pass the results back up the tree to be aggregated.

   * **Dremel Engine & Slots:** When you submit a SQL query, BigQuery dynamically breaks it down and spins up thousands of worker units called **Slots** to scan data in parallel.

* **INSERT Statements in BigQuery:** BigQuery is an analytical database, but it fully supports standard SQL (including Data Manipulation Language or DML) and stores data in tables. You *can* run a single `INSERT` statement, but doing it row-by-row (like you would in Cloud SQL) is highly discouraged. Because BigQuery stores data in compressed columnar blocks (Capacitor format) rather than individual rows, single-row inserts are inefficient. BigQuery is designed to ingest thousands or millions of records at once via batch loading or streaming APIs.

* **Data Warehouse:** A data warehouse is a centralized repository that stores massive amounts of historical data aggregated from various sources across a business. Unlike transactional databases that handle real-time app interactions, a warehouse is specifically optimized for Online Analytical Processing (OLAP)—running heavy queries to generate reports, dashboards, and business intelligence (BI) insights.

* **ETL (Extract, Transform, Load):** ETL is the automated pipeline process used to move data into a warehouse. It involves **Extracting** data from live transactional databases (like Cloud SQL or external APIs), **Transforming** it (cleaning, formatting, or summarizing the data so it makes sense for reporting), and **Loading** it into a data warehouse like BigQuery in large batches.


* **Partitioning & Clustering:**
     * **Partitioning:** Slices your big table by date (e.g., one partition per day) so you only scan relevant dates.


     * **Clustering:** Groups related data together by column values (e.g., sorting by `country`) to avoid scanning the whole partition.





### 6.5 GCP Architecture & Provisioning

* **Hierarchy:** Google Cloud Project $\rightarrow$ **Dataset** (Folder / Region boundary) $\rightarrow$ **Tables / Views**.


* **Pricing Models:** **On-Demand** (Pay for TB scanned) or **Capacity Editions** (Pay for reserved compute slots).



```text
┌────────────────────────────────────────────────────────────────────────┐
│                        BigQuery Serverless Cloud                       │
│                                                                        │
│   [ SQL Query / BI Tool ] ──► [ Dremel Execution Engine (Slots) ]      │
│                                              │                         │
│   ═══════════════════════════════════════════╪══════════════════════   │
│             Jupiter High-Speed Network (Petabit Bandwidth)             │
│   ═══════════════════════════════════════════╪══════════════════════   │
│                                              ▼                         │
│            Colossus Storage (Capacitor Columnar Compressed Files)      │
└────────────────────────────────────────────────────────────────────────┘

```

### 6.6 Sample Structure & Example

```sql
-- Create a Date-Partitioned Table
CREATE TABLE `my_project.sales_dw.orders_report`
(
    order_id STRING,
    customer_id INT64,
    order_date DATE,
    amount NUMERIC,
    country STRING
)
PARTITION BY order_date
CLUSTER BY country;

-- Run analytics across 5 years of historical sales data
SELECT 
    country,
    COUNT(DISTINCT customer_id) AS total_customers,
    SUM(amount) AS total_revenue
FROM `my_project.sales_dw.orders_report`
WHERE order_date BETWEEN '2026-01-01' AND '2026-12-31'
GROUP BY country;

```

---

## 7. 🎯 Production Engineering Scenarios & Interview Playbook

### Scenario 1: Moving an Application Database to Cloud SQL

**Structured 6-Pillar Answer:**

1. **Engine Compatibility:** Confirm target database engine, version, and necessary plugins/extensions (e.g., PostGIS).


2. **Sizing & Capacity:** Calculate required CPU, RAM, and initial disk space based on peak concurrent user connections.


3. **Private Connectivity:** Provision Private Services Access (PSA) so the application connects over secure private IPs.


4. **Security:** Use Cloud SQL Auth Proxy, enforce IAM authentication, and keep passwords safe in Secret Manager.


5. **HA & Disaster Recovery:** Turn on High Availability (regional failover) and enable automated backups with Point-in-Time Recovery.


6. **Migration & Cutover:** Use Google Database Migration Service (DMS) for continuous replication (CDC), verify data consistency, and switch application traffic during a planned maintenance window.



---

### Scenario 2: Your Database is Slow (Bottleneck Troubleshooting)

```text
                     YOUR DATABASE IS SLOW: WHAT TO CHECK?
                                      │
          ┌───────────────────────────┼───────────────────────────┐
          ▼                           ▼                           ▼
  1. Too Many Reads           2. Slow SQL Queries         3. Connection Limit Hit
          │                           │                           │
  Add Read Replicas or        Open Query Insights,        Add a Connection Pooler
  add Memorystore (Redis)     add missing indexes         (e.g., PgBouncer)

```

---

### Scenario 3: Cloud SQL Primary Instance Crashes

* **What happens?** The Google Cloud control plane detects the lost heartbeat and triggers automatic failover.


* **How it recovers:** The Standby instance in Zone B mounts the shared persistent storage and assumes the primary role.


* **Application impact:** The database IP automatically redirects to the new primary; apps reconnect in under 60 seconds.



---

### Scenario 4: An Engineer Accidentally Deletes a Production Table

* **How to fix:** You cannot undo a raw `DROP TABLE` command directly on the live database.
* **Recovery process:** Use **Point-in-Time Recovery (PITR)** to restore a clone of the database to the exact minute before the command ran. Export the deleted data from the clone and import it back into your live production table.



---

### Scenario 5: Connecting GKE Pods to Cloud SQL Securely

* **No Hardcoded Passwords:** Never write passwords in Kubernetes YAML files.


* **Workload Identity:** Link your Kubernetes Service Account (KSA) to a Google Service Account (GSA) with Cloud SQL Client permissions.


* **Auth Proxy Sidecar:** Run the Cloud SQL Auth Proxy alongside your application container in the pod to handle secure mTLS connections automatically.


* **Secret Manager:** Fetch database passwords dynamically at pod startup using the Secret Manager CSI Driver.
