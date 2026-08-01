# Terraform Quick Revision

## Day 1 Must Remember
- Terraform is declarative IaC.
- Providers manage resources.
- Backends manage state.
- State is Terraform's memory.
- Dependency graph controls execution order, not file order.
- Independent resources can run in parallel.
- Data sources read existing infrastructure.
- `terraform_remote_state` reads outputs from another Terraform stack.
- Only one Terraform stack should own a resource.
- Remote state is essential for teams.
- State locking prevents concurrent corruption.
- GCS backend uses generation numbers for safe locking.
- `terraform init -migrate-state` moves existing state to a new backend.
- `terraform init -reconfigure` switches backend without migrating existing state.
- Drift means reality differs from desired Terraform configuration.
- `terraform refresh` is deprecated; modern Terraform refreshes during plan/apply.
- `terraform import` helps adopt existing infrastructure.

## Short Interview Answers
**Why Terraform?**
Because it is declarative, repeatable, scalable, and supports many providers.

**Why remote state?**
For collaboration, locking, recovery, and consistent ownership.

**What is a data source?**
A read-only view of existing infrastructure.

**What is `terraform_remote_state`?**
A way to consume outputs from another Terraform stack.

**What is drift?**
A mismatch between Terraform configuration and actual infrastructure.

**Can two Terraform projects own one resource?**
No.

**What if state is lost?**
Restore from versioned backend or use `terraform import`.

## Common Mistakes
- Using one state for every environment.
- Recreating resources that already exist.
- Overusing `depends_on`.
- Disabling locking.
- Manually editing state.
- Treating `terraform_remote_state` as if it exposes the whole state.

## One-Sentence Recall
Terraform writes the desired infrastructure in code, tracks ownership in state, uses a dependency graph to order work, and relies on backends, locking, and versioning for safe enterprise collaboration.
