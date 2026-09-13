## Critical real-world Kubernetes challenges

**1. The Resource Sharing Challenge**
* In a production environment, multiple microservices and project teams often share the same Kubernetes cluster rather than having separate clusters.
* The core challenge lies in how to fairly and safely allocate cluster resources (CPU and RAM) among these competing teams.

**2. The Problem: Memory Leaks and Blast Radius**
* Without constraints, if an application in one namespace suffers from a memory leak, it can consume a disproportionate amount of the cluster's resources.
* This leads to other services in the cluster being starved of resources, eventually causing them to crash with **OOMKilled** (Out Of Memory) errors.
* The "blast radius" is initially the entire cluster, meaning one team's bad code can take down the entire system.

**3. The Solution: Implementing Resource Quotas**
* DevOps engineers must implement a **ResourceQuota** at the **Namespace level**. 
* A ResourceQuota acts as a hard limit for total CPU and RAM that all pods within that specific namespace can consume collectively.
* These limits should be derived from **performance benchmarking** conducted by development teams to ensure each namespace receives enough resources for its specific load.

**4. Narrowing the Blast Radius: Resource Limits**
* Even with a namespace quota, a single misbehaving pod could still potentially impact other pods within the same namespace.
* To fix this, you must set **Resource Limits** on individual **pods**. 
* By defining specific `requests` (minimum) and `limits` (maximum) in your deployment YAMLs, you restrict the blast radius to that **specific pod**. 
* If a pod leaks memory beyond its assigned limit, only that pod will crash (OOMKilled), keeping the rest of the namespace and cluster stable.
