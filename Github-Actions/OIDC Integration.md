# Technical Reference: OpenID Connect (OIDC) Integration in GitHub Actions

---

## 1. OIDC Fundamentals & The Security Problem

### Static Credentials (Legacy Pattern)

Historically, authenticating GitHub Actions to cloud providers required storing static, long-lived access keys (e.g., `AWS_ACCESS_KEY_ID`, GCP Service Account JSON keys) inside GitHub Secrets.

```text
❌ STATIC CREDENTIAL PATTERN (High Risk):
[Long-lived Secret stored in GitHub] ────► [Cloud Provider API]
- Secrets never rotate automatically.
- Leakage (in logs, compromised dependencies) grants permanent cloud access.

```

### OIDC Keyless Pattern (Modern Architecture)

OpenID Connect (OIDC) replaces static long-lived keys with **short-lived JSON Web Tokens (JWTs)** issued dynamically per workflow run.

```text
✅ OIDC KEYLESS PATTERN (Zero Long-Lived Secrets):

┌──────────────────┐               1. Request JWT (id-token: write)               ┌────────────────────────┐
│ GitHub Actions   │ ───────────────────────────────────────────────────────────► │ GitHub OIDC Identity   │
│ Runner           │ ◄─────────────────────────────────────────────────────────── │ Provider (IdP)         │
└────────┬─────────┘               2. Issue Signed Claims JWT                     └────────────────────────┘
         │
         │ 3. Exchange JWT for Temporary Token (sts:AssumeRoleWithWebIdentity)
         ▼
┌──────────────────┐
│ Cloud Provider   │ 4. Verify Signature & Sub Claims vs Trust Policy
│ (AWS / GCP /     │ ───────────────────────────────────────────────────────────┐
│ HashiCorp Vault) │                                                            │
└──────────────────┘                                                            ▼
                                                                  ┌────────────────────────┐
                                                                  │ Short-Lived Token /    │
                                                                  │ Session (e.g., 1 hour) │
                                                                  └────────────────────────┘

```

---

## 2. Token Exchange Protocol Mechanics & Claims Structure

When a workflow step runs with `permissions: id-token: write`, the GitHub Actions runtime requests an OIDC token signed by GitHub's IdP (`[https://token.actions.githubusercontent.com](https://token.actions.githubusercontent.com)`).

### JWT Payload & Claims Breakdown

```json
{
  "iss": "https://token.actions.githubusercontent.com",
  "aud": "sts.amazonaws.com",
  "sub": "repo:Your-Org/payment-service:ref:refs/heads/main",
  "actor": "dinesh-sontenam",
  "repository": "Your-Org/payment-service",
  "repository_owner": "Your-Org",
  "run_id": "893412567",
  "job_workflow_ref": "Your-Org/payment-service/.github/workflows/deploy.yml@refs/heads/main",
  "environment": "production",
  "exp": 1773400000,
  "nbf": 1773396400
}

```

* **`iss` (Issuer):** Identifies GitHub as the token authority.
* **`aud` (Audience):** The target service accepting the token (e.g., `sts.amazonaws.com`, `[https://vault.example.com](https://vault.example.com)`).
* **`sub` (Subject):** **The primary security assertion.** Uniquely pinpoints the organizational scope, repository, event type, or git reference.

---

## 3. GitHub Permissions Scope (`id-token: write`)

By default, workflow runs receive read-only access to GitHub tokens. To request an OIDC JWT from GitHub's IdP, you must explicitly enable `id-token: write`.

> **Critical Rule:** Explicitly defining any permission setting strips away all default permissions. You must re-declare `contents: read` so the runner can clone your repository.

```yaml
permissions:
  id-token: write # Mandatory to request the OIDC JWT token
  contents: read  # Mandatory to grant repository checkout access

```

---

## 4. Platform Implementation Blueprint 2: Google Cloud Platform (GCP)

GCP uses **Workload Identity Federation** to map external GitHub JWT claims to GCP Service Accounts without JSON keys.

### Step 1: Workload Identity Setup (gcloud CLI)

```bash
# 1. Create a Workload Identity Pool
gcloud iam workload-identity-pools create "github-pool" \
  --project="your-gcp-project-id" \
  --location="global" \
  --display-name="GitHub Actions Pool"

# 2. Add an OIDC Provider to the Pool
gcloud iam workload-identity-pools providers create-oidc "github-provider" \
  --project="your-gcp-project-id" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository"

# 3. Allow GitHub repository to impersonate Service Account
gcloud iam service-accounts add-iam-policy-binding "ci-deployer@your-gcp-project-id.iam.gserviceaccount.com" \
  --project="your-gcp-project-id" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-pool/attribute.repository/Your-Org/payment-service"

```

### Step 2: GitHub Actions Workflow (`google-github-actions/auth`)

```yaml
name: GCP OIDC Deployment

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  deploy-gcp:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/123456789/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
          service_account: 'ci-deployer@your-gcp-project-id.iam.gserviceaccount.com'

      - name: Test GCP Access
        run: gcloud gke clusters list

```

---

## 5. Platform Implementation Blueprint 3: HashiCorp Vault Integration

HashiCorp Vault supports native JWT/OIDC authentication, letting GitHub Actions pull secrets directly into memory.

### Step 1: Vault JWT Auth Role Setup

```hcl
# Configure Vault JWT auth backend for GitHub Actions
resource "vault_jwt_auth_backend" "github" {
  path               = "jwt"
  bound_issuer       = "https://token.actions.githubusercontent.com"
  oidc_discovery_url = "https://token.actions.githubusercontent.com"
}

# Define role tied to specific repository & environment
resource "vault_jwt_auth_backend_role" "ci_role" {
  backend        = vault_jwt_auth_backend.github.path
  role_name      = "ci-role"
  token_policies = ["deploy-policy"]
  user_claim     = "sub"

  bound_claims = {
    repository  = "Your-Org/payment-service"
    environment = "production"
  }
}

```

### Step 2: GitHub Actions Workflow (`hashicorp/vault-action`)

```yaml
name: Vault Secret Retrieval

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  retrieve-secrets:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Fetch Secrets from Vault
        uses: hashicorp/vault-action@v3
        with:
          url: https://vault.yourcompany.com:8200
          method: jwt
          role: ci-role
          secrets: |
            secret/data/production/db password | DB_PASSWORD ;
            secret/data/production/api key | API_KEY

      - name: Execute Build Step
        run: |
          # Secrets are automatically masked in console outputs
          echo "Connecting to DB with fetched credentials..."

```

---

## 7. Security Hardening & Claim Validation Matrix

| Security Parameter | Weak Pattern (High Vulnerability) | Hardened Pattern (Production Standard) |
| --- | --- | --- |
| **`sub` Assertion Match** | `repo:Your-Org/*` *(Any repository in org can assume role)* | `repo:Your-Org/payment-service:environment:production` |
| **Branch Targeting** | `repo:Your-Org/repo:*` *(Feature branches can deploy to prod)* | `repo:Your-Org/repo:ref:refs/heads/main` |
| **Audience (`aud`) Field** | Wildcard / Default | Explicit string validation (`sts.amazonaws.com` or custom Vault endpoint) |
| **Token Lifetime** | Default high limits | Request minimal token TTL per session |

---
