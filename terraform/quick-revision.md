# ⚡ Terraform Quick Revision

> Day-wise condensed notes for fast morning revision.
> 
> **Rule:** never remove earlier notes; only append and improve.

---

# Day 0 — Core Foundations

## 🎯 Must Remember
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

## 🧠 Memory Lines
- **Terraform is declarative and stateful.**
- **Backend stores state; provider talks to cloud API.**
- **State is Terraform's memory, not the infrastructure itself.**
- **One owner per resource.**

## ⚠ Common Mistakes
- Using one state for every environment.
- Recreating resources that already exist.
- Overusing `depends_on`.
- Disabling locking.
- Manually editing state.
- Treating `terraform_remote_state` as if it exposes the whole state.

## 📊 Key Comparisons
| Concept | Meaning |
|---|---|
| Provider | Manages cloud resources |
| Backend | Manages state storage and locking |
| Data source | Reads existing infrastructure |
| Resource | Creates and manages infrastructure |
| Remote state | Reads outputs from another Terraform stack |
| Drift | Desired state and reality do not match |

| Situation | Best Choice | Why |
|---|---|---|
| Existing VPC owned by another team | `terraform_remote_state` or data source | Reuse instead of recreate |
| Unmanaged existing VM | `terraform import` | Adopt into Terraform |
| New network for the project | Resource | Terraform should create it |
| Multiple engineers on same state | Remote backend + locking | Prevent corruption |

---

# Day 1 — State, Variables, Modules, Meta Arguments

## 🧩 State Commands
### 🎯 Must Remember
- `terraform state list` shows managed resource addresses.
- `terraform state show` shows one managed resource's attributes.
- `terraform state mv` moves a state address without touching infra.
- `terraform state rm` removes Terraform ownership without deleting infra.
- `terraform state pull` downloads current backend state as JSON.
- `terraform state push` uploads a validated local state to backend in recovery cases.
- `terraform import` adopts existing infrastructure into state.
- `terraform force-unlock` removes a stale lock only when safe.

### 🧠 Memory Lines
- **`state list` = what Terraform owns.**
- **`state show` = what Terraform knows about one resource.**
- **`state mv` = rename Terraform's address, not the cloud resource.**
- **`state rm` = forget the resource, do not delete it.**
- **`state pull` = read-only snapshot of remote state.**
- **`state push` = exceptional recovery write-back.**

### ⚠ Common Mistakes
- Thinking `state mv` changes cloud infra.
- Thinking `state rm` deletes the VM.
- Thinking `state pull` changes the backend.
- Using `state push` casually.

---

## 🧪 State Recovery & DR
### 🎯 Must Remember
- Remote backend versioning already gives historical copies.
- Manual backup via `terraform state pull > backup.tfstate` is still a useful checkpoint before risky state operations.
- If the backend is lost, restore the local backup as `terraform.tfstate`, update backend config, then run `terraform init -migrate-state`.
- Always verify with `terraform plan` after restoring state.

### 🧠 Memory Lines
- **Remote backend is the source of truth.**
- **`state pull` creates a local snapshot of that truth.**
- **After DR restore, always re-check with plan.**

### ⚠ Common Mistakes
- Using `-reconfigure` when you actually need migration.
- Forgetting to update the backend block to the new bucket.
- Restoring stale state without reviewing the plan.

---

## 🧾 Variables and Precedence
### 🎯 Must Remember
- Variables are external inputs.
- Defaults are the lowest priority.
- `terraform.tfvars` is auto-loaded.
- `*.auto.tfvars` overrides `terraform.tfvars`.
- `-var-file` overrides auto-loaded tfvars.
- `-var` has the highest priority.
- `prod.auto.tfvars` wins over `terraform.tfvars` because of load order, not because it says prod.

### 🧠 Memory Lines
- **The closer the value is to the command line, the higher the priority.**
- **`terraform.tfvars` is project-level, `*.auto.tfvars` is later-loaded and stronger.**

### ⚠ Common Mistakes
- Assuming `prod.tfvars` is automatically loaded.
- Thinking file name `prod` gives priority by meaning.
- Forgetting that CLI `-var` wins last.

---

## 🧠 Locals
### 🎯 Must Remember
- Locals are internal values or computed expressions.
- Use locals for values the caller should not override.
- Place locals in the narrowest scope needed.
- Module-only computed values belong in that module's `locals.tf`.
- Shared environment values can live in root environment `locals.tf`.
- Provider-level common labels in GCP can use `default_labels` instead of locals.

### 🧠 Memory Lines
- **Variables = external inputs.**
- **Locals = internal implementation.**
- **Use locals at the lowest necessary scope.**

### 📌 Quick Rule
| Need | Put it where? |
|---|---|
| Module-specific computed name | module `locals.tf` |
| Shared environment labels | environment `locals.tf` |
| Common GCP labels across resources | provider `default_labels` |

---

## 🧷 Outputs
### 🎯 Must Remember
- Variables are inputs.
- Outputs are return values.
- Outputs are the public interface of a Terraform module.
- Expose only what consumers need.
- Outputs are how teams publish stable values like VPC ID or subnet ID.

### 🧠 Memory Lines
- **Variables define what the module needs.**
- **Locals define how the module works.**
- **Outputs define what the module exposes.**

---

## 🏗️ Modules and Sources
### 🎯 Must Remember
- Modules package reusable infrastructure behind a clean interface.
- Keep one module focused on one responsibility.
- Use Git module sources for enterprise sharing.
- Use Terraform Registry modules for reusable public/community modules.
- Pin Git module versions with `?ref=`.

### 📊 Module Source Types
| Source | Example | When Used |
|---|---|---|
| Local path | `../../modules/vpc` | same repository |
| Git repository | `git::https://github.com/company/terraform-modules.git//vpc?ref=v1.2.0` | enterprise reusable modules |
| Terraform Registry | `terraform-google-modules/network/google` | reusable public modules |
| Private registry | enterprise registry | large organizations |

### 🧠 Memory Lines
- **Modules are APIs: inputs, internals, outputs.**
- **Pin Git modules with `?ref=` to keep deployments reproducible.**
- **A module should have one clear responsibility.**

### ⚠ Common Mistakes
- Letting a module manage too many unrelated resources.
- Forgetting to pin module versions.
- Reading tfvars directly inside child modules.

---

## 🧱 Strong Typing and Validation
### 🎯 Must Remember
- `map(object(...))` is ideal for rich environment/module input.
- Strong typing validates structure.
- Validation blocks enforce business rules.
- Types do not validate business meaning.
- Validation belongs in variable definitions and travels with the module.

### 🧠 Memory Lines
- **Type checks shape; validation checks rules.**
- **Use `map(object(...))` for related environment properties.**
- **Module validation is the first line of defense; CI/CD is extra governance.**

### ⚠ Common Mistakes
- Using `any` for complex inputs.
- Splitting related environment data into parallel maps.
- Leaving business rules only to CI pipelines.

---

## 🔁 Meta Arguments
### 🎯 Must Remember
- `count` = homogeneous resources, index-based.
- `for_each` = stable keys, unique resources.
- `dynamic` = repeated nested blocks inside a resource.
- `depends_on` = only when Terraform cannot infer dependency.
- `lifecycle` controls create/destroy/replace behavior.

### 🧠 Memory Lines
- **`count` is for quantity.**
- **`for_each` is for identity.**
- **`dynamic` repeats nested blocks.**
- **`depends_on` is for hidden operational dependencies only.**

### ⚠ Common Mistakes
- Using `count` for named resources with unique identities.
- Using `depends_on` everywhere and slowing down parallelism.
- Confusing `dynamic` with `for_each`.

---

## 🔒 Lifecycle
### 🎯 Must Remember
- `create_before_destroy` reduces downtime during replacement.
- `prevent_destroy` protects critical resources from accidental deletion.
- `ignore_changes` ignores selected attributes during planning.
- `replace_triggered_by` forces replacement on another change.
- `ignore_changes` does not mean the attribute is no longer refreshed.

### 🧠 Memory Lines
- **`prevent_destroy` for prod protection.**
- **`create_before_destroy` when temporary coexistence is possible.**
- **`ignore_changes` is for externally managed drift, not for core identity attributes.**

### ⚠ Common Mistakes
- Using `create_before_destroy` where coexistence is impossible.
- Treating `ignore_changes` like a workaround for all replacements.
- Ignoring identity attributes like VM name in GCE.

---

## 🧩 ForceNew
### 🎯 Must Remember
- ForceNew is decided by the provider schema.
- Terraform Core plans, the provider decides whether a change is in-place or replacement.
- GCE VM names are identity attributes and cannot be renamed in place.
- A plan showing `-/+` means replacement.

### 🧠 Memory Lines
- **Terraform doesn't guess; the provider schema decides.**
- **ForceNew = destroy and recreate.**
- **If the cloud API can't rename it, Terraform can't either.**

### 📊 Plan Symbols
| Symbol | Meaning |
|---|---|
| `+` | create |
| `-` | destroy |
| `~` | update in place |
| `-/+` | replace |

---

## 🧭 Data Sources vs Remote State
### 🎯 Must Remember
- Data sources read directly from the cloud provider.
- Remote state reads outputs from another Terraform stack.
- Prefer remote state when another Terraform configuration owns the resource.
- Prefer data sources when the resource exists outside Terraform ownership.

### 🧠 Memory Lines
- **Data source = read cloud.**
- **Remote state = read another Terraform stack's outputs.**
- **Use the owner's published interface, not internal implementation.**

### ⚠ Common Mistakes
- Using data sources to couple to another team's internal resource naming.
- Using remote state for unmanaged/manual resources.

---

## 🪟 Workspaces
### 🎯 Must Remember
- Workspaces isolate state, not configuration.
- Good for multiple instances of the same configuration.
- Not ideal for Dev/QA/Prod.
- Separate environment directories are the enterprise choice for long-lived environments.
- Hardcoded names can cause conflicts across workspaces.

### 🧠 Memory Lines
- **Workspaces isolate state, not config.**
- **Use environment folders for Dev/QA/Prod.**
- **Use workspaces for same code, same pattern, different state.**

### ⚠ Common Mistakes
- Using workspaces as a substitute for proper environment architecture.
- Expecting workspaces to solve different project IDs, IAM, or backends.

---

## 🛠️ Provisioners
### 🎯 Must Remember
- Terraform provisioners are a last resort.
- Terraform provisions infrastructure; Ansible configures software and OS.
- GitHub Actions should orchestrate the pipeline.
- Avoid using provisioners to install packages or applications.

### 🧠 Memory Lines
- **Terraform provisions; Ansible configures; GitHub Actions orchestrates.**
- **Provisioners are for rare bootstrapping, not primary configuration.**

### ⚠ Common Mistakes
- Installing Docker, Java, or apps via Terraform provisioners.
- Mixing orchestration and configuration management into one tool.

---

## 🧩 Dynamic Blocks
### 🎯 Must Remember
- Dynamic blocks generate repeated nested blocks inside a resource.
- Use them when the provider schema has repeatable nested configuration.
- They are not a replacement for resources or `for_each`.

### 🧠 Memory Lines
- **`for_each` creates resources; `dynamic` creates nested blocks.**
- **Repeat what the provider expects inside the resource, not the resource itself.**

---

## 🔍 `lookup()` and Environment Maps
### 🎯 Must Remember
- `lookup(map, key, default)` returns a map value or a default.
- Put common environment maps in `locals.tf` when they are internal logic.
- Use `map(object(...))` when an environment contains multiple properties.

### 🧠 Memory Lines
- **`lookup()` is map selection with fallback.**
- **`locals.tf` is the right place for internal environment mappings.**
- **Group related environment data into one object, not many parallel maps.**

### ⚠ Common Mistakes
- Writing nested ternary expressions for environment selection.
- Using multiple maps that can drift out of sync.

---

## 🌍 GCP Labels
### 🎯 Must Remember
- `default_labels` is the GCP-native way to apply common labels to supported resources.
- Use `merge()` when you need environment-specific or resource-specific overrides.
- `default_labels` is provider-specific; `merge()` is provider-agnostic.

### 🧠 Memory Lines
- **For GCP common labels, prefer provider-level `default_labels`.**
- **Use `merge()` for overrides and cross-provider patterns.**

---

## 🧠 30-Second Recall
- `state list` = managed resources
- `state show` = one resource details
- `state mv` = rename state address
- `state rm` = forget resource only
- `state pull` = snapshot remote state
- `state push` = recovery write-back
- `import` = adopt existing infrastructure
- `count` = quantity
- `for_each` = identity
- `dynamic` = nested blocks
- `depends_on` = hidden dependency only
- `lifecycle` = control replacement/deletion behavior
- `ForceNew` = provider says replace
- `lookup()` = map with fallback
- `map(object(...))` = rich typed config
- `default_labels` = common GCP labels
- `merge()` = combine maps
- `workspaces` = state isolation only

---

# Global Commands & Flags Cheat Sheet

| Command / Flag | Purpose |
|---|---|
| `terraform init` | initialize working directory and backend |
| `terraform init -migrate-state` | migrate state to a new backend |
| `terraform init -reconfigure` | reconfigure backend without migrating |
| `terraform plan` | preview changes |
| `terraform apply` | apply changes |
| `terraform destroy` | destroy managed infrastructure |
| `terraform import` | adopt existing infrastructure |
| `terraform state list` | list managed resources |
| `terraform state show` | inspect one resource |
| `terraform state mv` | move state address |
| `terraform state rm` | remove ownership only |
| `terraform state pull` | download current state |
| `terraform state push` | upload validated local state |
| `terraform force-unlock` | remove stale lock |
| `-parallelism` | control concurrency |
| `-auto-approve` | skip confirmation prompt |
| `-replace` | force resource recreation |
| `-refresh-only` | refresh state without config change |
| `-target` | apply selected resources only |
| `-var` | pass a single variable |
| `-var-file` | pass a variable file |

---

# Day 1 High Probability Questions
- Why use `terraform_remote_state` instead of data source?
- Why prefer `for_each` over `count` for unique resources?
- Why use `lookup()` with locals?
- Why use `map(object(...))`?
- Why is `default_labels` better than duplicating labels in every tfvars file?
- Why should provisioners be avoided?
- Why is `depends_on` often unnecessary?
- Why is `ignore_changes` not a general workaround for ForceNew?
- Why are workspaces not ideal for Dev/QA/Prod?

---

# Day 1 Interview Memory Blocks
- **Modules are APIs.**
- **Variables are inputs.**
- **Locals are private implementation.**
- **Outputs are return values.**
- **`count` is quantity; `for_each` is identity.**
- **`dynamic` repeats nested blocks.**
- **`ignore_changes` suppresses drift in selected attributes, but doesn't rewrite reality.**
- **Provider schema decides ForceNew vs update.**
- **`terraform_remote_state` is ownership-aware collaboration.**
- **Workspaces isolate state, not configuration.**
