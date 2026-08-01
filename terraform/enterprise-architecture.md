# Terraform Enterprise Architecture

## 1. Shared Module Pattern
A shared module should encapsulate reusable infrastructure such as VPC, subnets, firewall rules, compute instances, disks, and load balancers. Environment-specific values should be passed through variables.

## 2. Recommended Repository Structure
```text
terraform/
├── modules/
│   ├── network/
│   ├── compute/
│   ├── database/
│   └── monitoring/
└── environments/
    ├── dev/
    ├── qa/
    ├── stage/
    └── prod/
```

## 3. Ownership Model
Each resource should have one owner. If the networking team owns VPC and subnet resources, the application team should consume those via outputs or remote state instead of creating them again.

## 4. Outputs as Contracts
Outputs are the public contract between Terraform stacks. Use outputs to expose only what other stacks need, such as VPC IDs, subnet IDs, region, or private DNS names.

## 5. Remote State Collaboration
`terraform_remote_state` is the preferred mechanism when one Terraform stack must consume outputs from another stack. This avoids hardcoding and keeps stacks loosely coupled.

## 6. Existing Infrastructure Consumption
If a resource already exists outside the current stack, use a data source to read it. If it belongs to another Terraform stack, prefer remote state outputs.

## 7. State Separation by Environment
Use separate state files or prefixes for dev, qa, stage, and prod. Do not store all environments in a single state file.

## 8. Drift Prevention Strategy
Avoid manual changes in cloud consoles. Manage infrastructure only through Terraform, Git, code review, and CI/CD.

## 9. Backend Strategy
Use a remote backend with locking and versioning. For GCP, the GCS backend is a common enterprise choice.

## 10. Production Design Principles
- One stack owns one resource set.
- Reuse modules across environments.
- Keep environment differences in variables.
- Expose only necessary outputs.
- Store state remotely with locking and versioning.
- Avoid manual configuration drift.
