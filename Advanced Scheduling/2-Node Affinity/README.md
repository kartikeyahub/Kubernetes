# Node Affinity in Kubernetes

## Overview

Node Affinity is a Kubernetes feature that allows you to constrain which nodes your Pod can be scheduled on based on node labels. It provides more expressive and complex scheduling rules compared to simple NodeSelectors.

### Key Features:
- More flexible and powerful than NodeSelector
- Supports both hard requirements ("must match") and soft preferences ("should match")
- Allows for complex logical operations (In, NotIn, Exists, DoesNotExist, Gt, Lt)
- Part of the Kubernetes scheduling framework

## Types of Node Affinity

1. **requiredDuringSchedulingIgnoredDuringExecution** (Hard requirement)
   - Pod must be scheduled on a node that meets the rules
   - If no matching node exists, Pod remains unscheduled

2. **preferredDuringSchedulingIgnoredDuringExecution** (Soft preference)
   - Kubernetes tries to schedule Pod on a node that meets the rules
   - If no matching node exists, Pod is scheduled on another available node
  
## Prerequisites

1. **Label your nodes**:
   ```bash
   # Server label
   kubectl label nodes Kartikeyasoft-worker2 servertype=stage
   kubectl label nodes Kartikeyasoft-worker3 servertype=prod

   
   # Zone labels
   kubectl label nodes <node1> topology.kubernetes.io/zone=us-west-2a
   kubectl label nodes <node2> topology.kubernetes.io/zone=us-west-2b
   

   # Instance type labels
   kubectl label nodes <node3> instance-type=gpu.large
   kubectl label nodes <node4> instance-type=gpu.xlarge
   ```

2. **Verify node labels**:
   ```bash
   kubectl get nodes --show-labels
   ```


## Example: Pod with Node Affinity - Example1

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: cart
  template:
    metadata:
      labels:
        app: cart
    spec:
      containers:
      - name: my-con
        image: nginx
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

## Example: Pod with Node Affinity - Example2

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: cart
  template:
    metadata:
      labels:
        app: cart
    spec:
      containers:
      - name: my-con
        image: nginx
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
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 2
            preference:
              matchExpressions:
              - key: servertype
                operator: In
                values:
                  - prod

          - weight: 1
            preference:
              matchExpressions:
              - key: servertype
                operator: In
                values:
                  - stage                
```

**Verify node labels**:
   ```bash
   kubectl get pods -o wide
   ```


## Operators Available

| Operator | Description |
|----------|-------------|
| In | Label's value is in the provided set |
| NotIn | Label's value is not in the provided set |
| Exists | Node has the label (value doesn't matter) |
| DoesNotExist | Node does not have the label |
| Gt | Label's value is greater than the specified value |
| Lt | Label's value is less than the specified value |

## Best Practices

- Use `requiredDuringScheduling` for critical requirements
- Use `preferredDuringScheduling` for optimization preferences
- Combine with Pod Affinity/Anti-Affinity for advanced scheduling
- Test affinity rules in a non-production environment first
- Monitor scheduling failures when using strict requirements

## Comparison with NodeSelector

| Feature | NodeSelector | Node Affinity |
|---------|-------------|---------------|
| Complexity | Simple key-value matching | Complex expressions |
| Requirements | Only hard requirements | Both hard and soft |
| Operators | Only equality | Multiple operators |
| Logical AND | Implicit (all must match) | Explicit |
| Logical OR | Not supported | Supported via multiple terms |

## Cleanup

To remove the example pod:
```bash
kubectl delete pod node-affinity-example
```

To remove node labels:
```bash
kubectl label nodes <node-name> topology.kubernetes.io/zone-
kubectl label nodes <node-name> instance-type-
```

# **Kubernetes Node Affinity Examples (`nodeAffinity`)**

Node affinity in Kubernetes allows you to constrain which nodes your Pod can be scheduled on based on node labels. Below are examples of different operators (`In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`) in a `README.md` format.

---

## **1. `In` Operator (Match a Label Value)**
Ensures the Pod is scheduled on nodes with a specific label value.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-node-affinity-in
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "disktype"
            operator: "In"
            values: ["ssd"]
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Pod will **only** run on nodes with the label `disktype=ssd`.

---

## **2. `NotIn` Operator (Exclude a Label Value)**
Ensures the Pod is **not** scheduled on nodes with a specific label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-node-affinity-notin
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "environment"
            operator: "NotIn"
            values: ["production"]
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Pod will **not** run on nodes labeled `environment=production`.

---

## **3. `Exists` Operator (Label Must Exist)**
Ensures the Pod is scheduled only if a node has a specific label (regardless of value).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-node-affinity-exists
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "gpu"
            operator: "Exists"
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Pod will run **only if** the node has a `gpu` label (value doesn’t matter).

---

## **4. `DoesNotExist` Operator (Label Must Not Exist)**
Ensures the Pod is scheduled only if a node **does not** have a specific label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-node-affinity-doesnotexist
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "gpu"
            operator: "DoesNotExist"
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Pod will run **only if** the node **does not** have a `gpu` label.

---

## **5. `Gt` (Greater Than) and `Lt` (Less Than) Operators**
Used for numeric comparisons (e.g., CPU, memory).

### **Example: `Gt` (Greater Than)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-node-affinity-gt
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "cpu-cores"
            operator: "Gt"
            values: ["4"]
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Pod will run **only if** the node has `cpu-cores > 4`.

### **Example: `Lt` (Less Than)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-node-affinity-lt
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "memory-size"
            operator: "Lt"
            values: ["16"]
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Pod will run **only if** the node has `memory-size < 16` (e.g., in GB).

---

## **6. Combining Multiple Rules**
You can combine multiple expressions for complex scheduling.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-complex-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "disktype"
            operator: "In"
            values: ["ssd", "nvme"]
          - key: "environment"
            operator: "NotIn"
            values: ["staging", "dev"]
          - key: "cpu-cores"
            operator: "Gt"
            values: ["2"]
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Pod will run on nodes with:
  - `disktype=ssd` **or** `disktype=nvme`
  - `environment` **not** `staging` or `dev`
  - `cpu-cores > 2`

---

## **7. Preferred Affinity (`preferredDuringSchedulingIgnoredDuringExecution`)**
Soft rule (preference, not requirement).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-preferred-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: "zone"
            operator: "In"
            values: ["us-east-1a"]
  containers:
  - name: nginx
    image: nginx
```
**Explanation:**  
- Kubernetes **prefers** nodes in `zone=us-east-1a`, but schedules elsewhere if needed.

---

## **Summary Table of Operators**
| Operator | Description | Example |
|----------|------------|---------|
| `In` | Label value must match one in the list | `values: ["ssd"]` |
| `NotIn` | Label value must **not** match any in the list | `values: ["production"]` |
| `Exists` | Label must exist (value ignored) | `key: "gpu"` |
| `DoesNotExist` | Label must **not** exist | `key: "gpu"` |
| `Gt` | Value must be **greater than** (numeric) | `values: ["4"]` |
| `Lt` | Value must be **less than** (numeric) | `values: ["16"]` |

---

## **Next Steps**
- Apply these examples using `kubectl apply -f <file>.yaml`.
- Check scheduling with `kubectl describe pod <pod-name>`.
- Modify based on your node labels (`kubectl get nodes --show-labels`).

