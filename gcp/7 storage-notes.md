# Google Cloud Storage — Complete LevelUp Notes

> **Purpose:** Source-of-truth notes for Google Cloud Storage preparation for EPAM LevelUp (A2 → A3).
>
> These notes consolidate the concepts, architecture diagrams, mental models, commands, doubts, security concepts, troubleshooting scenarios, and interview points covered during preparation.

---

# Table of Contents

1. [Cloud Storage Fundamentals](#1-cloud-storage-fundamentals)
2. [Buckets](#2-buckets)
3. [Objects](#3-objects)
4. [Object Storage vs File System](#4-object-storage-vs-file-system)
5. [Bucket Locations](#5-bucket-locations)
6. [Storage Classes](#6-storage-classes)
7. [Lifecycle Management](#7-lifecycle-management)
8. [Object Versioning](#8-object-versioning)
9. [Soft Delete](#9-soft-delete)
10. [Retention Policies](#10-retention-policies)
11. [Object Holds](#11-object-holds)
12. [IAM and Cloud Storage Access Control](#12-iam-and-cloud-storage-access-control)
13. [Uniform Bucket-Level Access](#13-uniform-bucket-level-access)
14. [ACLs](#14-acls)
15. [Public Access Prevention](#15-public-access-prevention)
16. [Signed URLs](#16-signed-urls)
17. [Signed Policy Documents](#17-signed-policy-documents)
18. [Signed URL vs Signed Policy Document](#18-signed-url-vs-signed-policy-document)
19. [Cloud Storage Authentication](#19-cloud-storage-authentication)
20. [Service Accounts and Application Default Credentials](#20-service-accounts-and-application-default-credentials)
21. [Metadata Server and Cloud Storage Access](#21-metadata-server-and-cloud-storage-access)
22. [Private Google Access](#22-private-google-access)
23. [Cloud Storage Encryption Fundamentals](#23-cloud-storage-encryption-fundamentals)
24. [Encryption at Rest vs In Transit](#24-encryption-at-rest-vs-in-transit)
25. [Google-Managed Encryption](#25-google-managed-encryption)
26. [CMEK](#26-cmek)
27. [Cloud KMS](#27-cloud-kms)
28. [Key Rotation](#28-key-rotation)
29. [Key Compromise and Revocation](#29-key-compromise-and-revocation)
30. [CSEK](#30-csek)
31. [Client-Side Encryption](#31-client-side-encryption)
32. [Encryption Comparison](#32-encryption-comparison)
33. [Storage Security Architecture](#33-storage-security-architecture)
34. [Data Protection and Recovery](#34-data-protection-and-recovery)
35. [RPO and RTO](#35-rpo-and-rto)
36. [Backup and DR with Cloud Storage](#36-backup-and-dr-with-cloud-storage)
37. [Replication Concepts](#37-replication-concepts)
38. [Performance and Scalability](#38-performance-and-scalability)
39. [Cloud Storage Cost Model](#39-cloud-storage-cost-model)
40. [Cost Optimization](#40-cost-optimization)
41. [Capacity Planning](#41-capacity-planning)
42. [Storage Transfer and Migration](#42-storage-transfer-and-migration)
43. [Cloud Storage + GCE Integration](#43-cloud-storage--gce-integration)
44. [Cloud Storage + GKE Integration](#44-cloud-storage--gke-integration)
45. [Cloud Storage + Application Architecture](#45-cloud-storage--application-architecture)
46. [Direct Upload Architecture](#46-direct-upload-architecture)
47. [Direct Download Architecture](#47-direct-download-architecture)
48. [Cloud Storage Commands](#48-cloud-storage-commands)
49. [Lifecycle Commands](#49-lifecycle-commands)
50. [Versioning and Soft Delete Commands](#50-versioning-and-soft-delete-commands)
51. [IAM Commands](#51-iam-commands)
52. [CMEK Commands](#52-cmek-commands)
53. [Signed URL Commands](#53-signed-url-commands)
54. [Signed Policy Document Flow](#54-signed-policy-document-flow)
55. [Troubleshooting Framework](#55-troubleshooting-framework)
56. [403 Troubleshooting](#56-403-troubleshooting)
57. [404 Troubleshooting](#57-404-troubleshooting)
58. [Signed URL Troubleshooting](#58-signed-url-troubleshooting)
59. [CMEK Troubleshooting](#59-cmek-troubleshooting)
60. [Storage Security Checklist](#60-storage-security-checklist)
61. [Production Architecture](#61-production-architecture)
62. [Important Comparisons](#62-important-comparisons)
63. [Common Misconceptions](#63-common-misconceptions)
64. [LevelUp Interview Questions](#64-levelup-interview-questions)
65. [Scenario-Based Interview Questions](#65-scenario-based-interview-questions)
66. [Final Storage Mental Map](#66-final-storage-mental-map)
67. [Final Storage Cheat Sheet](#67-final-storage-cheat-sheet)

---

# 1. Cloud Storage Fundamentals

Google Cloud Storage (GCS) is a managed **object storage service** used to store and retrieve data as objects.

Common use cases:

- Backups
- Logs
- Images
- Videos
- Documents
- Application files
- Database dumps
- Data lakes
- Static content
- Archive data

Basic architecture:

```text
                Google Cloud Storage
                         |
                +--------+--------+
                |                 |
             Bucket             Bucket
                |                 |
          +-----+-----+       +---+---+
          |     |     |       |       |
        Obj1  Obj2  Obj3     Obj1    Obj2
```

---

# 2. Buckets

A bucket is a logical container used to store objects.

Example:

```text
gs://my-production-data
```

Inside:

```text
gs://my-production-data/
    customers/001.json
    customers/002.json
    reports/report1.pdf
    backups/db.sql
```

Bucket-level properties can include:

- Location
- Storage class/default storage class
- IAM configuration
- Uniform Bucket-Level Access
- Public Access Prevention
- Lifecycle configuration
- Retention policy
- Versioning
- Soft delete
- Encryption configuration

---

# 3. Objects

An object is the actual data stored in Cloud Storage.

Conceptually:

```text
Object
 |
 +-- Data
 |
 +-- Object name
 |
 +-- Metadata
 |
 +-- Generation/version information
 |
 +-- Storage class
 |
 +-- Encryption information
```

Example:

```text
Bucket:
gs://production-data

Object:
backups/postgres-2026-08-09.sql
```

The object name is:

```text
backups/postgres-2026-08-09.sql
```

---

# 4. Object Storage vs File System

## Traditional file system

```text
/
├── home/
├── var/
├── etc/
└── data/
```

Directories are actual filesystem structures.

## Cloud Storage

```text
Bucket
 |
 +-- reports/2026/january.pdf
 +-- reports/2026/february.pdf
 +-- backups/db.sql
```

The `/` characters are part of the object name/prefix.

Conceptually:

```text
Object name:
reports/2026/january.pdf
```

There does not need to be a traditional POSIX directory underneath.

> Cloud Storage is object storage, not a traditional filesystem.

---

# 5. Bucket Locations

Cloud Storage supports:

- Region
- Dual-region
- Multi-region

## Region

Data is stored in a specific Google Cloud region.

Example:

```text
asia-south1
```

Useful when:

- Application is concentrated in one region
- Data residency requirements exist
- You want predictable locality

## Dual-region

Data is designed for geographic redundancy across two regions.

Useful for:

- Higher availability
- Regional failure protection
- DR requirements

## Multi-region

Data is distributed across a broader geographic area.

Useful for:

- Globally distributed access
- Availability
- Large-scale applications

---

# 6. Storage Classes

Main Cloud Storage classes:

```text
STANDARD
NEARLINE
COLDLINE
ARCHIVE
```

Conceptual model:

```text
                    Frequent Access
                         |
                         v
                     STANDARD
                         |
                         v
                     NEARLINE
                         |
                         v
                     COLDLINE
                         |
                         v
                     ARCHIVE
                         |
                         v
                    Rare Access
```

## Standard

For frequently accessed data.

Examples:

- Active application content
- Frequently accessed images
- Hot datasets

## Nearline

For data accessed less frequently.

Examples:

- Monthly backups
- Occasionally accessed data
- Some logs

## Coldline

For colder data.

Examples:

- Disaster recovery backups
- Older data
- Infrequently accessed datasets

## Archive

For very rarely accessed long-term data.

Examples:

- Compliance archives
- Long-term backups
- Historical records

---

# 7. Storage Class — Important Points

Storage class is associated with the object.

A bucket can have a default storage class.

New objects generally inherit the bucket's default unless explicitly specified otherwise.

Changing a bucket's default storage class does **not** automatically change the storage class of existing objects.

Important:

```text
Cheapest storage class
        !=
Cheapest total architecture
```

Always consider:

```text
Storage cost
+
Access frequency
+
Retrieval charges
+
Minimum storage duration
+
Network cost
+
Application requirements
```

---

# 8. Lifecycle Management

Object Lifecycle Management automatically performs actions when objects satisfy configured conditions.

Mental model:

```text
Object
  |
  v
Condition
  |
  +---- Age > 30 days
  |
  v
Action
  |
  +---- Change storage class
  |
  +---- Delete
```

Common actions:

- Delete
- SetStorageClass
- Abort incomplete multipart uploads where applicable

---

# 9. Lifecycle Example

```text
Day 0
  |
  v
STANDARD
  |
  | 30 days
  v
COLDLINE
  |
  | 180 days
  v
ARCHIVE
  |
  | Retention requirement fulfilled
  v
DELETE
```

Use cases:

- Backup management
- Log retention
- Cost optimization
- Automatic cleanup
- Noncurrent version management

---

# 10. Lifecycle vs Retention

Very important:

```text
Lifecycle
    |
    +-- "When condition X occurs, perform action Y"

Retention
    |
    +-- "Do not delete before period X"
```

Example:

```text
Lifecycle:
Object age > 30 days
    |
    +-- Set storage class to Coldline


Retention:
7 years
    |
    +-- Object cannot be deleted before retention requirement is fulfilled
```

They solve different problems.

---

# 11. Lifecycle Execution and Concurrent Activity

Suppose:

```text
Lifecycle:
After 30 days -> Coldline
```

At approximately the same time:

```text
User uploads data
User accesses existing object
Lifecycle evaluation occurs
```

Lifecycle rules apply to objects that satisfy the rule conditions.

Important:

- Lifecycle is object-based.
- A newly uploaded object does not become 30 days old merely because the lifecycle process is running.
- An existing object that meets the condition can transition.
- Lifecycle is automatic; it is not a human approval workflow.

If an object is being accessed while its storage class changes, the request is handled according to Cloud Storage's normal semantics. The lifecycle operation does not mean user data is sent to a different customer or lost.

---

# 12. Object Versioning

Object Versioning preserves noncurrent versions of objects.

Example:

```text
file.txt
 |
 +-- Generation 1
 |
 +-- Generation 2
 |
 +-- Generation 3  <-- current
```

If a live object is replaced:

```text
Version 1
   |
   v
Version 2
```

Version 1 can remain as a noncurrent version.

Useful for:

- Accidental overwrite recovery
- Tracking object changes
- Recovery of previous object versions

---

# 13. Versioning and Cost

Versioning can increase storage usage.

Example:

```text
file.txt
  |
  +-- 1 GB version 1
  +-- 1 GB version 2
  +-- 1 GB version 3
  +-- 1 GB version 4
```

Potential retained storage:

```text
4 GB
```

Therefore:

```text
Versioning
    |
    +-- Better recovery
    |
    +-- More storage consumption
```

Lifecycle rules can be used to manage noncurrent versions.

---

# 14. Soft Delete

Soft Delete protects against accidental or malicious deletion.

Conceptual flow:

```text
Object
  |
  | DELETE
  v
Soft-deleted state
  |
  | retention window
  v
Permanent deletion
```

Current Cloud Storage behavior should be checked against the current Google Cloud documentation. The default soft-delete retention period is seven days unless configured otherwise.

Important:

```text
Soft Delete
    |
    +-- Protects against deletion
```

---

# 15. Versioning vs Soft Delete

```text
Versioning
    |
    +-- Keeps noncurrent versions
    +-- Useful for object changes/overwrites
```

```text
Soft Delete
    |
    +-- Keeps deleted resources temporarily
    +-- Useful for accidental/malicious deletion recovery
```

If the primary requirement is deletion recovery, Soft Delete is a key mechanism. Versioning is primarily about retaining object generations.

---

# 16. Retention Policies

A retention policy prevents objects from being deleted before a specified retention period is fulfilled.

Example:

```text
Retention = 7 years
```

Then:

```text
Object created
    |
    |
    |---- 1 year ----
    |
    |---- 5 years ----
    |
    |---- 7 years ----
    |
    v
Retention fulfilled
```

Before retention is fulfilled:

```text
DELETE
  |
  v
BLOCKED
```

Even if an IAM principal has delete permission, retention can prevent deletion.

---

# 17. Retention vs Lifecycle

Example:

```text
Lifecycle:
After 30 days -> Coldline

Retention:
Keep object for 7 years
```

These can coexist.

Think:

```text
Lifecycle = automated management
Retention = protection/compliance
```

---

# 18. Object Holds

Object holds provide object-specific protection against deletion/replacement under applicable retention controls.

Conceptual model:

```text
Object
  |
  +-- Temporary Hold
  |
  +-- Event-based Hold
```

Useful when:

- Legal review is pending
- Event-driven retention is needed
- Object must not be deleted yet

Lifecycle deletion can be blocked while a hold is active.

---

# 19. IAM and Cloud Storage Access Control

IAM answers:

> Who can do what?

Mental model:

```text
Principal
    |
    v
IAM Role
    |
    v
Permissions
    |
    v
Cloud Storage Resource
```

Example:

```text
Application Service Account
        |
        v
Storage Object Viewer
        |
        v
Production Bucket
```

---

# 20. Authentication vs Authorization

This distinction is critical.

## Authentication

```text
WHO ARE YOU?
```

Example:

```text
Service Account
Access Token
```

## Authorization

```text
WHAT ARE YOU ALLOWED TO DO?
```

Example:

```text
Storage Object Viewer
```

Mental model:

```text
Authentication
      |
      v
Identity established
      |
      v
Authorization
      |
      v
IAM checks permissions
      |
      v
ALLOW / DENY
```

---

# 21. Least Privilege

Always grant the minimum permissions required.

Bad:

```text
Application
   |
   v
Storage Admin
```

Better:

```text
Application
   |
   v
Storage Object Viewer
```

if it only needs to read objects.

For a backup application, grant only the permissions required for backup operations.

Do not automatically give:

```text
Storage Admin
```

---

# 22. Uniform Bucket-Level Access (UBLA)

UBLA makes bucket-level IAM the consistent access-control model rather than relying on object ACLs.

Conceptual model:

```text
Bucket
 |
 v
IAM
 |
 +-- User
 +-- Group
 +-- Service Account
```

Without a uniform model, you can have:

```text
IAM
+
ACL
+
Object-specific permissions
```

This can become difficult to reason about.

For modern enterprise designs:

```text
UBLA
+
IAM
+
Least Privilege
```

is generally preferred.

---

# 23. ACLs

ACL = Access Control List.

ACLs can provide access control at the bucket/object level.

Historical model:

```text
Bucket
 |
 +-- ACL
 |
 +-- Object ACL
```

For many modern workloads, IAM + UBLA is simpler.

Use ACLs only when there is a specific requirement that needs them.

---

# 24. Public Access Prevention

Public Access Prevention is a guardrail to prevent public access.

Conceptual model:

```text
Public principal
      |
      v
Public Access Prevention
      |
      X
   BLOCKED
```

Useful for:

- Banking data
- Internal data
- Customer data
- Backups
- Sensitive enterprise data

Important:

```text
Private today
    !=
Protected against accidental public exposure
```

Public Access Prevention provides an additional preventive control.

---

# 25. Public Bucket vs Private Bucket

## Public

```text
Internet
   |
   v
Bucket/Object
```

## Private

```text
Authorized identity
   |
   v
IAM
   |
   v
Bucket/Object
```

A customer does not necessarily need a Google Cloud account to access a private object if you use a suitable signed URL.

---

# 26. Signed URLs

A Signed URL is a web link that gives temporary permission to download or upload a specific file without needing a login or account.

- A signed URL is a URL containing authentication/signature information that gives time-limited access to a Cloud Storage resource.

Typical use cases:

- Temporary downloads
- Temporary uploads
- External customers
- Browser uploads
- Sharing private objects
- Avoiding broad IAM access

Important:

> Anyone who possesses a valid signed URL can use it according to the URL's permissions until it expires or otherwise becomes invalid.

Therefore:

```text
Signed URL
    |
    +-- Treat like a temporary credential
```

---

# 27. Signed URL Mental Model

```text
                         APPLICATION
                              |
                     Generate Signed URL
                              |
                              v
                      +----------------+
                      | Signed URL     |
                      |                |
                      | Object         |
                      | HTTP method    |
                      | Expiration     |
                      | Signature      |
                      +----------------+
                              |
                              v
                           CUSTOMER
                              |
                              v
                       Cloud Storage
                              |
                              v
                       Validate Signature
                              |
                    +---------+---------+
                    |                   |
                  VALID               INVALID
                    |                   |
                    v                   v
                 ALLOW                DENY
```

---

# 28. Signed URL Is Not a Password

A signed URL is not simply:

```text
https://bucket/object?password=abc
```

It contains cryptographic authentication information.

Cloud Storage validates the signature against the request.

Conceptually:

```text
Request information
       +
Signing key
       |
       v
Cryptographic signature
```

Cloud Storage verifies the signature.

---

# 29. Signed URL Does Not Require Recipient IAM

Suppose:

```text
Bucket = private
Customer = no Google account
```

You can create:

```text
Signed URL
```

and provide it to the customer.

The customer can use it while valid.

Therefore:

```text
Customer
   |
   +-- No Google Cloud IAM account required
   |
   v
Signed URL
   |
   v
Private Object
```

---

# 30. Signed URL Sharing

Suppose:

```text
User A receives signed URL
```

User A forwards it to:

```text
User B
```

If the URL is still valid:

```text
User B
   |
   v
Same Signed URL
   |
   v
Cloud Storage
```

It can work.

The URL is not inherently bound to User A.

Therefore:

> Do not treat a signed URL as identity-bound access unless your application adds additional controls around how the URL is issued and used.

---

# 31. Signed URL Supported Operations

Depending on the signed request:

```text
GET
PUT
DELETE
HEAD
```

can be used.

Examples:

```text
GET
  |
  v
Download object
```

```text
PUT
  |
  v
Upload object
```

The HTTP method is part of the signed request.

---

# 32. Signed URL and HTTP Method

Suppose URL was generated for:

```text
GET
```

Then using:

```text
PUT
```

does not magically turn it into an upload URL.

The signed request must match the request being made.

Conceptually:

```text
Signed:
GET + object + headers + expiration

Actual:
PUT + object

Mismatch
   |
   v
DENY
```

---

# 33. Signed URL and Headers

If specific headers are included in the signed request, the actual request must match the signed constraints.

Example:

```text
Signed:
Content-Type = text/plain
```

Actual:

```text
Content-Type = application/pdf
```

Potential signature mismatch.

Therefore:

> Do not modify signed request parameters/headers casually.

---

# 34. Signed URL and Expiration

Example:

```text
URL valid for 10 minutes
```

Timeline:

```text
00:00  URL generated
00:05  User accesses -> ALLOW
00:09  User accesses -> ALLOW
00:10+ User accesses -> EXPIRED
```

Generate a new signed URL if access is required later.

---

# 35. Signed URL Maximum Duration

For the current `gcloud storage sign-url` workflow using system-managed signing, the documented maximum duration is 12 hours.

Always verify the current Cloud SDK documentation when the exact limit matters.

---

# 36. Signed URL Security

Signed URLs are bearer-style credentials.

If:

```text
User A
   |
   v
Signed URL
```

and User A shares it:

```text
User B
   |
   v
Signed URL
```

User B can potentially use it while valid.

Therefore:

- Keep expiration short where possible.
- Use HTTPS.
- Do not log signed URLs carelessly.
- Do not put sensitive signed URLs into public repositories.
- Generate only the required operation.
- Scope to the required object/resource.

---

# 37. Signed URL Architecture — Download

```text
                    CUSTOMER
                        |
                        v
                  Your Application
                        |
                  Authenticate user
                        |
                        v
                Authorize request
                        |
                        v
                Generate Signed URL
                        |
                        v
                     Customer
                        |
                        v
                 Cloud Storage
                        |
                        v
                 Download Object
```

The application does not need to stream the entire file through itself.

---

# 38. Signed URL Architecture — Upload

```text
                    CUSTOMER
                        |
                        v
                  Your Application
                        |
                  Authenticate user
                        |
                        v
                Generate Signed URL
                        |
                        v
                     Customer
                        |
                        v
                 Cloud Storage
                        |
                        v
                    Upload
```

---

# 39. Why Direct Upload Is Useful

Without signed direct upload:

```text
Customer
   |
   v
Application Server
   |
   v
Cloud Storage
```

Application handles the entire file.

With signed URL:

```text
Customer
   |
   +----------------------+
                          |
                          v
                    Cloud Storage
```

Application handles:

- Authentication
- Authorization
- Signed URL generation
- Business logic

Benefits:

- Less application bandwidth
- Less application CPU/memory
- Better scalability
- Better large-file handling

---

# 40. Creating Signed URLs with gcloud

Modern Google Cloud CLI command:

```bash
gcloud storage sign-url gs://BUCKET/OBJECT \
  --duration=10m \
  --impersonate-service-account=SERVICE_ACCOUNT_EMAIL
```

Example:

```bash
gcloud storage sign-url \
  gs://my-bucket/private/report.pdf \
  --duration=10m \
  --impersonate-service-account=signer@my-project.iam.gserviceaccount.com
```

For download, the default HTTP method is generally:

```text
GET
```

---

# 41. Signed Upload URL with gcloud

Example:

```bash
gcloud storage sign-url \
  gs://my-bucket/uploads/file.txt \
  --http-verb=PUT \
  --duration=1h \
  --headers=content-type=text/plain \
  --impersonate-service-account=signer@my-project.iam.gserviceaccount.com
```

Client:

```bash
curl -X PUT \
  -H "Content-Type: text/plain" \
  --data-binary @file.txt \
  "SIGNED_URL"
```

The request must match the constraints that were signed.

---

# 42. Signed URL with a Private Key File

Example:

```bash
gcloud storage sign-url \
  gs://my-bucket/file.txt \
  --duration=1h \
  --private-key-file=service-account-key.json
```

However:

> Prefer service-account impersonation / managed signing where possible rather than distributing long-lived service-account private keys.

---

# 43. Who Can Create a Signed URL?

The signer must have sufficient authority to sign the request.

A service account is commonly used.

A common architecture:

```text
Application
   |
   v
Signer Service Account
   |
   v
IAM permissions
   |
   v
Generate signature
   |
   v
Signed URL
```

For service-account impersonation, the caller needs the appropriate permission to impersonate/sign on behalf of the service account.

Do not assume:

```text
Any Google user
   |
   v
Can generate signed URLs
```

---

# 44. V4 Signed URL

V4 signing is the modern signing process.

Conceptually:

```text
HTTP Request
    |
    +-- Method
    +-- URI
    +-- Headers
    +-- Query parameters
    +-- Expiration
    |
    v
Canonical Request
    |
    v
String-to-Sign
    |
    v
Cryptographic Signature
    |
    v
Signed URL
```

---

# 45. Signed URL and OAuth Token Are Different

Do not confuse:

```text
OAuth 2.0 access token
```

with:

```text
Signed URL signature
```

OAuth tokens are generally used across Google Cloud APIs.

Signed URLs use query-string authentication/signatures for Cloud Storage.

Mental model:

```text
OAuth
    |
    +-- General Google Cloud API authentication

Signed URL
    |
    +-- Temporary URL-based Cloud Storage access
```

---

# 46. Signed Policy Documents

A signed policy document defines conditions that an upload must satisfy when using an HTML form POST to Cloud Storage.

It is primarily designed for:

> Controlled browser/form-based uploads.

---

# 47. Why Signed Policy Documents Exist

Suppose a website allows users to upload images.

You want:

```text
Allowed bucket:
my-upload-bucket

Allowed content type:
image/*

Maximum size:
10 MB

Allowed object prefix:
uploads/
```

A signed URL can authorize a particular signed request.

A signed policy document can express **multiple upload conditions**.

---

# 48. Signed Policy Document Mental Model

```text
                         APPLICATION
                              |
                              v
                    Create Policy Document
                              |
                              v
                  +-----------------------+
                  | expiration            |
                  | bucket                |
                  | object prefix         |
                  | content type          |
                  | content length range  |
                  | other conditions      |
                  +-----------------------+
                              |
                              v
                       Encode Policy
                              |
                              v
                     Sign Policy
                              |
                              v
                        HTML FORM
                              |
                              v
                           USER
                              |
                              v
                    Cloud Storage
                              |
                              v
                  Validate all conditions
                              |
                     +--------+--------+
                     |                 |
                   VALID             INVALID
                     |                 |
                     v                 v
                  Upload             DENY
```

---

# 49. Policy Document Structure

A policy document is JSON and contains:

```json
{
  "expiration": "...",
  "conditions": [
    ...
  ]
}
```

It must be encoded appropriately for the form/signing workflow.

---

# 50. Policy Document Conditions

## Exact matching

```json
{
  "Content-Type": "image/jpeg"
}
```

Meaning:

```text
Only image/jpeg
```

## Starts-with

```json
[
  "starts-with",
  "$key",
  "uploads/"
]
```

Meaning:

```text
Object name must begin with:
uploads/
```

## Content-length-range

```json
[
  "content-length-range",
  0,
  10485760
]
```

Meaning:

```text
Minimum = 0 bytes
Maximum = 10 MB
```

---

# 51. Signed Policy Document Example

Conceptually:

```json
{
  "expiration": "20260809T120000Z",
  "conditions": [
    {
      "bucket": "my-upload-bucket"
    },
    [
      "starts-with",
      "$key",
      "uploads/"
    ],
    {
      "Content-Type": "image/jpeg"
    },
    [
      "content-length-range",
      0,
      10485760
    ]
  ]
}
```

Meaning:

```text
Upload must:
    |
    +-- Go to my-upload-bucket
    +-- Use uploads/ prefix
    +-- Be image/jpeg
    +-- Be <= 10 MB
    +-- Occur before expiration
```

---

# 52. Signed Policy Document Flow

Browser conceptually submits:

```text
key
bucket
Content-Type
policy
signature
file
```

Cloud Storage checks:

```text
Policy
   |
   +-- Expiration
   +-- Bucket
   +-- Key/prefix
   +-- Content type
   +-- Size
   +-- Other conditions
```

If valid:

```text
UPLOAD
```

Otherwise:

```text
DENY
```

---

# 53. Signed URL vs Signed Policy Document

| Feature | Signed URL | Signed Policy Document |
|---|---|---|
| Main use | Temporary object access | Controlled browser/form uploads |
| Download | Yes | No |
| Upload | Yes | Yes |
| Delete | Can be signed | No |
| Multiple upload constraints | Limited | Stronger |
| Content type condition | Can sign headers | Explicit policy condition |
| File size range | Not the main mechanism | Strong |
| Prefix restrictions | Limited | Strong |
| HTML form POST | Not the primary model | Yes |
| Best use | Specific object/action | Flexible upload rules |

Key difference:

```text
SIGNED URL
    |
    +-- "Here is a temporary URL to perform this request."


SIGNED POLICY DOCUMENT
    |
    +-- "Here are the conditions your upload must satisfy."
```

---

# 54. Signed URL vs IAM

```text
IAM
 |
 +-- Identity-based
 +-- Long-term access policy
 +-- User/service account/group
 +-- Resource permissions
```

```text
Signed URL
 |
 +-- Bearer-style
 +-- Temporary
 +-- Specific request/resource
 +-- No recipient IAM account required
```

---

# 55. Signed Policy Document vs IAM

IAM:

```text
Who can access?
```

Policy document:

```text
What upload conditions must the browser satisfy?
```

They can work alongside each other.

---

# 56. Cloud Storage Authentication Architecture

A workload can access Cloud Storage using:

```text
Service Account
```

or:

```text
User credentials
```

or:

```text
Signed URL
```

or other supported authentication mechanisms.

For workloads running in Google Cloud, prefer workload identity/service identity patterns rather than long-lived key files.

---

# 57. Service Accounts

A service account is a non-human identity used by workloads.

Example:

```text
app-prod@my-project.iam.gserviceaccount.com
```

Architecture:

```text
Application
    |
    v
Service Account
    |
    v
IAM
    |
    v
Cloud Storage
```

---

# 58. Service Account JSON Key

A JSON key can contain long-lived private credential material.

Historically:

```text
Terraform
   |
   v
service-account.json
   |
   v
Google Cloud
```

This can work, but it creates credential-management risks.

Problems:

- Key leakage
- Accidental Git commit
- Difficult rotation
- Long-lived credentials
- Copying between machines
- Secrets in CI/CD

Prefer:

```text
Workload Identity
+
Short-lived credentials
```

where possible.

---

# 59. Application Default Credentials (ADC)

ADC allows Google client libraries to locate credentials appropriate to the environment.

Conceptual model:

```text
Application
    |
    v
Google Client Library
    |
    v
ADC
    |
    +-----------------------+
    |                       |
 Local                    GCP
    |                       |
 User/ADC credentials    Metadata/Workload identity
```

The application code can remain similar while credential sourcing changes between environments.

---

# 60. GCE VM → Cloud Storage

Important architecture:

```text
                         GCE VM
                            |
                            v
                       Application
                            |
                            v
                           ADC
                            |
                            v
                    Metadata Server
                            |
                            v
                   Attached Service
                      Account
                            |
                            v
                  Short-lived Token
                            |
                            v
                    Cloud Storage API
                            |
                            v
                           IAM
                            |
                  +---------+---------+
                  |                   |
                ALLOW                DENY
```

---

# 61. Metadata Server

The VM can access the metadata server using the internal metadata hostname.

Conceptually:

```text
VM
 |
 v
metadata.google.internal
 |
 v
Metadata Server
 |
 v
Instance metadata / service identity information
```

An application can request an access token through the metadata server.

Important:

> The metadata server does not continuously push credentials into the VM.

The application/credential library requests credentials when needed.

---

# 62. Token Lifecycle

Conceptually:

```text
Application
    |
    v
Request token
    |
    v
Short-lived token
    |
    v
Call Cloud Storage
    |
    v
Token expires
    |
    v
Obtain/refresh another token
```

This is safer than storing a permanent credential file on disk.

---

# 63. GCE Service Account vs IAM

Remember:

```text
Service Account
    |
    +-- WHO AM I?
```

```text
IAM Roles
    |
    +-- WHAT CAN I DO?
```

Example:

```text
VM
 |
 +-- Service Account A
          |
          +-- Storage Object Viewer
```

Then:

```text
VM -> GET object
    |
    +-- ALLOW

VM -> DELETE object
    |
    +-- DENY
```

if the service account lacks delete permission.

---

# 64. Private Google Access

Private Google Access allows eligible resources without external IP addresses to reach Google APIs/services using private connectivity.

Conceptually:

```text
GCE VM
(no external IP)
      |
      v
Private Google Access
      |
      v
Google APIs
      |
      v
Cloud Storage
```

Important:

> Private Google Access provides connectivity; it does not grant IAM permissions.

You still need:

```text
Connectivity
+
Authentication
+
Authorization
```

---

# 65. Encryption Fundamentals

## What is encryption?

Encryption transforms readable plaintext into ciphertext using cryptographic algorithms and keys.

Conceptually:

```text
Readable Data
     |
     | Encryption + Key
     v
Unreadable Ciphertext
     |
     | Decryption + Required Key
     v
Readable Data
```

Example:

```text
Plaintext:
"HELLO"

Encryption
    |
    v

Ciphertext:
"8fA91x..."
```

The actual cryptographic process is much more sophisticated than simply "adding a key to the data".

---

# 66. Why Encryption Is Needed

Without encryption:

```text
Data
 |
 v
Storage
 |
 v
Readable if exposed
```

With encryption:

```text
Data
 |
 v
Encryption
 |
 v
Ciphertext
 |
 v
Storage
```

---

# 67. Encryption at Rest

Data stored in Cloud Storage is encrypted server-side before being written to storage.

Mental model:

```text
Client
  |
  v
Cloud Storage
  |
  v
Encrypt
  |
  v
Disk
```

---

# 68. Encryption in Transit

When data travels between systems:

```text
Client
  |
  | Encrypted communication
  v
Cloud Storage
```

This protects data during transmission.

---

# 69. Client-Side Encryption

Client-side encryption happens before the data is sent to Cloud Storage.

```text
Application
    |
    | Encrypt
    v
Ciphertext
    |
    v
Cloud Storage
```

Cloud Storage receives already-encrypted data.

---

# 70. Server-Side Encryption

Cloud Storage receives data and encrypts it server-side before writing to storage.

```text
Application
    |
    v
Cloud Storage
    |
    | Encrypt
    v
Storage infrastructure
```

---

# 71. Cloud Storage Encryption Options

Major concepts:

```text
1. Google-managed encryption
2. CMEK
3. CSEK
4. Client-side encryption
```

Mental model:

```text
Encryption
 |
 +-- Server-side
 |      |
 |      +-- Google-managed
 |      +-- CMEK
 |      +-- CSEK
 |
 +-- Client-side
```

---

# 72. Google-Managed Encryption

Default Cloud Storage behavior.

```text
Object
  |
  v
Cloud Storage
  |
  v
Google-managed encryption
  |
  v
Encrypted at rest
```

Advantages:

- No customer key management required
- Simple
- Secure
- Suitable for many workloads
- No additional key-management operational burden

---

# 73. Why Companies Use CMEK

A company may require:

- Regulatory compliance
- Customer-controlled key lifecycle
- Key rotation control
- Separation of duties
- Audit requirements
- Ability to disable/destroy key access under controlled procedures
- Organizational security policy

Then:

```text
Cloud Storage
      |
      v
CMEK
      |
      v
Cloud KMS
```

---

# 74. CMEK

CMEK = Customer-Managed Encryption Key.

Conceptual architecture:

```text
                  Cloud Storage
                       |
                       v
                      CMEK
                       |
                       v
                  Cloud KMS
                       |
                       v
                Crypto Key
                       |
                +------+------+
                |             |
             Version 1     Version 2
```

Cloud KMS manages the cryptographic key.

---

# 75. CMEK Flow

Typical flow:

```text
1. Choose location
        |
        v
2. Create Key Ring
        |
        v
3. Create Crypto Key
        |
        v
4. Grant required IAM to
   Cloud Storage service identity
        |
        v
5. Configure bucket to use CMEK
        |
        v
6. Upload objects
        |
        v
7. Objects are protected using
   the configured encryption model
```

Important:

> Configuring a CMEK on a bucket means that the bucket's object encryption behavior uses that KMS key according to the bucket configuration.

---

# 76. Key Ring

A key ring is a logical grouping of KMS keys.

```text
Key Ring
 |
 +-- Key A
 |
 +-- Key B
 |
 +-- Key C
```

The key ring itself is organizational.

---

# 77. Crypto Key

A crypto key represents the key resource used for cryptographic operations.

```text
Crypto Key
 |
 +-- Version 1
 +-- Version 2
 +-- Version 3
```

---

# 78. Key Version

Key versions are important.

Example:

```text
Crypto Key
 |
 +-- Version 1
 |
 +-- Version 2
 |
 +-- Version 3
```

A new primary version can be used for new cryptographic operations.

Older versions may still be required to decrypt data encrypted under them.

---

# 79. Key Rotation

Key rotation does **not** mean:

```text
Every day:
Old key disappears
New key replaces everything
```

Instead:

```text
Crypto Key
 |
 +-- Version 1
 |
 +-- Version 2  <-- newer
 |
 +-- Version 3  <-- newest
```

New encryption can use the newer version.

Existing encrypted data may still depend on older versions.

---

# 80. Why Rotate Keys?

Reasons include:

- Security best practices
- Limiting amount of data protected under one key version
- Compliance
- Organizational policy
- Cryptographic hygiene
- Reducing exposure if a key is suspected of compromise

Important:

> The old key version may still be required to decrypt existing ciphertext.

---

# 81. Key Rotation Does Not Mean Re-encrypt Everything Automatically

Suppose:

```text
Object A
encrypted using Key Version 1
```

Then:

```text
Key Version 2 becomes primary
```

Object A does not necessarily become:

```text
encrypted with Version 2
```

automatically just because the primary key version changed.

Think:

```text
Old objects
   |
   +-- May continue using old version

New cryptographic operations
   |
   +-- Use newer primary version
```

---

# 82. What Happens if Key A Is Compromised?

Suppose:

```text
Key Version 1 = compromised
```

You may:

```text
1. Investigate impact
2. Disable/destroy/restrict the compromised version
3. Create/use a new key version
4. Make the new version primary
5. Identify affected ciphertext
6. Re-encrypt/rewrap data where required
```

Important:

> Rotating the key does not automatically re-encrypt all existing objects.

---

# 83. How to Re-encrypt Existing Objects

Conceptually:

```text
Object
  |
  | encrypted using old key
  v
Decrypt using old key
  |
  v
Re-encrypt using new key
  |
  v
Object now protected by new key
```

Depending on the encryption mechanism, object rewrite/update operations may be required.

---

# 84. How to Know Which Key an Object Uses

Object metadata includes encryption information.

You can inspect object metadata using Cloud Storage tooling.

Conceptually:

```text
Object
 |
 v
Metadata
 |
 +-- Encryption type
 +-- KMS key information
 +-- Generation
 +-- Metadata
```

Cloud KMS provides key/version management and auditability, while object metadata tells you the encryption configuration associated with the object.

---

# 85. Where Are KMS Keys Stored?

Cloud KMS is a managed key-management service.

Mental model:

```text
Cloud Storage
      |
      v
Cloud KMS
      |
      v
Key Ring
      |
      v
Crypto Key
      |
      v
Key Version
```

You do not simply store a KMS key as:

```text
/home/key.txt
```

inside your VM.

Cloud KMS manages the key material according to the selected protection level.

CMEK keys can use software, HSM, or external key-management options depending on KMS configuration.

---

# 86. KMS Key Location

The KMS key location matters.

Consider compatibility between:

```text
Cloud Storage bucket location
```

and:

```text
Cloud KMS key location
```

When designing CMEK:

> Choose compatible locations and verify current Google Cloud location requirements.

---

# 87. CMEK Permission Flow

Important:

```text
Application
    |
    v
Cloud Storage
    |
    v
CMEK
    |
    v
Cloud KMS
```

The relevant Google Cloud service identity must have permission to use the KMS key.

Therefore a storage operation can fail because:

```text
Bucket IAM = correct
BUT
KMS IAM = missing
```

---

# 88. CSEK

CSEK = Customer-Supplied Encryption Key.

Conceptual model:

```text
Customer
   |
   v
Provides encryption key
   |
   v
Cloud Storage
   |
   v
Object encrypted using CSEK
```

CSEK is different from CMEK.

```text
CMEK
 |
 +-- Key managed through Cloud KMS

CSEK
 |
 +-- Customer supplies the key
```

---

# 89. CSEK Security Responsibility

CSEK introduces significant customer responsibility.

If the customer loses the required encryption key:

```text
Encrypted Object
      |
      v
Required key unavailable
      |
      v
Data may become inaccessible
```

Therefore key management is critical.

---

# 90. Google-Managed vs CMEK vs CSEK

| Feature | Google-managed | CMEK | CSEK |
|---|---|---|---|
| Google manages encryption infrastructure | Yes | Yes | Yes |
| Customer controls key | No | Yes | Yes |
| Cloud KMS | No | Yes | No |
| Customer supplies raw key | No | No | Yes |
| Operational complexity | Low | Medium/High | High |
| Compliance/control | Standard | Strong | Strong |
| Key rotation management | Google | Customer/KMS | Customer |
| Key loss risk | Low customer burden | Customer must manage access | High customer responsibility |

---

# 91. Client-Side Encryption vs CMEK

## Client-side

```text
Application
    |
    | Encrypt
    v
Ciphertext
    |
    v
Cloud Storage
```

## CMEK

```text
Application
    |
    v
Cloud Storage
    |
    v
Cloud KMS
    |
    v
CMEK
```

CMEK controls the server-side encryption key.

Client-side encryption means the customer encrypts before Cloud Storage receives the plaintext.

---

# 92. Encryption Does Not Replace IAM

Very important.

```text
Encryption
   |
   +-- Protects data confidentiality

IAM
   |
   +-- Controls authorization
```

You need both.

Example:

```text
Private object
+
Encryption
+
IAM
```

Encryption does not replace IAM.

---

# 93. Encryption + CMEK Architecture

```text
                         OBJECT
                           |
                           v
                    CLOUD STORAGE
                           |
                           v
                          CMEK
                           |
                           v
                       CLOUD KMS
                           |
                    +------+------+
                    |             |
                 Key Ring      Crypto Key
                                  |
                           +------+------+
                           |             |
                       Version 1     Version 2
```

---

# 94. Storage Security — Defense in Depth

```text
                    SECURE BUCKET
                         |
       +-----------------+------------------+
       |                 |                  |
       v                 v                  v
      IAM            Encryption         Public Access
       |                 |               Prevention
       v                 v                  |
 Least Privilege       CMEK                |
       |                 |                  |
       +-----------------+------------------+
                         |
                         v
                   Data Protection
                         |
              +----------+----------+
              |                     |
              v                     v
         Retention             Recovery
              |                     |
              v               +-----+-----+
          Compliance          |           |
                           Versioning   Soft Delete
```

---

# 95. Production Bucket Security Checklist

```text
Access:
[ ] IAM
[ ] Least privilege
[ ] Dedicated service accounts
[ ] UBLA
[ ] Public Access Prevention

Encryption:
[ ] Google-managed encryption OR
[ ] CMEK where required
[ ] KMS IAM reviewed
[ ] Key rotation policy

Recovery:
[ ] Soft Delete
[ ] Versioning if required
[ ] Retention
[ ] Backups
[ ] Object holds where needed

Visibility:
[ ] Audit Logs
[ ] Monitoring
[ ] Alerting

Cost:
[ ] Lifecycle
[ ] Appropriate storage class
[ ] Version lifecycle
[ ] Retention-aware cleanup
```

---

# 96. Data Protection Layers

```text
Data
 |
 +-- IAM
 |     |
 |     +-- Unauthorized access protection
 |
 +-- Encryption
 |     |
 |     +-- Data confidentiality
 |
 +-- Versioning
 |     |
 |     +-- Previous version recovery
 |
 +-- Soft Delete
 |     |
 |     +-- Deleted object recovery
 |
 +-- Retention
 |     |
 |     +-- Compliance / deletion prevention
 |
 +-- Backup
       |
       +-- Larger recovery scenarios
```

---

# 97. Backup Architecture

Example:

```text
Database
    |
    v
Backup Process
    |
    v
Cloud Storage
    |
    +-- Encryption
    |
    +-- Retention
    |
    +-- Lifecycle
    |
    +-- Soft Delete
    |
    +-- Audit Logs
```

---

# 98. Incremental Backup Concept

Suppose:

```text
Full backup:
2 GB
```

Then:

```text
Incremental 1:
100 MB

Incremental 2:
150 MB

Incremental 3:
80 MB
```

You do not necessarily need another full 2 GB copy every time.

This can reduce:

```text
Storage
+
Transfer
+
Backup time
```

depending on the backup system.

---

# 99. RPO

RPO = Recovery Point Objective.

Question:

> How much data can we afford to lose?

Example:

```text
Last backup:
10:00

Failure:
10:15
```

Potential RPO:

```text
15 minutes
```

---

# 100. RTO

RTO = Recovery Time Objective.

Question:

> How long can the service remain unavailable?

Example:

```text
Failure
   |
   | < 60 seconds
   v
Service restored
```

RTO:

```text
< 60 seconds
```

---

# 101. AlloyDB Omni Example

A real project example:

```text
AlloyDB Omni HA/DR architecture
        |
        +-- RPO = 0
        |
        +-- RTO < 60 seconds
```

Use carefully in interviews.

Say:

> "In our evaluated AlloyDB Omni architecture and tested failure scenarios, we achieved RPO 0 and RTO below 60 seconds."

Do not claim that every AlloyDB Omni architecture automatically provides these values.

---

# 102. RPO = 0

RPO = 0 means:

```text
Acceptable committed data loss
        =
0
```

Conceptually:

```text
Transaction
   |
   v
Replication
   |
   v
Required durability/commit condition
   |
   v
Commit success
```

If the architecture truly guarantees that committed data is preserved through the relevant failure scenario:

```text
Failure
   |
   v
RPO = 0
```

Do not equate:

```text
Replication exists
```

automatically with:

```text
RPO = 0
```

Replication semantics matter.

---

# 103. RTO Measurement

A meaningful RTO measurement should define:

```text
Start:
Failure injected/detected

End:
Service/application successfully restored
```

Potential flow:

```text
Failure
   |
   v
Failure detection
   |
   v
Leader election/failover
   |
   v
New primary
   |
   v
Database ready
   |
   v
Application connectivity restored
```

Be clear about what timestamps your test used.

---

# 104. Performance and Scalability

Cloud Storage is designed for large-scale storage and access.

Performance depends on:

- Object naming/request patterns
- Client network
- VM network capacity
- Parallelism
- Upload/download method
- Object size
- Application bottlenecks
- Location
- Retrieval behavior

Do not assume:

```text
Slow transfer = Cloud Storage problem
```

Investigate the entire path.

---

# 105. Large File Transfer

Conceptual architecture:

```text
Large File
   |
   +-- Part 1
   +-- Part 2
   +-- Part 3
   +-- ...
        |
        v
Parallel / resumable transfer
        |
        v
Cloud Storage
```

Useful concepts:

- Resumable uploads
- Parallelism
- Network throughput
- Client CPU
- Disk throughput

---

# 106. Cloud Storage Cost Model

Cloud Storage cost can involve:

```text
1. Data storage
2. Operations/requests
3. Network/data transfer
4. Retrieval-related charges
5. Minimum storage duration/early deletion considerations
6. Replication/location architecture
```

Exact pricing varies by location, storage class, operation, and current Google Cloud pricing.

---

# 107. Storage Cost Example

Suppose:

```text
Stored data = 10 TB
```

But application performs:

```text
Millions of requests
```

and:

```text
Large outbound data transfer
```

Total cost is not determined only by:

```text
10 TB
```

Think:

```text
Total Cost
 |
 +-- Storage
 +-- Requests
 +-- Network
 +-- Retrieval
 +-- Architecture
```

---

# 108. Cost Optimization

Use:

```text
Correct storage class
+
Lifecycle management
+
Data deletion
+
Version management
+
Network optimization
+
Request optimization
+
Appropriate retention
```

---

# 109. Why Not Put Everything in Archive?

Because:

```text
Low storage cost
    +
Retrieval/access considerations
    +
Minimum storage duration
    +
Network
```

can result in a higher total cost.

Correct principle:

> Optimize total cost, not just storage price per GB.

---

# 110. Capacity Planning

Track:

```text
Current data
+
Daily/monthly growth
+
Retention period
+
Versioning
+
Soft Delete
+
Backup frequency
+
Replication
```

Example:

```text
Current = 10 TB
Growth = 2 TB/month

12-month forecast:
10 + (2 * 12)
= 34 TB
```

---

# 111. Storage Transfer and Migration

Main approaches:

```text
1. gcloud storage
2. Storage Transfer Service
3. Transfer Appliance
4. Cloud-to-cloud transfer
5. Application/API-based transfer
```

---

# 112. gcloud storage

Modern Google Cloud CLI for Cloud Storage.

Examples:

```bash
gcloud storage ls
```

```bash
gcloud storage ls gs://my-bucket
```

```bash
gcloud storage cp file.txt gs://my-bucket/
```

```bash
gcloud storage cp gs://my-bucket/file.txt .
```

```bash
gcloud storage cp --recursive ./data gs://my-bucket/data/
```

---

# 113. `gcloud storage` vs `gsutil`

Historically:

```text
gsutil
```

was commonly used.

Modern direction:

```text
gcloud storage
```

is preferred for new workflows.

You may still encounter:

```bash
gsutil cp
gsutil rsync
gsutil ls
```

in existing environments.

For LevelUp:

```text
Modern:
gcloud storage

Legacy/existing:
gsutil
```

---

# 114. Useful gcloud storage Commands

```bash
# List buckets
gcloud storage ls

# List objects
gcloud storage ls gs://BUCKET

# Upload
gcloud storage cp FILE gs://BUCKET/

# Download
gcloud storage cp gs://BUCKET/OBJECT .

# Recursive upload
gcloud storage cp --recursive DIRECTORY gs://BUCKET/

# Move
gcloud storage mv SOURCE DESTINATION

# Delete
gcloud storage rm gs://BUCKET/OBJECT

# Synchronize
gcloud storage rsync SOURCE DESTINATION

# Display storage usage
gcloud storage du gs://BUCKET

# Calculate hashes
gcloud storage hash FILE

# Diagnose
gcloud storage diagnose ...
```

Always check the current CLI help for exact flags.

---

# 115. Create a Bucket

Example:

```bash
gcloud storage buckets create gs://BUCKET_NAME \
  --location=REGION
```

Example:

```bash
gcloud storage buckets create gs://my-prod-data \
  --location=asia-south1
```

---

# 116. Lifecycle Configuration

Create a lifecycle JSON file:

```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "COLDLINE"
        },
        "condition": {
          "age": 30
        }
      }
    ]
  }
}
```

Apply:

```bash
gcloud storage buckets update gs://BUCKET_NAME \
  --lifecycle-file=lifecycle.json
```

---

# 117. Lifecycle Delete Example

```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {
          "type": "Delete"
        },
        "condition": {
          "age": 365
        }
      }
    ]
  }
}
```

Apply:

```bash
gcloud storage buckets update gs://BUCKET_NAME \
  --lifecycle-file=lifecycle.json
```

Important:

Lifecycle deletion is automatic.

There is no default human approval step such as:

```text
5 years reached
    |
    v
Ask administrator?
```

Instead:

```text
Condition satisfied
    |
    v
Lifecycle action
```

Retention policies/holds can prevent deletion while applicable.

---

# 118. Enable Versioning

Command pattern:

```bash
gcloud storage buckets update gs://BUCKET_NAME \
  --versioning
```

Check configuration:

```bash
gcloud storage buckets describe gs://BUCKET_NAME
```

---

# 119. Soft Delete

Soft Delete is configured at the bucket level.

Inspect bucket configuration:

```bash
gcloud storage buckets describe gs://BUCKET_NAME
```

Restore operations use:

```bash
gcloud storage restore ...
```

Always check current Cloud SDK syntax for the exact restore flags.

---

# 120. IAM Commands

Grant bucket-level object viewer role:

```bash
gcloud storage buckets add-iam-policy-binding gs://BUCKET_NAME \
  --member="serviceAccount:SERVICE_ACCOUNT_EMAIL" \
  --role="roles/storage.objectViewer"
```

Example:

```bash
gcloud storage buckets add-iam-policy-binding gs://prod-data \
  --member="serviceAccount:app@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"
```

---

# 121. Common Storage IAM Roles

Examples:

```text
roles/storage.objectViewer
    |
    +-- Read objects

roles/storage.objectCreator
    |
    +-- Create objects

roles/storage.objectUser
    |
    +-- Common object operations depending on role definition

roles/storage.objectAdmin
    |
    +-- Broad object-level administration

roles/storage.admin
    |
    +-- Broad Storage administration
```

Always verify the current permission set of a role before granting it.

---

# 122. Service Account Creation

Example:

```bash
gcloud iam service-accounts create app-storage \
  --display-name="Application Storage Account"
```

Then:

```text
app-storage@PROJECT_ID.iam.gserviceaccount.com
```

Grant bucket permissions:

```bash
gcloud storage buckets add-iam-policy-binding gs://BUCKET_NAME \
  --member="serviceAccount:app-storage@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"
```

---

# 123. CMEK Setup — Mental Flow

```text
Project
 |
 v
Cloud KMS
 |
 +-- Key Ring
       |
       +-- Crypto Key
              |
              +-- Version 1
              +-- Version 2
 |
 v
Grant Cloud Storage service identity permission
 |
 v
Bucket
 |
 v
Default KMS key / CMEK configuration
 |
 v
Objects
```

---

# 124. CMEK IAM

The Cloud Storage service identity needs the required permission on the KMS key.

Conceptually:

```text
Cloud Storage Service Identity
          |
          v
Cloud KMS Crypto Key
          |
          v
Permission to use key
```

If missing:

```text
Upload
  |
  v
Cloud Storage
  |
  v
CMEK
  |
  v
KMS
  |
  X
Permission denied
```

---

# 125. Encryption Enforcement

Cloud Storage can enforce/restrict which encryption types can be used for new objects.

Possible concepts:

```text
Google-managed encryption
CMEK
CSEK
```

Example enforcement configuration is available through the Cloud Storage bucket encryption-enforcement controls.

Apply using the current `gcloud storage` encryption-enforcement options.

Important:

> Encryption enforcement applies to new object creation operations. It does not automatically change the encryption type of existing objects.

---

# 126. Signed URL — Full Example

Create a download URL:

```bash
gcloud storage sign-url \
  gs://my-bucket/private/report.pdf \
  --duration=10m \
  --impersonate-service-account=signer@my-project.iam.gserviceaccount.com
```

Create an upload URL:

```bash
gcloud storage sign-url \
  gs://my-bucket/uploads/report.pdf \
  --http-verb=PUT \
  --duration=10m \
  --headers=content-type=application/pdf \
  --impersonate-service-account=signer@my-project.iam.gserviceaccount.com
```

---

# 127. Signed URL Request Flow

```text
Application
    |
    | authenticate customer
    v
Application authorization
    |
    | customer allowed?
    v
Impersonate/sign as service account
    |
    v
Generate signed URL
    |
    v
Return URL to customer
    |
    v
Customer
    |
    v
Cloud Storage
    |
    v
Signature validation
    |
    v
Object operation
```

---

# 128. Signed Policy Document — Detailed Flow

```text
Customer Browser
       |
       v
Application
       |
       v
Authenticate Customer
       |
       v
Authorize Upload
       |
       v
Create Policy JSON
       |
       +-- expiration
       +-- bucket
       +-- object prefix
       +-- content type
       +-- size limit
       +-- other constraints
       |
       v
Encode Policy
       |
       v
Cryptographic signing
       |
       v
HTML Form
       |
       v
Customer Upload
       |
       v
Cloud Storage
       |
       v
Validate Policy
       |
   +---+---+
   |       |
 VALID   INVALID
   |       |
   v       v
UPLOAD   DENY
```

---

# 129. Storage Transfer Service

For large-scale managed transfers:

```text
Source
   |
   v
Storage Transfer Service
   |
   v
Cloud Storage
```

Useful for:

- On-premises transfers
- Cloud-to-cloud transfers
- Large datasets
- Recurring synchronization
- Managed transfer workflows

---

# 130. Transfer Appliance

When:

```text
Dataset = massive
+
Network bandwidth = insufficient
+
Migration deadline = tight
```

consider:

```text
On-premises
     |
     v
Transfer Appliance
     |
     | Physical shipment
     v
Google
     |
     v
Cloud Storage
```

---

# 131. Network Transfer vs Appliance

Decision:

```text
                 DATA MIGRATION
                      |
             Is network sufficient?
                /             \
              YES              NO
               |                |
               v                v
      Storage Transfer      Transfer
          Service           Appliance
```

Do not use arbitrary dataset-size thresholds.

Calculate:

```text
Data volume
/
Available throughput
=
Theoretical transfer time
```

Then account for:

- Protocol overhead
- Network contention
- Retries
- Source performance
- Destination constraints

---

# 132. Migration Calculation

Example:

```text
Data = 100 TB
Bandwidth = 1 Gbps
```

Approximate:

```text
1 Gbps ≈ 125 MB/s
```

Theoretical transfer time:

```text
100 TB / 125 MB/s
≈ 9.3 days
```

This is idealized.

Real-world transfer can take longer.

---

# 133. Migration with Minimal Downtime

```text
Phase 1:
Initial bulk transfer
        |
        v
Phase 2:
Incremental synchronization
        |
        v
Phase 3:
Final validation
        |
        v
Phase 4:
Freeze/control writes if required
        |
        v
Phase 5:
Final sync
        |
        v
Phase 6:
Application cutover
```

This reduces downtime.

---

# 134. Migration Validation

Do not assume:

```text
Transfer completed
    =
Migration correct
```

Validate:

```text
Source
 |
 +-- Object count
 +-- Size
 +-- Checksums
 +-- Metadata
 |
 v
Destination
 |
 +-- Object count
 +-- Size
 +-- Checksums
 +-- Metadata
 |
 v
Compare
```

---

# 135. Checksums

Same file size does not guarantee same content.

Example:

```text
Source:
100 MB

Destination:
100 MB
```

Could still be:

```text
Different content
```

Hash/checksum comparison can provide stronger integrity validation.

---

# 136. File System Permissions vs GCS IAM

On Linux:

```bash
chmod 640 file
```

uses POSIX permissions.

GCS:

```text
IAM
+
Object/bucket access model
```

There is no direct one-to-one mapping.

When migrating:

```text
Linux permissions
        |
        v
Redesign destination IAM
```

---

# 137. Cloud Storage + GCE Backup Architecture

```text
                    GCE VM
                       |
                       v
                 Backup Process
                       |
                       v
                   Cloud Storage
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Encryption   Lifecycle    Retention
          |
          v
       CMEK/KMS
```

Service identity:

```text
VM
 |
 v
Attached Service Account
 |
 v
Minimum required Storage role
```

---

# 138. GCE Restore Architecture

```text
Cloud Storage
     |
     v
Backup Object
     |
     v
GCE VM
     |
     v
Restore Process
     |
     v
Database/Application
```

Required permission might be:

```text
storage.objects.get
```

rather than full Storage Admin.

---

# 139. GCE Without External IP

```text
GCE VM
(no external IP)
      |
      v
Private Google Access
      |
      v
Google APIs
      |
      v
Cloud Storage
```

Still requires:

```text
Authentication
+
IAM authorization
```

---

# 140. Application → Storage Architecture

```text
                  USERS
                    |
                    v
                APPLICATION
                    |
             +------+------+
             |             |
             v             v
        Service Identity  Signed URL
             |             |
             v             v
            IAM          Customer
             |             |
             +------+------+
                    |
                    v
              CLOUD STORAGE
```

---

# 141. Customer Download Architecture

```text
Customer
   |
   v
Application
   |
   | Authenticate
   | Authorize
   |
   v
Generate Signed URL
   |
   v
Customer
   |
   v
Cloud Storage
   |
   v
Download
```

---

# 142. Customer Upload Architecture

```text
Customer
   |
   v
Application
   |
   | Authenticate
   | Authorize
   |
   v
Generate Signed URL
   |
   v
Customer
   |
   v
Cloud Storage
   |
   v
Upload
```

For flexible browser/form upload constraints:

```text
Customer
   |
   v
Signed Policy Document
   |
   v
Cloud Storage
```

---

# 143. Troubleshooting Framework

Whenever Cloud Storage fails:

```text
                         ERROR
                           |
                           v
                   What operation?
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          Identity      Resource       Network
             |             |             |
             v             v             v
            WHO?         WHICH?          PATH?
             |             |
             +------+------+
                    |
                    v
                   IAM
                    |
             +------+------+
             |             |
           ALLOW          DENY
             |             |
             |       Investigate
             |             |
             |     +-------+-------+
             |     |       |       |
             |    CMEK  Retention Policy
             |
             v
        Cloud Storage
             |
             v
         Audit Logs
             |
             v
          Root Cause
```

---

# 144. 403 Forbidden

A 403 generally indicates an authorization/permission problem.

Check:

```text
1. Which identity?
2. Which service account?
3. Which operation?
4. Which bucket/object?
5. Required permission?
6. IAM role?
7. IAM Deny?
8. Organization policy?
9. VPC Service Controls?
10. CMEK/KMS?
```

---

# 145. 403 Example

Application:

```text
GET gs://prod-data/customer.csv
```

Service account:

```text
app-prod@project.iam.gserviceaccount.com
```

Check:

```text
Does it have:
storage.objects.get
```

If not:

```text
403
```

---

# 146. 401 vs 403

General mental model:

```text
401
 |
 +-- Authentication/credential problem
```

```text
403
 |
 +-- Caller recognized but operation not authorized
```

Do not treat these as absolute rules for every implementation, but they are useful troubleshooting starting points.

---

# 147. 404 Not Found

Possible causes:

```text
Bucket doesn't exist
Object doesn't exist
Wrong bucket name
Wrong object name
Wrong prefix
Case mismatch
Application configuration error
```

Example:

```text
reports/file.txt
```

is not necessarily the same as:

```text
Reports/file.txt
```

Object names must be treated exactly.

---

# 148. 403 vs 404

Do not always assume:

```text
404 = definitely doesn't exist
```

Authorization/privacy behavior can affect what information is exposed.

Therefore investigate:

```text
Identity
+
IAM
+
Resource name
+
Object existence
+
Audit logs
```

---

# 149. Signed URL Troubleshooting

If signed URL fails:

```text
Check:
 |
 +-- Expired?
 +-- Correct HTTP method?
 +-- Correct object?
 +-- Signature valid?
 +-- Required headers correct?
 +-- Required query parameters correct?
 +-- URL modified?
 +-- Signing credentials valid?
```

---

# 150. Signed URL Signature Mismatch

Example:

Signed:

```text
PUT
Content-Type: text/plain
```

Actual:

```text
PUT
Content-Type: application/pdf
```

Potential result:

```text
Signature mismatch
```

Similarly:

```text
Signed GET
```

cannot simply be reused as:

```text
PUT
```

---

# 151. CMEK Troubleshooting

If upload/download fails around encryption:

```text
Check:
 |
 +-- Correct KMS key?
 +-- Correct location?
 +-- Key version enabled?
 +-- Cloud Storage service identity?
 +-- KMS IAM permission?
 +-- Bucket CMEK configuration?
```

---

# 152. IAM ALLOW But Operation Still Fails

Even if:

```text
IAM = ALLOW
```

operation may be blocked by:

```text
Retention
Object Hold
IAM Deny
Organization Policy
VPC Service Controls
CMEK/KMS
Public Access Prevention
Other applicable controls
```

Therefore:

> "I have Storage Admin" does not always mean every operation must succeed.

---

# 153. Disabled KMS Key Version

Suppose:

```text
Crypto Key
 |
 +-- Version 1  <-- disabled
 +-- Version 2  <-- enabled
```

Object:

```text
Object A
 |
 +-- encrypted using Version 1
```

If the object requires Version 1 for decryption and that version is disabled:

```text
Object access
    |
    v
Needs Version 1
    |
    v
Version 1 disabled
    |
    v
Operation may fail
```

Therefore key lifecycle changes require careful planning.

---

# 154. Audit Logs

Audit logs help answer:

```text
WHO?
WHAT?
WHEN?
WHICH RESOURCE?
```

Example:

```text
Object deleted
    |
    v
Audit Logs
    |
    v
Principal
Timestamp
Resource
Operation
```

Essential for:

- Security investigations
- Compliance
- Troubleshooting
- Change tracking

---

# 155. Monitoring vs Audit Logs

```text
Audit Logs
    |
    +-- What happened?
    +-- Who did it?
    +-- When?
```

```text
Monitoring
    |
    +-- How is the system behaving?
    +-- Error rates
    +-- Latency
    +-- Usage
```

They are complementary.

---

# 156. Slow Transfer Troubleshooting

If upload is slow:

```text
Client
 |
 +-- Disk performance
 +-- CPU
 +-- Compression
 +-- Encryption
 |
 v
Network
 |
 +-- Bandwidth
 +-- Latency
 +-- Congestion
 |
 v
Cloud Storage
```

Also check:

- Parallelism
- Resumable uploads
- Object size
- Region/location
- Application bottlenecks

---

# 157. "It Works on My Laptop but Not on VM"

Laptop:

```text
Developer
 |
 v
User credentials
 |
 v
IAM
 |
 v
Cloud Storage
```

VM:

```text
Application
 |
 v
VM Service Account
 |
 v
IAM
 |
 v
Cloud Storage
```

Different identities.

Therefore:

```text
Laptop works
VM fails
```

may simply mean:

```text
User permissions != Service Account permissions
```

---

# 158. Upload Works but Download Fails

Possible:

```text
Create permission = ALLOW
Get permission = DENY
```

Conceptually:

```text
Upload
 |
 +-- storage.objects.create
 |
 +-- ALLOW

Download
 |
 +-- storage.objects.get
 |
 +-- DENY
```

Permissions are operation-specific.

---

# 159. Delete Allowed but Blocked

Example:

```text
IAM
 |
 +-- delete permission = ALLOW
```

but:

```text
Retention policy
 |
 +-- deletion not yet allowed
```

Result:

```text
DELETE
 |
 v
BLOCKED
```

---

# 160. Unexpected Public Access

If someone says:

> "Why is this object publicly accessible?"

Check:

```text
1. IAM
2. allUsers
3. allAuthenticatedUsers
4. ACLs if applicable
5. UBLA
6. Public Access Prevention
7. Signed URL
```

A signed URL does not make the bucket publicly accessible.

---

# 161. Deleted Object Recovery

If an object was accidentally deleted:

```text
Object missing
    |
    v
Audit Logs
    |
    v
Was it deleted?
    |
    v
Check:
 |
 +-- Soft Delete
 +-- Versioning
 +-- Backup
 +-- Retention
```

Recovery depends on what protections were enabled before deletion.

---

# 162. Overwritten Object Recovery

If versioning is enabled:

```text
Object
 |
 +-- V1
 +-- V2
 +-- V3 current
```

You may recover an older version.

Without versioning, recovery may depend on:

```text
Soft Delete
+
Backup
+
Other recovery mechanisms
```

---

# 163. Production Storage Architecture

```text
                     PRODUCTION BUCKET
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
         IAM               UBLA              PAP
          |                                   |
          v                                   v
    Least privilege                       No Public Access
          |
          v
      Service Accounts
          |
          v
      Cloud Storage
          |
   +------+-------+----------------+
   |              |                |
   v              v                v
Encryption     Lifecycle       Retention
   |              |                |
   v              v                v
 CMEK/KMS      Cost Control     Compliance
   |
   v
Recovery
   |
 +------+------+
 |             |
 v             v
Versioning  Soft Delete
 |
 v
Audit Logs
 |
 v
Monitoring
```

---

# 164. Banking Bucket Example

Requirement:

```text
Sensitive financial data
Private
7-year retention
Application read/write
Customer-controlled encryption
Auditing
```

Architecture:

```text
                         BANKING DATA
                              |
                              v
                       Cloud Storage
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
         IAM                 UBLA                PAP
          |                                       |
          v                                       v
    Least privilege                         Block public
          |
          v
    Application SA
          |
          v
        CMEK
          |
          v
      Cloud KMS
          |
          v
      Retention
          |
          v
  Versioning / Soft Delete
          |
          v
      Audit Logs
          |
          v
      Monitoring
```

---

# 165. Enterprise Application Architecture

```text
                          USERS
                            |
                            v
                       APPLICATION
                            |
             +--------------+--------------+
             |                             |
             v                             v
       Service Identity               Signed URL
             |                             |
             v                             v
            IAM                        Customer
             |                             |
             +--------------+--------------+
                            |
                            v
                     CLOUD STORAGE
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
      Encryption        Lifecycle         Retention
          |                 |                 |
          v                 v                 v
       CMEK/KMS          Cost             Compliance
                            |
                            v
                      Recovery
                            |
                    +-------+-------+
                    |               |
                    v               v
                Versioning      Soft Delete
```

---

# 166. Multi-Application Architecture

```text
                     CLOUD STORAGE
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
       App SA          Backup SA       Restore SA
          |               |               |
       Viewer          Creator          Viewer
          |               |               |
          +---------------+---------------+
                          |
                          v
                    Least Privilege
```

This reduces blast radius.

---

# 167. Storage + CDN Concept

For public/high-volume content:

```text
Users
  |
  v
CDN
  |
  v
Cloud Storage
```

Benefits can include:

- Lower latency
- Cache frequently accessed content
- Reduce repeated origin requests
- Better global distribution

CDN introduces its own pricing and architecture considerations.

---

# 168. Storage + Application Direct Upload

```text
                 CUSTOMER
                    |
                    v
                APPLICATION
                    |
             Authenticate
                    |
             Authorize upload
                    |
                    v
            Signed Upload URL
                    |
                    v
                 CUSTOMER
                    |
                    v
              CLOUD STORAGE
```

Application does not have to proxy the file.

---

# 169. Storage + Application Direct Download

```text
CLOUD STORAGE
      |
      v
Signed URL
      |
      v
CUSTOMER
```

Instead of:

```text
Cloud Storage
      |
      v
Application
      |
      v
Customer
```

This can reduce application server load.

---

# 170. Cloud Storage Migration Architecture

```text
                     DATA MIGRATION
                           |
          +----------------+----------------+
          |                                 |
          v                                 v
     Good Network                      Poor Network
          |                                 |
          v                                 v
Storage Transfer                    Transfer Appliance
Service                                  |
          |                              |
          +---------------+--------------+
                          |
                          v
                    Cloud Storage
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
          Validate     Secure      Monitor
              |
              v
           Cutover
```

---

# 171. Migration with Continuous Synchronization

```text
On-Premises
    |
    | Initial bulk transfer
    v
Cloud Storage
    |
    | Incremental changes
    v
Cloud Storage
    |
    | Final synchronization
    v
Cutover
```

Useful when source data continues changing during migration.

---

# 172. Migration Rollback

Always consider:

```text
Cutover
   |
   +-- SUCCESS
   |      |
   |      v
   |   Cloud workload
   |
   +-- FAILURE
          |
          v
       Rollback
          |
          v
    Original workload
```

A migration without rollback planning is risky.

---

# 173. Storage Security Checklist

```text
ACCESS
[ ] Least privilege
[ ] IAM
[ ] Dedicated service accounts
[ ] UBLA
[ ] Public Access Prevention

AUTHENTICATION
[ ] Workload identity
[ ] ADC
[ ] Short-lived credentials
[ ] Avoid long-lived JSON keys

ENCRYPTION
[ ] Google-managed encryption
[ ] CMEK where required
[ ] KMS permissions
[ ] Key rotation
[ ] Key lifecycle

DATA PROTECTION
[ ] Retention
[ ] Object holds
[ ] Soft Delete
[ ] Versioning where required
[ ] Backups

COST
[ ] Storage class
[ ] Lifecycle
[ ] Version cleanup
[ ] Soft Delete implications
[ ] Network costs
[ ] Retrieval costs

OBSERVABILITY
[ ] Audit Logs
[ ] Monitoring
[ ] Alerts

MIGRATION
[ ] Integrity validation
[ ] Incremental sync
[ ] Cutover plan
[ ] Rollback plan
```

---

# 174. Common Misconceptions

## Misconception 1

> "Cloud Storage is basically a filesystem."

No.

It is object storage.

## Misconception 2

> "Archive is always cheapest overall."

No.

Access/retrieval and other costs matter.

## Misconception 3

> "Private bucket means nobody without Google account can access it."

Not necessarily.

A valid signed URL can provide temporary access without the recipient having a Google Cloud account.

## Misconception 4

> "Signed URL is bound to the person I sent it to."

No.

It is generally a bearer-style URL.

## Misconception 5

> "Signed URL and Signed Policy Document are the same."

No.

Signed URL:

```text
Specific signed request/resource
```

Signed policy:

```text
Upload conditions for HTML form POST
```

## Misconception 6

> "Signed URL makes bucket public."

No.

A private bucket can still use signed URLs.

## Misconception 7

> "Encryption means adding a key to the data."

Too simplistic.

Encryption is a cryptographic transformation using algorithms and keys.

## Misconception 8

> "Key rotation means old key disappears."

No.

Old key versions may be required to decrypt existing data.

## Misconception 9

> "Changing primary KMS key version automatically re-encrypts all objects."

No.

Changing key version and re-encrypting existing data are separate operations.

## Misconception 10

> "Private Google Access gives the VM access to GCS."

Not by itself.

It provides connectivity to Google APIs/services.

IAM still controls authorization.

## Misconception 11

> "If IAM says ALLOW, operation must succeed."

No.

Other controls can block operations.

## Misconception 12

> "Upload permission means download permission."

No.

Different operations require different permissions.

## Misconception 13

> "Transfer completed means migration succeeded."

No.

Validate data integrity.

## Misconception 14

> "RPO = 0 just because replicas exist."

No.

Replication semantics and commit guarantees matter.

---

# 175. Important Comparisons

## Bucket vs Object

```text
Bucket
 |
 +-- Container

Object
 |
 +-- Actual data + metadata
```

## Standard vs Nearline vs Coldline vs Archive

```text
Standard
 |
 +-- Frequent

Nearline
 |
 +-- Less frequent

Coldline
 |
 +-- Infrequent

Archive
 |
 +-- Very rare
```

## Lifecycle vs Retention

```text
Lifecycle
 |
 +-- Automatic action

Retention
 |
 +-- Prevent premature deletion
```

## Versioning vs Soft Delete

```text
Versioning
 |
 +-- Retain noncurrent versions

Soft Delete
 |
 +-- Retain deleted resources temporarily
```

## IAM vs Signed URL

```text
IAM
 |
 +-- Identity-based authorization

Signed URL
 |
 +-- Temporary bearer-style access
```

## Signed URL vs Signed Policy

```text
Signed URL
 |
 +-- Specific signed request
 +-- GET/PUT/etc.
 +-- Temporary access


Signed Policy
 |
 +-- Upload conditions
 +-- HTML form POST
 +-- Size/type/prefix restrictions
```

## Google-managed vs CMEK vs CSEK

```text
Google-managed
 |
 +-- Google manages key management

CMEK
 |
 +-- Customer controls key through Cloud KMS

CSEK
 |
 +-- Customer supplies key
```

## Authentication vs Authorization

```text
Authentication
 |
 +-- WHO?

Authorization
 |
 +-- WHAT CAN THEY DO?
```

## RPO vs RTO

```text
RPO
 |
 +-- How much data can be lost?

RTO
 |
 +-- How long can service be unavailable?
```

## Storage Transfer Service vs Transfer Appliance

```text
Storage Transfer Service
 |
 +-- Network-based managed transfer


Transfer Appliance
 |
 +-- Physical data transfer
```

---

# 176. LevelUp Interview Question: Secure Bucket

### Question

> Design a secure GCS bucket for a banking application.

### Strong Answer

```text
1. Use least-privilege IAM.
2. Use a dedicated service account for the application.
3. Enable UBLA.
4. Enable Public Access Prevention where public access isn't required.
5. Use Google-managed encryption or CMEK depending on compliance.
6. If CMEK is required, manage keys through Cloud KMS.
7. Configure retention according to regulatory requirements.
8. Consider Soft Delete and Versioning based on recovery requirements.
9. Configure lifecycle policies for cost optimization without violating retention.
10. Enable audit logging and monitoring.
11. Restrict network access as required.
12. Regularly review IAM and key permissions.
```

---

# 177. LevelUp Interview Question: VM Gets 403

### Question

> A GCE VM gets 403 when accessing a private bucket. How do you troubleshoot?

### Answer

```text
1. Identify the actual VM service account.
2. Confirm the application is using the expected identity.
3. Identify the exact bucket/object.
4. Identify the operation:
   GET / PUT / DELETE / LIST
5. Determine the required permission.
6. Check IAM role assignment.
7. Check IAM Deny policies.
8. Check organization/policy restrictions.
9. Check VPC Service Controls if applicable.
10. Check CMEK/KMS permissions if encryption is involved.
11. Check Audit Logs.
12. Fix only the missing permission; do not blindly grant Storage Admin.
```

---

# 178. LevelUp Interview Question: External Customer Download

### Question

> A customer without a Google account needs to download one private object for 10 minutes. What would you use?

### Answer

```text
Generate a signed URL for:
    |
    +-- Specific object
    +-- GET
    +-- 10-minute expiration

Return URL to customer.

Keep bucket private.
```

Do not make the bucket public simply for this requirement.

---

# 179. LevelUp Interview Question: Customer Upload

### Question

> A customer needs to upload images up to 10 MB to `uploads/`. How would you design it?

If exact object upload is enough:

```text
Signed PUT URL
```

If you need flexible browser/form upload constraints:

```text
Signed Policy Document
```

Policy can enforce:

```text
Bucket
Prefix = uploads/
Content-Type = image/*
Size <= 10 MB
Expiration
```

---

# 180. LevelUp Interview Question: CMEK

### Question

> Why would a company choose CMEK instead of Google-managed encryption?

### Answer

> "The company may have compliance, regulatory, separation-of-duties, key lifecycle, audit, or customer-controlled key management requirements. CMEK allows the organization to manage encryption keys through Cloud KMS while Cloud Storage uses those keys for encryption."

---

# 181. LevelUp Interview Question: Key Rotation

### Question

> What happens when you rotate a CMEK?

### Answer

> "A new key version is created and can become the primary version for new cryptographic operations. Existing objects encrypted with an older version may continue to depend on that older version. Rotation does not automatically re-encrypt every existing object."

---

# 182. LevelUp Interview Question: Compromised Key

### Question

> What would you do if a key version is compromised?

### Answer

```text
1. Investigate impact.
2. Restrict/disable the compromised key version according to incident procedure.
3. Create/use a new key version.
4. Make the new version primary where appropriate.
5. Identify affected objects/data.
6. Re-encrypt or rewrite affected objects as required.
7. Review audit logs.
8. Investigate how the compromise occurred.
9. Update security controls.
```

Do not simply delete the old key without understanding which objects still depend on it.

---

# 183. LevelUp Interview Question: Cost Increase

### Question

> GCS cost increased 50%. How do you investigate?

```text
1. Check stored data growth.
2. Check storage class distribution.
3. Check request/operation volume.
4. Check network egress.
5. Check retrieval.
6. Check versioning.
7. Check soft-delete retention.
8. Check backup frequency.
9. Check lifecycle policies.
10. Check replication/location changes.
11. Identify dominant cost driver.
12. Optimize without violating security/compliance.
```

---

# 184. LevelUp Interview Question: 500 TB Migration

### Question

> You have 500 TB on-premises and need minimal downtime.

### Answer

```text
1. Calculate network transfer time.
2. Determine whether bandwidth is sufficient.
3. Use Storage Transfer Service for managed network transfer if suitable.
4. Perform initial bulk transfer.
5. Perform incremental synchronization.
6. Validate object counts/checksums/metadata.
7. Perform final synchronization.
8. Control/freeze writes if necessary.
9. Cut over application.
10. Maintain rollback plan.
```

If network cannot meet the migration window:

```text
Evaluate Transfer Appliance.
```

---

# 185. LevelUp Interview Question: Slow Upload

### Question

> Large GCS uploads are slow. What do you check?

```text
Client
 |
 +-- Disk
 +-- CPU
 |
Network
 |
 +-- Bandwidth
 +-- Latency
 +-- Congestion
 |
Application
 |
 +-- Parallelism
 +-- Resumable upload
 |
Cloud
 |
 +-- Location
 +-- Architecture
```

Do not automatically blame Cloud Storage.

---

# 186. LevelUp Interview Question: Upload Works, Download Fails

Possible explanation:

```text
Create permission = ALLOW
Get permission = DENY
```

Therefore:

```text
Upload works
Download fails
```

because operations require different permissions.

---

# 187. LevelUp Interview Question: IAM Allows Delete but Delete Fails

Possible causes:

```text
Retention policy
Object hold
IAM Deny
Organization policy
Other applicable controls
```

The answer is:

> IAM permission is necessary but not always sufficient for an operation to succeed.

---

# 188. LevelUp Interview Question: Laptop Works, VM Fails

Check:

```text
Laptop
 |
 +-- User identity

VM
 |
 +-- Service account identity
```

They may have different IAM permissions.

---

# 189. LevelUp Interview Question: Why Service Account Instead of JSON Key?

Strong answer:

> "A service account provides workload identity, while attached-service-account/workload-identity patterns allow applications to obtain short-lived credentials without storing long-lived private keys on disk. This reduces credential leakage and rotation risks and supports least privilege."

---

# 190. LevelUp Interview Question: What Is ADC?

Answer:

> "Application Default Credentials is a mechanism used by Google Cloud client libraries to locate credentials appropriate for the environment. On GCP workloads, this can integrate with the workload's service identity and metadata/identity infrastructure, while local development can use developer-configured credentials."

---

# 191. LevelUp Interview Question: Does Metadata Server Continuously Push Updates?

No.

Mental model:

```text
Application
    |
    | Request
    v
Metadata Server
    |
    v
Credential/token
```

It is not:

```text
Metadata Server
    |
    | Every few seconds
    v
Push credentials into VM
```

The application/credential library requests what it needs.

---

# 192. LevelUp Interview Question: Does Private Google Access Give Storage Permission?

No.

```text
Private Google Access
    |
    +-- Connectivity

IAM
    |
    +-- Authorization
```

Both may be needed.

---

# 193. Complete Storage Troubleshooting Decision Tree

```text
                        CLOUD STORAGE ISSUE
                                |
                                v
                         What operation?
                                |
          +---------------------+---------------------+
          |                     |                     |
          v                     v                     v
       Identity              Resource             Network
          |                     |                     |
          v                     v                     v
         WHO?                 WHICH?                 PATH?
          |                     |
          +----------+----------+
                     |
                     v
                    IAM
                     |
             +-------+-------+
             |               |
           ALLOW            DENY
             |               |
             |         Check policies
             |               |
             |       +-------+-------+
             |       |       |       |
             |      CMEK  Retention  IAM Deny
             |       |
             |       v
             |      KMS
             |
             v
       Cloud Storage
             |
             v
        Audit Logs
             |
             v
         Root Cause
```

---

# 194. Complete Storage Access Architecture

```text
                              CLIENT
                                |
                +---------------+---------------+
                |                               |
                v                               v
          Internal Workload              External Customer
                |                               |
                v                               v
          Service Identity                 Application
                |                               |
                v                               v
          ADC / Workload Identity       Signed URL / Policy
                |                               |
                +---------------+---------------+
                                |
                                v
                         CLOUD STORAGE
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
                   IAM                  Encryption
                    |                       |
                    v                       v
              Authorization            CMEK/KMS
                    |
                    v
                 Object
```

---

# 195. Complete Storage Security Architecture

```text
                         SECURE GCS
                             |
        +--------------------+--------------------+
        |                    |                    |
        v                    v                    v
      ACCESS              ENCRYPTION           RECOVERY
        |                    |                    |
        v                    v                    v
      IAM                  CMEK                Soft Delete
      UBLA                 KMS                 Versioning
      PAP                  Rotation            Backup
      Least Privilege                          Retention
        |                    |                    |
        +--------------------+--------------------+
                             |
                             v
                       OBSERVABILITY
                             |
                     +-------+-------+
                     |               |
                     v               v
                 Audit Logs      Monitoring
```

---

# 196. Complete Storage Cost Architecture

```text
                         TOTAL COST
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
       Storage             Requests            Network
          |                   |                   |
          v                   v                   v
     Storage Class       GET/PUT/LIST         Egress
          |
          v
     Lifecycle
          |
          v
    Data Retention
          |
          v
     Versioning
          |
          v
      Soft Delete
```

---

# 197. Complete Storage Migration Architecture

```text
                           SOURCE
                             |
                +------------+------------+
                |                         |
                v                         v
          Network Good               Network Poor
                |                         |
                v                         v
      Storage Transfer             Transfer Appliance
           Service                       |
                |                         |
                +------------+------------+
                             |
                             v
                       CLOUD STORAGE
                             |
                +------------+------------+
                |            |            |
                v            v            v
            Validate      Secure       Monitor
                |
                v
        Incremental Sync
                |
                v
          Final Validation
                |
                v
             Cutover
                |
                v
           Cloud Workload
```

---

# 198. Final Storage Mental Map

```text
                           CLOUD STORAGE
                                |
              +-----------------+-----------------+
              |                 |                 |
              v                 v                 v
          STRUCTURE          STORAGE            ACCESS
              |                 |                 |
              v                 v                 v
           Bucket            Classes              IAM
           Object            Standard             UBLA
           Location          Nearline             PAP
                             Coldline              Signed URL
                             Archive               Policy Document
              |                 |                 |
              +-----------------+-----------------+
                                |
                                v
                         DATA LIFECYCLE
                                |
                +---------------+---------------+
                |               |               |
                v               v               v
            Lifecycle       Versioning       Soft Delete
                |                               |
                v                               v
           Cost Control                    Recovery
                |
                v
             Retention
                |
                v
            Compliance
                                |
                                v
                           ENCRYPTION
                                |
              +-----------------+----------------+
              |                 |                |
              v                 v                v
         Google-managed       CMEK              CSEK
                                |
                                v
                           Cloud KMS
                                |
                                v
                           Key Rotation
                                |
                                v
                             Recovery
                                |
                                v
                           BACKUP / DR
                                |
                                v
                           RPO / RTO
                                |
                                v
                         OBSERVABILITY
                                |
                     +----------+----------+
                     |                     |
                     v                     v
                 Audit Logs            Monitoring
                                |
                                v
                         MIGRATION / OPERATIONS
                                |
                     +----------+----------+
                     |                     |
                     v                     v
             Storage Transfer       Transfer Appliance
                     |
                     v
                 Validation
                     |
                     v
                  Cutover
```

---

# 199. Final Storage Cheat Sheet

```text
BUCKET
    = Container for objects

OBJECT
    = Data + metadata

OBJECT STORAGE
    != Traditional filesystem

STORAGE CLASS
    = Access/pricing behavior

STANDARD
    = Frequent access

NEARLINE
    = Less frequent access

COLDLINE
    = Infrequent access

ARCHIVE
    = Very rare access

LIFECYCLE
    = Automatic object action

RETENTION
    = Prevent premature deletion

VERSIONING
    = Keep noncurrent versions

SOFT DELETE
    = Temporary recovery from deletion

IAM
    = Who can do what?

UBLA
    = Uniform bucket-level IAM access model

PAP
    = Prevent public access

SIGNED URL
    = Temporary scoped/bearer access to a specific request/resource

SIGNED POLICY DOCUMENT
    = Conditional browser/form upload authorization

AUTHENTICATION
    = Who are you?

AUTHORIZATION
    = What can you do?

SERVICE ACCOUNT
    = Workload identity

ADC
    = Application credential discovery mechanism

METADATA SERVER
    = VM identity/token source

PRIVATE GOOGLE ACCESS
    = Private connectivity to Google APIs/services

GOOGLE-MANAGED ENCRYPTION
    = Google-managed server-side encryption

CMEK
    = Customer-managed key through Cloud KMS

CSEK
    = Customer-supplied encryption key

CLIENT-SIDE ENCRYPTION
    = Encrypt before sending to GCS

KEY ROTATION
    = New key version for future cryptographic operations

RPO
    = Maximum acceptable data loss

RTO
    = Maximum acceptable recovery time

STORAGE TRANSFER SERVICE
    = Managed network transfer

TRANSFER APPLIANCE
    = Physical data transfer

AUDIT LOGS
    = Who did what and when?

MONITORING
    = How system behaves

LIFECYCLE + STORAGE CLASS
    = Cost optimization

IAM + UBLA + PAP + ENCRYPTION
    = Strong bucket security

SERVICE ACCOUNT + ADC + METADATA
    = Secure GCE workload access

SIGNED URL
    = External temporary access

SIGNED POLICY
    = Controlled browser upload

CMEK + KMS
    = Customer-controlled server-side encryption

SOFT DELETE + VERSIONING + BACKUP
    = Recovery layers
```

---

# 200. Top 20 Things to Remember Before LevelUp

If time is very limited before the interview, revise these first:

```text
1. Cloud Storage is object storage.

2. Bucket is a container; object is the actual stored data.

3. Storage class should be selected based on access pattern,
   not simply cheapest price.

4. Lifecycle automates future object actions.

5. Retention prevents premature deletion.

6. Versioning protects against object-version changes/overwrites.

7. Soft Delete protects against deletion for a configured recovery window.

8. IAM controls authorization.

9. UBLA simplifies bucket-level access control using IAM.

10. Public Access Prevention prevents public exposure.

11. Signed URLs provide temporary access without requiring
    the recipient to have a Google Cloud account.

12. Signed URLs are bearer-style credentials.

13. Signed Policy Documents are primarily for controlled
    HTML-form POST uploads and support richer upload constraints.

14. Google-managed encryption is the default server-side model.

15. CMEK uses Cloud KMS and gives the customer more control
    over the encryption key lifecycle.

16. Key rotation creates new key versions; it does not
    automatically re-encrypt all existing objects.

17. GCE workloads should use attached service accounts /
    workload identity and short-lived credentials instead
    of long-lived JSON keys whenever possible.

18. Private Google Access provides connectivity, not IAM permission.

19. RPO = acceptable data loss.
    RTO = acceptable recovery time.

20. When troubleshooting, identify:
    WHO + WHICH RESOURCE + WHAT OPERATION + WHICH POLICY
    before changing permissions.
```

---

# 201. Final A3-Level Storage Answer

If the interviewer asks:

> "How would you design Cloud Storage for an enterprise production workload?"

A strong answer:

> "I would start with the data access pattern and business requirements, then select the appropriate bucket location and storage class. For access control, I'd use least-privilege IAM with dedicated service identities and Uniform Bucket-Level Access, and enable Public Access Prevention when public access isn't required. For data protection, I'd use Google-managed encryption by default or CMEK through Cloud KMS when compliance or customer-controlled key management requires it. I'd evaluate retention, Soft Delete, Object Versioning and backups based on recovery and compliance requirements. Lifecycle policies would automate storage-class transitions and cleanup where allowed. For workloads running on GCP, I'd use service identities and short-lived credentials rather than long-lived service-account keys. For external customers, I'd use signed URLs or signed policy documents depending on whether I need temporary object access or constrained browser uploads. Finally, I'd use audit logging and monitoring for visibility and design the solution around cost, performance, RPO, RTO and operational requirements."

---

# 202. Final Mental Model — One Page

```text
                         ┌─────────────────────┐
                         │   CLOUD STORAGE     │
                         └──────────┬──────────┘
                                    |
             +----------------------+----------------------+
             |                      |                      |
             v                      v                      v
         STRUCTURE               ACCESS                 SECURITY
             |                      |                      |
        Bucket/Object          IAM / UBLA               Encryption
        Location               PAP                      CMEK/KMS
                               Signed URL               CSEK
                               Policy Doc               Key Rotation
             |                      |                      |
             +----------------------+----------------------+
                                    |
                                    v
                              DATA LIFECYCLE
                                    |
                   +----------------+----------------+
                   |                |                |
                   v                v                v
              Lifecycle        Versioning       Soft Delete
                   |                                 |
                   v                                 v
                Cost                              Recovery
                   |
                   v
              Retention
                   |
                   v
              Compliance
                                    |
                                    v
                              OPERATIONS
                                    |
               +--------------------+--------------------+
               |                    |                    |
               v                    v                    v
          GCE/GKE/App          Migration             Backup/DR
               |                    |                    |
               v                    v                    v
       Service Identity       STS/Appliance          RPO/RTO
       ADC/Metadata
       Private Google Access
                                    |
                                    v
                             OBSERVABILITY
                                    |
                              +-----+-----+
                              |           |
                              v           v
                          Audit Logs   Monitoring
                                    |
                                    v
                              TROUBLESHOOTING
                                    |
                   +----------------+----------------+
                   |                |                |
                  403              404            Signed URL
                   |                |                |
                  IAM          Object/name        Expiration
                  KMS          configuration       Signature
                  Policy                           Method
```

---

# Storage Module — Final Revision Checklist

```text
[✓] Cloud Storage fundamentals
[✓] Buckets
[✓] Objects
[✓] Object storage vs filesystem
[✓] Locations
[✓] Storage classes
[✓] Lifecycle management
[✓] Object versioning
[✓] Soft Delete
[✓] Retention
[✓] Object holds
[✓] IAM
[✓] Least privilege
[✓] UBLA
[✓] ACL concepts
[✓] Public Access Prevention
[✓] Signed URLs
[✓] Signed URL architecture
[✓] Signed URL commands
[✓] Signed URL upload/download
[✓] Signed Policy Documents
[✓] Signed Policy conditions
[✓] Signed URL vs Signed Policy
[✓] Authentication
[✓] Authorization
[✓] Service accounts
[✓] ADC
[✓] Metadata server
[✓] Private Google Access
[✓] GCE → GCS
[✓] GKE → GCS
[✓] Encryption fundamentals
[✓] Encryption at rest
[✓] Encryption in transit
[✓] Client-side encryption
[✓] Google-managed encryption
[✓] CMEK
[✓] Cloud KMS
[✓] Key rings
[✓] Crypto keys
[✓] Key versions
[✓] Key rotation
[✓] Key compromise
[✓] CSEK
[✓] Encryption comparison
[✓] Backup
[✓] DR
[✓] RPO
[✓] RTO
[✓] Performance
[✓] Cost model
[✓] Cost optimization
[✓] Capacity planning
[✓] Storage Transfer Service
[✓] Transfer Appliance
[✓] Migration architecture
[✓] Migration validation
[✓] GCS CLI commands
[✓] IAM commands
[✓] Lifecycle commands
[✓] Versioning commands
[✓] CMEK commands
[✓] Signed URL commands
[✓] Troubleshooting
[✓] 403
[✓] 404
[✓] Signed URL failures
[✓] CMEK failures
[✓] Audit Logs
[✓] Monitoring
[✓] Enterprise architecture
[✓] LevelUp scenarios
```

---

# Final Principle

> **Cloud Storage is not just "a bucket where files are stored."**
>
> At A3 level, think about the complete lifecycle:
>
> ```text
> STORE
>   ↓
> ACCESS
>   ↓
> PROTECT
>   ↓
> MANAGE
>   ↓
> RECOVER
>   ↓
> MONITOR
>   ↓
> OPTIMIZE
>   ↓
> MIGRATE
> ```
>
> And whenever designing or troubleshooting a Storage solution, ask:
>
> ```text
> WHO?
> WHICH RESOURCE?
> WHAT OPERATION?
> HOW AUTHENTICATED?
> WHAT IAM PERMISSION?
> WHAT OTHER POLICY?
> HOW IS DATA PROTECTED?
> HOW IS DATA RECOVERED?
> WHAT DOES IT COST?
> HOW IS IT MONITORED?
> ```
>
> This is the mindset expected from a Senior System Engineer.
