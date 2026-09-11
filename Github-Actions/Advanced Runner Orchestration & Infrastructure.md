# Module 1: Advanced Runner Orchestration & Infrastructure

---

## 1. GitHub-Hosted vs. Self-Hosted Runners

### Deep Comparison

| Capability / Metric | GitHub-Hosted Runners | Self-Hosted Runners |
| --- | --- | --- |
| **Infrastructure Management** | Fully managed by GitHub (SaaS) | Customer managed (Cloud, On-Prem, K8s) |
| **Network & Firewall Access** | External IPs; requires NAT/VPN for private resources | Native access to private VPCs, databases, and internal APIs |
| **Security Lifecycle** | Clean VM isolated per job (Zero persistence) | Can be persistent (risk of pollution) or ephemeral |
| **Hardware Flexibility** | Standard CPU/RAM tiers (Standard & Larger Runners) | Custom hardware support (GPUs, ARM, High RAM) |
| **Cost Model** | Pay-per-minute execution fee | Fixed infrastructure cost (Compute + Memory + Storage) |
| **Egress Costs** | Free out-of-band artifact transfers within GitHub | Standard Cloud Provider egress rates apply |

### Key Enterprise Trade-offs

* **GitHub-Hosted:** Eliminates operational overhead and patched maintenance. However, it requires exposing internal endpoints or maintaining dynamic IP allowlists (or using GitHub Private Networking).
* **Self-Hosted:** Provides speed, internal network connectivity, and specialized hardware. However, it introduces operational overhead, security patching duties, and potential runner contamination if persistent runners are improperly configured.

### Real-World Production Use Cases

* **GitHub-Hosted:** Microservice unit testing, linting, public package publishing, and standard SaaS deployments.
* **Self-Hosted:** Database schema migrations inside private subnets, ML/AI model training requiring CUDA GPUs, legacy builds needing specific OS kernels, and air-gapped environment builds.

---

## 1.1 Self Hosted Runners:

To understand how a self-hosted runner works and where the `--ephemeral` flag fits, it helps to walk through a complete, step-by-step physical example from scratch.

---

### The Big Picture: How a Self-Hosted Runner Connects

A self-hosted runner is just an ordinary virtual machine (VM) or server (like an AWS EC2 instance, GCP VM, or local Ubuntu machine) running a small program written by GitHub called the **GitHub Actions Runner Application**.

1. **No Inbound Connections:** GitHub **never** reaches directly into your server or VPC. Your server does not need a public IP address or open inbound firewall ports.
2. **Outbound Long-Polling:** Once started, the runner application on your VM opens a secure outbound HTTPS connection to GitHub (`github.com`) and continuously asks: *"Is there a job assigned to my queue?"*
3. **Execution:** When you trigger a workflow, GitHub places the job in a queue. The runner pulls the job down, executes the steps locally on your VM, and streams the logs back to GitHub over that same outbound connection.

---

### Complete End-to-End Scenario: Provisioning an Ephemeral Runner VM

Imagine you create a fresh **Ubuntu 22.04 Virtual Machine** in AWS or GCP to run a single, isolated build job. Here is the exact lifecycle:

#### Step 1: Get a Registration Token from GitHub

Before your VM can connect, GitHub needs to authorize it.

  * You go to your **GitHub Repository/Org Settings** $\rightarrow$ **Actions** $\rightarrow$ **Runners** $\rightarrow$ **New self-hosted runner**.
  * GitHub gives you a temporary **Registration Token** (e.g., `AXXYYZZ12345`).

#### Step 2: SSH into Your Fresh VM and Download the Runner Application

On your clean Virtual Machine, you download and extract the runner package:

```bash
# Create a folder for the runner
mkdir actions-runner && cd actions-runner

# Download the official GitHub Actions runner package
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# Extract the installer
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

```

#### Step 3: Configure the Runner (THIS IS WHERE `--ephemeral` LIVES!)

This is the key step. You run the `./config.sh` script to register the VM with your repository.

By adding the **`--ephemeral`** flag here, you tell the application: **"Accept exactly ONE job from GitHub, run it, unregister yourself, and exit."**

```bash
./config.sh \
  --url https://github.com/Your-Org/Your-Repo \
  --token AXXYYZZ12345 \
  --name "my-temp-aws-runner-01" \
  --labels "ubuntu-latest,custom-aws" \
  --ephemeral \
  --unattended

```

* **What happens inside the VM during `./config.sh`?**
* It registers `my-temp-aws-runner-01` in your GitHub repository's **Runners** tab.
* It creates local configuration files inside the `actions-runner` folder containing credentials and authentication keys.



#### Step 4: Start the Runner Process

Now, start the runner listening loop on the VM:

```bash
./run.sh

```

- At this moment, the output in your terminal displays:
`Listening for Jobs...`

- In the **GitHub Web UI**, your runner `my-temp-aws-runner-01` changes status from **Offline** to **Idle** (Green dot).

`./run.sh` is a wrapper shell script provided by GitHub in the runner package. It sets up the environment and launches the core runner executable (written in C# / .NET Core).

---

### Key Responsibilities of `./run.sh`

1. **Environment Setup:** Exports environment variables saved during configuration (like path overrides and custom variables from `.env` and `.path`).
2. **Runtime Verification:** Checks for required dependencies and validates the .NET runtime binaries located in `bin/`.
3. **Execution:** Hands off control to the main compiled runner binary (`bin/Runner.Listener`).

---

### Simplified Code Structure

While the exact file can vary slightly by OS, `./run.sh` looks approximately like this:

```bash
#!/bin/bash

# Navigate to the runner directory
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
cd "$DIR"

# Source environment files if they exist
if [ -f .env ]; then
    source .env
fi

# Execute the core C# binary
exec ./bin/Runner.Listener "$@"

```

---

### What `bin/Runner.Listener` Executes

When `./run.sh` launches `Runner.Listener`, it starts the main event loop:

```text
1. HTTPS Long-Poll ──► Connects to GitHub API (https://pipelines.actions.githubusercontent.com)
                            │
                            ▼
2. Job Request     ──► Receives job payload & JWT access token
                            │
                            ▼
3. Worker Process  ──► Spawns Runner.Worker to execute step-by-step shell commands
                            │
                            ▼
4. Cleanup         ──► Streams logs back, unregisters if --ephemeral was set, and exits

```

---

### What Happens When a Workflow Triggers?

1. **Trigger:** A developer opens a Pull Request.
2. **Matching:** The workflow contains `runs-on: self-hosted`.
3. **Job Pickup:** GitHub assigns the job to `my-temp-aws-runner-01`.
4. **Execution:** The VM executes the commands (`actions/checkout`, `npm test`, `docker build`, etc.).
5. **Auto-Deregistration & Destruction:**
    * Because you passed `--ephemeral` in Step 3, as soon as the last step completes, the runner process automatically unregisters itself from GitHub and terminates the `./run.sh` process.
    * On GitHub's UI, `my-temp-aws-runner-01` disappears from the runner list.
    * If using an auto-scaling group (like AWS Auto Scaling or Kubernetes ARC), an automated script detects that `./run.sh` stopped, **terminates/deletes the entire VM**, and spins up a brand-new VM to wait for the next job.


## Runner application is in listening mode

When we say the runner application is in **"listening mode"** (or listening for jobs), it means the application is running an **outbound event loop**.

---

### Web Server vs. Long-Polling Listener

To understand the difference, look at how the connection opens:

* **If it were a Web Server (Inbound):**
GitHub would need to send a request *into* your VM (`GitHub ──► Your VM`). This would require your VM to have a public IP, open firewall ports (like Port 80/443), and an inbound web server like Nginx or Express.
* **How it Actually Works (Outbound Long-Polling):**
Your VM initiates a continuous HTTPS request *out* to GitHub (`Your VM ──► GitHub`).

---

### What "Listening" Actually Means Under the Hood

When you run `./run.sh`, the process launches a program called `Runner.Listener`. "Listening" is a technique called **HTTP Long-Polling**:

```text
       Your Self-Hosted VM                                    GitHub Service
┌────────────────────────────────┐                        ┌──────────────────┐
│  Runner.Listener Executable    │                        │  GitHub Actions  │
│                                │                        │  Job Queue       │
│  1. Sends HTTPS GET request ──────────────────────────► │                  │
│     "Do you have a job for me?"│                        │                  │
│                                │                        │                  │
│  2. GitHub holds request open  │                        │ (No job available│
│     up to 50 seconds...        │                        │  yet... waiting) │
│                                │                        │                  │
│                                │   3. Job arrives!      │                  │
│  4. Receives Job Details ◄───────────────────────────────│                  │
│                                │                        │                  │
│  5. Spawns Worker Process      │                        │                  │
│     and executes workflow      │                        │                  │
└────────────────────────────────┘                        └──────────────────┘

```

1. **The Request:** The VM asks GitHub: *"Is there any job assigned to me?"*
2. **The Wait (Long-Poll):** GitHub doesn't answer immediately if the queue is empty. Instead, GitHub **holds the HTTP connection open** for up to 50 seconds waiting for a workflow to trigger.
3. **The Response:**
    * If a developer triggers a pipeline, GitHub immediately sends the job payload down that open connection.
    * If 50 seconds pass with no jobs, GitHub responds with an empty message. The VM instantly opens a new request to start listening again.





## 2. Ephemeral Runners

An **Ephemeral Runner** is a single-use worker instance. It registers with GitHub, accepts **exactly one job**, executes it, and is immediately destroyed upon job completion.

### The Security & Reliability Hazard of Persistent Runners

```
[ Persistent Runner Instance ]
Job 1: Installs custom tool / Leaves credentials in /tmp / Modifies system config
       │
       ▼
Job 2: Inherits contaminated state from Job 1 ──► Risk of credential theft or false passes

```

### The Ephemeral Solution Architecture

```
[ Trigger Event ] ──► [ Provision Clean Pod/VM ] ──► [ Process Single Job ] ──► [ Destroy Worker ]

```

### Key Advantages

1. **Zero State Leakage:** Prevents build artifacts, environment variables, or temporary tokens from persisting into subsequent builds.
2. **Deterministic Builds:** Guarantees that every build runs against a clean environment, eliminating "works on my machine" inconsistencies.
3. **Automated Credential Flush:** Short-lived tokens stored in memory or `/tmp` vanish when the instance terminates.

### Implementation Pattern (`--ephemeral` Flag)

When registering a runner manually via the CLI, the `--ephemeral` flag enforces auto-deregistration after a single job execution:

```bash
./config.sh --url https://github.com/Your-Org/Your-Repo \
  --token <REGISTRATION_TOKEN> \
  --name "ephemeral-runner-01" \
  --ephemeral \
  --unattended

```

---

## 3. Actions Runner Controller (ARC)

**Actions Runner Controller (ARC)** is a Kubernetes operator that automates the provisioning, scaling, and lifecycle management of self-hosted runners using native Scale Sets.

### Architecture Topology

```
                  ┌──────────────────────────────────────────────┐
                  │            GitHub Actions Service            │
                  └──────────────────────┬───────────────────────┘
                                         │ Webhook / Long-Poll
                                         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Kubernetes Cluster                                                              │
│                                                                                 │
│   ┌────────────────────────┐  Watches  ┌────────────────────────────────────┐   │
│   │ gha-runner-scale-set   │──────────►│      Auto-scaling Runner Pods      │   │
│   │      Controller        │           │ (Runner Container + Auto-Scaler)   │   │
│   └────────────────────────┘           └────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘

```

### Primary Kubernetes Components

1. **`gha-runner-scale-set-controller`**: The core operator managing lifecycle events and custom resources.
2. **`gha-runner-scale-set`**: The Helm chart deployment that connects to GitHub's auto-scaling APIs and provisions ephemeral pods dynamically based on queue pressure.

### Helm Installation Reference

#### Step 1: Install the Controller Manager

```bash
helm install arc \
  --namespace arc-systems \
  --create-namespace \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller

```

#### Step 2: Install the Runner Scale Set

```bash
helm install arc-runner-set \
  --namespace arc-runners \
  --create-namespace \
  --set githubConfigUrl="https://github.com/Your-Org" \
  --set githubConfigSecret.github_token="<GITHUB_PAT_OR_APP_TOKEN>" \
  --set minRunners=1 \
  --set maxRunners=20 \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set

```

### Production Workflow Integration

Targeting the deployed ARC Scale Set in your workflow:

```yaml
name: Production Enterprise Build
on:
  push:
    branches: [ main ]

jobs:
  build:
    # Match the runner scale set name defined during Helm deployment
    runs-on: arc-runner-set
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Execute Build Script
        run: |
          echo "Running build on dynamically autoscaled ARC Pod"
          hostname

```

---

## 4. Runner Groups & Access Isolation

Runner Groups allow organization administrators to segment access to self-hosted runners, preventing unauthorized repositories from executing workloads on sensitive infrastructure.

### Logical Organization Layout

```
                        ┌──────────────────────────────┐
                        │     Organization Level       │
                        └──────────────┬───────────────┘
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼                                                     ▼
┌───────────────────────┐                             ┌───────────────────────┐
│  Prod Runner Group    │                             │ Non-Prod Runner Group │
│ (Restricted Access)   │                             │   (Open Access)       │
├───────────────────────┤                             ├───────────────────────┤
│ Target: prod-repo     │                             │ Target: * (All Repos) │
│ Access: Protected Ops │                             │ Access: Dev / Feature │
└───────────────────────┘                             └───────────────────────┘

```

### Enterprise Isolation Strategies

1. **Production Boundary:** Assign high-privilege runners (e.g., runners with access to AWS production VPCs) exclusively to protected repositories and deployment pipelines.
2. **Prevent Token Abuse:** Restrict public repositories or untrusted internal forks from running workflows on self-hosted runners to guard against malicious Pull Requests designed to steal infrastructure credentials.
3. **Environment Segregation:** Map dedicated Runner Groups directly to deployment environments (`dev`, `staging`, `prod`).

---

## 5. Custom Docker Container Jobs

Instead of installing build dependencies directly on the host machine, GitHub Actions can run workflow steps inside an isolated Docker container on top of the runner host.

### Execution Flow

```
[ Host Runner (VM/Pod) ]
   └── Spawns ──► [ Docker Container (e.g., golang:1.22) ]
                     └── Executes ──► [ Workflow Steps ]

```

### Advanced Workflow Configuration

```yaml
name: Isolated Container Build
on:
  push:
    branches: [ main ]

jobs:
  golang-build:
    runs-on: self-hosted
    # Specifies the target container environment for job execution
    container:
      image: golang:1.22-alpine
      env:
        GOPROXY: https://proxy.golang.org,direct
      options: --user 1000
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Inspect Execution Context
        run: |
          cat /etc/os-release
          go version
          whoami

```

### Why Use Container Jobs?

* **Tool Isolation:** Eliminates the need to pre-install dependencies (Node, Go, Python, Terraform) on base host images.
* **Version Parity:** Guarantees exact tool versions by using explicit Docker image tags.
* **Clean Cleanup:** Docker automatically destroys the execution container once the job finishes, leaving the underlying runner host untouched.

---
