# 📘 Terraform Handbook

> Living handbook for DevOps interview prep.
> 
> **Focus:** EPAM LevelUp A2 → A3 + enterprise project interviews

---

## 📌 How to Read This Book
- **Handbook** = complete textbook with theory, architecture, production patterns, and examples.
- **Quick Revision** = short recap for morning review.
- **Interview Companion** = question bank with senior-level answers.

---

# Chapter 1: Terraform Foundations

## 1. Infrastructure Evolution
Before Infrastructure as Code, infrastructure was often created manually through cloud consoles or scripts. This caused human errors, long provisioning times, inconsistent setups, and configuration drift.

## 2. Infrastructure as Code (IaC)
Infrastructure as Code means managing and provisioning infrastructure through code instead of manual steps. It improves repeatability, version control, collaboration, and disaster recovery.

## 3. Why Terraform?
Terraform is a declarative Infrastructure as Code tool. You describe the desired infrastructure and Terraform figures out the execution steps. It is widely used because it supports many providers, has a dependency graph, manages state, and works well in enterprise environments.

## 4. Terraform vs Other Approaches
- **Terraform vs Bash:** Bash is imperative and does not manage state well.
- **Terraform vs Ansible:** Terraform provisions infrastructure; Ansible configures systems and applications.
- **Terraform vs CloudFormation:** CloudFormation is AWS-specific, while Terraform supports many providers.
- **Terraform vs Pulumi:** Pulumi uses general-purpose languages; Terraform uses HCL and a mature IaC model.

## 5. Terraform Internal Architecture
Terraform reads configuration files, compares them with state, builds a dependency graph, uses providers to talk to cloud APIs, and updates state after apply.

## 6. Dependency Graph
Terraform does not follow file order. It uses dependencies between resources to decide execution order. This allows parallel creation of independent resources and safe destroy ordering.

## 7. Parallelism
Independent resources can be created in parallel. The default parallelism is limited so provider APIs are not overwhelmed. Too much parallelism can cause rate limits or quota issues.

## 8. Providers
Providers are plugins that allow Terraform to communicate with platforms like Google Cloud, AWS, Azure, Kubernetes, or GitHub.

## 9. Data Sources
A data source reads existing infrastructure. It does not create or manage it. Use it when you need to reference an existing VPC, subnet, image, or other resource.

## 10. terraform_remote_state
`terraform_remote_state` is used to read outputs from another Terraform state. It is useful for cross-team or cross-stack collaboration. It exposes only outputs, not the whole state.

## 11. Infrastructure Ownership
Only one Terraform stack should own a resource. If another stack needs the resource, it should consume outputs or read via data sources rather than trying to recreate the resource.

## 12. Working with Existing Infrastructure
If a VPC or subnet already exists, do not recreate it. If it is outside your current stack, use a data source. If it is owned by another Terraform stack, prefer `terraform_remote_state` outputs.

## 13. State
Terraform State is Terraform's memory. It maps configuration resources to real infrastructure and stores IDs, attributes, dependencies, outputs, serial, and lineage.

## 14. State File Anatomy
A state file contains version, terraform version, serial, lineage, resources, outputs, and metadata. Serial tracks revision; lineage identifies the state family.

## 15. Local vs Remote State
Local state is simple but not suitable for teams. Remote state is stored centrally, usually in cloud storage like GCS, and supports collaboration, recovery, and locking.

## 16. State Locking
State locking prevents multiple Terraform operations from modifying the same state at once. In GCS, Terraform relies on object generation numbers to protect state updates.

## 17. Backend
A backend defines where Terraform stores state and how it handles state operations such as read, write, lock, and migration. Backends do not manage infrastructure; providers do.

## 18. Backend Migration
When moving from local state to remote state, use `terraform init -migrate-state` to transfer state. Use `terraform init -reconfigure` when the remote backend already has the correct state.

## 19. Drift Detection
Drift occurs when actual infrastructure no longer matches Terraform configuration. Terraform detects drift during plan by refreshing resource information from the provider.

## 20. Configuration Drift vs Environment Drift
Configuration drift is the difference between Terraform configuration and actual infrastructure. Environment drift is the unintended difference between environments such as dev, qa, stage, and prod.

## 21. Refresh
Terraform refreshes state automatically during `terraform plan` and `terraform apply`. The standalone `terraform refresh` command is deprecated because it modified state without a reviewed plan.

## 22. Rollback Reality
Terraform does not have a native rollback command. Rollback is achieved by reverting configuration in Git, then reviewing the plan carefully before applying the changes.

## 23. Disaster Recovery
If Terraform state is lost but infrastructure still exists, recover from versioned remote state if possible. If not, use `terraform import` to rebuild ownership mapping.

## 24. Enterprise Design Principles
- One stack owns one resource set.
- Reuse modules across environments.
- Keep environment differences in variables.
- Expose only necessary outputs.
- Store state remotely with locking and versioning.
- Avoid manual configuration drift.

## 25. Interview Summary
Senior Terraform interviews focus on design, state management, ownership, remote state, drift detection, and production architecture rather than just syntax.

---

# Chapter 2: Terraform State

## 1. Why Terraform Needs State
Terraform state is the mapping that allows Terraform to know which real cloud objects correspond to the resources in configuration. Without state, Terraform cannot safely update or destroy the correct objects.

## 2. State File Internals
A state file stores version, terraform version, serial, lineage, resources, outputs, dependencies, and metadata.

## 3. What Serial Means
Serial is the revision number of the state file. It increases whenever Terraform writes new state.

## 4. What Lineage Means
Lineage is the unique identifier of the state family. It helps Terraform distinguish one state history from another.

## 5. State Is Not the Infrastructure
The state file does not contain running VMs or live resources. It contains Terraform's recorded mapping and metadata about those resources.

## 6. Local State
Local state is stored on your machine. It is fine for learning and small lab work, but not for teams or production.

## 7. Remote State
Remote state is stored centrally in a backend such as GCS. It enables collaboration, locking, recovery, and safer enterprise workflows.

## 8. Backend
A backend manages state read/write, locking, and migration. It is separate from providers.

## 9. GCS Backend
For GCP, a GCS bucket with a prefix is a common backend choice. The prefix acts like a logical state path.

## 10. Backend Migration
`terraform init -migrate-state` moves existing state to a new backend. `terraform init -reconfigure` changes backend configuration without migrating state.

## 11. State Locking
State locking protects the shared state from concurrent modification. In GCS, Terraform uses generation numbers and optimistic concurrency control.

## 12. Force Unlock
Use `terraform force-unlock` only if a stale lock remains after confirming no Terraform operation is active.

## 13. Object Versioning
Enable object versioning in the backend so previous state snapshots can be recovered if needed.

## 14. Drift Detection
Terraform detects drift during plan by refreshing the latest infrastructure data from the provider and comparing it to configuration.

## 15. State Recovery
If state is lost, first restore from backend versioning if available. If not, use `terraform import` to adopt the existing infrastructure.

## 16. State Commands
Useful commands include `terraform state list`, `terraform state show`, `terraform state mv`, `terraform state rm`, `terraform state pull`, and `terraform import`.

## 17. Existing Infrastructure
If a VPC or subnet already exists, reference it using a data source or remote state instead of recreating it.

## 18. Ownership Rules
Never let two Terraform stacks manage the same resource. If another team owns it, consume it through outputs or data sources.

## 19. Configuration Drift vs Environment Drift
Configuration drift is code versus reality. Environment drift is one environment differing from another unintentionally.

## 20. Serial, Lineage, and Versioning Together
Serial tracks the order of state updates, lineage tracks the state family, and backend versioning preserves historical snapshots.

## 21. Production Best Practices
Use a remote backend, enable locking, enable versioning, avoid manual edits, keep one owner per resource, and split state by environment or domain.

## 22. Recovery Mindset
State restoration is a recovery procedure, not a casual rollback shortcut. Always verify with `terraform plan` after any state restoration.

## 23. Interview Summary
State is one of the most important Terraform topics because it explains ownership, collaboration, locking, drift, recovery, and safe updates.

---

# Chapter 3: Production Architecture and Decision Making

## 1. Why Use Data Sources
Use data sources when existing infrastructure is outside your current Terraform stack and you need to read its attributes.

## 2. Why Use Remote State Outputs
Use remote state outputs when another Terraform stack owns the infrastructure and explicitly exports the values you need.

## 3. Outputs as Contracts
Outputs act as a contract between Terraform stacks. They expose only what consumers need and hide internal implementation details.

## 4. Why Not Expose the Entire State
Exposing the entire state would tightly couple stacks and reveal internal details. Outputs keep the interface clean and controlled.

## 5. Existing VPC Example
If networking owns the VPC, your stack should consume the VPC ID from outputs or data source lookup rather than creating another VPC.

## 6. Renaming Resources
If a resource name changes, any hardcoded name-based lookup can fail. Remote state outputs are often more stable than name-only lookup because they expose the actual managed identifiers.

## 7. One Resource, One Owner
Each resource should be owned by one Terraform stack only. Multiple owners create conflicts and unpredictable behavior.

## 8. Environment Design
Keep dev, qa, stage, and prod separated with distinct state files and environment-specific variables.

## 9. Modular Design
Build reusable modules for shared capabilities such as network, compute, database, and monitoring.

## 10. Reduced Drift
Reuse the same modules across environments and keep differences in variables so environments stay consistent.

## 11. Production Scenario: Lost State
If state disappears but resources still exist, use versioned state restore first, then `terraform import` if needed.

## 12. Production Scenario: Parallel Applies
Two engineers applying against the same state must be prevented by locking to avoid lost updates.

## 13. Production Scenario: Drift
If someone manually changes a firewall or machine type, the next plan should reveal the difference and the team should decide whether to revert or adopt the change.

## 14. Enterprise Terraform Mindset
Terraform design is about ownership, contracts, collaboration, and safe change management, not just writing resources.

## 15. Interview Summary
Senior Terraform design questions usually test whether you understand architecture trade-offs and production ownership boundaries.

---

# Chapter 4: Interview Summary and Common Questions

## Common Questions to Practice
- Why Terraform?
- Why remote state?
- What is the difference between provider, backend, and data source?
- What is state and why is it needed?
- What is drift?
- Why is `terraform refresh` deprecated?
- How do you recover lost state?
- What is `terraform_remote_state`?
- How do you avoid environment drift?
- How do you avoid two Terraform stacks managing the same resource?

## Strong One-Line Answers
**Terraform:** Declarative IaC with state and dependency graph.

**State:** Terraform's memory and ownership mapping.

**Backend:** Where state lives and how it is managed.

**Data source:** Read-only access to existing infrastructure.

**Remote state:** Outputs from another Terraform stack.

**Drift:** Difference between desired configuration and actual infrastructure.

**Environment drift:** Unintended difference between dev, qa, stage, and prod.

**Locking:** Protection against concurrent state corruption.

**Import:** Bring existing infrastructure under Terraform management.

**Rollback:** Revert code in Git and review the plan before applying.

## Revision Focus
- Always explain the why, not only the what.
- Always mention ownership and production safety.
- Always connect the answer back to state, backend, and provider behavior.

---

# Chapter 5: State Commands — Quick Handbook Reference

## `terraform state list`
Lists all resource addresses currently managed by Terraform state.

## `terraform state show`
Shows detailed attributes for one managed resource.

## `terraform state mv`
Moves a resource address in state without touching real infrastructure.

## `terraform state rm`
Removes Terraform ownership without deleting the real infrastructure.

## `terraform state pull`
Downloads the current state from the backend as JSON.

## `terraform state push`
Uploads a validated local state file to the backend in recovery scenarios.

## `terraform import`
Adopts existing infrastructure into Terraform state.

## `terraform force-unlock`
Removes stale state locks when you are certain no Terraform operation is active.

---

# Chapter 6: Variables, Locals, Outputs

## Variables
Variables define the external interface of a module. They are inputs from the caller or environment.

## Locals
Locals are internal values or computed expressions inside the module. They cannot be overridden by the caller.

## Outputs
Outputs are the public return values of a module. They expose only the values other modules or teams need.

## Scope Matters
- **Provider scope:** `default_labels`, provider settings.
- **Root module scope:** environment-level locals and orchestration.
- **Child module scope:** module-specific computation.

## Variables vs Locals vs Outputs
| Concept | Role |
|---|---|
| Variable | Input |
| Local | Internal computation |
| Output | Return value |

## Senior Design Rule
- Use variables for caller-controlled values.
- Use locals for common or computed values.
- Use outputs for published values.

---

# Chapter 7: Variable Precedence

## Loading Order
Lowest to highest priority:
1. default values in `variables.tf`
2. environment variables (`TF_VAR_*`)
3. `terraform.tfvars`
4. `*.auto.tfvars`
5. `-var-file`
6. `-var`

## Best Practice
Use defaults for safe generic values, tfvars for environment-specific values, and CLI overrides only for one-off executions.

## Interview Trap
`prod.auto.tfvars` wins over `terraform.tfvars` because Terraform loads auto tfvars after the standard tfvars file, not because the filename contains `prod`.

---

# Chapter 8: Module Interface Design

## Strong Typing
Use `map(object(...))` for complex inputs so Terraform can validate the shape of the input early.

## Why Type Constraints Matter
Types check structure, while validation checks business rules.

## Why Modules Feel Like APIs
- Inputs = variables
- Private implementation = locals
- Return values = outputs

## Module Design Rule
Each module should expose the minimum interface needed by consumers and hide internal implementation details.

---

# Chapter 9: Meta Arguments

## `count`
Use for identical resources that differ only by quantity.

## `for_each`
Use for uniquely identified resources with stable keys.

## `dynamic`
Use to generate repeated nested blocks inside a resource.

## `depends_on`
Use only when Terraform cannot infer the dependency automatically.

## `lifecycle`
Use to control create/destroy/replace behavior.

---

# Chapter 10: `count` vs `for_each`

## `count`
Best for homogeneous resources and simple scaling.

## `for_each`
Best for stable identities and different per-resource configuration.

## Enterprise Rule
If adding/removing one resource should not affect others, prefer `for_each`.

---

# Chapter 11: `lifecycle`

## `create_before_destroy`
Creates the replacement before destroying the old resource.

## `prevent_destroy`
Stops accidental deletion of critical resources.

## `ignore_changes`
Ignores drift for selected attributes during planning.

## `replace_triggered_by`
Forces replacement when another value changes.

## Important Nuance
`ignore_changes` helps with externally managed drift, not with intentionally changing the Terraform configuration for a ForceNew attribute.

---

# Chapter 12: ForceNew Attributes

## Meaning
If the provider marks an attribute as ForceNew, changing it requires resource replacement.

## Provider Responsibility
Terraform Core builds the plan, but the provider schema decides whether a change can happen in place or requires replacement.

## Reading the Plan
| Symbol | Meaning |
|---|---|
| `+` | create |
| `-` | destroy |
| `~` | update in place |
| `-/+` | replace |

---

# Chapter 13: Data Sources vs Remote State

## Data Source
Reads existing infrastructure directly from the cloud provider.

## Remote State
Reads outputs published by another Terraform configuration.

## Best Use
- Use data sources for unmanaged or cloud-managed resources.
- Use remote state for resources owned by another Terraform stack.

---

# Chapter 14: Workspaces

## Workspaces Isolate State, Not Configuration
Use them for multiple instances of essentially the same configuration.

## Not Ideal For
Use separate environment directories for Dev, QA, Stage, and Prod.

## Common Pitfall
Hardcoded names create conflicts across workspaces unless names are made workspace-aware.

---

# Chapter 15: Provisioners

## Use Case
Rare bootstrapping or temporary orchestration.

## Enterprise Rule
Use Terraform for provisioning and Ansible for configuration.

---

# Chapter 16: Dynamic Blocks

## What They Do
Generate repeated nested blocks inside a resource.

## When to Use
When the provider resource has nested repeatable configuration blocks.

---

# Chapter 17: Terraform Functions

## Common Functions
- `lookup()`
- `merge()`
- `try()`
- `can()`
- `coalesce()`
- `flatten()`
- `zipmap()`
- `jsonencode()`
- `jsondecode()`

## Practical Rule
Use `lookup()` for map selection, `merge()` for combining maps, and `map(object(...))` for rich structured inputs.

---

# Chapter 18: Module Sources

## Common Sources
| Source | Example | When Used |
|---|---|---|
| Local path | `../../modules/vpc` | same repository |
| Git repository | `git::https://github.com/company/terraform-modules.git//vpc?ref=v1.2.0` | enterprise shared modules |
| Terraform Registry | `terraform-google-modules/network/google` | reusable public modules |
| Private registry | enterprise module registry | large organizations |

## Version Pinning
Always pin Git module versions using `?ref=` for reproducibility.

---

# Chapter 19: Enterprise Label Design

## GCP `default_labels`
Use provider-level `default_labels` for organization-wide labels that should apply to supported resources.

## `merge()`
Use `merge()` when you need to combine common labels with environment-specific or application-specific overrides.

## Recommendation
For GCP, prefer `default_labels` for common labels and use variables or `merge()` for exceptions.

---

# Chapter 20: Functions Deep Dive – `lookup()`

## Purpose
`lookup()` retrieves a value from a map using a key and falls back to a default value if the key does not exist.

## Where to Define the Map
Use `locals.tf` for common environment-to-value maps when the caller should not modify the mapping.

## Enterprise Pattern
Use `map(object(...))` when one environment has multiple related properties like machine type, disk size, region, and zone.

---

# Chapter 21: Terraform Architecture Reminders

## Terraform Core vs Provider vs Cloud API
- **Terraform Core** builds the graph and plan.
- **Provider** defines schema and implements API actions.
- **Cloud API** actually performs the operation.

## Why This Matters
Different providers can behave differently even when the Terraform language looks the same.

---

# Chapter 22: Tool Choice

## Use the Right Tool
| Task | Tool |
|---|---|
| Provision infrastructure | Terraform |
| Configure software | Ansible |
| Deploy containers | Kubernetes |
| Build application | Maven / Gradle |
| Orchestrate pipeline | GitHub Actions |

## Enterprise Flow
GitHub Actions orchestrates the pipeline, Terraform creates infrastructure, and Ansible configures it.

---

# Chapter 23: Day 1 Knowledge Nuggets

- `lookup()` is ideal for environment maps.
- `locals.tf` is the right place for fixed internal mappings.
- `map(object(...))` is the clean way to represent rich per-environment config.
- Git module source pinning prevents accidental breaking changes.
- `terraform_remote_state` is an ownership boundary, not just a backend file read.
- `default_labels` is the most GCP-native way to apply common labels.

---

# Chapter 24: Interview Expectations

## What Interviewers Want To Hear
- Why this solution fits production.
- Why this is safer than a simpler alternative.
- Who owns the resource or value.
- Where the interface boundary is.
- How drift, versioning, and recovery are handled.

## Always Mention
- Ownership
- Safety
- Reusability
- Maintainability
- Production behavior

---

# Chapter 25: Study Flow Reminder
- Learn a concept.
- Add the senior design rule.
- Add the production scenario.
- Add the interview question.
- Add the command or pattern to quick revision.

