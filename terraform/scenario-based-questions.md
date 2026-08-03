# 🎯 Terraform Scenario-Based Questions

> Terraform interview practice workbook.
> 
> This file focuses on production scenarios, architecture decisions, trade-offs, and follow-up questions.

---

## Chapter 1 — Foundations & Ownership

### 🟢 Scenario 1
A team has been creating VMs manually in the cloud console and now wants consistent, repeatable infrastructure. How would you explain Terraform to them?

**Expected Senior Answer**
Terraform is declarative IaC. The team defines the desired end state, and Terraform uses providers and a dependency graph to create and manage infrastructure safely and repeatably.

**Why this is correct**
It explains Terraform as a workflow and a control system, not just a tool name.

**Common Wrong Answer**
Terraform is just a scripting tool to create cloud resources.

**Follow-up Question**
How would you handle the same requirement if different teams own different parts of the infrastructure?

---

### 🟡 Scenario 2
A project has one networking team and multiple application teams. Why is that an important Terraform design signal?

**Expected Senior Answer**
It means resource ownership matters. Networking should own the shared network resources, and app teams should consume outputs or data sources rather than recreating the same infrastructure.

**Why this is correct**
It shows ownership boundaries and avoids duplicate resource management.

**Common Wrong Answer**
Each team can create the same VPC in their own repo as long as the names are different.

**Follow-up Question**
What would you use if the networking team already exposes outputs from another Terraform stack?

---

### 🔴 Scenario 3
The same infrastructure must be deployed across Dev, QA, Stage, and Prod, but with different values. What design pattern would you propose?

**Expected Senior Answer**
I would propose a reusable module-driven design with environment-specific values passed through variables or environment folders, keeping the infrastructure logic shared and the environment differences isolated.

**Why this is correct**
It keeps infrastructure logic reusable and environment differences explicit.

**Common Wrong Answer**
Make four copies of the same Terraform code and edit each one manually.

**Follow-up Question**
When would you choose workspaces instead of separate environment folders?

---

### ⚫ Architecture Discussion
A project is suffering from drift because engineers keep modifying resources in the cloud console. What would you change in the Terraform workflow?

**Expected Senior Answer**
I would enforce remote state, locking, code review, and a clear policy that all changes must go through Terraform and Git, so drift is detected and handled deliberately.

**Why this is correct**
It addresses governance, collaboration, and safe change management.

**Common Wrong Answer**
Tell people not to make mistakes.

**Follow-up Question**
How would you detect and manage manual console changes to labels or firewall rules?

---

## Chapter 2 — State & State Command Scenarios

### 🟢 Scenario 1
A VM was deleted manually in the cloud console, but it still appears in Terraform state. What will Terraform show on the next plan?

**Expected Senior Answer**
Terraform will refresh state, detect the missing resource, and likely propose recreating it if the configuration still declares it.

**Why this is correct**
State refresh compares real infrastructure with declared configuration.

**Common Wrong Answer**
Terraform will not notice because the state file still has it.

**Follow-up Question**
What changes if the deletion happened outside Terraform but the resource was still needed?

---

### 🟡 Scenario 2
You renamed a resource in Terraform code from `web` to `app`, but Terraform now wants to destroy and recreate the same VM. How would you avoid that?

**Expected Senior Answer**
I would use `terraform state mv` to move the state address to the new Terraform address before applying the renamed configuration.

**Why this is correct**
It preserves the real resource while updating Terraform’s address mapping.

**Common Wrong Answer**
Delete the VM manually and recreate it with the new name.

**Follow-up Question**
When is `state mv` safer than `import`?

---

### 🔴 Scenario 3
You want Terraform to stop managing a resource, but you do not want to delete the actual infrastructure. What would you do?

**Expected Senior Answer**
Use `terraform state rm` to remove ownership from state, then update configuration or transfer ownership appropriately.

**Why this is correct**
It removes ownership without destroying the cloud object.

**Common Wrong Answer**
Remove the resource block and run apply.

**Follow-up Question**
What is the difference between `state rm` and `destroy`?

---

### ⚫ Architecture Discussion
A production resource exists in GCP, but Terraform does not manage it yet. How do you bring it under Terraform safely?

**Expected Senior Answer**
Write the Terraform configuration first, then use `terraform import` to map the existing infrastructure into state, and verify with plan.

**Why this is correct**
It avoids duplicate creation and validates the adoption path.

**Common Wrong Answer**
Just write the resource block and run `apply`.

**Follow-up Question**
What if another team already owns that resource in a different Terraform state?

---

## Chapter 3 — Variables, Locals, Outputs

### 🟢 Scenario 1
A module is being used in Dev, QA, and Prod, but only the environment name and machine type should differ. Where should those values come from?

**Expected Senior Answer**
The differing values should come from variables supplied by the root environment configuration, while module internals should remain in locals.

**Why this is correct**
It keeps inputs external and implementation internal.

**Common Wrong Answer**
Hardcode environment names inside the module.

**Follow-up Question**
Which values belong in `locals.tf` and which belong in `variables.tf`?

---

### 🟡 Scenario 2
You see the same label map repeated in every environment tfvars file. How would you redesign it?

**Expected Senior Answer**
I’d move common labels to provider `default_labels` for GCP or to root/environment locals, and use variables or `merge()` only for environment-specific overrides.

**Why this is correct**
It removes duplication and centralizes common values.

**Common Wrong Answer**
Copy the labels into every tfvars file forever.

**Follow-up Question**
When is `merge()` a better choice than `default_labels`?

---

### 🔴 Scenario 3
A reviewer asks why a machine type mapping belongs in `locals.tf` instead of `variables.tf`. What would you say?

**Expected Senior Answer**
The mapping is internal implementation detail, not a caller input. Locals keep the module interface smaller and avoid forcing every environment to pass the same map.

**Why this is correct**
It preserves module encapsulation and reduces noise.

**Common Wrong Answer**
Use variables for everything because variables are flexible.

**Follow-up Question**
What if the caller really needs to override the mapping per deployment?

---

### ⚫ Architecture Discussion
A VM module needs disk size, region, zone, and machine type per environment. How would you model the input?

**Expected Senior Answer**
I would use `map(object({...}))` so each environment key owns all its related settings in one structure.

**Why this is correct**
It keeps related properties together and avoids parallel map drift.

**Common Wrong Answer**
Create separate maps for each property.

**Follow-up Question**
Why is a map of objects easier to extend over time?

---

## Chapter 4 — State Commands in Production

### 🟢 Scenario 1
You changed a Terraform resource name in code, and plan wants destroy/create. How do you preserve the existing resource?

**Expected Senior Answer**
Move the state address with `terraform state mv` so Terraform sees the same real resource under the new name.

**Why this is correct**
It preserves the cloud resource and aligns Terraform’s address.

**Common Wrong Answer**
Accept the replacement because the code changed.

**Follow-up Question**
When would you instead use `import`?

---

### 🟡 Scenario 2
The application team no longer wants Terraform to manage a VM, but the VM must stay alive. What would you do?

**Expected Senior Answer**
Use `terraform state rm` and then hand over ownership or manage it manually elsewhere.

**Why this is correct**
It lets Terraform forget the object without deleting it.

**Common Wrong Answer**
Delete the VM from code and hope it stays alive.

**Follow-up Question**
What happens if you remove the code without removing the state entry?

---

### 🔴 Scenario 3
A manually created resource should now be managed by Terraform. What is the sequence?

**Expected Senior Answer**
Add the Terraform configuration first, then run `terraform import` using the real infrastructure ID.

**Why this is correct**
Import needs a matching configuration address.

**Common Wrong Answer**
Run import first and write code later.

**Follow-up Question**
What should you check immediately after import?

---

### ⚫ Architecture Discussion
A state change is risky and you want a quick snapshot before modifying ownership. What command helps?

**Expected Senior Answer**
`terraform state pull > backup.tfstate`, because it creates a local snapshot of the remote backend state.

**Why this is correct**
It gives you a recovery copy before performing state surgery.

**Common Wrong Answer**
Assume backend versioning alone is enough for every risky operation.

**Follow-up Question**
What would you do if the backend itself was lost?

---

## Chapter 5 — Remote State vs Data Source

### 🟢 Scenario 1
The networking team owns the VPC in another Terraform repo. The app team needs the VPC ID. What should they use?

**Expected Senior Answer**
`terraform_remote_state`, because the app team should consume the networking team’s published outputs rather than infer the network by name.

**Why this is correct**
It respects ownership and uses the owner’s contract.

**Common Wrong Answer**
Create another VPC in the app repo.

**Follow-up Question**
What if the VPC already exists outside Terraform?

---

### 🟡 Scenario 2
A resource exists in GCP but was not created by Terraform at all. How should Terraform consume it?

**Expected Senior Answer**
Use a data source, because there is no Terraform owner or published state contract to consume.

**Why this is correct**
Data sources are for read-only lookup of externally managed resources.

**Common Wrong Answer**
Use `terraform_remote_state` for everything that already exists.

**Follow-up Question**
What if the resource is owned by a different Terraform stack?

---

### 🔴 Scenario 3
An engineer suggests using a data source to read a VPC owned by another Terraform repo. What would you say?

**Expected Senior Answer**
I would prefer remote state when another Terraform stack owns the resource, because it uses the owner’s outputs as the contract and avoids relying on internal resource naming.

**Why this is correct**
It decouples consumers from implementation details.

**Common Wrong Answer**
Use the cloud API lookup by name because it is simpler.

**Follow-up Question**
What risks appear if the networking team renames the VPC internally?

---

### ⚫ Architecture Discussion
Why is `terraform_remote_state` considered an ownership boundary in enterprise design?

**Expected Senior Answer**
Because the consuming team uses the owner’s outputs, not the owner’s internal implementation details.

**Why this is correct**
It preserves encapsulation and team autonomy.

**Common Wrong Answer**
Because remote state is faster than data sources.

**Follow-up Question**
How should outputs be designed to remain stable across internal refactors?

---

## Chapter 6 — Meta Arguments

### 🟢 Scenario 1
You need 10 identical VMs for a lab. Would you choose `count`, `for_each`, or `dynamic`?

**Expected Senior Answer**
`count`, because the resources are homogeneous and only quantity matters.

**Why this is correct**
Count is ideal for identical instances.

**Common Wrong Answer**
Use `dynamic` because you need multiple VMs.

**Follow-up Question**
What changes if each VM has a unique role?

---

### 🟡 Scenario 2
You need Frontend, Backend, and Database VMs, each with different machine types. Would you choose `count` or `for_each`?

**Expected Senior Answer**
`for_each`, because each VM has a stable identity and different configuration.

**Why this is correct**
It avoids index shifting and keeps identities stable.

**Common Wrong Answer**
Use `count` with a list of names.

**Follow-up Question**
What happens if Backend is removed later?

---

### 🔴 Scenario 3
A firewall rule has multiple repeated `allow {}` blocks. Would you use `for_each`?

**Expected Senior Answer**
No. I’d use a `dynamic` block because the repetition is inside one resource, not across multiple resources.

**Why this is correct**
`dynamic` is for nested blocks.

**Common Wrong Answer**
`for_each` can be used anywhere repetition exists.

**Follow-up Question**
What is the difference between repeating resources and repeating nested blocks?

---

### ⚫ Architecture Discussion
A VM depends on a subnet that depends on a VPC. Do you need `depends_on` on the VM for the VPC?

**Expected Senior Answer**
No. Terraform already infers the implicit and transitive dependency chain.

**Why this is correct**
References already build the graph.

**Common Wrong Answer**
Add `depends_on` everywhere to be safe.

**Follow-up Question**
When is an explicit dependency actually required?

---

## Chapter 7 — Lifecycle & ForceNew

### 🟢 Scenario 1
A production database must never be deleted accidentally. Which lifecycle setting would you use?

**Expected Senior Answer**
`prevent_destroy`.

**Why this is correct**
It blocks accidental deletion of critical resources.

**Common Wrong Answer**
Use `ignore_changes` on the whole resource.

**Follow-up Question**
How do you intentionally delete a protected resource later?

---

### 🟡 Scenario 2
You want a replacement VM to be created before the old one is removed. Which lifecycle setting would you use?

**Expected Senior Answer**
`create_before_destroy`, if the resource can coexist temporarily.

**Why this is correct**
It reduces downtime during replacement.

**Common Wrong Answer**
Always use `create_before_destroy` on every resource.

**Follow-up Question**
Why can this fail for some uniquely named resources?

---

### 🔴 Scenario 3
A security team manages labels manually, and Terraform should not constantly revert them. Which lifecycle setting fits?

**Expected Senior Answer**
`ignore_changes` for the labels attribute.

**Why this is correct**
It suppresses drift correction for intentionally external-managed attributes.

**Common Wrong Answer**
Remove labels from the resource entirely.

**Follow-up Question**
Would you use `ignore_changes` for VM name as well?

---

### ⚫ Architecture Discussion
A VM name change in Terraform causes replacement. Why does that happen?

**Expected Senior Answer**
Because the Google provider marks the name as ForceNew; GCE treats the name as identity, so a different name means a different resource.

**Why this is correct**
It ties Terraform behavior to cloud provider identity semantics.

**Common Wrong Answer**
Terraform is buggy and always recreates resources when names change.

**Follow-up Question**
Can `ignore_changes` override a deliberately changed ForceNew attribute?

---

## Chapter 8 — Workspaces

### 🟢 Scenario 1
The team wants Dev, QA, and Prod to use workspaces because it sounds simpler. Would you recommend it?

**Expected Senior Answer**
No. I’d recommend separate environment directories because these environments typically need different providers, backends, IAM, and configurations.

**Why this is correct**
Workspaces isolate state, not configuration.

**Common Wrong Answer**
Yes, because workspaces are always the standard way to handle environments.

**Follow-up Question**
When are workspaces actually useful?

---

### 🟡 Scenario 2
A workspace-created bucket uses the same hardcoded name in every workspace. What happens?

**Expected Senior Answer**
The second workspace will fail when trying to create a bucket that already exists, because the configuration is identical and the cloud name is not workspace-aware.

**Why this is correct**
Only state changed, not the resource name.

**Common Wrong Answer**
Workspaces automatically make all names unique.

**Follow-up Question**
How could `terraform.workspace` be used to make the names unique technically?

---

### 🔴 Scenario 3
How could you make workspace-specific names work technically?

**Expected Senior Answer**
By including `terraform.workspace` in the name, such as `bucket-${terraform.workspace}`, but that still does not make workspaces the best choice for long-lived environments.

**Why this is correct**
It shows the technical workaround and its limitation.

**Common Wrong Answer**
If names use `terraform.workspace`, then workspaces are always the best solution.

**Follow-up Question**
Why do enterprises still prefer environment directories?

---

### ⚫ Architecture Discussion
When are workspaces actually useful in production-like workflows?

**Expected Senior Answer**
For identical infrastructure templates used in multiple isolated instances, temporary sandboxes, or ephemeral environments.

**Why this is correct**
It matches the original design intent of workspaces.

**Common Wrong Answer**
For all Dev/QA/Prod separation.

**Follow-up Question**
What problems appear when environment-specific logic spreads across `terraform.workspace` conditionals?

---

## Chapter 9 — Provisioners

### 🟢 Scenario 1
A junior engineer wants Terraform to create a VM and install Docker, Java, and the application using `remote-exec`. Would you approve the design?

**Expected Senior Answer**
No. Terraform should provision the VM, while Ansible should configure the software and GitHub Actions should orchestrate the workflow.

**Why this is correct**
It separates infrastructure from configuration management and orchestration.

**Common Wrong Answer**
Yes, because fewer tools means a simpler pipeline.

**Follow-up Question**
What is the risk if the provisioner fails halfway?

---

### 🟡 Scenario 2
A provisioner fails after the VM is already created. Why is that a problem?

**Expected Senior Answer**
You end up with partial infrastructure and a broken configuration step, which is much harder to reason about than separate provisioning and configuration stages.

**Why this is correct**
Provisioners are brittle compared to purpose-built config tools.

**Common Wrong Answer**
That is fine because Terraform will retry everything automatically.

**Follow-up Question**
What tool would you use instead to install and configure software?

---

### 🔴 Scenario 3
What would you say if asked why Terraform even has provisioners?

**Expected Senior Answer**
They exist for rare bootstrapping or integration edge cases, but they should be treated as a last resort rather than the normal design pattern.

**Why this is correct**
It reflects how enterprises use Terraform in practice.

**Common Wrong Answer**
Provisioners are the main way to configure servers.

**Follow-up Question**
What should GitHub Actions do in the flow instead?

---

## Chapter 10 — Module Sources & Versioning

### 🟢 Scenario 1
A team wants to reuse a VPC module across many projects. Which source approach would you prefer?

**Expected Senior Answer**
A Git source or private registry with version pinning, because it keeps the module reusable and reproducible.

**Why this is correct**
It makes module reuse controlled and repeatable.

**Common Wrong Answer**
Copy the module into each repo.

**Follow-up Question**
Why does `?ref=` matter in the Git source?

---

### 🟡 Scenario 2
Why is `?ref=v1.2.0` important in a Git module source?

**Expected Senior Answer**
It locks the module to a tested version so a new commit on main does not unexpectedly change live infrastructure behavior.

**Why this is correct**
It avoids unintended module drift.

**Common Wrong Answer**
Leaving out `ref` is better because it always uses the latest fixes.

**Follow-up Question**
What would happen if a breaking change is merged into main?

---

### 🔴 Scenario 3
A module is sourced from the Terraform Registry. Why is that useful in an enterprise context?

**Expected Senior Answer**
It gives you a standardized, reusable, and versioned module implementation that can reduce custom code.

**Why this is correct**
Registry modules are curated and easier to consume.

**Common Wrong Answer**
Registry modules are only for learning, not production.

**Follow-up Question**
When would you still prefer a private registry or Git module?

---

## Chapter 11 — Labels, `default_labels`, and `merge()`

### 🟢 Scenario 1
A GCP team has 15 labels repeated in every tfvars file. What would you do?

**Expected Senior Answer**
I’d put organization-wide labels in provider `default_labels` and use `merge()` or variables for environment-specific overrides.

**Why this is correct**
It centralizes repeated configuration and reduces duplication.

**Common Wrong Answer**
Keep copying the same label map everywhere.

**Follow-up Question**
Why is provider-level labeling easier to govern?

---

### 🟡 Scenario 2
When would `merge()` be a better choice than `default_labels`?

**Expected Senior Answer**
When the design must be provider-agnostic, or when you need to combine multiple label maps with explicit overrides.

**Why this is correct**
It works across providers and gives explicit precedence.

**Common Wrong Answer**
`merge()` is always better than provider settings.

**Follow-up Question**
Can both approaches coexist in the same codebase?

---

### 🔴 Scenario 3
Why do enterprise teams like `default_labels` for GCP?

**Expected Senior Answer**
Because it centralizes common labels and avoids duplication across every resource definition.

**Why this is correct**
It keeps GCP-wide conventions in one place.

**Common Wrong Answer**
Because labels cannot be managed any other way.

**Follow-up Question**
How do labels differ from tags in GCP design discussions?

---

## Chapter 12 — Functions and Expressions

### 🟢 Scenario 1
You have Dev, QA, and Prod machine type mappings. Would you use nested ternaries or `lookup()`?

**Expected Senior Answer**
`lookup()` with a map, because it is cleaner and easier to maintain as the environment list grows.

**Why this is correct**
It avoids unreadable condition chains.

**Common Wrong Answer**
Use nested ternaries because they are quicker.

**Follow-up Question**
Where should that mapping live?

---

### 🟡 Scenario 2
A single environment has machine type, disk size, region, and zone. How would you model it?

**Expected Senior Answer**
As a `map(object(...))`, because related properties should stay together.

**Why this is correct**
It keeps the environment contract structured.

**Common Wrong Answer**
Use four separate maps.

**Follow-up Question**
What happens when a new field is added later?

---

### 🔴 Scenario 3
A map is missing a key in an environment lookup. What should happen?

**Expected Senior Answer**
`lookup()` should return a safe default, so the module remains robust even if a key is absent.

**Why this is correct**
It prevents hard failures for optional or fallback behavior.

**Common Wrong Answer**
Let Terraform crash because missing keys are always fatal.

**Follow-up Question**
What is the risk of choosing the wrong default value?

---

### ⚫ Architecture Discussion
Why define environment maps in `locals.tf` rather than tfvars?

**Expected Senior Answer**
Because the mapping is internal module logic and should not be redefined by every consumer.

**Why this is correct**
It preserves module ownership and reduces duplicate configuration.

**Common Wrong Answer**
Put everything in tfvars because that is where values belong.

**Follow-up Question**
When would a caller-controlled override be appropriate?

---

## Chapter 13 — Real Production Decisions

### 🟢 Scenario 1
A VPC already exists and the app team needs its ID. What decision do you make: resource, data source, or remote state?

**Expected Senior Answer**
If another Terraform stack owns it, use `terraform_remote_state`; if it is unmanaged or cloud-created, use a data source.

**Why this is correct**
It matches ownership to the correct consumption method.

**Common Wrong Answer**
Always use data sources because they are simpler.

**Follow-up Question**
What if the networking team changes internal implementation later?

---

### 🟡 Scenario 2
A database must never be deleted accidentally. How do you protect it in Terraform?

**Expected Senior Answer**
I use `prevent_destroy` and I keep the resource under strong change control.

**Why this is correct**
It adds a guardrail against accidental destruction.

**Common Wrong Answer**
Rely on the developer to never run destroy.

**Follow-up Question**
What is the recovery path if the resource must be intentionally removed?

---

### 🔴 Scenario 3
A team wants to change the same state file from two terminals. What should happen?

**Expected Senior Answer**
State locking should prevent concurrent modification; only one operation should proceed at a time.

**Why this is correct**
It prevents race conditions and corrupted state.

**Common Wrong Answer**
Both applies can run and Terraform will merge them.

**Follow-up Question**
What should you check when a lock appears to be stale?

---

### ⚫ Architecture Discussion
A manual console change occurs on labels only. How would you keep Terraform calm but informed?

**Expected Senior Answer**
Use `ignore_changes` for the label attribute if those labels are intentionally managed elsewhere.

**Why this is correct**
It tolerates approved external management without fighting every plan.

**Common Wrong Answer**
Ignore the resource forever.

**Follow-up Question**
Would you use `ignore_changes` for VM name as well?

---

## Chapter 14 — Short Recall and Rapid Fire

### 🟢 Scenario 1
In one line, when do you use `count`?

**Expected Senior Answer**
For multiple identical resources.

**Why this is correct**
It’s quantity-driven.

**Common Wrong Answer**
For any repeated configuration.

**Follow-up Question**
What if the instances have different identities?

---

### 🟢 Scenario 2
In one line, when do you use `for_each`?

**Expected Senior Answer**
For multiple uniquely identified resources.

**Why this is correct**
It’s identity-driven.

**Common Wrong Answer**
Whenever there is a list.

**Follow-up Question**
What happens if one key is removed?

---

### 🟢 Scenario 3
In one line, when do you use `dynamic`?

**Expected Senior Answer**
For repeated nested blocks inside a resource.

**Why this is correct**
It repeats provider-defined blocks, not resources.

**Common Wrong Answer**
For creating multiple VMs.

**Follow-up Question**
What is the difference between nested blocks and top-level resources?

---

### 🟡 Lightning Round
- Why not `depends_on` everywhere?
- Why not workspaces for Dev/QA/Prod?
- Why not install Docker with provisioners?
- Why not use `ignore_changes` on identity attributes?
- Why not use parallel maps for environment config?

**Expected Senior Answer**
Because each of those choices harms maintainability, safety, or correctness in production.

---

## Chapter 15 — Closing Scenarios

### 🟢 Scenario 1
The networking team changes internal implementation but keeps the same outputs. What happens to application teams?

**Expected Senior Answer**
Nothing should break if they consume the published outputs contract.

**Why this is correct**
The interface remains stable even if internals change.

**Common Wrong Answer**
Any internal refactor automatically breaks every consumer.

**Follow-up Question**
How should outputs be designed to remain stable over time?

---

### 🟡 Scenario 2
A Terraform plan shows `-/+`. What is your first response?

**Expected Senior Answer**
Check whether the change is expected, whether the attribute is ForceNew, and whether downtime risk is acceptable.

**Why this is correct**
Replacement is the highest-risk plan outcome.

**Common Wrong Answer**
Click apply first and investigate later.

**Follow-up Question**
Which attributes on GCE resources often trigger replacement?

---

### 🔴 Scenario 3
A module needs a human-readable and a machine-readable environment representation. What would you use?

**Expected Senior Answer**
Strongly typed variables plus a structured map/object-based config, keeping the interface deterministic.

**Why this is correct**
It preserves readability and strictness.

**Common Wrong Answer**
Use free-form strings for everything.

**Follow-up Question**
What validation would you add to protect that interface?

---

## Chapter 16 — Interview Pattern Reminder

### ⚫ Scenario
Should this workbook prioritize scenario questions over definitions?

**Expected Senior Answer**
Yes. Definitions belong in the handbook and quick revision; this file should train interview thinking.

**Why this is correct**
It matches how senior interviews are usually conducted.

**Common Wrong Answer**
A workbook should repeat definitions from the handbook.

**Follow-up Question**
What type of question do you want more of: architecture, recovery, or trade-off?

---

## Final Note
This workbook intentionally focuses on scenarios, trade-offs, and production decisions because that is how senior Terraform interviews are typically asked.
