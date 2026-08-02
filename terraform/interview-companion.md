# 🎤 Terraform Interview Companion

> Senior-level interview Q&A aligned to the living handbook.

---

## Chapter 1 — Foundations

### Q1. Why Terraform?
**Answer:** Terraform is declarative, supports many providers, uses a dependency graph, and manages state for safe enterprise infrastructure automation.

### Q2. Terraform vs Ansible?
**Answer:** Terraform provisions infrastructure. Ansible configures systems and applications. They complement each other.

### Q3. What is state?
**Answer:** Terraform State is Terraform's memory. It maps configuration resources to real cloud resources.

### Q4. What is backend?
**Answer:** A backend defines where state is stored and how state operations such as locking and migration are handled.

### Q5. What is remote state?
**Answer:** Remote state is centrally stored state used by multiple users or stacks. It enables collaboration and recovery.

### Q6. Why use outputs?
**Answer:** Outputs expose only the values other stacks need. They act as contracts between infrastructure layers.

### Q7. What is drift?
**Answer:** Drift is a difference between Terraform's desired state and the actual cloud infrastructure.

### Q8. Configuration drift vs environment drift?
**Answer:** Configuration drift is mismatch inside one environment. Environment drift is unintended inconsistency between dev, qa, stage, or prod.

### Q9. Why is `terraform refresh` deprecated?
**Answer:** Because it could update state directly without a reviewed plan. Modern Terraform refreshes automatically during plan and apply.

### Q10. What is `terraform init -migrate-state`?
**Answer:** It moves state from one backend to another, such as local to GCS.

### Q11. What is `terraform init -reconfigure`?
**Answer:** It tells Terraform to use the new backend configuration without migrating state.

### Q12. Why state locking?
**Answer:** To prevent multiple Terraform operations from corrupting the same state.

### Q13. What happens if state is lost?
**Answer:** Infrastructure still exists, but Terraform loses ownership mapping. Restore from versioned remote state or use `terraform import`.

### Q14. Existing VPC or subnet?
**Answer:** If owned outside the current stack, read it using a data source. If another stack owns it, consume its outputs with `terraform_remote_state`.

### Q15. Can two Terraform projects manage the same resource?
**Answer:** No. That causes conflicts because only one stack should own a resource.

---

## Chapter 2 — State Commands

### Q16. What does `terraform state list` do?
**Answer:** Lists all resource addresses currently managed by Terraform state.

### Q17. What does `terraform state show` do?
**Answer:** Shows detailed attributes for one managed resource.

### Q18. What does `terraform state mv` do?
**Answer:** Moves a resource address in state without touching real infrastructure.

### Q19. What does `terraform state rm` do?
**Answer:** Removes Terraform ownership without deleting the real infrastructure.

### Q20. What does `terraform state pull` do?
**Answer:** Downloads the current state from the backend as JSON.

### Q21. What does `terraform state push` do?
**Answer:** Uploads a validated local state file to the backend in recovery scenarios.

### Q22. What does `terraform import` do?
**Answer:** Adopts existing infrastructure into Terraform state.

### Q23. What does `terraform force-unlock` do?
**Answer:** Removes stale state locks when you are certain no Terraform operation is active.

### Q24. Is `terraform state pull` a backup?
**Answer:** It is a local snapshot of remote state; it becomes a backup when you redirect output to a file.

### Q25. Does `ignore_changes` stop state refresh?
**Answer:** No. Terraform still refreshes state with the real-world value, but it ignores the specified attribute in the plan.

### Q26. Does `ignore_changes` apply well to VM name in GCE?
**Answer:** Practically no. VM name is an identity attribute and GCE does not support renaming in place, so ignoring it has no real value.

---

## Chapter 3 — Variables, Locals, Outputs

### Q27. Variables vs locals?
**Answer:** Variables are external inputs; locals are internal computed values controlled by the module.

### Q28. Outputs vs variables?
**Answer:** Variables are inputs to a module; outputs are return values from a module.

### Q29. Where should common environment maps go?
**Answer:** Usually in `locals.tf` when the caller should not change the mapping.

### Q30. Why use strong types like `map(object(...))`?
**Answer:** They validate input shape early, improve readability, and make module contracts safer.

### Q31. Why use variable validation?
**Answer:** Strong types validate structure, while validation blocks enforce business rules such as allowed disk sizes or allowed environments.

### Q32. Where should module variables be declared?
**Answer:** In the module where they are consumed. The root module passes values into child modules.

### Q33. Can child modules read tfvars directly?
**Answer:** No. tfvars are loaded for the root module, which then passes values into child modules.

### Q34. Why use `lookup()`?
**Answer:** To fetch values from a map with a default fallback instead of using nested conditionals.

### Q35. Why use `map(object(...))` instead of parallel maps?
**Answer:** It keeps related properties together and avoids synchronization issues across multiple separate maps.

### Q36. Why use `default_labels` in GCP?
**Answer:** It automatically applies common organizational labels across supported Google resources and removes duplication.

### Q37. Why still use `merge()`?
**Answer:** For environment-specific or resource-specific overrides, or when working in multi-cloud / provider-independent Terraform.

### Q38. `merge()` vs `default_labels`?
**Answer:** `default_labels` is provider-specific and ideal for GCP common labels; `merge()` is provider-agnostic and useful for combining maps.

---

## Chapter 4 — Meta Arguments

### Q39. When do you use `count`?
**Answer:** For homogeneous resources that differ only by quantity.

### Q40. When do you use `for_each`?
**Answer:** For uniquely identified resources with stable keys and per-resource configuration.

### Q41. When do you use `dynamic`?
**Answer:** When you need to repeat nested blocks inside a resource.

### Q42. When do you use `depends_on`?
**Answer:** Only when Terraform cannot infer the dependency automatically.

### Q43. What is `create_before_destroy` for?
**Answer:** It reduces downtime by creating the replacement before deleting the old resource.

### Q44. What is `prevent_destroy` for?
**Answer:** It protects critical resources from accidental deletion.

### Q45. What is `ignore_changes` for?
**Answer:** It tells Terraform to ignore drift for selected attributes during planning.

### Q46. What is `replace_triggered_by` for?
**Answer:** It forces replacement when a different value or resource changes.

### Q47. `count` vs `for_each`?
**Answer:** `count` uses numeric indexes and is best for identical resources; `for_each` uses stable keys and is best for unique resources.

### Q48. Why prefer `for_each` over `count` for named resources?
**Answer:** Because stable keys prevent unnecessary replacement when one instance is removed.

---

## Chapter 5 — ForceNew and Provider Behavior

### Q49. What decides whether Terraform updates or recreates a resource?
**Answer:** The provider schema decides whether an attribute is updatable or ForceNew.

### Q50. What is a ForceNew attribute?
**Answer:** An attribute change that requires destroy-and-recreate rather than an in-place update.

### Q51. Can `ignore_changes` suppress a ForceNew replacement?
**Answer:** If the change is external drift on an ignored attribute, yes; if you intentionally change the configuration, Terraform still follows provider behavior.

### Q52. Can a GCE VM name be changed without recreation?
**Answer:** No. GCE VM names are identity attributes and cannot be renamed in place.

### Q53. Why is the VM name ForceNew?
**Answer:** Because the cloud platform doesn't support renaming the existing VM; a different name means a different resource.

### Q54. How do you read a Terraform plan quickly?
**Answer:** `+` create, `-` destroy, `~` in-place update, `-/+` replace.

---

## Chapter 6 — Modules and Sources

### Q55. Why use modules?
**Answer:** To package reusable infrastructure behind a clean interface with variables and outputs.

### Q56. What should be inside a module?
**Answer:** A focused responsibility such as VPC, VM, GKE, or Cloud SQL.

### Q57. What is a module source?
**Answer:** The location from where Terraform loads a module.

### Q58. What is a Git module source example?
**Answer:** `git::https://github.com/company/terraform-modules.git//vpc?ref=v1.2.0`

### Q59. What is a Terraform Registry module source example?
**Answer:** `terraform-google-modules/network/google`

### Q60. Why pin module versions with `?ref=`?
**Answer:** To make infrastructure deployments reproducible and avoid accidental breaking changes from the latest branch.

### Q61. Why use `terraform_remote_state` between teams?
**Answer:** To consume the owning team's published outputs instead of depending on internal implementation details.

### Q62. Data source vs `terraform_remote_state`?
**Answer:** A data source reads directly from the cloud provider; remote state reads outputs from another Terraform stack.

### Q63. Which should be used for a VPC owned by the networking team?
**Answer:** `terraform_remote_state` is preferred because it respects ownership boundaries and published outputs.

---

## Chapter 7 — Workspaces

### Q64. What do workspaces isolate?
**Answer:** State, not configuration.

### Q65. Are workspaces recommended for dev/qa/prod?
**Answer:** Generally no; separate environment directories are cleaner for long-lived environments.

### Q66. When are workspaces useful?
**Answer:** Temporary sandboxes or multiple instances of essentially identical infrastructure.

### Q67. What problem occurs with hardcoded resource names across workspaces?
**Answer:** Resource name collisions because the configuration stays identical across workspaces.

### Q68. Why do enterprises prefer environment directories?
**Answer:** Because Dev, QA, and Prod usually need different projects, backends, IAM, networking, and pipelines.

---

## Chapter 8 — Provisioners

### Q69. What are provisioners?
**Answer:** Mechanisms for running scripts or copying files after resource creation.

### Q70. Why do enterprises avoid them?
**Answer:** They blur infrastructure and configuration responsibilities, are harder to reason about, and can leave partial failures.

### Q71. Terraform vs Ansible?
**Answer:** Terraform creates infrastructure; Ansible configures software and operating systems.

### Q72. What should GitHub Actions do in this flow?
**Answer:** Orchestrate the pipeline: checkout, build, test, deploy, call Terraform, then call Ansible.

### Q73. Should a PR that uses Terraform to install Docker/Java/app code be approved?
**Answer:** No. The responsibilities should be split across Terraform for infra, Ansible for config, and GitHub Actions for orchestration.

---

## Chapter 9 — Dynamic Blocks

### Q74. What is a dynamic block?
**Answer:** A way to generate repeated nested blocks inside a single Terraform resource.

### Q75. Dynamic block vs `for_each`?
**Answer:** `for_each` creates resources; `dynamic` repeats nested blocks inside a resource.

### Q76. When should dynamic blocks be used?
**Answer:** When the provider resource schema includes repeatable nested configuration blocks such as allow rules or attached disks.

---

## Chapter 10 — Terraform Functions

### Q77. Why use `lookup()`?
**Answer:** To select a value from a map with a default fallback.

### Q78. Where should lookup maps be defined?
**Answer:** Usually in `locals.tf` when the mapping is internal module logic.

### Q79. Why use `map(object(...))` with lookup scenarios?
**Answer:** When each key has multiple related properties like machine type, region, zone, and disk size.

### Q80. Why use `merge()`?
**Answer:** To combine common maps with environment-specific or resource-specific overrides.

### Q81. Why use `try()` and `can()`?
**Answer:** To handle missing attributes or invalid expressions safely during evaluation.

---

## Chapter 11 — Lifecycle and Recovery

### Q82. What is `prevent_destroy` useful for?
**Answer:** Protecting critical production resources like databases, state buckets, or DNS zones.

### Q83. What is `create_before_destroy` useful for?
**Answer:** Reducing downtime during replacement where the resource can temporarily coexist.

### Q84. What does `ignore_changes` do in practice?
**Answer:** Terraform refreshes state, but it ignores the listed attributes during change planning.

### Q85. Does `ignore_changes` make sense for a VM name in GCE?
**Answer:** Practically no, because the name is an identity attribute and cannot drift by renaming in place.

### Q86. Does `ignore_changes` stop state refresh?
**Answer:** No. It only suppresses planning for the specified attributes.

---

## Chapter 12 — Commands and Flags

### Q87. What does `-parallelism` do?
**Answer:** Limits or increases the number of operations Terraform runs concurrently.

### Q88. What does `-auto-approve` do?
**Answer:** Skips the interactive approval prompt, commonly used in CI/CD.

### Q89. What does `-refresh-only` do?
**Answer:** Refreshes state without proposing config changes.

### Q90. What does `-replace` do?
**Answer:** Forces recreation of a resource for the current apply.

### Q91. What does `-var-file` do?
**Answer:** Passes a variable file for a specific execution.

### Q92. What does `-var` do?
**Answer:** Passes a single variable override at runtime.

### Q93. What does `terraform state list` help with?
**Answer:** Finding exact resource addresses before state manipulation.

### Q94. What does `terraform state show` help with?
**Answer:** Inspecting the state attributes of a managed resource during troubleshooting.

### Q95. What does `terraform state mv` help with?
**Answer:** Refactoring Terraform code without recreating infrastructure.

### Q96. What does `terraform state rm` help with?
**Answer:** Transferring ownership or letting Terraform forget a resource without deleting it.

### Q97. What does `terraform import` help with?
**Answer:** Bringing existing unmanaged infrastructure under Terraform control.

### Q98. What does `terraform state pull` help with?
**Answer:** Creating a local snapshot of the current remote state for backup or auditing.

---

## Chapter 13 — Enterprise Best Practices

### Q99. Should one Terraform stack manage the same resource as another stack?
**Answer:** No.

### Q100. Should every label be duplicated in every tfvars file?
**Answer:** No; use provider-level `default_labels` for GCP common labels or locals + merge when appropriate.

### Q101. Should workspaces be used for Dev/QA/Prod?
**Answer:** Usually no; use separate environment directories.

### Q102. Should provisioners be the primary way to install software?
**Answer:** No; use Ansible or similar configuration management tools.

### Q103. Should state be edited manually?
**Answer:** No.

### Q104. Should module versions be pinned?
**Answer:** Yes.

### Q105. Should validation live only in CI/CD?
**Answer:** No; module-level validation should exist in Terraform as the first line of defense.

---

## Chapter 14 — Senior Interview Takeaways

### Q106. What makes an A3 Terraform answer stronger than an A2 answer?
**Answer:** It explains ownership, trade-offs, safety, production behavior, and why the chosen approach is better than the alternatives.

### Q107. What should you mention when answering scenario questions?
**Answer:** State, ownership, drift, validation, and rollback/recovery impact.

### Q108. How do you answer architecture questions well?
**Answer:** By describing the interface boundary between modules and teams, not only the syntax.

---

## Chapter 15 — Lookups and Environment Maps

### Q109. Why use `lookup()` with `locals.tf`?
**Answer:** It keeps common environment mappings internal to the module and avoids repetitive logic in tfvars.

### Q110. Why use `map(object(...))` over multiple maps?
**Answer:** It keeps related values together and avoids synchronization bugs.

### Q111. Why use a map of objects for environments?
**Answer:** Because an environment often contains multiple related values such as machine type, disk size, region, and zone.

---

## Chapter 16 — Looking Ahead

### Q112. What functions are coming next?
**Answer:** `merge()`, `try()`, `can()`, `coalesce()`, `flatten()`, `zipmap()`, and more expressions for enterprise Terraform.

### Q113. What is the next big interview topic after functions?
**Answer:** Enterprise repository architecture and CI/CD integration.

---

## Short Memory Lines
- `lookup()` → map value with fallback.
- `map(object(...))` → rich, typed config.
- `default_labels` → common GCP labels.
- `merge()` → combine maps.
- `for_each` → stable keys.
- `count` → quantity.
- `dynamic` → nested blocks.
- `ignore_changes` → ignore selected drift.
- `prevent_destroy` → protect prod resources.
- `terraform_remote_state` → consume another stack's outputs.

---

## Study Notes for Tomorrow
- Continue from `merge()`.
- Then `try()`, `can()`, `coalesce()`.
- Then expressions and collection functions.
- Then enterprise repository patterns and CI/CD.
