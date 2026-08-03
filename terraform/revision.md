# ⚡ Terraform Revision Cheat Sheet

> Quick last-minute Terraform revision.
> 
> Use this file for rapid interview prep. It focuses on the most important concepts, comparisons, and commands in a compact format.

---

## 1) Core Ideas

| Topic | Remember |
|---|---|
| Terraform | Declarative IaC tool |
| Declarative | Describe desired end state |
| Idempotent | Repeated apply should not change anything if already aligned |
| State | Terraform’s memory of real infrastructure |
| Provider | Plugin that talks to cloud/API |
| Resource | Managed infrastructure object |
| Data source | Read-only lookup of existing infrastructure |
| Backend | Where state is stored |
| Graph | Terraform executes dependency graph, not file order |

---

## 2) Terraform Workflow

```text
Write .tf files -> init -> validate -> plan -> apply -> state update
```

Key points:
- `init` downloads providers and modules.
- `plan` shows intended changes.
- `apply` makes real changes.
- Terraform refreshes cloud state before planning.

---

## 3) Variables / Locals / Outputs

| Concept | Use |
|---|---|
| Variable | Input from caller or environment |
| Local | Internal computed value |
| Output | Published value for consumers |

### Variable precedence
Lowest to highest:
1. default in `variables.tf`
2. `TF_VAR_*`
3. `terraform.tfvars`
4. `*.auto.tfvars`
5. `-var-file`
6. `-var`

### Quick notes
- Use `variables` for inputs.
- Use `locals` for internal logic.
- Use `outputs` to expose values.
- `map(object(...))` is great for structured inputs.

---

## 4) Useful Functions

| Function | Use |
|---|---|
| `lookup()` | Get map value with fallback |
| `merge()` | Combine maps |
| `try()` | Return first expression that does not error |
| `can()` | Check whether expression is valid |
| `coalesce()` | First non-null / non-empty string |
| `contains()` | Check list/set membership |
| `flatten()` | Flatten nested lists |
| `concat()` | Join lists |
| `keys()` / `values()` | Get map keys or values |
| `length()` | Count items |
| `split()` / `join()` | Convert between strings and lists |
| `toset()` | Convert list to set |

### Quick memory trick
- `lookup` = read map
- `merge` = join maps
- `try` = fallback on error
- `can` = boolean check

---

## 5) Meta Arguments

| Meta Argument | Use |
|---|---|
| `count` | Identical resources, quantity-based |
| `for_each` | Unique resources, stable identity |
| `depends_on` | Explicit dependency when Terraform cannot infer it |
| `lifecycle` | Control create/update/delete behavior |
| `provider` | Choose provider config for a resource |
| `providers` | Pass provider aliases into modules |

### count vs for_each
- `count` = simple, index-based, identical resources
- `for_each` = better for unique resources and stable addresses

### dynamic blocks
- Use for repeated **nested blocks**, not repeated resources.

---

## 6) Lifecycle

| Setting | Use |
|---|---|
| `create_before_destroy` | Replace resource with less downtime |
| `prevent_destroy` | Block accidental deletion |
| `ignore_changes` | Ignore selected drift |
| `replace_triggered_by` | Force replacement when dependency changes |

### Important
- `ignore_changes` is not a blanket “ignore all drift” switch.
- ForceNew attributes still require replacement.

---

## 7) Validation / Preconditions / Postconditions

| Feature | Use |
|---|---|
| Variable validation | Validate input values early |
| Precondition | Check a rule before create/read |
| Postcondition | Check a rule after create/read |
| `self` | Current resource instance inside postcondition / provisioner context |

### Quick note
- Validation = input check
- Precondition = before resource action
- Postcondition = after resource action

---

## 8) Dependencies

| Type | Use |
|---|---|
| Implicit dependency | Created by references |
| Explicit dependency | Added with `depends_on` |
| Transitive dependency | A depends on B, B depends on C, so A depends on C indirectly |

### Quick note
Terraform executes the graph, not the files.

---

## 9) State

| Topic | Remember |
|---|---|
| State | Terraform memory |
| Remote state | Team-friendly shared state |
| Locking | Prevents concurrent writes |
| Drift | Cloud differs from configuration |
| Recovery | Restore versioned state or import resources |

### State commands
| Command | Use |
|---|---|
| `terraform state list` | List addresses in state |
| `terraform state show` | Show a managed resource |
| `terraform state mv` | Move state address without changing cloud infra |
| `terraform state rm` | Remove Terraform ownership without deleting resource |
| `terraform state pull` | Download raw state |
| `terraform state push` | Upload raw state carefully |
| `terraform import` | Adopt existing infrastructure |
| `terraform force-unlock` | Remove stale lock carefully |
| `terraform state replace-provider` | Replace provider source in state |

### Quick note
- One resource should have one owner.
- Do not edit state manually unless absolutely necessary.

---

## 10) Backend / Workspace

| Topic | Remember |
|---|---|
| Backend | State storage and state operations |
| Workspace | Separate state, not separate configuration |

### Workspace rule
Use workspaces only when configuration is the same and only state differs.
For dev/qa/prod with different config, prefer separate environment folders.

---

## 11) Providers / Aliases

| Topic | Remember |
|---|---|
| Provider | Talks to cloud/API |
| Alias | Second config for same provider |
| `providers` map | Pass alias into module |

### Quick note
- Child modules should not hardcode provider configs.
- Root module injects provider aliases.
- Useful for multi-project / multi-account / shared VPC designs.

---

## 12) Modules

| Topic | Remember |
|---|---|
| Root module | Top-level deployment entry point |
| Child module | Reusable building block |
| Module versioning | Use Git tags / registry versions |

### Quick note
- Modules should be reusable.
- Environment differences belong in root config or environment folders.
- Git modules should be pinned with `?ref=`.

---

## 13) Import / Moved / Refactor

| Topic | Remember |
|---|---|
| `terraform import` | Add existing cloud resource to state |
| `import {}` block | Declarative import instruction |
| `moved {}` | Remap resource address in state |
| `terraform state mv` | Manual state refactor command |

### Quick note
- Import = adopt existing infra
- Moved = refactor without recreation
- State mv = CLI version of refactor

---

## 14) Provisioners / null_resource / terraform_data

| Topic | Remember |
|---|---|
| Provisioners | Last-resort side effects |
| `null_resource` | Placeholder resource with lifecycle/triggers |
| `terraform_data` | Cleaner Terraform-native data holder |
| `triggers` | Cause `null_resource` replacement |

### Quick note
- Terraform should provision infrastructure; Ansible should configure software.
- Provisioners are usually a last resort.
- `triggers` do not directly run the provisioner; they trigger replacement.

---

## 15) Versioning

| Topic | Remember |
|---|---|
| `required_providers` | Allowed provider version range |
| `.terraform.lock.hcl` | Exact selected provider version + hashes |
| `~>` | Pessimistic constraint |
| `=` | Exact pin |

### Quick note
- `~> 6.2` means compatible 6.x releases.
- `.terraform.lock.hcl` improves reproducibility and checksum verification.
- Module versioning is separate from provider versioning.

---

## 16) Highly Asked Comparisons

| Compare | Key Difference |
|---|---|
| Variable vs Local | Input vs internal computed value |
| Data source vs Remote state | Cloud lookup vs another Terraform stack output |
| Import vs Moved | Adopt infra vs refactor state address |
| `count` vs `for_each` | Quantity vs identity |
| `null_resource` vs `terraform_data` | Old side-effect placeholder vs clearer data holder |
| Workspaces vs Environment folders | State isolation vs config isolation |
| `try()` vs `can()` | Return fallback value vs boolean validity check |
| `ignore_changes` vs `prevent_destroy` | Ignore selected drift vs block deletion |

---

## 17) Quick Interview Nuggets

- Configuration describes.
- State remembers.
- Cloud is reality.
- Terraform compares all three.
- One resource = one owner.
- Terraform executes a graph.
- File order does not matter.
- `for_each` is usually better than `count` for unique resources.
- `depends_on` should be used only when necessary.
- Workspaces are not a replacement for separate environments.
- Prefer `moved {}` for refactoring and `import` for adoption.
- Prefer `-replace` over old `taint` workflows.
- Avoid `-lock=false` in shared backends.
- Prefer `remote state` when another Terraform stack owns the resource.
- Use provider aliases for multi-project / multi-account access.

---

## 18) Commands to Remember

| Command | Use |
|---|---|
| `terraform init` | Initialize working directory |
| `terraform validate` | Validate configuration |
| `terraform fmt` | Format files |
| `terraform plan` | Preview changes |
| `terraform apply` | Apply changes |
| `terraform destroy` | Destroy managed infra |
| `terraform show` | Show state/plan |
| `terraform output` | Show outputs |
| `terraform console` | Interactive expression console |
| `terraform graph` | Show dependency graph |
| `terraform workspace list` | List workspaces |
| `terraform workspace show` | Show current workspace |
| `terraform workspace select` | Switch workspace |
| `terraform workspace new` | Create workspace |

---

## 19) Final 30-Second Revision

- Terraform is declarative, idempotent IaC.
- Providers talk to cloud APIs.
- State is Terraform memory.
- Use locals for internal logic and variables for inputs.
- Use `count` for quantity, `for_each` for identity.
- `dynamic` is for repeated nested blocks.
- `lifecycle` controls replacement and deletion behavior.
- Use `import` to adopt existing resources.
- Use `moved {}` or `state mv` to refactor without recreation.
- Commit `.terraform.lock.hcl` for reproducible providers.
- Prefer separate environment folders for real dev/qa/prod differences.
