# Database Fundamentals — AlloyDB Omni Preparation

> **Track:** New AlloyDB Omni + Kubernetes + CSI Project  
> **Chapter:** 1 — Database Fundamentals  
> **Status:** ✅ Complete  
> **Focus:** Practical fundamentals required for AlloyDB Omni, VLDB, storage, backup/restore, HA/DR and CSI discussions.

---

## 1. What is a Database?

A **database** is a system used to store, organize, retrieve and manage data.

Example:

```text
Bank Database
│
├── Customers
├── Accounts
├── Transactions
├── Loans
└── Payments
```

A database provides mechanisms for:

- Storing data
- Searching and retrieving data
- Updating data
- Maintaining consistency
- Supporting multiple users
- Recovering after failures

---

## 2. What is a DBMS?

**DBMS = Database Management System**

It is the software that manages databases.

Examples:

- PostgreSQL
- MySQL
- Oracle
- SQL Server
- MongoDB

For this project:

```text
PostgreSQL
    ↓
Database Management System
```

**AlloyDB Omni is based on PostgreSQL**, so PostgreSQL fundamentals are directly relevant.

---

## 3. Database vs Table

A **database** is a logical container.

A **table** stores structured data inside the database.

```text
PostgreSQL
│
├── Database: banking
│   │
│   ├── customers
│   ├── accounts
│   └── transactions
│
└── Database: reporting
    │
    ├── reports
    └── summaries
```

### Mental model

```text
PostgreSQL
    ↓
Database
    ↓
Schema
    ↓
Table
    ↓
Rows + Columns
```

---

## 4. Rows and Columns

### Row

A **row represents one record**.

```text
id | name   | city
---|--------|----------
1  | Dinesh | Hyderabad
```

One customer = one row.

### Column

A **column represents an attribute/property**.

```text
Customer
│
├── ID
├── Name
├── City
└── Phone
```

---

## 5. What is a Schema?

A **schema** is a logical namespace/container for database objects such as:

- Tables
- Views
- Functions
- Other database objects

Example:

```text
Database
│
├── public
│   ├── customers
│   └── accounts
│
└── reporting
    ├── monthly_sales
    └── yearly_sales
```

> **Database ≠ Schema ≠ Table**

---

## 6. What is SQL?

**SQL = Structured Query Language**

It is used to interact with relational databases.

### SELECT

```sql
SELECT * FROM customers;
```

Retrieve records from a table.

### INSERT

```sql
INSERT INTO customers
VALUES (1, 'Dinesh', 'Hyderabad');
```

Add a record.

### UPDATE

```sql
UPDATE customers
SET city = 'Vizag'
WHERE id = 1;
```

Modify a record.

### DELETE

```sql
DELETE FROM customers
WHERE id = 1;
```

Remove a record.

> For this project, deep SQL knowledge is **not required**. Basic understanding is enough.

---

## 7. Primary Key

A **primary key uniquely identifies a row**.

```text
customer_id | name
------------|-------
101         | Dinesh
102         | Ravi
103         | Kumar
```

Here, `customer_id` can be the primary key.

```text
customer_id = 101
        ↓
Uniquely identifies Dinesh's record
```

---

## 8. Index

An **index helps the database find data faster**.

Think of a book:

```text
Without index:
Read page → page → page → ... → find topic

With index:
Index → directly find relevant page
```

Database:

```text
Table
  ↓
10 million rows
  ↓
Index
  ↓
Faster lookup
```

Indexes also consume storage and have maintenance/write overhead.

For this project, understanding the **purpose** of indexes is enough.

---

## 9. Transactions

A **transaction is a logical unit of database work**.

Example: transferring ₹10,000:

```text
Account A
   ↓
- ₹10,000

Account B
   ↓
+ ₹10,000
```

These operations should logically happen together.

We should not end up with:

```text
A = -₹10,000
B = unchanged
```

A transaction helps maintain the correctness of the operation.

---

## 10. ACID

ACID describes important transaction properties.

| Property | Meaning | Simple explanation |
|---|---|---|
| **A — Atomicity** | All or nothing | A transaction fully succeeds or fails |
| **C — Consistency** | Valid state | Data remains within defined rules |
| **I — Isolation** | Concurrent operations | Transactions don't improperly interfere |
| **D — Durability** | Survives failures | Committed data should survive a crash |

### Why Durability matters for this project

Eventually we need to answer:

> **Where does committed database data actually live, and how do we protect it?**

That leads directly to:

```text
Durability
    ↓
Data + WAL
    ↓
Persistent Storage
    ↓
Backup / Snapshot / Replication
```

---

# 11. Where is Database Data Actually Stored?

Conceptually:

```text
AlloyDB Omni
     │
     ▼
PostgreSQL Database
     │
     ▼
Data Files
     │
     ▼
Persistent Storage
```

The database uses physical storage for things such as:

- Tables
- Indexes
- System catalogs
- Other database structures

PostgreSQL also maintains **WAL (Write-Ahead Log)**.

```text
                 PostgreSQL
                      │
             ┌────────┴────────┐
             ▼                 ▼
         Data Files           WAL
             │                 │
             └────────┬────────┘
                      ▼
               Persistent Storage
```

> **WAL will be studied in depth in a later chapter.**

---

# 12. Database Storage vs Backup

These are **not the same thing**.

### Live database storage

```text
AlloyDB
   ↓
Data Files + WAL
   ↓
Persistent Storage
```

This is where the live database operates.

### Backup

```text
Live Database
      ↓
Backup Mechanism
      ↓
Backup Repository
```

A backup is a **protection/recovery copy**, not the primary live database storage.

---

# 13. Why Storage Matters for VLDB

Consider:

```text
Database = 10 TB
```

The live database needs persistent storage:

```text
10 TB Database
      ↓
Persistent Storage
```

A traditional backup may involve reading/copying a large amount of data:

```text
Database
   ↓
Read / Copy
   ↓
Backup
   ↓
Object Storage
```

For very large databases, this can take significant time and consume resources.

The new project investigates whether storage-native operations can improve this:

```text
Database
   ↓
Persistent Volume
   ↓
Storage Platform
   ↓
Snapshot
   ↓
Fast protection / clone / restore workflows
```

This is one of the key reasons database fundamentals matter for the new project.

---

# 14. Database Layer vs Storage Layer

This distinction is critical.

```text
┌─────────────────────────────────────┐
│          AlloyDB Omni               │
│          DATABASE LAYER             │
│                                     │
│  PostgreSQL engine                  │
│  Tables                             │
│  Transactions                       │
│  WAL                                │
│  Queries                            │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│          Kubernetes                 │
│          PLATFORM                   │
│                                     │
│  Pod / PVC / StorageClass            │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│              CSI                    │
│       STORAGE INTERFACE             │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│       Enterprise / Cloud Storage    │
│                                     │
│  NetApp / Pure / Hyperdisk / etc.   │
└─────────────────────────────────────┘
```

### Remember:

| Component | Role |
|---|---|
| **AlloyDB Omni** | Database |
| **Kubernetes** | Container orchestration platform |
| **PVC** | Request for persistent storage |
| **CSI** | Standard interface between Kubernetes and storage |
| **NetApp / Pure** | Enterprise storage platforms |
| **Hyperdisk** | Google Cloud-native block storage |

---

# 15. How This Connects to the New Project

The new project is moving toward:

```text
                  AlloyDB Omni
                       │
                  Kubernetes
                       │
                      PVC
                       │
                      CSI
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     NetApp           Pure         Hyperdisk
        │              │              │
        └──────────────┼──────────────┘
                       │
               Persistent Storage
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Snapshot       Clone     Replication
          │            │            │
          ▼            ▼            ▼
       Backup         Test          DR
```

The project asks whether these storage capabilities can make **VLDB day-2 operations** more efficient.

---

# 16. What You Should NOT Deep-Dive Into Yet

For this project, don't spend significant time on:

- Advanced SQL
- Complex JOINs
- Query planner internals
- Advanced indexing
- Normalization theory
- Stored procedures
- Advanced PostgreSQL tuning

These may matter for a DBA role, but they are **not the critical path for this project**.

---

# 17. Interview Quick Revision

### Q: What is AlloyDB Omni?

> AlloyDB Omni is Google's PostgreSQL-compatible database software designed to run in environments such as on-premises and other infrastructure outside the managed AlloyDB service.

### Q: What is a database?

> A system for storing, organizing, retrieving and managing data.

### Q: Database vs table?

> A database is a logical container; a table stores structured records inside it.

### Q: What is a schema?

> A logical namespace for database objects such as tables and functions.

### Q: What is a transaction?

> A logical unit of database work that should be treated as one operation.

### Q: What is ACID?

> Atomicity, Consistency, Isolation and Durability — properties that help maintain reliable transactions.

### Q: What is a primary key?

> A column or set of columns that uniquely identifies a row.

### Q: What is an index?

> A data structure that helps the database find rows more efficiently.

### Q: Where does database data live?

> PostgreSQL stores database information in data files and maintains WAL; these ultimately reside on persistent storage.

### Q: Is database storage the same as backup?

> No. Database storage is where the live database operates; backup is a separate recovery/protection mechanism.

### Q: Why is storage important for VLDB?

> Very large databases involve huge amounts of data, so copying and moving that data for backup, restore or DR can be expensive and time-consuming.

### Q: Why are storage snapshots interesting?

> They can potentially provide faster ways to capture, clone or restore large volumes without performing a traditional full data copy every time.

---

# 18. Chapter Completion Checklist

- [x] Database and DBMS
- [x] PostgreSQL and AlloyDB Omni relationship
- [x] Database / schema / table hierarchy
- [x] Rows and columns
- [x] SQL basics
- [x] Primary keys
- [x] Indexes
- [x] Transactions
- [x] ACID
- [x] Basic database storage
- [x] Data files
- [x] WAL introduction
- [x] Database storage vs backup
- [x] VLDB storage problem
- [x] Database vs storage layer
- [x] Connection to the new CSI project

---

## 🎯 Chapter 1 Status

**DATABASE FUNDAMENTALS — ✅ COMPLETE**

### Next Chapter

# Chapter 2 — PostgreSQL Architecture

We will learn:

```text
PostgreSQL Server
       │
       ├── Client Connections
       ├── Backend Processes
       ├── Shared Memory
       ├── Background Processes
       ├── Data Directory
       ├── WAL
       └── Storage
```

The goal is to understand **what happens inside PostgreSQL when a client sends a request and how that eventually relates to storage**.

> **Depth target:** Interview-ready understanding, not PostgreSQL DBA-level internals.
