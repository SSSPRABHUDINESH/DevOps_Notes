# Terraform Handbook

## Chapter 1: Terraform Foundations

### 1. Infrastructure Evolution
Before Infrastructure as Code, infrastructure was often created manually through cloud consoles or scripts. This caused human errors, long provisioning times, inconsistent setups, and configuration drift.

### 2. Infrastructure as Code (IaC)
Infrastructure as Code means managing and provisioning infrastructure through code instead of manual steps. It improves repeatability, version control, collaboration, and disaster recovery.

### 3. Why Terraform?
Terraform is a declarative Infrastructure as Code tool. You describe the desired infrastructure and Terraform figures out the execution steps. It is widely used because it supports many providers, has a dependency graph, manages state, and works well in enterprise environments.

### 4. Terraform Internal Architecture
Terraform reads configuration files, compares them with state, builds a dependency graph, uses providers to talk to cloud APIs, and updates state after apply.

### 5. Dependency Graph
Terraform does not follow file order. It uses dependencies between resources to decide execution order. This allows parallel creation of independent resources and safe destroy ordering.

### 6. Providers
Providers are plugins that allow Terraform to communicate with platforms like Google Cloud, AWS, Azure, Kubernetes, or GitHub.

### 7. Data Sources
A data source reads existing infrastructure. It does not create or manage it. Use it when you need to reference an existing VPC, subnet, image, or other resource.

### 8. terraform_remote_state
`terraform_remote_state` is used to read outputs from another Terraform state. It is useful for cross-team or cross-stack collaboration. It exposes only outputs, not the whole state.

### 9. Infrastructure Ownership
Only one Terraform stack should own a resource. If another stack needs the resource, it should consume outputs or read via data sources rather than trying to recreate the resource.

### 10. State
Terraform State is Terraform's memory. It maps configuration resources to real infrastructure and stores IDs, attributes, dependencies, outputs, serial, and lineage.

### 11. Local vs Remote State
Local state is simple but not suitable for teams. Remote state is stored centrally, usually in cloud storage like GCS, and supports collaboration, recovery, and locking.

### 12. State Locking
State locking prevents multiple Terraform operations from modifying the same state at once. In GCS, Terraform relies on object generation numbers to protect state updates.

### 13. Backend
A backend defines where Terraform stores state and how it handles state operations such as read, write, lock, and migration. Backends do not manage infrastructure; providers do.

### 14. Backend Migration
When moving from local state to remote state, use `terraform init -migrate-state` to transfer state. Use `terraform init -reconfigure` when the remote backend already has the correct state.

### 15. Drift Detection
Drift occurs when actual infrastructure no longer matches Terraform configuration. Terraform detects drift during plan by refreshing resource information from the provider.

### 16. Configuration Drift vs Environment Drift
Configuration drift is the difference between Terraform configuration and actual infrastructure. Environment drift is the unintended difference between environments such as dev, qa, stage, and prod.

### 17. Parallelism
Terraform executes independent resources in parallel, but the default concurrency is limited. Too much parallelism can hit provider API rate limits or quotas.

### 18. Refresh
Terraform refreshes state automatically during `terraform plan` and `terraform apply`. The standalone `terraform refresh` command is deprecated because it modified state without a reviewed plan.

### 19. Working with Existing Infrastructure
If infrastructure already exists, use data sources or `terraform_remote_state` to consume it rather than recreating it. If Terraform lost its state, use `terraform import` or restore from versioned remote state.

### 20. Interview Summary
Senior Terraform interviews focus on design, state management, ownership, remote state, drift detection, and production architecture rather than just syntax.
