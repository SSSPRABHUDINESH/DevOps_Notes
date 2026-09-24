
---

### 📝 Interview Notes: Kubernetes HPA for Unpredictable Traffic

**Q: How would you configure Horizontal Pod Autoscaler (HPA) to handle unpredictable or sudden traffic spikes?**

**A:**
To handle sudden, unpredictable traffic spikes, relying on default HPA metrics (like CPU/Memory) and standard scaling behavior is insufficient because default HPA reacts too slowly and can cause dropped requests.

I would configure HPA using a **5-pillar strategy**:

1. **Configure Asymmetric Scaling (`behavior` block):**
* **Scale Up Aggressively (Instant):** Set `stabilizationWindowSeconds: 0` so HPA reacts immediately without delay. Combine two policies using `selectPolicy: Max`—a percentage policy (e.g., `100%`) for exponential growth at high scale, and a fixed pod count policy (e.g., `+10 pods`) to ensure fast scaling at low scale.
* **Scale Down Cautiously (Slow):** Set a stabilization window of 5 minutes (`300s`) to avoid thrashing during temporary lulls, and cap scale-down rate (e.g., `10%` per minute) to handle secondary traffic waves safely.


2. **Autoscale on Leading Indicators (Custom Metrics):**
* Instead of lagging indicators like CPU or Memory, scale on HTTP Requests Per Second (RPS) or message queue depth (using Prometheus Adapter or KEDA).


3. **Maintain an Over-Provisioned Baseline:**
* Set a higher `minReplicas` floor in production to absorb the initial impact.
* Use low-priority "Pause Pods" (preemption) so new HPA pods get instant node capacity without waiting for cluster autoscaling.


4. **Optimize Container Startup Time:**
* Keep container images lightweight and tune application startup and readiness probes so new pods start serving traffic in seconds.


5. **Pair with Fast Infrastructure Autoscaling:**
* Ensure underlying cluster node autoscaling (e.g., Karpenter or Cluster Autoscaler) is active to back up rapid pod expansion.



---

#### 💻 Reference YAML Manifest

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 10          # Baseline floor to absorb initial hits
  maxReplicas: 100
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0 # React instantly on first evaluation
      policies:
      - type: Percent
        value: 100                  # Strategy A: Double pod capacity
        periodSeconds: 15
      - type: Pods
        value: 10                   # Strategy B: Add 10 pods
        periodSeconds: 15
      selectPolicy: Max             # Pick whichever adds MORE pods
    scaleDown:
      stabilizationWindowSeconds: 300 # Wait 5 minutes before scaling down
      policies:
      - type: Percent
        value: 10                   # Gradually remove max 10% per minute
        periodSeconds: 60

```

---

#### 📌 Key takeaway for execution:

HPA is **fully declarative**. Applying this YAML via `kubectl apply -f hpa.yaml` registers the object with the `kube-controller-manager`, which continuously monitors metrics and executes scaling automatically 24/7.
