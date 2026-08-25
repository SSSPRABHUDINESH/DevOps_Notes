Absolutely. 🚀 Let's start **Ansible from Topic 1**, but this time we'll go much deeper because **Ansible is your core skill** and we want you to be able to handle production and LevelUp interview questions confidently.

# 🚀 Ansible Topic 1 — What is Ansible & Its Architecture

We'll build the foundation first. Don't worry about commands like `remote_src`, `serial`, `delegate_to`, etc. yet—we'll reach them systematically.

---

## 1. 🧠 What is Ansible?

**Ansible is an open-source automation and configuration-management tool used to automate tasks across servers and infrastructure.**

It can automate:

* 🖥️ OS configuration
* 📦 Package installation
* ⚙️ Service management
* 👤 User management
* 📁 File management
* 🔐 Security configuration
* ☁️ Cloud provisioning/configuration
* 🐳 Container/Kubernetes operations
* 🗄️ Database configuration
* 🚀 Application deployment

The key idea is:

> **You describe the desired state or sequence of operations in YAML, and Ansible executes those operations on target systems.**

Example:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

You don't tell Ansible:

```text
Run rpm command
Check whether nginx exists
If it doesn't exist, install it
If it exists, do nothing
```

Instead, you say:

```text
I want nginx to be present.
```

The module determines what action is required.

---

# 2. 🏗️ Ansible Architecture

The simplest architecture is:

```text
                 👨‍💻 Ansible Controller
                         │
                         │
                    ansible-playbook
                         │
              ┌──────────┼──────────┐
              │          │          │
             SSH        SSH        SSH
              │          │          │
              ▼          ▼          ▼
          Server 1    Server 2    Server 3
          Managed     Managed     Managed
           Node        Node        Node
```

### Important terminology

| Component        | Meaning                                                       |
| ---------------- | ------------------------------------------------------------- |
| **Control Node** | Machine from which Ansible runs                               |
| **Managed Node** | Server/device being managed                                   |
| **Inventory**    | List/grouping of managed nodes                                |
| **Playbook**     | YAML automation definition                                    |
| **Play**         | Set of tasks applied to selected hosts                        |
| **Task**         | Individual unit of work                                       |
| **Module**       | Code that performs the actual operation                       |
| **Collection**   | Distribution package containing modules, plugins, roles, etc. |
| **Role**         | Reusable structure for organizing automation                  |

---

# 3. 🎛️ Control Node

The **Control Node** is where Ansible is installed and executed.

For example:

```text
                    Control Node
                   192.168.1.10
                         │
                Ansible installed
                         │
                ansible-playbook
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
         server01     server02     server03
```

It could be:

* Your laptop
* Linux VM
* CI/CD runner
* AWX/Automation Controller environment
* Bastion/jump host

### Important

The control node generally **does not need an Ansible agent running on every managed server**.

That's one of Ansible's major architectural characteristics.

---

# 4. 🖥️ Managed Node

A **managed node** is the system Ansible controls.

Examples:

```text
RHEL server
Ubuntu server
Windows server
Network device
Cloud resource
Database
Kubernetes API
```

For Linux/Unix systems, Ansible commonly connects through:

```text
SSH
```

For Windows:

```text
WinRM / PSRP
```

There are also other connection mechanisms depending on the target and collection.

---

# 5. 🆚 Agent vs Agentless

This is a classic interview question.

### Traditional agent-based model

```text
Controller
    │
    ▼
Agent installed on server
    │
    ▼
Server
```

The server needs an agent.

### Ansible's common model

```text
Controller
    │
    │ SSH
    ▼
Server
```

No continuously running Ansible agent is required.

Therefore:

> **Ansible is commonly described as agentless for its standard SSH-based Linux/Unix automation model.**

⚠️ Don't say:

> "Ansible never uses agents."

That's too absolute.

The important point is that Ansible's normal architecture does **not require a persistent Ansible agent on managed Linux nodes**.

---

# 6. 🔑 How Does Ansible Connect?

For Linux:

```text
Ansible Controller
       │
       │ SSH
       ▼
Managed Node
```

Authentication can use:

### SSH key

```text
~/.ssh/id_rsa
```

or:

```text
~/.ssh/id_ed25519
```

### Password

Possible, although key-based authentication is generally preferred for automation.

### Become

Suppose you connect as:

```text
dinesh
```

but need to install a package as:

```text
root
```

Ansible can use privilege escalation:

```yaml
become: true
```

Flow:

```text
Ansible
   │
   │ SSH
   ▼
dinesh
   │
   │ sudo
   ▼
root
```

We'll study `become` deeply later.

---

# 7. 📦 What Actually Happens When You Run a Playbook?

This is **very important**.

Suppose you run:

```bash
ansible-playbook install-nginx.yml
```

Ansible roughly goes through:

```text
                 Playbook
                    │
                    ▼
                Inventory
                    │
                    ▼
             Select target hosts
                    │
                    ▼
                 Gather facts
                    │
                    ▼
                 Execute tasks
                    │
                    ▼
              Call modules
                    │
                    ▼
          Managed Node performs work
                    │
                    ▼
          Result returned to Ansible
                    │
                    ▼
             changed/ok/failed
```

---

# 8. 🧩 What is a Module?

A **module** performs an actual operation.

Examples:

```yaml
ansible.builtin.package
ansible.builtin.service
ansible.builtin.copy
ansible.builtin.template
ansible.builtin.file
ansible.builtin.user
ansible.builtin.command
ansible.builtin.shell
```

Example:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

Here:

```text
ansible.builtin.package
        │
        ▼
      Module
        │
        ▼
Installs/manages packages
```

---

# 9. 🧱 Task

A **task** is a single unit of work in a play.

Example:

```yaml
tasks:

  - name: Install nginx
    ansible.builtin.package:
      name: nginx
      state: present

  - name: Start nginx
    ansible.builtin.service:
      name: nginx
      state: started
```

There are two tasks:

```text
Task 1 → Install nginx
Task 2 → Start nginx
```

---

# 10. 🎭 Play

A **play** connects:

```text
Hosts
+
Tasks
```

Example:

```yaml
- name: Configure web servers
  hosts: webservers

  tasks:

    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Here:

```yaml
hosts: webservers
```

selects the target machines.

And:

```yaml
tasks:
```

defines what should happen.

---

# 11. 📖 Playbook

A **playbook** is a YAML file containing one or more plays.

Example:

```yaml
---
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present

- name: Configure database servers
  hosts: databases

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.package:
        name: postgresql
        state: present
```

One playbook:

```text
site.yml
 │
 ├── Play 1
 │     └── webservers
 │
 └── Play 2
       └── databases
```

---

# 12. 📋 Inventory

Inventory tells Ansible:

> **Which machines do I manage?**

Example:

```ini
[webservers]
web01 ansible_host=10.10.1.10
web02 ansible_host=10.10.1.11

[databases]
db01 ansible_host=10.10.2.10
db02 ansible_host=10.10.2.11
```

Visual:

```text
Inventory
│
├── webservers
│   ├── web01
│   └── web02
│
└── databases
    ├── db01
    └── db02
```

Then:

```yaml
hosts: webservers
```

means:

```text
web01
web02
```

---

# 13. 🔄 Complete Execution Flow

Let's put everything together.

```text
                    👨‍💻 Engineer
                         │
                         │
                         ▼
                  site.yml
                  Playbook
                         │
                         ▼
                  Ansible CLI
                         │
                         ▼
                    Inventory
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
            web01      web02      web03
              │          │          │
             SSH        SSH        SSH
              │          │          │
              ▼          ▼          ▼
           Modules     Modules     Modules
              │          │          │
              ▼          ▼          ▼
            Result     Result     Result
              │          │          │
              └──────────┼──────────┘
                         ▼
                    Ansible output
```

---

# 14. 🧠 Declarative vs Imperative

This is especially relevant because we discussed this earlier for Terraform/Ansible.

### Imperative

You specify **how**:

```bash
systemctl start nginx
```

### Declarative

You specify **what state you want**:

```yaml
state: started
```

Example:

```yaml
- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
```

You're saying:

> Nginx should be running.

Not:

> Execute `systemctl start nginx`.

---

# 15. ♻️ Idempotency

This is one of the **most important Ansible concepts for you**.

An operation is idempotent if repeatedly applying it results in the same desired state without unnecessary changes.

Example:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

First execution:

```text
nginx absent
     │
     ▼
install
     │
     ▼
changed
```

Second execution:

```text
nginx already present
     │
     ▼
no action required
     │
     ▼
ok
```

Output conceptually:

```text
First run:
changed=1

Second run:
changed=0
```

### ⭐ Interview answer

> **Ansible modules are generally designed to be idempotent by managing desired state, although not every module/action or arbitrary command is inherently idempotent. When using command/shell or custom logic, the engineer must design the task carefully to preserve idempotency.**

This distinction is extremely important.

---

# 16. ⚠️ `command` vs `shell`

We'll study these deeply later, but understand the fundamental difference now.

### command

```yaml
- name: Run command
  ansible.builtin.command:
    cmd: systemctl status nginx
```

Doesn't invoke a shell by default.

### shell

```yaml
- name: Run shell command
  ansible.builtin.shell:
    cmd: systemctl status nginx | grep Active
```

Runs through a shell and therefore supports shell features such as:

```text
pipes
redirection
shell expansion
```

But:

> Prefer dedicated Ansible modules over `command`/`shell` whenever a suitable module exists.

---

# 17. 🎯 Why Modules Matter

Bad:

```yaml
- name: Install nginx
  ansible.builtin.shell:
    cmd: yum install -y nginx
```

Better:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

Why?

The module understands desired state.

```text
shell
 │
 └── "run this command"

package
 │
 └── "ensure this package state"
```

This helps with:

* idempotency
* portability
* predictable results
* check mode support
* structured output

---

# 18. 🧩 Ansible Collections

Modern Ansible functionality is distributed through **collections**.

A collection can contain:

```text
modules
plugins
roles
playbooks
module_utils
```

Example:

```yaml
ansible.builtin.copy
```

Here:

```text
ansible
   │
   └── builtin
          │
          └── copy
```

The first two components are the **collection namespace/name**, followed by the plugin/module name.

Third-party examples may look like:

```yaml
community.general.some_module
```

or:

```yaml
ansible.posix.mount
```

Using the **FQCN** is a production best practice.

---

# 19. ⭐ Why FQCN?

Instead of:

```yaml
copy:
```

prefer:

```yaml
ansible.builtin.copy:
```

Benefits:

* clear module origin
* avoids name collisions
* easier maintenance
* better tooling/linting
* explicit dependency on the collection

We'll go deep into collections later because this is one of your strongest areas.

---

# 20. 🔥 Production Mental Model

When you see:

```yaml
- name: Deploy application
  hosts: app_servers
  become: true

  tasks:
    - name: Install package
      ansible.builtin.package:
        name: myapp
        state: present
```

you should mentally translate it to:

```text
                    Playbook
                       │
                       ▼
                 Select hosts
                       │
                       ▼
                    SSH
                       │
                       ▼
                    become
                       │
                       ▼
                  package module
                       │
                       ▼
             Desired state:
             package = present
                       │
                ┌──────┴──────┐
                ▼             ▼
              absent        present
                │             │
              install        ok
                │             │
              changed         ok
```

---

# 🎤 LevelUp Interview Questions

### Q1. What is Ansible?

> Ansible is an automation and configuration-management platform used to automate infrastructure and application operations through playbooks, modules, roles, collections and plugins.

### Q2. Is Ansible agentless?

> Ansible's standard Linux/Unix automation model is agentless, meaning a persistent Ansible agent is not required on managed nodes. It commonly uses SSH for remote execution.

### Q3. What is a control node?

> The system where Ansible is installed and from which Ansible commands and playbooks are executed.

### Q4. What is a managed node?

> A system that Ansible manages.

### Q5. What is a playbook?

> A YAML file containing one or more plays that define automation.

### Q6. Play vs task?

> A play maps a set of tasks to a group of hosts. A task is an individual unit of work.

### Q7. What is a module?

> A reusable unit of Ansible functionality that performs a specific operation against a managed system or API.

### Q8. Why use modules instead of shell commands?

> Modules generally provide structured, state-aware, idempotent behavior and better integration with Ansible features such as check mode.

### Q9. Is every Ansible task idempotent?

**No.**

This is a common trap.

For example:

```yaml
- ansible.builtin.shell:
    cmd: echo hello >> /tmp/test
```

Repeated execution keeps modifying the file.

So:

> **Ansible itself supports idempotent automation, but the engineer must use modules or design tasks appropriately to ensure idempotency.**

---

# 🧪 Mini Hands-On

Create:

```text
ansible-learning/
├── inventory
└── site.yml
```

### inventory

```ini
[webservers]
server01
server02
```

### site.yml

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Ensure nginx is installed
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Ensure nginx is running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

Run:

```bash
ansible-playbook -i inventory site.yml
```

The important mental flow is:

```text
inventory
    ↓
webservers
    ↓
server01 + server02
    ↓
package module
    ↓
service module
    ↓
desired state
```

---

# 🏆 Topic 1 — What You Must Remember

```text
Ansible
   │
   ├── Control Node
   │
   ├── Inventory
   │
   ├── Playbook
   │      │
   │      └── Play
   │             │
   │             └── Tasks
   │                    │
   │                    └── Modules
   │
   ├── Roles
   │
   └── Collections
```

And the **five concepts I want you to be able to explain without thinking**:

> **Control Node → Inventory → Playbook → Task → Module**

Then:

> **Desired state → Idempotency → Repeated execution**

---


### Doubt: ansible-playbook -i inventory site.yml here inventory file can have any extension? like inventory.yml?


Yes 👍. **The inventory file can have different filenames/extensions**, as long as Ansible can parse the inventory format.

For example, all of these can be used:

```text
inventory
inventory.ini
hosts
hosts.ini
inventory.yml
inventory.yaml
```

But there is an important distinction between **INI-style inventory** and **YAML inventory**.

### 1. INI-style inventory

For example:

```ini
[webservers]
server01
server02

[databases]
db01
db02
```

You can save it as:

```text
inventory
```

or:

```text
inventory.ini
```

Then:

```bash
ansible-playbook -i inventory.ini site.yml
```

This is very common and simple.

---

### 2. YAML inventory

You can also use YAML:

```yaml
all:
  children:
    webservers:
      hosts:
        server01:
        server02:

    databases:
      hosts:
        db01:
        db02:
```

Save it as:

```text
inventory.yml
```

and:

```bash
ansible-playbook -i inventory.yml site.yml
```

Ansible can recognize the YAML inventory format.

---

### ⚠️ One important point

The `-i` option means:

> **"Use this as my inventory source."**

So:

```bash
ansible-playbook -i inventory.yml site.yml
```

means:

```text
-i inventory.yml
      │
      └── Inventory source

site.yml
      │
      └── Playbook
```

The filename itself isn't what makes it an inventory. **The content/format is what matters.**

---

### 🏭 Production recommendation

You'll commonly see:

```text
ansible/
├── inventories/
│   ├── dev/
│   │   └── hosts.yml
│   ├── staging/
│   │   └── hosts.yml
│   └── prod/
│       └── hosts.yml
│
├── playbooks/
│   └── site.yml
│
└── roles/
```

Then:

```bash
ansible-playbook \
  -i inventories/prod/hosts.yml \
  playbooks/site.yml
```

We'll cover **static INI inventory → YAML inventory → inventory variables → dynamic inventory → inventory plugins → cloud inventories** later.

One more important thing: **`inventory.yml` is not the same thing as `group_vars/*.yml` or `host_vars/*.yml`**. Those have different purposes. We'll get to that when we study inventory structure.



# 🚀 Ansible Topic 2 — Control Node, Managed Node & Connection Methods

Now let's go one level deeper into **how Ansible actually communicates with machines**.

This topic is important because interviewers often ask:

> "How does Ansible connect to a remote server?"

> "What is `ansible_host` vs `ansible_user`?"

> "How does `become` work?"

> "Does Ansible always use SSH?"

> "What are connection plugins?"

Let's build the mental model first.

---

# 1. 🏗️ Ansible Communication Architecture

The basic architecture is:

```text
                 🎛️ CONTROL NODE
                 Ansible installed
                        │
                        │
                 ansible-playbook
                        │
                        ▼
                   INVENTORY
                        │
            ┌───────────┼───────────┐
            │           │           │
            ▼           ▼           ▼
         server01    server02    server03
            │           │           │
           SSH         SSH         SSH
            │           │           │
            ▼           ▼           ▼
        Managed      Managed      Managed
          Node         Node         Node
```

### Remember:

**Control Node**

> Where Ansible runs.

**Managed Node**

> The machine Ansible manages.

---

# 2. 🎛️ Control Node

The control node is where you execute:

```bash
ansible
```

or:

```bash
ansible-playbook
```

For example:

```text
Your Laptop
     │
     └── Ansible
```

or:

```text
CI/CD Runner
     │
     └── Ansible
```

or:

```text
Automation Controller / AWX
     │
     └── Ansible
```

The control node needs:

* Ansible
* Python/runtime required by your Ansible installation
* Network access to managed systems
* Credentials
* Inventory
* Collections/plugins required by the automation

---

# 3. 🖥️ Managed Node

A managed node is the target.

For example:

```text
web01
web02
db01
db02
```

Your inventory might contain:

```ini
[webservers]
web01
web02

[databases]
db01
db02
```

Then:

```text
              Control Node
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
        web01    web02     db01
```

---

# 4. 🔌 How Does Ansible Connect?

For Linux/Unix systems, the most common connection method is:

```text
SSH
```

So:

```text
Ansible Controller
       │
       │ SSH
       ▼
Managed Linux Server
```

For Windows, Ansible commonly uses:

```text
WinRM / PSRP
```

And Ansible supports other connection mechanisms through **connection plugins**.

So don't memorize:

> "Ansible always uses SSH."

Instead say:

> **Ansible commonly uses SSH for Linux/Unix managed nodes, but the connection mechanism is configurable through connection plugins.**

That's a much better interview answer.

---

# 5. 🔑 SSH Connection — Basic Example

Suppose:

```text
Controller:
10.10.1.10

Server:
10.10.1.20
```

You can manually test:

```bash
ssh dinesh@10.10.1.20
```

If this works:

```text
Controller
    │
    │ SSH
    ▼
10.10.1.20
```

Ansible can use the same connectivity.

---

# 6. 📋 Inventory Connection Variables

Now we reach an important concept.

Suppose inventory contains:

```ini
[webservers]
web01 ansible_host=10.10.1.20
```

Here:

```text
web01
```

is the **inventory hostname**.

And:

```text
10.10.1.20
```

is the actual network address.

---

# 7. 🧠 `ansible_host`

`ansible_host` tells Ansible:

> **"What address should I actually connect to?"**

Example:

```ini
[webservers]
web01 ansible_host=10.10.1.20
```

Visualize:

```text
Inventory name
     │
     ▼
   web01
     │
     │ ansible_host
     ▼
10.10.1.20
```

You can therefore use a friendly logical name:

```text
web01
```

while the actual machine is:

```text
10.10.1.20
```

---

# 8. 👤 `ansible_user`

This specifies the remote user Ansible should use.

Example:

```ini
[webservers]
web01 ansible_host=10.10.1.20 ansible_user=dinesh
```

Equivalent conceptually to:

```bash
ssh dinesh@10.10.1.20
```

So:

```text
ansible_host
      ↓
WHERE?

ansible_user
      ↓
WHO?
```

### ⭐ Easy memory

> `ansible_host` = **which machine?**

> `ansible_user` = **which user?**

---

# 9. 🔐 `ansible_ssh_private_key_file`

You can specify the SSH private key:

```ini
[webservers]
web01 \
  ansible_host=10.10.1.20 \
  ansible_user=dinesh \
  ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Then Ansible can use:

```text
Private key
    │
    ▼
SSH
    │
    ▼
10.10.1.20
```

You may also encounter:

```text
ansible_private_key_file
```

in Ansible configurations/inventory examples.

For current practice, follow the connection-variable documentation and your organization's conventions; `ansible_private_key_file` is the generic connection variable.

---

# 10. 🔥 Complete SSH Inventory Example

```ini
[webservers]
web01 ansible_host=10.10.1.20 ansible_user=ansible
web02 ansible_host=10.10.1.21 ansible_user=ansible

[databases]
db01 ansible_host=10.10.2.20 ansible_user=dbadmin
```

Architecture:

```text
                    Control Node
                         │
             ┌───────────┼───────────┐
             │           │           │
             ▼           ▼           ▼
           web01       web02        db01
             │           │           │
       10.10.1.20   10.10.1.21   10.10.2.20
             │           │           │
           ansible     ansible      dbadmin
```

---

# 11. 🧪 Test Connectivity Before Running a Playbook

One of the most useful Ansible commands:

```bash
ansible all -i inventory -m ansible.builtin.ping
```

This does **not** mean ICMP ping.

This is an Ansible module test.

Flow:

```text
Ansible Controller
       │
       │ SSH
       ▼
Managed Node
       │
       ▼
Ansible ping module
       │
       ▼
pong
```

Expected:

```text
web01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### ⭐ Interview trap

Ansible `ping`:

```bash
ansible all -m ansible.builtin.ping
```

does **not** send an ICMP packet like:

```bash
ping 10.10.1.20
```

It verifies Ansible connectivity and remote execution capability.

---

# 12. 🔑 SSH Key Authentication

Production environments generally prefer SSH keys rather than interactive passwords.

Flow:

```text
                 Controller
                     │
              Private Key 🔑
                     │
                     ▼
                  SSH
                     │
                     ▼
              Managed Server
                     │
              authorized_keys
                     │
                     ▼
                 Authentication
```

On the managed node:

```text
~/.ssh/authorized_keys
```

contains the corresponding public key.

---

# 13. 🔐 SSH Password Authentication

You can also use a password.

For example:

```bash
ansible all -i inventory \
  -m ansible.builtin.ping \
  -u dinesh \
  --ask-pass
```

Ansible asks for the SSH password.

But for production automation, storing plain passwords in command lines or inventory is a bad practice.

Better approaches include:

* SSH keys
* Ansible Vault
* Automation Controller credentials
* enterprise secret-management systems

We'll cover these later.

---

# 14. 👑 What is `become`?

This is **very important**.

Suppose you connect as:

```text
ansible
```

but need root privileges.

You don't necessarily need to SSH directly as root.

Instead:

```yaml
become: true
```

Example:

```yaml
- name: Install nginx
  hosts: webservers
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Flow:

```text
Controller
    │
    │ SSH
    ▼
ansible user
    │
    │ sudo
    ▼
root
    │
    ▼
Install package
```

---

# 15. 🧠 `become` vs `ansible_user`

These are often confused.

### `ansible_user`

Controls:

> **Who do I connect as?**

Example:

```ini
ansible_user=ansible
```

### `become`

Controls:

> **Should I escalate privileges after connecting?**

Example:

```yaml
become: true
```

So:

```text
ansible_user
     ↓
SSH login identity

become
     ↓
Privilege escalation
```

---

# 16. 👤 `become_user`

You can specify the target user.

```yaml
become: true
become_user: root
```

Typical:

```text
SSH as ansible
       │
       ▼
sudo
       │
       ▼
root
```

But you can also become another user:

```yaml
become: true
become_user: postgres
```

Flow:

```text
SSH
 │
 ▼
ansible
 │
 ▼
sudo
 │
 ▼
postgres
```

This is very useful for database automation.

---

# 17. 🔧 `become_method`

By default on many Linux systems:

```text
sudo
```

But Ansible supports different privilege-escalation methods through become plugins.

Example:

```yaml
become: true
become_method: sudo
```

Common concepts include:

```text
sudo
su
doas
```

depending on platform and configuration.

---

# 18. 🏭 Production Example

Imagine your PostgreSQL server:

```text
db01
```

You connect as:

```text
ansible
```

But PostgreSQL commands need to run as:

```text
postgres
```

You could use:

```yaml
- name: PostgreSQL configuration
  hosts: databases

  tasks:

    - name: Check PostgreSQL user
      ansible.builtin.command:
        cmd: whoami
      become: true
      become_user: postgres
      register: result

    - name: Show user
      ansible.builtin.debug:
        var: result.stdout
```

Conceptually:

```text
Controller
    │
    │ SSH
    ▼
 ansible
    │
    │ sudo
    ▼
postgres
    │
    ▼
PostgreSQL operation
```

---

# 19. 🔥 Important: `become` Doesn't Mean "SSH as Root"

This is a very common interview trap.

If:

```yaml
become: true
```

it does **not necessarily mean**:

```text
SSH directly as root
```

Usually:

```text
SSH → normal user
          │
          ▼
     privilege escalation
          │
          ▼
         root
```

That's safer and more controllable.

---

# 20. 🌐 Connection Plugins

Now let's introduce an important Ansible architecture concept.

Ansible uses **connection plugins** to determine how it connects to the target.

Conceptually:

```text
                 Ansible
                    │
             Connection Plugin
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
      SSH          Local        WinRM
       │             │            │
       ▼             ▼            ▼
    Linux        Controller     Windows
```

The common connection variable is:

```ini
ansible_connection=
```

For example:

```ini
[webservers]
web01 ansible_connection=ssh
```

For local execution:

```ini
localhost ansible_connection=local
```

---

# 21. 🏠 Local Connection

Suppose you don't want Ansible to SSH anywhere.

You want the tasks to execute on the same machine where Ansible is running.

You can use:

```ini
localhost ansible_connection=local
```

Then:

```text
Controller
   │
   │ local connection
   ▼
Same machine
```

This is useful for tasks such as:

* generating files locally
* calling local commands
* interacting with local APIs
* orchestration logic

---

# 22. 🪟 Windows Connection

For Windows automation, common connection mechanisms include:

```text
WinRM
PSRP
```

Conceptually:

```text
Ansible Controller
        │
        │ WinRM / PSRP
        ▼
   Windows Server
```

The exact connection setup depends on the environment.

For your LevelUp preparation, know:

> **SSH is common for Linux; Windows commonly uses WinRM/PSRP; Ansible uses connection plugins to abstract the connection mechanism.**

---

# 23. 🛰️ Network Devices

Ansible can also manage network devices.

For example:

```text
Controller
    │
    ├── SSH
    ├── HTTP/API
    └── network-specific connection
          │
          ▼
      Router/Switch
```

Network automation often uses collections such as:

```text
cisco.ios
arista.eos
junipernetworks.junos
```

The exact connection type depends on the platform/collection.

---

# 24. 🔄 What Happens During Module Execution?

This is a little deeper and **very useful for interviews**.

Suppose:

```yaml
- name: Create directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
```

Conceptually:

```text
Controller
    │
    │ SSH
    ▼
Managed Node
    │
    │ module execution
    ▼
file module
    │
    ▼
/opt/myapp
```

Ansible transfers/executes the required module logic on the target as appropriate, passes parameters, collects the structured result, and reports:

```text
ok
changed
failed
skipped
```

The exact implementation details vary by module and Ansible version, so don't oversimplify this to:

> "Ansible always copies a Python file and runs it."

That's an incomplete model.

---

# 25. 🐍 Python on Managed Nodes

For many Linux modules, Python is commonly required on the managed node.

For example:

```text
Controller
    │
    │ SSH
    ▼
Managed Node
    │
    └── Python/runtime
```

However, there are exceptions and special execution models.

For example:

```text
raw
```

can execute commands without requiring Python on the remote host.

This is useful when bootstrapping a system where Python is not installed.

Example:

```yaml
- name: Bootstrap Python
  ansible.builtin.raw:
    cmd: dnf install -y python3
```

Then regular Ansible modules can be used.

### ⭐ Interview point

> Ansible often requires Python on Linux managed nodes for normal module execution, but modules such as `raw` can be used before Python is available.

---

# 26. 🧠 `ansible_host` vs `inventory_hostname`

This is an excellent interview question.

Suppose:

```ini
web01 ansible_host=10.10.1.20
```

Then:

```text
inventory_hostname
        ↓
web01

ansible_host
        ↓
10.10.1.20
```

So:

```text
inventory_hostname = inventory identifier

ansible_host = actual connection address
```

This distinction becomes extremely useful with:

* dynamic inventory
* cloud environments
* bastion hosts
* aliases
* multiple interfaces

---

# 27. 🛣️ Bastion / Jump Host

Production environments often don't allow:

```text
Controller ─────X────> Private Server
```

Instead:

```text
Controller
     │
     ▼
Bastion / Jump Host
     │
     ▼
Private Server
```

SSH can use a proxy/jump configuration.

Conceptually:

```text
                    Internet
                       │
                       ▼
                 Bastion Host
                       │
                 Private Network
                       │
                       ▼
                 Managed Server
```

This is very common in cloud environments.

We'll later cover how to configure this using SSH arguments and Ansible connection settings.

---

# 28. 🔥 Important Connection Variables

You don't need to memorize every variable today, but know these:

| Variable                     | Purpose                                 |
| ---------------------------- | --------------------------------------- |
| `ansible_host`               | Actual host/address to connect to       |
| `ansible_user`               | Remote login user                       |
| `ansible_port`               | SSH/connection port                     |
| `ansible_connection`         | Connection plugin/type                  |
| `ansible_password`           | Connection password, where applicable   |
| `ansible_private_key_file`   | Private key used for authentication     |
| `ansible_ssh_common_args`    | Extra SSH arguments                     |
| `ansible_python_interpreter` | Python interpreter path on managed node |
| `ansible_become`             | Enable privilege escalation             |
| `ansible_become_user`        | User to become                          |
| `ansible_become_method`      | Privilege escalation method             |

---

# 29. 🧪 Complete Production-Like Inventory

```ini
[webservers]
web01 ansible_host=10.10.1.20
web02 ansible_host=10.10.1.21

[databases]
db01 ansible_host=10.10.2.20

[webservers:vars]
ansible_user=ansible
ansible_private_key_file=~/.ssh/prod_ed25519
ansible_port=22

[databases:vars]
ansible_user=ansible
ansible_private_key_file=~/.ssh/prod_ed25519
```

Playbook:

```yaml
---
- name: Configure production web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Execution:

```bash
ansible-playbook -i inventory production.yml
```

Flow:

```text
                     Controller
                         │
                   inventory
                         │
                 webservers group
                    /          \
                   ▼            ▼
                web01         web02
                   │            │
                  SSH          SSH
                   │            │
                ansible      ansible
                   │            │
                 sudo         sudo
                   │            │
                  root         root
                   │            │
                   ▼            ▼
               nginx         nginx
```

---

# 30. 🚨 Common Interview Traps

### ❌ "Ansible always uses SSH."

Better:

> Ansible commonly uses SSH for Linux/Unix, but supports multiple connection plugins.

---

### ❌ "`ansible_host` is the inventory name."

No.

```text
inventory_hostname → web01
ansible_host       → 10.10.1.20
```

---

### ❌ "`become: true` means SSH as root."

No.

Usually:

```text
SSH as normal user
       ↓
Privilege escalation
       ↓
root
```

---

### ❌ "Ansible ping sends ICMP."

No.

```bash
ansible all -m ansible.builtin.ping
```

tests Ansible connectivity/module execution.

---

### ❌ "Managed nodes always need Python."

Not always.

Normal modules commonly need Python on Linux, but mechanisms such as `raw` can be used for bootstrapping.

---

# 🎤 LevelUp Interview Questions

### Q1. What is the difference between control and managed node?

> Control node runs Ansible; managed nodes are the systems Ansible automates.

### Q2. How does Ansible connect to Linux?

> Typically through SSH using the appropriate connection plugin.

### Q3. What is `ansible_host`?

> The actual hostname/IP/address Ansible uses to connect to an inventory host.

### Q4. What is `ansible_user`?

> The remote user used for the connection.

### Q5. What is `become`?

> Ansible's privilege-escalation mechanism, commonly used to execute tasks with elevated privileges such as root.

### Q6. `ansible_user` vs `become_user`?

> `ansible_user` is the account used for the initial connection. `become_user` is the account Ansible switches to through privilege escalation.

### Q7. How do you troubleshoot an SSH problem?

Start with:

```bash
ssh -v user@host
```

Then:

```bash
ansible-inventory -i inventory --graph
```

and:

```bash
ansible webservers -i inventory -m ansible.builtin.ping -vvv
```

`-vvv` provides verbose debugging information.

---

# 🧠 Final Mental Model

Remember this:

```text
                         🎛️ CONTROL NODE
                              │
                         Ansible CLI
                              │
                         Inventory
                              │
               ┌──────────────┼──────────────┐
               │              │              │
              SSH            SSH            SSH
               │              │              │
               ▼              ▼              ▼
            web01           web02           db01
               │              │              │
           ansible         ansible        ansible
               │              │              │
             sudo           sudo           sudo
               │              │              │
             root           root           postgres
```

And remember the distinction:

```text
┌─────────────────────────────────────┐
│         CONNECTION                  │
│                                     │
│ ansible_host  → WHERE?              │
│ ansible_user  → WHO?                │
│ ansible_port  → WHICH PORT?         │
│ ansible_connection → HOW?           │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│       PRIVILEGE ESCALATION          │
│                                     │
│ become       → escalate?            │
│ become_user  → TO WHOM?             │
│ become_method → HOW?                │
└─────────────────────────────────────┘
```

### 🎯 The most important production flow

```text
Ansible Controller
       │
       │ connection plugin
       ▼
SSH / WinRM / other
       │
       ▼
Managed Node
       │
       │ become
       ▼
Privileged User
       │
       ▼
Ansible Module
       │
       ▼
Desired State
```


### Question: 1. my doubt is, ssh key need to be generated on which node? on control node? and public key will be passed to remote nodes and private key will be placed in specific location on control node, hence it can be able to connect, please correct me?
2. why python needed on remote nodes? or managed nodes?

Yes — your understanding is **almost exactly correct**. 👍 Let's make both concepts crystal clear because these are important Ansible fundamentals.

# 1. 🔑 Where is the SSH key generated?

Yes, **normally the SSH key pair is generated on the Control Node**.

For example:

```bash
ssh-keygen -t ed25519
```

You run this on:

```text
🎛️ Control Node
```

It creates a key pair:

```text
~/.ssh/
├── id_ed25519          ← 🔒 PRIVATE KEY
└── id_ed25519.pub      ← 🔓 PUBLIC KEY
```

### The flow

```text
              🎛️ CONTROL NODE
                    │
              ssh-keygen
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
   id_ed25519          id_ed25519.pub
   🔒 PRIVATE KEY       🔓 PUBLIC KEY
          │                   │
          │                   │
          │              copied to
          │                   │
          │                   ▼
          │            Managed Node
          │                   │
          │          ~/.ssh/authorized_keys
          │                   │
          └────── SSH ────────┘
```

So your understanding is:

> **Generate the key pair on the Control Node, keep the private key on the Control Node, and place the corresponding public key in the managed node's `authorized_keys`.**

✅ **Correct.**

---

## 🔐 What exactly happens during SSH?

Suppose:

```text
Control Node
10.10.1.10
```

and:

```text
Managed Node
10.10.1.20
```

The Control Node has:

```text
🔒 private key
```

The managed server has:

```text
🔓 public key
```

When you do:

```bash
ssh ansible@10.10.1.20
```

SSH performs cryptographic authentication.

Conceptually:

```text
Control Node                         Managed Node
     │                                    │
     │ 🔒 Private Key                     │
     │                                    │
     │────── SSH authentication ─────────►│
     │                                    │
     │                            🔓 Public Key
     │                                    │
     │◄────── Authentication success ─────│
     │                                    │
     │──────── SSH session ───────────────►│
```

### ⚠️ Important correction

You don't normally **send the private key** to the managed node.

The private key should remain protected on the Control Node.

```text
🔒 PRIVATE KEY
     │
     └── NEVER copy to managed servers
```

Only the public key is installed on the managed server.

---

# 2. 🐍 Why does Ansible need Python on the managed node?

This is an extremely good question.

The reason is:

> **Many Ansible modules execute code on the managed node, and the standard Linux/Unix module execution model commonly uses Python.**

Let's take:

```yaml
- name: Create directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
```

You might think:

```text
Controller
   │
   │ SSH
   ▼
Server
   │
   └── create directory
```

But conceptually, Ansible does more than simply send:

```bash
mkdir /opt/myapp
```

The Ansible module needs to execute on the managed node.

Simplified:

```text
🎛️ Control Node
       │
       │ SSH
       ▼
🖥️ Managed Node
       │
       ▼
Python/runtime
       │
       ▼
Ansible module logic
       │
       ▼
/opt/myapp
```

---

# 3. 🧩 Why execute the module on the remote node?

Consider this:

```yaml
- name: Install package
  ansible.builtin.package:
    name: nginx
    state: present
```

The package actually needs to be installed **on the managed server**.

Therefore:

```text
Controller
    │
    │ sends module + parameters
    ▼
Managed Node
    │
    │ executes module
    ▼
Package manager
    │
    ▼
nginx installed
```

The module needs access to the managed node's:

* filesystem
* package manager
* services
* users
* permissions
* operating system
* network configuration

So the logic needs to execute **on the managed node**.

---

# 4. 🐍 Why Python specifically?

A large portion of Ansible's traditional Unix/Linux module ecosystem is implemented using Python.

For example:

```text
ansible.builtin.file
ansible.builtin.package
ansible.builtin.service
ansible.builtin.user
ansible.builtin.copy
ansible.builtin.template
```

Many of these modules rely on the Python runtime on the managed node.

So:

```text
Control Node
    │
    │ SSH
    ▼
Managed Node
    │
    └── Python
          │
          └── Ansible module
```

The Control Node also has Python/runtime requirements, but **the remote Python requirement is specifically important because modules commonly execute on the managed node**.

---

# 5. 🧠 Does the entire Ansible installation need to be installed on the managed node?

### ❌ No.

This is an important distinction.

You don't normally install:

```text
ansible
ansible-playbook
```

on every managed server.

Instead:

```text
🎛️ Control Node
├── Ansible
├── ansible-playbook
├── collections
└── playbooks

🖥️ Managed Node
├── Python/runtime as required
├── SSH
└── Target OS/application
```

So:

> **Ansible is installed on the Control Node; the managed node generally needs only the runtime/dependencies required by the modules being used.**

---

# 6. 🚨 What if Python isn't installed?

This is where `raw` becomes useful.

Suppose you have a brand-new RHEL machine:

```text
Managed Node

Python ❌
```

You can't necessarily use normal Python-based modules yet.

You can use:

```yaml
- name: Install Python
  ansible.builtin.raw:
    cmd: dnf install -y python3
```

Why can `raw` work?

Because it executes the command directly through the connection mechanism rather than requiring the normal Python module execution path.

Flow:

```text
Control Node
     │
     │ SSH
     ▼
Managed Node
     │
     │ raw
     ▼
dnf install python3
     │
     ▼
Python installed
     │
     ▼
Normal Ansible modules
can now be used
```

This is commonly called **bootstrapping** the managed node.

---

# 7. 🎯 Very Important Interview Answer

If an interviewer asks:

> **"Why does Ansible need Python on managed nodes?"**

A good answer is:

> "Ansible's standard module execution model for many Linux/Unix modules relies on Python on the managed node. The controller connects to the managed node, transfers or invokes the required module logic with its parameters, and the module executes there against the target system. Ansible itself does not need to be installed as a persistent agent on the managed node. For initial bootstrapping when Python isn't available, modules such as `raw` can be used."

That's a **strong A3-level answer**. 💪

---

# 8. 🧠 Put Both Concepts Together

Now you can visualize the complete connection:

```text
                  🎛️ CONTROL NODE
                  ────────────────
                  Ansible
                  Playbooks
                  Collections
                  🔒 Private SSH Key
                        │
                        │ SSH
                        │
                        ▼
              🖥️ MANAGED NODE
              ─────────────────
              🔓 Public Key
              ~/.ssh/authorized_keys
                        │
                        ▼
                     Python
                        │
                        ▼
                Ansible Modules
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
          Package     Service     File
             │          │          │
             ▼          ▼          ▼
           nginx      systemd    filesystem
```

### 🔑 Remember these two rules:

**SSH:**

> 🔒 Private key stays on the **Control Node**; 🔓 public key is installed on the **Managed Node**.

**Ansible modules:**

> 🐍 Many standard Linux/Unix modules execute on the **Managed Node** and commonly require Python there.

One subtlety: **not every Ansible module requires Python on the target**—`raw`, some network/API/local-style operations, and specialized module implementations can differ. So avoid saying "Ansible always requires Python."



# 🚀 Ansible Topic 4 — Ad-hoc Commands & Modules

Now we'll learn the **`ansible` command itself**.

You already know:

```bash
ansible-playbook -i inventory site.yml
```

That's for executing **playbooks**.

But:

```bash
ansible
```

is used for **ad-hoc operations**.

---

# 1. 🧠 What is an Ad-hoc Command?

### Definition

> An Ansible ad-hoc command is a one-time command used to perform a specific operation on one or more managed hosts without creating a playbook.

Example:

```bash
ansible webservers -i inventory -m ansible.builtin.ping
```

This says:

```text
webservers
    ↓
Target hosts

-m ping
    ↓
Use the ping module
```

---

# 2. `ansible` vs `ansible-playbook`

This distinction is fundamental.

| Command             | Purpose                     |
| ------------------- | --------------------------- |
| `ansible`           | Ad-hoc operations           |
| `ansible-playbook`  | Execute playbooks           |
| `ansible-inventory` | Inspect inventory           |
| `ansible-galaxy`    | Manage roles/collections    |
| `ansible-config`    | Inspect configuration       |
| `ansible-doc`       | Module/plugin documentation |

Think:

```text
              Ansible CLI
                  │
       ┌──────────┼───────────┐
       ▼          ▼           ▼
    ansible   ansible-     ansible-
               playbook     inventory
       │          │           │
    one-off    automation   inventory
    command     workflow     inspection
```

---

# 3. 🧪 Basic Ad-hoc Syntax

The general structure is:

```bash
ansible <pattern> -i <inventory> -m <module> -a "<arguments>"
```

For example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.ping
```

Break it down:

```text
ansible
   │
   ├── webservers       → target
   │
   ├── -i inventory     → inventory
   │
   ├── -m ping          → module
   │
   └── -a "..."         → module arguments
```

---

# 4. 🎯 Targeting Hosts

You can target:

### One host

```bash
ansible web01 -i inventory -m ansible.builtin.ping
```

### A group

```bash
ansible webservers -i inventory -m ansible.builtin.ping
```

### All hosts

```bash
ansible all -i inventory -m ansible.builtin.ping
```

### Exclude a host

```bash
ansible 'webservers:!web02' \
  -i inventory \
  -m ansible.builtin.ping
```

This uses the same **host-pattern system** we learned in Topic 3.

---

# 5. 🧩 `-m` — Module

`-m` specifies which Ansible module to execute.

Example:

```bash
-m ansible.builtin.ping
```

or:

```bash
-m ansible.builtin.package
```

or:

```bash
-m ansible.builtin.service
```

or:

```bash
-m ansible.builtin.copy
```

---

# 6. 📦 Example — Install a Package

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.package \
  -a "name=nginx state=present"
```

Meaning:

```text
webservers
     ↓
package module
     ↓
name = nginx
state = present
```

Conceptually:

```text
Controller
    │
    │ SSH
    ▼
web01
    │
    ▼
package module
    │
    ▼
nginx = present
```

---

# 7. 🛑 Don't Confuse `-m` and `-a`

This is an important syntax point.

```bash
-m ansible.builtin.package
```

means:

> **Which module?**

And:

```bash
-a "name=nginx state=present"
```

means:

> **What arguments should I give that module?**

So:

```text
-m → module
-a → module arguments
```

---

# 8. 🖥️ Example — Check Service

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.service \
  -a "name=nginx state=started"
```

This ensures:

```text
nginx
  ↓
started
```

Notice something important:

This is **declarative-style module usage**.

You're saying:

```text
state = started
```

rather than:

```bash
systemctl start nginx
```

---

# 9. 📁 Example — Create Directory

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.file \
  -a "path=/opt/myapp state=directory"
```

This ensures:

```text
/opt/myapp
    ↓
exists as directory
```

---

# 10. 👤 Example — Create User

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.user \
  -a "name=appuser state=present"
```

This ensures:

```text
appuser
   ↓
exists
```

---

# 11. 📄 Example — Copy a File

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.copy \
  -a "src=/tmp/test.txt dest=/tmp/test.txt"
```

Here:

```text
src
 ↓
Controller

dest
 ↓
Managed Node
```

Remember our earlier `remote_src` discussion.

By default:

```text
copy.src
   ↓
Controller
```

With:

```bash
remote_src=true
```

the source is on the managed node.

Example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.copy \
  -a "src=/tmp/source.txt dest=/opt/source.txt remote_src=true"
```

Now:

```text
Managed Node

/tmp/source.txt
       │
       ▼
/opt/source.txt
```

🔥 We'll revisit module parameters like this in detail.

---

# 12. 🐚 `command`

You can execute a command using:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.command \
  -a "uname -a"
```

This executes:

```text
uname -a
```

on the managed node.

---

# 13. 🐚 `shell`

You can also use:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.shell \
  -a "ps -ef | grep nginx"
```

The difference is important.

### `command`

Doesn't invoke a shell by default.

### `shell`

Executes through a shell.

Therefore shell features such as:

```text
|
>
>>
&&
$
```

can be used.

---

# 14. ⚠️ Why Not Always Use `shell`?

Bad:

```bash
ansible webservers \
  -m ansible.builtin.shell \
  -a "yum install -y nginx"
```

Better:

```bash
ansible webservers \
  -m ansible.builtin.package \
  -a "name=nginx state=present"
```

Why?

The module provides:

* desired-state semantics
* idempotency
* structured output
* better check-mode support
* clearer intent

So the production principle is:

> **Use a dedicated Ansible module whenever one exists.**

---

# 15. 🧨 `raw`

`raw` is particularly interesting.

Example:

```bash
ansible all \
  -i inventory \
  -m ansible.builtin.raw \
  -a "uname -a"
```

Unlike most normal modules, `raw` doesn't depend on the normal remote Python module execution mechanism.

This makes it useful for **bootstrapping**.

For example:

```bash
ansible all \
  -i inventory \
  -m ansible.builtin.raw \
  -a "dnf install -y python3"
```

Then normal Python-based modules can be used.

---

# 16. 📜 `script`

`script` lets you execute a local script on the remote host.

For example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.script \
  -a "./setup.sh"
```

Conceptually:

```text
Controller
    │
    │ setup.sh
    ▼
Managed Node
    │
    ▼
execute script
```

Again, if a proper Ansible module can express the operation, that is generally preferable.

---

# 17. 👑 `-b` — Become

You can enable privilege escalation with:

```bash
-b
```

Example:

```bash
ansible webservers \
  -i inventory \
  -b \
  -m ansible.builtin.package \
  -a "name=nginx state=present"
```

Equivalent conceptually to:

```yaml
become: true
```

Flow:

```text
SSH
 │
 ▼
ansible user
 │
 ▼
sudo
 │
 ▼
root
 │
 ▼
package operation
```

---

# 18. 🔐 `-K` — Ask Become Password

If sudo requires a password:

```bash
ansible webservers \
  -i inventory \
  -b \
  -K \
  -m ansible.builtin.command \
  -a "whoami"
```

`-K` means:

> Ask for the privilege-escalation password.

Don't confuse it with:

```text
-k
```

which relates to the connection password.

---

# 19. 🔑 `-k` — Ask Connection Password

```bash
ansible webservers \
  -i inventory \
  -k \
  -m ansible.builtin.ping
```

`-k` asks for the SSH/connection password.

So:

```text
-k → connection password

-K → become/sudo password
```

🔥 This is a common interview/command-line trap.

---

# 20. 👤 `-u` — Remote User

You can override the inventory's `ansible_user`:

```bash
ansible webservers \
  -i inventory \
  -u ansible \
  -m ansible.builtin.ping
```

Equivalent conceptually to:

```ini
ansible_user=ansible
```

---

# 21. 🌐 `-e` — Extra Variables

You can pass variables from the command line:

```bash
ansible-playbook \
  -i inventory \
  site.yml \
  -e "app_version=2.0"
```

Or:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.debug \
  -a "msg={{ app_version }}" \
  -e "app_version=2.0"
```

This is called an **extra variable**.

⚠️ Extra vars have very high precedence in Ansible's variable precedence hierarchy.

We'll study this deeply later.

---

# 22. 🔎 Verbosity — `-v`, `-vv`, `-vvv`

When troubleshooting:

```bash
-v
```

more information.

```bash
-vv
```

even more.

```bash
-vvv
```

very detailed debugging.

Example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.ping \
  -vvv
```

This is extremely useful for diagnosing:

* SSH problems
* authentication
* connection plugins
* privilege escalation
* module execution
* Python interpreter problems

---

# 23. 🧪 The Most Useful Ad-hoc Command

Probably:

```bash
ansible all \
  -i inventory \
  -m ansible.builtin.ping
```

Use it when you want to quickly verify:

```text
Inventory
   ↓
Host matching
   ↓
Connection
   ↓
Authentication
   ↓
Remote execution
```

---

# 24. 🔥 Ad-hoc Command Examples Cheat Sheet

| Requirement       | Command                                                                  |
| ----------------- | ------------------------------------------------------------------------ |
| Test connectivity | `ansible all -m ansible.builtin.ping`                                    |
| Check hostname    | `ansible all -m ansible.builtin.command -a "hostname"`                   |
| Check uptime      | `ansible all -m ansible.builtin.command -a "uptime"`                     |
| Install package   | `ansible all -m ansible.builtin.package -a "name=nginx state=present"`   |
| Start service     | `ansible all -m ansible.builtin.service -a "name=nginx state=started"`   |
| Create directory  | `ansible all -m ansible.builtin.file -a "path=/opt/app state=directory"` |
| Create user       | `ansible all -m ansible.builtin.user -a "name=appuser state=present"`    |
| Execute shell     | `ansible all -m ansible.builtin.shell -a "ps -ef \| grep nginx"`         |
| Become root       | `ansible all -b -m ...`                                                  |
| Ask sudo password | `ansible all -b -K -m ...`                                               |
| Ask SSH password  | `ansible all -k -m ...`                                                  |
| Verbose debug     | `ansible all -vvv -m ...`                                                |

---

# 25. 🧠 Ad-hoc vs Playbook

This distinction matters a lot in production.

### Ad-hoc

```bash
ansible webservers -m ansible.builtin.service \
  -a "name=nginx state=started"
```

Good for:

* quick troubleshooting
* one-time operations
* checking connectivity
* emergency operations
* inspecting systems

### Playbook

```yaml
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Ensure nginx is installed
      ansible.builtin.package:
        name: nginx
        state: present
```

Good for:

* repeatable automation
* version control
* CI/CD
* review
* testing
* production deployments
* complex workflows

### Production rule

> **If you're repeatedly performing the same operation, put it in a playbook rather than relying on ad-hoc commands.**

---

# 26. 🎯 Real Production Scenario

Imagine you're on call.

You receive:

> "Is nginx running on all production web servers?"

You don't need to create a playbook.

You can quickly execute:

```bash
ansible webservers \
  -i production.ini \
  -m ansible.builtin.service \
  -a "name=nginx state=started"
```

Or inspect:

```bash
ansible webservers \
  -i production.ini \
  -m ansible.builtin.command \
  -a "systemctl is-active nginx"
```

Then you discover:

```text
web01 → active
web02 → active
web03 → inactive
```

Now you can investigate `web03`.

This is where ad-hoc commands shine.

---

# 27. ⚠️ Important: `command` Isn't Automatically Idempotent

Consider:

```bash
ansible webservers \
  -m ansible.builtin.command \
  -a "touch /tmp/test"
```

You're essentially saying:

> Run this command.

Ansible doesn't automatically understand the desired state behind arbitrary commands.

Similarly:

```bash
ansible webservers \
  -m ansible.builtin.shell \
  -a "echo hello >> /tmp/test"
```

Repeated execution modifies the file repeatedly.

So:

```text
Dedicated module
       ↓
Usually state-aware
       ↓
Better idempotency

command/shell
       ↓
Arbitrary operation
       ↓
You must design carefully
```

---

# 28. 🔥 `creates` and `removes`

This connects directly to the production options you asked me not to skip.

Suppose you really need `command`.

You can use:

```bash
ansible webservers \
  -m ansible.builtin.command \
  -a "touch /opt/app/initialized creates=/opt/app/initialized"
```

The idea is:

> Don't execute the command if the specified file already exists.

Conceptually:

```text
Does /opt/app/initialized exist?
          │
     ┌────┴────┐
     │         │
    YES        NO
     │         │
   skip      execute
```

Similarly, `removes` can condition execution based on a path existing.

We'll cover these parameters deeply when we study the individual modules.

---

# 29. 🧠 `ansible-doc`

When you're unsure about a module, don't rely on memory.

Use:

```bash
ansible-doc ansible.builtin.copy
```

or:

```bash
ansible-doc ansible.builtin.file
```

You can search:

```bash
ansible-doc -l
```

This lists available modules/plugins.

For a module:

```bash
ansible-doc ansible.builtin.copy
```

You'll find:

* parameters
* descriptions
* defaults
* examples
* return values
* attributes
* check-mode support
* idempotency-related behavior

🔥 This is a **real production skill**.

---

# 30. 🧠 `ansible-doc` + `remote_src`

For your earlier question, you can literally verify it:

```bash
ansible-doc ansible.builtin.copy
```

Search for:

```text
remote_src
```

You'll see that it controls whether the `src` path is interpreted as being on the controller or the remote host.

This is how I want you to work with Ansible long-term:

> **Don't memorize every module parameter. Know how to quickly find, understand and correctly apply it.**

But for LevelUp, we **will memorize the high-value production parameters**.

---

# 31. 🎤 LevelUp Interview Questions

### Q1. What is an ad-hoc command?

> A one-time Ansible operation executed directly from the command line without creating a playbook.

### Q2. Difference between `ansible` and `ansible-playbook`?

> `ansible` executes ad-hoc operations; `ansible-playbook` executes YAML playbooks.

### Q3. What does `-m` mean?

> Specifies the Ansible module to execute.

### Q4. What does `-a` mean?

> Specifies arguments passed to the module.

### Q5. Difference between `-k` and `-K`?

> `-k` asks for the connection password; `-K` asks for the become/privilege-escalation password.

### Q6. What does `-b` do?

> Enables privilege escalation using Ansible's become mechanism.

### Q7. How do you debug an SSH problem?

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.ping \
  -vvv
```

### Q8. When should you use ad-hoc commands?

> For quick, one-time operations, diagnostics, and troubleshooting. Repeatable or complex automation should generally be implemented as version-controlled playbooks.

---


# 🚀 Ansible Topic 4 — Ad-hoc Commands & Modules

Now we'll learn the **`ansible` command itself**.

You already know:

```bash
ansible-playbook -i inventory site.yml
```

That's for executing **playbooks**.

But:

```bash
ansible
```

is used for **ad-hoc operations**.

---

# 1. 🧠 What is an Ad-hoc Command?

### Definition

> An Ansible ad-hoc command is a one-time command used to perform a specific operation on one or more managed hosts without creating a playbook.

Example:

```bash
ansible webservers -i inventory -m ansible.builtin.ping
```

This says:

```text
webservers
    ↓
Target hosts

-m ping
    ↓
Use the ping module
```

---

# 2. `ansible` vs `ansible-playbook`

This distinction is fundamental.

| Command             | Purpose                     |
| ------------------- | --------------------------- |
| `ansible`           | Ad-hoc operations           |
| `ansible-playbook`  | Execute playbooks           |
| `ansible-inventory` | Inspect inventory           |
| `ansible-galaxy`    | Manage roles/collections    |
| `ansible-config`    | Inspect configuration       |
| `ansible-doc`       | Module/plugin documentation |

Think:

```text
              Ansible CLI
                  │
       ┌──────────┼───────────┐
       ▼          ▼           ▼
    ansible   ansible-     ansible-
               playbook     inventory
       │          │           │
    one-off    automation   inventory
    command     workflow     inspection
```

---

# 3. 🧪 Basic Ad-hoc Syntax

The general structure is:

```bash
ansible <pattern> -i <inventory> -m <module> -a "<arguments>"
```

For example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.ping
```

Break it down:

```text
ansible
   │
   ├── webservers       → target
   │
   ├── -i inventory     → inventory
   │
   ├── -m ping          → module
   │
   └── -a "..."         → module arguments
```

---

# 4. 🎯 Targeting Hosts

You can target:

### One host

```bash
ansible web01 -i inventory -m ansible.builtin.ping
```

### A group

```bash
ansible webservers -i inventory -m ansible.builtin.ping
```

### All hosts

```bash
ansible all -i inventory -m ansible.builtin.ping
```

### Exclude a host

```bash
ansible 'webservers:!web02' \
  -i inventory \
  -m ansible.builtin.ping
```

This uses the same **host-pattern system** we learned in Topic 3.

---

# 5. 🧩 `-m` — Module

`-m` specifies which Ansible module to execute.

Example:

```bash
-m ansible.builtin.ping
```

or:

```bash
-m ansible.builtin.package
```

or:

```bash
-m ansible.builtin.service
```

or:

```bash
-m ansible.builtin.copy
```

---

# 6. 📦 Example — Install a Package

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.package \
  -a "name=nginx state=present"
```

Meaning:

```text
webservers
     ↓
package module
     ↓
name = nginx
state = present
```

Conceptually:

```text
Controller
    │
    │ SSH
    ▼
web01
    │
    ▼
package module
    │
    ▼
nginx = present
```

---

# 7. 🛑 Don't Confuse `-m` and `-a`

This is an important syntax point.

```bash
-m ansible.builtin.package
```

means:

> **Which module?**

And:

```bash
-a "name=nginx state=present"
```

means:

> **What arguments should I give that module?**

So:

```text
-m → module
-a → module arguments
```

---

# 8. 🖥️ Example — Check Service

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.service \
  -a "name=nginx state=started"
```

This ensures:

```text
nginx
  ↓
started
```

Notice something important:

This is **declarative-style module usage**.

You're saying:

```text
state = started
```

rather than:

```bash
systemctl start nginx
```

---

# 9. 📁 Example — Create Directory

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.file \
  -a "path=/opt/myapp state=directory"
```

This ensures:

```text
/opt/myapp
    ↓
exists as directory
```

---

# 10. 👤 Example — Create User

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.user \
  -a "name=appuser state=present"
```

This ensures:

```text
appuser
   ↓
exists
```

---

# 11. 📄 Example — Copy a File

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.copy \
  -a "src=/tmp/test.txt dest=/tmp/test.txt"
```

Here:

```text
src
 ↓
Controller

dest
 ↓
Managed Node
```

Remember our earlier `remote_src` discussion.

By default:

```text
copy.src
   ↓
Controller
```

With:

```bash
remote_src=true
```

the source is on the managed node.

Example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.copy \
  -a "src=/tmp/source.txt dest=/opt/source.txt remote_src=true"
```

Now:

```text
Managed Node

/tmp/source.txt
       │
       ▼
/opt/source.txt
```

🔥 We'll revisit module parameters like this in detail.

---

# 12. 🐚 `command`

You can execute a command using:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.command \
  -a "uname -a"
```

This executes:

```text
uname -a
```

on the managed node.

---

# 13. 🐚 `shell`

You can also use:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.shell \
  -a "ps -ef | grep nginx"
```

The difference is important.

### `command`

Doesn't invoke a shell by default.

### `shell`

Executes through a shell.

Therefore shell features such as:

```text
|
>
>>
&&
$
```

can be used.

---

# 14. ⚠️ Why Not Always Use `shell`?

Bad:

```bash
ansible webservers \
  -m ansible.builtin.shell \
  -a "yum install -y nginx"
```

Better:

```bash
ansible webservers \
  -m ansible.builtin.package \
  -a "name=nginx state=present"
```

Why?

The module provides:

* desired-state semantics
* idempotency
* structured output
* better check-mode support
* clearer intent

So the production principle is:

> **Use a dedicated Ansible module whenever one exists.**

---

# 15. 🧨 `raw`

`raw` is particularly interesting.

Example:

```bash
ansible all \
  -i inventory \
  -m ansible.builtin.raw \
  -a "uname -a"
```

Unlike most normal modules, `raw` doesn't depend on the normal remote Python module execution mechanism.

This makes it useful for **bootstrapping**.

For example:

```bash
ansible all \
  -i inventory \
  -m ansible.builtin.raw \
  -a "dnf install -y python3"
```

Then normal Python-based modules can be used.

---

# 16. 📜 `script`

`script` lets you execute a local script on the remote host.

For example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.script \
  -a "./setup.sh"
```

Conceptually:

```text
Controller
    │
    │ setup.sh
    ▼
Managed Node
    │
    ▼
execute script
```

Again, if a proper Ansible module can express the operation, that is generally preferable.

---

# 17. 👑 `-b` — Become

You can enable privilege escalation with:

```bash
-b
```

Example:

```bash
ansible webservers \
  -i inventory \
  -b \
  -m ansible.builtin.package \
  -a "name=nginx state=present"
```

Equivalent conceptually to:

```yaml
become: true
```

Flow:

```text
SSH
 │
 ▼
ansible user
 │
 ▼
sudo
 │
 ▼
root
 │
 ▼
package operation
```

---

# 18. 🔐 `-K` — Ask Become Password

If sudo requires a password:

```bash
ansible webservers \
  -i inventory \
  -b \
  -K \
  -m ansible.builtin.command \
  -a "whoami"
```

`-K` means:

> Ask for the privilege-escalation password.

Don't confuse it with:

```text
-k
```

which relates to the connection password.

---

# 19. 🔑 `-k` — Ask Connection Password

```bash
ansible webservers \
  -i inventory \
  -k \
  -m ansible.builtin.ping
```

`-k` asks for the SSH/connection password.

So:

```text
-k → connection password

-K → become/sudo password
```

🔥 This is a common interview/command-line trap.

---

# 20. 👤 `-u` — Remote User

You can override the inventory's `ansible_user`:

```bash
ansible webservers \
  -i inventory \
  -u ansible \
  -m ansible.builtin.ping
```

Equivalent conceptually to:

```ini
ansible_user=ansible
```

---

# 21. 🌐 `-e` — Extra Variables

You can pass variables from the command line:

```bash
ansible-playbook \
  -i inventory \
  site.yml \
  -e "app_version=2.0"
```

Or:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.debug \
  -a "msg={{ app_version }}" \
  -e "app_version=2.0"
```

This is called an **extra variable**.

⚠️ Extra vars have very high precedence in Ansible's variable precedence hierarchy.

We'll study this deeply later.

---

# 22. 🔎 Verbosity — `-v`, `-vv`, `-vvv`

When troubleshooting:

```bash
-v
```

more information.

```bash
-vv
```

even more.

```bash
-vvv
```

very detailed debugging.

Example:

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.ping \
  -vvv
```

This is extremely useful for diagnosing:

* SSH problems
* authentication
* connection plugins
* privilege escalation
* module execution
* Python interpreter problems

---

# 23. 🧪 The Most Useful Ad-hoc Command

Probably:

```bash
ansible all \
  -i inventory \
  -m ansible.builtin.ping
```

Use it when you want to quickly verify:

```text
Inventory
   ↓
Host matching
   ↓
Connection
   ↓
Authentication
   ↓
Remote execution
```

---

# 24. 🔥 Ad-hoc Command Examples Cheat Sheet

| Requirement       | Command                                                                  |
| ----------------- | ------------------------------------------------------------------------ |
| Test connectivity | `ansible all -m ansible.builtin.ping`                                    |
| Check hostname    | `ansible all -m ansible.builtin.command -a "hostname"`                   |
| Check uptime      | `ansible all -m ansible.builtin.command -a "uptime"`                     |
| Install package   | `ansible all -m ansible.builtin.package -a "name=nginx state=present"`   |
| Start service     | `ansible all -m ansible.builtin.service -a "name=nginx state=started"`   |
| Create directory  | `ansible all -m ansible.builtin.file -a "path=/opt/app state=directory"` |
| Create user       | `ansible all -m ansible.builtin.user -a "name=appuser state=present"`    |
| Execute shell     | `ansible all -m ansible.builtin.shell -a "ps -ef \| grep nginx"`         |
| Become root       | `ansible all -b -m ...`                                                  |
| Ask sudo password | `ansible all -b -K -m ...`                                               |
| Ask SSH password  | `ansible all -k -m ...`                                                  |
| Verbose debug     | `ansible all -vvv -m ...`                                                |

---

# 25. 🧠 Ad-hoc vs Playbook

This distinction matters a lot in production.

### Ad-hoc

```bash
ansible webservers -m ansible.builtin.service \
  -a "name=nginx state=started"
```

Good for:

* quick troubleshooting
* one-time operations
* checking connectivity
* emergency operations
* inspecting systems

### Playbook

```yaml
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Ensure nginx is installed
      ansible.builtin.package:
        name: nginx
        state: present
```

Good for:

* repeatable automation
* version control
* CI/CD
* review
* testing
* production deployments
* complex workflows

### Production rule

> **If you're repeatedly performing the same operation, put it in a playbook rather than relying on ad-hoc commands.**

---

# 26. 🎯 Real Production Scenario

Imagine you're on call.

You receive:

> "Is nginx running on all production web servers?"

You don't need to create a playbook.

You can quickly execute:

```bash
ansible webservers \
  -i production.ini \
  -m ansible.builtin.service \
  -a "name=nginx state=started"
```

Or inspect:

```bash
ansible webservers \
  -i production.ini \
  -m ansible.builtin.command \
  -a "systemctl is-active nginx"
```

Then you discover:

```text
web01 → active
web02 → active
web03 → inactive
```

Now you can investigate `web03`.

This is where ad-hoc commands shine.

---

# 27. ⚠️ Important: `command` Isn't Automatically Idempotent

Consider:

```bash
ansible webservers \
  -m ansible.builtin.command \
  -a "touch /tmp/test"
```

You're essentially saying:

> Run this command.

Ansible doesn't automatically understand the desired state behind arbitrary commands.

Similarly:

```bash
ansible webservers \
  -m ansible.builtin.shell \
  -a "echo hello >> /tmp/test"
```

Repeated execution modifies the file repeatedly.

So:

```text
Dedicated module
       ↓
Usually state-aware
       ↓
Better idempotency

command/shell
       ↓
Arbitrary operation
       ↓
You must design carefully
```

---

# 28. 🔥 `creates` and `removes`

This connects directly to the production options you asked me not to skip.

Suppose you really need `command`.

You can use:

```bash
ansible webservers \
  -m ansible.builtin.command \
  -a "touch /opt/app/initialized creates=/opt/app/initialized"
```

The idea is:

> Don't execute the command if the specified file already exists.

Conceptually:

```text
Does /opt/app/initialized exist?
          │
     ┌────┴────┐
     │         │
    YES        NO
     │         │
   skip      execute
```

Similarly, `removes` can condition execution based on a path existing.

We'll cover these parameters deeply when we study the individual modules.

---

# 29. 🧠 `ansible-doc`

When you're unsure about a module, don't rely on memory.

Use:

```bash
ansible-doc ansible.builtin.copy
```

or:

```bash
ansible-doc ansible.builtin.file
```

You can search:

```bash
ansible-doc -l
```

This lists available modules/plugins.

For a module:

```bash
ansible-doc ansible.builtin.copy
```

You'll find:

* parameters
* descriptions
* defaults
* examples
* return values
* attributes
* check-mode support
* idempotency-related behavior

🔥 This is a **real production skill**.

---

# 30. 🧠 `ansible-doc` + `remote_src`

For your earlier question, you can literally verify it:

```bash
ansible-doc ansible.builtin.copy
```

Search for:

```text
remote_src
```

You'll see that it controls whether the `src` path is interpreted as being on the controller or the remote host.

This is how I want you to work with Ansible long-term:

> **Don't memorize every module parameter. Know how to quickly find, understand and correctly apply it.**

But for LevelUp, we **will memorize the high-value production parameters**.

---

# 31. 🎤 LevelUp Interview Questions

### Q1. What is an ad-hoc command?

> A one-time Ansible operation executed directly from the command line without creating a playbook.

### Q2. Difference between `ansible` and `ansible-playbook`?

> `ansible` executes ad-hoc operations; `ansible-playbook` executes YAML playbooks.

### Q3. What does `-m` mean?

> Specifies the Ansible module to execute.

### Q4. What does `-a` mean?

> Specifies arguments passed to the module.

### Q5. Difference between `-k` and `-K`?

> `-k` asks for the connection password; `-K` asks for the become/privilege-escalation password.

### Q6. What does `-b` do?

> Enables privilege escalation using Ansible's become mechanism.

### Q7. How do you debug an SSH problem?

```bash
ansible webservers \
  -i inventory \
  -m ansible.builtin.ping \
  -vvv
```

### Q8. When should you use ad-hoc commands?

> For quick, one-time operations, diagnostics, and troubleshooting. Repeatable or complex automation should generally be implemented as version-controlled playbooks.

---



# 🚀 Ansible Topic 5 — Modules & Module Parameters

Now we're entering a **very important Ansible area**.

You already know:

```text
Playbook
   ↓
Task
   ↓
Module
```

The **module is what actually performs the operation**.

For your LevelUp preparation, I don't want you to memorize hundreds of modules. Instead, we'll build a strong understanding of the **production modules + important parameters + when to use each one + common interview traps**.

---

# 1. 🧠 What is an Ansible Module?

### Definition

> Ansible module is a reusable unit of code that performs a specific operation on a managed node or interacts with an external system.

Examples:

```text
ansible.builtin.file
ansible.builtin.copy
ansible.builtin.template
ansible.builtin.package
ansible.builtin.service
ansible.builtin.user
ansible.builtin.command
ansible.builtin.shell
ansible.builtin.uri
```

Example:

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
```

Here:

```text
Task
 │
 └── file module
       │
       ├── path
       └── state
```

---

# 2. 🎯 Why Modules Instead of Shell Commands?

Suppose you want nginx installed.

### Imperative approach

```yaml
- name: Install nginx
  ansible.builtin.shell:
    cmd: dnf install -y nginx
```

You're telling Ansible:

> Execute this command.

### Module approach

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

You're telling Ansible:

> Ensure this desired state exists.

That's a major difference.

```text
shell/command
      ↓
"DO THIS"

module
      ↓
"ENSURE THIS STATE"
```

---

# 3. ♻️ Idempotency and Modules

This is one of your **most important LevelUp concepts**.

Suppose:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

First run:

```text
nginx absent
     ↓
install
     ↓
changed
```

Second run:

```text
nginx already installed
     ↓
nothing to do
     ↓
ok
```

Therefore:

```text
First run  → changed
Second run → ok
```

But don't make the mistake of saying:

> "Every Ansible module is automatically idempotent."

That's not universally true.

Modules that manage state are generally designed for idempotency, while modules such as:

```text
command
shell
raw
```

execute operations and may require you to design idempotency yourself.

---

# 4. 🧰 The Modules You Should Know

For your production/LevelUp preparation, these are particularly important:

| Area         | Modules                                |
| ------------ | -------------------------------------- |
| Files        | `file`, `copy`, `template`, `fetch`    |
| Text/config  | `lineinfile`, `blockinfile`, `replace` |
| Packages     | `package`, `dnf`, `apt`                |
| Services     | `service`, `systemd_service`           |
| Users        | `user`, `group`                        |
| Archives     | `unarchive`, `archive`                 |
| Network/HTTP | `uri`, `get_url`                       |
| Inspection   | `stat`, `find`, `slurp`                |
| Commands     | `command`, `shell`, `raw`, `script`    |
| System       | `mount`, `hostname`, `reboot`          |
| Automation   | `set_fact`, `debug`, `assert`          |

We'll go through them.

---

# 5. 📁 `file` Module

The `file` module manages filesystem objects.

It can manage:

* directories
* files
* symbolic links
* permissions
* ownership
* deletion

---

## Create directory

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
```

Result:

```text
/opt
 └── myapp/
```

---

## Set permissions

```yaml
- name: Create secure directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    mode: '0755'
```

---

## Set owner/group

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appgroup
    mode: '0755'
```

---

## Create empty file

```yaml
- name: Create configuration file
  ansible.builtin.file:
    path: /opt/myapp/app.conf
    state: touch
```

⚠️ `state: touch` is not the same as simply ensuring an existing file remains unchanged; timestamps can be affected.

If you just want to ensure a file exists, understand the behavior carefully and consider whether `copy`, `template`, or another module better expresses the desired state.

---

## Delete file

```yaml
- name: Remove old configuration
  ansible.builtin.file:
    path: /opt/myapp/old.conf
    state: absent
```

---

# 6. 🔗 Symbolic Link

```yaml
- name: Create current symlink
  ansible.builtin.file:
    src: /opt/myapp/releases/1.2.0
    dest: /opt/myapp/current
    state: link
```

Result:

```text
/opt/myapp/
├── releases/
│   └── 1.2.0/
│
└── current -> releases/1.2.0
```

Very common in application deployments.

---

# 7. 📋 Important `file` Parameters

| Parameter | Meaning                                  |
| --------- | ---------------------------------------- |
| `path`    | Target path                              |
| `state`   | Desired filesystem state                 |
| `owner`   | Owner                                    |
| `group`   | Group                                    |
| `mode`    | Permissions                              |
| `recurse` | Recursively apply attributes             |
| `src`     | Source for links                         |
| `force`   | Force certain operations                 |
| `follow`  | Follow symlinks in applicable operations |

### ⭐ `recurse`

Example:

```yaml
- name: Set permissions recursively
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appgroup
    recurse: true
```

This applies ownership/group changes recursively.

---

# 8. 📄 `copy` Module

The `copy` module copies files from the **controller to the managed node** by default.

```yaml
- name: Deploy configuration
  ansible.builtin.copy:
    src: files/app.conf
    dest: /etc/myapp/app.conf
```

Flow:

```text
Controller
   │
   │ src
   ▼
Managed Node
   │
   └── /etc/myapp/app.conf
```

---

# 9. 🔥 `remote_src`

This is the parameter you specifically asked about earlier.

Normally:

```yaml
src: /tmp/app.tar.gz
```

means:

```text
Controller
    │
    └── /tmp/app.tar.gz
```

With:

```yaml
remote_src: true
```

the source is on the managed node:

```yaml
- name: Copy remote file
  ansible.builtin.copy:
    src: /tmp/app.tar.gz
    dest: /opt/app/app.tar.gz
    remote_src: true
```

Now:

```text
Managed Node

/tmp/app.tar.gz
       │
       ▼
/opt/app/app.tar.gz
```

### ⭐ Interview answer

> By default, `copy.src` refers to a source on the controller. `remote_src: true` tells the module that the source already exists on the managed node.

---

# 10. 🔥 Important `copy` Parameters

| Parameter        | Purpose                                   |
| ---------------- | ----------------------------------------- |
| `src`            | Source file                               |
| `dest`           | Destination                               |
| `remote_src`     | Source is on remote host                  |
| `owner`          | File owner                                |
| `group`          | File group                                |
| `mode`           | Permissions                               |
| `backup`         | Backup destination before replacement     |
| `force`          | Replace if contents differ                |
| `validate`       | Validate before replacing                 |
| `directory_mode` | Permissions for newly created directories |
| `follow`         | Follow destination symlink behavior       |

---

# 11. 🛡️ `validate` — Very Important Production Option

This is an excellent production feature.

Suppose you're deploying an nginx configuration.

You don't want:

```text
copy configuration
       ↓
nginx configuration broken
       ↓
restart nginx
       ↓
💥 service fails
```

Instead:

```yaml
- name: Deploy nginx configuration
  ansible.builtin.copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
    validate: '/usr/sbin/nginx -t -c %s'
```

Flow:

```text
New configuration
       │
       ▼
   Validate
       │
   ┌───┴───┐
   ▼       ▼
 valid   invalid
   │       │
   ▼       ▼
 install  reject
```

This is **excellent production automation practice**.

---

# 12. 💾 `backup`

```yaml
- name: Update configuration
  ansible.builtin.copy:
    src: app.conf
    dest: /etc/myapp/app.conf
    backup: true
```

This tells Ansible to create a backup when replacing the destination file.

Useful when configuration changes need a rollback artifact.

---

# 13. 🧱 `template`

This is one of the **most important Ansible modules**.

`template` uses **Jinja2 templates**.

Example:

```yaml
- name: Deploy application configuration
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
```

Template:

```jinja2
server:
  port: {{ http_port }}

environment:
  {{ environment }}
```

Variables:

```yaml
http_port: 8080
environment: production
```

Generated file:

```text
server:
  port: 8080

environment:
  production
```

---

# 14. 🧠 `copy` vs `template`

This is a common interview question.

### `copy`

Use when the content is mostly static:

```text
config.txt
```

### `template`

Use when content depends on variables:

```text
config.txt.j2
```

Mental model:

```text
Static file
    ↓
copy

Dynamic configuration
    ↓
template
    ↓
Jinja2
    ↓
Rendered configuration
```

---

# 15. 📥 `fetch`

`fetch` is basically the reverse direction of `copy`.

### `copy`

```text
Controller → Managed Node
```

### `fetch`

```text
Managed Node → Controller
```

Example:

```yaml
- name: Collect application log
  ansible.builtin.fetch:
    src: /var/log/myapp/app.log
    dest: ./collected-logs/
```

Flow:

```text
Managed Node
    │
    │ fetch
    ▼
Controller
```

Very useful for:

* collecting logs
* collecting configuration
* troubleshooting
* evidence gathering

---

# 16. 🌐 `get_url`

Downloads files from a URL to the managed node.

```yaml
- name: Download application package
  ansible.builtin.get_url:
    url: https://example.com/app.tar.gz
    dest: /opt/app/app.tar.gz
```

Flow:

```text
Internet
   │
   ▼
Managed Node
   │
   ▼
/opt/app/app.tar.gz
```

### Important distinction

`get_url`:

```text
URL → Managed Node
```

`copy`:

```text
Controller → Managed Node
```

---

# 17. 🌐 `uri`

`uri` is used for HTTP/HTTPS interactions.

Example:

```yaml
- name: Check application health
  ansible.builtin.uri:
    url: http://localhost:8080/health
    method: GET
    status_code: 200
```

This is very useful for:

* REST APIs
* health checks
* deployment validation
* API calls
* authentication workflows

---

# 18. 📦 `package`

Generic package management module.

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

The advantage:

> You don't have to hard-code the underlying package manager in simple cross-platform automation.

For example:

```text
RHEL
 ↓
dnf

Debian
 ↓
apt
```

The generic package module can abstract that in appropriate cases.

---

# 19. 🟥 `dnf`

For RHEL/Fedora-family systems:

```yaml
- name: Install PostgreSQL
  ansible.builtin.dnf:
    name: postgresql
    state: present
```

Use `dnf` when you need package-manager-specific capabilities.

---

# 20. 🟦 `apt`

For Debian/Ubuntu:

```yaml
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true
```

---

# 21. ⚙️ `service`

Generic service management.

```yaml
- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

This expresses:

```text
nginx
 ├── running
 └── enabled at boot
```

---

# 22. ⚙️ `systemd_service`

On systemd systems, you can use:

```yaml
- name: Restart application
  ansible.builtin.systemd_service:
    name: myapp
    state: restarted
```

You may also encounter:

```yaml
ansible.builtin.systemd
```

in existing codebases. The modern module name is `ansible.builtin.systemd_service`.

---

# 23. 👤 `user`

Create/manage users:

```yaml
- name: Create application user
  ansible.builtin.user:
    name: appuser
    state: present
    shell: /bin/bash
```

---

# 24. 👥 `group`

```yaml
- name: Create application group
  ansible.builtin.group:
    name: appgroup
    state: present
```

Then:

```yaml
- name: Create application user
  ansible.builtin.user:
    name: appuser
    group: appgroup
    state: present
```

---

# 25. 📝 `lineinfile`

Use this when you need to ensure or modify a **specific line** in a file.

Example:

```yaml
- name: Configure SSH setting
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PasswordAuthentication'
    line: 'PasswordAuthentication no'
```

Concept:

```text
Search matching line
       ↓
Replace with desired line
```

---

# 26. 🧱 `blockinfile`

Useful when managing a **block of configuration**.

```yaml
- name: Add application configuration
  ansible.builtin.blockinfile:
    path: /etc/myapp/app.conf
    block: |
      worker_threads=4
      max_connections=200
      timeout=30
```

Ansible can manage the block as a unit.

This is useful when:

> You own a particular section of a configuration file but not the entire file.

---

# 27. 🔄 `replace`

Useful when replacing text based on a regular expression.

```yaml
- name: Update timeout
  ansible.builtin.replace:
    path: /etc/myapp/app.conf
    regexp: '^timeout=.*$'
    replace: 'timeout=60'
```

Mental model:

```text
lineinfile
   ↓
specific line

blockinfile
   ↓
specific block

replace
   ↓
regex-based replacement
```

---

# 28. 📦 `unarchive`

Extract archives.

Example:

```yaml
- name: Extract application
  ansible.builtin.unarchive:
    src: app.tar.gz
    dest: /opt/myapp/
```

By default, the archive source may be handled from the controller.

If the archive is already on the remote node:

```yaml
- name: Extract remote archive
  ansible.builtin.unarchive:
    src: /tmp/app.tar.gz
    dest: /opt/myapp/
    remote_src: true
```

🔥 Notice that **`remote_src` also appears here**.

---

# 29. 🗜️ `archive`

The opposite direction/use case:

```text
archive
    ↓
Create archive
```

Example:

```yaml
- name: Archive logs
  ansible.builtin.archive:
    path:
      - /var/log/myapp/
    dest: /tmp/myapp-logs.tar.gz
    format: gz
```

---

# 30. 🔍 `stat`

`stat` lets you inspect a file/directory without changing it.

```yaml
- name: Check configuration
  ansible.builtin.stat:
    path: /etc/myapp/app.conf
  register: config_file
```

Then:

```yaml
- name: Show whether file exists
  ansible.builtin.debug:
    msg: "Configuration exists"
  when: config_file.stat.exists
```

Flow:

```text
stat
 │
 ▼
Inspect
 │
 ▼
register
 │
 ▼
conditional decision
```

This is extremely useful for writing safe automation.

---

# 31. 🔎 `find`

Search for files/directories.

```yaml
- name: Find old log files
  ansible.builtin.find:
    paths: /var/log/myapp
    patterns: "*.log"
    age: 30d
```

You can then use the registered result.

---

# 32. 📦 `slurp`

Reads a file from the managed node and returns its contents encoded in base64.

```yaml
- name: Read configuration
  ansible.builtin.slurp:
    src: /etc/myapp/app.conf
  register: config
```

Because the content is base64 encoded, you may decode it when needed.

This is useful when you need to bring remote file contents into Ansible logic.

---

# 33. 🐚 `command`

Use when you need to execute a command and no suitable module exists.

```yaml
- name: Check kernel version
  ansible.builtin.command:
    cmd: uname -r
```

---

# 34. 🐚 `shell`

Use when shell functionality is genuinely required.

```yaml
- name: Find nginx processes
  ansible.builtin.shell:
    cmd: ps -ef | grep nginx
```

But prefer:

```text
module > command > shell
```

when possible.

This is not an absolute hierarchy for every situation, but it's a very useful production rule.

---

# 35. 🧨 `raw`

Useful for low-level commands where normal module execution isn't available.

Especially useful for bootstrapping:

```yaml
- name: Install Python
  ansible.builtin.raw:
    cmd: dnf install -y python3
```

---

# 36. 📜 `script`

Run a script from the controller on the managed node.

```yaml
- name: Run bootstrap script
  ansible.builtin.script:
    cmd: ./bootstrap.sh
```

Use carefully.

If the logic can be represented cleanly with native Ansible modules, that's usually preferable.

---

# 37. 🔄 `reboot`

Ansible provides a dedicated reboot module:

```yaml
- name: Reboot server
  ansible.builtin.reboot:
```

It can handle waiting for the machine to go down and become reachable again.

This is much better than:

```yaml
- shell: reboot
```

because the dedicated module understands the reboot lifecycle.

---

# 38. 💽 `mount`

Manage filesystem mounts.

```yaml
- name: Mount application filesystem
  ansible.posix.mount:
    path: /data
    src: /dev/sdb1
    fstype: xfs
    state: mounted
```

This is an example where you may need a **non-builtin collection**.

Notice:

```text
ansible.posix.mount
```

rather than:

```text
ansible.builtin.mount
```

That's why FQCN matters.

---

# 39. 🧮 `set_fact`

`set_fact` creates variables during play execution.

```yaml
- name: Set application version
  ansible.builtin.set_fact:
    app_version: "2.5.0"
```

Then:

```yaml
- name: Show version
  ansible.builtin.debug:
    msg: "{{ app_version }}"
```

This is different from normal static variable definition because it creates/updates a variable dynamically during execution.

We'll study this deeply in the Variables chapter.

---

# 40. 🐛 `debug`

Used to inspect values.

```yaml
- name: Show application version
  ansible.builtin.debug:
    var: app_version
```

Or:

```yaml
- name: Display message
  ansible.builtin.debug:
    msg: "Deploying {{ app_name }}"
```

Extremely useful for troubleshooting.

---

# 41. ✅ `assert`

Used to enforce assumptions.

Example:

```yaml
- name: Verify supported OS
  ansible.builtin.assert:
    that:
      - ansible_os_family == "RedHat"
    fail_msg: "This playbook supports only RedHat-family systems"
```

Flow:

```text
Check condition
     │
 ┌───┴───┐
 ▼       ▼
TRUE    FALSE
 │       │
continue fail
```

This is very useful in production automation.

---

# 42. 🧠 Module Parameters — The Important Categories

Rather than memorizing parameters randomly, group them.

### File-related

```text
path
src
dest
mode
owner
group
state
recurse
```

### Execution control

```text
creates
removes
```

### Safety

```text
backup
validate
force
```

### Remote/local behavior

```text
remote_src
```

### Service

```text
state
enabled
daemon_reload
```

### Package

```text
name
state
update_cache
```

---

# 43. 🔥 `creates` and `removes`

These are **very important when using `command` or `shell`**.

### `creates`

```yaml
- name: Initialize application
  ansible.builtin.command:
    cmd: /opt/myapp/bin/init
    creates: /opt/myapp/.initialized
```

Meaning:

```text
Does .initialized exist?
       │
   ┌───┴───┐
   ▼       ▼
  YES      NO
   │       │
 skip    execute
```

### `removes`

```yaml
- name: Cleanup old application
  ansible.builtin.command:
    cmd: /opt/myapp/bin/cleanup
    removes: /opt/myapp/old-data
```

Conceptually:

```text
Does old-data exist?
       │
   ┌───┴───┐
   ▼       ▼
  YES      NO
   │       │
execute   skip
```

These help make command-based tasks more idempotent.

---

# 44. 🛡️ `check_mode`

Some modules support Ansible check mode.

Run:

```bash
ansible-playbook site.yml --check
```

This asks:

> What would happen without actually applying changes?

For example:

```text
Current:
nginx absent

Check mode:
would install nginx
```

But ⚠️ **not every module behaves identically in check mode**.

Some modules have full support, some partial support, and some operations cannot accurately predict changes.

This is an important production limitation.

---

# 45. 🔍 `diff`

You can use:

```bash
ansible-playbook site.yml --diff
```

This can show differences for modules that support diff output, especially useful for configuration management.

Often:

```bash
ansible-playbook site.yml --check --diff
```

is extremely useful before production changes.

Think:

```text
--check
   ↓
What would change?

--diff
   ↓
What content would change?
```

---

# 46. 🏭 Production Example — Safe Nginx Configuration

Let's combine several concepts.

```yaml
- name: Configure nginx
  hosts: webservers
  become: true

  tasks:

    - name: Deploy nginx configuration
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
        validate: '/usr/sbin/nginx -t -c %s'
        backup: true

      notify:
        - Restart nginx

  handlers:

    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

This is much better than:

```yaml
- shell: |
    cp nginx.conf /etc/nginx/
    nginx -t
    systemctl restart nginx
```

Why?

```text
template
   ↓
render configuration
   ↓
validate
   ↓
only if changed
   ↓
notify handler
   ↓
restart
```

That's production-quality Ansible thinking. 🔥

---

# 47. 🧠 Module Selection Decision Tree

When writing a task, think:

```text
What am I trying to do?
          │
          ▼
Is there a dedicated module?
      │            │
     YES           NO
      │             │
      ▼             ▼
Use module      Can command do it?
                     │
                ┌────┴────┐
               YES        NO
                │          │
                ▼          ▼
             command    shell/raw/
                       specialized approach
```

For configuration:

```text
Static file       → copy
Dynamic file      → template
One line          → lineinfile
Block             → blockinfile
Regex replacement → replace
```

---

# 48. 🎯 Production Module Decision Table

| Requirement                                 | Preferred module              |
| ------------------------------------------- | ----------------------------- |
| Create directory                            | `file`                        |
| Change permissions                          | `file`                        |
| Copy static file                            | `copy`                        |
| Dynamic config                              | `template`                    |
| Collect remote file                         | `fetch`                       |
| Download URL                                | `get_url`                     |
| HTTP/API call                               | `uri`                         |
| Install package                             | `package`                     |
| RHEL-specific package operation             | `dnf`                         |
| Debian-specific package operation           | `apt`                         |
| Manage service                              | `service` / `systemd_service` |
| Create user                                 | `user`                        |
| Create group                                | `group`                       |
| Modify one line                             | `lineinfile`                  |
| Manage config block                         | `blockinfile`                 |
| Regex replacement                           | `replace`                     |
| Extract archive                             | `unarchive`                   |
| Create archive                              | `archive`                     |
| Inspect file                                | `stat`                        |
| Find files                                  | `find`                        |
| Read remote file                            | `slurp`                       |
| Execute command                             | `command`                     |
| Execute shell syntax                        | `shell`                       |
| Bootstrap without normal Python module path | `raw`                         |
| Run local script remotely                   | `script`                      |
| Reboot                                      | `reboot`                      |
| Dynamic variable                            | `set_fact`                    |
| Print/debug values                          | `debug`                       |
| Validate assumptions                        | `assert`                      |

---

# 49. 🚨 Common Interview Traps

### Q: `copy` vs `template`?

```text
copy     → static content
template → Jinja2/dynamic content
```

---

### Q: `copy` vs `fetch`?

```text
copy  → Controller → Managed Node

fetch → Managed Node → Controller
```

---

### Q: `command` vs `shell`?

```text
command → no shell processing by default

shell   → executes through shell
```

---

### Q: `command` vs `raw`?

`command` uses Ansible's normal module execution mechanism.

`raw` executes directly through the connection and is particularly useful when the normal remote Python/module execution environment isn't available.

---

### Q: How do you make a command task more idempotent?

Use appropriate mechanisms such as:

```yaml
creates:
removes:
```

or use a dedicated state-aware module where possible.

---

### Q: What is `remote_src`?

> Tells applicable modules such as `copy`/`unarchive` that the source is already on the managed node rather than on the controller.

---

### Q: What is `validate`?

> It allows a configuration file to be validated before Ansible replaces the destination, where the module supports it.

---

### Q: What does `recurse` do?

> It applies applicable file attributes recursively to contents beneath a directory.

---

# 🧠 50. Your Module Mental Model

Don't memorize:

```text
copy = file copying
template = template
...
```

Instead think:

```text
             🎯 DESIRED OPERATION
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Files        Packages      Services
        │            │            │
      file         package      service
      copy         dnf          systemd
      template     apt
        │
        ▼
    Configuration
        │
 ┌──────┼────────┐
 ▼      ▼        ▼
line  block    replace
```

And when there is no dedicated module:

```text
Dedicated module?
      │
   YES│NO
      │
      ▼
   command
      │
      ▼
Need shell features?
      │
   YES│NO
      │
   shell   command
```

---

# 🏆 LevelUp Takeaway

The senior-level mindset is **not**:

> "I know 100 Ansible modules."

It's:

> **"I know how to select the right module, understand its state semantics, use its important parameters safely, maintain idempotency, validate changes, and handle production failure scenarios."**

That's what I want us to build throughout this course. 💪

---


# 🚀 Ansible Topic 6 — Playbooks, Plays, Tasks & Handlers

Now we move from **individual modules** to understanding how Ansible organizes real automation.

This is especially important for you because in your project you already write Ansible playbooks. For LevelUp, we need to be comfortable explaining **why the playbook is structured a certain way**, not just writing YAML.

---

# 1. 🧠 First: The Ansible Hierarchy

Keep this picture in your head:

```text
📘 PLAYBOOK
    │
    ├── 🎭 PLAY
    │     │
    │     ├── 📋 TASK
    │     │     ├── Module
    │     │     └── Parameters
    │     │
    │     ├── 📋 TASK
    │     │
    │     └── 📢 HANDLERS
    │
    └── 🎭 PLAY
          │
          ├── 📋 TASK
          └── 📢 HANDLERS
```

### In simple words:

**Playbook** → the entire automation file.

**Play** → defines **which hosts** you're automating and the overall execution context.

**Task** → one unit of work.

**Module** → performs the actual operation.

**Handler** → special task that runs when notified, usually after a change.

---

# 2. 📘 What is a Playbook?

A playbook is a YAML file containing one or more plays.

Example:

```yaml
---
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Save:

```text
site.yml
```

Run:

```bash
ansible-playbook -i inventory site.yml
```

---

# 3. 🎭 What is a Play?

A play connects:

```text
Hosts
  +
Tasks
  +
Execution configuration
```

Example:

```yaml
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Here:

```text
name:
    Configure web servers

hosts:
    webservers

become:
    true

tasks:
    ...
```

Think:

> **A play says "what should I do, and against whom?"**

---

# 4. 📋 What is a Task?

A task is one operation.

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

The task contains:

```text
Task
 │
 ├── name
 │
 └── module
       │
       ├── name
       └── state
```

A play normally contains multiple tasks:

```yaml
tasks:

  - name: Create user
    ansible.builtin.user:
      name: appuser

  - name: Create directory
    ansible.builtin.file:
      path: /opt/myapp
      state: directory

  - name: Deploy configuration
    ansible.builtin.template:
      src: app.conf.j2
      dest: /etc/myapp/app.conf
```

---

# 5. 🔄 How Ansible Executes a Play

Suppose:

```yaml
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
```

Inventory:

```text
web01
web02
web03
```

Conceptually:

```text
                PLAY
                 │
          hosts: webservers
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
     web01     web02     web03
       │         │         │
       └─────────┼─────────┘
                 │
              TASK 1
          Install nginx
                 │
                 ▼
              TASK 2
           Start nginx
```

Ansible normally processes tasks in the order they're written.

---

# 6. 🧠 Task Ordering

Example:

```yaml
tasks:

  - name: Task A
    ...

  - name: Task B
    ...

  - name: Task C
    ...
```

Conceptually:

```text
Task A
  ↓
Task B
  ↓
Task C
```

However, remember that Ansible's execution model is more nuanced when you introduce:

* `serial`
* `strategy`
* `async`
* `delegate_to`
* blocks
* handlers

We'll cover those separately.

---

# 7. 🔥 A Very Important Concept — Task vs Handler

Suppose you modify nginx configuration.

You **don't want to restart nginx every time the playbook runs**.

You only want to restart it **if the configuration actually changed**.

That's exactly where handlers are useful.

```yaml
- name: Deploy nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - Restart nginx

handlers:

  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

---

# 8. 📢 What is a Handler?

### Definition

> A handler is a special task that runs when it is notified by another task that reports a change.

Think:

```text
Normal task
     │
     │ changed?
     ▼
   YES
     │
     ▼
 notify handler
     │
     ▼
 handler executes
```

---

# 9. 🔄 Handler Flow

Suppose:

```yaml
tasks:

  - name: Deploy config
    ansible.builtin.template:
      src: app.conf.j2
      dest: /etc/myapp/app.conf
    notify:
      - Restart myapp

handlers:

  - name: Restart myapp
    ansible.builtin.service:
      name: myapp
      state: restarted
```

### First run

Configuration doesn't exist:

```text
template
   ↓
file created
   ↓
changed
   ↓
notify
   ↓
Restart myapp
```

### Second run

Configuration is already correct:

```text
template
   ↓
no change
   ↓
ok
   ↓
no notification
   ↓
NO restart
```

🔥 This is exactly why handlers are so useful.

---

# 10. 🧠 Handler = Event-Driven Task

You can think of a handler as:

```text
EVENT
  │
  ▼
"Configuration changed"
  │
  ▼
ACTION
  │
  ▼
"Restart service"
```

This is similar to event-driven automation.

---

# 11. 📢 `notify`

The task uses:

```yaml
notify:
  - Restart nginx
```

The handler:

```yaml
handlers:

  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

The names must match.

```text
notify:
"Restart nginx"
       │
       ▼
handler:
"Restart nginx"
```

---

# 12. 🚨 Handler Does NOT Run Immediately

This is a very important concept.

Suppose:

```yaml
tasks:

  - name: Change config
    ...
    notify:
      - Restart nginx

  - name: Do something else
    ...

  - name: Do another thing
    ...
```

The handler generally doesn't execute immediately after the notifying task.

Instead:

```text
Task 1
  ↓
changed
  ↓
handler notified
  ↓
Task 2
  ↓
Task 3
  ↓
...
  ↓
Handlers
```

By default, handlers are normally executed at the end of the relevant play.

---

# 13. 🏁 Default Handler Execution

Visualize:

```text
TASKS
│
├── Task 1 → changed → notify
├── Task 2
├── Task 3 → changed → notify
└── Task 4
       │
       ▼
  HANDLERS
       │
       ├── Handler A
       └── Handler B
```

This prevents repeated restarts during a single play.

---

# 14. 🔥 Why is this useful?

Imagine:

```yaml
tasks:

  - name: Deploy nginx config
    template:
      ...
    notify:
      - Restart nginx

  - name: Deploy nginx SSL config
    template:
      ...
    notify:
      - Restart nginx

  - name: Deploy nginx upstream config
    template:
      ...
    notify:
      - Restart nginx
```

Suppose all three files change.

Without handler behavior, you might restart nginx three times:

```text
change → restart
change → restart
change → restart
```

With handlers:

```text
config 1 → notify
config 2 → notify
config 3 → notify
             ↓
        Restart once
```

Much better. 🚀

---

# 15. 🧠 Multiple Notifications

Suppose:

```yaml
tasks:

  - name: Deploy config
    ansible.builtin.template:
      src: app.conf.j2
      dest: /etc/myapp/app.conf
    notify:
      - Restart myapp
      - Reload monitoring

handlers:

  - name: Restart myapp
    ansible.builtin.service:
      name: myapp
      state: restarted

  - name: Reload monitoring
    ansible.builtin.service:
      name: monitoring-agent
      state: reloaded
```

One task can notify multiple handlers.

---

# 16. 🔁 Handler Deduplication

Suppose:

```yaml
Task A
 notify: Restart nginx

Task B
 notify: Restart nginx

Task C
 notify: Restart nginx
```

Even if all three tasks change:

```text
Task A → notify
Task B → notify
Task C → notify

        ↓

Restart nginx
     ONE TIME
```

This is one of the biggest advantages of handlers.

---

# 17. ⚠️ Handler Ordering

Suppose:

```yaml
handlers:

  - name: Restart application
    ...

  - name: Reload nginx
    ...
```

If both are notified, handlers execute according to **handler ordering**, not simply according to the order in which tasks notified them.

The key principle:

> **Handlers are queued when notified and executed according to handler definition/order rules.**

So don't build automation that depends on:

```text
Task A notified Handler B
then Task C notified Handler A
therefore B must execute before A
```

If ordering matters, design the handlers appropriately.

---

# 18. 🔥 `listen`

There is another very useful production feature.

Instead of notifying a specific handler name, handlers can listen to a topic.

Example:

```yaml
tasks:

  - name: Deploy nginx config
    ansible.builtin.template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify:
      - restart web services

handlers:

  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
    listen:
      - restart web services
```

Now:

```text
notify:
restart web services
        │
        ▼
listen:
restart web services
        │
        ├── Restart nginx
        └── Restart another service
```

This is useful for grouping handlers around an event.

---

# 19. 🧱 Multiple Plays

A playbook can contain multiple plays.

Example:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present


- name: Configure databases
  hosts: databases
  become: true

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.package:
        name: postgresql
        state: present
```

Architecture:

```text
📘 site.yml
│
├── 🎭 Play 1
│    └── webservers
│         └── nginx
│
└── 🎭 Play 2
     └── databases
          └── PostgreSQL
```

This is very common.

---

# 20. 🏭 Real Production Example

Let's build something closer to your environment.

Suppose:

```text
webservers
    │
    ├── web01
    └── web02
```

You want:

1. Install application
2. Create user
3. Create directory
4. Deploy configuration
5. Restart service **only when configuration changes**

```yaml
---
- name: Deploy application
  hosts: webservers
  become: true

  tasks:

    - name: Create application user
      ansible.builtin.user:
        name: myapp
        state: present

    - name: Create application directory
      ansible.builtin.file:
        path: /opt/myapp
        state: directory
        owner: myapp
        group: myapp
        mode: '0755'

    - name: Deploy application configuration
      ansible.builtin.template:
        src: app.conf.j2
        dest: /opt/myapp/app.conf
        owner: myapp
        group: myapp
        mode: '0644'
      notify:
        - Restart myapp

    - name: Ensure application is running
      ansible.builtin.service:
        name: myapp
        state: started
        enabled: true

  handlers:

    - name: Restart myapp
      ansible.builtin.service:
        name: myapp
        state: restarted
```

---

# 21. 🔄 What Happens During First Run?

Assume:

```text
myapp user → absent
directory → absent
config → absent
service → stopped
```

Execution:

```text
Create user
    ↓
changed

Create directory
    ↓
changed

Deploy config
    ↓
changed
    ↓
notify Restart myapp

Ensure service started
    ↓
changed

Tasks complete
    ↓
Handler
    ↓
Restart myapp
```

---

# 22. 🔄 What Happens During Second Run?

Everything is already correct:

```text
Create user
    ↓
ok

Create directory
    ↓
ok

Deploy config
    ↓
ok
    ↓
NO notification

Ensure service
    ↓
ok

No handler execution
```

This gives you:

```text
♻️ Idempotent automation
```

---

# 23. 🚨 Important Handler Scenario

Suppose:

```text
Task 1 → configuration changed
Task 2 → something fails
```

What happens to the handler?

By default, if the play is aborted due to a later failure, a previously notified handler may not run.

This can create a dangerous situation:

```text
Config changed
     ↓
Restart notified
     ↓
Later task fails
     ↓
Play stops
     ↓
Handler may not run
```

This is where:

```yaml
force_handlers: true
```

can become important.

---

# 24. 🛡️ `force_handlers`

Example:

```yaml
---
- name: Deploy application
  hosts: webservers
  become: true
  force_handlers: true

  tasks:

    - name: Deploy config
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf
      notify:
        - Restart myapp

    - name: Perform some operation
      ansible.builtin.command:
        cmd: /opt/myapp/check
```

With:

```yaml
force_handlers: true
```

Ansible will attempt to run notified handlers even when a later task failure would otherwise prevent normal handler execution.

⚠️ But don't blindly enable it everywhere. If the handler itself is unsafe after a failure, forcing it can create another problem.

Production automation requires understanding **why** you're forcing handlers.

---

# 25. 🔥 `meta: flush_handlers`

Sometimes you need a handler to execute **before the play reaches its end**.

You can explicitly flush handlers:

```yaml
- name: Deploy configuration
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
  notify:
    - Restart myapp

- name: Flush handlers
  ansible.builtin.meta: flush_handlers

- name: Run operation requiring restarted service
  ansible.builtin.command:
    cmd: /opt/myapp/verify
```

Flow:

```text
Deploy config
     ↓
notify
     ↓
flush_handlers
     ↓
Restart myapp
     ↓
verification
```

This is an important advanced technique.

---

# 26. 🧠 When Would You Use `flush_handlers`?

Imagine:

```text
Update config
    ↓
Restart service
    ↓
Run validation against new configuration
```

You need the restart before the validation.

Without flushing:

```text
Update config
    ↓
Validation
    ↓
Handlers later
```

Potentially wrong.

With:

```yaml
- meta: flush_handlers
```

you get:

```text
Update config
    ↓
Handler executes
    ↓
Validation
```

---

# 27. 📊 `changed` and Handler Notification

This is important.

A handler is notified when its notifying task reports:

```text
changed: true
```

Example:

```yaml
- name: Deploy config
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
  notify:
    - Restart myapp
```

If template changes:

```text
changed → true
```

handler notified.

If no change:

```text
changed → false
```

handler isn't notified.

So:

```text
TASK
 │
 ├── changed = true
 │       ↓
 │    notify
 │
 └── changed = false
         ↓
       nothing
```

---

# 28. 🚨 A Common Mistake

Don't do this:

```yaml
- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

after every configuration task.

Instead:

```yaml
- name: Deploy config
  ansible.builtin.template:
    ...
  notify:
    - Restart nginx
```

Then:

```yaml
handlers:

  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

This gives you:

```text
Configuration changed?
       │
   ┌───┴───┐
  YES      NO
   │        │
restart    don't
```

---

# 29. 🧠 `state: restarted` vs `state: started`

Another common interview question.

```yaml
state: started
```

means:

> Ensure service is running.

If already running:

```text
ok
```

It doesn't need to restart.

---

```yaml
state: restarted
```

means:

> Restart the service.

So:

```text
started
  ↓
desired state

restarted
  ↓
explicit action
```

That's why `restarted` is commonly placed inside a handler.

---

# 30. 🏗️ Complete Production Architecture

```text
                         📘 PLAYBOOK
                              │
                              ▼
                           🎭 PLAY
                              │
                       hosts: webservers
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
             web01                        web02
                │                           │
                ├── Task 1                 ├── Task 1
                ├── Task 2                 ├── Task 2
                ├── Task 3 ──changed─────► │
                │       │                   │
                │       └── notify         │
                │             │             │
                └─────────────┼─────────────┘
                              ▼
                          📢 HANDLER
                              │
                       Restart service
```

---

# 31. 🎯 Interview Questions

### Q1. What is the difference between playbook, play, task and module?

> A playbook contains one or more plays. A play targets hosts and defines the execution context. A task is a unit of work within a play. A module performs the actual operation.

---

### Q2. What is a handler?

> A handler is a special task triggered by a notification from a changed task, commonly used for actions such as restarting or reloading services after configuration changes.

---

### Q3. When does a handler execute?

> By default, notified handlers execute after the normal tasks of the relevant play complete.

---

### Q4. If three tasks notify the same handler, how many times does it execute?

> Normally once during that handler flush, even if it was notified multiple times.

---

### Q5. Why use handlers instead of restarting a service directly?

> To avoid unnecessary restarts and to make service restarts conditional on actual configuration changes.

---

### Q6. What does `force_handlers` do?

> It causes notified handlers to be attempted even when a later task failure would otherwise prevent normal handler execution.

---

### Q7. What does `meta: flush_handlers` do?

> It forces pending notified handlers to execute at that point instead of waiting for the normal handler execution point.

---

# 🧠 32. The Mental Model You Need

Memorize this architecture:

```text
                    📘 PLAYBOOK
                         │
                         ▼
                      🎭 PLAY
                         │
                  hosts: webservers
                         │
                         ▼
                     📋 TASK
                         │
                         ▼
                     🧩 MODULE
                         │
                 ┌───────┴───────┐
                 │               │
              changed          unchanged
                 │               │
                 ▼               ▼
               notify          nothing
                 │
                 ▼
             📢 HANDLER
                 │
                 ▼
           Restart / Reload
```

And this:

```text
TASK
  │
  │ configuration changed?
  │
  ├── NO  → continue
  │
  └── YES → notify handler
                  │
                  ▼
             handler queue
                  │
                  ▼
            handler execution
```

---

# 🏆 Production-Level Takeaway

When you're writing an Ansible playbook, think:

> **"What is the desired state?"**

Then:

> **"Which module manages that state?"**

Then:

> **"What should happen if this state actually changes?"**

That leads naturally to:

```text
Desired state
     ↓
Module
     ↓
changed?
     ↓
notify
     ↓
Handler
```

This is one of the core patterns behind good production Ansible automation. 🔥

---


# 🚀 Ansible Topic 7 — Variables, Facts & Variable Precedence

This is one of the **most important Ansible topics for your LevelUp preparation**. 🔥

You already know:

```text
Inventory
   ↓
Playbook
   ↓
Tasks
   ↓
Modules
```

Now we need to make the automation **dynamic and reusable**.

Instead of hardcoding:

```yaml
port: 8080
```

we want:

```yaml
port: "{{ app_port }}"
```

And Ansible can obtain `app_port` from different places.

---

# 1. 🧠 What is a Variable?

A variable stores a value that can be reused during playbook execution.

Example:

```yaml
vars:
  app_name: myapp
  app_port: 8080
```

Then:

```yaml
- name: Display application
  ansible.builtin.debug:
    msg: "Application {{ app_name }} runs on port {{ app_port }}"
```

Output:

```text
Application myapp runs on port 8080
```

The syntax:

```jinja2
{{ variable_name }}
```

is Jinja2 expression syntax.

---

# 2. 🎯 Why Do We Need Variables?

Without variables:

```yaml
- name: Configure app
  ansible.builtin.template:
    src: app.conf.j2
    dest: /opt/myapp/app.conf
```

Template:

```text
port=8080
environment=production
```

Now the playbook is tightly coupled to production.

Instead:

```yaml
vars:
  app_port: 8080
  environment: production
```

Template:

```jinja2
port={{ app_port }}
environment={{ environment }}
```

Now staging can use:

```yaml
app_port: 8081
environment: staging
```

Same playbook. Different data. 💪

---

# 3. 📦 Variable Types

Ansible variables can hold different data types.

### String

```yaml
app_name: myapp
```

### Integer

```yaml
app_port: 8080
```

### Boolean

```yaml
enable_monitoring: true
```

### List

```yaml
packages:
  - nginx
  - curl
  - vim
```

### Dictionary

```yaml
database:
  host: db01
  port: 5432
  name: mydb
```

Access:

```jinja2
{{ database.host }}
```

or:

```jinja2
{{ database['host'] }}
```

---

# 4. 🎭 Variables Inside a Play

You can define variables using `vars`.

```yaml
---
- name: Configure application
  hosts: webservers

  vars:
    app_name: myapp
    app_port: 8080

  tasks:

    - name: Display application
      ansible.builtin.debug:
        msg: "{{ app_name }} runs on {{ app_port }}"
```

Scope:

```text
Play
│
├── vars
│    ├── app_name
│    └── app_port
│
└── tasks
     └── can use those variables
```

---

# 5. 📁 `vars_files`

Instead of putting variables directly into the playbook:

```yaml
vars:
  app_name: myapp
  app_port: 8080
```

you can put them in another file.

`vars/prod.yml`:

```yaml
app_name: myapp
app_port: 8080
environment: production
```

Playbook:

```yaml
- name: Deploy application
  hosts: webservers

  vars_files:
    - vars/prod.yml

  tasks:

    - name: Display environment
      ansible.builtin.debug:
        msg: "Deploying {{ app_name }} to {{ environment }}"
```

This helps separate:

```text
📘 Automation logic
        +
📦 Configuration data
```

---

# 6. 🗂️ `group_vars`

We already introduced this in inventory.

Example:

```text
group_vars/
└── webservers.yml
```

```yaml
app_port: 8080
app_name: myapp
```

If:

```ini
[webservers]
web01
web02
```

then both hosts get those variables.

```text
webservers
   │
   ├── web01 → app_port=8080
   │
   └── web02 → app_port=8080
```

---

# 7. 🖥️ `host_vars`

Specific host variables:

```text
host_vars/
└── web01.yml
```

```yaml
app_port: 9090
```

Now:

```text
web01 → 9090
web02 → 8080
```

assuming `webservers.yml` defines 8080.

This is useful when one host needs a specific override.

---

# 8. 🧠 `register`

This is **very important**.

`register` captures the result of a task into a variable.

Example:

```yaml
- name: Check disk usage
  ansible.builtin.command:
    cmd: df -h /
  register: disk_result
```

Now:

```text
disk_result
   │
   ├── stdout
   ├── stderr
   ├── rc
   ├── changed
   └── ...
```

You can inspect it:

```yaml
- name: Show disk information
  ansible.builtin.debug:
    var: disk_result
```

---

# 9. 🔍 Understanding `register`

Suppose:

```yaml
- name: Check application
  ansible.builtin.command:
    cmd: systemctl is-active myapp
  register: app_status
```

Conceptually:

```text
Command
   ↓
systemctl is-active myapp
   ↓
Result
   ↓
app_status
```

The registered variable contains information about **that task's execution result**.

This is different from a normal variable.

---

# 10. 📊 Typical Registered Result

A registered result can look roughly like:

```yaml
app_status:
  changed: false
  cmd:
    - systemctl
    - is-active
    - myapp
  rc: 0
  stdout: active
  stderr: ''
```

The exact fields vary by module.

Important fields you'll frequently encounter:

```text
stdout
stderr
rc
changed
failed
skipped
```

---

# 11. 🎯 Using `register` with `when`

Example:

```yaml
- name: Check application status
  ansible.builtin.command:
    cmd: systemctl is-active myapp
  register: app_status
  changed_when: false

- name: Show warning
  ansible.builtin.debug:
    msg: "Application is not running"
  when: app_status.stdout != "active"
```

Flow:

```text
Check service
      ↓
register result
      ↓
app_status.stdout
      ↓
condition
      ↓
warning if necessary
```

This pattern is extremely common in real automation.

---

# 12. 🧠 `changed_when`

Notice:

```yaml
changed_when: false
```

Why?

The command:

```bash
systemctl is-active myapp
```

is only checking status.

It doesn't actually change anything.

Without explicitly controlling the result, command/shell tasks can report change behavior that isn't semantically appropriate.

So:

```yaml
changed_when: false
```

means:

> This task should report `changed: false`.

This is an important technique for command-based checks.

---

# 13. 🚦 `failed_when`

You can also control failure behavior.

Example:

```yaml
- name: Check application
  ansible.builtin.command:
    cmd: /opt/myapp/healthcheck
  register: health_result
  failed_when: health_result.rc not in [0, 1]
```

Now you define exactly what counts as failure.

Conceptually:

```text
rc = 0 → success
rc = 1 → acceptable
rc = 2 → failure
```

This is useful when a command has meaningful non-zero return codes that don't all represent fatal failure.

---

# 14. 🧠 `set_fact`

`set_fact` creates variables dynamically during execution.

Example:

```yaml
- name: Set application version
  ansible.builtin.set_fact:
    app_version: "2.5.0"
```

Then:

```yaml
- name: Display version
  ansible.builtin.debug:
    msg: "{{ app_version }}"
```

Output:

```text
2.5.0
```

---

# 15. 🔥 `register` vs `set_fact`

This is a very common interview question.

### `register`

Captures:

> **The result of a task.**

```yaml
- command: hostname
  register: hostname_result
```

### `set_fact`

Creates:

> **A variable/value during execution.**

```yaml
- set_fact:
    server_name: "{{ hostname_result.stdout }}"
```

Flow:

```text
Task
 │
 ▼
register
 │
 ▼
hostname_result.stdout
 │
 ▼
set_fact
 │
 ▼
server_name
```

### Easy memory:

```text
register → capture result

set_fact → create/update variable
```

---

# 16. 🧩 Example: Register + Set Fact

```yaml
- name: Get hostname
  ansible.builtin.command:
    cmd: hostname
  register: hostname_result
  changed_when: false

- name: Store hostname
  ansible.builtin.set_fact:
    server_name: "{{ hostname_result.stdout }}"

- name: Display server
  ansible.builtin.debug:
    msg: "Server is {{ server_name }}"
```

---

# 17. 🔎 Ansible Facts

Now we reach another major concept.

When Ansible starts a play, it can gather information about the managed host.

For example:

```text
OS
CPU
Memory
IP addresses
Hostname
Interfaces
Disks
Architecture
Kernel
Python
etc.
```

These are called:

> **Ansible facts.**

---

# 18. 🔍 `gather_facts`

By default, plays generally gather facts.

Example:

```yaml
- name: Example
  hosts: all

  tasks:
    - name: Display OS family
      ansible.builtin.debug:
        var: ansible_facts.os_family
```

You might get:

```text
RedHat
```

or:

```text
Debian
```

---

# 19. 🧠 Fact-Gathering Flow

```text
Play starts
    │
    ▼
Gather facts
    │
    ▼
Managed Node
    │
    ├── OS
    ├── CPU
    ├── memory
    ├── network
    ├── hostname
    └── filesystem information
    │
    ▼
ansible_facts
```

---

# 20. 🔥 Useful Facts

Some common examples:

```jinja2
{{ ansible_facts.os_family }}
```

```jinja2
{{ ansible_facts.distribution }}
```

```jinja2
{{ ansible_facts.hostname }}
```

```jinja2
{{ ansible_facts.architecture }}
```

```jinja2
{{ ansible_facts.default_ipv4.address }}
```

```jinja2
{{ ansible_facts.memtotal_mb }}
```

The exact fact names can vary by Ansible version/platform, so use:

```bash
ansible <host> -m ansible.builtin.setup
```

to inspect available facts.

---

# 21. 🔍 `setup` Module

The module responsible for gathering facts is:

```text
ansible.builtin.setup
```

You can manually run:

```bash
ansible web01 -m ansible.builtin.setup
```

This produces a large amount of information.

You can filter:

```bash
ansible web01 \
  -m ansible.builtin.setup \
  -a "filter=ansible_distribution*"
```

This is useful for troubleshooting.

---

# 22. 🛑 Disable Fact Gathering

If you don't need facts:

```yaml
- name: Simple operation
  hosts: webservers
  gather_facts: false

  tasks:
    - name: Create directory
      ansible.builtin.file:
        path: /opt/myapp
        state: directory
```

Why would you do this?

Because fact gathering:

* consumes time
* consumes network traffic
* executes work on every host

For very large environments or simple tasks, disabling it can improve performance.

---

# 23. 🏭 Production Example — OS-Specific Package

Suppose you want:

```text
RHEL → dnf
Debian → apt
```

You can use facts:

```yaml
- name: Install package on RedHat
  ansible.builtin.dnf:
    name: nginx
    state: present
  when: ansible_facts.os_family == "RedHat"

- name: Install package on Debian
  ansible.builtin.apt:
    name: nginx
    state: present
  when: ansible_facts.os_family == "Debian"
```

Flow:

```text
             Managed Node
                  │
             gather facts
                  │
          ┌───────┴────────┐
          ▼                ▼
       RedHat            Debian
          │                │
          ▼                ▼
         dnf              apt
```

---

# 24. 🌍 `hostvars`

Now an important advanced variable.

`hostvars` lets you access variables/facts of **another host**.

Suppose:

```text
web01
web02
db01
```

You can access:

```jinja2
{{ hostvars['db01']['ansible_facts']['default_ipv4']['address'] }}
```

Conceptually:

```text
web01
 │
 │ hostvars
 ▼
db01
 │
 └── IP address
```

This becomes extremely useful in multi-tier deployments.

---

# 25. 👥 `groups`

`groups` provides information about inventory groups.

Suppose:

```ini
[webservers]
web01
web02

[databases]
db01
db02
```

You can access:

```jinja2
{{ groups['webservers'] }}
```

Conceptually:

```text
groups
 │
 ├── webservers → [web01, web02]
 └── databases  → [db01, db02]
```

---

# 26. 🔥 `groups` + `hostvars`

This combination is very powerful.

Suppose your application needs to know all database servers.

You can loop through:

```jinja2
{% for host in groups['databases'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }}
{% endfor %}
```

Conceptually:

```text
groups['databases']
       │
       ├── db01
       └── db02
            │
            ▼
       hostvars[host]
            │
            ▼
        IP address
```

This is commonly used for generating dynamic configuration.

---

# 27. 🏷️ `inventory_hostname`

We covered this earlier.

If:

```ini
web01 ansible_host=10.10.1.20
```

then:

```jinja2
{{ inventory_hostname }}
```

returns:

```text
web01
```

While:

```jinja2
{{ ansible_host }}
```

refers to the connection address.

---

# 28. 🧠 `inventory_hostname` vs `ansible_hostname`

These are easy to confuse.

### `inventory_hostname`

The name from your inventory:

```text
web01
```

### `ansible_hostname`

The actual hostname reported by the managed machine's facts.

For example:

```text
inventory_hostname = web01
ansible_hostname   = prod-web-server-01
```

They don't have to be identical.

---

# 29. 📦 Variable Sources

Now we reach the big question:

> **Where can Ansible variables come from?**

There are many sources.

Common ones:

```text
defaults
vars
inventory
group_vars
host_vars
facts
registered variables
set_fact
role variables
include variables
extra vars
```

That's why variable precedence exists.

---

# 30. 🥇 Variable Precedence — The Concept

Imagine the same variable:

```text
app_port
```

is defined in several places:

```text
group_vars:
8080

host_vars:
8081

play vars:
8082

extra vars:
9090
```

Which one wins?

Ansible needs a deterministic rule.

That's:

> **Variable precedence.**

---

# 31. 🧠 High-Level Precedence Mental Model

Don't memorize every precedence level yet.

Think:

```text
LOWER PRECEDENCE
       │
       ▼
Defaults
Inventory
group_vars
host_vars
Play vars
Task vars
set_fact / registered values
Extra vars
       │
       ▼
HIGHER PRECEDENCE
```

⚠️ This is a **conceptual model**, not the exact official 22-level precedence list.

The exact ordering has many categories and exceptions.

We'll have a dedicated deep-dive later.

---

# 32. 🔥 `extra_vars` — Very High Precedence

Suppose:

```yaml
vars:
  app_version: "1.0"
```

Run:

```bash
ansible-playbook site.yml \
  -e "app_version=2.0"
```

Then:

```text
Play variable
app_version = 1.0

        ↓ overridden by

Extra variable
app_version = 2.0
```

Output:

```text
2.0
```

This is why extra vars are often described as having **very high precedence**.

---

# 33. 🚨 Why Variable Precedence Matters

Imagine:

```text
Production:
app_port = 8080
```

But someone runs:

```bash
ansible-playbook site.yml \
  -e "app_port=9999"
```

Your playbook may now use:

```text
9999
```

This can cause unexpected behavior.

So in production:

> **Understand where important variables come from and control who can override them.**

---

# 34. 🔐 Don't Put Secrets in Normal Variables

Bad:

```yaml
db_password: SuperSecret123
```

inside:

```text
group_vars/production.yml
```

Instead use:

```text
Ansible Vault
```

We'll cover Vault later.

---

# 35. 🧠 `set_fact` and Precedence

`set_fact` creates variables during execution.

Example:

```yaml
- name: Calculate release directory
  ansible.builtin.set_fact:
    release_dir: "/opt/myapp/releases/{{ app_version }}"
```

Then:

```jinja2
{{ release_dir }}
```

This is useful for calculated/dynamic values.

---

# 36. ⚠️ `set_fact` Is Not the Same as `vars`

Compare:

### Static variable

```yaml
vars:
  app_version: "1.0"
```

### Runtime-generated variable

```yaml
- ansible.builtin.set_fact:
    release_dir: "/opt/myapp/releases/{{ app_version }}"
```

Think:

```text
vars
 ↓
defined before execution

set_fact
 ↓
created/updated during execution
```

---

# 37. 🧠 Facts vs Variables

Another common interview question.

### Variable

Something you define/provide:

```yaml
app_port: 8080
```

### Fact

Information Ansible discovers from the managed node:

```text
OS
CPU
Memory
IP
Hostname
```

So:

```text
Variable
   ↓
"I tell Ansible"

Fact
   ↓
"Ansible discovers"
```

---

# 38. 🎯 Production Example

Let's create a dynamic configuration.

Inventory:

```ini
[webservers]
web01
web02

[databases]
db01
```

Playbook:

```yaml
- name: Configure application
  hosts: webservers

  tasks:

    - name: Show database address
      ansible.builtin.debug:
        msg: "Database is {{ hostvars['db01']['ansible_facts']['default_ipv4']['address'] }}"
```

This lets a web server obtain information about another host.

---

# 39. 🧠 Better Production Pattern

Instead of hardcoding:

```jinja2
{{ hostvars['db01']... }}
```

you can use groups:

```jinja2
{% for db in groups['databases'] %}
{{ hostvars[db]['ansible_facts']['default_ipv4']['address'] }}
{% endfor %}
```

Now if:

```text
db01
db02
db03
```

are added:

```text
databases
├── db01
├── db02
└── db03
```

the configuration can dynamically discover them.

This is one of the powerful features of Ansible's inventory + variables model.

---

# 40. 🧮 Variable Example With Environment

A good production structure:

```text
inventory/
├── production/
│   ├── hosts.yml
│   └── group_vars/
│       └── webservers.yml
│
└── staging/
    ├── hosts.yml
    └── group_vars/
        └── webservers.yml
```

Production:

```yaml
app_port: 8080
environment: production
```

Staging:

```yaml
app_port: 8081
environment: staging
```

Same playbook:

```yaml
- name: Deploy application
  hosts: webservers

  tasks:
    - name: Show configuration
      ansible.builtin.debug:
        msg: "{{ environment }} uses port {{ app_port }}"
```

Result depends on inventory/environment.

🔥 This is how you make automation reusable.

---

# 41. 🚨 Common Mistakes

### ❌ Using facts without fact gathering

If:

```yaml
gather_facts: false
```

then don't assume:

```jinja2
{{ ansible_facts.os_family }}
```

will be available.

---

### ❌ Confusing `register` and `set_fact`

Remember:

```text
register
    ↓
capture task result

set_fact
    ↓
create/update variable
```

---

### ❌ Assuming `ansible_hostname` = inventory hostname

Not necessarily.

```text
inventory_hostname
    ↓
inventory name

ansible_hostname
    ↓
actual machine hostname discovered as a fact
```

---

### ❌ Hardcoding environment-specific values

Bad:

```yaml
app_port: 8080
```

everywhere.

Better:

```text
group_vars/
environment-specific variables
```

---

# 42. 🎤 LevelUp Interview Questions

### Q1. What are Ansible facts?

> Facts are system information gathered from managed nodes, such as operating system, hostname, IP addresses, memory and CPU information.

---

### Q2. Which module gathers facts?

```text
ansible.builtin.setup
```

---

### Q3. How do you disable fact gathering?

```yaml
gather_facts: false
```

---

### Q4. Difference between `register` and `set_fact`?

> `register` stores the result of a task, while `set_fact` creates or updates a variable during play execution.

---

### Q5. What is `hostvars`?

> A special variable that allows access to variables and facts associated with other inventory hosts.

---

### Q6. What is `groups`?

> A special variable containing the inventory group membership information.

---

### Q7. What is `inventory_hostname`?

> The host's name as represented in the inventory.

---

### Q8. What is variable precedence?

> The rules Ansible uses to determine which value wins when the same variable is defined in multiple places.

---

### Q9. Which has very high precedence?

```text
extra vars (-e)
```

---

### Q10. Why might you disable fact gathering?

> To reduce execution time and overhead when the playbook doesn't need system facts.

---

# 🏆 43. Final Mental Model

This is the picture I want you to remember:

```text
                         VARIABLES
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
   group_vars            host_vars             vars
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                             ▼
                     PLAYBOOK EXECUTION
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
           FACTS          REGISTER        SET_FACT
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                      TEMPLATE / TASK
                             │
                             ▼
                         RESULT
```

And remember these three:

```text
register
   ↓
"I want the result of this task."

set_fact
   ↓
"I want to create a variable dynamically."

ansible_facts
   ↓
"I want information discovered from the machine."
```

---

# 🎯 The A3-Level Mindset

When you write production Ansible, think:

```text
                    Environment
                         │
                         ▼
                  Inventory / Vars
                         │
                         ▼
                    Playbook
                         │
                  ┌──────┴──────┐
                  ▼             ▼
                Facts        Variables
                  │             │
                  └──────┬──────┘
                         ▼
                       Task
                         │
                         ▼
                       Module
                         │
                         ▼
                    Desired State
```

And when something doesn't have the expected value:

```text
"Where did this variable come from?"
              ↓
"Which source defined it?"
              ↓
"Which source has higher precedence?"
```

That mindset is much more valuable than memorizing isolated variable names. 💪

---



# 🚀 Ansible Topic 8.1 — Conditions, `when`, Loops & Iteration

This is another **core Ansible topic**. You'll use these concepts constantly in production playbooks.

The big idea is:

```text
                Ansible Task
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     Should I run it?      How many times?
          │                     │
        when                  loop
          │                     │
          ▼                     ▼
      Condition             Iteration
```

---

# 1. 🎯 What is a Conditional?

A conditional tells Ansible:

> **Run this task only when a particular condition is true.**

The primary mechanism is:

```yaml
when:
```

Example:

```yaml
- name: Install nginx on RedHat
  ansible.builtin.dnf:
    name: nginx
    state: present
  when: ansible_facts.os_family == "RedHat"
```

Conceptually:

```text
Managed node
     │
     ▼
Check OS
     │
 ┌───┴────┐
 ▼        ▼
RedHat   Debian
 │        │
 ▼        ▼
RUN      SKIP
```

---

# 2. 🧠 Basic `when`

Example:

```yaml
- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started
  when: nginx_enabled
```

If:

```yaml
nginx_enabled: true
```

→ task runs.

If:

```yaml
nginx_enabled: false
```

→ task is skipped.

---

# 3. ⚠️ Important: Don't Use `{{ }}` in `when`

Correct:

```yaml
when: app_port == 8080
```

Incorrect:

```yaml
when: "{{ app_port }} == 8080"
```

Why?

`when` already expects an expression.

So:

```text
Normal YAML/Jinja expression:
{{ variable }}

when:
variable == value
```

This is a very common interview/code-review point.

---

# 4. 🔢 Comparing Values

You can use:

```yaml
when: app_port == 8080
```

Not equal:

```yaml
when: app_port != 8080
```

Greater than:

```yaml
when: app_port > 1024
```

Less than:

```yaml
when: app_port < 65535
```

Greater than or equal:

```yaml
when: app_port >= 8080
```

---

# 5. 🧩 `and`

Multiple conditions must all be true:

```yaml
when:
  - ansible_facts.os_family == "RedHat"
  - app_environment == "production"
```

This means:

```text
RedHat
   AND
production
   ↓
RUN
```

You can also write:

```yaml
when: ansible_facts.os_family == "RedHat" and app_environment == "production"
```

The list format is generally easier to read.

---

# 6. 🔀 `or`

Run when at least one condition is true:

```yaml
when: >
  ansible_facts.os_family == "RedHat"
  or ansible_facts.os_family == "Debian"
```

Conceptually:

```text
RedHat ──┐
         ├── OR → RUN
Debian ──┘
```

---

# 7. 🚫 `not`

You can negate a condition:

```yaml
when: not maintenance_mode
```

Meaning:

```text
maintenance_mode = true
        ↓
      SKIP

maintenance_mode = false
        ↓
       RUN
```

---

# 8. 🧠 Multiple Conditions — Best Practice

Instead of:

```yaml
when: ansible_facts.os_family == "RedHat" and app_enabled == true and maintenance_mode == false
```

you can write:

```yaml
when:
  - ansible_facts.os_family == "RedHat"
  - app_enabled
  - not maintenance_mode
```

Much easier to read.

---

# 9. 🔍 `is defined`

Very important.

Sometimes a variable may not exist.

You can check:

```yaml
when: app_version is defined
```

Example:

```yaml
- name: Deploy application
  ansible.builtin.debug:
    msg: "Deploying {{ app_version }}"
  when: app_version is defined
```

---

# 10. 🚫 `is not defined`

```yaml
when: app_version is not defined
```

Useful for defaults/fallback behavior.

Example:

```yaml
- name: Set default version
  ansible.builtin.set_fact:
    app_version: "1.0"
  when: app_version is not defined
```

---

# 11. 🛡️ `default`

Another important technique:

```jinja2
{{ app_port | default(8080) }}
```

Meaning:

> If `app_port` isn't available, use `8080`.

Example:

```yaml
- name: Show port
  ansible.builtin.debug:
    msg: "Port: {{ app_port | default(8080) }}"
```

---

# 12. ⚠️ `default` and False Values

There is a subtle point.

```jinja2
{{ variable | default("value") }}
```

normally uses the default when the variable is **undefined**.

If you also want to treat false-y values as needing the default:

```jinja2
{{ variable | default("value", true) }}
```

This distinction can matter in production.

---

# 13. 🧪 Checking Strings

Example:

```yaml
when: environment == "production"
```

You can also check:

```yaml
when: environment in ["production", "staging"]
```

This is cleaner than:

```yaml
when: environment == "production" or environment == "staging"
```

---

# 14. 📋 Checking Lists

Suppose:

```yaml
supported_os:
  - RedHat
  - Debian
```

Then:

```yaml
when: ansible_facts.os_family in supported_os
```

---

# 15. 🔤 String Tests

You can use tests such as:

```yaml
when: app_name is string
```

or:

```yaml
when: app_port is number
```

Other useful Jinja tests include:

```text
defined
undefined
string
number
boolean
mapping
sequence
```

You don't need to memorize every test.

Know how to use `ansible-doc` and Jinja documentation when needed.

---

# 16. 🔄 What is a Loop?

A loop means:

> **Execute the same task repeatedly for different items.**

Instead of:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present

- name: Install curl
  ansible.builtin.package:
    name: curl
    state: present

- name: Install vim
  ansible.builtin.package:
    name: vim
    state: present
```

You can do:

```yaml
- name: Install packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim
```

---

# 17. 🧠 Visualizing a Loop

```text
                TASK
                 │
              loop list
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      nginx     curl     vim
        │        │        │
        ▼        ▼        ▼
      module   module   module
```

Each iteration gets a variable called:

```text
item
```

So:

```yaml
name: "{{ item }}"
```

becomes:

```text
nginx
curl
vim
```

---

# 18. 🔥 `item`

For:

```yaml
loop:
  - nginx
  - curl
  - vim
```

Ansible internally processes:

```text
item = nginx
item = curl
item = vim
```

Therefore:

```yaml
- name: Install package
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim
```

---

# 19. 📦 Loop Over Dictionaries

Suppose:

```yaml
users:
  - name: appuser
    shell: /bin/bash

  - name: deploy
    shell: /bin/bash
```

Then:

```yaml
- name: Create users
  ansible.builtin.user:
    name: "{{ item.name }}"
    shell: "{{ item.shell }}"
    state: present
  loop: "{{ users }}"
```

Conceptually:

```text
item
 ├── name
 └── shell
```

So:

```jinja2
{{ item.name }}
{{ item.shell }}
```

---

# 20. 🏭 Real Production Example

```yaml
vars:
  application_users:
    - name: appuser
      shell: /bin/bash
    - name: backupuser
      shell: /bin/bash
    - name: monitoruser
      shell: /sbin/nologin

tasks:

  - name: Create application users
    ansible.builtin.user:
      name: "{{ item.name }}"
      shell: "{{ item.shell }}"
      state: present
    loop: "{{ application_users }}"
```

This is much cleaner than writing three separate tasks.

---

# 21. 🏷️ `loop_control`

`loop_control` allows you to customize loop behavior.

Example:

```yaml
- name: Install packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim
  loop_control:
    label: "{{ item }}"
```

This makes output cleaner.

---

# 22. 🔢 `index_var`

You can track the iteration number.

```yaml
- name: Process packages
  ansible.builtin.debug:
    msg: "Processing {{ item }} at index {{ package_index }}"
  loop:
    - nginx
    - curl
    - vim
  loop_control:
    index_var: package_index
```

Conceptually:

```text
item       index
----------------
nginx        0
curl         1
vim          2
```

---

# 23. 🏷️ `label`

Suppose you're looping over complex objects:

```yaml
users:
  - name: appuser
    shell: /bin/bash
  - name: deploy
    shell: /bin/bash
```

Instead of showing the entire object in output:

```yaml
loop_control:
  label: "{{ item.name }}"
```

Output becomes easier to understand:

```text
TASK → appuser
TASK → deploy
```

---

# 24. 🔁 `loop` vs `with_items`

You may see old Ansible code:

```yaml
with_items:
  - nginx
  - curl
  - vim
```

Modern Ansible generally prefers:

```yaml
loop:
  - nginx
  - curl
  - vim
```

### Interview answer

> `with_*` looping constructs are older loop syntax. `loop` is the modern unified looping mechanism and is generally preferred for new playbooks.

You will still encounter `with_items`, `with_dict`, `with_fileglob`, etc. in legacy code.

---

# 25. 🧩 Looping Through Dictionaries

Suppose:

```yaml
users:
  appuser:
    shell: /bin/bash
  deploy:
    shell: /bin/bash
```

This is a dictionary, not a list.

Use:

```yaml
loop: "{{ users | dict2items }}"
```

Then:

```yaml
- name: Create users
  ansible.builtin.user:
    name: "{{ item.key }}"
    shell: "{{ item.value.shell }}"
    state: present
  loop: "{{ users | dict2items }}"
```

---

# 26. 🧠 `dict2items`

Transforms:

```yaml
users:
  appuser:
    shell: /bin/bash
  deploy:
    shell: /bin/bash
```

into approximately:

```yaml
- key: appuser
  value:
    shell: /bin/bash

- key: deploy
  value:
    shell: /bin/bash
```

Visual:

```text
Dictionary
    │
    ▼
dict2items
    │
    ▼
List of key/value objects
    │
    ▼
loop
```

---

# 27. 🔄 `items2dict`

The reverse operation.

```text
dict
 ↓
dict2items
 ↓
list
 ↓
items2dict
 ↓
dict
```

Useful when transforming structured data.

---

# 28. 🔁 Nested Loops

Suppose:

```yaml
applications:
  - name: app1
    ports:
      - 8080
      - 8081

  - name: app2
    ports:
      - 9090
      - 9091
```

You may need nested iteration.

However, don't immediately reach for nested loops. They can become difficult to read.

Often it's better to transform the data first.

This is an important production principle:

> **Good Ansible is often more about good data structure than complicated looping.**

---

# 29. 🔥 `subelements`

A common Ansible pattern for nested data is:

```yaml
users:
  - name: appuser
    authorized_keys:
      - key1
      - key2
```

You can use:

```yaml
loop: "{{ users | subelements('authorized_keys') }}"
```

Then:

```text
item.0 → parent object
item.1 → child element
```

Example:

```yaml
- name: Show keys
  ansible.builtin.debug:
    msg: "User={{ item.0.name }}, Key={{ item.1 }}"
  loop: "{{ users | subelements('authorized_keys') }}"
```

This is useful for structured nested data.

---

# 30. 🧠 Loop + Register

This is **very important**.

Suppose:

```yaml
- name: Check services
  ansible.builtin.command:
    cmd: "systemctl is-active {{ item }}"
  loop:
    - nginx
    - sshd
    - chronyd
  register: service_checks
  changed_when: false
```

Now:

```text
service_checks
      │
      └── results
           ├── nginx result
           ├── sshd result
           └── chronyd result
```

The results are generally available under:

```jinja2
{{ service_checks.results }}
```

---

# 31. 🔍 Loop Registered Results

You can process them:

```yaml
- name: Show service results
  ansible.builtin.debug:
    msg: "{{ item.item }} = {{ item.stdout }}"
  loop: "{{ service_checks.results }}"
```

Notice:

```text
item.item
```

This can look confusing initially.

Why?

Because:

```text
Outer loop
   ↓
service_checks.results
   ↓
each result
   ↓
item
```

And each registered result remembers the original loop item.

So:

```text
item.item
```

means:

> The original item used for that iteration.

---

# 32. 🔁 `until` — Retry Until Condition

This is a very important production feature.

Suppose your application takes time to become healthy.

You can do:

```yaml
- name: Wait for application
  ansible.builtin.uri:
    url: http://localhost:8080/health
    status_code: 200
  register: health_check
  until: health_check.status == 200
  retries: 10
  delay: 5
```

Flow:

```text
Health check
     │
     ▼
status == 200?
   │       │
  YES      NO
   │       │
   ▼       ▼
success   wait
             │
             ▼
          retry
```

---

# 33. 🔥 `until` + `retries` + `delay`

These three work together.

```yaml
until: condition
retries: 10
delay: 5
```

Meaning approximately:

> Retry the task until the condition succeeds, up to the configured retry attempts, waiting between attempts.

This is extremely useful for:

* application startup
* service availability
* API readiness
* cluster node readiness
* database readiness
* cloud resources becoming available

---

# 34. 🏭 Production Example — Wait for Service

```yaml
- name: Wait for application health
  ansible.builtin.uri:
    url: http://localhost:8080/health
    return_content: false
    status_code: 200
  register: health
  until: health.status == 200
  retries: 12
  delay: 5
```

Potential behavior:

```text
0 sec  → check → 503
5 sec  → check → 503
10 sec → check → 503
15 sec → check → 200
                ↓
              success
```

---

# 35. 🧠 `when` + Loop

You can combine them.

```yaml
- name: Install production packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim
  when: environment == "production"
```

The task is skipped entirely if:

```text
environment != production
```

---

# 36. 🔥 Conditional Inside a Loop

You can also use `when` based on the current item.

```yaml
- name: Install selected packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim
  when: item != "vim"
```

Result:

```text
nginx → RUN
curl  → RUN
vim   → SKIP
```

---

# 37. 🧠 `when` With Registered Results

Example:

```yaml
- name: Check configuration
  ansible.builtin.stat:
    path: /etc/myapp/app.conf
  register: config_file

- name: Create configuration
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
  when: not config_file.stat.exists
```

Flow:

```text
stat
 ↓
exists?
 ↓
YES → skip creation
NO  → create
```

---

# 38. 🛡️ Safe Variable Handling

Consider:

```yaml
when: app_config.enabled
```

What if:

```text
app_config
```

doesn't exist?

You can get an error.

A safer pattern can be:

```yaml
when:
  - app_config is defined
  - app_config.enabled
```

Or use appropriate defaults when designing the data model.

---

# 39. 🎯 `is defined` + `default`

These solve different problems.

### Check whether variable exists

```yaml
when: app_port is defined
```

### Provide a fallback value

```jinja2
{{ app_port | default(8080) }}
```

Think:

```text
is defined
    ↓
Decision

default
    ↓
Fallback value
```

---

# 40. 🚨 `failed_when` vs `when`

Don't confuse them.

### `when`

Controls:

> **Should the task execute?**

```yaml
when: environment == "production"
```

### `failed_when`

Controls:

> **Should the result be considered a failure?**

```yaml
failed_when: result.rc not in [0, 1]
```

---

# 41. 🚨 `changed_when` vs `when`

### `when`

```text
Should task run?
```

### `changed_when`

```text
Should Ansible report changed?
```

Example:

```yaml
- name: Check service
  ansible.builtin.command:
    cmd: systemctl is-active nginx
  register: result
  changed_when: false
```

The task still executes.

It simply reports:

```text
changed = false
```

---

# 42. 🧠 Four Control Concepts

Memorize this:

```text
when
 ↓
Should I run?

loop
 ↓
How many items?

changed_when
 ↓
Should this count as changed?

failed_when
 ↓
Should this count as failed?
```

🔥 This is a very useful mental model.

---

# 43. 🏭 Real Production Scenario

Imagine your AlloyDB automation has:

```text
3 nodes
```

You want to execute a validation command against each node.

```yaml
- name: Validate PostgreSQL service
  ansible.builtin.command:
    cmd: systemctl is-active postgresql
  register: postgres_status
  changed_when: false
```

Then:

```yaml
- name: Fail if PostgreSQL is not running
  ansible.builtin.assert:
    that:
      - postgres_status.stdout == "active"
    fail_msg: "PostgreSQL is not running"
```

Now add a loop for multiple services:

```yaml
- name: Validate required services
  ansible.builtin.command:
    cmd: "systemctl is-active {{ item }}"
  loop:
    - postgresql
    - patroni
    - etcd
    - haproxy
  register: service_status
  changed_when: false
```

This is much closer to real production automation.

---

# 44. ⚠️ Common Loop Mistake

Don't write:

```yaml
loop:
  "{{ packages }}"
```

The normal form is:

```yaml
loop: "{{ packages }}"
```

For example:

```yaml
vars:
  packages:
    - nginx
    - curl
```

Then:

```yaml
- name: Install packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop: "{{ packages }}"
```

---

# 45. 🔥 Loop Variable Collision

Suppose you're calling another task/role that itself uses `item`.

Nested loops can cause collisions.

Use:

```yaml
loop_control:
  loop_var: package_name
```

Example:

```yaml
- name: Install packages
  ansible.builtin.package:
    name: "{{ package_name }}"
    state: present
  loop:
    - nginx
    - curl
  loop_control:
    loop_var: package_name
```

Now instead of:

```text
item
```

you use:

```text
package_name
```

This is particularly useful when loops become complex.

---

# 46. 🧠 `loop_var`

Example:

```yaml
loop_control:
  loop_var: service_name
```

Then:

```yaml
name: "{{ service_name }}"
```

instead of:

```yaml
name: "{{ item }}"
```

Mental model:

```text
Default:

loop item → item


Custom:

loop item → service_name
```

This improves readability and prevents nested-loop conflicts.

---

# 47. 🔢 Looping Over Numbers

You can use:

```yaml
loop: "{{ range(1, 6) | list }}"
```

This produces:

```text
1
2
3
4
5
```

Example:

```yaml
- name: Show numbers
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop: "{{ range(1, 6) | list }}"
```

---

# 48. 📁 Looping Over Files

You may encounter:

```yaml
with_fileglob:
```

in older code.

Modern patterns can use appropriate lookup/query mechanisms, for example:

```yaml
loop: "{{ query('ansible.builtin.fileglob', 'files/*.conf') }}"
```

This returns matching controller-side file paths.

This is an advanced area we'll revisit when we cover **lookups and advanced Jinja/data manipulation**.

---

# 49. 🧠 Don't Overuse Loops

Bad:

```yaml
loop:
  - nginx
  - curl
  - vim
```

when a module can natively accept a list.

For example, `package` can often take a list:

```yaml
- name: Install packages
  ansible.builtin.package:
    name:
      - nginx
      - curl
      - vim
    state: present
```

This may be better than looping.

### Production principle:

> **Use a module's native list support when it provides the desired behavior; don't loop unnecessarily.**

---

# 50. 🔥 This Is an Important Optimization

Instead of:

```yaml
- name: Install packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim
```

you can often do:

```yaml
- name: Install packages
  ansible.builtin.package:
    name:
      - nginx
      - curl
      - vim
    state: present
```

Advantages:

```text
Less YAML
Less task iteration
Potentially more efficient
Clearer intent
```

---

# 51. 🎤 LevelUp Interview Questions

### Q1. What is `when`?

> `when` conditionally controls whether a task, block, play, or other supported construct executes.

---

### Q2. Should you use `{{ }}` inside `when`?

No.

Correct:

```yaml
when: app_port == 8080
```

---

### Q3. Difference between `when` and `failed_when`?

```text
when
 → whether task executes

failed_when
 → whether task result is considered failed
```

---

### Q4. What is `loop`?

> A mechanism for repeatedly executing a task for each item in a collection.

---

### Q5. What variable represents the current loop item?

```text
item
```

unless overridden with:

```yaml
loop_control:
  loop_var: something
```

---

### Q6. How do you retry a task until it succeeds?

```yaml
until: condition
retries: 10
delay: 5
```

---

### Q7. What is `register` with loops?

> The registered result generally contains the individual iteration results under its `results` attribute.

---

### Q8. Why use `loop_control.loop_var`?

> To give the loop variable a meaningful name and avoid collisions, particularly with nested loops.

---

### Q9. `loop` vs `with_items`?

> `loop` is the modern unified looping syntax; `with_*` constructs are older syntax still found in legacy playbooks.

---

# 🏆 52. The Mental Model

This is the picture I want you to remember:

```text
                         TASK
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
            when          loop      execution
              │            │
          Should run?   items?
              │            │
              ▼            ▼
             YES      ┌────┼────┐
                      ▼    ▼    ▼
                     A    B    C
```

And:

```text
when
 │
 └── Should I execute?


loop
 │
 └── How many times?


register
 │
 └── What was the result?


changed_when
 │
 └── Did it count as changed?


failed_when
 │
 └── Did it count as failed?


until
 │
 └── Should I retry?
```

---

# 🎯 A3-Level Production Pattern

A strong Ansible task often combines several of these:

```yaml
- name: Wait for application health
  ansible.builtin.uri:
    url: "http://{{ inventory_hostname }}:8080/health"
    status_code: 200
  register: health
  until: health.status == 200
  retries: 12
  delay: 5
  changed_when: false
  when:
    - app_enabled
    - environment in ["staging", "production"]
```

Read it as:

```text
Is application enabled?
       │
       ▼
Is environment staging/production?
       │
       ▼
Run health check
       │
       ▼
Don't report it as changed
       │
       ▼
If not healthy → retry
       │
       ▼
Up to 12 attempts
       │
       ▼
5 seconds between attempts
```

That's the kind of **production reasoning** we're aiming for. 💪

---


# 🚨 Ansible Topic 8.2 — Error Handling & Failure Management

This is a **very important production topic**.

In a real environment, Ansible isn't just about:

> "What happens when everything succeeds?"

A senior engineer must understand:

> **"What happens when something fails halfway through?"**

For example:

```text
web01 → success
web02 → success
web03 → configuration failed
web04 → what happens?
```

And:

```text
Configuration changed
       ↓
Service restart notified
       ↓
Later task fails
       ↓
Will handler run?
       ↓
Will other hosts continue?
       ↓
Can we recover?
```

That's what this chapter is about. 🔥

---

# 1. 🧠 Ansible Failure Model

By default, if a task fails on a host:

```text
Host
 │
 ├── Task 1 → OK
 ├── Task 2 → OK
 ├── Task 3 → FAILED ❌
 │
 └── remaining tasks for THIS HOST
       generally stop
```

But importantly:

> A failure on one host does **not automatically mean every other host stops**.

Example:

```text
web01 → Task 3 FAILED ❌
web02 → Task 3 SUCCESS ✅
web03 → Task 3 SUCCESS ✅
```

The other hosts can continue according to Ansible's execution strategy and failure controls.

---

# 2. 🎯 Basic Failure Example

```yaml
- name: Test failure
  hosts: webservers

  tasks:

    - name: Task 1
      ansible.builtin.debug:
        msg: "Task 1"

    - name: Task 2
      ansible.builtin.command:
        cmd: /bin/false

    - name: Task 3
      ansible.builtin.debug:
        msg: "Task 3"
```

`/bin/false` returns a non-zero exit code.

Result:

```text
Task 1 → OK
Task 2 → FAILED
Task 3 → SKIPPED/NOT EXECUTED for that host
```

---

# 3. 🛑 `ignore_errors`

You can tell Ansible:

> If this task fails, don't stop execution for this host.

Example:

```yaml
- name: Run optional command
  ansible.builtin.command:
    cmd: /opt/myapp/optional-check
  ignore_errors: true
```

Flow:

```text
Task
 │
 ▼
FAILED ❌
 │
 ▼
ignore_errors: true
 │
 ▼
Continue
```

---

# 4. ⚠️ When Should You Use `ignore_errors`?

Good example:

```text
Optional diagnostic command
```

Bad example:

```yaml
- name: Install database
  ansible.builtin.package:
    name: postgresql
    state: present
  ignore_errors: true
```

Why is that dangerous?

You might continue with:

```text
PostgreSQL installation failed
        ↓
Continue anyway
        ↓
Configure PostgreSQL
        ↓
Start PostgreSQL
```

That can create confusing failures later.

### Production rule

> **Don't use `ignore_errors` simply to make a playbook "green." Use it only when failure is genuinely acceptable and you have a reason to continue.**

---

# 5. 🔥 `failed_when`

We've seen this before, but now let's understand it as an error-handling mechanism.

Suppose:

```yaml
- name: Check application
  ansible.builtin.command:
    cmd: /opt/myapp/check
  register: result
```

Maybe the command returns:

```text
rc=1
```

but in your application:

```text
rc=1 → warning
rc=2 → actual failure
```

You can define:

```yaml
failed_when: result.rc == 2
```

Now Ansible uses **your definition of failure**.

---

# 6. 🧠 Example — Custom Failure Condition

```yaml
- name: Check replication
  ansible.builtin.command:
    cmd: /opt/check-replication
  register: replication

  failed_when: replication.rc > 1
```

Meaning:

```text
rc = 0 → success
rc = 1 → acceptable
rc > 1 → failure
```

This is useful when external commands have special exit-code semantics.

---

# 7. 🧩 Multiple `failed_when` Conditions

You can write:

```yaml
failed_when:
  - result.rc != 0
  - "'ERROR' in result.stdout"
```

⚠️ Be careful here.

A list of `failed_when` conditions is effectively evaluated as an **AND** relationship.

So this means:

```text
rc != 0
   AND
ERROR appears
```

If you need OR logic, explicitly write:

```yaml
failed_when: >
  result.rc != 0 or
  'ERROR' in result.stdout
```

This distinction is worth remembering.

---

# 8. 🟢 `changed_when`

Error handling isn't only about failure.

Sometimes you need to tell Ansible:

> "This task executed, but it should not be reported as changed."

Example:

```yaml
- name: Check service status
  ansible.builtin.command:
    cmd: systemctl is-active nginx
  register: nginx_status
  changed_when: false
```

Result:

```text
TASK
 │
 ▼
command executes
 │
 ▼
changed = false
```

---

# 9. 🧠 Why `changed_when` Matters

Handlers depend on `changed`.

Suppose:

```yaml
- name: Run health check
  ansible.builtin.command:
    cmd: /opt/myapp/healthcheck
  notify:
    - Restart myapp
```

If Ansible reports the command as changed:

```text
health check
    ↓
changed
    ↓
Restart myapp 😱
```

That's probably wrong.

Instead:

```yaml
changed_when: false
```

means:

```text
health check
    ↓
changed = false
    ↓
no handler notification
```

---

# 10. 🚦 Four Important Controls

You should now clearly distinguish:

| Mechanism       | Question it answers                     |
| --------------- | --------------------------------------- |
| `when`          | Should the task run?                    |
| `changed_when`  | Should it report changed?               |
| `failed_when`   | Should it report failure?               |
| `ignore_errors` | If it fails, should execution continue? |

Mental model:

```text
                TASK
                  │
                  ▼
               when?
                  │
             ┌────┴────┐
            NO         YES
            │           │
          SKIP          ▼
                      Execute
                         │
                    ┌────┴────┐
                    ▼         ▼
               changed?     failed?
                    │         │
                    ▼         ▼
             changed_when  failed_when
                              │
                              ▼
                       ignore_errors?
```

---

# 11. 🧱 `block`

Now we get into a **very important production feature**.

You can group tasks using:

```yaml
block:
```

Example:

```yaml
- name: Application deployment
  hosts: webservers

  tasks:

    - name: Deployment operations
      block:

        - name: Stop application
          ansible.builtin.service:
            name: myapp
            state: stopped

        - name: Deploy application
          ansible.builtin.copy:
            src: myapp.jar
            dest: /opt/myapp/myapp.jar

        - name: Start application
          ansible.builtin.service:
            name: myapp
            state: started
```

Visual:

```text
Deployment block
│
├── Stop application
├── Deploy application
└── Start application
```

---

# 12. 🎯 Why Use `block`?

`block` is useful for:

* grouping related tasks
* applying common conditions
* applying common privilege escalation
* error handling with `rescue`
* cleanup with `always`

The most important reason we'll use it here:

```text
block
  +
rescue
  +
always
```

This gives us structured error handling.

---

# 13. 🛟 `rescue`

Think of:

```text
try / catch
```

from programming languages.

Ansible equivalent:

```yaml
block:
  ...
rescue:
  ...
```

Example:

```yaml
- name: Deployment
  block:

    - name: Deploy application
      ansible.builtin.copy:
        src: myapp.jar
        dest: /opt/myapp/myapp.jar

    - name: Start application
      ansible.builtin.service:
        name: myapp
        state: started

  rescue:

    - name: Rollback application
      ansible.builtin.copy:
        src: myapp.jar.backup
        dest: /opt/myapp/myapp.jar
```

Flow:

```text
             BLOCK
               │
        ┌──────┴──────┐
        ▼             ▼
     SUCCESS        FAILURE
        │             │
        ▼             ▼
    Continue        rescue
                      │
                      ▼
                   rollback
```

---

# 14. 🧠 Important `rescue` Behavior

`rescue` runs when a task in the `block` **fails**.

Example:

```yaml
block:

  Task A → success
  Task B → FAILED ❌
  Task C → not executed

rescue:

  Rollback
```

So:

```text
Task A
 ↓
Task B ❌
 ↓
rescue
```

Task C isn't executed because the block encountered a failure.

---

# 15. 🧹 `always`

`always` runs regardless of whether the block succeeds or fails, subject to Ansible's execution/failure semantics.

Example:

```yaml
- name: Deployment
  block:

    - name: Deploy application
      ...

  rescue:

    - name: Rollback
      ...

  always:

    - name: Cleanup temporary files
      ansible.builtin.file:
        path: /tmp/myapp
        state: absent
```

Flow:

```text
             BLOCK
               │
        ┌──────┴──────┐
        ▼             ▼
     SUCCESS        FAILURE
        │             │
        │           rescue
        │             │
        └──────┬──────┘
               ▼
            always
```

Think:

```text
block  = try
rescue = catch
always = finally
```

That's an excellent interview analogy.

---

# 16. 🔥 Production Deployment Example

Let's build something realistic.

```yaml
- name: Deploy application
  hosts: webservers
  become: true

  tasks:

    - name: Application deployment
      block:

        - name: Stop application
          ansible.builtin.service:
            name: myapp
            state: stopped

        - name: Deploy new binary
          ansible.builtin.copy:
            src: myapp.jar
            dest: /opt/myapp/myapp.jar
            backup: true

        - name: Start application
          ansible.builtin.service:
            name: myapp
            state: started

        - name: Validate application
          ansible.builtin.uri:
            url: http://localhost:8080/health
            status_code: 200

      rescue:

        - name: Report deployment failure
          ansible.builtin.debug:
            msg: "Deployment failed. Starting rollback."

        - name: Restore backup
          ansible.builtin.copy:
            src: /opt/myapp/myapp.jar.backup
            dest: /opt/myapp/myapp.jar
            remote_src: true

        - name: Start application after rollback
          ansible.builtin.service:
            name: myapp
            state: started

      always:

        - name: Remove temporary deployment directory
          ansible.builtin.file:
            path: /tmp/myapp-deploy
            state: absent
```

This is much closer to **senior-level production automation**.

---

# 17. 🚨 Important: `rescue` Doesn't Mean "Ignore the Failure"

This is a common misunderstanding.

Suppose:

```yaml
block:
  task fails

rescue:
  rollback
```

The failure triggers the rescue process.

The goal is:

```text
failure
 ↓
recovery
```

not:

```text
failure
 ↓
pretend everything is fine
```

After successful rescue handling, Ansible can continue with subsequent tasks.

---

# 18. 🔥 `rescue` Can Recover Execution

Example:

```yaml
- name: Main operation
  block:

    - name: Operation that may fail
      ansible.builtin.command:
        cmd: /opt/myapp/deploy

  rescue:

    - name: Perform fallback
      ansible.builtin.command:
        cmd: /opt/myapp/rollback

- name: Continue deployment workflow
  ansible.builtin.debug:
    msg: "Continuing..."
```

If rescue succeeds, Ansible can continue.

Conceptually:

```text
deploy
  ↓
FAIL
  ↓
rollback
  ↓
recovered
  ↓
continue
```

---

# 19. ⚠️ What If `rescue` Itself Fails?

Then you have another failure.

For example:

```text
Deployment
   ↓
FAIL ❌
   ↓
Rollback
   ↓
FAIL ❌
```

Now the recovery itself failed.

You should design rescue operations to be:

* simple
* reliable
* minimally dependent on the failed operation

---

# 20. 🧠 `any_errors_fatal`

Now let's talk about **multi-host failure control**.

Suppose:

```text
web01
web02
web03
web04
```

and:

```text
web02 → failure
```

Normally, the failure is primarily scoped to that host.

But sometimes you want:

> If any host encounters a fatal task failure, stop the play across all hosts.

Use:

```yaml
any_errors_fatal: true
```

Example:

```yaml
- name: Critical deployment
  hosts: webservers
  any_errors_fatal: true

  tasks:
    - name: Critical operation
      ...
```

Visual:

```text
web01 → success
web02 → FAILED ❌
web03 → STOP
web04 → STOP
```

---

# 21. 🏭 When Is `any_errors_fatal` Useful?

Use it when continuing on other hosts could make the environment inconsistent or dangerous.

Example:

```text
Database schema migration
```

Imagine:

```text
DB1 → migration failed
DB2 → continue migration
DB3 → continue migration
```

That could be disastrous.

A better strategy might be:

```text
one critical failure
      ↓
stop broader operation
```

---

# 22. 📊 `max_fail_percentage`

Another multi-host control.

Example:

```yaml
max_fail_percentage: 30
```

This tells Ansible to stop the play when failures exceed the configured threshold.

Suppose:

```text
10 servers
```

and:

```text
3 servers fail
```

That's:

```text
30%
```

The behavior is based on exceeding the configured threshold, so don't interpret the value as simply "stop exactly when equal to 30%" without considering Ansible's failure counting semantics.

### Important interview point:

> `max_fail_percentage` is useful for limiting how many hosts can fail before Ansible aborts the play.

---

# 23. 🧠 `any_errors_fatal` vs `max_fail_percentage`

| Feature                  | Meaning                                              |
| ------------------------ | ---------------------------------------------------- |
| `any_errors_fatal: true` | A fatal failure causes the play to stop across hosts |
| `max_fail_percentage`    | Allow failures up to a threshold before stopping     |

Mental model:

```text
any_errors_fatal
      ↓
ONE critical failure
      ↓
STOP


max_fail_percentage
      ↓
SOME failures acceptable
      ↓
STOP when threshold exceeded
```

---

# 24. 🔥 Why This Matters for `serial`

We'll later study:

```yaml
serial:
```

for rolling deployments.

Imagine:

```text
10 servers
serial: 2
```

You update two at a time.

Now imagine one server fails.

You need to decide:

```text
Should the rollout continue?
```

That's where concepts like:

```text
max_fail_percentage
any_errors_fatal
```

become very important.

So this topic is directly preparing you for your **production `serial` interview question**. 🔥

---

# 25. 🛡️ `force_handlers`

We covered this in the previous topic, but now let's connect it to failure handling.

Suppose:

```yaml
force_handlers: true
```

and:

```text
Task 1 → configuration changed
Task 2 → failure
```

Normally:

```text
Task 1
 ↓
notify handler
 ↓
Task 2 fails
 ↓
handler may not execute
```

With:

```yaml
force_handlers: true
```

Ansible will attempt to run the notified handler despite the later failure.

---

# 26. 🧠 `block` Can Have Conditions

You don't have to put `when` on every task.

Example:

```yaml
- name: Production configuration
  block:

    - name: Configure nginx
      ...

    - name: Configure TLS
      ...

  when: environment == "production"
```

Now the condition applies to the block's tasks.

Visual:

```text
Production block
       │
       ▼
environment == production?
       │
   ┌───┴────┐
  YES       NO
   │         │
   ▼         ▼
run tasks   skip
```

This is cleaner than repeating:

```yaml
when: environment == "production"
```

on every task.

---

# 27. 🧱 `block` + `become`

You can also apply common privilege escalation.

```yaml
- name: System configuration
  block:

    - name: Modify config
      ansible.builtin.template:
        ...

    - name: Restart service
      ansible.builtin.service:
        ...

  become: true
```

Both tasks inherit the block-level setting.

This is useful for reducing duplication.

---

# 28. 🔥 Production Error Handling Pattern

A very useful pattern is:

```yaml
- name: Deployment
  block:

    - name: Pre-check
      ...

    - name: Deploy
      ...

    - name: Validate
      ...

  rescue:

    - name: Rollback
      ...

    - name: Validate rollback
      ...

  always:

    - name: Cleanup
      ...
```

Visual:

```text
              DEPLOYMENT
                   │
                   ▼
              ┌─────────┐
              │  BLOCK  │
              └────┬────┘
                   │
             ┌─────┴─────┐
             ▼           ▼
          SUCCESS      FAILURE
             │           │
             │        RESCUE
             │           │
             │        ROLLBACK
             │           │
             └─────┬─────┘
                   ▼
                ALWAYS
                   │
                CLEANUP
```

---

# 29. 🧠 `ignore_errors` vs `rescue`

This is an important interview comparison.

### `ignore_errors`

```yaml
ignore_errors: true
```

means:

> The failure is acceptable; continue.

### `rescue`

```yaml
block:
rescue:
```

means:

> The failure occurred; execute recovery logic.

So:

```text
ignore_errors
     ↓
"Continue despite failure."


rescue
     ↓
"Failure occurred; recover."
```

Production-wise, `rescue` is usually more intentional for actual recovery workflows.

---

# 30. 🧠 `rescue` vs `always`

### `rescue`

Runs for failure recovery.

```text
failure → rescue
```

### `always`

Runs regardless of block success/failure.

```text
success ──┐
          ├──→ always
failure ──┘
```

---

# 31. 🚨 A Very Important Edge Case

`rescue` responds to **task failures**, but not every possible reason for a host to become unreachable.

For example:

```text
SSH connection lost
```

is fundamentally different from:

```text
A module executed and returned failed
```

An unreachable host is generally handled as an **unreachable** condition, not a normal task failure.

This distinction becomes important when designing recovery logic.

---

# 32. 🛑 `ignore_unreachable`

You can separately control unreachable hosts.

Example:

```yaml
- name: Test connectivity
  ansible.builtin.ping:
  ignore_unreachable: true
```

This is different from:

```yaml
ignore_errors: true
```

Remember:

```text
Task failure
    ↓
ignore_errors


Host unreachable
    ↓
ignore_unreachable
```

---

# 33. 🔄 `meta: clear_host_errors`

Suppose a host becomes unreachable and you want to make it available for subsequent tasks after the underlying issue has been corrected.

You can use:

```yaml
- name: Clear host errors
  ansible.builtin.meta: clear_host_errors
```

Conceptually:

```text
Host becomes unreachable
        ↓
Ansible marks host failed/unreachable
        ↓
Underlying issue fixed
        ↓
clear_host_errors
        ↓
host can participate again
```

This is an advanced recovery mechanism.

---

# 34. 🧠 `meta` Actions

`meta` isn't a normal module in the same sense as `file` or `copy`.

It controls Ansible's execution behavior.

Examples:

```yaml
ansible.builtin.meta: flush_handlers
```

and:

```yaml
ansible.builtin.meta: clear_host_errors
```

You'll encounter other meta actions as well.

For LevelUp, understand the important ones rather than memorizing every action.

---

# 35. 🏭 Real Multi-Host Failure Scenario

Imagine:

```text
10 production servers
```

You deploy:

```text
Application v2
```

using:

```yaml
serial: 2
```

Batch 1:

```text
web01 → success
web02 → success
```

Batch 2:

```text
web03 → success
web04 → FAILED ❌
```

Now what?

You might configure:

```yaml
max_fail_percentage: 25
```

or:

```yaml
any_errors_fatal: true
```

depending on your risk model.

This is why error handling can't be studied separately from deployment strategy.

---

# 36. 🔥 Production Example With `serial`

We'll only preview it now.

```yaml
- name: Rolling deployment
  hosts: webservers
  serial: 2
  max_fail_percentage: 25

  tasks:

    - name: Deploy application
      ...

    - name: Validate application
      ...
```

Conceptually:

```text
10 servers

Batch 1
web01
web02
 ↓
success

Batch 2
web03
web04
 ↓
web04 fails

Failure threshold evaluated
 ↓
continue or stop
```

We'll go deep into this later.

---

# 37. 🎤 LevelUp Interview Questions

### Q1. What happens when a task fails?

> By default, Ansible marks that host as failed and does not execute subsequent tasks for that host in the normal flow. Other hosts may continue depending on the execution strategy and failure controls.

---

### Q2. What is `ignore_errors`?

> It allows a task failure to be ignored so execution can continue for that host.

---

### Q3. `ignore_errors` vs `ignore_unreachable`?

> `ignore_errors` handles task failures; `ignore_unreachable` handles unreachable-host conditions.

---

### Q4. What is `block/rescue/always`?

> `block` groups tasks, `rescue` provides failure-recovery logic, and `always` provides cleanup/finalization logic that should execute regardless of block success or failure.

---

### Q5. What is the equivalent of try/catch/finally?

```text
try     → block
catch   → rescue
finally → always
```

Excellent interview analogy.

---

### Q6. What is `failed_when`?

> It lets you define a custom condition under which a task should be considered failed.

---

### Q7. What is `changed_when`?

> It lets you control whether Ansible considers a task changed.

---

### Q8. What is `any_errors_fatal`?

> It causes a fatal task failure to abort the play across hosts rather than allowing the operation to continue normally on other hosts.

---

### Q9. What is `max_fail_percentage`?

> It defines the failure threshold beyond which Ansible aborts the play.

---

### Q10. Why use `rescue` instead of `ignore_errors`?

> `rescue` lets you implement deliberate recovery or rollback logic rather than simply continuing after an error.

---

# 38. 🧠 The Four Failure Scenarios You Must Understand

### Scenario 1 — Normal failure

```text
Task fails
   ↓
Host stops normal task execution
```

### Scenario 2 — Ignore failure

```text
Task fails
   ↓
ignore_errors
   ↓
continue
```

### Scenario 3 — Recover

```text
Task fails
   ↓
rescue
   ↓
rollback/recovery
   ↓
continue if recovery succeeds
```

### Scenario 4 — Critical multi-host failure

```text
Host failure
   ↓
any_errors_fatal / failure threshold
   ↓
stop broader operation
```

---

# 39. 🏆 Final Mental Model

Keep this diagram:

```text
                         TASK
                           │
                           ▼
                       Execute
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
               SUCCESS            FAILURE
                  │                 │
                  │          ┌──────┼──────┐
                  │          │      │      │
                  │          ▼      ▼      ▼
                  │       ignore   rescue  fatal
                  │       errors    block   rules
                  │          │      │      │
                  │          ▼      ▼      ▼
                  │       continue recover stop
                  │
                  └──────────┬──────────┘
                             ▼
                           always
```

And remember:

```text
when
 ↓
Should it run?

changed_when
 ↓
Is it changed?

failed_when
 ↓
Is it failed?

ignore_errors
 ↓
Continue despite failure?

block/rescue
 ↓
Recover from failure?

always
 ↓
Cleanup regardless?

any_errors_fatal
 ↓
Should one failure stop everyone?

max_fail_percentage
 ↓
How many failures can we tolerate?
```

---



# 🧱 Ansible Topic 9 — Blocks, Rescue & Always: Advanced Production Patterns

We already introduced these in Topic 9. Now let's go deeper because **`block/rescue/always` is very useful for production-grade Ansible**.

The simplest mental model is:

```text
block  → normal operation
rescue → recovery
always → cleanup/finalization
```

Think of it like:

```text
try     → block
catch   → rescue
finally → always
```

---

# 1. 🧠 Basic Structure

```yaml
- name: Application deployment
  block:

    - name: Task 1
      ...

    - name: Task 2
      ...

  rescue:

    - name: Recovery task
      ...

  always:

    - name: Cleanup task
      ...
```

Flow:

```text
              ┌─────────────┐
              │    BLOCK    │
              └──────┬──────┘
                     │
                Task execution
                     │
             ┌───────┴───────┐
             ▼               ▼
          SUCCESS          FAILURE
             │               │
             │               ▼
             │            RESCUE
             │               │
             └───────┬───────┘
                     ▼
                  ALWAYS
```

---

# 2. 🎯 Why `block` Exists

Without a block:

```yaml
- name: Task A
  ...

- name: Task B
  ...

- name: Task C
  ...
```

If all three need the same:

```yaml
when: environment == "production"
```

you might repeat it:

```yaml
- name: Task A
  ...
  when: environment == "production"

- name: Task B
  ...
  when: environment == "production"

- name: Task C
  ...
  when: environment == "production"
```

Instead:

```yaml
- name: Production operations
  block:

    - name: Task A
      ...

    - name: Task B
      ...

    - name: Task C
      ...

  when: environment == "production"
```

Cleaner. 👍

---

# 3. 🧩 Block-Level `when`

Example:

```yaml
- name: Configure production server
  block:

    - name: Deploy configuration
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf

    - name: Restart application
      ansible.builtin.service:
        name: myapp
        state: restarted

  when: environment == "production"
```

The condition applies to the tasks inside the block.

---

# 4. 🔐 Block-Level `become`

You can also do:

```yaml
- name: System configuration
  block:

    - name: Modify configuration
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf

    - name: Restart service
      ansible.builtin.service:
        name: myapp
        state: restarted

  become: true
```

Instead of:

```yaml
become: true
```

on every task.

---

# 5. 🏷️ Block-Level Tags

Similarly:

```yaml
- name: Application configuration
  block:

    - name: Deploy config
      ...

    - name: Restart application
      ...

  tags:
    - application
```

Later:

```bash
ansible-playbook site.yml --tags application
```

runs the tagged block's tasks.

We'll study Tags properly in the next chapter.

---

# 6. 🛟 Rescue — The Important Part

Suppose:

```yaml
- name: Deployment
  block:

    - name: Deploy application
      ansible.builtin.copy:
        src: myapp.jar
        dest: /opt/myapp/myapp.jar

    - name: Validate application
      ansible.builtin.uri:
        url: http://localhost:8080/health
        status_code: 200

  rescue:

    - name: Rollback
      ...
```

If validation fails:

```text
Deploy
  ↓
Validation ❌
  ↓
RESCUE
  ↓
Rollback
```

---

# 7. 🔥 Important: Rescue Is Triggered by Failure

Suppose:

```yaml
block:

  - Task A
  - Task B
  - Task C
```

If:

```text
Task A → success
Task B → failure
Task C → not executed
```

Then:

```yaml
rescue:
```

starts.

So:

```text
Task A
 ↓
Task B ❌
 ↓
Task C ✋
 ↓
rescue
```

---

# 8. 🧠 What Counts as a Failure?

A normal module failure can trigger `rescue`.

For example:

```yaml
- name: Validate application
  ansible.builtin.uri:
    url: http://localhost:8080/health
    status_code: 200
```

If the endpoint returns:

```text
500
```

the task fails.

Therefore:

```text
500
 ↓
task failure
 ↓
rescue
```

---

# 9. 🚨 `ignore_errors` Changes the Story

Consider:

```yaml
- name: Validate application
  ansible.builtin.uri:
    url: http://localhost:8080/health
    status_code: 200
  ignore_errors: true
```

The task fails, but Ansible is instructed to ignore that failure.

Therefore, don't design a block expecting `rescue` to handle a failure that you've deliberately ignored.

Mental model:

```text
failure
  │
  ├── ignored → continue
  │
  └── normal failure → rescue
```

---

# 10. 🔄 Rescue Is for Recovery, Not Just Logging

Weak:

```yaml
rescue:

  - name: Print error
    ansible.builtin.debug:
      msg: "Deployment failed"
```

Better:

```yaml
rescue:

  - name: Restore previous configuration
    ...

  - name: Restart application
    ...

  - name: Validate rollback
    ...
```

A good rescue block should answer:

> **What should the system look like after this failure?**

---

# 11. 🏭 Production Example — Configuration Deployment

Suppose the current configuration is:

```text
app.conf
```

You want to deploy:

```text
app.conf.new
```

A production workflow might be:

```text
Backup current config
        ↓
Deploy new config
        ↓
Validate configuration
        │
     ┌──┴──┐
    OK    FAIL
     │      │
     │   Restore backup
     │      │
     │   Restart
     │      │
     │   Validate
     │      │
     └──┬───┘
        ▼
      Cleanup
```

Ansible:

```yaml
- name: Deploy application configuration
  block:

    - name: Backup current configuration
      ansible.builtin.copy:
        src: /etc/myapp/app.conf
        dest: /etc/myapp/app.conf.backup
        remote_src: true
      when: config_file.stat.exists

    - name: Deploy new configuration
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf

    - name: Validate configuration
      ansible.builtin.command:
        cmd: /usr/bin/myapp --check-config

    - name: Restart application
      ansible.builtin.service:
        name: myapp
        state: restarted

  rescue:

    - name: Restore configuration
      ansible.builtin.copy:
        src: /etc/myapp/app.conf.backup
        dest: /etc/myapp/app.conf
        remote_src: true

    - name: Restart application after rollback
      ansible.builtin.service:
        name: myapp
        state: restarted

  always:

    - name: Remove backup
      ansible.builtin.file:
        path: /etc/myapp/app.conf.backup
        state: absent
```

⚠️ In a real playbook, you'd make the backup/restore logic more defensive. This is the architectural pattern.

---

# 12. 🧠 Important Problem With Rollbacks

Don't blindly write:

```yaml
rescue:
  - copy:
      src: backup
      dest: config
```

What if:

```text
Backup itself failed?
```

or:

```text
Backup doesn't exist?
```

Your rescue can fail.

Therefore production rescue code should consider:

```text
Was backup created?
Does backup exist?
Can it be restored?
Did restoration succeed?
Can the service start?
```

That's senior-level thinking.

---

# 13. 🛡️ Validation Before Restart

A much better pattern is:

```text
Deploy configuration
       ↓
Validate configuration
       │
   ┌───┴────┐
  valid   invalid
    │         │
    ▼         ▼
 restart    rescue
```

Instead of:

```text
Deploy
 ↓
Restart
 ↓
Oops, configuration invalid
```

This is a major production principle:

> **Validate configuration before applying a disruptive action whenever the application provides a validation mechanism.**

---

# 14. 🧪 `assert` for Validation

Ansible provides:

```yaml
ansible.builtin.assert
```

Example:

```yaml
- name: Validate application version
  ansible.builtin.assert:
    that:
      - app_version is defined
      - app_version != ""
    fail_msg: "Application version must be defined"
```

If the condition fails:

```text
assertion failed
     ↓
block failure
     ↓
rescue
```

---

# 15. 🔥 Production Pattern — Pre-check

```yaml
- name: Deployment
  block:

    - name: Verify required variable
      ansible.builtin.assert:
        that:
          - app_version is defined
        fail_msg: "app_version is required"

    - name: Verify disk space
      ...

    - name: Deploy application
      ...

    - name: Validate application
      ...

  rescue:

    - name: Rollback
      ...
```

This gives:

```text
Pre-check
   ↓
Deploy
   ↓
Validate
   ↓
Success
```

rather than discovering obvious problems halfway through.

---

# 16. 🧱 Nested Blocks

Blocks can be nested.

Example:

```yaml
- name: Main deployment
  block:

    - name: Configuration
      block:

        - name: Deploy config
          ...

        - name: Validate config
          ...

      rescue:

        - name: Restore config
          ...

    - name: Application
      block:

        - name: Deploy binary
          ...

        - name: Start service
          ...

      rescue:

        - name: Rollback binary
          ...

  rescue:

    - name: Global rollback
      ...
```

Architecture:

```text
Main Block
│
├── Configuration Block
│   ├── Tasks
│   └── Rescue
│
├── Application Block
│   ├── Tasks
│   └── Rescue
│
└── Main Rescue
```

---

# 17. ⚠️ Don't Overuse Nested Blocks

Although technically possible:

```text
block
 └── block
      └── block
           └── block
```

this can become extremely difficult to maintain.

For production:

> Use blocks to represent meaningful operational boundaries.

Good:

```text
Configuration
Application deployment
Validation
Rollback
```

Bad:

```text
Task group 1
  └── Task group 2
       └── Task group 3
```

with no meaningful separation.

---

# 18. 🧹 `always` — Cleanup

`always` is particularly useful for:

* temporary files
* temporary directories
* mounted resources
* debug artifacts
* cleanup
* status reporting

Example:

```yaml
always:

  - name: Remove temporary directory
    ansible.builtin.file:
      path: /tmp/myapp
      state: absent
```

Think:

```text
Whatever happens:
       ↓
CLEAN UP
```

---

# 19. 🧠 `always` Is Not "Ignore Errors"

This is important.

`always` means:

> Execute this section regardless of whether the block succeeded or entered rescue.

It does **not** mean:

> Ignore errors in this section.

If an `always` task itself fails, that can still cause a failure.

---

# 20. 🔥 `always` for Temporary Mounts

Imagine:

```text
Mount filesystem
      ↓
Perform operation
      ↓
Unmount filesystem
```

Even if the operation fails, you want:

```text
Unmount
```

So:

```yaml
- name: Temporary filesystem operation
  block:

    - name: Mount filesystem
      ...

    - name: Process data
      ...

  always:

    - name: Unmount filesystem
      ...
```

Very useful production pattern.

---

# 21. 🧠 `rescue` Can Have Its Own `when`

Example:

```yaml
rescue:

  - name: Rollback only in production
    ansible.builtin.command:
      cmd: /opt/myapp/rollback
    when: environment == "production"
```

The rescue section is itself composed of tasks, so those tasks can have normal task controls.

---

# 22. 🔄 `rescue` and `register`

You can capture recovery results too.

```yaml
rescue:

  - name: Attempt rollback
    ansible.builtin.command:
      cmd: /opt/myapp/rollback
    register: rollback_result
    changed_when: rollback_result.rc == 0
```

Then:

```yaml
  - name: Report rollback result
    ansible.builtin.debug:
      var: rollback_result
```

This lets you distinguish:

```text
Original operation failed
        ↓
Rollback succeeded
```

from:

```text
Original operation failed
        ↓
Rollback ALSO failed ❌
```

---

# 23. 🚨 Recovery Failure Is More Serious

Consider:

```text
Deployment
   ↓
FAILED ❌
   ↓
Rollback
   ↓
FAILED ❌
```

Now your environment could be in an uncertain state.

A senior engineer should make this visible.

For example:

```yaml
rescue:

  - name: Rollback
    ...

  - name: Verify rollback
    ...

  - name: Fail explicitly if rollback failed
    ansible.builtin.assert:
      that:
        - rollback_success
      fail_msg: "Deployment and rollback both failed"
```

The exact implementation depends on how you perform the rollback.

---

# 24. 🏆 Idempotency + Rescue

A very important connection to what you wanted for your LevelUp preparation:

> **Rescue operations should also be idempotent whenever possible.**

For example, prefer:

```yaml
ansible.builtin.file:
  path: /tmp/myapp
  state: absent
```

over blindly running:

```bash
rm -rf /tmp/myapp
```

Similarly:

```yaml
ansible.builtin.service:
  name: myapp
  state: started
```

is generally preferable to:

```bash
systemctl start myapp
```

Why?

Because the recovery itself should be predictable if executed again.

---

# 25. 🧠 Block + Handler

You can combine blocks and handlers.

```yaml
- name: Deploy application
  block:

    - name: Deploy config
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf
      notify:
        - Restart myapp

    - name: Validate config
      ansible.builtin.command:
        cmd: /opt/myapp/check-config

  rescue:

    - name: Rollback config
      ...

  always:

    - name: Cleanup
      ...
```

Remember:

```text
notify
   ↓
handler queue
```

The handler does not necessarily run immediately.

If you need the handler before leaving the block:

```yaml
- name: Flush handlers
  ansible.builtin.meta: flush_handlers
```

---

# 26. 🔥 Deployment Pattern With Handler Flush

Example:

```yaml
- name: Deploy configuration
  block:

    - name: Deploy config
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf
      notify:
        - Restart myapp

    - name: Execute pending handlers
      ansible.builtin.meta: flush_handlers

    - name: Validate application
      ansible.builtin.uri:
        url: http://localhost:8080/health
        status_code: 200

  rescue:

    - name: Rollback configuration
      ...
```

Flow:

```text
Deploy config
     ↓
changed
     ↓
notify
     ↓
flush_handlers
     ↓
Restart
     ↓
Health check
     ↓
FAIL?
  │
  └──→ rescue
```

This is a powerful pattern.

---

# 27. 🚨 Be Careful With Rollback + Handler

Suppose:

```text
New config
   ↓
notify restart
   ↓
restart
   ↓
validation fails
   ↓
rescue restores old config
```

Now the old configuration is restored.

But:

> **You may need another restart/reload for the old configuration to actually take effect.**

So rollback might need:

```text
restore config
    ↓
restart/reload
    ↓
validate rollback
```

This is a very common real-world consideration.

---

# 28. 🏭 Complete Production Architecture

Here's a good mental model for a deployment:

```text
                  🚀 DEPLOYMENT
                       │
                       ▼
                 ┌───────────┐
                 │   BLOCK   │
                 └─────┬─────┘
                       │
                       ▼
                  PRE-CHECKS
                       │
                       ▼
                  BACKUP STATE
                       │
                       ▼
                 DEPLOY NEW STATE
                       │
                       ▼
                 VALIDATE CONFIG
                       │
                  ┌────┴────┐
                 OK         FAIL
                  │           │
                  ▼           ▼
              RESTART       RESCUE
                  │           │
                  ▼       RESTORE STATE
             HEALTH CHECK     │
                  │        RESTART
                  │           │
                  │       VALIDATE
                  │           │
                  └─────┬─────┘
                        ▼
                     ALWAYS
                        │
                        ▼
                     CLEANUP
```

This is the kind of architecture you should be able to explain in an interview.

---

# 29. 🎤 Interview Question — Why use `block/rescue/always`?

A strong answer:

> "I use `block/rescue/always` to structure production workflows around failure recovery. I put the main operation in `block`, rollback or recovery actions in `rescue`, and cleanup or finalization actions in `always`. This makes failure handling explicit rather than simply ignoring errors."

That's a good A3-level answer. 💪

---

# 30. 🎤 Interview Question — Can `rescue` be used for rollback?

Yes.

But be careful:

> `rescue` itself doesn't automatically rollback anything.

You must implement the rollback.

```text
rescue
   ↓
Your recovery logic
```

For example:

```yaml
rescue:

  - name: Restore previous version
    ...
```

---

# 31. 🎤 Interview Question — What happens if the rescue succeeds?

The block's failure can be considered handled, allowing Ansible to continue with subsequent execution.

But don't interpret this as:

> "The original task succeeded."

The original operation **did fail**.

The important distinction is:

```text
Original operation → failed
Recovery → succeeded
Workflow → can continue
```

---

# 32. 🎤 Interview Question — What happens if `always` fails?

The `always` section doesn't magically suppress failures.

If an `always` task fails, that failure can affect play execution.

Therefore:

> Cleanup tasks should also be designed carefully and, where appropriate, made resilient/idempotent.

---

# 33. 🧠 `block` vs Role

Don't confuse these.

### Block

Execution/error-handling structure:

```text
block
rescue
always
```

### Role

Reusable automation structure:

```text
tasks/
handlers/
templates/
files/
vars/
defaults/
```

We'll study roles later.

Think:

```text
block
 ↓
"How should these tasks execute/recover?"


role
 ↓
"How should I package and reuse this automation?"
```

---

# 34. 🧠 `block` vs `include_tasks`

Another distinction:

```text
block
 ↓
group existing tasks


include_tasks
 ↓
load another task file
```

Example:

```yaml
- name: Include deployment tasks
  ansible.builtin.include_tasks:
    file: deploy.yml
```

We'll cover task inclusion later.

---

# 35. 🔥 Production Best Practices

### ✅ 1. Validate before destructive operations

```text
check
 ↓
change
 ↓
validate
```

---

### ✅ 2. Make rollback explicit

Don't assume Ansible will magically restore state.

---

### ✅ 3. Keep rescue logic simple

The recovery path should have fewer dependencies than the primary path.

---

### ✅ 4. Make cleanup idempotent

```yaml
state: absent
```

is usually better than blindly executing deletion commands.

---

### ✅ 5. Don't use `ignore_errors` as a replacement for recovery

Bad:

```yaml
ignore_errors: true
```

Good:

```yaml
block:
rescue:
```

when recovery is required.

---

### ✅ 6. Validate rollback

Don't assume:

```text
rollback command succeeded
```

means:

```text
system is healthy
```

Actually verify it.

---

# 36. 🏆 Final Mental Model

Memorize this:

```text
                     BLOCK
                       │
                       ▼
                 Normal workflow
                       │
               ┌───────┴───────┐
               ▼               ▼
            SUCCESS          FAILURE
               │               │
               │               ▼
               │            RESCUE
               │               │
               │          Recovery
               │               │
               └───────┬───────┘
                       ▼
                     ALWAYS
                       │
                       ▼
                 Cleanup / Finalize
```

And this:

```text
🧱 block
   = Main operation

🛟 rescue
   = Recovery / rollback

🧹 always
   = Cleanup / finalization
```

---

## 🎯 One production scenario to remember

If the interviewer asks:

> **"How would you safely deploy a new application configuration using Ansible?"**

A strong answer is:

```text
1. Pre-check the environment
        ↓
2. Backup current configuration
        ↓
3. Deploy new configuration
        ↓
4. Validate configuration
        ↓
5. Restart/reload service
        ↓
6. Perform health check
        ↓
7. If anything fails → rescue
        ↓
8. Restore previous configuration
        ↓
9. Restart/reload
        ↓
10. Validate rollback
        ↓
11. Always clean temporary files
```

That demonstrates **automation + idempotency + failure handling + production thinking**, rather than simply knowing YAML syntax. 🔥

---



# 🏷️ Ansible Topic 10 — Tags & Selective Execution

Tags are very useful when your playbook becomes large.

Imagine you have one production playbook that performs:

```text
OS configuration
    ↓
Packages
    ↓
Users
    ↓
Configuration files
    ↓
Application deployment
    ↓
Monitoring
    ↓
Validation
```

You don't always want to run **everything**.

Sometimes you want:

> "Run only the monitoring configuration."

That's where **Ansible tags** come in. 🎯

---

# 1. 🧠 What is an Ansible Tag?

A tag is a label attached to a task, block, role, or other supported automation component.

Example:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
  tags:
    - packages
```

Now you can run only tasks tagged `packages`.

```bash
ansible-playbook -i inventory site.yml --tags packages
```

---

# 2. 🎯 Basic Flow

```text
                    PLAYBOOK
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     packages        config       monitoring
        │              │              │
       🏷️             🏷️             🏷️
     packages        config       monitoring
                       │
                       ▼
             --tags monitoring
                       │
                       ▼
                Only monitoring
                   tasks run
```

---

# 3. 📋 Basic Example

```yaml
---
- name: Configure server
  hosts: webservers
  become: true

  tasks:

    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
      tags:
        - packages

    - name: Deploy nginx configuration
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags:
        - configuration

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
      tags:
        - services
```

Now you have:

```text
packages
configuration
services
```

---

# 4. ▶️ Run Specific Tags

Run only package tasks:

```bash
ansible-playbook -i inventory site.yml --tags packages
```

Run configuration:

```bash
ansible-playbook -i inventory site.yml --tags configuration
```

Run services:

```bash
ansible-playbook -i inventory site.yml --tags services
```

---

# 5. 🏷️ Multiple Tags

A task can have multiple tags.

```yaml
- name: Deploy application configuration
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
  tags:
    - configuration
    - application
    - deployment
```

Now this task runs when you specify **any matching tag**:

```bash
ansible-playbook site.yml --tags configuration
```

or:

```bash
ansible-playbook site.yml --tags application
```

or:

```bash
ansible-playbook site.yml --tags deployment
```

Think:

```text
Task
 │
 ├── configuration
 ├── application
 └── deployment
```

---

# 6. 🎯 Multiple Tags on the Command Line

You can specify multiple tags:

```bash
ansible-playbook site.yml \
  --tags "packages,configuration"
```

This means:

```text
Run tasks tagged:
    packages
       OR
    configuration
```

---

# 7. 🚫 `--skip-tags`

Instead of saying:

> "Run only these."

you can say:

> "Run everything except these."

Example:

```bash
ansible-playbook site.yml --skip-tags monitoring
```

Conceptually:

```text
packages      → RUN
configuration → RUN
services      → RUN
monitoring    → SKIP
```

---

# 8. 🧠 `--tags` vs `--skip-tags`

| Command         | Meaning                    |
| --------------- | -------------------------- |
| `--tags X`      | Run tasks matching X       |
| `--skip-tags X` | Don't run tasks matching X |

Example:

```bash
ansible-playbook site.yml --tags deployment
```

vs.

```bash
ansible-playbook site.yml --skip-tags deployment
```

---

# 9. 🔥 Real Production Example

Suppose your playbook:

```text
site.yml
│
├── OS configuration
├── Packages
├── Users
├── Application
├── Monitoring
└── Validation
```

You assign:

```yaml
tags:
  - os
```

```yaml
tags:
  - packages
```

```yaml
tags:
  - users
```

```yaml
tags:
  - application
```

```yaml
tags:
  - monitoring
```

```yaml
tags:
  - validation
```

Now operations can selectively execute:

```bash
ansible-playbook site.yml --tags monitoring
```

without redeploying the application.

---

# 10. 🏗️ Tags on a Block

You don't need to tag every individual task.

Example:

```yaml
- name: Configure monitoring
  block:

    - name: Install node exporter
      ansible.builtin.package:
        name: node_exporter
        state: present

    - name: Deploy exporter configuration
      ansible.builtin.template:
        src: exporter.yml.j2
        dest: /etc/node_exporter/config.yml

    - name: Start exporter
      ansible.builtin.service:
        name: node_exporter
        state: started

  tags:
    - monitoring
```

Then:

```bash
ansible-playbook site.yml --tags monitoring
```

The block's tasks can be selected through the block tag.

---

# 11. 🧠 Block Tags + Conditions

You can combine them:

```yaml
- name: Production monitoring
  block:

    - name: Configure exporter
      ...

    - name: Start exporter
      ...

  when: environment == "production"
  tags:
    - monitoring
```

Now:

```text
--tags monitoring
        │
        ▼
Monitoring block selected
        │
        ▼
environment == production?
        │
   ┌────┴────┐
  YES        NO
   │          │
   ▼          ▼
 RUN         SKIP
```

Tags select **what is eligible to run**; `when` still determines whether the task actually executes.

---

# 12. 🏷️ `always` Tag

Ansible has a special tag:

```text
always
```

Tasks tagged `always` are designed to run even when you use tag selection.

Example:

```yaml
- name: Validate inventory
  ansible.builtin.assert:
    that:
      - environment is defined
  tags:
    - always
```

Even if you run:

```bash
ansible-playbook site.yml --tags monitoring
```

the `always` task is still selected.

This is useful for:

* prerequisite checks
* important validation
* setup required by other tagged tasks

---

# 13. ⚠️ `always` Does NOT Mean "Ignore Everything"

This is important.

```yaml
tags:
  - always
```

doesn't mean:

> "This task can never fail."

It means:

> "This task participates in execution even when tag filtering is being used."

If the task fails, it can still fail the play.

---

# 14. 🚫 `never`

There is another special tag:

```text
never
```

A task tagged `never` won't normally execute unless you explicitly select its tag.

Example:

```yaml
- name: Debug production system
  ansible.builtin.debug:
    msg: "Detailed diagnostic information"
  tags:
    - never
    - debug
```

Normally:

```bash
ansible-playbook site.yml
```

→ task isn't executed.

But:

```bash
ansible-playbook site.yml --tags debug
```

→ task executes.

This is useful for **optional diagnostics**. 🔍

---

# 15. 🧠 `always` vs `never`

| Special tag | Behavior                                       |
| ----------- | ---------------------------------------------- |
| `always`    | Intended to run even with tag filtering        |
| `never`     | Intended not to run unless explicitly selected |

Mental model:

```text
always
  ↓
"Don't accidentally skip me."


never
  ↓
"Don't run me unless explicitly requested."
```

---

# 16. 🧪 Debugging with `never`

Suppose you have:

```yaml
- name: Collect detailed debug information
  ansible.builtin.command:
    cmd: /opt/myapp/debug
  register: debug_result
  tags:
    - never
    - debug
```

Normal production deployment:

```bash
ansible-playbook site.yml
```

No debug operation.

When troubleshooting:

```bash
ansible-playbook site.yml --tags debug
```

Now it executes.

That's a very useful production pattern.

---

# 17. 🔎 Listing Available Tags

Before running a playbook, you can inspect tags:

```bash
ansible-playbook site.yml --list-tags
```

This is extremely useful.

You might see:

```text
playbook: site.yml

  play #1
    TASK TAGS:
      application
      configuration
      monitoring
      packages
      validation
```

Now you know what can be selectively executed.

---

# 18. 🔍 Listing Tasks

You can also inspect tasks:

```bash
ansible-playbook site.yml --list-tasks
```

This doesn't execute them.

Useful before production execution.

```text
--list-tasks
      ↓
"What is this playbook going to run?"
```

---

# 19. 🧪 Combine With Check Mode

Tags work very nicely with:

```bash
--check
```

Example:

```bash
ansible-playbook site.yml \
  --tags configuration \
  --check
```

Meaning:

> Show me what the configuration tasks would do without making normal changes.

This is an excellent production workflow.

---

# 20. 🔥 Production Workflow

Before modifying production:

```bash
ansible-playbook site.yml --list-tags
```

Then:

```bash
ansible-playbook site.yml \
  --tags application \
  --check
```

Review.

Then:

```bash
ansible-playbook site.yml \
  --tags application
```

This gives:

```text
Discover
   ↓
Dry run
   ↓
Review
   ↓
Execute
```

---

# 21. 🏷️ Tags on Roles

Later we'll learn roles properly.

For now, imagine:

```yaml
- name: Configure web servers
  hosts: webservers

  roles:
    - role: nginx
      tags:
        - web
```

Then:

```bash
ansible-playbook site.yml --tags web
```

can select the role's tagged execution.

---

# 22. 🧩 Tags and Includes

This is where things get slightly more advanced.

Ansible has:

```text
include_tasks
import_tasks
include_role
import_role
```

These don't behave identically with respect to tag propagation.

You don't need to memorize this yet, but understand the distinction:

```text
import
   ↓
static / processed earlier

include
   ↓
dynamic / processed during execution
```

We'll cover this properly when we study **Roles and Reuse**.

---

# 23. ⚠️ Common Production Mistake

Imagine:

```yaml
- name: Stop application
  ansible.builtin.service:
    name: myapp
    state: stopped
  tags:
    - application

- name: Deploy application
  ...
  tags:
    - application

- name: Start application
  ansible.builtin.service:
    name: myapp
    state: started
  tags:
    - application
```

Now someone runs:

```bash
ansible-playbook site.yml --tags application
```

That's fine.

But if you tag only:

```text
Deploy application
```

and forget:

```text
Start application
```

a selective execution might leave the system in an unexpected state.

### Production lesson:

> **Tags should represent coherent operational units, not arbitrary individual tasks.**

---

# 24. 🎯 Good Tag Design

Good:

```text
os
packages
users
application
configuration
monitoring
validation
```

Bad:

```text
task1
task2
task3
fix
thing
temporary
```

Tags should communicate **intent**.

---

# 25. 🏭 Example Production Playbook

```yaml
---
- name: Configure production server
  hosts: webservers
  become: true

  tasks:

    - name: Install required packages
      ansible.builtin.package:
        name:
          - nginx
          - curl
        state: present
      tags:
        - packages

    - name: Create application user
      ansible.builtin.user:
        name: myapp
        state: present
      tags:
        - users

    - name: Deploy application configuration
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf
      notify:
        - Restart myapp
      tags:
        - configuration
        - application

    - name: Configure monitoring
      ansible.builtin.template:
        src: monitoring.yml.j2
        dest: /etc/myapp/monitoring.yml
      tags:
        - monitoring

    - name: Validate application
      ansible.builtin.uri:
        url: http://localhost:8080/health
        status_code: 200
      tags:
        - validation

  handlers:

    - name: Restart myapp
      ansible.builtin.service:
        name: myapp
        state: restarted
```

Now:

```bash
# Everything
ansible-playbook site.yml

# Only packages
ansible-playbook site.yml --tags packages

# Only application
ansible-playbook site.yml --tags application

# Application + monitoring
ansible-playbook site.yml --tags "application,monitoring"

# Everything except monitoring
ansible-playbook site.yml --skip-tags monitoring
```

---

# 26. 🧠 Tags Don't Change Dependencies

This is important.

Suppose:

```text
Install package
     ↓
Deploy config
     ↓
Start service
```

You select:

```bash
--tags configuration
```

Ansible doesn't automatically say:

> "Oh, configuration needs the package, so I'll install the package too."

Tag filtering selects tasks based on their tags.

Therefore, you need to design tags carefully.

---

# 27. 🚨 A3-Level Production Scenario

Imagine you have:

```text
100 production servers
```

Your playbook contains:

```text
OS
Packages
Application
Monitoring
Security
Validation
```

You discover a monitoring configuration issue.

You don't want:

```text
100 servers
×
full application deployment
```

Instead:

```bash
ansible-playbook site.yml \
  --limit production_monitoring_nodes \
  --tags monitoring
```

Now you combine:

```text
inventory targeting
       +
tag selection
       ↓
precise execution
```

🔥 This is where tags become extremely powerful.

---

# 28. 🎯 Tags + `--limit`

This combination is worth remembering.

```bash
ansible-playbook site.yml \
  --limit web01 \
  --tags configuration
```

Means:

> Run only configuration-tagged tasks on `web01`.

Visual:

```text
                 PLAYBOOK
                    │
          ┌─────────┴─────────┐
          │                   │
       --limit              --tags
          │                   │
       web01            configuration
          │                   │
          └─────────┬─────────┘
                    ▼
             Precise execution
```

This is extremely useful during production troubleshooting.

---

# 29. 🔥 Tags + `--check` + `--limit`

A very practical workflow:

```bash
ansible-playbook site.yml \
  --limit web01 \
  --tags configuration \
  --check
```

Then, after validation:

```bash
ansible-playbook site.yml \
  --limit web01 \
  --tags configuration
```

Then expand:

```text
web01
 ↓
small group
 ↓
larger group
 ↓
full environment
```

This is a good operational approach.

---

# 30. 🧠 Tags vs Variables

Don't confuse:

```text
Tags
```

with:

```text
Variables
```

### Tags

Control:

> **Which tasks execute?**

### Variables

Control:

> **What values those tasks use?**

Example:

```text
Tag:
application

Variable:
app_version=2.5.0
```

Together:

```text
--tags application
        +
app_version=2.5.0
```

---

# 31. 🧠 Tags vs `when`

Again:

### Tag

```text
Should this task be considered for this invocation?
```

### `when`

```text
Given that this task is being considered,
should it actually execute on this host?
```

Example:

```yaml
- name: Configure production monitoring
  ansible.builtin.template:
    src: monitoring.yml.j2
    dest: /etc/myapp/monitoring.yml
  tags:
    - monitoring
  when: environment == "production"
```

Both conditions matter.

```text
--tags monitoring
       ↓
Task selected
       ↓
when condition
       ↓
environment == production?
       ↓
RUN / SKIP
```

---

# 32. 🏆 Interview Questions

### Q1. What are Ansible tags?

> Tags are labels that allow selective execution of tasks, blocks, roles, and other supported components of a playbook.

### Q2. How do you run only tasks with a specific tag?

```bash
ansible-playbook site.yml --tags monitoring
```

### Q3. How do you skip a tag?

```bash
ansible-playbook site.yml --skip-tags monitoring
```

### Q4. How do you list tags?

```bash
ansible-playbook site.yml --list-tags
```

### Q5. How do you combine tags with a specific host?

```bash
ansible-playbook site.yml --limit web01 --tags monitoring
```

### Q6. What is the `always` tag?

> A special tag intended to make a task participate even when tag filtering is being used.

### Q7. What is the `never` tag?

> A special tag intended to prevent normal execution unless explicitly selected.

### Q8. Do tags automatically handle task dependencies?

> No. Tag filtering does not automatically execute prerequisite tasks that were omitted by the selected tags.

---

# 33. 🎯 Production Best Practices

### ✅ Use meaningful tags

```text
application
configuration
monitoring
security
validation
```

### ✅ Keep tags aligned with operational activities

Don't create hundreds of meaningless tags.

### ✅ Test selective execution

```bash
--list-tags
--list-tasks
--check
```

### ✅ Combine tags with `--limit`

Especially useful for production troubleshooting.

### ✅ Don't assume dependencies will execute

If a task requires another task, design the tags appropriately.

### ✅ Use `never` for dangerous/diagnostic operations

This prevents accidental execution.

---

# 🧠 34. Final Mental Model

Remember:

```text
                         PLAYBOOK
                            │
                            ▼
                         TASKS
                            │
                   ┌────────┴────────┐
                   │                 │
                 TAGS              when
                   │                 │
                   ▼                 ▼
            Which tasks?       Should it run?
                   │                 │
                   └────────┬────────┘
                            ▼
                         MODULE
                            │
                            ▼
                        EXECUTION
```

And the commands:

```text
--tags
   ↓
"I want these."

--skip-tags
   ↓
"I don't want these."

--list-tags
   ↓
"What tags exist?"

--list-tasks
   ↓
"What tasks exist?"

--limit
   ↓
"Which hosts?"

--check
   ↓
"What would happen?"
```

🔥 **The real power comes from combining them:**

```bash
ansible-playbook site.yml \
  --limit web01 \
  --tags configuration \
  --check
```

That means:

> **"Show me what the configuration-tagged tasks would do on web01 without making the normal changes."**

That's a very practical production command to know.

---


# 🧩 Ansible Topic 11 — Roles & Production Project Structure

This is a **major Ansible topic** and very important for your LevelUp/A3 preparation.

So far we've been writing:

```text
site.yml
  │
  ├── tasks
  ├── handlers
  └── variables
```

That works for small automation.

But imagine your project grows to:

```text
50+ tasks
20 templates
10 handlers
multiple environments
multiple applications
```

A single playbook becomes difficult to maintain.

That's where **Ansible Roles** come in. 🚀

---

# 1. 🧠 What is an Ansible Role?

A role is a **standardized, reusable directory structure for packaging Ansible automation**.

Think:

> **A role packages related tasks, handlers, variables, templates, files and metadata into a reusable unit.**

Instead of:

```text
site.yml
 ├── 300 tasks
 ├── 30 handlers
 ├── 50 variables
 └── 20 templates
```

we can organize:

```text
roles/
├── nginx/
├── postgresql/
├── monitoring/
└── myapp/
```

Each role owns its own automation.

---

# 2. 🎯 Why Do We Need Roles?

Without roles:

```text
site.yml
│
├── Install nginx
├── Configure nginx
├── Deploy SSL
├── Configure logging
├── Install monitoring
├── Configure monitoring
├── Deploy application
├── Configure application
└── ...
```

This becomes difficult to manage.

With roles:

```text
site.yml
│
├── nginx role
├── monitoring role
└── myapp role
```

Much cleaner:

```text
PLAYBOOK
   │
   ├── nginx
   │
   ├── monitoring
   │
   └── myapp
```

---

# 3. 🏗️ Visualize a Role

Think of a role as a **self-contained automation package**:

```text
                  📦 ROLE
                     │
        ┌────────────┼────────────┐
        │            │            │
      Tasks       Handlers      Templates
        │            │            │
        ├── main.yml │            └── config.j2
        │            │
        │            └── main.yml
        │
        ├── Defaults
        │
        ├── Variables
        │
        ├── Files
        │
        └── Metadata
```

---

# 4. 📁 Standard Role Structure

A typical role:

```text
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    │
    ├── handlers/
    │   └── main.yml
    │
    ├── defaults/
    │   └── main.yml
    │
    ├── vars/
    │   └── main.yml
    │
    ├── templates/
    │   └── nginx.conf.j2
    │
    ├── files/
    │   └── index.html
    │
    ├── meta/
    │   └── main.yml
    │
    └── README.md
```

Not every role needs every directory.

---

# 5. 🧠 What Does Each Directory Mean?

| Directory    | Purpose                               |
| ------------ | ------------------------------------- |
| `tasks/`     | Main automation tasks                 |
| `handlers/`  | Handlers                              |
| `defaults/`  | Default variables, easy to override   |
| `vars/`      | Role variables with higher precedence |
| `templates/` | Jinja2 templates                      |
| `files/`     | Static files                          |
| `meta/`      | Role metadata and dependencies        |
| `README.md`  | Documentation                         |

This table is worth remembering. 🧠

---

# 6. 📋 `tasks/main.yml`

This is normally the **entry point for the role's tasks**.

Example:

```yaml
---
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present

- name: Deploy nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - Restart nginx

- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

When the role runs, Ansible starts from:

```text
roles/nginx/tasks/main.yml
```

---

# 7. 📢 `handlers/main.yml`

Handlers belonging to the role go here.

```yaml
---
- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

Then:

```text
tasks/main.yml
       │
       │ notify
       ▼
handlers/main.yml
```

Example:

```yaml
notify:
  - Restart nginx
```

---

# 8. 🎨 `templates/`

Templates are Jinja2 files.

Example:

```text
roles/nginx/templates/nginx.conf.j2
```

Contents:

```jinja2
worker_processes {{ nginx_worker_processes }};

events {
    worker_connections {{ nginx_worker_connections }};
}
```

Task:

```yaml
- name: Deploy nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

Ansible automatically knows to look inside the role's:

```text
templates/
```

directory.

---

# 9. 📄 `files/`

Static files go here.

Example:

```text
roles/nginx/files/index.html
```

Task:

```yaml
- name: Deploy index page
  ansible.builtin.copy:
    src: index.html
    dest: /usr/share/nginx/html/index.html
```

Ansible looks for:

```text
roles/nginx/files/index.html
```

---

# 10. 🧠 `templates` vs `files`

Very important:

### `files/`

For:

> Static content.

Example:

```text
index.html
certificate.pem
script.sh
```

### `templates/`

For:

> Dynamic content containing variables/Jinja expressions.

Example:

```jinja2
server_name {{ nginx_server_name }};
port {{ nginx_port }};
```

Mental model:

```text
files/
   ↓
"Copy exactly."


templates/
   ↓
"Render first, then deploy."
```

---

# 11. ⚙️ `defaults/main.yml`

This is one of the **most important role concepts**.

Example:

```yaml
---
nginx_port: 80
nginx_worker_processes: 2
nginx_server_name: localhost
```

These are default values.

A user can override them.

Think:

```text
defaults
   ↓
Safe starting values
   ↓
Easy to override
```

---

# 12. 🧠 Why Put Variables in `defaults`?

Suppose your role supports:

```text
nginx_port
nginx_server_name
nginx_worker_processes
```

You want users to customize these without modifying the role itself.

So:

```yaml
# defaults/main.yml

nginx_port: 80
```

Then:

```yaml
vars:
  nginx_port: 8080
```

or inventory variables can override it according to precedence.

This makes the role reusable.

---

# 13. 🔐 `vars/main.yml`

Role variables can also be defined in:

```text
vars/main.yml
```

Example:

```yaml
---
nginx_package_name: nginx
nginx_config_path: /etc/nginx/nginx.conf
```

The important difference is:

```text
defaults/main.yml
   ↓
lower precedence
   ↓
easy to override


vars/main.yml
   ↓
higher precedence
   ↓
harder to override
```

---

# 14. ⭐ Most Important Difference

For roles:

```text
defaults/
    ↓
"user configurable"

vars/
    ↓
"role-controlled values"
```

### Production guideline

Use:

```text
defaults/
```

for values you expect users/environments to customize.

Use:

```text
vars/
```

for values that are more internal to the role and shouldn't normally be overridden.

---

# 15. 🧠 Example

### `defaults/main.yml`

```yaml
nginx_port: 80
nginx_worker_processes: 2
```

### `vars/main.yml`

```yaml
nginx_config_file: /etc/nginx/nginx.conf
```

Why?

User may want:

```text
port → 8080
workers → 4
```

but usually shouldn't need to redefine:

```text
config_file → /etc/nginx/nginx.conf
```

---

# 16. 📦 `meta/main.yml`

This contains metadata about the role.

Example:

```yaml
---
galaxy_info:
  author: dinesh
  description: Configure nginx
  min_ansible_version: "2.15"

dependencies: []
```

It can also define **role dependencies**.

We'll cover dependencies separately.

---

# 17. 🎯 Calling a Role From a Playbook

Suppose:

```text
roles/
└── nginx/
```

Playbook:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  roles:
    - nginx
```

That's it.

Ansible finds:

```text
roles/nginx/
```

and executes its role structure.

---

# 18. 🧠 Execution Flow

```text
site.yml
   │
   ▼
roles:
  - nginx
   │
   ▼
roles/nginx/
   │
   ├── defaults/main.yml
   │
   ├── tasks/main.yml
   │
   ├── templates/
   │
   ├── files/
   │
   └── handlers/main.yml
```

Think:

> `site.yml` says **which role**.

The role says:

> **how to perform the automation.**

---

# 19. 🏭 Real Production Structure

For your kind of infrastructure automation, I'd expect something like:

```text
ansible-project/
│
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   │
│   └── staging/
│       ├── hosts.yml
│       └── group_vars/
│
├── roles/
│   ├── common/
│   ├── postgresql/
│   ├── patroni/
│   ├── etcd/
│   ├── haproxy/
│   └── monitoring/
│
├── playbooks/
│   ├── site.yml
│   ├── database.yml
│   └── monitoring.yml
│
└── ansible.cfg
```

Architecture:

```text
                     ANSIBLE PROJECT
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Inventory         Playbooks          Roles
          │                │                │
      Environments      site.yml        postgresql
      group_vars        database.yml    patroni
      host_vars         monitoring.yml   etcd
                                         haproxy
                                         monitoring
```

This is much closer to how production repositories are structured.

---

# 20. 🚀 Why Roles Are Powerful

Imagine you create a PostgreSQL role once:

```text
roles/postgresql/
```

Now you can use it for:

```text
Project A
Project B
Project C
```

with different variables.

```text
                PostgreSQL Role
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Project A   Project B   Project C
          │           │           │
       version 16   version 17   version 16
```

The automation logic stays reusable.

---

# 21. 🧠 Role Reusability

Suppose:

```yaml
# defaults/main.yml

postgresql_version: 16
postgresql_port: 5432
```

Project A:

```yaml
postgresql_version: 16
```

Project B:

```yaml
postgresql_version: 17
```

Same role.

Different configuration.

This is exactly what we want from automation.

---

# 22. 🔥 Role Variables From Playbook

Example:

```yaml
---
- name: Configure database
  hosts: databases
  become: true

  roles:

    - role: postgresql
      vars:
        postgresql_version: 17
        postgresql_port: 5433
```

This lets you configure the role from the caller.

---

# 23. 🧠 Role Variables vs Global Variables

You should avoid making every variable global.

Instead:

```text
postgresql role
   │
   ├── postgresql_version
   ├── postgresql_port
   └── postgresql_data_dir
```

and:

```text
nginx role
   │
   ├── nginx_port
   ├── nginx_worker_processes
   └── nginx_server_name
```

This reduces naming collisions.

---

# 24. 🏷️ Role Tags

You can tag a role:

```yaml
roles:

  - role: nginx
    tags:
      - web
```

Then:

```bash
ansible-playbook site.yml --tags web
```

This connects directly with our previous topic.

---

# 25. 🧩 `include_role`

You can also dynamically include a role:

```yaml
- name: Configure nginx
  ansible.builtin.include_role:
    name: nginx
```

This is different from simply:

```yaml
roles:
  - nginx
```

We'll cover the difference later when we discuss **static vs dynamic reuse**.

For now:

```text
roles:
  - nginx
```

is the simplest and most common approach.

---

# 26. 🧩 `import_role`

You can also:

```yaml
- name: Import nginx role
  ansible.builtin.import_role:
    name: nginx
```

Conceptually:

```text
import_role
   ↓
static reuse


include_role
   ↓
dynamic reuse
```

We'll deep-dive this later.

---

# 27. 🧠 Role Execution Order

When you use:

```yaml
roles:
  - common
  - nginx
  - monitoring
```

think:

```text
common
   ↓
nginx
   ↓
monitoring
```

Each role brings its own:

```text
tasks
handlers
variables
templates
files
```

---

# 28. 🔥 Important: Handlers Are Still Queued

Suppose:

```text
nginx role
  ↓
template changed
  ↓
notify Restart nginx
```

The handler isn't necessarily executed immediately.

It follows the normal Ansible handler behavior we learned earlier.

This means:

```text
Role
 ↓
Task
 ↓
notify
 ↓
Handler queue
 ↓
Handler execution
```

---

# 29. 🏗️ Build a Complete Nginx Role

Let's create one.

```text
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    ├── vars/
    │   └── main.yml
    └── meta/
        └── main.yml
```

---

# 30. 📄 `defaults/main.yml`

```yaml
---
nginx_port: 80
nginx_worker_processes: 2
nginx_server_name: localhost
```

---

# 31. 📄 `tasks/main.yml`

```yaml
---
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present

- name: Deploy nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify:
    - Restart nginx

- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

---

# 32. 📄 `templates/nginx.conf.j2`

Simplified example:

```jinja2
worker_processes {{ nginx_worker_processes }};

events {
    worker_connections 1024;
}

http {
    server {
        listen {{ nginx_port }};

        server_name {{ nginx_server_name }};

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
}
```

---

# 33. 📄 `handlers/main.yml`

```yaml
---
- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

---

# 34. 📄 `vars/main.yml`

```yaml
---
nginx_config_path: /etc/nginx/nginx.conf
```

Then task can use:

```yaml
dest: "{{ nginx_config_path }}"
```

---

# 35. 📄 `meta/main.yml`

```yaml
---
galaxy_info:
  author: dinesh
  description: Nginx configuration role
  min_ansible_version: "2.15"

dependencies: []
```

---

# 36. 📘 `site.yml`

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  roles:
    - role: nginx
```

That's a complete basic role.

---

# 37. 🔄 Full Execution Flow

```text
                 site.yml
                    │
                    ▼
               role: nginx
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     defaults     tasks       vars
        │           │
        │           ├── install
        │           ├── template
        │           └── service
        │                 │
        │              changed
        │                 │
        │               notify
        │                 │
        │                 ▼
        │             handlers
        │                 │
        └─────────────────┘
```

---

# 38. 🧠 Role Directory Conventions

Ansible knows special directories automatically.

For example:

```yaml
ansible.builtin.template:
  src: nginx.conf.j2
```

inside the role means:

```text
roles/nginx/templates/nginx.conf.j2
```

Similarly:

```yaml
ansible.builtin.copy:
  src: index.html
```

means:

```text
roles/nginx/files/index.html
```

This is why role structure is standardized.

---

# 39. 🛠️ Creating a Role

You can create a skeleton using:

```bash
ansible-galaxy role init nginx
```

This generates the standard role structure.

You might see:

```text
nginx/
├── README.md
├── defaults/
├── handlers/
├── meta/
├── tasks/
├── templates/
├── tests/
├── vars/
└── files/
```

Very useful when starting a role.

---

# 40. 🧠 What is `ansible-galaxy`?

`ansible-galaxy` is a command-line tool used for working with:

* roles
* collections

For example:

```bash
ansible-galaxy role init nginx
```

creates a role.

Later we'll use it for:

```text
installing roles
publishing roles
collections
dependencies
```

---

# 41. 🧩 Role Dependencies

Suppose:

```text
myapp
```

requires:

```text
postgresql
```

You can define:

```text
myapp
  ↓
depends on
  ↓
postgresql
```

Role dependencies can be defined in:

```text
meta/main.yml
```

Example:

```yaml
dependencies:
  - role: postgresql
```

Then:

```text
myapp role
    │
    ▼
postgresql role
    │
    ▼
myapp tasks
```

We'll cover role dependencies in depth later.

---

# 42. 🏭 Production Role Design

For your Ansible work, I'd recommend thinking in terms of **responsibilities**.

Good:

```text
roles/
├── common/
├── users/
├── packages/
├── postgresql/
├── patroni/
├── etcd/
├── haproxy/
├── monitoring/
└── alloydb/
```

Each role should have a clear responsibility.

Bad:

```text
roles/
└── everything/
```

with:

```text
2,000 lines of tasks
```

---

# 43. 🧠 Role Granularity

Don't make roles too small:

```text
install_nginx_role
create_nginx_user_role
create_nginx_directory_role
```

That's usually unnecessary.

But don't make one giant:

```text
everything_role
```

Either.

A good boundary is:

> **One role should represent a meaningful reusable component or responsibility.**

For example:

```text
postgresql
patroni
etcd
haproxy
monitoring
```

are meaningful components.

---

# 44. 🔥 Your AlloyDB-Type Project

For a project like your AlloyDB automation, a possible structure could look like:

```text
roles/
│
├── os_baseline/
│
├── postgres/
│
├── patroni/
│
├── etcd/
│
├── keepalived/
│
├── haproxy/
│
├── prometheus/
│
├── postgres_exporter/
│
└── rsyslog/
```

Then:

```text
site.yml
   │
   ├── os_baseline
   ├── etcd
   ├── postgres
   ├── patroni
   ├── keepalived
   ├── haproxy
   └── monitoring
```

That's a very natural production architecture.

---

# 45. 🧠 Role vs Playbook

This is an interview favorite.

### Playbook

> Orchestrates automation.

Example:

```yaml
roles:
  - postgres
  - patroni
  - monitoring
```

### Role

> Packages reusable automation.

Think:

```text
Playbook
   ↓
"What components should I deploy?"

Role
   ↓
"How do I deploy this component?"
```

---

# 46. 🎤 Interview Answer

If asked:

> **Why do you use roles instead of putting everything in a playbook?**

A strong answer:

> "Roles provide a standardized and reusable structure for organizing tasks, handlers, templates, files, variables and dependencies. They make large Ansible projects easier to maintain, test and reuse across environments. The playbook can then focus primarily on orchestration while the roles encapsulate component-specific implementation."

That's an excellent A3-level answer.

---

# 47. 🧠 Role vs Collection

Don't confuse these.

### Role

```text
One reusable automation unit.
```

Example:

```text
postgresql role
```

### Collection

```text
A distribution/package containing Ansible content.
```

A collection can contain:

```text
roles
modules
plugins
playbooks
module_utils
```

So:

```text
Collection
   │
   ├── Roles
   ├── Modules
   ├── Plugins
   └── Other Ansible content
```

We'll have a dedicated **Collections** topic later.

---

# 48. 🏆 Important Role Directories to Memorize

If you're asked:

> "What is the standard Ansible role structure?"

You should immediately be able to say:

```text
roles/
└── role_name/
    ├── tasks/
    ├── handlers/
    ├── defaults/
    ├── vars/
    ├── templates/
    ├── files/
    ├── meta/
    └── tests/
```

And explain:

```text
tasks      → automation
handlers   → event-driven actions
defaults   → overridable defaults
vars       → role variables
templates  → Jinja templates
files      → static files
meta       → metadata/dependencies
tests      → role testing
```

🔥 This is worth knowing very well.

---

# 49. ⚠️ Common Mistakes

### ❌ Putting user-configurable values in `vars/main.yml`

Prefer:

```text
defaults/main.yml
```

for values intended to be customized.

---

### ❌ Hardcoding values inside tasks

Instead of:

```yaml
port: 8080
```

consider:

```yaml
port: "{{ nginx_port }}"
```

with:

```yaml
defaults/main.yml
```

---

### ❌ One giant role

Break components into meaningful roles.

---

### ❌ Mixing unrelated responsibilities

Avoid:

```text
postgres role
 ├── PostgreSQL
 ├── Nginx
 ├── Docker
 └── monitoring
```

unless there is a very deliberate reason.

---

# 50. 🎯 Final Mental Model

Remember this:

```text
                    📘 PLAYBOOK
                         │
                         │ orchestrates
                         ▼
                      📦 ROLES
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      PostgreSQL       Patroni       Monitoring
          │              │              │
          ▼              ▼              ▼
        tasks          tasks          tasks
        handlers       handlers       handlers
        templates      templates      templates
        defaults       defaults       defaults
```

And:

```text
📋 tasks/main.yml
    → What should the role do?

📢 handlers/main.yml
    → What should happen after a change?

⚙️ defaults/main.yml
    → What are the configurable defaults?

🔐 vars/main.yml
    → What internal role variables are needed?

🎨 templates/
    → What dynamic configuration should be rendered?

📄 files/
    → What static files should be copied?

🧩 meta/main.yml
    → What metadata/dependencies does the role have?
```

---

# 🏆 The most important distinction

Memorize this:

```text
PLAYBOOK
   ↓
ORCHESTRATION

ROLE
   ↓
REUSABLE IMPLEMENTATION

TASK
   ↓
INDIVIDUAL OPERATION

MODULE
   ↓
ACTUAL ACTION
```

For example:

```text
site.yml
   ↓
postgres role
   ↓
install PostgreSQL task
   ↓
ansible.builtin.package
```

That's the complete mental hierarchy. 💪

---



# 🧩 Ansible Topic 12 — Role Dependencies, Variables & Advanced Role Reuse

This is the **advanced part of Roles**.

You already know:

```text
Playbook
   ↓
Role
   ↓
tasks/
handlers/
templates/
defaults/
vars/
```

Now we'll answer the production questions:

> How does one role depend on another?
> How do I pass variables to a role?
> What's the difference between `include_role` and `import_role`?
> When should I use each?
> Can I run the same role multiple times with different configuration?

These are very useful interview areas. 🔥

---

# 1. 🧠 Role Dependencies

Imagine you have:

```text
myapp
```

and your application requires:

```text
PostgreSQL
```

You could manually write:

```yaml
roles:
  - postgresql
  - myapp
```

But the `myapp` role itself knows:

> "I require PostgreSQL."

You can express that dependency inside:

```text
roles/myapp/meta/main.yml
```

---

# 2. 📄 Basic Dependency

```yaml
---
dependencies:
  - role: postgresql
```

Now:

```text
myapp
  │
  │ depends on
  ▼
postgresql
```

When `myapp` is executed:

```text
PostgreSQL
    ↓
myapp
```

The dependency is executed before the dependent role.

---

# 3. 🏗️ Real Example

Structure:

```text
roles/
├── postgresql/
│   ├── tasks/
│   └── ...
│
└── myapp/
    ├── tasks/
    ├── defaults/
    └── meta/
        └── main.yml
```

`myapp/meta/main.yml`:

```yaml
---
dependencies:
  - role: postgresql
```

Playbook:

```yaml
---
- name: Deploy application
  hosts: appservers

  roles:
    - myapp
```

Execution:

```text
site.yml
   │
   ▼
myapp
   │
   │ dependency
   ▼
postgresql
   │
   ▼
myapp tasks
```

---

# 4. 🧠 Why Dependencies Are Useful

Imagine:

```text
Patroni
  ↓
requires PostgreSQL

Monitoring
  ↓
requires postgres_exporter

MyApp
  ↓
requires PostgreSQL
```

Instead of remembering the order manually:

```yaml
roles:
  - postgresql
  - patroni
  - postgres_exporter
  - myapp
```

you can model relationships:

```text
Patroni
   ↓
PostgreSQL

MyApp
   ↓
PostgreSQL
```

This makes roles more self-contained.

---

# 5. ⚠️ Don't Create Dependency Chains Everywhere

You can create:

```text
A
 ↓
B
 ↓
C
 ↓
D
 ↓
E
```

but this can become difficult to understand.

Production guideline:

> Keep role dependencies meaningful and minimal.

For example:

```text
patroni
   ↓
postgresql
```

makes sense.

But:

```text
application
 ↓
monitoring
 ↓
logging
 ↓
users
 ↓
packages
 ↓
common
```

may become unnecessarily complicated.

---

# 6. 🎯 Passing Variables to Dependencies

Dependencies can receive variables.

Example:

```yaml
---
dependencies:

  - role: postgresql
    vars:
      postgresql_version: "17"
      postgresql_port: 5433
```

Flow:

```text
myapp
 │
 └── dependency
       │
       ▼
   postgresql
       │
       ├── version = 17
       └── port = 5433
```

---

# 7. 🧠 Role Variables From the Playbook

You already saw:

```yaml
roles:

  - role: postgresql
    vars:
      postgresql_version: "17"
```

This is one of the cleanest ways to configure a reusable role.

For example:

```yaml
---
- name: Configure databases
  hosts: databases

  roles:

    - role: postgresql
      vars:
        postgresql_version: "17"
        postgresql_port: 5432
```

The role itself remains reusable.

---

# 8. 🏆 Role Defaults

Suppose:

```yaml
# defaults/main.yml

postgresql_version: "16"
postgresql_port: 5432
```

Caller:

```yaml
roles:

  - role: postgresql
    vars:
      postgresql_version: "17"
```

Result:

```text
default version = 16
        ↓
caller specifies 17
        ↓
effective value = 17
```

This is exactly why `defaults` exists.

---

# 9. 🔐 Role `vars`

Suppose:

```yaml
# vars/main.yml

postgresql_config_path: /etc/postgresql/postgresql.conf
```

This is intended to be more role-controlled.

Generally:

```text
defaults
   ↓
user/environment configurable

vars
   ↓
internal role values
```

Don't put everything into `vars/main.yml`.

---

# 10. 🧠 Important Precedence Concept

You don't need to memorize the entire Ansible variable precedence list yet.

For roles, remember this practical rule:

```text
defaults/main.yml
        ↓
low precedence
        ↓
easy to override
```

while:

```text
vars/main.yml
        ↓
higher precedence
        ↓
harder to override
```

This is why:

```text
Configurable parameters → defaults
Internal implementation values → vars
```

is a good design.

---

# 11. 🔄 `include_role`

Now we reach an important concept.

Instead of:

```yaml
roles:
  - nginx
```

you can execute a role from inside tasks:

```yaml
tasks:

  - name: Configure nginx
    ansible.builtin.include_role:
      name: nginx
```

This is called **dynamic role inclusion**.

---

# 12. 🧠 Why Would You Need `include_role`?

Because you can put conditions around it.

Example:

```yaml
tasks:

  - name: Configure nginx
    ansible.builtin.include_role:
      name: nginx
    when: install_web_server | bool
```

Flow:

```text
install_web_server?
       │
   ┌───┴───┐
  YES      NO
   │        │
   ▼        ▼
nginx     skip
```

This gives you more dynamic control.

---

# 13. 🔥 `include_role` With Variables

```yaml
tasks:

  - name: Configure nginx
    ansible.builtin.include_role:
      name: nginx
    vars:
      nginx_port: 8080
```

The role receives:

```text
nginx_port = 8080
```

---

# 14. 🧩 `import_role`

You can also:

```yaml
tasks:

  - name: Import nginx role
    ansible.builtin.import_role:
      name: nginx
```

The important conceptual difference:

```text
import_role
    ↓
static

include_role
    ↓
dynamic
```

This is an interview favorite.

---

# 15. 🧠 Static vs Dynamic

Think about Ansible processing.

### `import_role`

Ansible knows about the role's tasks earlier during playbook processing.

```text
Playbook
   ↓
IMPORT role
   ↓
Tasks become part of play structure
   ↓
Execution
```

### `include_role`

Ansible decides to load the role dynamically during execution.

```text
Playbook
   ↓
Task reached
   ↓
INCLUDE role
   ↓
Role loaded
   ↓
Execution
```

---

# 16. 🎯 Simple Analogy

Think:

```text
import
   =
"Bring this content in."


include
   =
"At runtime, decide whether to bring this content in."
```

Not a perfect programming-language analogy, but useful.

---

# 17. 🆚 `roles:` vs `include_role` vs `import_role`

| Method         | Main idea           |
| -------------- | ------------------- |
| `roles:`       | Standard role usage |
| `import_role`  | Static role reuse   |
| `include_role` | Dynamic role reuse  |

For most normal roles:

```yaml
roles:
  - nginx
```

is perfectly fine.

Use `include_role` or `import_role` when you specifically need task-level control.

---

# 18. 🔥 Conditional Role Execution

With `include_role`:

```yaml
tasks:

  - name: Configure database
    ansible.builtin.include_role:
      name: postgresql
    when: database_enabled | bool
```

This is very useful.

Example:

```text
production
   ↓
database_enabled=true
   ↓
PostgreSQL role runs


development
   ↓
database_enabled=false
   ↓
PostgreSQL role skipped
```

---

# 19. 🧠 Role Tags With `include_role`

You can do:

```yaml
tasks:

  - name: Configure monitoring
    ansible.builtin.include_role:
      name: monitoring
    tags:
      - monitoring
```

Then:

```bash
ansible-playbook site.yml --tags monitoring
```

---

# 20. 🔁 Running the Same Role Multiple Times

This is a very useful feature.

Suppose you have a generic role:

```text
roles/user/
```

You want:

```text
appuser
backupuser
```

You don't need:

```text
roles/appuser/
roles/backupuser/
```

You can reuse the same role.

```yaml
roles:

  - role: user
    vars:
      username: appuser

  - role: user
    vars:
      username: backupuser
```

Conceptually:

```text
          user role
          /       \
         /         \
   appuser       backupuser
```

---

# 21. 🚨 Role Reuse and `allow_duplicates`

Ansible has protections around duplicate role execution.

You may encounter:

```yaml
allow_duplicates: true
```

in role metadata.

Example:

```yaml
---
allow_duplicates: true
```

This tells Ansible that the role is allowed to execute multiple times even when Ansible's role dependency de-duplication would otherwise prevent repeated execution in some scenarios.

You don't need to use this routinely.

The important interview concept is:

> Ansible can deduplicate role executions in dependency processing, and `allow_duplicates` can control that behavior.

---

# 22. 🧠 Why Would You Run the Same Role Twice?

A common example:

```text
user role
```

Input:

```text
username
shell
groups
```

Then:

```yaml
roles:

  - role: user
    vars:
      username: appuser

  - role: user
    vars:
      username: monitoring
```

Same implementation.

Different data.

That's excellent role design.

---

# 23. 🏗️ Another Example — Multiple PostgreSQL Instances

Suppose your role supports:

```text
postgresql_port
postgresql_data_dir
```

You could conceptually have:

```yaml
roles:

  - role: postgresql
    vars:
      postgresql_port: 5432
      postgresql_data_dir: /data/postgres1

  - role: postgresql
    vars:
      postgresql_port: 5433
      postgresql_data_dir: /data/postgres2
```

Whether this is safe depends heavily on how the role is implemented.

The lesson is:

> A reusable role can be parameterized and potentially invoked multiple times.

---

# 24. 🧠 Role Dependency Chain

Suppose:

```text
myapp
  ↓
patroni
  ↓
postgresql
```

You could have:

### `myapp/meta/main.yml`

```yaml
dependencies:
  - role: patroni
```

### `patroni/meta/main.yml`

```yaml
dependencies:
  - role: postgresql
```

Then:

```text
myapp
 ↓
patroni
 ↓
postgresql
```

Ansible resolves the dependency chain.

---

# 25. 🔥 Production Concern — Dependency Ordering

Suppose:

```text
PostgreSQL
   ↓
Patroni
```

You don't want:

```text
Patroni starts
   ↓
PostgreSQL doesn't exist
```

The dependency relationship establishes the intended ordering.

```text
PostgreSQL configured
        ↓
Patroni configured
        ↓
Patroni started
```

This is especially useful for complex infrastructure.

---

# 26. 🧠 Role Dependency vs Explicit Role Ordering

You could write:

```yaml
roles:
  - postgresql
  - patroni
```

This explicitly gives the order.

Or:

```text
patroni
  ↓ dependency
postgresql
```

The second approach allows the `patroni` role to declare what it requires.

### Tradeoff

Explicit playbook ordering:

```text
easy to see
```

Dependencies:

```text
component owns its prerequisites
```

Use dependencies when the relationship is genuinely intrinsic.

---

# 27. 🔐 Role Variable Scope

A common confusion:

> "If I pass a variable to a role, where does it exist?"

Example:

```yaml
roles:

  - role: nginx
    vars:
      nginx_port: 8080
```

Inside the role:

```jinja2
{{ nginx_port }}
```

works.

So:

```text
Playbook
   │
   │ nginx_port=8080
   ▼
Role
   │
   ▼
Tasks/templates
   │
   ▼
{{ nginx_port }}
```

---

# 28. 🎯 Role-Specific Variable Naming

Good:

```text
nginx_port
nginx_worker_processes
postgresql_port
postgresql_version
patroni_cluster_name
```

Less desirable:

```text
port
version
name
```

Why?

Because many roles may use those generic names.

Role-specific prefixes reduce collisions.

---

# 29. 🧠 `public` With Role Inclusion

This is a more advanced concept.

You may see:

```yaml
include_role:
  name: nginx
  public: true
```

or:

```yaml
import_role:
  name: nginx
```

The `public` option affects whether variables from the role's `vars` and `defaults` become available to the rest of the play.

The important distinction:

```text
public role variables
        ↓
can become visible outside the role
```

while private role inclusion can keep them from being exposed in the same way.

You don't need to memorize every nuance yet, but understand that **role variable exposure is controllable**.

---

# 30. 🧠 Why Does Variable Exposure Matter?

Imagine:

```text
Role A
  ↓
defines internal variables

Role B
  ↓
might accidentally depend on them
```

This creates coupling.

Good role design tries to keep:

```text
Role internals
      ↓
encapsulated
```

and expose only intentional inputs/outputs.

This is similar to good software engineering.

---

# 31. 🏗️ Think of Roles Like Functions

A useful mental model:

```text
Role
 =
Reusable function/module
```

For example:

```text
postgresql(
    version=17,
    port=5432
)
```

Then:

```text
myapp(
    database=postgresql
)
```

The analogy isn't exact, but it helps you understand why roles should be:

* reusable
* parameterized
* loosely coupled
* responsible for one component

---

# 32. 🔥 Production Composition

Imagine your environment:

```text
                  site.yml
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     common       database      monitoring
                     │
                     ▼
                  patroni
                     │
                     ▼
                postgresql
```

One possible implementation:

```text
database role
   ↓
depends on PostgreSQL

patroni role
   ↓
depends on PostgreSQL

monitoring role
   ↓
depends on exporters
```

Now your playbook becomes an orchestration layer instead of a huge implementation file.

---

# 33. 🎯 `include_role` — When I Would Use It

Good use case:

```yaml
- name: Configure monitoring
  ansible.builtin.include_role:
    name: monitoring
  when: monitoring_enabled
```

Another:

```yaml
- name: Configure database
  ansible.builtin.include_role:
    name: postgresql
  when: "'database' in group_names"
```

This makes dynamic execution easy.

---

# 34. 🎯 `import_role` — When I Would Use It

Use static import when you want role content treated as part of the playbook structure earlier.

Example:

```yaml
tasks:

  - name: Import common role
    ansible.builtin.import_role:
      name: common
```

It can be useful when working with:

* static structure
* predictable parsing
* tag behavior
* task listing

---

# 35. 🆚 `include_role` vs `import_role`

This table is important:

| Feature                        | `include_role`   | `import_role` |
| ------------------------------ | ---------------- | ------------- |
| Processing                     | Dynamic          | Static        |
| Loaded                         | During execution | Earlier       |
| Good for conditional inclusion | ✅                | More limited  |
| Can be used as a task          | ✅                | ✅             |
| Typical use                    | Dynamic reuse    | Static reuse  |

Mental model:

```text
include_role
   ↓
"Decide at runtime."


import_role
   ↓
"Know this structure earlier."
```

---

# 36. 🧠 `include_tasks` vs `include_role`

Don't confuse:

```text
include_tasks
```

with:

```text
include_role
```

### `include_tasks`

Loads:

```text
tasks.yml
```

### `include_role`

Loads a whole role structure:

```text
tasks
handlers
defaults
vars
templates
files
meta
```

Conceptually:

```text
include_tasks
    ↓
specific task file


include_role
    ↓
whole reusable role
```

---

# 37. 🧠 Role Dependencies vs `include_role`

Dependencies:

```text
meta/main.yml
```

mean:

> "This role requires another role."

`include_role` means:

> "At this point in execution, execute this role."

So:

```text
Dependency
    =
architectural requirement


include_role
    =
execution decision
```

Very useful distinction.

---

# 38. 🏭 Production Example

Suppose:

```text
site.yml
```

contains:

```yaml
---
- name: Configure AlloyDB nodes
  hosts: database
  become: true

  roles:
    - common
    - postgresql
    - patroni
    - haproxy
    - monitoring
```

Now perhaps:

```text
patroni/meta/main.yml
```

contains:

```yaml
dependencies:
  - role: postgresql
```

Then you have:

```text
site.yml
 │
 ├── common
 │
 ├── postgresql
 │
 ├── patroni
 │      │
 │      └── postgresql dependency
 │
 ├── haproxy
 │
 └── monitoring
```

Ansible handles role execution according to the dependency graph and role ordering rules.

---

# 39. ⚠️ Don't Over-Engineer Dependencies

Suppose:

```text
common
 ↓
packages
 ↓
users
 ↓
directories
 ↓
postgres
 ↓
patroni
 ↓
monitoring
```

Everything depending on everything else becomes difficult to reason about.

Instead:

```text
common
postgres
patroni → postgres
monitoring
haproxy
```

Keep relationships meaningful.

---

# 40. 🎤 Interview Questions

### Q1. What is a role dependency?

> A role dependency declares another role that must be executed as a prerequisite of the current role, usually through `meta/main.yml`.

---

### Q2. Where do you define role dependencies?

```text
roles/<role>/meta/main.yml
```

Example:

```yaml
dependencies:
  - role: postgresql
```

---

### Q3. Difference between `include_role` and `import_role`?

> `include_role` is dynamic and evaluated during execution, while `import_role` is static and processed earlier as part of playbook parsing.

---

### Q4. When would you use `include_role`?

> When I need dynamic behavior such as conditionally executing a role or selecting it during task execution.

---

### Q5. Why use `defaults/main.yml`?

> To provide configurable default values that users or environments can easily override.

---

### Q6. Why use `vars/main.yml`?

> For role-specific variables that generally have higher precedence and are intended to be more controlled by the role.

---

### Q7. Can a role be reused multiple times?

Yes, provided the role is designed for it and Ansible's role de-duplication behavior is accounted for.

---

### Q8. Why use role dependencies?

> To make intrinsic prerequisites explicit and allow a reusable component to declare what it requires.

---

# 41. 🏆 Senior-Level Mental Model

You should now see Ansible Roles like this:

```text
                         PLAYBOOK
                            │
                     orchestration
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       common           postgresql        monitoring
                            │
                            │ dependency
                            ▼
                         patroni
                            │
                       uses variables
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           defaults        vars        templates
              │
              ▼
         configuration
```

And:

```text
📦 Role
 │
 ├── tasks       → implementation
 ├── handlers    → reactions
 ├── defaults    → public/configurable defaults
 ├── vars        → role-controlled variables
 ├── templates   → dynamic files
 ├── files       → static files
 └── meta        → dependencies/metadata
```

---

# 🎯 The 5 Things You Should Remember

If you don't remember everything from this chapter, remember these five:

### 1️⃣ Dependencies

```yaml
# meta/main.yml

dependencies:
  - role: postgresql
```

---

### 2️⃣ Configurable role inputs

```yaml
# defaults/main.yml

postgresql_version: "17"
```

---

### 3️⃣ Dynamic role

```yaml
- ansible.builtin.include_role:
    name: postgresql
```

---

### 4️⃣ Static role

```yaml
- ansible.builtin.import_role:
    name: postgresql
```

---

### 5️⃣ The difference

```text
roles:
  → normal role usage

include_role:
  → dynamic

import_role:
  → static
```

---


# 📦 Ansible Topic 13 — Collections & `ansible-galaxy`

You already know **roles**. Now we're moving one level higher:

```text
Task
  ↓
Role
  ↓
Collection
```

Collections are important because modern Ansible heavily uses **Fully Qualified Collection Names (FQCN)** such as:

```yaml
ansible.builtin.copy
ansible.posix.mount
community.general.xxx
```

For LevelUp, you should be very comfortable with this topic.

---

# 1. 🧠 What is an Ansible Collection?

An **Ansible Collection** is a distributable package of Ansible content.

A collection can contain:

```text
📦 Collection
│
├── Modules
├── Plugins
├── Roles
├── Playbooks
├── Module utilities
└── Documentation
```

So compared with a role:

```text
Role
 ↓
One reusable automation component
```

while:

```text
Collection
 ↓
A package containing many types of Ansible content
```

---

# 2. 🏗️ Visualize the Hierarchy

Think:

```text
                    📦 COLLECTION
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
      Roles           Modules          Plugins
        │                │                │
        ▼                ▼                ▼
    postgres          custom_module     lookup
    patroni           cloud_module      filter
    monitoring        network_module    callback
```

A collection is therefore **much broader than a role**.

---

# 3. 🤔 Why Were Collections Introduced?

Historically, Ansible content was distributed in several ways.

As Ansible grew:

```text
More modules
More plugins
More roles
More vendors
More communities
```

it became necessary to provide a standardized packaging mechanism.

Collections provide:

> **A consistent way to package, version, distribute and consume Ansible automation content.**

---

# 4. 🆚 Role vs Collection

This is an important interview comparison.

| Role                                 | Collection                                |
| ------------------------------------ | ----------------------------------------- |
| Reusable automation component        | Distribution/package of Ansible content   |
| Mainly tasks/handlers/templates/etc. | Can contain roles, modules, plugins, etc. |
| Narrow scope                         | Broader scope                             |
| Usually one component                | Can contain many components               |

Example:

```text
Role:
postgresql
```

Collection:

```text
community.general
```

could contain many different modules/plugins/roles.

---

# 5. 🎯 Why Are You Seeing `ansible.builtin.copy`?

You previously saw:

```yaml
ansible.builtin.copy:
```

instead of:

```yaml
copy:
```

This is called an **FQCN**.

FQCN =

> **Fully Qualified Collection Name**

Format:

```text
namespace.collection.resource
```

Example:

```text
ansible.builtin.copy
│       │       │
│       │       └── resource/module
│       └── collection
└── namespace
```

---

# 6. 🧠 FQCN Examples

```yaml
ansible.builtin.copy
```

```yaml
ansible.builtin.template
```

```yaml
ansible.builtin.service
```

```yaml
ansible.posix.mount
```

```yaml
community.general.archive
```

Each tells Ansible exactly where the content comes from.

---

# 7. 🔍 Why Not Just Use `copy`?

You can encounter short names:

```yaml
copy:
```

But modern best practice is:

```yaml
ansible.builtin.copy:
```

because it is explicit.

Imagine two collections contain resources with the same name:

```text
collectionA.copy
collectionB.copy
```

Which one should Ansible use?

FQCN removes ambiguity.

```text
ansible.builtin.copy
```

means:

> Use the `copy` module from the `ansible.builtin` collection.

---

# 8. 🧠 `ansible.builtin`

This is extremely important.

```text
ansible.builtin
```

is the collection containing Ansible's built-in content.

Examples:

```yaml
ansible.builtin.copy
ansible.builtin.file
ansible.builtin.template
ansible.builtin.package
ansible.builtin.service
ansible.builtin.command
ansible.builtin.shell
ansible.builtin.user
```

So:

```yaml
ansible.builtin.copy:
```

means:

> Use Ansible's built-in `copy` module.

---

# 9. 🧩 `ansible.posix`

Another example:

```yaml
ansible.posix.mount:
```

Here:

```text
ansible
   ↓
namespace

posix
   ↓
collection

mount
   ↓
module
```

So:

```text
ansible.posix.mount
```

means:

> `mount` module from the `ansible.posix` collection.

---

# 10. 🌎 `community.general`

You will frequently encounter:

```yaml
community.general.some_module:
```

Structure:

```text
community
    ↓
namespace

general
    ↓
collection

some_module
    ↓
resource
```

The `community.general` collection contains a broad set of community-maintained Ansible content.

---

# 11. 🏆 Why FQCN Is Important for LevelUp

If an interviewer asks:

> "What is FQCN?"

You should be able to immediately say:

> "FQCN stands for Fully Qualified Collection Name. It identifies Ansible content using the namespace, collection and resource name, for example `ansible.builtin.copy`. It avoids ambiguity and is the recommended explicit way to reference Ansible modules and other collection content."

That's a good answer.

---

# 12. 📦 Installing a Collection

You can install a collection using:

```bash
ansible-galaxy collection install community.general
```

Ansible downloads and installs it into the configured collection path.

Then you can use:

```yaml
community.general.some_module:
```

---

# 13. 🧠 What is `ansible-galaxy`?

`ansible-galaxy` is the command-line interface for working with Ansible content.

It can work with:

```text
Roles
Collections
Dependencies
```

Examples:

```bash
ansible-galaxy role init myrole
```

```bash
ansible-galaxy collection install community.general
```

So:

```text
ansible-galaxy
      │
      ├── roles
      └── collections
```

---

# 14. 🔎 Search for Collections

You can search Galaxy content using:

```bash
ansible-galaxy collection search
```

For example:

```bash
ansible-galaxy collection search postgresql
```

This helps discover available collections.

---

# 15. 📋 List Installed Collections

Use:

```bash
ansible-galaxy collection list
```

You'll see installed collections and their versions.

Conceptually:

```text
Collection                     Version
---------------------------------------
ansible.builtin                ...
ansible.posix                  ...
community.general              ...
```

This is useful when troubleshooting:

> "Which collection version is actually installed on this automation node?"

---

# 16. 🧠 Why Version Matters

Imagine your playbook uses:

```yaml
community.general.some_module:
```

Today:

```text
collection version = 10.x
```

Tomorrow:

```text
collection version = 11.x
```

The module behavior or parameters could potentially change.

Therefore, production automation should avoid blindly relying on whatever version happens to be installed.

---

# 17. 🔒 Version Pinning

You can specify a collection version:

```bash
ansible-galaxy collection install community.general:==10.5.0
```

The exact version depends on the collection's available releases.

The important production principle:

> **Pin versions when reproducibility matters.**

---

# 18. 📄 `requirements.yml`

Instead of manually installing collections one by one:

```bash
ansible-galaxy collection install ...
ansible-galaxy collection install ...
ansible-galaxy collection install ...
```

you can create:

```text
requirements.yml
```

Example:

```yaml
---
collections:

  - name: ansible.posix

  - name: community.general

  - name: community.crypto
```

Then:

```bash
ansible-galaxy collection install -r requirements.yml
```

Ansible installs everything listed.

---

# 19. 🏭 Production Dependency Management

A production repository might have:

```text
ansible-project/
│
├── requirements.yml
├── ansible.cfg
├── inventories/
├── playbooks/
└── roles/
```

Then CI/CD can do:

```text
requirements.yml
       ↓
install dependencies
       ↓
run lint/tests
       ↓
run playbook
```

This makes the environment reproducible.

---

# 20. 🔒 Pinning in `requirements.yml`

You can specify versions.

Example:

```yaml
---
collections:

  - name: ansible.posix
    version: ">=1.5.0,<2.0.0"

  - name: community.general
    version: ">=10.0.0,<11.0.0"
```

The exact constraint syntax can depend on the source/versioning mechanism, but the key idea is:

> Define the expected dependency version rather than allowing arbitrary upgrades.

---

# 21. 🎯 Why `requirements.yml` Is Better

Suppose a new engineer joins your project.

Without:

```text
requirements.yml
```

they might have:

```text
Ansible 2.x
Collection X version 1
Collection Y version 8
```

while your machine has:

```text
Collection X version 2
Collection Y version 9
```

Result:

```text
"It works on my machine." 😵
```

With:

```text
requirements.yml
```

everyone installs the expected dependencies.

---

# 22. 🧱 Collection Directory Structure

A collection has a standardized structure.

Conceptually:

```text
namespace/
└── collection/
    ├── galaxy.yml
    ├── plugins/
    │   ├── modules/
    │   ├── lookup/
    │   ├── filter/
    │   └── callback/
    │
    ├── roles/
    │
    ├── playbooks/
    │
    └── README.md
```

A collection can therefore package many types of content.

---

# 23. 📄 `galaxy.yml`

A collection contains:

```text
galaxy.yml
```

This is collection metadata.

It can contain information such as:

```yaml
namespace: mycompany
name: infrastructure
version: 1.0.0
authors:
  - My Company
description: Infrastructure automation
```

Conceptually:

```text
galaxy.yml
   ↓
Collection metadata
```

---

# 24. 🏢 Private Company Collections

This is especially relevant to enterprise environments.

A company might create:

```text
mycompany.infrastructure
```

containing:

```text
roles/
modules/
plugins/
```

Then internal teams can use:

```yaml
mycompany.infrastructure.some_module:
```

instead of copying automation between repositories.

Architecture:

```text
             🏢 Internal Collection
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Roles      Modules    Plugins
          │
          ▼
     Multiple Projects
```

This is a very useful enterprise pattern.

---

# 25. 🔐 Private Automation Hub

Organizations may not want to download all automation from public Galaxy.

Instead:

```text
Developer
    ↓
CI/CD
    ↓
Private Automation Hub
    ↓
Internal Collections
```

A private automation repository can provide:

* approved collections
* approved versions
* internal roles
* security controls
* organization-specific automation

For enterprise Ansible environments, this is important architecture knowledge.

---

# 26. 🌐 Galaxy vs Private Repository

Think:

```text
Ansible Galaxy
      ↓
Public/community content


Private Automation Hub
      ↓
Enterprise/internal content
```

You may have:

```text
Public:
community.general

Private:
mycompany.infrastructure
```

---

# 27. 🧠 Why Collections Matter in CI/CD

A production pipeline might look like:

```text
Git commit
    ↓
CI pipeline
    ↓
Create clean automation environment
    ↓
Install requirements.yml
    ↓
ansible-lint
    ↓
Molecule/tests
    ↓
Ansible execution
```

Notice:

```text
requirements.yml
```

is part of reproducibility.

---

# 28. 🧪 `ansible-galaxy collection install`

Basic:

```bash
ansible-galaxy collection install community.general
```

From requirements:

```bash
ansible-galaxy collection install -r requirements.yml
```

To a specific path:

```bash
ansible-galaxy collection install \
  -r requirements.yml \
  -p ./collections
```

Then your project can have:

```text
ansible-project/
└── collections/
    └── ansible_collections/
        └── community/
            └── general/
```

---

# 29. 🧠 Why Use a Local Collection Path?

This can be useful in CI/CD or isolated environments.

Instead of relying entirely on a global installation:

```text
system
 └── collections
```

you can keep dependencies within the project/environment.

This helps with:

* reproducibility
* isolation
* CI environments
* offline workflows

---

# 30. ⚙️ `ansible.cfg` and Collection Paths

You can configure collection search paths in `ansible.cfg`.

Conceptually:

```ini
[defaults]
collections_path = ./collections:/usr/share/ansible/collections
```

This tells Ansible where to look for collections.

The exact path syntax can vary by platform/environment.

---

# 31. 🧠 Collection Resolution

When you write:

```yaml
community.general.some_module:
```

Ansible needs to find:

```text
community
   ↓
general
   ↓
some_module
```

It searches configured collection paths.

Visual:

```text
FQCN
 │
 ▼
collection search paths
 │
 ├── project collections
 ├── user collections
 └── system collections
       │
       ▼
community.general
       │
       ▼
some_module
```

---

# 32. 🚨 Why FQCN Helps Security & Reliability

Using:

```yaml
copy:
```

can be ambiguous.

Using:

```yaml
ansible.builtin.copy:
```

is explicit.

You know:

```text
namespace = ansible
collection = builtin
resource = copy
```

This makes code easier to review.

For production code, I'd recommend consistently using FQCN.

---

# 33. 🧠 FQCN Is Not Only for Modules

You can encounter FQCNs for other content types too.

For example:

```yaml
ansible.builtin.template:
```

is a module.

But collections can also provide:

```text
lookup plugins
filter plugins
callback plugins
connection plugins
inventory plugins
roles
```

So FQCN is fundamentally a way to identify collection content.

---

# 34. 🔥 Built-in vs Community

You may see:

```yaml
ansible.builtin.copy:
```

and:

```yaml
community.general.archive:
```

Think:

```text
ansible.builtin
      ↓
Core/built-in Ansible content


community.general
      ↓
Community-maintained collection
```

Don't assume "community" means unofficial or unsafe.

But in production:

> Always check the collection's source, maintenance, version, compatibility and security posture before adopting it.

---

# 35. 🏭 Production Dependency File

A realistic project could have:

```yaml
---
collections:

  - name: ansible.posix
    version: "1.5.4"

  - name: community.general
    version: "10.3.0"

  - name: community.crypto
    version: "2.22.0"
```

Then:

```bash
ansible-galaxy collection install -r requirements.yml
```

Your CI pipeline can reproduce the same dependency environment.

---

# 36. 🎯 Collection vs Role vs Module

This is probably the most important conceptual hierarchy:

```text
                         COLLECTION
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
            ROLE           MODULE           PLUGIN
              │               │
              ▼               ▼
           tasks/        performs action
           handlers/
           templates/
```

So:

### Module

> Performs an individual operation.

```yaml
ansible.builtin.copy:
```

### Role

> Packages reusable automation around a component.

```text
postgresql role
```

### Collection

> Packages and distributes roles, modules, plugins and other Ansible content.

---

# 37. 🧠 Example: Your PostgreSQL Automation

Imagine you create:

```text
mycompany.database
```

collection.

Inside:

```text
mycompany.database/
│
├── roles/
│   ├── postgresql/
│   └── patroni/
│
├── plugins/
│   └── modules/
│       └── database_health.py
│
└── galaxy.yml
```

Then another project could use:

```yaml
roles:
  - mycompany.database.postgresql
```

or a module:

```yaml
mycompany.database.database_health:
```

This is a powerful enterprise packaging model.

---

# 38. 🔥 Why This Is Relevant to Your Work

You work heavily with:

```text
Ansible
RPM validation
PostgreSQL
Patroni
ETCD
HAProxy
Monitoring
```

A mature organization could package common automation into internal collections.

For example:

```text
epam.infrastructure
```

might contain:

```text
roles/
├── rhel_baseline
├── postgres
├── patroni
├── etcd
├── haproxy
└── monitoring
```

Then different projects can consume the same approved automation.

---

# 39. 🎤 Interview Questions

### Q1. What is an Ansible collection?

> A collection is a distributable package of Ansible content such as roles, modules, plugins and playbooks.

---

### Q2. What is FQCN?

> Fully Qualified Collection Name. It identifies content using namespace, collection and resource, for example `ansible.builtin.copy`.

---

### Q3. Why use FQCN?

> It makes the source of the Ansible content explicit, avoids ambiguity and is the recommended modern style.

---

### Q4. What is `ansible.builtin`?

> The built-in Ansible collection containing core Ansible content.

---

### Q5. How do you install a collection?

```bash
ansible-galaxy collection install community.general
```

---

### Q6. How do you install dependencies from a file?

```bash
ansible-galaxy collection install -r requirements.yml
```

---

### Q7. What is `requirements.yml`?

> A dependency definition file used to specify Ansible roles and/or collections required by a project.

---

### Q8. Why pin collection versions?

> To improve reproducibility and prevent unexpected behavior caused by dependency upgrades.

---

### Q9. How do you list installed collections?

```bash
ansible-galaxy collection list
```

---

### Q10. Difference between Role and Collection?

> A role is a reusable automation unit, while a collection is a broader package that can contain roles, modules, plugins and other Ansible content.

---

# 40. 🏆 Production Architecture

A mature Ansible environment can look like:

```text
                       Git Repository
                             │
                             ▼
                     Ansible Project
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
        requirements.yml   roles/         playbooks/
             │
             ▼
      Dependency Install
             │
       ┌─────┴──────┐
       ▼            ▼
 Public Galaxy   Private Hub
       │            │
       └──────┬─────┘
              ▼
         Collections
              │
      ┌───────┼────────┐
      ▼       ▼        ▼
    Roles   Modules  Plugins
              │
              ▼
         Ansible Runtime
              │
              ▼
        Managed Nodes
```

That's the big picture.

---

# 41. 🚨 Important Production Practices

### ✅ Use FQCN

Prefer:

```yaml
ansible.builtin.copy:
```

over:

```yaml
copy:
```

---

### ✅ Track dependencies

Keep:

```text
requirements.yml
```

in Git.

---

### ✅ Pin versions where appropriate

Avoid:

```text
"whatever is latest today"
```

for production.

---

### ✅ Test collection upgrades

Don't blindly upgrade:

```text
community.general
```

on production.

Test first.

---

### ✅ Use approved internal collections

Enterprise environments may maintain:

```text
private automation hub
```

for approved content.

---

# 42. 🧠 Final Mental Model

Remember:

```text
                    📦 COLLECTION
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
       ROLE            MODULE          PLUGIN
         │               │               │
         ▼               ▼               ▼
     reusable        one action      extension
     automation
```

And:

```text
FQCN
 │
 ├── namespace
 │
 ├── collection
 │
 └── resource
```

Example:

```text
ansible.builtin.copy
│       │       │
│       │       └── copy
│       └────────── builtin
└────────────────── ansible
```

---

# 🎯 Your Ansible Hierarchy So Far

You should now be able to visualize the whole thing:

```text
                    🧠 ANSIBLE
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
          PLAYBOOK              COLLECTION
             │                     │
       orchestration       ┌────────┼────────┐
                           ▼        ▼        ▼
                         ROLE    MODULE    PLUGIN
                           │        │
                           ▼        ▼
                        TASK     ACTION
                           │
                           ▼
                        HOST
```

And for dependency management:

```text
requirements.yml
       │
       ▼
ansible-galaxy
       │
       ▼
Install collections
       │
       ▼
Ansible project
       │
       ▼
CI/CD
       │
       ▼
Production
```

---


# 🔐 Ansible Topic 14 — Ansible Vault & Secrets Management

This is an **important production topic** because Ansible frequently needs sensitive information:

```text
🔑 SSH/private credentials
🔐 Database passwords
🔐 API tokens
🔐 TLS private keys
🔐 Service credentials
🔐 Cloud credentials
```

You should **never casually put these values in plain-text YAML and commit them to Git**.

Ansible Vault provides encryption for sensitive Ansible data.

---

# 1. 🧠 What is Ansible Vault?

**Ansible Vault is a feature for encrypting sensitive Ansible data at rest.**

For example, instead of:

```yaml
db_password: MySecretPassword123
```

you can store the value in an encrypted Vault file.

Conceptually:

```text
                 Plain Secret
                      │
                      ▼
                🔐 ansible-vault
                      │
                      ▼
               Encrypted File
                      │
                      ▼
                    Git
```

The important point:

> **Vault protects the secret while it is stored.**

It does **not** automatically make your entire Ansible execution secure.

---

# 2. 🚨 Why Do We Need Vault?

Imagine this:

```yaml
---
db_user: postgres
db_password: SuperSecret123
api_token: abc123xyz
```

You commit:

```bash
git add .
git commit -m "Add database configuration"
git push
```

Now your secret is sitting in Git history. 😨

Even if you later delete the line, the secret may remain in:

```text
Git history
CI logs
Backups
Developer clones
Pull request history
```

Instead:

```text
Git repository
     │
     ▼
Encrypted Vault file
     │
     ▼
Only authorized execution environment
     │
     ▼
Decrypted at runtime
```

---

# 3. 🏗️ Basic Vault Architecture

```text
                  🔐 Vault Password
                         │
                         ▼
                  Ansible Vault
                         │
              encrypt / decrypt
                         │
                         ▼
                Encrypted Secret
                         │
                         ▼
                       Git
```

During execution:

```text
Encrypted Secret
      +
Vault Password
      │
      ▼
Ansible
      │
      ▼
Decrypted value in memory
      │
      ▼
Module
      │
      ▼
Managed Node
```

---

# 4. 🔐 Create a Vault File

The most basic command:

```bash
ansible-vault create secrets.yml
```

Ansible asks for a password.

Then you can enter:

```yaml
---
db_username: postgres
db_password: SuperSecret123
api_token: abc123xyz
```

When you save the file, it is encrypted.

---

# 5. 👀 What Does the File Look Like?

Instead of seeing:

```yaml
db_password: SuperSecret123
```

the file looks something like:

```text
$ANSIBLE_VAULT;1.1;AES256
663362663034...
343135323538...
...
```

You can safely store this encrypted file in Git **from a confidentiality perspective**, assuming your vault password/key is kept separately and securely.

---

# 6. 🧠 Important: Vault File ≠ Plain YAML

You can still conceptually think:

```text
secrets.yml
```

contains YAML.

But on disk:

```text
secrets.yml
     ↓
encrypted
```

When Ansible uses it:

```text
encrypted YAML
     ↓
decrypt
     ↓
variables
```

---

# 7. ▶️ Using Vault Variables

Suppose:

```text
group_vars/
└── production/
    ├── vars.yml
    └── vault.yml
```

`vault.yml`:

```yaml
---
db_password: SuperSecret123
```

But the actual file is encrypted using:

```bash
ansible-vault create vault.yml
```

Then your playbook can use:

```yaml
- name: Configure database
  ansible.builtin.template:
    src: database.conf.j2
    dest: /etc/myapp/database.conf
```

Template:

```jinja2
database_password={{ db_password }}
```

Ansible decrypts the Vault data and makes the variable available.

---

# 8. 🔑 Running a Playbook With Vault

If Ansible needs the Vault password, you can use:

```bash
ansible-playbook site.yml --ask-vault-pass
```

You'll be prompted:

```text
Vault password:
```

Then Ansible decrypts the required data.

---

# 9. 🧠 Important Security Concept

Your Vault password should **not** be stored inside:

```text
vault.yml
```

Otherwise:

```text
Encrypted secret
      +
Password stored beside it
      =
Not useful
```

Keep the decryption credential separately.

---

# 10. 🔍 View an Encrypted Vault File

You can inspect it in decrypted form without modifying it:

```bash
ansible-vault view secrets.yml
```

It asks for the Vault password.

You see:

```yaml
db_password: SuperSecret123
```

but the file remains encrypted on disk.

---

# 11. ✏️ Edit a Vault File

Use:

```bash
ansible-vault edit secrets.yml
```

Ansible:

```text
Encrypted file
     ↓
Decrypt temporarily
     ↓
Open editor
     ↓
You modify it
     ↓
Encrypt again
```

You don't need to manually decrypt and re-encrypt it.

---

# 12. 🔓 Decrypt a Vault File

You can permanently decrypt:

```bash
ansible-vault decrypt secrets.yml
```

The file becomes normal YAML again.

Example:

```yaml
db_password: SuperSecret123
```

⚠️ Be careful.

You've now removed the encryption.

---

# 13. 🔐 Encrypt an Existing File

Suppose you already have:

```text
secrets.yml
```

containing:

```yaml
db_password: SuperSecret123
```

You can encrypt it:

```bash
ansible-vault encrypt secrets.yml
```

Now it becomes a Vault-encrypted file.

---

# 14. 🧠 Basic Vault Command Cheat Sheet

| Command                 | Purpose                 |
| ----------------------- | ----------------------- |
| `ansible-vault create`  | Create encrypted file   |
| `ansible-vault view`    | View decrypted contents |
| `ansible-vault edit`    | Edit encrypted file     |
| `ansible-vault encrypt` | Encrypt existing file   |
| `ansible-vault decrypt` | Decrypt file            |
| `ansible-vault rekey`   | Change Vault password   |

Memorize these. 🎯

---

# 15. 🔄 `rekey`

Suppose your Vault password is:

```text
OldPassword
```

You want:

```text
NewPassword
```

Use:

```bash
ansible-vault rekey secrets.yml
```

Ansible asks for:

```text
Old password:
New password:
```

The file remains encrypted but uses the new Vault password.

---

# 16. 🧩 Separate Normal Variables From Secrets

A good production structure is:

```text
group_vars/
└── production/
    ├── vars.yml
    └── vault.yml
```

### `vars.yml`

```yaml
---
app_port: 8080
app_name: myapp
db_host: postgres01
```

### `vault.yml`

```yaml
---
db_password: ...
api_token: ...
```

Then:

```text
Normal configuration
        +
Encrypted secrets
```

This makes it very obvious which values are sensitive.

---

# 17. 🏭 Production Repository Example

```text
ansible-project/
│
├── inventories/
│   └── production/
│       ├── hosts.yml
│       └── group_vars/
│           ├── all/
│           │   ├── vars.yml
│           │   └── vault.yml
│           │
│           └── database/
│               ├── vars.yml
│               └── vault.yml
│
├── roles/
│   ├── postgresql/
│   ├── patroni/
│   └── monitoring/
│
├── playbooks/
│   └── site.yml
│
└── requirements.yml
```

The encrypted Vault files can be committed to Git.

The Vault password should be managed separately.

---

# 18. 🔥 A Very Important Pattern

You might see:

```yaml
# vars.yml

db_host: postgres01
db_port: 5432
```

and:

```yaml
# vault.yml

db_username: postgres
db_password: <encrypted>
```

Then application code can simply use:

```jinja2
host={{ db_host }}
port={{ db_port }}
username={{ db_username }}
password={{ db_password }}
```

The application doesn't need to know whether the value came from Vault.

---

# 19. 🧠 Vault Encrypts Data, Not Variable Names

Suppose:

```yaml
db_password: SuperSecret123
```

Vault encrypts the file.

Someone looking at:

```text
$ANSIBLE_VAULT...
```

cannot simply read:

```text
db_password
```

from the encrypted content.

The purpose is confidentiality of the encrypted data.

---

# 20. 🔐 Vault Password File

Instead of manually typing:

```bash
--ask-vault-pass
```

you can provide a password file:

```bash
ansible-playbook site.yml \
  --vault-password-file ~/.ansible/vault_password
```

For example:

```text
~/.ansible/vault_password
```

contains the Vault password.

⚠️ But this file itself becomes a secret.

Do **not** commit it to Git.

---

# 21. 🚨 Don't Do This

Never do:

```text
ansible-project/
├── vault.yml
└── vault_password
```

and commit both.

You'd have:

```text
Encrypted data
      +
Decryption password
      ↓
Anyone with repository access can decrypt it
```

That's a serious security mistake.

---

# 22. 🏭 Better CI/CD Pattern

In CI/CD:

```text
Git
 │
 ├── playbooks
 ├── roles
 ├── requirements.yml
 └── encrypted vault.yml
             │
             ▼
          CI secret
             │
             ▼
       Vault password
             │
             ▼
       Ansible execution
```

The Vault password comes from the CI/CD platform's **secret store**, not from Git.

---

# 23. 🔥 Example CI/CD Flow

```text
Developer
    │
    ▼
Git commit
    │
    ├── playbook
    ├── role
    └── encrypted vault
              │
              ▼
         CI Pipeline
              │
              ▼
       Retrieve secret
              │
              ▼
     Vault password injected
              │
              ▼
     ansible-playbook
              │
              ▼
        Managed nodes
```

This is a much better architecture.

---

# 24. 🧠 What If I Use GCP?

Since you're preparing GCP, you should understand this distinction.

You have:

```text
Ansible Vault
```

and:

```text
GCP Secret Manager
```

They solve related but different problems.

### Ansible Vault

Primarily:

> Encrypt sensitive Ansible data stored in files.

### GCP Secret Manager

Primarily:

> Centralized secret storage and retrieval for applications/infrastructure.

---

# 25. 🆚 Ansible Vault vs Secret Manager

| Feature                              | Ansible Vault                | Secret Manager                             |
| ------------------------------------ | ---------------------------- | ------------------------------------------ |
| Encrypt secrets at rest              | ✅                            | ✅                                          |
| Store encrypted data in Git          | ✅                            | ❌ Usually secret value isn't stored in Git |
| Centralized secret service           | ❌                            | ✅                                          |
| Runtime secret retrieval             | Limited                      | ✅                                          |
| Secret rotation                      | Manual/application-dependent | Better suited                              |
| Audit/access control                 | Limited                      | Stronger centralized controls              |
| CI/CD integration                    | ✅                            | ✅                                          |
| Good for static Ansible variables    | ✅                            | —                                          |
| Good for application runtime secrets | Less ideal                   | ✅                                          |

---

# 26. 🏆 Production Architecture With GCP

A stronger enterprise design can be:

```text
                   Git
                    │
          ┌─────────┴─────────┐
          │                   │
      Ansible Code        No plaintext
                           secrets
                              │
                              ▼
                       CI/CD Pipeline
                              │
                              ▼
                    GCP Secret Manager
                              │
                       Retrieve secret
                              │
                              ▼
                          Ansible
                              │
                              ▼
                       Managed nodes
```

Depending on the environment, you might not need Vault at all if secrets are retrieved dynamically from a centralized secret manager.

---

# 27. 🧠 When Should I Use Ansible Vault?

Good use cases:

```text
Static secrets required by Ansible
Encrypted variables
Encrypted inventory/group_vars
Secrets that need to live alongside automation code
```

Example:

```yaml
db_password: ...
```

---

# 28. 🧠 When Should I Prefer Secret Manager?

If the requirement is:

```text
Centralized secrets
Rotation
Auditing
Access policies
Runtime retrieval
Multiple applications consuming same secret
```

then a dedicated secret-management system is generally better.

Examples:

```text
GCP Secret Manager
HashiCorp Vault
AWS Secrets Manager
Azure Key Vault
```

---

# 29. 🔐 Multiple Vault Passwords — Vault IDs

Now we get into an important advanced feature.

Suppose you have:

```text
development secrets
staging secrets
production secrets
```

You don't necessarily want one password for everything.

You can use **Vault IDs**.

Conceptually:

```text
dev     → dev password
staging → staging password
prod    → prod password
```

---

# 30. 🏷️ Vault ID Concept

You might create:

```bash
ansible-vault encrypt \
  --vault-id dev@prompt \
  dev-vault.yml
```

And:

```bash
ansible-vault encrypt \
  --vault-id prod@prompt \
  prod-vault.yml
```

Now each encrypted file is associated with a Vault ID.

---

# 31. 🧠 Why Vault IDs?

Imagine:

```text
Development
     │
     └── dev secrets


Production
     │
     └── prod secrets
```

Using separate Vault credentials provides better separation.

If the development Vault password is compromised:

```text
Production secrets
      ↓
still protected by
      ↓
different Vault credential
```

That's much better than one password for every environment.

---

# 32. 🔥 Example

Create production Vault:

```bash
ansible-vault create \
  --vault-id prod@prompt \
  prod-vault.yml
```

Create development Vault:

```bash
ansible-vault create \
  --vault-id dev@prompt \
  dev-vault.yml
```

During execution:

```bash
ansible-playbook site.yml \
  --vault-id dev@prompt \
  --vault-id prod@prompt
```

Ansible can use the appropriate Vault credentials.

---

# 33. 🧠 Vault ID Architecture

```text
                 Ansible
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      dev-vault.yml       prod-vault.yml
          │                   │
       🔐 dev              🔐 prod
          │                   │
      dev password       prod password
```

This gives environment-level separation.

---

# 34. 🔒 Vault Encryption Is Not Secret Redaction

Very important.

Suppose your secret is:

```yaml
db_password: SuperSecret123
```

Vault protects the value **at rest**.

But if your playbook does:

```yaml
- name: Print password
  ansible.builtin.debug:
    var: db_password
```

you can expose the secret in output.

So:

> **Vault does not automatically protect you from leaking secrets through logs.**

---

# 35. 🚨 `no_log: true`

For sensitive tasks:

```yaml
- name: Configure database password
  ansible.builtin.command:
    cmd: "/opt/myapp/set-password {{ db_password }}"
  no_log: true
```

This prevents Ansible from displaying sensitive task information in normal output.

This is extremely important.

---

# 36. 🧠 Vault + `no_log`

These solve different problems:

```text
Vault
 ↓
Protect secret at rest
```

while:

```text
no_log
 ↓
Reduce secret exposure in execution logs
```

You may need **both**.

---

# 37. 🚨 Bad Example

```yaml
- name: Set password
  ansible.builtin.shell:
    cmd: "myapp --password {{ db_password }}"

- name: Print password
  ansible.builtin.debug:
    msg: "Password is {{ db_password }}"
```

Even if:

```text
db_password
```

came from Vault:

```text
Vault encryption
      ↓
secret decrypted
      ↓
debug prints it
      ↓
💥 secret leaked
```

---

# 38. ✅ Better

```yaml
- name: Set application password
  ansible.builtin.command:
    cmd: "myapp --password {{ db_password }}"
  no_log: true
```

And never intentionally print:

```yaml
debug:
  var: db_password
```

---

# 39. 🧠 Secret Exposure Can Happen in Other Places

Be careful with:

```text
debug
command arguments
shell commands
failed task output
registered variables
CI logs
temporary files
configuration files
```

Vault doesn't automatically solve all of these.

---

# 40. 🏭 Production Secret Flow

A good mental model:

```text
                  Secret
                    │
             ┌──────┴──────┐
             │             │
        At Rest          Runtime
             │             │
             ▼             ▼
        Vault / SM       Ansible
             │             │
             │          no_log
             │             │
             └──────┬──────┘
                    ▼
             Managed System
```

Security has multiple layers.

---

# 41. 🎤 Interview Question

### "Does Ansible Vault encrypt the secret during network transmission?"

Be careful with the answer.

Vault primarily protects the secret **stored in encrypted form**.

Ansible's connection layer separately handles communication with managed nodes, typically using SSH for Linux systems.

So:

```text
Vault
→ encryption at rest

SSH/TLS/etc.
→ secure transport
```

Don't say:

> "Vault encrypts all Ansible communication."

That's incorrect.

---

# 42. 🎤 Interview Question

### "Can I commit Ansible Vault files to Git?"

Yes, **encrypted Vault files can be stored in Git**, provided:

* the Vault password/key is not stored with them
* access to the decryption credential is controlled
* secrets are not exposed elsewhere

This is one of the primary use cases of Vault.

---

# 43. 🎤 Interview Question

### "What if someone gets the encrypted Vault file?"

They still need the corresponding Vault password/key to decrypt it.

Therefore:

```text
Encrypted file
+
Strongly protected password
```

is the security model.

If the Vault password is compromised, the encrypted files using it should be considered compromised.

---

# 44. 🎤 Interview Question

### "How do you use Ansible Vault in CI/CD?"

Strong answer:

> "I keep encrypted Vault data in the repository and store the Vault credential separately in the CI/CD secret store. During the pipeline, the credential is injected securely and passed to Ansible using a Vault password mechanism. I also use `no_log` for tasks that could expose sensitive values."

Excellent production answer. 💪

---

# 45. 🧠 Common Mistakes

### ❌ Plaintext secrets in Git

```yaml
db_password: SuperSecret123
```

---

### ❌ Vault password in Git

```text
vault_password.txt
```

---

### ❌ Printing secrets

```yaml
debug:
  var: db_password
```

---

### ❌ Putting secrets into shell commands without `no_log`

```yaml
shell: "command --password {{ password }}"
```

---

### ❌ Assuming Vault handles secret rotation

Vault encryption isn't the same thing as centralized secret lifecycle management.

---

# 46. 🏆 Production Best Practices

### ✅ Encrypt secrets

```bash
ansible-vault encrypt secrets.yml
```

### ✅ Keep Vault credentials outside Git

Use:

```text
CI/CD secret store
password manager
secure credential mechanism
```

### ✅ Use `no_log: true`

For tasks involving sensitive values.

### ✅ Separate environments

Consider:

```text
dev Vault
staging Vault
prod Vault
```

with appropriate Vault IDs or, better, a dedicated secret manager where appropriate.

### ✅ Prefer centralized secret management for dynamic secrets

For example:

```text
GCP Secret Manager
```

when your architecture needs centralized lifecycle management.

---

# 47. 🧠 Complete Vault Flow

```text
             🔐 SECRET
                 │
                 ▼
          ansible-vault
                 │
                 ▼
       ENCRYPTED vault.yml
                 │
                 ▼
                Git
                 │
                 ▼
              CI/CD
                 │
          secure credential
                 │
                 ▼
             Ansible
                 │
             decrypt
                 │
                 ▼
          Variable in memory
                 │
                 ▼
              Module
                 │
                 ▼
          Managed Node
                 │
                 ▼
          no_log protects
             execution logs
```

---

# 🎯 48. The Commands You Must Know

### Create

```bash
ansible-vault create secrets.yml
```

### View

```bash
ansible-vault view secrets.yml
```

### Edit

```bash
ansible-vault edit secrets.yml
```

### Encrypt existing

```bash
ansible-vault encrypt secrets.yml
```

### Decrypt

```bash
ansible-vault decrypt secrets.yml
```

### Change password

```bash
ansible-vault rekey secrets.yml
```

### Run playbook

```bash
ansible-playbook site.yml --ask-vault-pass
```

### Password file

```bash
ansible-playbook site.yml \
  --vault-password-file ~/.ansible/vault_password
```

### Vault ID

```bash
ansible-playbook site.yml \
  --vault-id prod@prompt
```

---

# 🏆 Final Mental Model

Memorize these three things:

```text
🔐 Vault
    ↓
Protect secrets at rest


🤫 no_log
    ↓
Reduce secret exposure in logs


☁️ Secret Manager
    ↓
Centralized secret lifecycle
```

And the most important distinction:

```text
Ansible Vault
    =
encrypted Ansible data

NOT

complete enterprise secret-management solution
```

---


# ⚙️ Ansible Topic 15 — `ansible.cfg` & Ansible Configuration

Now we're moving from **Ansible syntax** into **how Ansible itself behaves**.

You already know:

```text
Playbook
   ↓
Roles
   ↓
Tasks
   ↓
Modules
   ↓
Managed Nodes
```

But who controls things like:

* Which inventory is used?
* Which SSH user?
* How many hosts run simultaneously?
* Where are roles searched?
* Where are collections searched?
* Is host-key checking enabled?
* How long does SSH wait?
* Where are logs written?

👉 **`ansible.cfg`**

---

# 1. 🧠 What is `ansible.cfg`?

`ansible.cfg` is Ansible's **configuration file**.

It controls default behavior of Ansible commands and execution.

For example:

```text
ansible.cfg
    │
    ├── Inventory
    ├── Remote user
    ├── SSH behavior
    ├── Forks
    ├── Roles path
    ├── Collections path
    ├── Fact gathering
    ├── Logging
    └── Many other defaults
```

---

# 2. 🏗️ Visualize It

Think of:

```text
                 ansible.cfg
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Connection   Execution   Project
          │           │           │
          ▼           ▼           ▼
         SSH        forks       roles
        user        timeout     collections
        become      strategy    inventory
```

So:

> **Playbooks define what Ansible should do. `ansible.cfg` defines many of the default ways Ansible should behave while doing it.**

---

# 3. 📁 Where Does Ansible Find `ansible.cfg`?

This is important.

Ansible can have configuration files at different locations.

The commonly relevant order is:

```text
1. ANSIBLE_CONFIG environment variable
2. ./ansible.cfg
3. ~/.ansible.cfg
4. /etc/ansible/ansible.cfg
```

The first applicable configuration takes precedence.

### Most important for projects:

```text
project/
├── ansible.cfg
├── inventory/
├── playbooks/
└── roles/
```

This is very common.

---

# 4. 🎯 Why Keep `ansible.cfg` in the Project?

Imagine:

```text
Project A
```

needs:

```text
forks = 20
roles_path = ./roles
```

while:

```text
Project B
```

needs:

```text
forks = 50
roles_path = ./ansible_roles
```

A project-specific:

```text
ansible.cfg
```

allows the project to define its expected behavior.

---

# 5. 📄 Basic `ansible.cfg`

Example:

```ini
[defaults]

inventory = ./inventory/hosts.yml
remote_user = ansible
roles_path = ./roles
collections_path = ./collections
host_key_checking = False
forks = 20
```

This is a configuration file, **not YAML**.

Format:

```ini
[section]

key = value
```

---

# 6. 🔍 Check Which Configuration Is Being Used

This command is extremely useful:

```bash
ansible --version
```

You may see:

```text
ansible [core ...]
  config file = /path/to/project/ansible.cfg
  configured module search path = ...
  ansible python module location = ...
  executable location = ...
```

Look specifically for:

```text
config file =
```

This tells you which `ansible.cfg` Ansible is using.

---

# 7. 🛠️ `ansible-config`

Ansible also provides:

```bash
ansible-config
```

For example:

```bash
ansible-config list
```

shows available configuration options.

You can also inspect the effective configuration:

```bash
ansible-config dump
```

And:

```bash
ansible-config view
```

can show the active configuration file.

---

# 8. 🧠 `ansible-config dump`

This is extremely useful for troubleshooting.

```bash
ansible-config dump
```

You may see things like:

```text
DEFAULT_FORKS
DEFAULT_REMOTE_USER
DEFAULT_HOST_LIST
HOST_KEY_CHECKING
ROLES_PATH
COLLECTIONS_PATHS
```

So if you're wondering:

> "Why is Ansible behaving this way?"

check the effective configuration.

---

# 9. 🎯 Inventory Configuration

You can specify the default inventory:

```ini
[defaults]
inventory = ./inventory/hosts.yml
```

Then instead of:

```bash
ansible-playbook -i inventory/hosts.yml site.yml
```

you can simply use:

```bash
ansible-playbook site.yml
```

because Ansible already knows:

```text
inventory = ./inventory/hosts.yml
```

---

# 10. 🧠 Command-Line Override

This is very important.

Suppose:

```ini
[defaults]
inventory = ./inventory/prod.yml
```

But you execute:

```bash
ansible-playbook -i inventory/staging.yml site.yml
```

The command-line value takes precedence.

Conceptually:

```text
ansible.cfg
     ↓
default

CLI option
     ↓
override
```

So:

```bash
-i
```

overrides the configured inventory.

---

# 11. 🏆 Configuration Hierarchy

For many Ansible settings, think:

```text
Lowest priority
      │
      ▼
ansible.cfg defaults
      │
      ▼
variables / play configuration
      │
      ▼
command-line options
      │
      ▼
Highest priority
```

⚠️ Exact precedence varies by setting, so don't memorize this as a universal rule for every Ansible option.

The practical lesson:

> **Explicit runtime options can override configuration defaults.**

---

# 12. 👤 `remote_user`

You can configure:

```ini
[defaults]
remote_user = ansible
```

Then Ansible connects using:

```text
ansible
```

instead of requiring:

```yaml
remote_user: ansible
```

in every play.

---

# 13. 🔑 Private Key

You can specify an SSH private key:

```ini
[defaults]
private_key_file = ~/.ssh/ansible_rsa
```

Then Ansible can use it for SSH connections.

However, in production you may prefer:

```text
SSH agent
credential management
CI/CD secret injection
```

rather than hardcoding sensitive paths/configuration unnecessarily.

---

# 14. 🔐 `host_key_checking`

You may encounter:

```ini
[defaults]
host_key_checking = False
```

This disables SSH host-key verification.

You may see this in labs because it makes initial connections easier.

### But production?

Be careful. ⚠️

SSH host-key checking exists to protect against:

```text
Man-in-the-middle attacks
unexpected host identity
```

So don't blindly disable it in production.

---

# 15. 🧠 Why Do Labs Often Disable It?

Imagine a temporary environment where servers are constantly recreated:

```text
VM destroyed
   ↓
new VM created
   ↓
same IP
   ↓
new SSH host key
```

Ansible/SSH may complain about the changed host key.

People sometimes use:

```ini
host_key_checking = False
```

in temporary environments.

But:

> **Convenience ≠ security best practice.**

---

# 16. 🚀 `forks`

This is **very important**, especially because your next topic will go deeper into execution concurrency.

Example:

```ini
[defaults]
forks = 20
```

`forks` controls the default number of parallel worker processes Ansible can use for executing tasks across hosts.

Suppose:

```text
100 hosts
forks = 10
```

Conceptually:

```text
100 hosts
   │
   ▼
10 workers
   │
   ▼
process hosts in batches/concurrent workers
```

Don't confuse this with `serial`.

We'll go deep into that next. 🔥

---

# 17. 🧠 `forks` vs `serial`

This is a common interview trap.

### `forks`

Controls the **maximum concurrency Ansible can use**.

### `serial`

Controls the **batch size for a play**.

Example:

```yaml
serial: 5
```

means:

> Process only 5 hosts in the current batch.

Whereas:

```ini
forks = 20
```

means Ansible has up to 20 worker slots available.

Think:

```text
forks
 ↓
"How much concurrency can Ansible provide?"


serial
 ↓
"How many hosts should this play process at a time?"
```

We'll deep dive into this next.

---

# 18. ⏱️ Connection Timeout

You may configure:

```ini
[defaults]
timeout = 30
```

This controls the default connection timeout behavior.

For infrastructure automation, network problems can otherwise cause long waits.

But don't blindly set it extremely low.

For example:

```text
timeout = 2
```

could be problematic in:

* high-latency networks
* cloud environments
* overloaded nodes
* VPN connections

---

# 19. 📦 `roles_path`

You can specify where Ansible searches for roles:

```ini
[defaults]
roles_path = ./roles
```

Then:

```text
project/
└── roles/
    ├── nginx/
    ├── postgresql/
    └── patroni/
```

Ansible can locate them.

You can specify multiple paths where supported:

```ini
roles_path = ./roles:/opt/company/ansible-roles
```

---

# 20. 📦 `collections_path`

Similarly:

```ini
[defaults]
collections_path = ./collections
```

Then:

```text
project/
└── collections/
    └── ansible_collections/
```

Ansible searches this location for collections.

This connects directly to our previous topic.

---

# 21. 🧠 Roles vs Collections Paths

```text
roles_path
     ↓
Where should Ansible find roles?

collections_path
     ↓
Where should Ansible find collections?
```

Don't mix them.

---

# 22. 🐍 Python Interpreter

For Linux managed nodes, Ansible often needs Python.

You can specify:

```ini
[defaults]
interpreter_python = auto
```

or configure the interpreter at inventory/host/group level:

```yaml
ansible_python_interpreter: /usr/bin/python3
```

For example:

```yaml
all:
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

This is often useful when a host has multiple Python installations.

---

# 23. 🧠 Why Interpreter Configuration Matters

Imagine:

```text
/usr/bin/python
/usr/bin/python3
/usr/local/bin/python3.11
```

Which one should Ansible use?

You may get module compatibility issues if the wrong interpreter is selected.

So:

```text
Ansible
   ↓
Python interpreter
   ↓
module execution
```

The interpreter matters.

---

# 24. 📊 Fact Gathering

You can control fact gathering:

```ini
[defaults]
gathering = smart
```

You may encounter values such as:

```text
implicit
explicit
smart
```

The exact behavior and supported options depend on the Ansible version.

The important concept:

> Fact gathering has a performance impact.

Remember our previous discussion:

```text
Play starts
   ↓
Gather facts
   ↓
Execute tasks
```

If you have:

```text
500 hosts
```

and each host requires fact gathering, that can become expensive.

---

# 25. 🚀 Disabling Fact Gathering

At play level:

```yaml
- name: Configure application
  hosts: app
  gather_facts: false
```

Then Ansible doesn't automatically gather facts at the beginning of that play.

Useful when:

* facts aren't needed
* you want faster execution
* you have a specialized automation workflow

But don't disable it blindly if tasks rely on:

```text
ansible_os_family
ansible_distribution
ansible_default_ipv4
ansible_memtotal_mb
```

etc.

---

# 26. 🧠 Fact Caching

For larger environments, fact caching can improve performance.

Instead of:

```text
Every play
   ↓
Every host
   ↓
Gather facts again
```

you can cache them.

Conceptually:

```text
Host
 ↓
Gather facts
 ↓
Cache
 ↓
Future play
 ↓
Reuse cached facts
```

We'll cover this properly later.

---

# 27. 📝 Logging

You can configure:

```ini
[defaults]
log_path = ./ansible.log
```

Then Ansible can write logs to that location.

This can be useful for:

```text
troubleshooting
auditing
automation debugging
```

But be careful:

> Logs can potentially contain sensitive information.

Especially if tasks aren't using:

```yaml
no_log: true
```

---

# 28. 🔇 `deprecation_warnings`

You may encounter:

```ini
deprecation_warnings = False
```

This suppresses deprecation warnings.

### Should you disable this in production?

Usually, **don't blindly suppress warnings**.

Warnings may tell you:

```text
"This syntax will stop working in a future version."
```

Those warnings are valuable during upgrades.

---

# 29. 🧪 Retry Files

Older Ansible setups may generate retry files such as:

```text
site.retry
```

You may see:

```ini
retry_files_enabled = False
```

depending on the Ansible version.

Retry-file behavior has changed across Ansible versions, so don't spend too much interview time on this.

The more important point is:

> Modern Ansible workflows generally rely on inventory/limits and CI/CD rather than old `.retry` workflows.

---

# 30. 🖥️ Output / Callback Configuration

Ansible can customize output through callback plugins.

You may encounter:

```ini
[defaults]
stdout_callback = ...
```

This controls how Ansible presents output.

For example, organizations may use callback plugins for:

```text
structured output
logging
reporting
CI integration
```

Don't worry about memorizing callback plugin names yet.

We'll cover plugins later.

---

# 31. 🏭 Production Example `ansible.cfg`

A reasonable project might have:

```ini
[defaults]

inventory = ./inventories/production/hosts.yml

roles_path = ./roles

collections_path = ./collections

remote_user = ansible

forks = 20

timeout = 30

interpreter_python = auto

log_path = ./ansible.log
```

Then:

```text
ansible-project/
│
├── ansible.cfg
│
├── inventories/
│   └── production/
│       └── hosts.yml
│
├── roles/
│   ├── common/
│   ├── postgresql/
│   └── patroni/
│
├── collections/
│
├── playbooks/
│   └── site.yml
│
└── requirements.yml
```

---

# 32. 🧠 What Should NOT Be Hardcoded?

Be careful about putting secrets directly into:

```ini
ansible.cfg
```

For example, don't treat this as a secret store:

```ini
password = MySecret123
```

❌ Bad practice.

Sensitive credentials should use proper secret-management mechanisms.

---

# 33. 🔥 Project Configuration vs Global Configuration

Suppose your machine has:

```text
/etc/ansible/ansible.cfg
```

but your project has:

```text
project/ansible.cfg
```

The project-specific configuration can allow the project to be self-contained.

Example:

```text
Machine
  │
  └── Global Ansible config

Project
  │
  └── Project-specific ansible.cfg
```

This is especially useful in CI/CD.

---

# 34. 🏗️ CI/CD Architecture

A clean pipeline might look like:

```text
                  Git Repository
                       │
                       ▼
                 Ansible Project
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ansible.cfg    requirements.yml   playbooks
        │
        ▼
     Ansible
        │
        ├── Roles
        ├── Collections
        └── Inventory
        │
        ▼
    Managed Nodes
```

Everything needed to understand the project's behavior is close to the project.

---

# 35. 🎯 `ansible.cfg` vs Inventory

Don't confuse these.

### `ansible.cfg`

Controls Ansible behavior:

```text
forks
timeout
roles_path
collections_path
remote_user
```

### Inventory

Defines:

```text
Which hosts exist?
Which groups?
Which host/group variables?
```

Example:

```yaml
web:
  hosts:
    web01:
    web02:

db:
  hosts:
    db01:
```

So:

```text
ansible.cfg
   ↓
How Ansible behaves


inventory
   ↓
Where Ansible operates
```

---

# 36. 🎯 `ansible.cfg` vs Playbook

Again:

### Configuration

```text
Default behavior
```

### Playbook

```text
Desired automation
```

Example:

```ini
[defaults]
forks = 20
```

means:

> Default concurrency configuration.

While:

```yaml
- name: Install PostgreSQL
  ansible.builtin.package:
    name: postgresql
    state: present
```

means:

> Actually perform an operation.

---

# 37. 🎤 Interview Questions

### Q1. What is `ansible.cfg`?

> The Ansible configuration file that defines default behavior and settings such as inventory, connection behavior, forks, roles paths, collections paths and other execution options.

---

### Q2. How do you know which `ansible.cfg` is being used?

```bash
ansible --version
```

Look for:

```text
config file =
```

---

### Q3. How do you inspect effective configuration?

```bash
ansible-config dump
```

---

### Q4. How do you specify the inventory in `ansible.cfg`?

```ini
[defaults]
inventory = ./inventory/hosts.yml
```

---

### Q5. What is `forks`?

> The default number of parallel worker processes Ansible can use when executing tasks across hosts.

---

### Q6. What is `roles_path`?

> The path or paths Ansible searches for roles.

---

### Q7. What is `collections_path`?

> The path or paths Ansible searches for collections.

---

### Q8. Why might you configure `ansible_python_interpreter`?

> To explicitly select the Python interpreter Ansible should use on a managed host when automatic discovery isn't suitable or multiple Python installations exist.

---

# 38. 🔥 A3 Interview Scenario

### Interviewer:

> "Ansible is running very slowly against 500 servers. What would you check?"

Don't immediately say:

> "Increase forks."

Think systematically:

```text
500 servers
     │
     ▼
Check connection latency
     │
     ▼
Check forks
     │
     ▼
Check fact gathering
     │
     ▼
Check fact caching
     │
     ▼
Check serial
     │
     ▼
Check strategy
     │
     ▼
Check task bottlenecks
     │
     ▼
Check SSH/pipelining
     │
     ▼
Measure before tuning
```

This leads directly into our next topic. 🚀

---

# 39. 🧠 The Critical Difference

You should now remember:

```text
             ansible.cfg
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
   Connection   Execution   Project
      │           │           │
    SSH/user     forks       roles
    timeout      strategy    collections
                 facts       inventory
```

And:

```text
ansible.cfg
     ↓
DEFAULTS


CLI / play / inventory
     ↓
Can override specific behavior
```

---

# 🏆 40. What You MUST Know for LevelUp

Don't try to memorize every configuration option.

Focus on these:

```text
🔥 inventory
🔥 remote_user
🔥 private_key_file
🔥 forks
🔥 timeout
🔥 roles_path
🔥 collections_path
🔥 host_key_checking
🔥 interpreter_python
🔥 gather_facts
🔥 fact caching
🔥 log_path
```

And know how to troubleshoot:

```bash
ansible --version
```

```bash
ansible-config dump
```

```bash
ansible-config list
```

---

# 🎯 Final Mental Model

Think of Ansible as:

```text
                    Ansible
                       │
              ┌────────┴────────┐
              ▼                 ▼
         Configuration       Automation
              │                 │
              ▼                 ▼
        ansible.cfg          playbook
              │                 │
      ┌───────┼───────┐         ▼
      ▼       ▼       ▼       roles
    forks   SSH    paths        │
                                ▼
                              tasks
                                │
                                ▼
                              hosts
```

### One sentence to remember:

> **`ansible.cfg` controls Ansible's default behavior; the playbook defines the automation; inventory defines the target infrastructure.**

---

# 🚀 Ansible Topic 16 — Execution Control & Performance

This is one of the **most important Ansible topics for your A3/LevelUp preparation**. 🔥

Especially because you already encountered **`serial` in an interview**.

We need to clearly understand four things:

```text
forks
serial
strategy
throttle
```

They all affect execution, but they solve **different problems**.

---

# 1. 🧠 The Core Problem

Suppose you have:

```text
100 production servers
```

and you run:

```bash
ansible-playbook site.yml
```

Ansible needs to decide:

> How many hosts should I work on at once?

> Should all hosts move through tasks together?

> Should I process hosts in batches?

> Should a particular task have limited concurrency?

That's where these options come in.

---

# 2. 🏗️ The Big Picture

```text
                         ANSIBLE
                            │
                  ┌─────────┴─────────┐
                  │                   │
               Execution           Concurrency
                  │                   │
                  ▼                   ▼
              strategy              forks
                  │
                  │
          ┌───────┴────────┐
          ▼                ▼
       linear             free
          
          
              serial
                │
                ▼
         Batch processing


             throttle
                │
                ▼
       Limit a specific task
```

The easiest way to remember:

| Option     | Controls                          |
| ---------- | --------------------------------- |
| `forks`    | Overall worker concurrency        |
| `serial`   | Hosts processed in batches        |
| `strategy` | How tasks are scheduled           |
| `throttle` | Concurrency for a particular task |

---

# 3. 🔥 `forks`

Let's start with `forks`.

In `ansible.cfg`:

```ini
[defaults]
forks = 5
```

This means Ansible can use up to **5 worker processes** for parallel host execution.

Suppose:

```text
10 servers
forks = 5
```

Conceptually:

```text
        10 servers

    ┌─────┬─────┬─────┬─────┬─────┐
    │ S1  │ S2  │ S3  │ S4  │ S5  │
    └─────┴─────┴─────┴─────┴─────┘
             5 workers

             ↓

    ┌─────┬─────┬─────┬─────┬─────┐
    │ S6  │ S7  │ S8  │ S9  │ S10 │
    └─────┴─────┴─────┴─────┴─────┘
```

Very simplified, but the idea is:

> `forks` determines the maximum number of worker processes Ansible can use concurrently.

---

# 4. 🧠 `forks` Does NOT Mean "Batch Size"

This is extremely important.

If:

```ini
forks = 10
```

it does **not** mean:

> Always execute exactly 10 hosts and then move to the next 10.

That's more closely related to:

```yaml
serial: 10
```

So:

```text
forks
 ↓
Worker capacity


serial
 ↓
Host batch size
```

---

# 5. 🔥 `serial`

Now the important one.

Suppose:

```yaml
serial: 10
```

and:

```text
100 servers
```

Ansible processes:

```text
Batch 1 → 10 servers
Batch 2 → 10 servers
Batch 3 → 10 servers
...
Batch 10 → 10 servers
```

Architecture:

```text
100 SERVERS
     │
     ▼
┌──────────────┐
│ Batch 1      │
│ 10 servers   │
└──────────────┘
     │
     ▼
┌──────────────┐
│ Batch 2      │
│ 10 servers   │
└──────────────┘
     │
     ▼
       ...
     │
     ▼
┌──────────────┐
│ Batch 10     │
│ 10 servers   │
└──────────────┘
```

---

# 6. 🎯 Why Do We Need `serial`?

This is extremely useful for **rolling deployments**.

Imagine:

```text
100 production application servers
```

You don't want:

```text
100 servers
     ↓
stop application
     ↓
deploy new version
```

because your entire service may become unavailable.

Instead:

```text
10 servers
   ↓
deploy
   ↓
validate
   ↓
next 10
```

This provides controlled rollout.

---

# 7. 🏭 Production Rolling Deployment

Example:

```yaml
---
- name: Rolling application deployment
  hosts: app_servers
  become: true
  serial: 10

  tasks:

    - name: Stop application
      ansible.builtin.service:
        name: myapp
        state: stopped

    - name: Deploy application
      ansible.builtin.copy:
        src: myapp.jar
        dest: /opt/myapp/myapp.jar

    - name: Start application
      ansible.builtin.service:
        name: myapp
        state: started
```

Execution:

```text
10 servers
    ↓
stop
    ↓
deploy
    ↓
start
    ↓
next 10
```

---

# 8. 🧠 Important Behavior of `serial`

Suppose:

```yaml
serial: 10
```

and:

```text
100 hosts
```

The play executes in batches.

But **inside each batch**, normal Ansible concurrency still applies.

This is where:

```text
serial + forks
```

interact.

---

# 9. 🆚 `serial` + `forks`

Suppose:

```text
100 hosts

serial = 20

forks = 5
```

Think:

```text
100 hosts
   │
   ▼
20-host batch
   │
   ▼
up to 5 workers
   │
   ▼
process those hosts concurrently
```

Then:

```text
next 20-host batch
```

So:

```text
serial
  ↓
How many hosts belong to the current batch?


forks
  ↓
How much parallel worker capacity is available?
```

---

# 10. 🏆 Interview Scenario

### Interviewer:

> "I have 100 production servers. I want to update only 10 servers at a time. What will you use?"

Answer:

```yaml
serial: 10
```

Example:

```yaml
- name: Rolling deployment
  hosts: app_servers
  serial: 10
```

Then explain:

> "`serial` controls the number of hosts processed in each batch. This is useful for rolling deployments because I can update and validate a subset before proceeding to the next batch."

🔥 That's a strong answer.

---

# 11. 🧠 `serial` Can Use Percentages

You don't have to specify only a fixed number.

You can use:

```yaml
serial: 20%
```

Suppose:

```text
100 hosts
```

Then approximately:

```text
20 hosts per batch
```

This can be useful when inventory sizes vary.

---

# 12. 📊 Multiple Serial Values

This is a very useful advanced feature.

You can write:

```yaml
serial:
  - 5
  - 10
  - 20
```

Conceptually:

```text
100 hosts

Batch 1 → 5
Batch 2 → 10
Batch 3 → 20
Batch 4 → 20
Batch 5 → 20
Batch 6 → 20
Batch 7 → remaining
```

This lets you do a **gradual rollout**.

---

# 13. 🚀 Canary Deployment With `serial`

This is especially useful.

Suppose:

```yaml
serial:
  - 1
  - 10
  - 50%
```

Conceptually:

```text
100 servers

        Canary
          │
          ▼
       1 server
          │
       validate
          │
          ▼
      10 servers
          │
       validate
          │
          ▼
       50 servers
          │
       validate
          │
          ▼
      remaining
```

This is similar to a **canary rollout**.

---

# 14. 🔥 `strategy`

Now let's talk about **strategy**.

Strategy controls:

> **How Ansible schedules tasks across hosts.**

The two strategies you must know are:

```text
linear
free
```

---

# 15. 🧠 Default Strategy: `linear`

Normally:

```yaml
strategy: linear
```

Ansible works roughly like:

```text
Task 1
  │
  ├── Host 1
  ├── Host 2
  ├── Host 3
  └── Host 4
       │
       ▼
Task 2
  │
  ├── Host 1
  ├── Host 2
  ├── Host 3
  └── Host 4
```

The hosts progress through tasks together.

---

# 16. 🎯 Example of `linear`

Playbook:

```yaml
- name: Application deployment
  hosts: app_servers
  strategy: linear

  tasks:

    - name: Install package
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Deploy configuration
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
```

Conceptually:

```text
Task 1
  ↓
all eligible hosts
  ↓
Task 2
  ↓
all eligible hosts
  ↓
Task 3
  ↓
all eligible hosts
```

---

# 17. 🆓 `strategy: free`

Now:

```yaml
strategy: free
```

Hosts don't have to wait for slower hosts before moving to the next task.

Example:

```text
Host 1
Task 1 ──→ Task 2 ──→ Task 3
                         ↑
Host 2
Task 1 ─────→ Task 2 ──→ Task 3

Host 3
Task 1 ───────────→ Task 2 ──→ Task 3
```

Fast hosts can move ahead.

---

# 18. 🆚 Linear vs Free

|                                   | `linear` | `free`    |
| --------------------------------- | -------- | --------- |
| Default                           | ✅        | ❌         |
| Hosts progress together           | ✅        | ❌         |
| Fast host can move ahead          | ❌        | ✅         |
| Predictability                    | High     | Lower     |
| Useful for coordinated workflows  | ✅        | Sometimes |
| Useful when hosts are independent | —        | ✅         |

---

# 19. 🧠 When Is `free` Useful?

Imagine:

```text
100 independent servers
```

and each server is doing:

```text
package installation
configuration
local service setup
```

There may be no reason for:

```text
fast host
```

to wait for:

```text
slow host
```

Then:

```yaml
strategy: free
```

may improve throughput.

---

# 20. ⚠️ When Should You Avoid `free`?

Be careful when tasks depend on synchronized state.

For example:

```text
Database cluster
```

You may want:

```text
Node 1
 ↓
validate
 ↓
Node 2
 ↓
validate
```

rather than:

```text
Node 1 → Node 2 → Node 3
Node 3 → Node 1
Node 2 → Node 3
```

Free strategy can make reasoning about ordering more difficult.

For coordinated infrastructure:

```text
linear
```

is often easier to reason about.

---

# 21. 🔥 `throttle`

Now another commonly confused option.

`throttle` limits the number of concurrent workers for a **specific task or block**.

Example:

```yaml
- name: Restart service
  ansible.builtin.service:
    name: myapp
    state: restarted
  throttle: 2
```

This means:

> Don't execute this task on more than 2 hosts concurrently.

---

# 22. 🧠 `throttle` vs `forks`

Suppose:

```ini
forks = 50
```

You have plenty of worker capacity.

But one particular task is dangerous to run against many systems simultaneously.

Use:

```yaml
throttle: 2
```

Then:

```text
Overall Ansible
    ↓
50 worker capacity


Specific task
    ↓
only 2 concurrent hosts
```

---

# 23. 🆚 `forks` vs `serial` vs `throttle`

This table is extremely important:

| Feature    | Scope         | Meaning                               |
| ---------- | ------------- | ------------------------------------- |
| `forks`    | Global/config | Maximum worker capacity               |
| `serial`   | Play          | Number of hosts in a batch            |
| `throttle` | Task/block    | Limit concurrency for that task/block |

Remember:

```text
forks
  → "How many workers can I have?"

serial
  → "How many hosts are in this batch?"

throttle
  → "How many hosts can run THIS task simultaneously?"
```

---

# 24. 🏗️ Visual Comparison

```text
                    100 HOSTS
                        │
                        ▼
                    serial: 20
                        │
                ┌───────┴───────┐
                ▼               ▼
             Batch 1         Batch 2
              20 hosts        20 hosts
                │
                ▼
             forks: 10
                │
                ▼
          up to 10 workers
                │
                ▼
         ┌──────────────┐
         │ Specific task│
         │ throttle: 2  │
         └──────────────┘
                │
                ▼
        only 2 concurrent
        for this task
```

This is the mental model you want.

---

# 25. 🔥 Real Production Example

Suppose you're upgrading PostgreSQL nodes.

You have:

```text
9 database nodes
```

You don't want to restart all of them simultaneously.

You might use:

```yaml
- name: Rolling database maintenance
  hosts: database
  serial: 1
  strategy: linear

  tasks:

    - name: Stop PostgreSQL
      ansible.builtin.service:
        name: postgresql
        state: stopped

    - name: Apply configuration
      ansible.builtin.template:
        src: postgresql.conf.j2
        dest: /etc/postgresql/postgresql.conf

    - name: Start PostgreSQL
      ansible.builtin.service:
        name: postgresql
        state: started
```

This gives:

```text
Node 1
 ↓
maintenance
 ↓
start
 ↓
next

Node 2
 ↓
maintenance
 ↓
start
 ↓
next
```

This is much safer than:

```text
Node 1 ─┐
Node 2 ─┤
Node 3 ─┤
Node 4 ─┼── restart simultaneously 💥
Node 5 ─┤
Node 6 ─┤
Node 7 ─┤
Node 8 ─┤
Node 9 ─┘
```

---

# 26. 🚨 But `serial: 1` Isn't Automatically Safe

Important A3-level point:

```yaml
serial: 1
```

only controls Ansible's rollout batch.

It does **not** understand:

* PostgreSQL replication health
* Patroni leader status
* application traffic
* load balancer health
* quorum
* cluster health

You need validation tasks.

---

# 27. 🏆 Production Database Rollout

A better pattern:

```text
Take one node
      │
      ▼
Check cluster health
      │
      ▼
Perform change
      │
      ▼
Restart
      │
      ▼
Wait for healthy
      │
      ▼
Verify replication
      │
      ▼
Proceed to next node
```

Example:

```yaml
serial: 1

tasks:

  - name: Check service
    ansible.builtin.service_facts:

  - name: Apply configuration
    ansible.builtin.template:
      src: postgresql.conf.j2
      dest: /etc/postgresql/postgresql.conf
    notify:
      - Restart PostgreSQL

  - name: Flush handlers
    ansible.builtin.meta: flush_handlers

  - name: Verify PostgreSQL
    ansible.builtin.command:
      cmd: pg_isready
    changed_when: false
```

Now the rollout has actual validation.

---

# 28. 🧠 `serial` and Failure

This is very important.

Suppose:

```yaml
serial: 10
```

and:

```text
Batch 1 → 10 hosts
```

One host fails.

Ansible does **not automatically mean**:

> Stop the entire deployment immediately.

Failure behavior depends on the play's failure controls and thresholds.

You can control this using:

```text
max_fail_percentage
any_errors_fatal
```

---

# 29. 🚨 `max_fail_percentage`

Example:

```yaml
serial: 10
max_fail_percentage: 20
```

Conceptually:

> If failures exceed the configured percentage threshold, stop the play.

Important nuance:

```text
max_fail_percentage: 20
```

means the failure count must **exceed** 20%, not merely equal it, for the threshold to trigger.

So don't casually interpret it as:

> "Stop at exactly 20%."

---

# 30. 🛑 `any_errors_fatal`

You can also use:

```yaml
any_errors_fatal: true
```

Meaning:

> A fatal error on a host can cause the play to stop across the remaining hosts.

Example:

```yaml
- name: Critical deployment
  hosts: app_servers
  serial: 10
  any_errors_fatal: true
```

Use carefully.

---

# 31. 🆚 `max_fail_percentage` vs `any_errors_fatal`

| Option                   | Purpose                                  |
| ------------------------ | ---------------------------------------- |
| `any_errors_fatal: true` | Treat a fatal error as globally stopping |
| `max_fail_percentage`    | Allow failures up to a threshold         |

Think:

```text
any_errors_fatal
    ↓
"One fatal failure is enough."


max_fail_percentage
    ↓
"Allow some failures, but stop if too many occur."
```

---

# 32. 🧩 `pause` During Rolling Deployments

You can also pause between batches or tasks.

Example:

```yaml
- name: Wait before continuing
  ansible.builtin.pause:
    seconds: 30
```

Useful when:

```text
Deploy
 ↓
wait for service stabilization
 ↓
continue
```

But don't use arbitrary sleeps when a proper health check can be performed.

Prefer:

```text
wait_for
until/retries
service validation
HTTP health check
```

when appropriate.

---

# 33. 🧠 Better Than `pause`

Instead of:

```yaml
- pause:
    seconds: 60
```

you may be better off with a condition:

```yaml
- name: Wait for application port
  ansible.builtin.wait_for:
    port: 8080
    host: "{{ inventory_hostname }}"
    timeout: 60
```

This means:

> Continue as soon as the application is ready.

Rather than:

> Always wait 60 seconds.

That's more production-friendly.

---

# 34. 🔄 `serial` + Health Check

A strong rolling deployment looks like:

```text
Batch 1
  ↓
Deploy
  ↓
Health check
  ↓
Success?
 ┌──────┴──────┐
Yes            No
 │              │
 ▼              ▼
Next batch    Stop/rollback
```

This is much better than blindly doing:

```text
10 → 10 → 10 → 10
```

---

# 35. 🏭 Production Application Deployment

Example:

```yaml
---
- name: Rolling application deployment
  hosts: app_servers
  become: true
  serial: 10
  strategy: linear

  tasks:

    - name: Remove server from load balancer
      ansible.builtin.command:
        cmd: /opt/lb/drain "{{ inventory_hostname }}"
      changed_when: true

    - name: Deploy application
      ansible.builtin.copy:
        src: myapp.jar
        dest: /opt/myapp/myapp.jar
        mode: '0644'

    - name: Restart application
      ansible.builtin.service:
        name: myapp
        state: restarted

    - name: Wait for application port
      ansible.builtin.wait_for:
        port: 8080
        timeout: 60

    - name: Health check
      ansible.builtin.uri:
        url: "http://{{ inventory_hostname }}:8080/health"
        status_code: 200

    - name: Add server back to load balancer
      ansible.builtin.command:
        cmd: /opt/lb/enable "{{ inventory_hostname }}"
      changed_when: true
```

Architecture:

```text
              Load Balancer
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
        App1     App2     App3
                   │
                   ▼
             Ansible batch
                   │
                   ▼
             deploy 10 hosts
                   │
                   ▼
              health check
                   │
             ┌─────┴─────┐
             ▼           ▼
           PASS         FAIL
             │           │
             ▼           ▼
       next batch      stop
```

That's a real-world use of `serial`.

---

# 36. 🧠 A Very Important Interview Scenario

### Interviewer:

> "You have 50 web servers. You want to update them 5 at a time. How?"

Answer:

```yaml
serial: 5
```

Then:

> "I would also validate each batch before proceeding, and depending on the application's availability requirements I might drain nodes from the load balancer before deployment and re-enable them after health checks."

🔥 Much stronger than simply saying `serial: 5`.

---

# 37. 🎤 Another Interview Scenario

> "What's the difference between `forks` and `serial`?"

Excellent answer:

> "`forks` controls the worker concurrency available to Ansible globally, while `serial` controls how many hosts are included in each batch of a play. For example, with 100 hosts and `serial: 10`, Ansible processes the play in groups of 10, while `forks` determines how much concurrency can be used within those eligible hosts."

---

# 38. 🎤 Another Interview Scenario

> "What's the difference between `serial` and `throttle`?"

Answer:

> "`serial` controls the number of hosts in a play's current batch, whereas `throttle` limits concurrent execution of a particular task or block. So `serial` is play-level batching, while `throttle` is task-level concurrency control."

---

# 39. 🎤 Another Interview Scenario

> "What does `strategy: free` do?"

Answer:

> "`free` allows each host to progress through tasks independently instead of waiting for other hosts to reach the same task, unlike the default `linear` strategy."

---

# 40. 🧠 Complete Comparison

| Feature                | `forks`         | `serial`         | `strategy`      | `throttle`       |
| ---------------------- | --------------- | ---------------- | --------------- | ---------------- |
| Scope                  | Config          | Play             | Play            | Task/block       |
| Controls               | Worker capacity | Host batches     | Task scheduling | Task concurrency |
| Example                | `forks=20`      | `serial: 5`      | `free`          | `throttle: 2`    |
| Rolling deployment     | ❌               | ✅                | Sometimes       | Sometimes        |
| Performance            | ✅               | Controls rollout | ✅               | ✅                |
| Limits individual task | ❌               | ❌                | ❌               | ✅                |

---

# 41. 🏆 The Mental Model You Need

Imagine:

```text
             100 SERVERS
                  │
                  ▼
            serial: 20
                  │
                  ▼
            20 eligible
                  │
                  ▼
             forks: 10
                  │
                  ▼
          up to 10 workers
                  │
                  ▼
          strategy: linear
                  │
                  ▼
        hosts move task-by-task
                  │
                  ▼
        Specific heavy task
                  │
           throttle: 2
                  │
                  ▼
       only 2 run simultaneously
```

This is the key concept. 🧠

---

# 42. ⚠️ Common Mistakes

### ❌ "forks controls batches"

No.

```text
forks → workers
serial → batches
```

---

### ❌ "serial controls task concurrency"

Not exactly.

It controls **how many hosts are in the current play batch**.

---

### ❌ "throttle controls the whole play"

No.

```text
throttle
   ↓
specific task/block
```

---

### ❌ "free is always faster"

Not necessarily.

It can improve throughput for independent hosts, but coordination and correctness may matter more than raw speed.

---

### ❌ "serial automatically performs health checks"

No.

You must implement appropriate validation.

---

# 43. 🚀 Production Performance Checklist

If an Ansible playbook is slow:

```text
1️⃣ Check forks
        ↓
2️⃣ Check serial
        ↓
3️⃣ Check strategy
        ↓
4️⃣ Check fact gathering
        ↓
5️⃣ Check fact caching
        ↓
6️⃣ Check SSH connection performance
        ↓
7️⃣ Check task-level bottlenecks
        ↓
8️⃣ Check throttle
        ↓
9️⃣ Measure before/after
```

Don't simply increase:

```text
forks = 100
```

and hope for the best. 😄

Your control node, network, managed nodes, SSH server limits, CPU and memory all matter.

---

# 44. 🎯 One Production Example Combining Everything

```yaml
---
- name: Rolling production deployment
  hosts: app_servers

  serial:
    - 1
    - 10
    - 25%

  strategy: linear

  max_fail_percentage: 20

  tasks:

    - name: Drain node
      ansible.builtin.command:
        cmd: "/opt/lb/drain {{ inventory_hostname }}"
      throttle: 2
      changed_when: true

    - name: Deploy application
      ansible.builtin.copy:
        src: myapp.jar
        dest: /opt/myapp/myapp.jar

    - name: Restart application
      ansible.builtin.service:
        name: myapp
        state: restarted

    - name: Wait for application
      ansible.builtin.wait_for:
        port: 8080
        timeout: 60

    - name: Validate application
      ansible.builtin.uri:
        url: "http://{{ inventory_hostname }}:8080/health"
        status_code: 200

    - name: Enable node
      ansible.builtin.command:
        cmd: "/opt/lb/enable {{ inventory_hostname }}"
      changed_when: true
```

This demonstrates:

```text
serial
strategy
max_fail_percentage
throttle
health checks
rolling deployment
```

🔥 This is the kind of example worth understanding for your A3 interview.

---

# 🏆 Final Cheat Sheet

```text
                    🚀 ANSIBLE EXECUTION
                           │
       ┌───────────────────┼──────────────────┐
       ▼                   ▼                  ▼
     FORKS               SERIAL            STRATEGY
       │                   │                  │
       ▼                   ▼                  ▼
   Worker limit        Host batches       Scheduling
                                            │
                                      ┌─────┴─────┐
                                      ▼           ▼
                                    linear      free


                    THROTTLE
                       │
                       ▼
                Task concurrency
```

### Memorize these four sentences:

> **`forks`** → How many worker processes can Ansible use?

> **`serial`** → How many hosts should be processed in each play batch?

> **`strategy`** → How should tasks be scheduled across hosts?

> **`throttle`** → How many hosts can execute this particular task/block concurrently?

---

# 🔌 Ansible Topic 17 — Connection Methods & Privilege Escalation

We'll keep this **short and practical** as requested. The goal is to know what you'll actually use in production and interviews.

---

# 1. 🧠 Basic Ansible Connection Flow

For a normal Linux server:

```text
Control Node
     │
     │ SSH
     ▼
Managed Node
     │
     │ become
     ▼
   root
```

There are actually **two separate concepts**:

```text
Connection
   ↓
How do I connect to the machine?


Privilege escalation
   ↓
Which user should execute the task?
```

---

# 2. 🔌 SSH Connection

The most common connection is SSH.

Inventory:

```yaml
all:
  hosts:
    web01:
      ansible_host: 192.168.1.10
      ansible_user: ansible
```

Ansible effectively connects:

```text
192.168.1.10
      ↑
SSH as user "ansible"
```

---

# 3. 🏷️ Important Connection Variables

These are the ones you should know:

| Variable                       | Purpose                            |
| ------------------------------ | ---------------------------------- |
| `ansible_host`                 | Actual IP/hostname to connect to   |
| `ansible_user`                 | SSH username                       |
| `ansible_port`                 | SSH port                           |
| `ansible_connection`           | Connection type                    |
| `ansible_ssh_private_key_file` | SSH private key                    |
| `ansible_python_interpreter`   | Python interpreter on managed node |

Example:

```yaml
web01:
  ansible_host: 10.10.10.20
  ansible_user: ansible
  ansible_port: 22
  ansible_ssh_private_key_file: ~/.ssh/ansible_key
```

---

# 4. 🔑 SSH Key Flow

You asked about this earlier, so remember:

```text
Control Node
    │
    ├── Private key 🔐
    │
    └── Public key
           │
           ▼
      Managed Node
      ~/.ssh/authorized_keys
```

The **private key stays on the control node**.

The managed node receives the public key.

Ansible uses the private key to authenticate.

---

# 5. 🧑‍💻 `ansible_user`

Example:

```yaml
ansible_user: ansible
```

Means:

> Connect to the managed node using the `ansible` Linux account.

This does **not** mean Ansible runs as root.

For example:

```text
SSH
 ↓
ansible user
```

The task executes as `ansible`.

---

# 6. 👑 `become`

If you need root privileges:

```yaml
become: true
```

Example:

```yaml
- name: Install nginx
  hosts: web
  become: true

  tasks:

    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Flow:

```text
Control Node
      │
      │ SSH
      ▼
  ansible user
      │
      │ sudo
      ▼
     root
      │
      ▼
 Install nginx
```

---

# 7. 🧠 `become` Does NOT Change SSH User

This is an important distinction.

```yaml
ansible_user: ansible
become: true
```

means:

```text
SSH → ansible
          │
          ▼
       become
          │
          ▼
        root
```

It does **not** mean:

```text
SSH directly as root
```

---

# 8. 👤 `become_user`

You can become another user:

```yaml
become: true
become_user: postgres
```

Flow:

```text
SSH
 ↓
ansible
 ↓
sudo
 ↓
postgres
```

Example:

```yaml
- name: Run PostgreSQL command
  ansible.builtin.command:
    cmd: psql -c "SELECT version();"
  become: true
  become_user: postgres
```

This is very useful in database automation.

---

# 9. 🔧 `become_method`

By default, Linux environments commonly use `sudo`.

You can explicitly specify:

```yaml
become: true
become_method: sudo
```

Other methods exist, such as:

```text
sudo
su
doas
```

For normal RHEL/Debian production environments:

```yaml
become: true
```

is usually enough.

---

# 10. 🏭 Typical Production Pattern

You might have:

```yaml
all:
  hosts:
    db01:
      ansible_host: 10.10.10.10
      ansible_user: ansible
```

Playbook:

```yaml
- name: Configure database
  hosts: db
  become: true

  tasks:

    - name: Install PostgreSQL
      ansible.builtin.package:
        name: postgresql
        state: present
```

Result:

```text
Control Node
     │
     │ SSH
     ▼
   ansible
     │
     │ sudo
     ▼
    root
     │
     ▼
Install PostgreSQL
```

---

# 11. 🐍 `ansible_python_interpreter`

Ansible modules on Linux commonly require Python.

If automatic detection isn't suitable:

```yaml
ansible_python_interpreter: /usr/bin/python3
```

Example:

```yaml
db01:
  ansible_host: 10.10.10.10
  ansible_user: ansible
  ansible_python_interpreter: /usr/bin/python3
```

Mental model:

```text
SSH
 ↓
Managed Node
 ↓
Python interpreter
 ↓
Ansible module
```

---

# 12. 🖥️ Local Connection

Ansible doesn't always need SSH.

You can use:

```yaml
ansible_connection: local
```

Example:

```yaml
localhost:
  ansible_connection: local
```

Then:

```text
Control Node
    │
    └── execute locally
```

Useful for:

* localhost automation
* bootstrapping
* local configuration

---

# 13. 🎯 `delegate_to`

This is an important production concept.

Suppose you're configuring:

```text
web01
web02
web03
```

but you want a particular task to execute on:

```text
load-balancer
```

Use:

```yaml
- name: Remove server from load balancer
  ansible.builtin.command:
    cmd: "/opt/lb/drain {{ inventory_hostname }}"
  delegate_to: load-balancer
```

Flow:

```text
Normal task
   ↓
web01


Delegated task
   ↓
load-balancer
```

The task belongs to the web-server workflow, but execution happens on the delegated host.

---

# 14. 🏃 `run_once`

Sometimes you want a task to execute only once, rather than once per host.

```yaml
- name: Create deployment record
  ansible.builtin.command:
    cmd: /opt/deploy/create-record
  run_once: true
```

Without:

```text
10 hosts
 ↓
10 executions
```

With:

```text
10 hosts
 ↓
1 execution
```

---

# 15. 🔥 `delegate_to` + `run_once`

Very common combination:

```yaml
- name: Update load balancer configuration
  ansible.builtin.command:
    cmd: /opt/lb/update
  delegate_to: load-balancer
  run_once: true
```

Meaning:

> Execute this task once, on the load balancer.

---

# 16. 🌐 Bastion / Jump Host

Sometimes:

```text
Control Node
     │
     │ cannot directly reach
     ▼
Private Server
```

You may need:

```text
Control Node
     │
     ▼
Bastion / Jump Host
     │
     ▼
Private Server
```

SSH supports this through mechanisms such as `ProxyJump`.

Ansible can use the SSH configuration for this.

For example, your SSH config might contain:

```text
Host private-server
    HostName 10.10.2.10
    User ansible
    ProxyJump bastion
```

Then Ansible can connect using the SSH configuration.

This is a common cloud architecture.

---

# 17. 🏗️ Complete Connection Architecture

```text
                  Control Node
                       │
                       │ SSH
                       ▼
                 Bastion Host
                       │
                       │ SSH / ProxyJump
                       ▼
                Private Managed Node
                       │
                       │ become
                       ▼
                     root
```

Very common in production networks.

---

# 18. 🎤 Interview Questions

### Q1. Difference between `ansible_user` and `become_user`?

> `ansible_user` is the user Ansible uses to establish the connection. `become_user` is the user Ansible switches to for task execution after connecting.

Example:

```yaml
ansible_user: ansible
become: true
become_user: postgres
```

Flow:

```text
SSH → ansible → sudo → postgres
```

---

### Q2. What does `become: true` do?

> It enables privilege escalation, typically using `sudo`, allowing tasks to execute with elevated privileges.

---

### Q3. Where does the SSH private key reside?

> Normally on the control node or in an SSH agent/credential system. The private key should not be copied to managed nodes.

---

### Q4. What is `delegate_to`?

> It causes a specific task to execute on another host instead of the current inventory host.

---

### Q5. What is `run_once`?

> It causes a task to execute only once for the play/batch rather than once per eligible host.

---

### Q6. How would Ansible reach a private server behind a bastion?

> Configure SSH ProxyJump or equivalent SSH connection settings so Ansible connects through the bastion host.

---

# 🧠 Final Cheat Sheet

```text
ansible_host
    ↓
Which machine?


ansible_user
    ↓
Which user for SSH?


ansible_port
    ↓
Which SSH port?


ansible_ssh_private_key_file
    ↓
Which private key?


become: true
    ↓
Enable privilege escalation


become_user
    ↓
Which user after escalation?


become_method
    ↓
How to escalate? sudo/su/etc.


ansible_python_interpreter
    ↓
Which Python on managed node?


delegate_to
    ↓
Execute this task somewhere else


run_once
    ↓
Execute this task only once
```

### The most important flow to remember:

```text
Control Node
     │
     │ SSH
     ▼
ansible user
     │
     │ become
     ▼
root / postgres / another user
     │
     ▼
Execute Ansible module
```

# 🧪 Ansible Topic 18 — Testing & Quality

We'll keep this **practical and production-focused**, especially because you already work with **Molecule, pytest, yamllint and Ansible automation**.

The goal is to understand the testing layers:

```text
        Ansible Code
             │
     ┌───────┼────────┐
     ▼       ▼        ▼
  Syntax   Lint    Functional
   Check   Check     Testing
     │       │          │
     ▼       ▼          ▼
  YAML/   Best       Molecule
  syntax  practices   / pytest
                        │
                        ▼
                   Idempotency
                        │
                        ▼
                       CI/CD
```

---

# 1. 🧠 Why Test Ansible?

Ansible is infrastructure code.

A bad change can:

```text
❌ Break services
❌ Change configuration incorrectly
❌ Restart hundreds of servers
❌ Cause non-idempotent behavior
❌ Fail halfway through deployment
```

So we want to test before production.

---

# 2. 🔍 Testing Layers

Think of Ansible testing in layers:

| Layer              | Tool / Method               | Purpose                       |
| ------------------ | --------------------------- | ----------------------------- |
| YAML validation    | `yamllint`                  | YAML formatting/syntax        |
| Ansible syntax     | `--syntax-check`            | Playbook structure            |
| Static analysis    | `ansible-lint`              | Ansible best practices        |
| Dry run            | `--check`                   | Predict changes               |
| Diff               | `--diff`                    | Show configuration changes    |
| Functional testing | Molecule                    | Test roles                    |
| Assertions         | `assert` / pytest           | Verify behavior               |
| Idempotency        | Second run                  | Ensure no unnecessary changes |
| CI/CD              | GitHub Actions/Jenkins/etc. | Automate all tests            |

---

# 3. 📄 YAML Linting

First layer:

```bash
yamllint site.yml
```

It checks YAML quality such as:

```text
indentation
spacing
line length
trailing spaces
syntax
```

Example bad YAML:

```yaml
- name: Install nginx
    ansible.builtin.package:
      name: nginx
```

The indentation is wrong.

`yamllint` can catch this before Ansible execution.

---

# 4. 🧠 `yamllint` vs `ansible-lint`

Don't confuse them.

### `yamllint`

Checks:

> Is this valid and well-formatted YAML?

### `ansible-lint`

Checks:

> Is this good Ansible code?

Think:

```text
YAML
 │
 ▼
yamllint
 │
 ▼
Valid YAML
 │
 ▼
ansible-lint
 │
 ▼
Good Ansible practices
```

---

# 5. 🔎 `ansible-lint`

Run:

```bash
ansible-lint
```

or:

```bash
ansible-lint site.yml
```

It can detect things such as:

* deprecated syntax
* poor module usage
* missing FQCN
* risky shell/command usage
* bad task naming
* inefficient patterns
* formatting/style issues
* problematic permissions

---

# 6. 🏆 Example

Instead of:

```yaml
- name: Install nginx
  yum:
    name: nginx
```

Modern style:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

Benefits:

```text
FQCN
explicit state
better portability
better lint compliance
```

---

# 7. 🧪 Syntax Check

Before actually running:

```bash
ansible-playbook site.yml --syntax-check
```

This checks whether the playbook structure is syntactically valid.

Example:

```yaml
- name: Configure web server
  hosts: web
  tasks:

    - name: Install nginx
      ansible.builtin.package:
        name: nginx
```

If YAML/playbook structure is broken, syntax check can catch it.

---

# 8. ⚠️ What Syntax Check Does NOT Do

This is important.

```bash
ansible-playbook site.yml --syntax-check
```

doesn't mean:

> "My automation is correct."

It doesn't actually perform the intended operations.

For example:

```yaml
- name: Delete important file
  ansible.builtin.file:
    path: /important/file
    state: absent
```

Syntax can be perfectly valid.

But the task may still be dangerous. 😄

So:

```text
syntax-check
    ≠
functional correctness
```

---

# 9. 🧪 Check Mode

Now:

```bash
ansible-playbook site.yml --check
```

This is Ansible's **check/dry-run mode**.

Conceptually:

```text
Normal run
   ↓
Make changes


Check mode
   ↓
Predict/report changes
   ↓
Don't normally modify target state
```

---

# 10. 🎯 Example

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

Run:

```bash
ansible-playbook site.yml --check
```

Ansible may report:

```text
changed: true
```

meaning:

> If I actually ran this, I expect a change.

---

# 11. ⚠️ Check Mode Isn't Perfect

Some modules don't support check mode fully.

Also:

```yaml
command:
shell:
```

can be difficult to predict because Ansible doesn't inherently know what arbitrary commands will do.

Example:

```yaml
- name: Run custom script
  ansible.builtin.command:
    cmd: /opt/scripts/setup.sh
```

Ansible can't always determine the exact result without executing it.

So:

> `--check` is useful, but it is **not a guarantee that no surprises will occur in a real run**.

---

# 12. 🔍 Diff Mode

Use:

```bash
ansible-playbook site.yml --diff
```

This is especially useful for:

```text
template
copy
lineinfile
blockinfile
```

Example:

```text
Before:
port=8080

After:
port=9090
```

Diff mode can show the change.

---

# 13. 🧪 Combine Check + Diff

Very useful:

```bash
ansible-playbook site.yml --check --diff
```

Conceptually:

```text
--check
   ↓
What would change?


--diff
   ↓
What exactly would change?
```

This is a great pre-production validation step.

---

# 14. 🔥 Be Careful With Secrets

If you're dealing with sensitive files:

```bash
ansible-playbook site.yml --check --diff
```

could potentially expose sensitive content in output.

You need to think about:

```text
Vault
no_log
diff exposure
CI logs
```

Security still matters during testing.

---

# 15. 🧠 Idempotency Testing

This is **very important for Ansible**.

Suppose you run:

```bash
ansible-playbook site.yml
```

First run:

```text
changed=5
```

Run it again:

```bash
ansible-playbook site.yml
```

A properly designed playbook should ideally show:

```text
changed=0
```

The second run should make no unnecessary changes.

---

# 16. 🏆 Example

Task:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

First run:

```text
changed=1
```

Second run:

```text
changed=0
```

That's good idempotent behavior.

---

# 17. 🚨 Non-Idempotent Example

Consider:

```yaml
- name: Create timestamp file
  ansible.builtin.command:
    cmd: touch /tmp/test
```

Every execution could be considered changed.

Better:

```yaml
- name: Ensure file exists
  ansible.builtin.file:
    path: /tmp/test
    state: touch
```

But even `state: touch` has timestamp semantics, so for a pure idempotent existence check, prefer:

```yaml
- name: Ensure file exists
  ansible.builtin.file:
    path: /tmp/test
    state: file
```

if the file should already exist, or:

```yaml
- name: Create file
  ansible.builtin.file:
    path: /tmp/test
    state: touch
    modification_time: preserve
    access_time: preserve
```

depending on the desired semantics.

The larger lesson:

> Don't use arbitrary commands when an Ansible module can model the desired state.

---

# 18. 🧠 How to Test Idempotency

A simple approach:

```bash
ansible-playbook site.yml
```

Then:

```bash
ansible-playbook site.yml
```

Check:

```text
First run:
changed > 0

Second run:
changed = 0
```

This is one of the most useful practical tests.

---

# 19. 🧪 Assertions

Ansible has an `assert` module.

Example:

```yaml
- name: Verify application port
  ansible.builtin.assert:
    that:
      - app_port == 8080
    fail_msg: "Application port is incorrect"
```

This turns assumptions into explicit tests.

---

# 20. 🔍 Example — OS Validation

```yaml
- name: Verify supported OS
  ansible.builtin.assert:
    that:
      - ansible_os_family in ['RedHat', 'Debian']
    fail_msg: "Unsupported operating system"
```

Flow:

```text
Host
 ↓
Check OS
 ↓
Supported?
 ┌──────┴──────┐
 YES           NO
 │              │
 ▼              ▼
Continue       Fail
```

Very useful in production roles.

---

# 21. 🧪 Molecule

Now the important testing framework for Ansible roles:

> **Molecule**

Molecule helps you develop and test Ansible roles in isolated environments.

Think:

```text
Ansible Role
     │
     ▼
  Molecule
     │
 ┌───┼────────────┐
 ▼   ▼            ▼
Create Configure Verify
 │      │           │
 └──────┴───────────┘
          │
          ▼
        Destroy
```

---

# 22. 🧠 Why Molecule?

Suppose you have:

```text
roles/
└── postgresql/
```

You don't want to test it directly on production.

Instead:

```text
Role
 ↓
Test environment
 ↓
Run role
 ↓
Verify
 ↓
Destroy environment
```

This makes role development safer.

---

# 23. 🏗️ Molecule Lifecycle

A simplified Molecule lifecycle:

```text
create
  ↓
prepare
  ↓
converge
  ↓
verify
  ↓
destroy
```

### `create`

Create the test environment.

### `prepare`

Perform any setup required before the role is tested.

### `converge`

Actually apply your Ansible role.

### `verify`

Check whether the desired result was achieved.

### `destroy`

Clean up the environment.

---

# 24. 🎯 Molecule Mental Model

```text
             Molecule
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
     Create   Converge   Verify
        │        │        │
        ▼        ▼        ▼
      VM/      Role      Tests
   Container   runs       run
                 │
                 ▼
              Destroy
```

---

# 25. 📁 Typical Molecule Structure

A role may have:

```text
roles/postgresql/
│
├── tasks/
├── handlers/
├── templates/
├── defaults/
│
└── molecule/
    └── default/
        ├── converge.yml
        ├── verify.yml
        ├── molecule.yml
        └── ...
```

The exact files can vary by Molecule version/configuration.

---

# 26. 📄 `converge.yml`

This defines how the role is tested.

Example:

```yaml
---
- name: Converge
  hosts: all
  become: true

  roles:
    - role: postgresql
```

Essentially:

> Run my role against the test instance.

---

# 27. 📄 `verify.yml`

This verifies the result.

Example:

```yaml
---
- name: Verify
  hosts: all
  become: true

  tasks:

    - name: Check PostgreSQL service
      ansible.builtin.service_facts:

    - name: Verify PostgreSQL is running
      ansible.builtin.assert:
        that:
          - ansible_facts.services['postgresql.service'].state == 'running'
```

Now Molecule isn't simply asking:

> "Did Ansible finish?"

It's asking:

> "Did the system actually reach the desired state?"

🔥 That's the important distinction.

---

# 28. 🧠 Converge vs Verify

Very important:

```text
converge
   ↓
Apply automation


verify
   ↓
Test resulting state
```

Don't confuse them.

---

# 29. 🧪 Example Testing Flow

Imagine your role should:

```text
Install PostgreSQL
Start service
Create config
Create user
```

Molecule:

```text
Create test VM
      ↓
Run role
      ↓
PostgreSQL installed?
      ↓
Service running?
      ↓
Config correct?
      ↓
User exists?
      ↓
All tests pass?
      ↓
Destroy VM
```

---

# 30. 🔥 Idempotency in Molecule

One of Molecule's useful capabilities is testing repeated convergence.

Conceptually:

```text
Converge
   ↓
changes occur
   ↓
Converge again
   ↓
no unexpected changes
```

You want:

```text
First convergence:
changed > 0

Second convergence:
changed = 0
```

This is especially valuable for Ansible roles.

---

# 31. 🧪 pytest

You can also use Python testing frameworks such as:

```text
pytest
```

For example, you might test:

```text
service state
configuration file
ports
users
permissions
application behavior
```

Depending on your project, pytest can complement Molecule.

---

# 32. 🧠 Molecule vs pytest

Don't think they're competitors.

They can work together.

```text
Molecule
   ↓
Creates/controls test environment
   ↓
Runs role
   ↓
pytest
   ↓
Verifies system behavior
```

For example:

```text
Molecule
   │
   ├── converge
   │
   └── verify
           │
           ▼
        pytest
```

---

# 33. 🏗️ Your Project-Type Testing

For an Ansible RPM validation project, you could have:

```text
Ansible Role
     │
     ▼
Molecule
     │
     ▼
Test VM
     │
     ▼
Install RPM
     │
     ▼
Configure PostgreSQL
     │
     ▼
Run verification
     │
 ┌───┴────┐
 ▼        ▼
pytest   assertions
```

This is a very realistic testing architecture.

---

# 34. 🧹 `ansible-lint` in CI

A good pipeline:

```text
Git push
   ↓
yamllint
   ↓
ansible-lint
   ↓
syntax-check
   ↓
Molecule
   ↓
pytest
   ↓
deploy
```

This catches different classes of problems at different stages.

---

# 35. 🚀 Production CI Pipeline

Example:

```text
                 Developer
                     │
                     ▼
                  Git Push
                     │
                     ▼
                ┌──────────┐
                │ CI       │
                └────┬─────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    yamllint    ansible-lint   syntax-check
        │            │            │
        └────────────┼────────────┘
                     ▼
                 Molecule
                     │
                     ▼
                  pytest
                     │
                     ▼
               Integration Test
                     │
                     ▼
              Production
```

---

# 36. 🎯 What Does Each Test Catch?

| Test             | Catches                           |
| ---------------- | --------------------------------- |
| `yamllint`       | YAML formatting/syntax            |
| `--syntax-check` | Ansible playbook syntax           |
| `ansible-lint`   | Ansible coding issues             |
| `--check`        | Potential changes                 |
| `--diff`         | Configuration differences         |
| Molecule         | Role behavior in test environment |
| pytest           | Detailed assertions               |
| Idempotency test | Repeated-run changes              |

This is the key table to remember.

---

# 37. 🧠 Don't Depend Only on `--check`

An interviewer might ask:

> "Is `ansible-playbook --check` enough for testing?"

Answer:

> No. Check mode is useful for predicting changes, but it doesn't fully validate actual behavior. Some modules have limited check-mode support, and arbitrary commands cannot always be predicted. Functional tests and integration tests are still required.

🔥 Good A3-level answer.

---

# 38. 🧠 Don't Depend Only on Syntax Check

Similarly:

```bash
ansible-playbook site.yml --syntax-check
```

only tells you:

> "The playbook is structurally valid."

It doesn't tell you:

```text
service works
configuration is correct
package installs
permissions are correct
role is idempotent
```

---

# 39. 🔥 Production Testing Pyramid

Think:

```text
             🏆
        Integration Tests
             │
        Molecule / pytest
             │
       ───────────────
         ansible-lint
       ───────────────
       syntax checking
       ───────────────
          yamllint
```

Cheap tests should happen first.

Expensive tests later.

---

# 40. 🧠 A3 Interview Scenario

### Interviewer:

> "How would you test an Ansible role before production?"

A strong answer:

> "First I'd validate YAML with yamllint and run ansible-lint for Ansible-specific issues. I'd run `--syntax-check` on the playbook. For the role itself I'd use Molecule to provision an isolated test environment, converge the role and verify the resulting state. I'd also test idempotency by converging more than once and use pytest or assertions for detailed functional validation. These checks would run automatically in CI before deployment."

That's an excellent answer. 💪

---

# 41. 🧪 Testing Idempotency — Interview Favorite

### Interviewer:

> "How do you verify that an Ansible role is idempotent?"

Answer:

> "I apply the role once and record the changes, then apply it again against the same state. The second execution should normally report no unnecessary changes. In Molecule, this can be incorporated into the role's test lifecycle."

---

# 42. 🚨 What If the Second Run Shows Changes?

Example:

```text
First run:
changed=8

Second run:
changed=3
```

Investigate tasks that repeatedly report changes.

Common causes:

```text
command/shell
incorrect template content
file timestamps
poorly designed handlers
non-deterministic data
always-changing values
```

---

# 43. 🧠 `changed_when`

Sometimes you intentionally use a command:

```yaml
- name: Check application status
  ansible.builtin.command:
    cmd: /opt/myapp/status
  register: result
  changed_when: false
```

This says:

> This command is informational; don't report it as changed.

This is important for testability and clean output.

---

# 44. 🧠 `failed_when`

You can also define what constitutes failure:

```yaml
- name: Check application
  ansible.builtin.command:
    cmd: /opt/myapp/health
  register: health
  changed_when: false
  failed_when: health.rc != 0
```

This makes the task's success criteria explicit.

---

# 45. 🏆 Production Quality Gates

A good Ansible repository might enforce:

```text
PR
 │
 ├── YAML valid?        ✅
 ├── ansible-lint?      ✅
 ├── syntax-check?      ✅
 ├── Molecule?          ✅
 ├── pytest?            ✅
 ├── idempotency?       ✅
 └── security checks?   ✅
          │
          ▼
       Merge
```

Then:

```text
Merge
  ↓
CI/CD
  ↓
Staging
  ↓
Validation
  ↓
Production
```

---

# 46. 🎯 What You Should Know for Your LevelUp

Don't memorize every Molecule configuration option.

Know this architecture:

```text
Ansible Role
     ↓
Molecule
     ↓
Test Environment
     ↓
Converge
     ↓
Verify
     ↓
Idempotency
     ↓
Destroy
```

And know:

```text
yamllint
ansible-lint
syntax-check
check mode
diff mode
pytest
```

That's enough for a strong interview foundation.

---

# 🏆 Final Cheat Sheet

```text
📝 yamllint
    → Is my YAML clean/valid?

🔎 ansible-lint
    → Is my Ansible code good?

🧪 --syntax-check
    → Is the playbook structurally valid?

👀 --check
    → What would change?

🔍 --diff
    → What configuration would change?

🧬 Molecule
    → Does my role work in an isolated environment?

🐍 pytest
    → Does the resulting system behave correctly?

🔄 Idempotency
    → Does running it again produce no unnecessary changes?
```


# 🌐 Ansible Topic 28 — Dynamic Inventory

We'll keep this **practical and GCP-focused**, since that's directly relevant to your LevelUp preparation.

---

# 1. 🧠 What is Dynamic Inventory?

You already know a normal/static inventory:

```yaml
all:
  hosts:
    web01:
      ansible_host: 10.10.1.10
    web02:
      ansible_host: 10.10.1.11
    db01:
      ansible_host: 10.10.2.10
```

The problem is that in cloud environments, servers can be created and destroyed frequently.

Instead of manually maintaining:

```text
web01
web02
web03
...
web100
```

Ansible can query the cloud provider and discover the machines automatically.

That's **dynamic inventory**.

---

# 2. 🆚 Static vs Dynamic Inventory

### Static

```text
hosts.yml
    │
    ├── web01
    ├── web02
    ├── web03
    └── db01
```

You maintain it manually.

### Dynamic

```text
              GCP
               │
        Compute Engine API
               │
               ▼
        Ansible inventory
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
     web01   web02   db01
```

Ansible discovers the current infrastructure.

---

# 3. 🏗️ Why Do We Need Dynamic Inventory?

Imagine Terraform creates:

```text
20 VM instances
```

Tomorrow:

```text
30 VM instances
```

Next week:

```text
15 VM instances
```

If you're maintaining:

```yaml
hosts.yml
```

manually, you're constantly updating it.

Dynamic inventory instead asks:

> "What machines currently exist?"

---

# 4. 🔥 Very Important: Dynamic Inventory Does NOT Create Servers

This is a common misunderstanding.

Terraform:

```text
Terraform
   ↓
Create infrastructure
```

Dynamic inventory:

```text
Dynamic Inventory
   ↓
Discover infrastructure
```

So:

```text
Terraform
   ↓
GCP
   ↓
VMs
   ↓
Dynamic Inventory
   ↓
Ansible
```

---

# 5. 🏭 Typical Terraform + Ansible Architecture

This is particularly relevant to your work:

```text
             Terraform
                 │
                 ▼
          GCP Infrastructure
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      web01    web02    db01
        │        │        │
        └────────┼────────┘
                 ▼
          Dynamic Inventory
                 │
                 ▼
              Ansible
                 │
                 ▼
        Configure / Deploy
```

Terraform answers:

> **What infrastructure should exist?**

Ansible answers:

> **How should those machines be configured?**

---

# 6. 🧠 How Does Dynamic Inventory Work?

Conceptually:

```text
Ansible
   │
   │ Query
   ▼
Cloud API
   │
   ▼
Get instances
   │
   ▼
Apply filters/groups
   │
   ▼
Generate inventory
   │
   ▼
Run playbook
```

---

# 7. 🔌 Inventory Plugins

Modern Ansible generally uses **inventory plugins** for dynamic inventory.

For example, GCP has an inventory plugin.

Conceptually:

```yaml
plugin: google.cloud.gcp_compute
```

The plugin knows how to communicate with the GCP API and turn Compute Engine instances into Ansible inventory hosts.

---

# 8. 📦 GCP Collection

The GCP inventory plugin belongs to the Google Cloud Ansible collection.

Typically:

```text
google.cloud
```

You may install the required collection using:

```bash
ansible-galaxy collection install google.cloud
```

In a production project, dependencies are normally captured in:

```text
requirements.yml
```

For example:

```yaml
collections:
  - name: google.cloud
```

Then:

```bash
ansible-galaxy collection install -r requirements.yml
```

---

# 9. 📄 Example GCP Dynamic Inventory

A simplified example:

```yaml
plugin: google.cloud.gcp_compute

projects:
  - my-gcp-project

auth_kind: serviceaccount

service_account_file: /path/to/service-account.json
```

⚠️ The exact options supported can depend on the installed collection/version, so in real projects always verify the plugin documentation.

---

# 10. 🏷️ Grouping Instances

The really useful part is **automatically grouping machines**.

Suppose GCP has:

```text
web-01
web-02
web-03

db-01
db-02
```

You don't necessarily want:

```text
all:
  hosts:
```

You want:

```text
web
db
```

Dynamic inventory can derive groups based on cloud metadata such as:

```text
labels
zones
machine properties
names
network information
```

depending on the inventory plugin.

---

# 11. 🎯 Why Groups Matter

Then your playbook can simply say:

```yaml
- name: Configure web servers
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

You don't care whether there are:

```text
3 servers
```

or:

```text
300 servers
```

The inventory determines which hosts belong to:

```text
web
```

---

# 12. 🏷️ Labels → Groups

A very common cloud pattern is:

```text
GCP instance labels

role=web
role=db
environment=production
```

Conceptually:

```text
GCP
 │
 ├── web-01
 │     label: role=web
 │
 ├── web-02
 │     label: role=web
 │
 └── db-01
       label: role=db
```

Dynamic inventory can use these properties to build useful Ansible groups, depending on the plugin configuration.

Then:

```text
role=web
   ↓
[web]

role=db
   ↓
[db]
```

---

# 13. 🌍 Environment Separation

You can also use labels like:

```text
environment=dev
environment=staging
environment=production
```

Then conceptually:

```text
production
   │
   ├── web
   ├── db
   └── monitoring
```

Your playbook can target:

```yaml
hosts: production
```

or:

```yaml
hosts: production_web
```

depending on how your inventory groups are configured.

---

# 14. 🔍 Inventory Inspection

One of the most important troubleshooting commands:

```bash
ansible-inventory --graph
```

It shows the inventory structure.

For example:

```text
@all:
  |--@web:
  |  |--web-01
  |  |--web-02
  |--@db:
     |--db-01
```

This is extremely useful when debugging dynamic inventory.

---

# 15. 🧠 `ansible-inventory --list`

You can also inspect the complete inventory:

```bash
ansible-inventory --list
```

It outputs structured inventory information.

You can also use:

```bash
ansible-inventory -i gcp_inventory.yml --graph
```

and:

```bash
ansible-inventory -i gcp_inventory.yml --list
```

---

# 16. 🎯 Why `ansible_host` Matters

Suppose your inventory hostname is:

```text
web-01
```

but the actual IP is:

```text
10.10.1.20
```

You can conceptually have:

```yaml
web-01:
  ansible_host: 10.10.1.20
```

Then:

```text
Inventory name
     ↓
web-01

Connection target
     ↓
10.10.1.20
```

This separation is useful.

---

# 17. 🧠 Dynamic Inventory Doesn't Mean "No Variables"

You can still have:

```text
ansible_user
ansible_port
ansible_python_interpreter
ansible_host
```

and other host/group variables.

Dynamic inventory is primarily about **discovering hosts automatically**.

---

# 18. 🔥 Terraform + Dynamic Inventory Example

Imagine Terraform creates:

```text
GCP:
  web-01
  web-02
  web-03
  db-01
```

with labels:

```text
web-01:
  role=web
  env=prod

web-02:
  role=web
  env=prod

web-03:
  role=web
  env=prod

db-01:
  role=db
  env=prod
```

Then:

```text
Terraform
    │
    ▼
GCP
    │
    ▼
Dynamic inventory
    │
    ├── prod_web
    │      ├── web-01
    │      ├── web-02
    │      └── web-03
    │
    └── prod_db
           └── db-01
```

Ansible:

```yaml
- name: Configure production web servers
  hosts: prod_web
```

This is a very clean architecture.

---

# 19. 🧩 Static Inventory Can Also Be Generated

You might hear:

> "We generate an inventory file from Terraform."

That's technically possible.

For example:

```text
Terraform
   ↓
Output IP addresses
   ↓
Generate hosts.yml
   ↓
Ansible
```

But that's **generated static inventory**, not necessarily a true dynamic inventory plugin.

True dynamic inventory:

```text
Ansible inventory plugin
        ↓
Queries cloud API
        ↓
Builds inventory
```

That's an important distinction.

---

# 20. 🆚 Terraform Output vs Dynamic Inventory

|                                            | Terraform Output    | Dynamic Inventory |
| ------------------------------------------ | ------------------- | ----------------- |
| Gets infrastructure data                   | ✅                   | ✅                 |
| Queries cloud API directly                 | Not necessarily     | ✅                 |
| Automatically reflects current cloud state | Depends on workflow | ✅                 |
| Creates infrastructure                     | ✅                   | ❌                 |
| Configures servers                         | ❌                   | ❌                 |
| Used by Ansible                            | Can be              | ✅                 |

---

# 21. 🔐 Authentication

The dynamic inventory plugin needs permission to query GCP.

Conceptually:

```text
Ansible
   │
   │ authentication
   ▼
GCP API
   │
   ▼
Compute Engine instances
```

The identity needs appropriate permissions to discover the resources.

In production, avoid casually using a long-lived service-account JSON key.

Prefer appropriate workload/identity mechanisms supported by your environment.

---

# 22. 🚨 Production Security

Avoid:

```yaml
service_account_file: ./my-secret-key.json
```

and then:

```bash
git add .
git push
```

😨

Instead, use secure credential mechanisms and appropriate GCP identity practices.

The same principle we discussed with Ansible Vault applies:

> **Don't put cloud credentials in Git.**

---

# 23. 🧠 Dynamic Inventory vs `group_vars`

These work together.

For example:

```text
Dynamic inventory
      │
      ▼
Creates:
web
db
monitoring
      │
      ▼
group_vars/
      │
      ├── web.yml
      ├── db.yml
      └── monitoring.yml
```

Then:

```yaml
# group_vars/web.yml

app_port: 8080
```

Every dynamically discovered `web` host can use:

```text
app_port=8080
```

This is a very powerful pattern.

---

# 24. 🏗️ Enterprise Architecture

A realistic cloud automation setup:

```text
                    Git
                     │
            ┌────────┴────────┐
            ▼                 ▼
        Terraform          Ansible
            │                 │
            ▼                 │
           GCP                │
            │                 │
      ┌─────┼─────┐           │
      ▼     ▼     ▼           │
    web01 web02 db01          │
      │     │     │           │
      └─────┼─────┘           │
            │                 │
            ▼                 │
      GCP API / Inventory Plugin
            │                 │
            └────────┬────────┘
                     ▼
                  Inventory
                     │
                     ▼
                 Playbooks
                     │
                     ▼
               Configuration
```

---

# 25. 🧠 When Should You Use Dynamic Inventory?

Good candidates:

### ☁️ Cloud environments

```text
GCP
AWS
Azure
```

### 📈 Auto-scaling environments

Instances constantly appear/disappear.

### 🏢 Large infrastructure

Hundreds/thousands of machines.

### 🔄 Ephemeral infrastructure

Machines are frequently recreated.

---

# 26. When Is Static Inventory Fine?

You don't need dynamic inventory for everything.

For example:

```text
3 permanent on-prem servers
```

with:

```yaml
web:
  hosts:
    web01:
    web02:
    web03:
```

Static inventory is perfectly reasonable.

Don't introduce complexity just because dynamic inventory exists.

---

# 27. 🎤 Interview Question

### "What is dynamic inventory?"

Good answer:

> "Dynamic inventory allows Ansible to discover hosts from an external source such as a cloud provider or CMDB instead of maintaining host definitions manually. An inventory plugin queries the source and generates the Ansible inventory dynamically."

---

# 28. 🎤 Interview Question

### "How would you use dynamic inventory with GCP?"

Good answer:

> "I can use the Google Cloud inventory plugin from the `google.cloud` collection. It queries GCP for Compute Engine instances and can use instance properties such as labels or other metadata to organize hosts into Ansible groups. Then playbooks target those groups rather than maintaining VM IPs manually."

---

# 29. 🎤 Interview Question

### "What happens if a new VM is created?"

Static inventory:

```text
New VM
  ↓
Not automatically known
```

Dynamic inventory:

```text
New VM
  ↓
Cloud API
  ↓
Inventory plugin
  ↓
New host appears
```

That's the key advantage.

---

# 30. 🎤 Interview Question

### "Does dynamic inventory create VMs?"

No.

> "Dynamic inventory only discovers existing infrastructure. Terraform or another provisioning system is responsible for creating the infrastructure."

🔥 Very important distinction.

---

# 31. 🎤 Interview Scenario

### Interviewer:

> "You have 500 GCP VMs and their IPs change frequently. How would you design Ansible inventory?"

Strong answer:

> "I would avoid manually maintaining IP addresses. I'd use the GCP dynamic inventory plugin to query Compute Engine, group instances using appropriate metadata or labels such as environment and role, and target those groups from Ansible. Authentication would use a secure GCP identity mechanism rather than committing service-account credentials."

That's a very good A3 answer.

---

# 32. 🏆 Static → Dynamic Evolution

Think of your Ansible journey:

```text
Small environment
      │
      ▼
Static inventory
      │
      ▼
More servers
      │
      ▼
Cloud infrastructure
      │
      ▼
Dynamic inventory
      │
      ▼
Labels / metadata
      │
      ▼
Automatic grouping
      │
      ▼
Scalable automation
```

---

# 33. 🎯 What You Need to Remember

You don't need to memorize every GCP inventory plugin parameter.

Remember these concepts:

```text
🌐 Dynamic inventory
       ↓
Discover hosts automatically


🔌 Inventory plugin
       ↓
Connects Ansible to external source


☁️ GCP plugin
       ↓
Discover Compute Engine instances


🏷️ Labels / metadata
       ↓
Organize hosts into groups


🔍 ansible-inventory
       ↓
Inspect what Ansible discovered
```

Useful commands:

```bash
ansible-inventory --graph
```

```bash
ansible-inventory --list
```

and with a specific inventory source:

```bash
ansible-inventory -i gcp_inventory.yml --graph
```

---

## ✅ Topic 20 Complete

### 📍 Remaining major topic: **Topic 29 — 🏭 Production Scenarios & Troubleshooting**

This is the **most important final technical section**.

Instead of another theoretical chapter, we'll switch to real situations such as:

```text
🔥 Ansible playbook is very slow
🔥 One server fails during a rolling deployment
🔥 Ansible says "UNREACHABLE"
🔥 sudo/become fails
🔥 Python missing on managed node
🔥 Playbook works manually but fails in CI
🔥 A task reports changed every time
🔥 Handler doesn't execute
🔥 Wrong variable is being used
🔥 Template works on one server but not another
🔥 100 production servers need a safe upgrade
🔥 Database cluster rolling maintenance
🔥 How to troubleshoot an Ansible failure
```

Then we'll finish with **Topic 30 — A3/LevelUp Ansible Interview Round**. 🚀



# 🏭 Ansible Topic 29 — Production Scenarios & Troubleshooting

This is the **most important practical topic** before we finish with the interview round. 🔥

At A3/Senior level, interviewers usually don't only ask:

> "What is `become`?"

They ask:

> **"Your Ansible deployment failed on 20 out of 200 servers. What will you do?"**

So let's learn the **troubleshooting mindset**.

---

# 1. 🧠 The Universal Ansible Troubleshooting Flow

Whenever an Ansible task fails, don't randomly change things.

Use:

```text
                  ❌ Ansible Failure
                         │
                         ▼
                1️⃣ Read the error
                         │
                         ▼
                2️⃣ Identify the layer
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      Connection      Privilege       Module/
         SSH            sudo           Task
          │              │              │
          ▼              ▼              ▼
       Network        become          Logic
                         │
                         ▼
                  3️⃣ Reproduce
                         │
                         ▼
                  4️⃣ Fix minimally
                         │
                         ▼
                  5️⃣ Test again
```

The key is identifying **which layer failed**.

---

# 2. 🔌 Scenario: `UNREACHABLE`

You run:

```bash
ansible-playbook site.yml
```

and get:

```text
fatal: [web01]: UNREACHABLE!
```

This usually means Ansible couldn't establish the connection.

Think:

```text
Control Node
     │
     │ SSH ❌
     ▼
Managed Node
```

This is **not necessarily a playbook/task problem**.

---

# 3. 🔍 Troubleshooting `UNREACHABLE`

Check connectivity first:

```bash
ssh ansible@web01
```

If SSH itself fails:

```text
Ansible
   ↓
not the first thing to debug
   ↓
SSH/network
```

Check:

```text
hostname/IP
DNS
routing
firewall
port 22
SSH service
username
private key
bastion/jump host
```

---

# 4. 🛠️ Test With Ansible Ping

Run:

```bash
ansible web01 -m ansible.builtin.ping
```

Expected:

```text
web01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Remember:

> Ansible `ping` does **not** mean ICMP ping.

It actually tests Ansible connectivity and module execution.

---

# 5. 🎯 `ping` vs Network Ping

Don't confuse:

```bash
ping 10.10.1.20
```

with:

```bash
ansible web01 -m ansible.builtin.ping
```

The first checks:

```text
ICMP/network reachability
```

The second checks:

```text
Ansible connection
+
remote module execution
```

A server can reject ICMP but still allow SSH.

---

# 6. 🔑 Scenario: Permission Denied

Error:

```text
Permission denied (publickey)
```

Likely causes:

```text
Wrong username
Wrong private key
Public key missing
Wrong permissions
SSH configuration
Wrong host
```

Check manually:

```bash
ssh -i ~/.ssh/ansible_key ansible@10.10.1.20
```

If this doesn't work, fix SSH before debugging Ansible.

---

# 7. 🔐 Scenario: `become` Failure

Suppose:

```yaml
become: true
```

but you get:

```text
Missing sudo password
```

or:

```text
Sorry, user ansible is not allowed to execute ...
```

Now the SSH connection worked:

```text
Control Node
     │
     │ SSH ✅
     ▼
ansible user
     │
     │ sudo ❌
     ▼
root
```

So the problem is **privilege escalation**, not connectivity.

---

# 8. 🧠 Troubleshoot `become`

Check:

```bash
sudo -l
```

on the managed node.

You need to understand:

```text
Can ansible user use sudo?
Does sudo require a password?
Is the required command allowed?
Is become_user correct?
```

Example:

```yaml
become: true
become_user: postgres
```

requires appropriate privilege escalation permissions.

---

# 9. 🐍 Scenario: Python Missing

Error might look like:

```text
/ usr/bin/python: No such file or directory
```

or:

```text
Interpreter discovery failed
```

Remember:

```text
SSH works
    ↓
Ansible connects
    ↓
Python/module execution fails
```

Check:

```bash
ssh ansible@web01
python3 --version
```

You can explicitly specify:

```yaml
ansible_python_interpreter: /usr/bin/python3
```

---

# 10. 📦 Scenario: Package Installation Fails

Example:

```text
No package nginx available
```

Don't immediately blame Ansible.

Check:

```text
OS distribution
repository configuration
network access
package manager
package name
repository availability
```

For example:

```yaml
ansible.builtin.package:
  name: nginx
  state: present
```

is valid Ansible, but the OS may not have a repository containing `nginx`.

---

# 11. 🧠 Scenario: Playbook Works on One OS but Not Another

Suppose:

```text
RHEL → works
Ubuntu → fails
```

Use facts:

```yaml
when: ansible_os_family == "RedHat"
```

or:

```yaml
when: ansible_os_family == "Debian"
```

Better architecture:

```text
common tasks
      │
      ├── RedHat-specific
      │
      └── Debian-specific
```

Don't blindly assume all Linux distributions behave identically.

---

# 12. 🔎 Scenario: Need More Information

Use verbose mode:

```bash
ansible-playbook site.yml -v
```

More verbose:

```bash
ansible-playbook site.yml -vv
```

Even more:

```bash
ansible-playbook site.yml -vvv
```

Maximum:

```bash
ansible-playbook site.yml -vvvv
```

---

# 13. 🧠 Verbosity Levels

Think:

```text
-v
 ↓
more information

-vv
 ↓
more detail

-vvv
 ↓
connection/debug details

-vvvv
 ↓
very detailed SSH debugging
```

Don't normally run `-vvvv` in routine CI pipelines because the output can become huge and potentially expose sensitive details.

---

# 14. 🎯 Scenario: Playbook Is Slow

Suppose:

```text
200 servers
```

and execution takes:

```text
3 hours
```

Don't immediately say:

```ini
forks = 100
```

Investigate:

```text
1. forks
2. serial
3. strategy
4. fact gathering
5. fact caching
6. SSH/network latency
7. slow tasks
8. throttle
9. package/repository performance
10. control-node resources
```

You learned these concepts in Topic 17.

---

# 15. 🚀 Scenario: Rolling Deployment

Requirement:

> Update 100 production servers, but only 10 at a time.

Use:

```yaml
serial: 10
```

But production-grade automation should be:

```text
10 servers
    ↓
drain
    ↓
deploy
    ↓
restart
    ↓
health check
    ↓
enable
    ↓
next 10
```

Not simply:

```yaml
serial: 10
```

and assume everything is safe.

---

# 16. 🔥 Scenario: One Batch Fails

Suppose:

```yaml
serial: 10
```

Batch 1:

```text
web01 → success
web02 → success
web03 → success
web04 → failure
...
```

Don't blindly continue.

Investigate:

```text
What failed?
Why?
Is the application healthy?
Should the rollout stop?
Should we rollback?
```

Use appropriate controls such as:

```yaml
max_fail_percentage: 20
```

or:

```yaml
any_errors_fatal: true
```

depending on your requirement.

---

# 17. 🏥 Production Health Check

A strong deployment doesn't simply do:

```text
restart
 ↓
next server
```

Instead:

```text
restart
  ↓
wait_for port
  ↓
HTTP health check
  ↓
application healthy?
  ├── YES → continue
  └── NO  → stop / rollback
```

Example:

```yaml
- name: Wait for application
  ansible.builtin.wait_for:
    port: 8080
    timeout: 60

- name: Check application health
  ansible.builtin.uri:
    url: "http://{{ inventory_hostname }}:8080/health"
    status_code: 200
```

---

# 18. 🔄 Scenario: Task Reports `changed` Every Time

You run:

```bash
ansible-playbook site.yml
```

and:

```text
changed=5
```

Run again:

```text
changed=5
```

That's suspicious.

Look for:

```text
command
shell
timestamp changes
random values
templates with dynamic content
poorly designed conditions
```

Ask:

> "Does this task describe the desired state, or am I just executing a command?"

---

# 19. 🧠 Better Module Usage

Bad:

```yaml
- name: Create directory
  ansible.builtin.command:
    cmd: mkdir -p /opt/myapp
```

Better:

```yaml
- name: Ensure application directory exists
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
```

Why?

```text
command
 ↓
"I executed something"


file
 ↓
"I want this state"
```

Ansible is designed around desired state.

---

# 20. 🧪 Scenario: Check Mode Gives Unexpected Result

You run:

```bash
ansible-playbook site.yml --check
```

and it doesn't correctly predict a change.

Possible reason:

```text
module doesn't support check mode fully
```

or:

```text
command/shell
```

is being used.

Don't assume:

```text
--check = perfect simulation
```

It isn't.

---

# 21. 🔍 Scenario: Template Is Wrong

Suppose:

```yaml
- name: Deploy config
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
```

But the resulting config is wrong.

Check:

```text
template variables
variable precedence
facts
group_vars
host_vars
role defaults
extra vars
```

You can inspect variables:

```yaml
- name: Show variable
  ansible.builtin.debug:
    var: app_port
```

⚠️ Never debug sensitive variables.

---

# 22. 🧠 Scenario: "Wrong Variable Value"

Example:

```text
Expected:
app_port = 8080

Actual:
app_port = 9090
```

Don't immediately modify the variable.

Find where it is defined:

```text
role defaults
group_vars
host_vars
play vars
set_fact
include_vars
extra vars
```

Then determine which source has precedence.

You skipped the detailed precedence topic earlier, but **you should still know this troubleshooting concept**.

---

# 23. 🔔 Scenario: Handler Didn't Run

You have:

```yaml
notify: Restart nginx
```

but nginx didn't restart.

Check:

```text
Did the task report changed?
Does the handler name exactly match?
Did the task actually execute?
Was the handler flushed?
Was the play interrupted?
```

Remember:

```text
notify
  ↓
handler runs at normal handler execution point
```

A handler normally runs only when the notifying task reports a change.

---

# 24. 🧠 Scenario: Handler Needs to Run Immediately

Sometimes you need:

```yaml
- name: Flush handlers
  ansible.builtin.meta: flush_handlers
```

Flow:

```text
Configuration changed
       ↓
notify handler
       ↓
flush_handlers
       ↓
restart immediately
       ↓
continue
```

This is useful when subsequent tasks depend on the service already being restarted.

---

# 25. 🔐 Scenario: Secret Appears in CI Logs

You see:

```text
TASK [Configure database]
changed: ...
db_password=SuperSecret123
```

Immediately investigate:

```text
debug tasks
command/shell output
registered variables
diff mode
verbose output
```

Use:

```yaml
no_log: true
```

for sensitive operations.

And keep the source secret in:

```text
Ansible Vault
or
Secret Manager
```

---

# 26. 🚨 Scenario: Vault Decryption Fails

Error:

```text
Decryption failed
```

Check:

```text
Vault password
Vault ID
correct secret
CI secret injection
password file
```

If using Vault IDs:

```bash
--vault-id prod@prompt
```

make sure the correct credential is available.

---

# 27. 🏗️ Scenario: Works Locally but Fails in CI

This is extremely common.

```text
Developer laptop
     │
     ▼
Works ✅


CI
     │
     ▼
Fails ❌
```

Compare:

```text
Ansible version
Python version
collection versions
ansible.cfg
environment variables
SSH credentials
Vault credentials
inventory
working directory
environment variables
file paths
```

---

# 28. 🔥 Dependency Pinning

A common cause is:

```text
Developer:
Ansible 2.x
collection 10.x

CI:
Ansible 2.y
collection 11.x
```

Behavior may differ.

Use:

```text
requirements.yml
```

and appropriate dependency/version management.

For production, reproducibility matters.

---

# 29. 📦 Scenario: Collection Module Not Found

Error:

```text
couldn't resolve module/action
```

For example:

```yaml
google.cloud.some_module:
```

Check:

```bash
ansible-galaxy collection list
```

and:

```bash
ansible-galaxy collection install -r requirements.yml
```

Also verify:

```text
collection name
module name
FQCN
collection version
```

---

# 30. 🌐 Scenario: Dynamic Inventory Doesn't Find Hosts

Run:

```bash
ansible-inventory -i inventory.yml --graph
```

If nothing appears:

```text
Check plugin
     ↓
Check authentication
     ↓
Check project
     ↓
Check permissions
     ↓
Check filters
     ↓
Check labels/metadata
     ↓
Check collection version
```

Don't start debugging the playbook yet.

First make sure:

> **Ansible actually sees the hosts.**

---

# 31. 🎯 Scenario: Only Some Hosts Fail

Suppose:

```text
100 hosts
95 success
5 failure
```

This is very different from:

```text
100 hosts
100 failure
```

95/5 suggests:

```text
host-specific issue
```

Look for:

```text
OS differences
network
disk
permissions
package repository
configuration drift
Python version
service state
```

---

# 32. 🧠 Scenario: All Hosts Fail

If:

```text
100/100 fail
```

think about shared infrastructure:

```text
inventory
SSH credentials
network
DNS
bastion
Ansible configuration
Vault
collection
playbook syntax
```

A useful troubleshooting heuristic:

```text
One host fails
   ↓
Host-specific problem likely


All hosts fail
   ↓
Shared configuration/environment likely
```

Not a hard rule, but a useful first direction.

---

# 33. 🗄️ Database Production Scenario

Since your work involves PostgreSQL/Patroni, imagine:

```text
Patroni cluster

Node1 → Leader
Node2 → Replica
Node3 → Replica
```

You need to upgrade PostgreSQL.

**Don't do:**

```yaml
serial: 3
```

and restart all nodes.

A safer approach:

```text
Check cluster health
       ↓
Identify replica
       ↓
Upgrade replica
       ↓
Validate replication
       ↓
Next replica
       ↓
Controlled leader switchover
       ↓
Upgrade former leader
       ↓
Validate cluster
```

Ansible controls the workflow, but **Patroni/database health determines whether it is safe to proceed**.

That's the kind of production thinking interviewers like.

---

# 34. 🏭 Infrastructure Production Scenario

Imagine:

```text
200 application servers
```

Requirement:

> Deploy a new version without taking the whole service down.

Architecture:

```text
             Load Balancer
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
     App01      App02      App03
       │
       ▼
    Ansible
       │
   serial: 10
       │
       ▼
   Drain 10
       │
       ▼
   Deploy 10
       │
       ▼
   Health check
       │
       ▼
   Enable 10
       │
       ▼
   Next batch
```

This combines:

```text
serial
delegate_to
wait_for
uri
handlers
failure controls
```

---

# 35. 🚨 Scenario: Disk Full

Ansible reports:

```text
No space left on device
```

Don't repeatedly retry.

Check:

```bash
df -h
```

and:

```bash
df -i
```

because it could be:

```text
disk capacity
```

or:

```text
inode exhaustion
```

Then investigate:

```text
logs
/tmp
package cache
old artifacts
Docker/container data
```

The important lesson:

> Ansible is reporting an infrastructure problem; Ansible itself may not be the problem.

---

# 36. 🔥 Scenario: Service Won't Start

Ansible:

```yaml
state: started
```

fails.

Don't just rerun.

On the managed host check:

```bash
systemctl status myapp
```

and:

```bash
journalctl -u myapp
```

Then:

```text
Configuration
Dependencies
Permissions
Port conflicts
Environment variables
Certificates
Disk space
SELinux
```

Ansible is only the automation layer.

---

# 37. 🛡️ Scenario: SELinux Causes Failure

On RHEL systems, you may have:

```text
SELinux enforcing
```

A file might have correct Unix permissions but still be inaccessible because of SELinux context.

Check:

```bash
getenforce
```

and:

```bash
ls -Z /path/to/file
```

Then fix the underlying configuration appropriately rather than simply disabling SELinux.

---

# 38. 🧠 Production Troubleshooting Layers

This is a powerful mental model:

```text
              Ansible Failure
                    │
        ┌───────────┼────────────┐
        ▼           ▼            ▼
      Control     Network      Managed
       Node                     Node
        │           │            │
      Python      SSH/DNS     OS/service
      Config      Firewall     Package
      Inventory   Routing      Permissions
      Plugins     Bastion      SELinux
                                Disk
```

Ask:

> **Which layer is actually broken?**

---

# 39. 🔍 Useful Commands Cheat Sheet

### Test inventory

```bash
ansible-inventory --graph
```

### Test connectivity

```bash
ansible all -m ansible.builtin.ping
```

### Test specific host

```bash
ansible web01 -m ansible.builtin.ping
```

### Syntax

```bash
ansible-playbook site.yml --syntax-check
```

### Dry run

```bash
ansible-playbook site.yml --check
```

### Diff

```bash
ansible-playbook site.yml --diff
```

### More debugging

```bash
ansible-playbook site.yml -vvv
```

### Very detailed SSH debugging

```bash
ansible-playbook site.yml -vvvv
```

### Show Ansible configuration

```bash
ansible-config dump
```

### Show version/config path

```bash
ansible --version
```

### Check installed collections

```bash
ansible-galaxy collection list
```

---

# 40. 🎤 A3 Scenario — "Ansible Is Failing"

### Interviewer:

> "Your Ansible playbook is failing in production. What do you do?"

Don't answer:

> "I run it again."

😄

Give this structured answer:

> "First I identify whether the failure is at the inventory, connectivity, privilege escalation, Python/module, or task/application layer. I check the exact error, validate inventory with `ansible-inventory`, test connectivity with the Ansible ping module, and if needed test SSH manually. I then inspect privilege escalation, Python interpreter and module/collection availability. For task-level issues I use appropriate verbosity, check mode or targeted execution. After identifying the root cause, I make the smallest safe fix and validate it in a controlled environment before retrying production."

🔥 That's a **strong Senior Engineer answer**.

---

# 41. 🎤 Scenario — "100 Servers, 5 Failed"

Answer:

> "I wouldn't immediately rerun the entire playbook. I'd isolate the five failed hosts, determine whether the failures have a common cause, validate the successful hosts are healthy, fix the underlying issue, and rerun only the affected hosts using `--limit` where appropriate."

Example:

```bash
ansible-playbook site.yml --limit "web01:web02:web03"
```

This is very useful.

---

# 42. 🎯 `--limit`

Suppose:

```text
100 hosts
```

but only:

```text
web01
web02
web03
```

need retry.

Use:

```bash
ansible-playbook site.yml \
  --limit "web01:web02:web03"
```

This allows targeted execution.

Think:

```text
Inventory = all available targets

--limit = subset for this execution
```

---

# 43. 🔥 Scenario — Production Emergency

Imagine a deployment failed halfway.

You have:

```text
100 servers

70 upgraded
30 not upgraded
```

Don't blindly rerun everything.

First determine:

```text
Which servers changed?
Which are healthy?
Which failed?
What version is each server running?
```

Then use inventory/grouping/limits to safely converge the remaining systems.

This is one of the biggest advantages of idempotent infrastructure automation.

---

# 44. 🧠 Idempotency Helps Recovery

Imagine:

```text
Deployment:
1 → success
2 → success
3 → success
...
70 → success
71 → failure
```

If your playbook is properly idempotent:

```text
rerun
```

should generally result in:

```text
1-70 → already correct
71 → fix
72-100 → configure
```

instead of destroying/recreating everything.

That's one of the biggest reasons idempotency matters.

---

# 45. 🏆 Production Troubleshooting Framework

Memorize this:

```text
        ❌ FAILURE
            │
            ▼
       READ ERROR
            │
            ▼
     IDENTIFY LAYER
            │
     ┌──────┼───────┐
     ▼      ▼       ▼
   SSH    Become   Task
     │      │       │
     ▼      ▼       ▼
  Network  sudo   Module
                    │
                    ▼
                 Logic
                    │
                    ▼
              REPRODUCE
                    │
                    ▼
                FIX ROOT
                 CAUSE
                    │
                    ▼
                TEST
                    │
                    ▼
               DEPLOY
```

---

# 46. 🎯 The Most Important Production Scenarios

For your LevelUp preparation, be comfortable explaining these:

| Scenario                | What you should think about |
| ----------------------- | --------------------------- |
| `UNREACHABLE`           | SSH/network/DNS/bastion     |
| `Permission denied`     | User/key/SSH                |
| `become` failure        | sudo/privileges             |
| Python missing          | Interpreter                 |
| Module not found        | Collection/FQCN             |
| Dynamic inventory empty | Plugin/API/auth/filter      |
| Playbook slow           | forks/serial/facts/network  |
| Repeated `changed`      | Idempotency                 |
| Handler didn't run      | notify/change/handler       |
| Secret in logs          | Vault/`no_log`              |
| Only some hosts fail    | Host-specific issue         |
| All hosts fail          | Shared environment issue    |
| Production rollout      | serial + health checks      |
| Partial deployment      | `--limit` + idempotency     |
| Service won't start     | OS/service/config/logs      |

---

# 🏆 Topic 29 — What You Should Be Able to Do Now

You should now be able to approach an Ansible failure as:

```text
❌ "Ansible failed"
       ↓
❓ Where did it fail?
       ↓
Connection?
Privilege?
Python?
Module?
Task?
Application?
       ↓
🔍 Investigate that layer
       ↓
🛠️ Fix root cause
       ↓
🧪 Validate
       ↓
🚀 Resume safely
```

That is much closer to **Senior Engineer thinking** than simply knowing Ansible syntax.

---

## ✅ Topic 29 Complete

We've now covered essentially all the **Ansible knowledge that is relevant to your current LevelUp preparation**, while skipping the areas you already know.

### 🎤 Next: Topic 30 — Ansible A3 / LevelUp Interview Preparation

This will be different from the previous topics.

I'll give you **production-style interview questions**, including:

```text
🔥 Basic → Intermediate → Advanced
🔥 Scenario-based questions
🔥 Troubleshooting questions
🔥 serial / forks / strategy
🔥 Idempotency
🔥 Roles
🔥 Vault
🔥 Dynamic inventory
🔥 Molecule/testing
🔥 GCP + Ansible
🔥 Terraform + Ansible
🔥 PostgreSQL/production scenarios
🔥 "What would you do?" questions
```

And most importantly, I'll show you **what a weak answer vs a strong A3 answer sounds like**.



# 🎤 Ansible Topic 30 — A3 / LevelUp Interview Preparation

Now we'll switch from **learning concepts → answering like a Senior System Engineer**. 🔥

The goal is not to memorize answers. For each question, think:

> **Concept → Production impact → How I would implement/troubleshoot it**

---

# 1. 🟢 Fundamentals

### Q1. What is Ansible?

**Strong A3 answer:**

> Ansible is an agentless automation and configuration-management tool. It typically connects to Linux managed nodes using SSH and executes modules remotely. It uses declarative/idempotent modules where possible, allowing infrastructure and configuration to converge toward a desired state.

Don't stop at:

> "Ansible is an automation tool."

That's too basic for A3.

---

### Q2. Why is Ansible called agentless?

Because you generally don't install an Ansible agent on managed Linux servers.

```text
Control Node
     │
     │ SSH
     ▼
Managed Node
```

The managed node needs the required runtime, commonly Python for many modules, but not an Ansible agent.

---

# 2. 🔐 SSH + Become

### Q3. Explain the difference between `ansible_user` and `become_user`.

Example:

```yaml
ansible_user: ansible
become: true
become_user: postgres
```

Flow:

```text
Control Node
     │
     │ SSH
     ▼
 ansible user
     │
     │ sudo
     ▼
 postgres
```

**Answer:**

> `ansible_user` is the user used to establish the connection. `become_user` is the user Ansible switches to for task execution through privilege escalation.

---

# 3. 🔥 Idempotency

### Q4. What is idempotency?

> An operation is idempotent when applying it repeatedly produces the same desired final state without causing unnecessary changes.

Example:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

First run:

```text
changed=1
```

Second run:

```text
changed=0
```

---

### Q5. How do you make a `shell` or `command` task idempotent?

Don't immediately use:

```yaml
command: some-command
```

First ask:

> Is there an Ansible module that represents the desired state?

If not, use mechanisms such as:

```yaml
creates:
removes:
changed_when:
failed_when:
```

Example:

```yaml
- name: Run initialization
  ansible.builtin.command:
    cmd: /opt/app/init.sh
    creates: /opt/app/.initialized
```

Now Ansible doesn't repeatedly execute it after the marker exists.

---

# 4. 🚀 `serial`

### Q6. You have 100 production servers. You want to deploy 10 at a time. What do you use?

```yaml
serial: 10
```

But a strong answer continues:

> I would combine the rolling deployment with health checks and, where applicable, load-balancer draining. I would validate each batch before continuing.

Architecture:

```text
100 servers
     │
     ▼
serial: 10
     │
     ▼
Drain → Deploy → Restart → Health Check
     │
     ▼
Next batch
```

---

# 5. 🧠 `forks` vs `serial`

### Q7. What's the difference?

**Excellent answer:**

> `forks` controls Ansible's worker concurrency, whereas `serial` controls the number of hosts included in each play batch.

Remember:

```text
forks
 ↓
worker capacity

serial
 ↓
host batch size
```

---

# 6. 🧩 `throttle`

### Q8. What if you want only one particular task to run on two hosts simultaneously?

Use:

```yaml
throttle: 2
```

Example:

```yaml
- name: Perform expensive operation
  ansible.builtin.command:
    cmd: /opt/scripts/heavy-operation.sh
  throttle: 2
```

**Interview answer:**

> `throttle` limits concurrency for a particular task or block, unlike `forks`, which provides overall worker capacity.

---

# 7. 🔄 Strategy

### Q9. Difference between `linear` and `free`?

### `linear`

```text
Host 1 ── Task 1 ── Task 2
Host 2 ── Task 1 ── Task 2
Host 3 ── Task 1 ── Task 2
```

Hosts progress through tasks in a coordinated manner.

### `free`

```text
Host 1 ── Task 1 ── Task 2 ── Task 3
Host 2 ── Task 1 ───────────── Task 2
Host 3 ── Task 1 ── Task 2
```

A host doesn't need to wait for slower hosts before progressing.

---

# 8. 🏭 Production Scenario

### Q10. You're deploying to 200 production servers. How would you design it?

A strong answer:

> I'd use a rolling deployment rather than updating everything simultaneously. I'd use `serial` to control batch size, drain nodes from the load balancer if applicable, deploy the application, perform health checks, and only then proceed to the next batch. I'd also configure appropriate failure handling and make sure the playbook is idempotent.

Architecture:

```text
             Load Balancer
                  │
                  ▼
           ┌─────────────┐
           │ 200 servers │
           └──────┬──────┘
                  │
             serial: 10
                  │
                  ▼
            ┌───────────┐
            │ 10 hosts  │
            └─────┬─────┘
                  │
          Drain → Deploy
                  │
               Restart
                  │
            Health Check
                  │
            ┌─────┴─────┐
            │           │
           PASS        FAIL
            │           │
            ▼           ▼
       Next batch      Stop
```

🔥 This is a strong LevelUp answer.

---

# 9. 🔐 Ansible Vault

### Q11. How do you protect secrets in Ansible?

Use:

```text
Ansible Vault
```

Example:

```bash
ansible-vault encrypt secrets.yml
```

Then:

```bash
ansible-playbook site.yml --ask-vault-pass
```

Production answer should also mention:

> In enterprise environments, credentials may additionally be integrated with a secret-management system or AWX/AAP credentials rather than storing secrets directly in repositories.

---

# 10. 🚨 Secret Exposure

### Q12. A password appeared in Ansible output. What do you do?

Check for:

```text
debug
register output
verbose logging
diff
command/shell output
```

Use:

```yaml
no_log: true
```

for sensitive tasks where appropriate.

And ensure secrets are stored securely.

---

# 11. 📦 Roles

### Q13. Why use Ansible roles?

A good answer:

> Roles provide a standardized structure for reusable automation. They separate tasks, handlers, templates, defaults, variables and files, making automation easier to maintain, test and reuse.

Example:

```text
roles/
└── nginx/
    ├── tasks/
    ├── handlers/
    ├── templates/
    ├── files/
    ├── defaults/
    └── vars/
```

---

# 12. 🧠 Role vs Playbook

Don't say:

> "They're the same."

Think:

```text
Playbook
   ↓
Orchestrates what should happen

Role
   ↓
Reusable implementation
```

Example:

```yaml
- hosts: web
  roles:
    - nginx
    - monitoring
```

---

# 13. 🌐 Dynamic Inventory

### Q14. You have 500 GCP VMs. Their IP addresses change frequently. What do you do?

Strong answer:

> I would use a GCP dynamic inventory plugin rather than manually maintaining IP addresses. The inventory plugin can query GCP and organize instances into Ansible groups based on appropriate metadata or labels.

Architecture:

```text
GCP
 │
 ▼
Compute Engine API
 │
 ▼
Dynamic Inventory
 │
 ├── web
 ├── database
 └── monitoring
 │
 ▼
Ansible Playbook
```

---

# 14. 🔥 Terraform + Ansible

### Q15. How would you combine Terraform and Ansible?

This is especially important for your background.

Answer:

> I'd generally use Terraform for infrastructure provisioning and Ansible for post-provisioning configuration and application setup.

```text
Terraform
   ↓
VPC / VM / Firewall / LB
   ↓
GCP
   ↓
Dynamic Inventory
   ↓
Ansible
   ↓
OS / Packages / Config
   ↓
Application
```

Simple mental model:

```text
Terraform → infrastructure

Ansible → configuration
```

---

# 15. 🧪 Testing

### Q16. How do you test an Ansible role?

Strong answer:

```text
yamllint
   ↓
ansible-lint
   ↓
syntax-check
   ↓
Molecule
   ↓
converge
   ↓
verify
   ↓
idempotency
   ↓
pytest/assertions
```

Don't just say:

> "I run the playbook."

---

# 16. 🧪 Idempotency Testing

### Q17. How would you verify idempotency?

Run the automation twice.

```text
First run
changed > 0

Second run
changed = 0
```

Then investigate any unexpected changes.

---

# 17. 🔍 Troubleshooting

### Q18. Ansible says `UNREACHABLE`. What do you check?

Use this order:

```text
1. Inventory
2. ansible_host
3. DNS/IP
4. SSH connectivity
5. Username
6. Private key
7. Port 22
8. Firewall/network
9. Bastion/ProxyJump
10. SSH service
```

Test:

```bash
ssh ansible@server
```

Then:

```bash
ansible server -m ansible.builtin.ping
```

---

# 18. 🔐 `become` Failure

### Q19. SSH works but `become: true` fails. What do you check?

This is an important distinction:

```text
SSH ✅
  ↓
ansible user
  ↓
sudo ❌
```

Check:

```bash
sudo -l
```

Then verify:

```text
sudo permissions
sudo password requirement
become_method
become_user
sudoers configuration
```

---

# 19. 🐍 Python Failure

### Q20. SSH works but Ansible modules fail because Python isn't available. What do you do?

Check:

```bash
python3 --version
```

Then configure:

```yaml
ansible_python_interpreter: /usr/bin/python3
```

or bootstrap/install the required Python runtime.

---

# 20. 🔎 Debugging

### Q21. How do you increase Ansible debugging information?

```bash
-v
-vv
-vvv
-vvvv
```

For example:

```bash
ansible-playbook site.yml -vvv
```

Use `-vvvv` primarily when you need detailed SSH-level debugging.

---

# 21. 🎯 `--limit`

### Q22. 3 servers failed out of 100. Would you rerun the entire playbook?

Not necessarily.

Use:

```bash
ansible-playbook site.yml \
  --limit "server1:server2:server3"
```

First investigate why they failed.

This is especially useful for production recovery.

---

# 22. 🧠 Partial Deployment

### Q23. Deployment stopped after 70 out of 100 servers. How do you recover?

Strong answer:

> First I'd identify which hosts successfully converged and which failed. I'd verify the application state and investigate the failed hosts. Because the automation should be idempotent, I'd fix the root cause and rerun against the affected or remaining hosts using `--limit` when appropriate rather than blindly repeating the entire deployment.

🔥 Excellent production thinking.

---

# 23. 🚨 Playbook Works Locally But Not in CI

### Q24. What do you check?

Compare:

```text
Ansible version
Python version
Collection versions
ansible.cfg
Inventory
Credentials
Vault
Environment variables
Working directory
SSH configuration
```

A very common solution is to **pin dependencies** and make the CI environment reproducible.

---

# 24. 🐌 Slow Ansible

### Q25. Your playbook takes 2 hours for 500 servers. How do you troubleshoot?

Think:

```text
forks
serial
strategy
fact gathering
fact caching
SSH latency
network
slow modules
package repositories
throttle
control-node resources
```

Don't simply increase:

```ini
forks = 500
```

without understanding the infrastructure limits.

---

# 25. 🔔 Handler Scenario

### Q26. A configuration changed but the service wasn't restarted. What do you check?

```text
Did the task report changed?
        ↓
Did it have notify?
        ↓
Does handler name match?
        ↓
Did the play reach handler execution?
        ↓
Was flush_handlers required?
```

Example:

```yaml
notify:
  - Restart nginx
```

---

# 26. 🎯 `delegate_to`

### Q27. You are configuring web servers but need to remove each server from a load balancer. Where should the task execute?

Use:

```yaml
delegate_to: load_balancer
```

Conceptually:

```text
Play host:
web01

Actual execution:
load-balancer
```

---

# 27. 🏃 `run_once`

### Q28. You have 100 hosts but need a task to execute only once.

```yaml
run_once: true
```

For example:

```yaml
- name: Create deployment record
  ansible.builtin.command:
    cmd: /opt/deploy/create-record
  run_once: true
```

---

# 28. 🧠 Advanced Scenario

### Q29. What's the difference between:

```yaml
delegate_to: localhost
```

and:

```yaml
connection: local
```

Conceptually:

```text
connection: local
```

means the host's tasks use a local connection.

Whereas:

```yaml
delegate_to: localhost
```

means:

> This particular task should execute on localhost instead of its normal target.

This distinction is useful in orchestration workflows.

---

# 29. 🏆 Senior-Level Scenario

### Q30. You need to upgrade a 3-node Patroni cluster.

A weak answer:

> "I'll use Ansible to upgrade all nodes."

A strong answer:

> "I would first understand the cluster topology and current health. I wouldn't update all nodes simultaneously. I'd maintain quorum and availability, typically handling replicas first, validating replication and cluster health after each operation, and carefully handling the leader through an appropriate controlled switchover. The Ansible playbook would enforce sequencing, validation and failure handling, while Patroni remains responsible for cluster state."

Architecture:

```text
             Patroni Cluster
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       Leader   Replica   Replica
          │        │        │
          └────────┼────────┘
                   │
              Ansible
                   │
        controlled maintenance
```

This is the type of answer that demonstrates **production experience**, not just Ansible knowledge.

---

# 30. 🔥 Rapid-Fire Questions

These are worth answering quickly in an interview.

| Question                      | Answer                                     |
| ----------------------------- | ------------------------------------------ |
| Agentless?                    | Usually SSH/WinRM without an Ansible agent |
| Default Linux connection?     | SSH                                        |
| Privilege escalation?         | `become`                                   |
| Rolling deployment?           | `serial`                                   |
| Worker concurrency?           | `forks`                                    |
| Task concurrency?             | `throttle`                                 |
| Independent host scheduling?  | `strategy: free`                           |
| Default strategy?             | `linear`                                   |
| Execute task elsewhere?       | `delegate_to`                              |
| Execute once?                 | `run_once`                                 |
| Dry run?                      | `--check`                                  |
| Configuration changes?        | `--diff`                                   |
| More debug?                   | `-v` through `-vvvv`                       |
| Limit hosts?                  | `--limit`                                  |
| Encrypt secrets?              | Ansible Vault                              |
| Test roles?                   | Molecule                                   |
| YAML quality?                 | yamllint                                   |
| Ansible quality?              | ansible-lint                               |
| Discover cloud hosts?         | Dynamic inventory                          |
| Repeated no-change execution? | Idempotency                                |

---

# 31. 🎯 The 10 Questions I Would Definitely Prepare

If your interviewer has limited time, these are the ones I'd be **very comfortable answering**:

### 🔥 1.

**What is idempotency and how do you achieve it in Ansible?**

### 🔥 2.

**Difference between `serial`, `forks`, `strategy` and `throttle`?**

### 🔥 3.

**How would you perform a zero/minimal-downtime rolling deployment?**

### 🔥 4.

**Ansible says `UNREACHABLE`. How do you troubleshoot it?**

### 🔥 5.

**SSH works but `become` fails. What do you check?**

### 🔥 6.

**How do you test an Ansible role?**

### 🔥 7.

**How would you manage secrets?**

### 🔥 8.

**How would you manage 500 dynamic GCP instances?**

### 🔥 9.

**Terraform vs Ansible — when do you use each?**

### 🔥 10.

**A production playbook failed halfway through. How do you safely recover?**

If you can answer those naturally, you're in a **very good position** for the Ansible portion of an A3 interview.

---

# 🏆 Your Ansible Mental Model

You don't need to remember hundreds of Ansible modules.

Think about Ansible as:

```text
                         ANSIBLE
                            │
       ┌────────────────────┼────────────────────┐
       ▼                    ▼                    ▼
   CONNECT                EXECUTE              CONTROL
       │                    │                    │
       ▼                    ▼                    ▼
      SSH                 Modules             serial
    become               Roles               forks
   inventory             Templates           strategy
   dynamic               Handlers            throttle
       │                    │                    │
       └────────────────────┼────────────────────┘
                            ▼
                       DESIRED STATE
                            │
                            ▼
                       IDEMPOTENCY
                            │
                            ▼
                         TESTING
                            │
                            ▼
                      PRODUCTION 🚀
```

---

## 🎉 Ansible Core Preparation Complete

You've now covered the important areas we selected, including:

```text
✅ Fundamentals
✅ Inventory
✅ Control/Managed nodes
✅ SSH
✅ Modules
✅ Playbooks
✅ Variables
✅ Conditions & loops
✅ Roles
✅ Templates
✅ Vault
✅ Blocks/error handling
✅ Execution control
✅ serial / forks / strategy / throttle
✅ Testing / Molecule
✅ Connection methods
✅ Dynamic inventory
✅ Production troubleshooting
✅ Production scenarios
✅ A3 interview scenarios
```

And we deliberately **skipped the chapters you said you already knew**, rather than wasting your preparation time.

### 📌 One thing I'd recommend before moving to the next technology

Don't immediately start another theory chapter.

We can do a **mock Ansible A3 interview** where I ask you one question at a time, you answer in your own words, and I evaluate your answer as:

**🟢 Strong A3 | 🟡 Needs improvement | 🔴 Incorrect**

That will reveal whether you actually know the concepts rather than just recognizing them.
