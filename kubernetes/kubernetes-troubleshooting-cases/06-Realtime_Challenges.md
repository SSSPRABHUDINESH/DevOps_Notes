## Critical real-world Kubernetes challenges

**1. The Resource Sharing Challenge**
* In a production environment, multiple microservices and project teams often share the same Kubernetes cluster rather than having separate clusters.
* The core challenge lies in how to fairly and safely allocate cluster resources (CPU and RAM) among these competing teams.

## **2. The Problem: Memory Leaks and Blast Radius**
* Without constraints, if an application in one namespace suffers from a memory leak, it can consume a disproportionate amount of the cluster's resources.
* This leads to other services in the cluster being starved of resources, eventually causing them to crash with **OOMKilled** (Out Of Memory) errors.
* The "blast radius" is initially the entire cluster, meaning one team's bad code can take down the entire system.
---
## **3. The Solution: Implementing Resource Quotas**
* DevOps engineers must implement a **ResourceQuota** at the **Namespace level**. 
* A ResourceQuota acts as a hard limit for total CPU and RAM that all pods within that specific namespace can consume collectively.
* These limits should be derived from **performance benchmarking** conducted by development teams to ensure each namespace receives enough resources for its specific load.

---

### Resource Quotas (Namespace Level)

A `ResourceQuota` caps the total compute resources that all pods inside a target namespace can collectively consume.

**`resource-quota.yaml`**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
    pods: "10"

```

* **Create Namespace & Apply Quota:**
```bash
kubectl create namespace dev-team
kubectl apply -f resource-quota.yaml

```


* **Verify Resource Quota Usage:**
```bash
kubectl get resourcequota dev-team-quota -n dev-team
# Or get detailed usage breakdown:
kubectl describe resourcequota dev-team-quota -n dev-team

```



---

### Imperative Commands (Quick Edits without Files)

If you need to edit existing configurations directly from your terminal:

* **Directly edit an active ResourceQuota:**
```bash
kubectl edit resourcequota dev-team-quota -n dev-team

```


* **Set/Update container resources via command line:**
```bash
kubectl set resources deployment sample-app -n dev-team --limits=cpu=500m,memory=512Mi --requests=cpu=250m,memory=128Mi
```
---
## **4. Narrowing the Blast Radius: Resource Limits**
* Even with a namespace quota, a single misbehaving pod could still potentially impact other pods within the same namespace.
* To fix this, you must set **Resource Limits** on individual **pods**. 
* By defining specific `requests` (minimum) and `limits` (maximum) in your deployment YAMLs, you restrict the blast radius to that **specific pod**. 
* If a pod leaks memory beyond its assigned limit, only that pod will crash (OOMKilled), keeping the rest of the namespace and cluster stable.
---

### 1. Pod Resource Limits (Pod Level)

You set requests and limits inside the `containers` section of your Deployment YAML file.

**`deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  namespace: dev-team
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: app-container
        image: nginx:latest
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

```

* **Apply Command:**
```bash
kubectl apply -f deployment.yaml

```

* **Verify Pod Limits:**
```bash
kubectl describe pod <pod-name> -n dev-team

```
### Imperative Commands (Quick Edits without Files)

If you need to edit existing configurations directly from your terminal:

* **Directly edit a Deployment's limits:**
```bash
kubectl edit deployment sample-app -n dev-team

```
