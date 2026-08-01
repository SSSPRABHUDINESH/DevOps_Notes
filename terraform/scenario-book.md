# Terraform Production Scenario Book

## Scenario 1: Existing VPC owned by another team
Use a data source if the VPC exists outside your Terraform stack. Use `terraform_remote_state` if another Terraform stack owns it and exports outputs.

## Scenario 2: Existing subnet
Do not recreate the subnet. Consume it as an external dependency using a data source or remote state outputs.

## Scenario 3: State lost but infrastructure still exists
Recover from versioned remote state if possible. If not, rebuild ownership mapping using `terraform import`.

## Scenario 4: Two engineers apply at the same time
State locking should prevent concurrent writes to the same remote state.

## Scenario 5: Environment drift
Keep reusable modules and separate environment-specific variables so dev, qa, stage, and prod remain aligned.

## Scenario 6: Configuration drift
Run `terraform plan` to detect differences between Terraform configuration and actual infrastructure.

## Scenario 7: Backend migration
Use `terraform init -migrate-state` when moving state from local to remote storage.

## Scenario 8: Backend already exists
Use `terraform init -reconfigure` when the destination backend already contains the correct state and migration is not needed.

## Scenario 9: Release rollback
Terraform does not have a native rollback command. Roll back by reverting configuration in Git and then reviewing the plan before applying.

## Scenario 10: Parallelism and rate limits
Terraform can create independent resources in parallel, but provider API rate limits and quotas can slow or fail large deployments.

## Scenario 11: Resource ownership conflict
If Project A already owns a resource, Project B must not create the same resource. It should consume outputs or read the existing resource.

## Scenario 12: Outputs as a contract
Expose only the minimum required values to other stacks, such as VPC ID, subnet ID, region, or DNS zone.

## Scenario 13: State versioning
Enable object versioning in the remote backend so earlier state snapshots can be recovered when needed.

## Scenario 14: Force unlock
Use `terraform force-unlock` only when a stale lock remains after confirming that no Terraform process is still running.

## Scenario 15: Refresh behavior
Modern Terraform refreshes state automatically during plan/apply. The old standalone refresh command is deprecated.
