# 🛠 Terraform Commands Cheat Sheet

> This file contains Terraform commands and what each command is used for.
> 
> It is intentionally concise and interview-friendly.

---

## Core Commands

| Command | Used For |
|---|---|
| `terraform init` | Initialize a working directory, download providers/modules, and set up the backend. |
| `terraform validate` | Validate configuration syntax and internal consistency. |
| `terraform fmt` | Format Terraform files to standard style. |
| `terraform plan` | Show what Terraform will create, update, or destroy. |
| `terraform apply` | Apply the planned infrastructure changes. |
| `terraform destroy` | Destroy all resources managed by the configuration/state. |
| `terraform show` | Display the current state or a saved plan in readable form. |
| `terraform output` | Show output values from the current state. |
| `terraform console` | Open an interactive REPL for Terraform expressions. |
| `terraform providers` | Show providers required by the configuration and state. |
| `terraform graph` | Generate a dependency graph of resources. |
| `terraform version` | Display Terraform version information. |

---

## State Commands

| Command | Used For |
|---|---|
| `terraform state list` | List all resources currently in state. |
| `terraform state show <address>` | Show detailed attributes of a resource in state. |
| `terraform state mv <src> <dst>` | Move a resource address in state without changing real infrastructure. |
| `terraform state rm <address>` | Remove a resource from state without deleting the real resource. |
| `terraform state pull` | Download the raw state file from the backend. |
| `terraform state push <file>` | Upload a local state file to the backend. Use with extreme care. |
| `terraform state replace-provider` | Replace provider source addresses in state during provider migration. |
| `terraform import <addr> <id>` | Import existing infrastructure into Terraform state. |
| `terraform taint <address>` | Mark a resource for recreation on next apply. Deprecated in newer workflows, prefer `-replace`. |
| `terraform untaint <address>` | Remove a taint mark from a resource. |

---

## Plan / Apply Safety Options

| Option | Used For |
|---|---|
| `-auto-approve` | Skip interactive approval during apply. Use carefully. |
| `-lock=false` | Disable state locking. Dangerous in shared environments. |
| `-lock-timeout=30s` | Wait for a state lock for a limited time. |
| `-parallelism=10` | Control how many operations Terraform runs at once. |
| `-refresh-only` | Refresh state without proposing infrastructure changes. |
| `-target=<address>` | Limit planning/apply to specific resources. Use only for special cases. |
| `-replace=<address>` | Force recreation of a specific resource. Preferred over `taint`. |

---

## Variable and Input Commands

| Command / Option | Used For |
|---|---|
| `-var 'key=value'` | Pass a single variable value from the CLI. |
| `-var-file=path.tfvars` | Load variable values from a file. |
| `terraform plan -var-file=prod.tfvars` | Plan using a specific variable file. |
| `terraform apply -var-file=prod.tfvars` | Apply using a specific variable file. |

---

## Workspace Commands

| Command | Used For |
|---|---|
| `terraform workspace list` | List all workspaces. |
| `terraform workspace show` | Show the currently selected workspace. |
| `terraform workspace new <name>` | Create a new workspace. |
| `terraform workspace select <name>` | Switch to an existing workspace. |
| `terraform workspace delete <name>` | Delete a workspace. |

---

## Backend and Initialization Commands

| Command | Used For |
|---|---|
| `terraform init -migrate-state` | Move local state to a new backend during backend change. |
| `terraform init -reconfigure` | Reconfigure backend settings without migrating state. |
| `terraform login` | Log in to Terraform Cloud / HCP Terraform when required. |
| `terraform logout` | Remove saved credentials for Terraform Cloud / HCP Terraform. |

---

## Import / Refactoring Commands

| Command | Used For |
|---|---|
| `terraform import` | Adopt existing infrastructure into Terraform state. |
| `terraform state mv` | Refactor resource addresses without recreation. |
| `terraform state rm` | Remove ownership from state while leaving real infra intact. |
| `terraform force-unlock <lock-id>` | Release a stale state lock. Use only after confirming no Terraform operation is active. |

---

## Useful Interview Notes

| Topic | Remember |
|---|---|
| `init` | Initializes providers, modules, and backend. |
| `plan` | Preview, do not change real infra. |
| `apply` | Makes changes to real infra. |
| `destroy` | Removes managed infra. |
| `state mv` | Refactoring helper, not a cloud operation. |
| `state rm` | Removes ownership, not the resource itself. |
| `import` | Adopts existing resources. |
| `-target` | Emergency/special-case option, not normal workflow. |
| `-replace` | Controlled recreation of one resource. |
| `-lock=false` | Avoid in teams because it can corrupt state. |

---

## Quick Production Guidance
- Use `terraform plan` before every apply.
- Avoid `-target` unless you know exactly why you need it.
- Avoid `-lock=false` in shared backends.
- Prefer `-replace` over the old `taint` workflow.
- Use `state mv` and `moved {}` for refactoring, not manual destroy/create.
- Use `import` to adopt existing infrastructure, not recreate it.
