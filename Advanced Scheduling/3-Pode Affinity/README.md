# **Pod Affinity & Anti-Affinity - Quick Guide**

### **1. What is Pod Affinity?**
Controls **co-location** of Pods:
- `podAffinity` = Schedule Pods **together** (same node/zone)
- `podAntiAffinity` = Schedule Pods **apart** (avoid single node)

---

### **2. Key Operators**
| Operator | Purpose | Example |
|----------|---------|---------|
| `requiredDuringScheduling...` | Hard rule (must satisfy) | High-availability |
| `preferredDuringScheduling...` | Soft rule (best-effort) | Optimizations |
| `topologyKey` | Grouping scope (e.g., `hostname`, `zone`) | Cross-zone deployments |

---

### **3. Quick Examples**

#### **Pod Affinity (Run together)**
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: cache  # Target Pods with this label
      topologyKey: kubernetes.io/hostname  # Same node
```

#### **Pod Anti-Affinity (Spread apart)**
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: web  # Avoid sharing nodes with "web" Pods
      topologyKey: topology.kubernetes.io/zone  # Different zones
```

---

### **4. When to Use?**
- **Use Affinity**:
  - Database + cache on same node
  - Microservices needing low latency
- **Use Anti-Affinity**:
  - HA (avoid single-node failures)
  - Stateful apps (e.g., databases)

---

### **5. Verify**
```sh
kubectl get pods -o wide  # Check node placement
kubectl describe pod <name>  # View affinity rules
```

---

> **ℹ️ Pro Tip**: Combine with `nodeAffinity` for advanced scheduling!  
> **📌 Note**: `topologyKey` must match node labels. Check with `kubectl get nodes --show-labels`.

---

## **1. Pod Affinity (`podAffinity`)**
Ensures Pods are scheduled **on the same node** (or zone) as other Pods matching a label.

### **Example: Deploy Pods Together (Co-location)**
```yaml
apiVersion : apps/v1
kind : Deployment
metadata :
  name : dp1
spec :
  selector :
    matchLabels :
      app : frontend
  replicas : 9
  template :
    metadata :
      labels :
        app : frontend
    spec :
      containers :
      - name : con1
        image : nginx
      affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: servertype
                  operator: In
                  values:
                  - prod
                  - stage

```
**Explanation:**  
- This Pod will **only** schedule on nodes where a node label is `servertype=prod, stage` .


### **Example: Deploy Pods Together (with label matching)**
```yaml
apiVersion : apps/v1
kind : Deployment
metadata :
  name : dp2
spec :
  replicas : 8
  selector :
    matchLabels :
      app : redis
  template :
    metadata :
      labels :
        app : redis
    spec :
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app   #key-pair ' other deployment label'
                operator: In
                values:
                - frontend
            topologyKey: kubernetes.io/hostname
      containers :
      - name : rediscon
        image : nginx

```

**Explanation:**  
- This Pod will **only** schedule on nodes where a Pod with `app=frontend` is already running.
- `topologyKey: kubernetes.io/hostname` ensures they run on the **same node**.
- Example 1 is fine for lab practice
- **************************************************************************************************************************************

---
## **2. Pod Anti-Affinity (`podAntiAffinity`)**
Ensures Pods are **not scheduled together** (avoid single point of failure).

### **Example: Spread Pods Across Nodes**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cluster
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: "app"
                operator: In
                values: ["redis"]
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis
        image: redis:alpine
```
**Explanation:**  
- Ensures **no two `redis` Pods** run on the same node (`hostname`).
- Useful for **high-availability** deployments.

---

## **3. Soft (Preferred) Anti-Affinity**
Preference (not hard requirement) to avoid scheduling together.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: "app"
                  operator: In
                  values: ["web"]
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: nginx
```
**Explanation:**  
- Kubernetes will **try** to spread `web` Pods across nodes, but may violate if resources are scarce.
- `weight: 100` defines priority (higher = stronger preference).

---

## **4. Multi-Zone Affinity (Using `topologyKey`)**
Schedule Pods in the same zone (but not necessarily same node).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: payment
  template:
    metadata:
      labels:
        app: payment
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: "app"
                operator: In
                values: ["database"]
            topologyKey: "topology.kubernetes.io/zone"
      containers:
      - name: payment
        image: payment-service:latest
```
**Explanation:**  
- Pods will schedule in the **same zone** (e.g., `us-east-1a`) as `app=database` Pods.
- Uses `topology.kubernetes.io/zone` instead of `hostname`.

---

## **5. Combining Affinity & Anti-Affinity**
Advanced control for complex scenarios.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: analytics
spec:
  replicas: 3
  selector:
    matchLabels:
      app: analytics
  template:
    metadata:
      labels:
        app: analytics
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: "service"
                operator: In
                values: ["monitoring"]
            topologyKey: "kubernetes.io/hostname"
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: "app"
                operator: In
                values: ["analytics"]
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: analytics
        image: analytics:latest
```
**Explanation:**  
- Pods **must** run on the same node as `service=monitoring`.
- Pods **must not** share a node with other `analytics` Pods.

---

## **Key Concepts**
| Term | Description |
|------|------------|
| `podAffinity` | Schedule Pods **near** others (same node/zone). |
| `podAntiAffinity` | Schedule Pods **away** from others (avoid colocation). |
| `topologyKey` | Defines the "domain" for grouping (e.g., `hostname`, `zone`). |
| `requiredDuringScheduling...` | Hard requirement (Pod won't schedule if rule fails). |
| `preferredDuringScheduling...` | Soft preference (best-effort). |

---

## **When to Use?**
| Scenario | Solution |
|----------|----------|
| Database + Cache on same node | `podAffinity` |
| HA (spread replicas across nodes) | `podAntiAffinity` |
| Microservices in same zone | `podAffinity` + `topologyKey: zone` |
| Avoid overloading a single node | `podAntiAffinity` with `hostname` |

---

## **Verification**
1. Deploy Pods:
   ```sh
   kubectl apply -f pod-affinity.yaml
   ```
2. Check scheduling:
   ```sh
   kubectl get pods -o wide
   ```
3. Describe a Pod to see affinity rules:
   ```sh
   kubectl describe pod <pod-name>
   ```

---

## **Troubleshooting**
- **Pending Pods?** Check events:
  ```sh
  kubectl describe pod <pod-name>
  ```
- **No nodes match?** Ensure:
  - Target Pods have the correct labels.
  - `topologyKey` is valid (`kubectl get nodes --show-labels`).

---

## **Final Notes**
- Use `affinity` for **fine-grained scheduling**.
- Prefer `podAntiAffinity` for **stateful workloads** (e.g., databases).
- Combine with `nodeAffinity` for **hybrid rules**.

Let me know if you need refinements! 🚀