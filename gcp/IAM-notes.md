# GCP IAM — Complete LevelUp Notes
## Source of Truth | A2 System Engineer → A3 Senior System Engineer

> **Purpose:** This document is designed to remain useful even if revisited years later. Every major IAM concept contains a definition, mental model, important points, examples, commands where useful, architecture, and interview guidance.

---

# 1. IAM — Identity and Access Management

## Definition

**IAM (Identity and Access Management)** is the Google Cloud authorization system used to control **which principal can perform which actions on which resources**.

The simplest mental model is:

```text
WHO
 │
 ▼
Principal
 │
 │ receives
 ▼
ROLE
 │
 │ contains
 ▼
PERMISSIONS
 │
 │ applied to
 ▼
RESOURCE
```

So IAM answers:

> **Who can do what, on which resource?**

### Example

```text
User: alice@example.com
        │
        ▼
roles/storage.objectViewer
        │
        ▼
Cloud Storage bucket
        │
        ▼
Can read objects
```

## Important points

- IAM is primarily about **authorization**.
- Authentication and authorization are different.
- IAM roles contain permissions.
- IAM policies bind principals to roles.
- IAM can be applied at different levels of the resource hierarchy.
- Permissions can be inherited.
- Follow **least privilege**.
- Avoid broad roles when a narrower role is sufficient.

---

# 2. Authentication vs Authorization

## Authentication

Authentication answers:

> **Who are you?**

Examples:

```text
User login
Service account identity
OIDC token
Workload Identity Federation
```

## Authorization

Authorization answers:

> **What are you allowed to do?**

Examples:

```text
storage.objects.get
storage.objects.list
compute.instances.create
```

## Mental model

```text
                 Request
                    │
                    ▼
            Authentication
                    │
             "Who are you?"
                    │
                    ▼
             Authorization
                    │
          "What can you do?"
                    │
                    ▼
                Resource
```

## Important point

A workload can successfully authenticate but still receive:

```text
403 PERMISSION_DENIED
```

because authentication succeeded but authorization failed.

---

# 3. Principal / Identity

## Definition

A **principal** is an identity that can be granted access to Google Cloud resources.

Common principals:

- User
- Group
- Service account
- Federated identity

Examples:

```text
user@example.com

group@example.com

terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

## Mental model

```text
Human
  ↓
User / Group

Application
  ↓
Service Account

External workload
  ↓
Federated Identity
```

## Important points

A principal is the **WHO** in IAM.

When troubleshooting access, first ask:

> **Which principal is actually making the request?**

---

# 4. IAM Permissions

## Definition

A **permission** is an individual authorization capability representing a specific operation.

Examples:

```text
storage.objects.get
storage.objects.list
storage.objects.create
storage.objects.delete

cloudkms.cryptoKeyVersions.useToEncrypt
cloudkms.cryptoKeyVersions.useToDecrypt
```

## Mental model

```text
Permission = one specific capability
```

For example:

```text
storage.objects.get
        │
        └── Read an object
```

## Important points

- Permissions are granular.
- Permissions are generally **included inside roles**.
- Users/workloads are normally granted roles rather than individual permissions.
- Exact permission names matter during troubleshooting.

---

# 5. IAM Roles

## Definition

A **role** is a collection of permissions that can be granted to a principal.

```text
Role
 ├── Permission A
 ├── Permission B
 ├── Permission C
 └── Permission D
```

Example:

```text
roles/storage.objectViewer
        │
        ├── object read-related permissions
        └── related viewing permissions
```

## Important point

Think:

```text
Permission = WHAT
Role       = COLLECTION OF WHATs
Principal  = WHO
Resource   = WHERE
```

---

# 6. Basic Roles

Google Cloud has broad basic roles such as:

```text
roles/viewer
roles/editor
roles/owner
```

## Meaning

### Viewer

Broad read-oriented access.

### Editor

Broad modification access.

### Owner

Very broad administrative ownership capabilities.

## Important points

Basic roles are broad.

For production environments:

> Prefer the narrowest suitable predefined role or a carefully designed custom role.

Do not solve:

```text
Permission denied
```

by automatically giving:

```text
roles/owner
```

---

# 7. Predefined Roles

## Definition

**Predefined roles** are Google-managed roles designed around particular services and responsibilities.

Examples:

```text
roles/storage.objectViewer
roles/storage.objectAdmin
roles/compute.viewer
```

## Why use them?

They are usually preferable because Google maintains them as part of the service's permission model.

## Important points

Use predefined roles when they provide exactly or reasonably closely the access required.

---

# 8. Custom Roles

## Definition

A **custom role** is a role created by an organization containing a selected set of supported permissions.

Example:

```text
custom-role-terraform-storage-kms
       │
       ├── storage.objects.get
       ├── storage.objects.list
       ├── cloudkms.cryptoKeyVersions.useToEncrypt
       └── cloudkms.cryptoKeyVersions.useToDecrypt
```

## Can one role contain permissions from multiple services?

**Yes, where the permissions are supported for custom roles.**

For example, a custom role can conceptually combine:

```text
Cloud Storage permissions
+
Cloud KMS permissions
+
Other supported service permissions
```

## But should you?

Only when they represent one clear responsibility.

Good:

```text
Application encryption workload
        │
        ├── Storage permissions
        └── KMS permissions
```

Bad:

```text
Random application
        │
        ├── BigQuery Admin
        ├── Compute Admin
        ├── Storage Admin
        ├── KMS Admin
        └── GKE Admin
```

## Important points

- Custom roles provide precision.
- Not every Google Cloud permission is eligible for custom roles.
- Check current Google Cloud documentation before building production custom roles.
- Avoid creating huge "everything" custom roles.
- Design roles around a responsibility.

---

# 9. IAM Policy

## Definition

An **IAM policy** is the access-control configuration containing bindings that associate principals with roles.

Conceptually:

```text
IAM Policy
   │
   └── Binding
        ├── Role
        └── Members
```

Example:

```text
Role:
roles/storage.objectViewer

Members:
- user:alice@example.com
- serviceAccount:terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

## Mental model

```text
Principal
    +
Role
    +
Resource
    ↓
Access decision
```

---

# 10. IAM Binding

## Definition

An **IAM binding** connects a role with one or more members/principals.

Example:

```text
Binding
 ├── Role:
 │     roles/storage.objectViewer
 │
 └── Members:
       ├── Alice
       ├── Bob
       └── terraform-sa
```

## Important distinction

```text
Permission
    ↓
inside
Role
    ↓
assigned through
Binding
    ↓
inside
Policy
    ↓
on
Resource
```

---

# 11. Resource Hierarchy

## Definition

Google Cloud resources are organized hierarchically.

Typical hierarchy:

```text
Organization
      │
      ▼
Folder
      │
      ▼
Project
      │
      ▼
Resource
```

Examples of resources:

```text
Cloud Storage bucket
Compute Engine VM
BigQuery dataset
Cloud KMS key
GKE cluster
```

## Important point

IAM can be applied at different levels depending on the resource and policy model.

---

# 12. IAM Inheritance

## Definition

A role granted at a higher level of the resource hierarchy can provide access to resources below that level.

Example:

```text
Project
   │
   └── User Alice
          │
          └── roles/storage.objectViewer
                    │
                    ▼
                 Bucket
                    │
                    ▼
                 Object
```

Alice may therefore have access even when the bucket's own policy does not directly mention Alice.

## Important point

When investigating access:

> **Do not look only at the resource's direct IAM policy.**

Check:

```text
Organization
Folder
Project
Resource
Group membership
Conditions
Deny policies
```

---

# 13. Effective Access

## Definition

**Effective access** is the access a principal actually has after all applicable IAM grants, inheritance, group membership, conditions, and applicable deny rules are considered.

Mental model:

```text
Direct grants
     +
Group grants
     +
Inherited grants
     +
Conditions
     +
Other applicable policy controls
     ↓
Effective access
```

## Important point

A user may have a permission even though you cannot find a direct binding at the resource level.

---

# 14. Groups

## Definition

An IAM group is a collection of users/principals managed together.

Instead of:

```text
Alice → role
Bob   → role
Carol → role
```

use:

```text
developers@example.com
        │
        ▼
roles/viewer
        │
        ▼
Project
```

Members:

```text
Alice
Bob
Carol
```

## Benefits

- Easier onboarding
- Easier offboarding
- Centralized access management
- Less policy clutter
- Better governance

## Important point

For human access at scale:

> Prefer groups where appropriate instead of individually binding every user.

---

# 15. Service Accounts

## Definition

A **service account** is a non-human identity intended for applications, automation, and workloads.

Examples:

```text
Terraform
GitHub Actions
VM
GKE workload
Application
CI/CD pipeline
```

Example:

```text
terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

## Mental model

```text
Human
  ↓
User identity

Workload
  ↓
Service account
```

## Important points

- Service accounts are identities.
- They can receive IAM roles.
- They should have only the permissions required by their workload.
- Dedicated service accounts are usually better than sharing one highly privileged identity across unrelated workloads.

---

# 16. Service Account vs User

| User | Service Account |
|---|---|
| Human identity | Workload identity |
| Used interactively | Used by applications/automation |
| Example: `alice@example.com` | `terraform-sa@PROJECT_ID...` |
| Human lifecycle | Workload lifecycle |

---

# 17. Service Account Keys

## Definition

A service-account key is a credential that can be used to authenticate as the service account.

Traditional architecture:

```text
Service Account
      │
      ▼
Private key / JSON credential
      │
      ▼
Application / CI
      │
      ▼
Google Cloud
```

## Problem

A long-lived private credential can be:

```text
Leaked
Copied
Stored insecurely
Committed accidentally
Exfiltrated
```

If compromised, an attacker may use the service account's permissions.

## Important point

> Prefer keyless authentication such as Workload Identity Federation and short-lived credentials where possible.

---

# 18. Service Account Key Compromise

If a key is compromised:

```text
1. Treat it as compromised
2. Disable/delete the affected key
3. Investigate audit logs
4. Determine whether it was used
5. Assess the service account's permissions
6. Investigate potentially affected resources
7. Replace the authentication design with WIF/impersonation where possible
```

## Important point

Do not wait to "see whether someone uses it."

A leaked credential should be treated as compromised.

---

# 19. Service Account Impersonation

## Definition

**Service account impersonation** allows one principal to temporarily act as another service account.

Architecture:

```text
Caller
   │
   │ permission to impersonate
   ▼
Target Service Account
   │
   │ its IAM roles
   ▼
GCP Resources
```

Example:

```text
alice@example.com
        │
        ▼
Can impersonate
        │
        ▼
terraform-sa
        │
        ▼
Terraform permissions
```

## Why use it?

Instead of:

```text
Developer
   ↓
Permanent production permissions
```

use:

```text
Developer
   ↓
Permission to impersonate
   ↓
Production service account
   ↓
Short-lived credentials
   ↓
Production
```

## Benefits

- No service-account private key required
- Short-lived credentials
- Better separation of duties
- Better auditing
- Reduced long-lived credential exposure

---

# 20. Service Account Impersonation Permission

A commonly used role is:

```text
roles/iam.serviceAccountTokenCreator
```

Important permission commonly associated with access-token impersonation:

```text
iam.serviceAccounts.getAccessToken
```

Mental model:

```text
Caller
   │
   ▼
roles/iam.serviceAccountTokenCreator
   │
   ▼
Target Service Account
   │
   ▼
Temporary credentials
```

## Important distinction

The caller needs permission to impersonate.

The **target service account** separately needs the roles that allow it to perform the actual workload operation.

Example:

```text
Alice
  │
  └── Token Creator on terraform-sa

terraform-sa
  │
  ├── Storage permissions
  ├── Compute permissions
  └── BigQuery permissions
```

---

# 21. Does Impersonation Require Another Service Account?

**No.**

Simple architecture:

```text
User
  ↓
terraform-sa
```

You do not inherently need:

```text
User
  ↓
SA-A
  ↓
SA-B
```

Multiple service accounts are useful only when the architecture requires separation.

---

# 22. gcloud — Grant Service Account Impersonation

```bash
gcloud iam service-accounts add-iam-policy-binding \
  terraform-sa@PROJECT_ID.iam.gserviceaccount.com \
  --member="user:user@example.com" \
  --role="roles/iam.serviceAccountTokenCreator"
```

Mental translation:

```text
user@example.com
       ↓
may impersonate
       ↓
terraform-sa
```

---

# 23. gcloud — Use Impersonation for One Command

Example:

```bash
gcloud storage buckets list \
  --impersonate-service-account=terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

Another:

```bash
gcloud compute instances list \
  --impersonate-service-account=terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

General pattern:

```bash
gcloud <COMMAND> \
  --impersonate-service-account=SERVICE_ACCOUNT_EMAIL
```

---

# 24. gcloud — Set Default Impersonation

```bash
gcloud config set auth/impersonate_service_account \
  terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

Remove:

```bash
gcloud config unset auth/impersonate_service_account
```

## Use case

This is useful when testing how a service account behaves without downloading a service-account key.

---

# 25. Console — Service Account Impersonation

High-level flow:

```text
Google Cloud Console
       ↓
IAM & Admin
       ↓
Service Accounts
       ↓
Select target service account
       ↓
Permissions / IAM
       ↓
Grant access
       ↓
Add caller
       ↓
Grant Service Account Token Creator
       ↓
Save
```

Role:

```text
roles/iam.serviceAccountTokenCreator
```

## Critical understanding

You grant the **caller** permission on the **target service account**.

```text
Caller
   │
   │ Token Creator
   ▼
Target SA
```

---

# 26. Workload Identity Federation — WIF

## Definition

**Workload Identity Federation (WIF)** allows external workloads to authenticate to Google Cloud using a trusted external identity provider without requiring a long-lived Google service-account private key.

Common example:

```text
GitHub Actions
      ↓
GitHub OIDC
      ↓
Google Workload Identity Federation
      ↓
Google service account
      ↓
GCP
```

## Why it matters

Traditional:

```text
GitHub
  ↓
JSON service-account key
  ↓
GCP
```

Modern:

```text
GitHub
  ↓
OIDC
  ↓
WIF
  ↓
Short-lived credentials
  ↓
GCP
```

---

# 27. OIDC

## Definition

**OIDC (OpenID Connect)** is an identity layer built on OAuth 2.0 that allows a workload to present a signed identity token containing claims about that workload.

GitHub Actions can issue an OIDC token.

Conceptually:

```text
GitHub Actions
      ↓
OIDC token
      ↓
Google WIF
```

The token contains identity information such as repository-related claims.

---

# 28. WIF Mental Model

```text
External Workload
       │
       ▼
OIDC Token
       │
       ▼
Workload Identity Provider
       │
       ▼
Validate External Identity
       │
       ▼
Attribute Mapping
       │
       ▼
Attribute Conditions
       │
       ▼
Trusted Federated Identity
       │
       ▼
Service Account
       │
       ▼
GCP APIs
```

---

# 29. Workload Identity Pool

## Definition

A **Workload Identity Pool** is a container for external identities.

Mental model:

```text
Workload Identity Pool
       │
       ├── Provider A
       ├── Provider B
       └── Provider C
```

For example, an organization can have a pool used for external CI/CD identities.

---

# 30. Workload Identity Provider

## Definition

A **Workload Identity Provider** defines how Google trusts an external identity provider.

For GitHub:

```text
GitHub OIDC
     ↓
Workload Identity Provider
     ↓
Google understands how to validate/trust it
```

The provider contains configuration such as issuer information and attribute mapping/conditions.

---

# 31. Attribute Mapping

## Definition

**Attribute mapping** maps claims from the external token into attributes that Google can use for authorization/federation decisions.

Conceptually:

```text
GitHub claim
     ↓
Google attribute
```

Example conceptual mapping:

```text
google.subject=assertion.sub

attribute.repository=assertion.repository

attribute.repository_owner=assertion.repository_owner

attribute.ref=assertion.ref
```

## Important point

Attribute mapping answers:

> **What information from the external identity should Google use?**

---

# 32. Attribute Conditions

## Definition

An **attribute condition** restricts which external identities are trusted.

Example concept:

```text
repository == "my-company/infrastructure"
```

More restrictive:

```text
repository == "my-company/infrastructure"
AND
branch == "main"
```

## Why important?

Without restrictions:

```text
External identity
      ↓
Potential trust
```

With restrictions:

```text
Correct organization
      ↓
Correct repository
      ↓
Correct branch/environment
      ↓
Trusted
```

## Important point

For production WIF:

> Do not blindly trust all identities from the external provider.

Restrict the trust boundary.

---

# 33. WIF Service Account Permission

A commonly used role is:

```text
roles/iam.workloadIdentityUser
```

Mental model:

```text
Federated identity
       │
       ▼
roles/iam.workloadIdentityUser
       │
       ▼
Target service account
```

This allows the federated workload to use the service account according to the configured trust relationship.

---

# 34. WIF vs Service Account Impersonation

This is a very important interview distinction.

## WIF

Answers:

> **How does an external identity become trusted by Google Cloud?**

```text
External identity
       ↓
WIF
       ↓
Federated identity
```

## Impersonation

Answers:

> **How can a caller act as a service account?**

```text
Caller
   ↓
Impersonation permission
   ↓
Service account
```

## Combined architecture

```text
GitHub
   ↓
OIDC
   ↓
WIF
   ↓
Federated identity
   ↓
Service account
   ↓
IAM roles
   ↓
GCP
```

---

# 35. GitHub Actions → GCP Architecture

```text
                     GitHub
                        │
                        ▼
                 GitHub Actions
                        │
                       OIDC
                        │
                        ▼
          Workload Identity Provider
                        │
                Attribute Mapping
                        │
               Attribute Conditions
                        │
                        ▼
                 terraform-sa
                        │
                  IAM roles
                        │
           ┌────────────┼────────────┐
           ▼            ▼            ▼
          GCS          GCE          GKE
```

## Important security controls

- Trust only the intended GitHub organization/repository.
- Restrict branch/environment where appropriate.
- Use dedicated service accounts.
- Use least privilege.
- Avoid service-account JSON keys.

---

# 36. WIF + Terraform Architecture

```text
Developer
    ↓
GitHub PR
    ↓
Review
    ↓
Merge
    ↓
GitHub Actions
    ↓
OIDC
    ↓
WIF
    ↓
Attribute conditions
    ↓
Terraform service account
    ↓
Least-privilege IAM
    ↓
GCP
```

Terraform may manage:

```text
GCS
GCE
GKE
IAM
BigQuery
VPC
etc.
```

The Terraform service account should receive only what the pipeline actually needs.

---

# 37. Dev / Stage / Prod IAM Separation

Recommended mental model:

```text
                 GitHub
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         DEV      STAGE      PROD
          │         │         │
          ▼         ▼         ▼
       dev-sa    stage-sa    prod-sa
          │         │         │
          ▼         ▼         ▼
         GCP       GCP       GCP
```

## Why?

If the development identity is compromised:

```text
dev-sa compromised
       ↓
Only development blast radius
```

rather than:

```text
dev-sa compromised
       ↓
Production access
```

---

# 38. IAM Conditions

## Definition

IAM Conditions allow access to be granted conditionally rather than unconditionally.

Conceptually:

```text
Principal
   +
Role
   +
Condition
   ↓
Conditional access
```

Conditions can use supported context such as resource/request/time-related attributes.

## Example concept

```text
Allow access
only under condition X
```

## Important point

IAM Conditions are useful when a role should not be valid in every circumstance.

---

# 39. IAM Deny Policies

## Definition

An IAM deny policy can explicitly prevent selected permissions from being used.

Mental model:

```text
Allow policy
     ↓
Permission may be granted

Applicable Deny
     ↓
Permission blocked
```

Therefore:

```text
User has a role
       ↓
Still receives PERMISSION_DENIED
       ↓
Check applicable deny policies
```

## Important point

Do not assume:

> "The user has the role, therefore access must work."

Effective authorization can be affected by more than a single allow binding.

---

# 40. IAM Policy Troubleshooter

## Definition

Policy Troubleshooter helps investigate why a principal can or cannot access a resource.

Mental model:

```text
Principal
+
Permission
+
Resource
      ↓
Policy Troubleshooter
      ↓
Authorization analysis
```

## Typical use case

```text
Terraform
   ↓
403 PERMISSION_DENIED
   ↓
Which permission is missing?
   ↓
Why isn't the role effective?
```

Use it before blindly granting a broad role.

---

# 41. IAM Policy Analyzer

## Definition

Policy Analyzer helps analyze who has access to what.

Think:

> **Who can access this resource?**

or:

> **What can this principal access?**

Useful for:

- Security reviews
- Access audits
- Finding unexpected access
- Understanding effective access

---

# 42. IAM Recommender

## Definition

IAM Recommender provides recommendations based on observed usage to help reduce excessive permissions.

Mental model:

```text
Granted access
      +
Observed usage
      ↓
Recommendation
      ↓
Potential reduction
```

## Important point

Recommendations should be validated against business requirements before applying them.

---

# 43. Cloud Audit Logs

## Definition

Cloud Audit Logs provide records of activity in Google Cloud.

They help answer:

```text
Who?
What?
When?
Where?
```

Example:

```text
Who changed IAM?
When?
Which policy changed?
Which API call occurred?
```

Mental model:

```text
GCP activity
     ↓
Cloud Audit Logs
     ↓
Cloud Logging
     ↓
Investigation / Monitoring
```

---

# 44. Admin Activity vs Data Access

## Admin Activity

Generally relates to administrative/configuration actions.

Examples:

```text
Change IAM policy
Create resource
Delete resource
Modify configuration
```

Mental model:

> **Someone changed configuration.**

## Data Access

Relates to access to data/resources where applicable and where the relevant logging is available/enabled.

Mental model:

> **Someone accessed data.**

## Important point

Do not assume all data access logging behaves identically across every service. Service-specific logging behavior and configuration matter.

---

# 45. IAM Policy vs Audit Logs

Very important distinction:

```text
IAM Policy
     ↓
CURRENT ACCESS STATE
```

while:

```text
Audit Logs
     ↓
ACTIVITY / HISTORY
```

Example:

```text
Current:
Bob → storage.objectViewer

History:
Alice changed Bob's access yesterday
```

---

# 46. Access Review

## Definition

An access review is a periodic process of checking whether identities still need their current permissions.

Questions:

```text
Who has access?
Why?
Do they still need it?
Is the access excessive?
Is the identity still active?
```

Process:

```text
Inventory identities
      ↓
Review roles
      ↓
Review actual usage
      ↓
Identify excessive access
      ↓
Validate requirement
      ↓
Remove unnecessary access
      ↓
Monitor
```

---

# 47. Permission Creep

## Definition

Permission creep occurs when users/workloads accumulate permissions over time and never lose old access.

Example:

```text
Developer
   ↓
Viewer
   ↓
Storage Admin
   ↓
Compute Admin
   ↓
BigQuery Admin
```

Result:

```text
Excess privilege
      ↓
Large security blast radius
```

Solutions:

- Least privilege
- Periodic access reviews
- Policy analysis
- IAM Recommender
- Role cleanup
- Group governance

---

# 48. Orphaned Service Accounts

## Definition

An orphaned service account is an identity that remains after its workload/application is gone or no longer requires it.

Example:

```text
Application deleted
       ↓
Old service account remains
       ↓
Old permissions remain
       ↓
Unnecessary security exposure
```

## Important point

Regularly review unused service accounts.

---

# 49. Break-Glass Access

## Definition

A **break-glass account** is an emergency administrative identity used only when normal administrative access is unavailable.

Normal:

```text
Administrator
    ↓
Normal access
```

Emergency:

```text
IAM incident / lockout
       ↓
Break-glass account
       ↓
Emergency access
```

## Security requirements

- Do not use daily.
- Protect strongly.
- Monitor closely.
- Alert when used.
- Investigate every use.

---

# 50. Separation of Duties

## Definition

Separation of duties means dividing sensitive responsibilities between different identities or people so one identity does not have unnecessary control over the entire process.

Example:

```text
Developer
   ↓
Can deploy

Security team
   ↓
Controls security policies

Production admin
   ↓
Controls production IAM
```

This reduces the risk of a single compromised identity becoming a complete production compromise.

---

# 51. Least Privilege

## Definition

**Least privilege** means giving a principal only the permissions necessary to perform its required task.

Bad:

```text
Terraform
   ↓
roles/owner
```

Better:

```text
Terraform
   ↓
Specific roles
   ↓
Only required resources/operations
```

## Golden rule

> **Do not grant access because it is convenient; grant access because it is required.**

---

# 52. IAM Troubleshooting Methodology

When a user/workload gets:

```text
403 PERMISSION_DENIED
```

follow this sequence.

```text
1. WHO?
   ↓
Identify principal

2. AUTHENTICATED?
   ↓
Verify identity/authentication

3. WHAT?
   ↓
Identify exact API operation

4. WHICH PERMISSION?
   ↓
Find exact permission required

5. WHERE?
   ↓
Identify resource

6. WHICH ROLE?
   ↓
Find role containing permission

7. WHICH SCOPE?
   ↓
Check resource/project/folder/org

8. INHERITED?
   ↓
Check hierarchy

9. GROUP?
   ↓
Check group membership

10. CONDITION?
    ↓
Check IAM Conditions

11. DENY?
    ↓
Check applicable deny policies

12. ANALYZE
    ↓
Policy Troubleshooter

13. AUDIT
    ↓
Check Audit Logs

14. FIX MINIMALLY
```

---

# 53. Terraform 403 Troubleshooting

Example:

```text
Terraform
   ↓
GCP API
   ↓
403 PERMISSION_DENIED
```

Checklist:

```text
Identify Terraform service account
        ↓
Identify exact operation
        ↓
Identify required permission
        ↓
Check service account roles
        ↓
Check scope
        ↓
Check inherited access
        ↓
Check conditions
        ↓
Check deny policies
        ↓
Use Policy Troubleshooter
        ↓
Grant minimum required access
```

## Important point

Do not immediately grant:

```text
roles/owner
```

---

# 54. GitHub WIF Authentication Failure

Check the chain:

```text
GitHub
  ↓
OIDC token
  ↓
Provider
  ↓
Issuer
  ↓
Audience
  ↓
Attribute mapping
  ↓
Attribute condition
  ↓
WIF IAM binding
  ↓
Target service account
```

Possible problems:

- Wrong provider
- Wrong issuer
- Wrong audience
- Incorrect mapping
- Condition mismatch
- Repository mismatch
- Branch/environment mismatch
- Incorrect service-account IAM binding

---

# 55. GitHub Authentication Works but Terraform Gets 403

This is a very important diagnostic distinction.

If:

```text
GitHub
 ↓
WIF
 ↓
Service account
```

works, but Terraform receives:

```text
403
```

then authentication/federation may already be working.

Focus on:

```text
Service account IAM
        ↓
Required permission
        ↓
Resource scope
        ↓
Conditions
        ↓
Deny policies
```

---

# 56. Employee Leaves the Organization

IAM lifecycle should include:

```text
Employee leaves
       ↓
Disable/remove identity
       ↓
Remove direct access
       ↓
Review group membership
       ↓
Review service-account impersonation
       ↓
Review exceptional access
       ↓
Verify effective access
```

---

# 57. Production IAM Changed Unexpectedly

Investigation:

```text
Unexpected IAM change
       ↓
Cloud Audit Logs
       ↓
Who changed it?
       ↓
When?
       ↓
What changed?
       ↓
Was it authorized?
       ↓
Revert/restrict if necessary
       ↓
Investigate
       ↓
Improve preventive controls
```

---

# 58. Secure CI/CD Authentication Pattern

Recommended:

```text
GitHub Actions
      ↓
OIDC
      ↓
WIF
      ↓
Restrictive attribute conditions
      ↓
Dedicated service account
      ↓
Least-privilege IAM
      ↓
Terraform
      ↓
GCP
```

Avoid:

```text
GitHub Actions
      ↓
Long-lived JSON key
      ↓
Production GCP
```

---

# 59. Service Account Impersonation vs WIF — Interview Table

| Topic | Service Account Impersonation | WIF |
|---|---|---|
| Main purpose | Act as another service account | Trust external workload identity |
| Typical caller | User/workload | External workload |
| Credential approach | Short-lived credentials | Federation / short-lived credentials |
| Common role | `roles/iam.serviceAccountTokenCreator` | `roles/iam.workloadIdentityUser` |
| Example | Developer → Terraform SA | GitHub → GCP |
| Can be combined? | Yes | Yes |

---

# 60. Important IAM Roles Cheat Sheet

| Role | Main purpose |
|---|---|
| `roles/viewer` | Broad read-oriented basic role |
| `roles/editor` | Broad modification basic role |
| `roles/owner` | Very broad ownership/admin role |
| `roles/iam.serviceAccountTokenCreator` | Common role for service-account token creation/impersonation |
| `roles/iam.workloadIdentityUser` | Common role for WIF workload access to a service account |
| `roles/storage.objectViewer` | Object viewing |
| `roles/storage.objectAdmin` | Object administration |

> **Always verify the current Google Cloud IAM documentation for exact permissions and role behavior before production implementation.**

---

# 61. gcloud — Common IAM Commands

## Set project

```bash
gcloud config set project PROJECT_ID
```

## List service accounts

```bash
gcloud iam service-accounts list
```

## Create service account

```bash
gcloud iam service-accounts create terraform-sa \
  --display-name="Terraform Service Account"
```

## Describe service account

```bash
gcloud iam service-accounts describe \
  terraform-sa@PROJECT_ID.iam.gserviceaccount.com
```

## View project IAM policy

```bash
gcloud projects get-iam-policy PROJECT_ID
```

---

# 62. gcloud — Grant a Project Role

```bash
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:terraform-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="ROLE_NAME"
```

Example:

```bash
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:terraform-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.admin"
```

> Use the actual minimum role required rather than blindly using `storage.admin`.

---

# 63. gcloud — Create WIF Pool

```bash
gcloud iam workload-identity-pools create POOL_ID \
  --location="global" \
  --display-name="GitHub Actions Pool"
```

Describe:

```bash
gcloud iam.workload-identity-pools describe POOL_ID \
  --location="global"
```

Provider creation is provider-specific; the conceptual flow is:

```text
Create pool
   ↓
Create OIDC provider
   ↓
Configure issuer
   ↓
Configure mapping
   ↓
Configure conditions
```

---

# 64. gcloud — WIF OIDC Provider

Conceptual command:

```bash
gcloud iam workload-identity-pools providers create-oidc PROVIDER_ID \
  --location="global" \
  --workload-identity-pool="POOL_ID" \
  --issuer-uri="OIDC_ISSUER_URI" \
  --attribute-mapping="google.subject=assertion.sub"
```

Production configuration normally includes additional mappings and restrictive conditions.

Example conceptual mappings:

```text
google.subject=assertion.sub
attribute.repository=assertion.repository
attribute.repository_owner=assertion.repository_owner
attribute.ref=assertion.ref
```

---

# 65. gcloud — Grant WIF Access to Service Account

Conceptually:

```bash
gcloud iam service-accounts add-iam-policy-binding \
  terraform-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="FEDERATED_PRINCIPAL"
```

The exact federated principal syntax depends on the pool/provider and attribute design.

---

# 66. Console — Create Service Account

```text
Google Cloud Console
        ↓
IAM & Admin
        ↓
Service Accounts
        ↓
Create Service Account
        ↓
Service account name / ID
        ↓
Create
        ↓
Grant only required roles
```

---

# 67. Console — Grant IAM Role

```text
IAM & Admin
      ↓
IAM
      ↓
Grant access
      ↓
Add principal
      ↓
Select role
      ↓
Save
```

Always confirm:

```text
WHO?
WHAT ROLE?
WHERE?
WHY?
```

---

# 68. Console — WIF Setup

```text
IAM & Admin
      ↓
Workload Identity Federation
      ↓
Create Pool
      ↓
Choose location
      ↓
Create Pool
      ↓
Create Provider
      ↓
Choose OIDC
      ↓
Configure issuer
      ↓
Configure attribute mapping
      ↓
Configure attribute conditions
      ↓
Create
      ↓
Grant access to target service account
```

---

# 69. WIF Production Checklist

Before trusting a GitHub workload:

```text
[ ] Pool created
[ ] Provider created
[ ] Correct OIDC issuer
[ ] Correct audience configuration
[ ] Required claims mapped
[ ] Repository restricted
[ ] Organization restricted
[ ] Branch/environment restricted where required
[ ] Dedicated service account
[ ] roles/iam.workloadIdentityUser configured appropriately
[ ] Service account has least-privilege roles
[ ] No long-lived service-account key
[ ] Audit/monitoring considered
```

---

# 70. IAM Security Architecture — Complete

```text
                    HUMAN / WORKLOAD
                           │
                           ▼
                    AUTHENTICATION
                           │
            ┌──────────────┴──────────────┐
            │                             │
        User Login                     OIDC/WIF
            │                             │
            ▼                             ▼
       User Identity                Federated Identity
            │                             │
            └──────────────┬──────────────┘
                           ▼
                    Service Account
                           │
                           ▼
                     IAM POLICY
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  Roles        Conditions
                    │             │
                    ▼             ▼
              Permissions      Restrictions
                    │             │
                    └──────┬──────┘
                           ▼
                        RESOURCE
                           │
                           ▼
                     AUDIT LOGS
                           │
                           ▼
                  MONITORING / REVIEW
                           │
                           ▼
                  LEAST PRIVILEGE
```

---

# 71. IAM + GitHub + Terraform — Complete Architecture

```text
Developer
    │
    ▼
GitHub Repository
    │
    ▼
Pull Request
    │
    ▼
Review
    │
    ▼
Main Branch
    │
    ▼
GitHub Actions
    │
    │ OIDC
    ▼
Workload Identity Federation
    │
    ├── Provider
    ├── Attribute Mapping
    └── Attribute Conditions
    │
    ▼
Terraform Service Account
    │
    ├── Least Privilege IAM
    │
    ├── State Access
    │
    └── Infrastructure Permissions
    │
    ▼
Google Cloud
    │
    ├── GCS
    ├── GKE
    ├── GCE
    ├── BigQuery
    └── Other resources
    │
    ▼
Cloud Audit Logs
    │
    ▼
Monitoring / Security
```

---

# 72. Five Questions to Ask in Any IAM Problem

This is the most useful mental shortcut.

## ① WHO?

```text
Who is making the request?
```

## ② WHAT?

```text
What exact operation is being attempted?
```

## ③ WHERE?

```text
Which resource is being accessed?
```

## ④ WHY?

```text
Why does this principal need the access?
```

## ⑤ WHY NOT?

If access is denied:

```text
Why is authorization failing?
```

Then check:

```text
Role
Inheritance
Group
Condition
Deny
Resource
Troubleshooter
Audit Logs
```

---

# 73. IAM Design Methodology

When designing IAM for a new application:

```text
Step 1
Identify workload/users
        ↓
Step 2
Identify resources
        ↓
Step 3
Identify exact operations
        ↓
Step 4
Identify required permissions
        ↓
Step 5
Choose predefined role
        ↓
Step 6
Use custom role only if necessary
        ↓
Step 7
Apply at minimum scope
        ↓
Step 8
Use federation/impersonation instead of keys
        ↓
Step 9
Add conditions for sensitive trust boundaries
        ↓
Step 10
Enable monitoring/auditing
        ↓
Step 11
Review periodically
```

---

# 74. LevelUp Interview Scenario — Secure GitHub Authentication

### Question

> How would you securely authenticate GitHub Actions to GCP?

### Strong answer

> I would avoid storing a long-lived Google service-account JSON key in GitHub. I would use GitHub OIDC with Google Cloud Workload Identity Federation. I would restrict the trust using attribute conditions such as the expected organization, repository and, where appropriate, branch or environment. The workflow would use a dedicated service account with least-privilege permissions. This gives us short-lived credentials and reduces the risk associated with long-lived service-account keys.

Architecture:

```text
GitHub Actions
      ↓
OIDC
      ↓
WIF
      ↓
Attribute Conditions
      ↓
Dedicated Service Account
      ↓
Least Privilege
      ↓
GCP
```

---

# 75. LevelUp Interview Scenario — Why Not Owner?

### Question

> Terraform needs many permissions. Why not give it Owner?

### Strong answer

> Owner is unnecessarily broad and violates least privilege. I would identify the resources and operations Terraform actually needs, use suitable predefined roles where possible, and use custom roles only when predefined roles cannot provide the required level of precision.

---

# 76. LevelUp Interview Scenario — User Has No Direct Bucket Role

### Question

> I don't see a bucket-level IAM role for the user, but the user can access the bucket. Why?

Check:

```text
Organization IAM
       ↓
Folder IAM
       ↓
Project IAM
       ↓
Bucket IAM
       ↓
Group membership
       ↓
Conditions
       ↓
Other applicable grants
       ↓
Deny policies
```

Likely explanation:

> The user may have inherited access from a higher resource level or through a group.

---

# 77. LevelUp Interview Scenario — 403 From Terraform

### Answer framework

```text
Who?
 ↓
Terraform service account

What?
 ↓
Exact Terraform operation

Permission?
 ↓
Exact missing permission

Where?
 ↓
Resource

Role?
 ↓
Role containing permission

Scope?
 ↓
Correct project/resource?

Inherited?
 ↓
Check hierarchy

Conditions?
 ↓
Check

Deny?
 ↓
Check

Troubleshooter?
 ↓
Analyze

Fix
 ↓
Grant minimum required permission
```

---

# 78. LevelUp Interview Scenario — WIF Auth Works but Access Fails

### Question

> GitHub successfully authenticates using WIF, but Terraform receives 403. What do you check?

Answer:

> I would distinguish authentication from authorization. If WIF authentication succeeded, I would inspect the target service account's IAM roles, the exact required permission, resource scope, IAM conditions, inherited policies, and applicable deny policies. I would use Policy Troubleshooter to identify why the permission is not effective.

---

# 79. LevelUp Interview Scenario — Service Account Impersonation

### Question

> Why would you use service-account impersonation?

Answer:

> It allows a user or workload to act as a service account using short-lived credentials rather than distributing a long-lived private key. The caller gets permission to impersonate the service account, while the service account retains the actual workload permissions.

Architecture:

```text
Caller
  ↓
Service Account Token Creator
  ↓
Target SA
  ↓
Short-lived credential
  ↓
GCP
```

---

# 80. LevelUp Interview Scenario — WIF vs Impersonation

### Question

> What's the difference between WIF and service-account impersonation?

Answer:

> WIF establishes trust between Google Cloud and an external identity provider. Service-account impersonation allows a caller to act as a service account. They can be combined so that an external workload authenticates through WIF and then uses a dedicated service account with the required permissions.

---

# 81. LevelUp Interview Scenario — Compromised Service Account Key

### Question

> A service-account key was leaked. What do you do?

Answer:

```text
Treat as compromised
        ↓
Disable/delete key
        ↓
Check Audit Logs
        ↓
Investigate usage
        ↓
Assess SA permissions
        ↓
Assess affected resources
        ↓
Rotate/replace credentials
        ↓
Move toward WIF/impersonation
        ↓
Review why key existed
```

---

# 82. LevelUp Interview Scenario — IAM Governance

### Question

> How do you prevent IAM permission creep?

Answer:

```text
Least privilege
      +
Groups
      +
Dedicated service accounts
      +
Periodic access reviews
      +
Policy Analyzer
      +
IAM Recommender
      +
Audit Logs
      +
Monitoring
```

---

# 83. IAM Golden Rules

## Rule 1

> **Authentication tells you WHO. Authorization tells you WHAT they can do.**

## Rule 2

> **Principal = WHO. Permission = WHAT. Resource = WHERE.**

## Rule 3

> **Role = collection of permissions.**

## Rule 4

> **Policy/binding connects principals to roles.**

## Rule 5

> **Always consider inherited access.**

## Rule 6

> **Do not solve every 403 by granting Owner/Admin.**

## Rule 7

> **Prefer least privilege.**

## Rule 8

> **Avoid long-lived service-account keys when keyless alternatives exist.**

## Rule 9

> **WIF establishes trust for external workloads.**

## Rule 10

> **Impersonation allows a caller to act as a service account.**

## Rule 11

> **Use conditions to narrow trust boundaries where appropriate.**

## Rule 12

> **Use Audit Logs to understand what happened, not IAM policy alone.**

## Rule 13

> **Review access continuously; IAM is not a one-time configuration.**

---

# 84. Final IAM Mental Map

Memorize this:

```text
                         IAM
                          │
                          ▼
                    WHO IS CALLING?
                          │
                          ▼
                      PRINCIPAL
                          │
                          ▼
                  AUTHENTICATION
                          │
                          ▼
                    AUTHORIZATION
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
            ROLE                  CONDITIONS
              │                       │
              ▼                       ▼
        PERMISSIONS              RESTRICTIONS
              │                       │
              └───────────┬───────────┘
                          ▼
                       RESOURCE
                          │
                          ▼
                    ACCESS DECISION
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
           ALLOWED                  DENIED
              │                       │
              ▼                       ▼
          USE DATA              TROUBLESHOOT
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
                 IAM Policy       Conditions          Deny
                    │                 │                 │
                    └─────────────────┼─────────────────┘
                                      ▼
                              Policy Troubleshooter
                                      │
                                      ▼
                                Audit Logs
                                      │
                                      ▼
                              Access Review
                                      │
                                      ▼
                              Least Privilege
```

---

# 85. Final WIF Mental Map

```text
                 EXTERNAL WORKLOAD
                         │
                         ▼
                       OIDC
                         │
                         ▼
              WORKLOAD IDENTITY POOL
                         │
                         ▼
                     PROVIDER
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       ATTRIBUTE MAPPING      ATTRIBUTE CONDITION
              │                     │
              └──────────┬──────────┘
                         ▼
                TRUSTED FEDERATED ID
                         │
                         ▼
                  SERVICE ACCOUNT
                         │
                         ▼
                  IAM WORKLOAD ID
                         │
                         ▼
                   IAM ROLES
                         │
                         ▼
                      GCP
```

---

# 86. Final Service Account Impersonation Mental Map

```text
             CALLER
          User / Workload
                │
                │
                ▼
     Service Account Token Creator
                │
                ▼
       TARGET SERVICE ACCOUNT
                │
                ▼
        Short-lived credentials
                │
                ▼
             GCP APIs
                │
                ▼
           Actual resources
```

---

# 87. Final IAM + CI/CD Mental Map

```text
Developer
   │
   ▼
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
OIDC
   │
   ▼
WIF
   │
   ▼
Attribute Conditions
   │
   ▼
Dedicated Service Account
   │
   ▼
Least Privilege IAM
   │
   ▼
Terraform
   │
   ▼
GCP Resources
   │
   ▼
Audit Logs
   │
   ▼
Monitoring
   │
   ▼
Periodic Review
```

---

# 88. Final LevelUp Checklist

Before considering IAM complete, you should be able to explain in your own words:

- [ ] What IAM is
- [ ] Authentication vs authorization
- [ ] Principal
- [ ] Permission
- [ ] Role
- [ ] IAM binding
- [ ] IAM policy
- [ ] Basic roles
- [ ] Predefined roles
- [ ] Custom roles
- [ ] Multiple-service permissions in custom roles
- [ ] Organization/folder/project/resource hierarchy
- [ ] IAM inheritance
- [ ] Effective access
- [ ] Groups
- [ ] Service accounts
- [ ] Service-account keys
- [ ] Key compromise response
- [ ] Least privilege
- [ ] Separation of duties
- [ ] Service-account impersonation
- [ ] `roles/iam.serviceAccountTokenCreator`
- [ ] `iam.serviceAccounts.getAccessToken`
- [ ] gcloud impersonation commands
- [ ] Console impersonation flow
- [ ] OIDC
- [ ] Workload Identity Federation
- [ ] Workload Identity Pool
- [ ] Workload Identity Provider
- [ ] Attribute mapping
- [ ] Attribute conditions
- [ ] `roles/iam.workloadIdentityUser`
- [ ] GitHub Actions → GCP
- [ ] WIF + Terraform
- [ ] WIF + impersonation
- [ ] Dev/stage/prod separation
- [ ] IAM Conditions
- [ ] IAM Deny
- [ ] Policy Troubleshooter
- [ ] Policy Analyzer
- [ ] IAM Recommender
- [ ] Cloud Audit Logs
- [ ] Admin Activity
- [ ] Data Access
- [ ] Access reviews
- [ ] Permission creep
- [ ] Orphaned service accounts
- [ ] Break-glass access
- [ ] IAM troubleshooting
- [ ] IAM architecture design
- [ ] LevelUp interview scenarios

---

# 89. One-Page Revision Sheet

```text
IAM
│
├── WHO?
│    └── Principal
│         ├── User
│         ├── Group
│         ├── Service Account
│         └── Federated Identity
│
├── AUTHENTICATION
│    ├── User authentication
│    ├── Service account
│    ├── OIDC
│    └── WIF
│
├── AUTHORIZATION
│    ├── Permission
│    ├── Role
│    ├── Binding
│    ├── Policy
│    ├── Conditions
│    └── Deny
│
├── WHERE?
│    ├── Organization
│    ├── Folder
│    ├── Project
│    └── Resource
│
├── WORKLOAD SECURITY
│    ├── Avoid long-lived keys
│    ├── WIF
│    ├── Impersonation
│    └── Short-lived credentials
│
├── GOVERNANCE
│    ├── Audit Logs
│    ├── Policy Analyzer
│    ├── Policy Troubleshooter
│    ├── IAM Recommender
│    ├── Access Reviews
│    └── Least Privilege
│
└── TROUBLESHOOTING
     ├── Who?
     ├── What?
     ├── Where?
     ├── Role?
     ├── Inheritance?
     ├── Group?
     ├── Condition?
     ├── Deny?
     ├── Troubleshooter?
     └── Audit Logs?
```

---

# 90. Final Definition

> **Google Cloud IAM is the authorization framework that determines which principals can perform which operations on which resources. A strong IAM design combines appropriate identities, least-privilege roles and permissions, resource-scoped policies, conditional controls, short-lived/federated authentication, service-account impersonation where appropriate, and continuous auditing and access review.**

