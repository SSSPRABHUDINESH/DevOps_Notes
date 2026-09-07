# 8. Real-world scenario

Suppose the company has:

```text
Node.js application
       +
Docker
       +
Kubernetes / GKE
       +
Terraform
       +
GitHub Actions
       +
Artifact Registry
```

Developer pushes code:

```text
Developer
   ↓
GitHub repository
   ↓
GitHub Actions
```

Now we divide the pipeline into:

```text
                 CI
                  ↓
        Build + Test + Security
                  ↓
             Artifact
                  ↓
                 CD
                  ↓
              Kubernetes
```

---

# 9. Complete CI pipeline

A production-style CI pipeline could be:

```text
Developer pushes code
        ↓
Pull Request
        ↓
Checkout source
        ↓
Setup Node
        ↓
npm ci
        ↓
Lint
        ↓
Unit Tests
        ↓
Integration Tests
        ↓
SonarQube
        ↓
SAST / dependency scan
        ↓
Docker build
        ↓
Container image scan
        ↓
Push image to Artifact Registry
```

Let's understand each.

---

## Step 1 — Checkout

```yaml
- uses: actions/checkout@v6
```

The GitHub runner is initially a fresh machine.

So:

```text
Fresh Runner
    ↓
checkout
    ↓
application source code
```

GitHub-hosted runners are newly provisioned for workflow execution. ([GitHub Docs][2])

---

# 10. Step 2 — Setup Node

```yaml
- uses: actions/setup-node@v7
  with:
    node-version: '20'
```

Now:

```text
Runner
 ├── node
 └── npm
```

are available.

---

# 11. Step 3 — Install dependencies

Use:

```bash
npm ci
```

rather than blindly using:

```bash
npm install
```

in CI when `package-lock.json` is committed.

Conceptually:

```text
package.json
package-lock.json
       ↓
     npm ci
       ↓
node_modules/
```

---

# 12. Step 4 — Lint

```yaml
- name: Lint
  run: npm run lint
```

This checks code quality/style.

---

# 13. Step 5 — Unit testing

```yaml
- name: Unit tests
  run: npm test
```

If tests fail:

```text
npm test
   ↓
FAILED
   ↓
CI stops
   ↓
No image
   ↓
No deployment
```

That's an important DevOps principle:

> **A failed quality gate should prevent promotion.**

---

# 14. Step 6 — SonarQube

Now code quality/security analysis:

```text
Source code
    ↓
SonarQube
    ↓
bugs
vulnerabilities
code smells
coverage
duplication
    ↓
Quality Gate
```

Example:

```text
SonarQube Quality Gate
       ↓
      PASS
       ↓
continue
```

or:

```text
Quality Gate
       ↓
      FAIL
       ↓
STOP PIPELINE
```

---

# 15. Step 7 — Dependency vulnerability scan

You can scan application dependencies.

For example:

```text
npm dependencies
      ↓
dependency vulnerability scanner
      ↓
CVE findings
      ↓
policy
```

For example:

```text
Critical vulnerability
        ↓
pipeline FAIL
```

The exact scanner could vary by organisation:

* Snyk
* Trivy
* OWASP Dependency-Check
* GitHub Dependabot/GHAS
* organisation-specific security tooling

Don't get stuck on the tool name.

The important concept is:

> **Scan dependencies before producing/promoting the deployable artifact.**

---

# 16. Step 8 — Build Docker image

Now:

```bash
docker build -t myapp:${GITHUB_SHA} .
```

Notice something important.

Don't use:

```text
latest
```

as your only production identity.

Prefer an immutable identifier:

```text
myapp:8f42ab1
```

where:

```text
8f42ab1 = Git commit SHA
```

So:

```text
Git commit
    ↓
Docker image
    ↓
myapp:<commit-sha>
```

Now we know exactly which source created that image.

---

# 17. Step 9 — Scan the Docker image

This directly matches the interviewer question.

```text
Docker image
     ↓
Trivy / image scanner
     ↓
OS vulnerabilities
package vulnerabilities
misconfigurations
secrets
     ↓
Policy
```

Example:

```text
CRITICAL CVE
    ↓
FAIL
    ↓
Do NOT push/promote/deploy
```

Or:

```text
No blocking vulnerability
    ↓
continue
```

---

# 18. Step 10 — Push to Artifact Registry

Only after the quality/security gates pass:

```text
Docker image
     ↓
Artifact Registry
```

For example:

```text
asia-south1-docker.pkg.dev/
    my-project/
       my-repo/
          myapp:8f42ab1
```

Now your artifact is stored centrally.

---

# 19. Important distinction: scan before or after registry?

The interviewer specifically mentioned:

> "security scanning after downloading the image/artifact from repository before deployment"

There are multiple valid enterprise designs.

### Pattern A — Scan before push

```text
Build image
   ↓
Scan
   ↓
PASS
   ↓
Push to Artifact Registry
```

Good because bad images never enter the registry.

### Pattern B — Push → scan → promote/deploy

```text
Build
 ↓
Push image
 ↓
Scan image in registry
 ↓
PASS
 ↓
Deploy
```

Also common.

### Stronger enterprise pattern

```text
Build
 ↓
Scan
 ↓
Push
 ↓
Registry scanning
 ↓
Policy verification
 ↓
Deploy
```

You can say:

> "I prefer immutable image tags and a security gate before deployment. Depending on the organisation, scanning can happen before pushing, after pushing to the registry, or both."

That's a very safe interview answer.

---

# 20. Now CD starts

This is the key distinction.

### CI

```text
"Is this code safe and valid?"
```

### CD

```text
"How do I safely release this validated artifact?"
```

So:

```text
CI
 ↓
validated image
 ↓
CD
 ↓
GKE
```

---

# 21. CD → Kubernetes

Suppose Artifact Registry contains:

```text
myapp:8f42ab1
```

CD obtains that exact image.

Then:

```text
GitHub Actions
      ↓
authenticate to GCP
      ↓
get GKE credentials
      ↓
kubectl / Helm
      ↓
update Deployment
      ↓
GKE rolling update
```

For example:

```bash
kubectl set image deployment/myapp \
  myapp=asia-south1-docker.pkg.dev/project/repo/myapp:${GITHUB_SHA}
```

or preferably in a Helm-based deployment:

```bash
helm upgrade --install myapp ./helm/myapp \
  --set image.tag=${GITHUB_SHA}
```

---

# 22. What happens inside Kubernetes?

This connects directly to the Kubernetes questions you already got.

```text
GitHub Actions
      ↓
Deployment updated
      ↓
Deployment controller
      ↓
ReplicaSet
      ↓
new Pods
      ↓
readiness probe
      ↓
Service
      ↓
traffic
```

For a rolling update:

```text
Old Pods:

Pod v1
Pod v1
Pod v1


New image:

Pod v2
```

Gradually:

```text
v1 v1 v1
   ↓
v2 v1 v1
   ↓
v2 v2 v1
   ↓
v2 v2 v2
```

If readiness fails:

```text
v2
 ↓
Readiness FAIL
 ↓
traffic not sent
 ↓
deployment rollout stalls
```

This is exactly the kind of **CI/CD + Kubernetes integration** TP2 may test.

---

# 23. Production CD should not immediately deploy to production

A better enterprise pipeline:

```text
                 CI
                  ↓
             Build image
                  ↓
             Security scan
                  ↓
           Artifact Registry
                  ↓
               Deploy
                  ↓
               DEV
                  ↓
             smoke tests
                  ↓
              STAGING
                  ↓
          integration tests
                  ↓
          approval / gate
                  ↓
              PRODUCTION
```

GitHub Actions supports environments, protection rules, approvals and concurrency controls for deployments. ([GitHub Docs][3])

---

# 24. Where Terraform fits?

This is **very important for your TP2**.

Do NOT make Terraform deploy every application version.

Think:

```text
Terraform
   ↓
Infrastructure
```

while:

```text
GitHub Actions + Helm/kubectl
   ↓
Application deployment
```

For example Terraform manages:

```text
GCP VPC
GKE cluster
Node pools
IAM
Service accounts
Artifact Registry
Cloud SQL
DNS
Load balancer-related infrastructure
```

Then application CD manages:

```text
Deployment
Service
Ingress
ConfigMap
application image version
Helm release
```

So:

```text
Terraform
   ↓
Build the platform


GitHub Actions
   ↓
Release application versions
```

That's a **very strong interview distinction**.

---

# 25. But Terraform can also be part of CI/CD

For infrastructure changes:

```text
Developer
   ↓
Terraform code
   ↓
Pull Request
   ↓
terraform fmt
   ↓
terraform validate
   ↓
terraform init
   ↓
terraform plan
   ↓
security scan
   ↓
review
   ↓
merge
   ↓
terraform apply
```

Production:

```text
Terraform Plan
       ↓
Review
       ↓
Approval
       ↓
Terraform Apply
       ↓
GCP infrastructure changed
```

So you can have **two pipelines**:

### Application pipeline

```text
Code
 ↓
Test
 ↓
Scan
 ↓
Build image
 ↓
Push image
 ↓
Deploy GKE
```

### Infrastructure pipeline

```text
Terraform
 ↓
fmt
 ↓
validate
 ↓
plan
 ↓
security scan
 ↓
approval
 ↓
apply
```

This is a very good TP2 topic.

---

# 26. The complete enterprise picture

This is the architecture I want you to be able to explain without hesitation:

```text
                 DEVELOPER
                     │
                     ▼
                 GitHub PR
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
    Application CI        Terraform CI
          │                     │
     npm ci                  fmt
     npm test              validate
     lint                   plan
     SonarQube             IaC scan
     dependency scan           │
          │                     ▼
          ▼                 Approval
     Docker build              │
          │                    ▼
     Image scan             Terraform
          │                  Apply
          ▼                    │
   Artifact Registry           ▼
          │                 GCP Infra
          │
          ▼
      CD Pipeline
          │
    authenticate GCP
          │
          ▼
        GKE
          │
       Helm/Kubectl
          │
          ▼
    Rolling Deployment
          │
     Readiness probes
          │
          ▼
       Service
          │
          ▼
       Application
```

**This is the model I would prepare for TP2.**

---

# 27. One very important security improvement

For GCP authentication from GitHub Actions, don't say:

> "I store a GCP service-account JSON key in GitHub Secrets."

That works technically, but for a modern enterprise answer, say:

```text
GitHub Actions
      ↓
OIDC
      ↓
Workload Identity Federation
      ↓
GCP service account
      ↓
GKE / Artifact Registry / GCP APIs
```

This avoids long-lived GCP service-account keys.

That is exactly the kind of answer that makes your CI/CD knowledge sound more production-grade.

---

# 28. What I think TP2 will focus on

Based on what happened in TP1, I would prepare these **in this exact order**:

### 🔴 Tier 1 — Must know

1. GitHub Actions workflow structure
2. `run` vs `uses`
3. runners
4. jobs vs steps
5. `needs`
6. `if`
7. secrets
8. environments
9. artifacts
10. cache
11. matrix
12. npm workflow
13. CI pipeline
14. CD pipeline
15. Docker build
16. Artifact Registry
17. image vulnerability scanning
18. SonarQube quality gate
19. GitHub Actions → GKE
20. Helm deployment
21. rolling deployment
22. rollback
23. GCP authentication from GitHub Actions
24. OIDC / Workload Identity Federation

GitHub officially describes Actions as a CI/CD platform and supports both build/test and deployment workflows. ([GitHub Docs][4])

### 🟠 Tier 2

25. Terraform CI/CD
26. Terraform plan/apply separation
27. environment-specific deployment
28. DEV → QA → PROD
29. approval gates
30. concurrency
31. self-hosted runners
32. private GKE deployment
33. secrets management
34. deployment failure/rollback
35. artifact promotion

### 🟡 Tier 3

36. reusable workflows
37. composite actions
38. workflow outputs
39. dynamic matrices
40. GitHub environments
41. deployment protection
42. pipeline optimisation
43. supply-chain security
44. SBOM
45. image signing / verification
46. GitOps awareness

---

## And one more thing

Your TP1 actually gives us a **very clear preparation strategy**.

You don't need:

> "Learn GitHub Actions theory."

You need:

> **"Given a real application + Terraform + Docker + GKE, design and explain the complete CI/CD system."**

That will automatically cover most of the individual GitHub Actions questions.

And because your actual AlloyDB project already involves **Terraform + Ansible + Molecule + CI/CD + HA/failover**, we can build the examples around a realistic infrastructure/platform engineering scenario rather than a toy Node.js app.

**I recommend our next preparation chapter be:**

### 🚀 GitHub Actions — Production CI/CD Pipeline for a Node.js → Docker → Artifact Registry → GKE application

We'll build it from zero:

```text
GitHub PR
 ↓
Checkout
 ↓
Node setup
 ↓
npm ci
 ↓
Lint
 ↓
Unit test
 ↓
SonarQube
 ↓
Dependency scan
 ↓
Docker build
 ↓
Trivy/image scan
 ↓
Artifact Registry
 ↓
GKE authentication
 ↓
Helm
 ↓
Rolling deployment
 ↓
Smoke test
 ↓
Production approval
 ↓
Rollback
```

And for every stage I'll explain **what actually happens on the runner, why that stage exists, what command/action is used, what can fail, and what you should say in TP2**. That is the gap exposed by your first round. 🔥

[1]: https://docs.github.com/en/actions/tutorials/create-an-example-workflow?utm_source=chatgpt.com "Creating an example workflow - GitHub Docs"
[2]: https://docs.github.com/en/actions/get-started/understand-github-actions?utm_source=chatgpt.com "Understanding GitHub Actions - GitHub Docs"
[3]: https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/control-deployments?utm_source=chatgpt.com "Deploying with GitHub Actions - GitHub Docs"
[4]: https://docs.github.com/en/actions/get-started/quickstart?utm_source=chatgpt.com "Quickstart for GitHub Actions - GitHub Docs"
