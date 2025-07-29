# **Kubernetes Taints & Tolerations - Quick Guide **

## **1. What are Taints & Tolerations?**
- **Taints**: Node property that *repels* Pods unless they explicitly tolerate it  
- **Tolerations**: Pod property that allows it to *ignore* node taints  

👉 **Purpose**: Reserve nodes for specific workloads (e.g., GPU nodes, dedicated environments).

---

## **2. Key Concepts**
| Term | Description |
|------|-------------|
| **Taint Effect** | `NoSchedule` (block new Pods), `PreferNoSchedule` (soft block), `NoExecute` (evict existing Pods) |
| **Toleration** | Matches a taint's `key`, `value`, and `effect` to allow scheduling |

---

## **3. Examples**

### **A. Taint a Node**
```sh
kubectl taint nodes <node-name> key=value:effect
```
**Example**: Reserve a node for GPU workloads  
```sh
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule
```

### **B. Add Toleration to a Pod**
```yaml
tolerations:
- key: "gpu"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"
```
**Full Pod Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
  - name: tensorflow
    image: tensorflow/gpu
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

---

## **4. Common Use Cases**
| Scenario | Taint | Toleration |
|----------|-------|------------|
| **Dedicated Nodes** | `env=prod:NoSchedule` | `env=prod` |
| **GPU/Specialized HW** | `accelerator=gpu:NoSchedule` | `accelerator=gpu` |
| **Maintenance Mode** | `maintenance=true:NoExecute` | `maintenance=true` |

---

## **5. Commands Cheatsheet**
| Command | Purpose |
|---------|---------|
| `kubectl taint node <node> key=value:effect` | Add taint |
| `kubectl taint node <node> key:effect-` | Remove taint |
| `kubectl describe node <node> \| grep Taint` | Check taints |

---

## **6. Important Notes**
- **Operator**: Use `Exists` (ignore value) or `Equal` (strict match).
- **Effect**: `NoExecute` evicts untolerated Pods (use cautiously!).
- **Combining with Affinity**: Tolerations allow scheduling, but affinity rules still apply.

---

## **7. Example: Production-Only Node**
```sh
# Step 1: Taint the node
kubectl taint nodes prod-node-1 env=prod:NoSchedule

# Step 2: Deploy Pod with toleration
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: prod-app
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"
EOF
```

---

> **📌 Pro Tip**: Use `kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints` to list all taints.  
> **⚠️ Warning**: `NoExecute` can disrupt running Pods. Prefer `NoSchedule` for new deployments.