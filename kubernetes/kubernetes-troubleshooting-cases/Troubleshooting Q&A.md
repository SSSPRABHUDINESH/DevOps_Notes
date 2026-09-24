
---

## 1. Three Ready-to-Use Production Problem-Solving Stories

Prepare these three stories using the **Problem $\rightarrow$ Investigation $\rightarrow$ Root Cause $\rightarrow$ Solution $\rightarrow$ Preventive Action** framework:

### Story 1: Kubernetes Workload / Deployment Failure (Storage & Networking)

* **Problem:** A production deployment rollout froze with pods stuck in `CrashLoopBackOff` and `ContainerCreating` states across multiple worker nodes.
* **Investigation:** Used `kubectl describe pod` to inspect event logs, checked PersistentVolumeClaim (PVC) binding states, and reviewed GCP storage API quotas.
* **Root Cause:** Multi-AZ pod affinity mismatch. A new pod was scheduled in `Zone-B`, but its underlying GCP PersistentDisk was provisioned in `Zone-A`, causing attachment timeouts.
* **Solution & Validation:** Updated the pod deployment manifest with proper `nodeAffinity` and `topologySpreadConstraints` matching the regional storage volumes.
* **Preventive Action:** Updated Terraform templates to dynamically enforce topology constraints on stateful workloads across all GCP modules.

---

## 2. How to Answer an Unfamiliar Problem Statement Live

If the interviewer gives you a scenario you haven't seen before (e.g., *"Our GKE application API latency suddenly jumped by 400%, what do you do?"*), **do not guess solutions immediately**.

Walk them through your analytical framework out loud:

```
Clarify Scope → Isolate Layer → Collect Evidence → Mitigate/Rollback → Root Cause → Fix

```

### Step-by-Step Execution:

1. **Clarify Scope:** *"Before diving in, I’d clarify: Is this affecting all users or a specific subset? Is it intermittent or constant? Were there any recent infrastructure or application deployments in the last few hours?"*
2. **Isolate the Layer:** *"I divide the problem into four potential layers: Ingress/Network, Kubernetes Pod/Node compute, Database/Backing services, or Dependencies."*
3. **Collect Evidence:** *"I check high-level golden signals first—latency, error rates, saturation, and traffic spikes in Prometheus/Grafana or Cloud Monitoring. Then I inspect Kubernetes events (`kubectl get events`), pod CPU/memory throttling, and database connection pool usage."*
4. **Mitigate / Restore First:** *"If production is degraded, my priority is restoring service before deep debugging. If a recent deployment caused it, I trigger a rollback to the last known healthy revision. If it's resource exhaustion, I temporarily scale the deployment replicas or nodepool."*
5. **Permanent Fix & RCA:** *"Once stable, I recreate the issue in a staging environment, identify the exact bug or bottleneck, apply the permanent patch, and publish a Post-Incident Review (PIR)."*

---
