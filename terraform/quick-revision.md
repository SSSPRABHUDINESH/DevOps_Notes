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
- `terraform force-unlock` should be used only for stale locks.

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

## Comparison Tables

| Concept | Meaning |
|---------|---------|
| Provider | Manages cloud resources |
| Backend | Manages state storage and locking |
| Data source | Reads existing infrastructure |
| Resource | Creates and manages infrastructure |
| Remote state | Reads outputs from another Terraform stack |
| Drift | Desired state and reality do not match |

| Situation | Best Choice | Why |
|-----------|-------------|-----|
| Existing VPC owned by another team | `terraform_remote_state` or data source | Reuse instead of recreate |
| Unmanaged existing VM | `terraform import` | Adopt into Terraform |
| New network for the project | Resource | Terraform should create it |
| Multiple engineers on same state | Remote backend + locking | Prevent corruption |

## Commands and Flags Cheat Sheet

| Command | What it is used for |
|---------|---------------------|
| `terraform init` | Initialize the working directory and backend |
| `terraform plan` | Preview the changes before applying |
| `terraform apply` | Create/update infrastructure |
| `terraform destroy` | Remove managed infrastructure |
| `terraform import` | Bring existing infrastructure under Terraform management |
| `terraform state list` | List resources in state |
| `terraform state show` | Display details of one resource in state |
| `terraform state mv` | Move a resource address in state without recreating |
| `terraform state rm` | Remove a resource from state without deleting real infrastructure |
| `terraform state pull` | Download the current state |
| `terraform force-unlock` | Remove a stale lock manually |

| Flag | Used For | Interview Note |
|------|----------|----------------|
| `-parallelism=20` | Increase concurrent operations | Limited by dependencies and provider API rate limits |
| `-auto-approve` | Skip confirmation prompt | Mostly used in CI/CD |
| `-replace` | Force recreation of a resource | Safer replacement approach than old taint workflows |
| `-refresh-only` | Refresh state without proposing config changes | Good for reconciliation checks |
| `-migrate-state` | Move state to a new backend | Used during backend migration |
| `-reconfigure` | Reconfigure backend without migrating | Used when remote backend already has the right state |
| `-target` | Apply only selected resources | Use carefully; can create partial plans |
| `-var` | Pass a single variable | Quick overrides |
| `-var-file` | Pass a variables file | Common for environment-specific values |

## High Probability Questions
- What is state?
- Why is backend needed?
- What is state locking?
- What is `terraform_remote_state`?
- What is drift?
- Why is `terraform refresh` deprecated?
- How do you recover lost state?
- Why should each resource have only one owner?
- What does `-parallelism` do?
- What does `-auto-approve` do?

## One Minute Recall
State = memory.
Backend = storage and locking.
Provider = cloud API bridge.
Data source = read existing.
Resource = create/manage.
Remote state = consume outputs.
Drift = desired ≠ reality.
One owner per resource.
Use locking.
Never edit state manually.

## Interview Traps
- `terraform_remote_state` does not expose the whole state file.
- `terraform refresh` is not the normal workflow anymore.
- `depends_on` should not be used everywhere.
- A backend is not the same as a provider.
- Parallelism is limited by dependencies and API rate limits.
