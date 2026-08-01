# Terraform Interview Companion

## 1. Why Terraform?
Terraform is declarative, supports many providers, uses a dependency graph, and manages state for safe enterprise infrastructure automation.

## 2. Terraform vs Ansible
Terraform provisions infrastructure. Ansible configures systems and applications. They complement each other.

## 3. Terraform vs Bash
Bash is imperative and harder to manage at scale. Terraform is declarative, idempotent, and stateful.

## 4. What is State?
Terraform State is Terraform's memory. It maps configuration resources to real cloud resources.

## 5. What is Backend?
A backend defines where state is stored and how state operations such as locking and migration are handled.

## 6. What is Remote State?
Remote state is centrally stored state used by multiple users or stacks. It enables collaboration and recovery.

## 7. Why use Outputs?
Outputs expose only the values other stacks need. They act as contracts between infrastructure layers.

## 8. What is Drift?
Drift is a difference between Terraform's desired state and the actual cloud infrastructure.

## 9. Configuration Drift vs Environment Drift
Configuration drift is mismatch inside one environment. Environment drift is unintended inconsistency between dev, qa, stage, or prod.

## 10. Why `terraform refresh` is deprecated?
Because it could update state directly without a reviewed plan. Modern Terraform refreshes automatically during plan and apply.

## 11. What is `terraform init -migrate-state`?
It moves state from one backend to another, such as local to GCS.

## 12. What is `terraform init -reconfigure`?
It tells Terraform to use the new backend configuration without migrating state.

## 13. Why state locking?
To prevent multiple Terraform operations from corrupting the same state.

## 14. What happens if state is lost?
Infrastructure still exists, but Terraform loses ownership mapping. Restore from versioned remote state or use `terraform import`.

## 15. Existing VPC or subnet?
If owned outside the current stack, read it using a data source. If another stack owns it, consume its outputs with `terraform_remote_state`.

## 16. Can two Terraform projects manage the same resource?
No. That causes conflicts because only one stack should own a resource.

## 17. What is serial?
Serial is the revision number of the state file.

## 18. What is lineage?
Lineage is the unique identity of a state file.

## 19. Why parallelism matters?
Terraform creates independent resources in parallel, but rate limits and quotas limit how far concurrency can go.

## 20. Interview takeaway
Strong Terraform answers explain not just the command, but the reason behind the design and the production trade-offs.
