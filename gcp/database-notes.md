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

* 🛑 **100GB in a Single Transaction ➔ OLAP**
* **Why:** This is a massive bulk load. Analytical DBs (like BigQuery) are built to swallow giant chunks at once. Trying this on an OLTP database will crash it, lock all tables, or exhaust the transaction log.


* 🚀 **1KB per Transaction (10,000 times) ➔ OLTP**
* **Why:** Fast, tiny, highly concurrent operations. This mimics a live web app (like an e-commerce site) processing thousands of simultaneous user actions.



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



### 6.3 Storage Hierarchy & Comparisons

* **Data Format:** Columnar (Capacitor format)—stores table data column by column instead of row by row.


| Feature | BigQuery (OLAP) | Cloud SQL / AlloyDB (OLTP) | Bigtable (NoSQL) |
| :--- | :--- | :--- | :--- |
| **How Data is Stored** | **Columns** (Great for math/aggregations) | **Rows** (Great for single user records) | **Wide Rows** (Sorted by key) |
| **Main Action** | Scan millions of rows for big trends | Quick Insert, Update, Delete | Super-fast streaming write ingestion |
| **Scale** | Petabytes / Exabytes | Gigabytes to Terabytes | Petabytes |

### 6.4 Under the Hood: Persistence & Durability

* **Dremel Engine & Slots:** When you submit a SQL query, BigQuery dynamically breaks it down and spins up thousands of worker units called **Slots** to scan data in parallel.


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
