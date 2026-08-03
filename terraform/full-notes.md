# 📘 Terraform Full Notes

> Terraform source-of-truth handbook v2.
>
> Search any topic here — from `locals` to `moved` blocks — and you should get definition, usage, example, best practices, common mistakes, and interview-ready guidance in one place.

---

## How to use this file
- **Definition** = what the concept means.
- **Usage** = when and why to use it.
- **Example** = code or realistic snippet.
- **Enterprise Usage** = how it appears in real projects.
- **Common Mistakes** = what not to do.
- **Interview Notes** = the senior-level explanation.

---

# Chapter 1: Terraform Foundations

## 1. Infrastructure Evolution
Before Infrastructure as Code, infrastructure was often created manually through cloud consoles or scripts. That caused human error, slow provisioning, inconsistent environments, and configuration drift.

### Example
```text
Manual console clicks -> drift -> hard to reproduce -> difficult to review
```

### Key Points
- Manual work is inconsistent.
- Reproducibility is poor.
- Drift becomes common.

## 2. Infrastructure as Code (IaC)
Infrastructure as Code means defining infrastructure through code rather than manual steps. Terraform is one of the most common IaC tools.

### Example
```hcl
resource "google_compute_network" "vpc" {
  name = "prod-vpc"
}
```

### Key Points
- Code is the source of truth.
- Infrastructure becomes versioned.
- Changes are reviewable.

## 3. Why Terraform?
Terraform is a declarative IaC tool. You describe the desired state and Terraform calculates the execution steps. It is popular because it supports many providers, uses state, and works well in enterprise environments.

### Key Points
- Declarative, not imperative.
- Strong provider ecosystem.
- State-aware and graph-based.

### Interview Notes
If asked why Terraform is useful in enterprise, mention declarative workflow, state management, provider ecosystem, and safe change control.

## 4. Terraform vs Other Tools
- **Terraform vs Bash**: Bash is imperative and not state-aware.
- **Terraform vs Ansible**: Terraform provisions infrastructure; Ansible configures systems and applications.
- **Terraform vs CloudFormation**: CloudFormation is AWS-specific, while Terraform is multi-provider.
- **Terraform vs Pulumi**: Pulumi uses general-purpose languages; Terraform uses HCL.

### Example
| Tool | Best For |
|---|---|
| Terraform | Provisioning infrastructure |
| Ansible | Configuration management |
| Bash | Small automation scripts |

### Key Points
- Use the right tool for the job.
- Terraform and Ansible are complementary.

## 5. Terraform Internal Architecture
Terraform loads configuration, refreshes state, builds a dependency graph, calls providers, and writes updated state after apply.

### Example Flow
```text
.tf files -> state -> graph -> provider API -> updated state
```

### Key Points
- Core builds the graph.
- Providers do the cloud work.
- State is updated after successful operations.

## 6. Dependency Graph
Terraform does not execute in file order. It uses references and dependencies to determine safe creation, update, and destruction order.

### Example
```hcl
resource "google_compute_network" "vpc" {
  name = "prod-vpc"
}

resource "google_compute_instance" "vm" {
  name = "app-vm"
  network_interface {
    network = google_compute_network.vpc.id
  }
}
```

### Key Points
- References create implicit dependencies.
- Independent resources may run in parallel.
- The graph drives order, not file order.

## 7. Parallelism
Terraform can run independent operations in parallel. The `-parallelism` flag adjusts how many operations Terraform attempts concurrently.

### Example
```bash
terraform apply -parallelism=20
```

### Key Points
- Higher values can speed up execution.
- Too much parallelism can cause API throttling.
- Dependencies still control safety.

## 8. Providers
Providers are plugins that let Terraform communicate with platforms such as Google Cloud, AWS, Azure, Kubernetes, GitHub, and others.

### Example
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}
```

### Key Points
- Providers translate Terraform actions into API calls.
- Different providers have different schemas.
- Terraform Core itself is provider-agnostic.

## 9. Data Sources
A data source reads existing infrastructure without managing it.

### Example
```hcl
data "google_compute_network" "vpc" {
  name = "shared-vpc"
}
```

### Key Points
- Read-only.
- Useful for unmanaged or cloud-created resources.
- Does not create ownership.

### Interview Notes
Use a data source when the object already exists and Terraform does not own it.

## 10. `terraform_remote_state`
`terraform_remote_state` reads outputs from another Terraform state. It is commonly used between teams or stacks when one stack owns the infrastructure and publishes outputs.

### Example
```hcl
data "terraform_remote_state" "network" {
  backend = "gcs"
  config = {
    bucket = "tf-state-bucket"
    prefix = "network/prod"
  }
}

resource "google_compute_instance" "vm" {
  network_interface {
    network = data.terraform_remote_state.network.outputs.vpc_id
  }
}
```

### Key Points
- Consumes published outputs.
- Keeps ownership boundaries clear.
- Better than depending on internal resource names.

## 11. Ownership
Only one Terraform stack should own a resource.

### Example
```text
Networking team owns VPC
Application team consumes VPC ID
```

### Key Points
- One resource, one owner.
- Avoid duplicate management.
- Use outputs as the contract.

## 12. Working With Existing Infrastructure
If a resource already exists, choose the right integration method:
- use a data source if it is unmanaged or cloud-created,
- use `terraform_remote_state` if another Terraform stack owns it,
- use `terraform import` if you want Terraform to adopt it.

### Key Points
- Match the tool to ownership.
- Avoid recreating existing objects.
- Adopt, do not duplicate.

## 13. State
Terraform state is Terraform’s memory. It stores the mapping between your configuration and the real infrastructure.

### Example
```text
google_compute_instance.vm -> projects/.../instances/app-vm
```

### Key Points
- State is a mapping, not the infrastructure itself.
- It enables safe updates.
- It is essential for drift detection and refactoring.

## 14. State File Anatomy
A state file contains version, terraform version, serial, lineage, resources, outputs, and metadata.

### Example
```json
{
  "version": 4,
  "terraform_version": "1.x.x",
  "serial": 12,
  "lineage": "abc123"
}
```

### Key Points
- `serial` = revision number.
- `lineage` = state family identity.
- The state file is JSON.

## 15. Local vs Remote State
Local state is stored on your machine. Remote state is stored centrally, usually in a cloud backend like GCS.

### Example
```text
Local: terraform.tfstate on laptop
Remote: GCS bucket + prefix
```

### Key Points
- Local state is fine for learning.
- Remote state is standard for teams.
- Remote state enables locking and collaboration.

## 16. State Locking
State locking prevents multiple Terraform operations from modifying the same state at once.

### Example
```text
Engineer A applies
Engineer B waits for the lock
```

### Key Points
- Prevents race conditions.
- Essential in shared backends.
- Protects state integrity.

## 17. Backend
A backend defines where Terraform stores state and how it handles state operations such as read, write, lock, and migration.

### Example
```hcl
terraform {
  backend "gcs" {
    bucket = "tf-state-bucket"
    prefix = "envs/prod"
  }
}
```

### Key Points
- Backend = state management.
- Provider = infrastructure management.
- Backends do not create resources.

## 18. Backend Migration
`terraform init -migrate-state` moves existing state to a new backend. `terraform init -reconfigure` changes backend config without moving the old state.

### Example
```bash
terraform init -migrate-state
terraform init -reconfigure
```

### Key Points
- Use the correct command for the task.
- Migration needs care.
- Reconfigure is not a transfer.

## 19. Drift Detection
Drift happens when the real infrastructure no longer matches Terraform configuration.

### Example
```text
Terraform says owner=platform
Console changed owner=security
```

### Key Points
- Drift can be manual or external.
- `terraform plan` reveals drift.
- Decide whether to revert or accept the change.

## 20. Configuration Drift vs Environment Drift
Configuration drift is mismatch between code and reality. Environment drift is unintended mismatch between dev, qa, stage, and prod.

### Example
```text
Config drift: prod labels changed manually
Environment drift: qa CIDR does not match dev unexpectedly
```

### Key Points
- Both matter in enterprise.
- Both can be dangerous.
- Both should be controlled.

## 21. Refresh
Terraform refreshes state automatically during plan and apply. The standalone `terraform refresh` command is deprecated.

### Example
```bash
terraform plan
terraform apply
```

### Key Points
- Refresh is part of normal workflows.
- Standalone refresh is outdated.
- Review plans before applying.

## 22. Rollback Reality
Terraform does not have a native rollback command. Rollback is performed by reverting the Git change and then reviewing the new plan.

### Example
```text
Git revert -> terraform plan -> terraform apply
```

### Key Points
- Git is the rollback source of truth.
- Always review the plan after reverting.
- Do not use state as a casual rollback tool.

## 23. Disaster Recovery
If state is lost but infrastructure still exists, restore versioned remote state if possible. If not, adopt the infrastructure with `terraform import`.

### Example
```text
Lost state -> restore version -> plan
Lost state + no version -> import existing infra
```

### Key Points
- Prefer versioned recovery.
- Import only when necessary.
- Always verify with plan.

## 24. Enterprise Design Principles
- One stack owns one resource set.
- Reuse modules across environments.
- Keep environment differences in variables.
- Expose only necessary outputs.
- Store state remotely with locking and versioning.
- Avoid manual drift.

### Key Points
- Ownership is a design choice.
- Modules should be reusable.
- State must be protected.

## 25. Interview Summary
Terraform interviews often focus on design, state, ownership, remote state, drift detection, and production architecture rather than only syntax.

### Key Points
- Explain the why.
- Mention ownership.
- Mention safety and recovery.

---

# Chapter 2: Terraform State

## Why Terraform Needs State
Terraform state maps configuration to real cloud objects so Terraform can safely manage them.

### Example
```text
google_compute_instance.vm -> actual instance ID
```

### Key Points
- State is the memory of Terraform.
- It prevents duplicate creation.
- It supports safe changes.

## State File Internals
A state file stores version, terraform version, serial, lineage, resources, outputs, dependencies, and metadata.

### Example
```json
{
  "serial": 42,
  "lineage": "abcd-1234"
}
```

### Key Points
- JSON structure.
- Serial increases on writes.
- Lineage identifies state history.

## Serial
Serial is the revision number of the state file.

### Example
```text
serial 11 -> 12 after a successful apply
```

### Key Points
- Tracks revision changes.
- Helps with state recovery understanding.

## Lineage
Lineage is the unique identifier of the state family.

### Example
```text
Same lineage = same state family
Different lineage = different state family
```

### Key Points
- Avoids mixing unrelated state histories.
- Important during migration and recovery.

## State Is Not the Infrastructure
The state file does not contain live VMs or actual cloud resources. It contains Terraform’s recorded mapping and metadata.

### Example
```text
State says VM exists; VM itself is in GCP
```

### Key Points
- State is metadata.
- Infrastructure lives in the cloud.
- Never edit state casually.

## Local State
Local state is stored on your machine.

### Example
```bash
terraform.tfstate
```

### Key Points
- Simple.
- Not team-safe.
- Easy to lose.

## Remote State
Remote state is stored centrally in a backend such as GCS.

### Example
```hcl
backend "gcs" {
  bucket = "company-tf-state"
  prefix = "envs/prod"
}
```

### Key Points
- Standard for teams.
- Supports recovery and collaboration.
- Usually paired with locking and versioning.

## Backend
A backend manages state storage and operations. It is not a provider.

### Example
```hcl
terraform {
  backend "gcs" {}
}
```

### Key Points
- Backend = state management.
- Provider = resource management.
- Do not confuse the two.

## GCS Backend
For GCP, GCS with a prefix is a common backend choice.

### Example
```hcl
bucket = "company-tf-state"
prefix = "envs/prod"
```

### Key Points
- Scales well.
- Supports versioning.
- Prefix helps split environments.

## Backend Migration
`terraform init -migrate-state` moves state. `terraform init -reconfigure` changes backend config without migrating state.

### Example
```bash
terraform init -migrate-state
terraform init -reconfigure
```

### Key Points
- Choose the right command.
- Migration needs care.
- Reconfigure is not transfer.

## State Locking
State locking prevents concurrent modification.

### Example
```text
Only one apply at a time
```

### Key Points
- Prevents lost updates.
- Mandatory in teams.
- Protects shared state.

## Force Unlock
Use `terraform force-unlock` only for stale locks after confirming no Terraform process is active.

### Example
```bash
terraform force-unlock <lock-id>
```

### Key Points
- Last resort only.
- Be careful in teams.
- Verify before forcing unlock.

## Object Versioning
Enable backend object versioning so previous state snapshots can be recovered.

### Example
```text
GCS versioning keeps old state generations
```

### Key Points
- Important for DR.
- Helps recover overwritten state.
- Highly recommended.

## Drift Detection
Terraform detects drift when it refreshes infrastructure data during plan.

### Example
```text
Console changed label -> plan shows drift
```

### Key Points
- Drift can be manual or external.
- Plan is the normal detection method.
- Decide whether to correct or adopt drift.

## State Recovery
If state is lost, restore from versioning if possible. Otherwise import the resources.

### Example
```text
Restore versioned state -> plan
Or import resources back into state
```

### Key Points
- Prefer restore over rebuild.
- Import is adoption.
- Verify after recovery.

## State Commands
Useful commands include `terraform state list`, `terraform state show`, `terraform state mv`, `terraform state rm`, `terraform state pull`, and `terraform import`.

### Example
```bash
terraform state list
terraform state show google_compute_instance.vm
```

### Key Points
- Each command has a specific ownership purpose.
- State commands do not change cloud resources directly.
- Learn them deeply.

## Existing Infrastructure
If a VPC or subnet already exists, reference it with a data source or remote state instead of recreating it.

### Example
```hcl
data "google_compute_network" "vpc" {
  name = "shared-vpc"
}
```

### Key Points
- Reuse, do not duplicate.
- Match tool to ownership.
- Keep names stable if using lookup by name.

## Ownership Rules
Never let two Terraform stacks manage the same resource.

### Example
```text
Network team owns subnet
App team consumes subnet ID
```

### Key Points
- One resource, one owner.
- Ownership prevents conflict.
- Cross-stack contracts should be explicit.

## Configuration Drift vs Environment Drift
Configuration drift is code versus reality. Environment drift is one environment differing from another unintentionally.

### Example
```text
Config drift: prod label changed manually
Environment drift: qa uses wrong CIDR compared to dev
```

### Key Points
- Both matter.
- Both are dangerous if unmanaged.
- Detection should be deliberate.

## Serial, Lineage, and Versioning Together
Serial tracks the order of state updates, lineage tracks the state family, and backend versioning preserves historical snapshots.

### Example
```text
serial + lineage + versioning = recovery story
```

### Key Points
- Use all three together conceptually.
- They serve different recovery purposes.
- Great interview topic.

## Production Best Practices
Use a remote backend, enable locking, enable versioning, avoid manual edits, keep one owner per resource, and split state by environment or domain.

### Example
```text
prod state in remote backend + versioning + locking
```

### Key Points
- Enterprise baseline.
- Reduces risk.
- Improves collaboration.

## Recovery Mindset
State restoration is a recovery procedure, not a casual rollback shortcut. Always verify with `terraform plan` after any state restoration.

### Example
```bash
terraform plan
```

### Key Points
- Restoration is not the same as rollback.
- Always validate after recovery.
- Never assume restored state is perfect.

## Interview Summary
State is one of the most important Terraform topics because it explains ownership, collaboration, locking, drift, recovery, and safe updates.

### Key Points
- State is central.
- Ownership matters.
- Recovery must be planned.

---

# Chapter 3: Variables, Locals, Outputs

## Variables
Variables define the external interface of a module. They are inputs from the caller or environment.

### Example
```hcl
variable "environment" {
  type = string
}
```

### Key Points
- Inputs.
- Caller-controlled.
- Part of the module API.

## Locals
Locals are internal values or computed expressions inside the module. They cannot be overridden by the caller.

### Example
```hcl
locals {
  vm_name = "app-${var.environment}"
}
```

### Key Points
- Internal implementation.
- Great for computed values.
- Improves readability.

## Outputs
Outputs are the public return values of a module. They expose only the values other modules or teams need.

### Example
```hcl
output "vpc_id" {
  value = google_compute_network.vpc.id
}
```

### Key Points
- Public interface of the module.
- Used by other modules/stacks.
- Keep them minimal and stable.

## Scope Matters
- **Provider scope:** `default_labels`, provider settings.
- **Root module scope:** environment-level locals and orchestration.
- **Child module scope:** module-specific computation.

### Example
```hcl
provider "google" {
  default_labels = {
    owner = "platform"
  }
}
```

### Key Points
- Put logic at the right scope.
- Avoid leaking internal decisions.
- Use provider scope when the provider supports it.

## Variables vs Locals vs Outputs
| Concept | Role |
|---|---|
| Variable | Input |
| Local | Internal computation |
| Output | Return value |

### Key Points
- Variables accept values.
- Locals compute values.
- Outputs expose values.

## Senior Design Rule
- Use variables for caller-controlled values.
- Use locals for common or computed values.
- Use outputs for published values.

### Example
```hcl
variable "environment" {
  type = string
}

locals {
  machine_type = lookup(local.env_map, var.environment, "e2-small")
}

output "vm_name" {
  value = google_compute_instance.vm.name
}
```

### Key Points
- Clear separation of responsibilities.
- Safer module design.
- Easier to consume.

---

# Chapter 4: Variable Precedence

## Loading Order
Lowest to highest priority:
1. default values in `variables.tf`
2. environment variables (`TF_VAR_*`)
3. `terraform.tfvars`
4. `*.auto.tfvars`
5. `-var-file`
6. `-var`

### Example
```text
var.default < terraform.tfvars < prod.auto.tfvars < -var-file < -var
```

### Key Points
- Later sources override earlier ones.
- CLI overrides win last.
- Useful for environment-specific configuration.

## Best Practice
Use defaults for safe generic values, tfvars for environment-specific values, and CLI overrides only for one-off executions.

### Example
```hcl
variable "region" {
  default = "asia-south1"
}
```

### Key Points
- Use tfvars for environment settings.
- Avoid overusing CLI overrides.
- Keep configuration predictable.

## Interview Trap
`prod.auto.tfvars` wins over `terraform.tfvars` because Terraform loads auto tfvars after the standard tfvars file.

### Example
```text
terraform.tfvars -> prod.auto.tfvars
```

### Key Points
- Load order matters more than filename meaning.
- Auto tfvars are automatically applied.
- Good interview trap.

---

# Chapter 5: Module Interface Design

## Strong Typing
Use `map(object(...))` for complex inputs so Terraform can validate the shape of the input early.

### Example
```hcl
variable "servers" {
  type = map(object({
    machine_type = string
    zone         = string
    disk_size    = number
  }))
}
```

### Key Points
- Strong typing prevents bad input.
- `map(object(...))` is excellent for environment objects.
- Improves module readability.

## Why Type Constraints Matter
Types check structure, while validation checks business rules.

### Example
```hcl
validation {
  condition     = var.disk_size >= 20 && var.disk_size <= 2000
  error_message = "Disk size must be between 20 and 2000 GB."
}
```

### Key Points
- Types are structural.
- Validation is business logic.
- Use both together.

## Why Modules Feel Like APIs
- Inputs = variables
- Private implementation = locals
- Return values = outputs

### Example
```text
module API:
inputs -> internals -> outputs
```

### Key Points
- Treat modules like public APIs.
- Keep the interface small.
- Avoid exposing internals.

## Module Design Rule
Each module should expose the minimum interface needed by consumers and hide internal implementation details.

### Example
```hcl
output "vpc_id" { value = google_compute_network.vpc.id }
```

### Key Points
- Minimal interface.
- Easier maintenance.
- Better reuse.

---

# Chapter 6: Meta Arguments

## `count`
Use for identical resources that differ only by quantity.

### Example
```hcl
resource "google_compute_instance" "vm" {
  count = 3
  name  = "app-${count.index}"
}
```

### Key Points
- Index-based.
- Best for homogeneous resources.
- Avoid if identity matters.

## `for_each`
Use for uniquely identified resources with stable keys.

### Example
```hcl
resource "google_compute_instance" "vm" {
  for_each = var.servers
  name     = each.key
}
```

### Key Points
- Key-based.
- Stable identities.
- Best for named resources.

## `dynamic`
Use to generate repeated nested blocks inside a resource.

### Example
```hcl
dynamic "allow" {
  for_each = var.allowed_ports
  content {
    protocol = "tcp"
    ports    = [each.value]
  }
}
```

### Key Points
- Repeats nested blocks.
- Does not create resources.
- Useful for provider schemas with repeatable blocks.

## `depends_on`
Use only when Terraform cannot infer the dependency automatically.

### Example
```hcl
resource "google_compute_instance" "vm" {
  depends_on = [google_compute_firewall.allow_http]
}
```

### Key Points
- For hidden dependencies only.
- Reduces ambiguity.
- Overuse hurts parallelism.

## `lifecycle`
Use to control create/destroy/replace behavior.

### Example
```hcl
lifecycle {
  prevent_destroy = true
}
```

### Key Points
- Protects resources.
- Can reduce downtime.
- Must be used intentionally.

---

# Chapter 7: `count` vs `for_each`

## `count`
Best for homogeneous resources and simple scaling.

### Example
```hcl
count = 5
```

### Key Points
- Simple and compact.
- Indexes can shift.
- Not ideal for unique identities.

## `for_each`
Best for stable identities and different per-resource configuration.

### Example
```hcl
for_each = {
  frontend = "e2-small"
  backend  = "e2-medium"
}
```

### Key Points
- Stable mapping.
- Safer for additions/removals.
- Better for named objects.

## Enterprise Rule
If adding/removing one resource should not affect others, prefer `for_each`.

### Example
```text
Remove backend -> frontend and database stay unchanged
```

### Key Points
- Reduces accidental churn.
- Easier to manage in production.
- Avoid index drift.

---

# Chapter 8: `lifecycle`

## `create_before_destroy`
Creates the replacement before destroying the old resource.

### Example
```hcl
lifecycle {
  create_before_destroy = true
}
```

### Key Points
- Reduces downtime.
- Requires temporary coexistence.
- May fail for unique names.

## `prevent_destroy`
Stops accidental deletion of critical resources.

### Example
```hcl
lifecycle {
  prevent_destroy = true
}
```

### Key Points
- Good for prod databases, DNS, state buckets.
- Forces deliberate deletion.
- Useful guardrail.

## `ignore_changes`
Ignores drift for selected attributes during planning.

### Example
```hcl
lifecycle {
  ignore_changes = [labels]
}
```

### Key Points
- Keeps Terraform from fighting external ownership.
- State still refreshes.
- Best for externally managed attributes.

## `replace_triggered_by`
Forces replacement when another value changes.

### Example
```hcl
lifecycle {
  replace_triggered_by = [local.startup_script]
}
```

### Key Points
- Stronger replacement control.
- Useful when a dependent value changes.
- Should be used intentionally.

## Important Nuance
`ignore_changes` helps with externally managed drift, not with intentionally changing the Terraform configuration for a ForceNew attribute.

### Example
```text
Security team changes labels -> ignore_changes can tolerate
VM name changed in config -> ForceNew still matters
```

### Key Points
- External drift and config change are different.
- Ignore changes is not a general “no replacement” switch.
- Use only when ownership is intentional.

---

# Chapter 9: ForceNew Attributes

## Meaning
If the provider marks an attribute as ForceNew, changing it requires resource replacement.

### Example
```text
name change -> recreate
```

### Key Points
- Provider decides, not Terraform syntax alone.
- ForceNew means destroy and recreate.
- Plan will show `-/+`.

## Provider Responsibility
Terraform Core builds the plan, but the provider schema decides whether a change can happen in place or requires replacement.

### Example
```text
Terraform Core -> Provider schema -> plan outcome
```

### Key Points
- Core is generic.
- Provider is resource-specific.
- Cloud API capabilities drive behavior.

## Reading the Plan
| Symbol | Meaning |
|---|---|
| `+` | create |
| `-` | destroy |
| `~` | update in place |
| `-/+` | replace |

### Key Points
- `-/+` is a red flag for production.
- Always inspect replacement carefully.
- Not every change is in-place.

---

# Chapter 10: Data Sources and Remote State

## Data Source
Reads existing infrastructure directly from the cloud provider.

### Example
```hcl
data "google_compute_network" "vpc" {
  name = "shared-vpc"
}
```

### Key Points
- Direct cloud lookup.
- Read-only.
- No ownership transfer.

## Remote State
Reads outputs from another Terraform stack.

### Example
```hcl
data "terraform_remote_state" "network" {
  backend = "gcs"
  config = {
    bucket = "tf-state-bucket"
    prefix = "network/prod"
  }
}
```

### Key Points
- Ownership-aware.
- Consumes published outputs.
- Stronger team boundary.

## Best Use
- Use data sources for unmanaged or cloud-managed resources.
- Use remote state for resources owned by another Terraform stack.

### Example Decision
| Situation | Use |
|---|---|
| Manual resource | Data source |
| Another Terraform repo owns it | Remote state |
| New infra | Resource |

### Key Points
- Match tool to ownership.
- Prefer contracts over assumptions.
- Avoid coupling to names when possible.

---

# Chapter 11: Workspaces

## Workspaces Isolate State, Not Configuration
Use them for multiple instances of essentially the same configuration.

### Example
```text
workspace dev -> separate state
workspace prod -> separate state
same configuration file
```

### Key Points
- State isolation only.
- Same code, different state.
- Not a substitute for env folders.

## Not Ideal For
Use separate environment directories for long-lived environments such as Development, QA, and Production.

### Example
```text
environments/dev
environments/qa
environments/prod
```

### Key Points
- Better for distinct configs.
- Easier IAM/backend separation.
- Cleaner for enterprise delivery.

## Common Pitfall
Hardcoded names create conflicts across workspaces unless names are made workspace-aware.

### Example
```hcl
name = "dinesh-gcp-bucket" # conflicts across workspaces
```

### Key Points
- Same config means same names.
- Use `terraform.workspace` only when appropriate.
- Think carefully before relying on workspaces.

---

# Chapter 12: Providers, Aliases, and Modules

## Provider Configuration
A provider block configures access to a specific cloud account, project, region, or subscription.

### Example
```hcl
provider "google" {
  project = var.project_id
  region  = var.region
}
```

### Key Points
- Provider config is root-module level.
- It is injected into resources and modules.
- It should be kept reusable and environment-specific only where needed.

## Provider Aliases
Provider aliases let one Terraform execution use multiple provider configurations.

### Example
```hcl
provider "google" {
  project = "dev-project"
}

provider "google" {
  alias   = "prod"
  project = "prod-project"
}
```

### Key Points
- Useful for multiple projects/accounts.
- Useful for read/write across different scopes.
- Not the same as environments.

## Providers Map in Modules
Use the `providers` argument in a module block to inject the desired provider configuration.

### Example
```hcl
module "vm" {
  source = "../../modules/vm"

  providers = {
    google = google.prod
  }
}
```

### Key Points
- Keeps modules reusable.
- Avoids hardcoding provider alias inside module resources.
- Makes dependency injection explicit.

## Should modules hardcode `provider = google.prod`?
No. That makes the module less reusable. The root module should inject the provider via the `providers` map.

### Example
```hcl
resource "google_compute_instance" "vm" {
  name = var.name
}
```

### Key Points
- Child modules should not hardcode environment/provider alias choices.
- Let the caller decide the provider.
- Hardcoding hurts reuse.

## Provider Alias vs Environment Folder
- **Environment folders** = dev/qa/prod separate configs and state.
- **Provider aliases** = one execution talks to multiple projects/accounts/regions.

### Example
```text
environments/prod/main.tf  -> application project
provider alias google.network -> shared network project
```

### Key Points
- They solve different problems.
- They are often used together.
- Aliases do not replace environment folders.

---

# Chapter 13: Provisioners, `null_resource`, and `terraform_data`

## Provisioners
Provisioners perform side effects during resource lifecycle, such as shell commands or remote commands.

### Example
```hcl
resource "google_compute_instance" "vm" {
  provisioner "remote-exec" {
    inline = ["sudo yum update -y"]
  }
}
```

### Key Points
- Use sparingly.
- Not a replacement for Ansible or CI/CD orchestration.
- Can be brittle.

## `null_resource`
A `null_resource` creates nothing in the cloud but participates in Terraform state and lifecycle.

### Example
```hcl
resource "null_resource" "configure" {
  triggers = {
    script_hash = filesha256("deploy.sh")
  }

  provisioner "local-exec" {
    command = "./deploy.sh"
  }
}
```

### Key Points
- It is a placeholder resource.
- `triggers` control replacement.
- Provisioners run as part of resource lifecycle, not directly from `triggers`.

## `terraform_data`
`terraform_data` is a newer Terraform-managed data resource that covers many `null_resource` use cases more clearly.

### Example
```hcl
resource "terraform_data" "metadata" {
  input = {
    version = "2.0"
    owner   = "platform"
  }
}
```

### Key Points
- Better intent than overloading `null_resource` triggers.
- Useful as a data holder in the graph.
- Still not a cloud resource.

## `triggers` vs `input`
- `triggers` means change should replace the `null_resource`.
- `input` means Terraform should track the data in a dedicated resource.

### Example
```hcl
triggers = { checksum = filesha256("script.sh") }
input    = { version = "2.0" }
```

### Key Points
- `triggers` is replacement-oriented.
- `input` is data-oriented.
- The resource lifecycle still matters.

## Destroy-Time Cleanup
Destroy-time provisioners can help clean up external resources while the target system still exists.

### Example
```hcl
resource "terraform_data" "cleanup" {
  provisioner "local-exec" {
    when    = destroy
    command = "helm uninstall my-app"
  }
}
```

### Key Points
- Useful when external systems must be cleaned before a dependency disappears.
- Prefer owning the lifecycle with Terraform when possible.
- Modern orchestration should live in CI/CD or config management.

---

# Chapter 14: Validation, Preconditions, Postconditions

## Variable Validation
Variable validation checks input values before plan proceeds.

### Example
```hcl
variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "qa", "prod"], var.environment)
    error_message = "Environment must be dev, qa, or prod."
  }
}
```

### Key Points
- Best for inputs and format checks.
- Great for quick fail-fast behavior.
- Should not contain complex resource logic.

## Preconditions
Preconditions validate assumptions before a resource or data source is created/read.

### Example
```hcl
resource "google_compute_instance" "vm" {
  lifecycle {
    precondition {
      condition     = var.disk_size >= 20
      error_message = "Disk size must be at least 20 GB."
    }
  }
}
```

### Key Points
- Can use locals, variables, and data.
- Useful for business rules.
- Runs before resource creation.

## Postconditions
Postconditions validate the outcome after the resource is created or read.

### Example
```hcl
resource "google_compute_instance" "vm" {
  lifecycle {
    postcondition {
      condition     = self.deletion_protection
      error_message = "Deletion protection must be enabled."
    }
  }
}
```

### Key Points
- Validates actual result.
- Uses `self`.
- Does not mean automatic rollback.

## `self`
`self` refers to the current resource instance inside contexts such as postconditions, provisioners, and connection blocks.

### Example
```hcl
postcondition {
  condition = self.zone != ""
}
```

### Key Points
- Only valid where a current resource instance exists.
- Not for variables or locals.
- It is the current resource context.

---

# Chapter 15: Terraform Functions

## `lookup()`
Selects a value from a map using a key, with a default fallback.

### Example
```hcl
locals {
  machine_types = {
    dev  = "e2-small"
    prod = "c3-standard-8"
  }
}

machine_type = lookup(local.machine_types, var.environment, "e2-small")
```

### Key Points
- Great for environment selection.
- Avoids nested conditionals.
- Common with `locals.tf`.

## `merge()`
Combines multiple maps, with later maps taking precedence.

### Example
```hcl
locals {
  common_labels = {
    owner = "platform"
  }

  env_labels = {
    environment = var.environment
  }

  final_labels = merge(local.common_labels, local.env_labels)
}
```

### Key Points
- Useful for labels and tags.
- Later values win.
- Very common in production modules.

## `coalesce()`
Returns the first non-null, non-empty string.

### Example
```hcl
local.region = coalesce(var.region, "asia-south1")
```

### Key Points
- Good for fallback strings.
- Does not catch evaluation errors.
- Often paired with `lookup()`.

## `try()`
Returns the first expression that does not error.

### Example
```hcl
local.machine_type = try(var.vm.machine_type, "e2-medium")
```

### Key Points
- Great for optional nested attributes.
- Catches evaluation errors.
- Useful for safely reading complex objects.

## `can()`
Returns true/false if an expression can be evaluated successfully.

### Example
```hcl
local.has_owner = can(var.vm.owner)
```

### Key Points
- Boolean result only.
- Good for validation checks.
- Often used with `regex()`.

## `flatten()`
Converts a list of lists into a single list.

### Example
```hcl
flatten([[80,443],[8080],[22]])
```

### Key Points
- Useful before `distinct()` or `toset()`.
- Common when building lists from nested structures.

## `concat()`
Appends lists together.

### Example
```hcl
concat(["vm1"], ["vm2", "vm3"])
```

### Key Points
- Joins lists.
- Different from `merge()` which joins maps.

## `distinct()`
Removes duplicates from a list.

### Example
```hcl
distinct(["prod", "dev", "prod"])
```

### Key Points
- Useful after flattening.
- Helps make stable `for_each` inputs.

## `contains()`
Checks whether a collection contains a value.

### Example
```hcl
contains(["dev", "qa", "prod"], var.environment)
```

### Key Points
- Very common in validation blocks.
- Good for allow-list logic.

## `length()`
Returns the number of items in a collection or string length.

### Example
```hcl
length(var.instances)
```

### Key Points
- Useful with `count`.
- Also useful in validations.

## `keys()` / `values()`
Return keys or values from a map.

### Example
```hcl
keys(local.servers)
values(local.servers)
```

### Key Points
- Helpful for debugging and dynamic logic.

## `join()` / `split()`
Convert between lists and strings.

### Example
```hcl
join(",", ["dev", "qa", "prod"])
split(",", "dev,qa,prod")
```

### Key Points
- Useful in formatting and parsing.

## `toset()` / `tolist()` / `tomap()`
Type conversion helpers.

### Example
```hcl
toset(["dev", "prod", "prod"])
```

### Key Points
- `toset()` removes duplicates.
- Useful with `for_each`.
- `tolist()` and `tomap()` are conversion helpers.

---

# Chapter 16: Dynamic Dependencies and Ordering

## Implicit Dependency
Terraform automatically creates dependencies when one resource references another resource attribute.

### Example
```hcl
network = google_compute_network.vpc.id
```

### Key Points
- No explicit `depends_on` needed when reference exists.
- Terraform builds the graph automatically.

## Explicit Dependency
Use `depends_on` when Terraform cannot infer the dependency from references.

### Example
```hcl
depends_on = [google_compute_firewall.allow_http]
```

### Key Points
- Hidden runtime dependencies are valid reasons.
- Avoid overusing it.
- Overuse can reduce parallelism.

## Hidden Runtime Dependency
Sometimes a script or startup process depends on an external resource that Terraform cannot see.

### Example
```text
VM startup script needs firewall rule already present
```

### Key Points
- Use `depends_on` carefully.
- This is a classic interview trap.

---

# Chapter 17: Import and Moved Blocks

## Import
Import tells Terraform that an existing cloud resource should be tracked in state.

### Example
```bash
terraform import google_compute_network.vpc projects/my-project/global/networks/prod-vpc
```

### Key Points
- Adds existing cloud resources to state.
- Does not generate Terraform code.
- Requires a matching resource block.

## Import Blocks
Import blocks are top-level blocks that drive import as part of planning.

### Example
```hcl
import {
  to = google_compute_network.vpc
  id = "projects/my-project/global/networks/prod-vpc"
}
```

### Key Points
- Top-level block.
- Auto-discovered by Terraform.
- Usually placed in `imports.tf`.

## Moved Blocks
Moved blocks tell Terraform a resource address changed, so state can be remapped without destroying infrastructure.

### Example
```hcl
moved {
  from = google_compute_instance.vm
  to   = google_compute_instance.application_server
}
```

### Key Points
- Used for refactoring/renaming.
- Must accompany a config change.
- Prevents destroy/recreate.

## Import vs Moved
| Import | Moved |
|---|---|
| Cloud -> State | State address -> new state address |
| Existing resource not yet managed | Existing managed resource renamed/moved |
| One-time adoption | Refactoring aid |

### Key Points
- Different problems.
- Both are top-level planning instructions.

---

# Chapter 18: Versioning and Reproducibility

## `.terraform.lock.hcl`
Terraform’s dependency lock file records the exact provider versions and checksums selected for the project.

### Example
```hcl
provider "registry.terraform.io/hashicorp/google" {
  version = "6.2.3"
  hashes = ["h1:...", "zh:..."]
}
```

### Key Points
- Should usually be committed.
- Helps reproducibility.
- Stores integrity hashes.

## Provider Version Constraints
Version constraints define the acceptable provider versions.

### Example
```hcl
version = "~> 6.2"
```

### Key Points
- `~>` is the common enterprise choice.
- Exact pinning is possible.
- Constraint and lock file solve different problems.

## Exact Pinning
You can pin an exact provider version using `=`.

### Example
```hcl
version = "= 6.2.3"
```

### Key Points
- Very strict.
- Reduces version flexibility.
- Lock file still records checksums.

## Why Keep the Lock File Even When Pinned?
Even with exact version pinning, the lock file still provides checksum verification and standard dependency workflow. It is generally kept in enterprise repos.

### Key Points
- Exact pinning reduces the version-selection benefit.
- The lock file still adds integrity checks.
- Terraform will regenerate it if missing.

## Module Versioning
Version Git modules using tags or commits and registry modules using version constraints.

### Example
```hcl
source = "git::https://github.com/company/terraform-modules.git//vpc?ref=v1.2.0"
```

### Key Points
- Avoid `main` for production.
- Pin Git tags or commit SHAs.
- Registry modules use `version =`.

## Semantic Versioning
Most teams use MAJOR.MINOR.PATCH.

### Example
```text
1.2.3 -> 1.2.4 (patch)
1.2.3 -> 1.3.0 (minor)
1.2.3 -> 2.0.0 (major)
```

### Key Points
- Helps communicate breaking changes.
- Supports predictable upgrades.

---

# Chapter 19: Enterprise Terraform Architecture

## Folder Structure
A common enterprise structure separates reusable modules from environment-specific configurations.

### Example
```text
terraform/
├── modules/
│   ├── vm/
│   ├── network/
│   └── gke/
└── environments/
    ├── dev/
    ├── qa/
    └── prod/
```

### Key Points
- Modules contain reusable logic.
- Environments contain project-specific values.
- Keeps code maintainable and scalable.

## Environment Folders
Use separate environment folders when dev/qa/prod differ in project, backend, permissions, or resource shape.

### Example
```hcl
# environments/prod/main.tf
module "vm" {
  source = "../../modules/vm"
}
```

### Key Points
- Clearer than workspaces for distinct environments.
- Easier to reason about state and permissions.

## Module Responsibilities
Modules should focus on resource logic, not environment ownership.

### Example
```hcl
resource "google_compute_instance" "vm" {
  name = var.name
}
```

### Key Points
- Modules should be reusable.
- Root modules decide environment and provider wiring.

## State Separation
Keep state separated by environment or domain.

### Example
```text
prod-network.tfstate
prod-app.tfstate
```

### Key Points
- Reduces blast radius.
- Clarifies ownership.

## CI/CD Flow
Typical enterprise flow: format, validate, plan, review, approve, apply.

### Example
```text
GitHub -> PR -> terraform fmt -> terraform validate -> terraform plan -> approval -> apply
```

### Key Points
- Production should not be changed directly from laptops.
- Review before apply.

## Best Practices Summary
- Use modules for reuse.
- Use environment folders for actual environment differences.
- Keep state remote, locked, and versioned.
- Keep resource ownership single and clear.
- Keep provider aliases only for multi-project/multi-account execution.
- Use `import`/`moved` carefully during migration and refactoring.
- Prefer `lookup`, `merge`, `try`, `can`, `flatten`, `concat`, `distinct`, `contains`, `length`, and `toset` for clean expressions.

---

# Chapter 20: Quick Interview Recall
- Terraform is declarative and stateful.
- Backend stores state; provider talks to cloud.
- State is Terraform’s memory.
- Use `for_each` for identity, `count` for quantity.
- `dynamic` repeats nested blocks.
- `ignore_changes` tolerates approved external drift.
- ForceNew means replace.
- Workspaces isolate state, not config.
- `terraform_remote_state` is ownership-aware.
- `lookup()` and `merge()` are core enterprise functions.
- `null_resource` is a placeholder; `triggers` drive replacement.
- `terraform_data` is the cleaner modern data resource.
- `import {}` adopts existing cloud objects into state.
- `moved {}` remaps Terraform addresses without recreating infra.
- `.terraform.lock.hcl` locks provider versions and checksums.
- Module versioning should be explicit and reproducible.

---

# Chapter 21: Interview-Ready Summary
If you search this handbook for a topic, you should find:
- what it means,
- when to use it,
- a code example,
- key points,
- and the enterprise trade-off.

That is the intended source-of-truth format for this repository.
