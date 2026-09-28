### **GitOps & Argo CD Crash Course: Fundamentals**

GitOps is a methodology for managing infrastructure and application deployments by using **Git as the single source of truth**. It bridges the gap in Continuous Delivery (CD) by applying the same version-controlled, auditable practices used in software development to infrastructure management.

---

#### **1. The "Why" Behind GitOps**
Before GitOps, managing Kubernetes infrastructure (like node configurations or resource updates) lacked tracking. If a change was made manually (e.g., via shell scripts or `kubectl`), there was no easy way to:
*   **Audit** who made the change.
*   **Version** the infrastructure state.
*   **Roll back** or understand the history of modifications.

GitOps solves this by requiring all infrastructure and application states to be defined declaratively in Git repositories. By using a Pull Request workflow, changes are reviewed and approved before being applied to the cluster.

---

#### **2. Core Principles of GitOps**
To be considered a true GitOps-managed system, it must adhere to these foundational principles:

*   **Declarative System:** The desired state of the entire system must be expressed declaratively (e.g., YAML manifests for Kubernetes pods, namespaces, or node configurations).
*   **Versioned and Immutable:** Changes are tracked in version control. While Git is the industry standard, other storage mechanisms like S3 buckets can work if they provide versioning for declarative manifests.
*   **Automated Pulling:** The system must automatically pull the state from the repository. This is facilitated by a GitOps controller that resides within the cluster.
*   **Continuous Reconciliation:** The controller continuously compares the live state of the cluster with the desired state in Git. If a deviation occurs (e.g., manual intervention or an unauthorized change), the controller automatically overrides the cluster state to match the Git source of truth.

---

#### **3. Security and Advantages**
*   **Enhanced Security:** By removing manual access to the cluster and enforcing Git-only changes, the attack surface is reduced. Any unauthorized changes are automatically corrected by the controller.
*   **Auto-Healing:** The system detects drift between the Git repository and the actual cluster state, ensuring the cluster self-corrects back to the approved version.
*   **Simplified Auditing:** Every change is recorded as a commit in Git, creating a clear, immutable history of who changed what and when.

---

#### **4. Is GitOps only for Kubernetes?**
*   **By Principle:** No. GitOps is a broader methodology for infrastructure management.
*   **In Practice:** Yes, currently. The most popular tools (such as *Argo CD* and *Flux CD*) are built specifically to target Kubernetes clusters. While future tools may extend this to other platforms (like AWS infrastructure or Docker Swarm), today's GitOps ecosystem is tightly coupled with Kubernetes.

---

#### **5. When to use what?**

Rather than choosing one over the other, the standard industry pattern combines both:

*    **GitHub Actions (CI):** Builds application binaries, runs unit tests, creates Docker images, pushes images to a registry, and commits the updated image tag into a Git manifest repository.

*    **Argo CD (CD/GitOps):** Detects the new image tag commit in the Git repository, pulls the updated manifests, and safely applies them inside the Kubernetes cluster
