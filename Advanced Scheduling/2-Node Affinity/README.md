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
   kubectl label nodes Kartikeyasoft-worker2 servertype=prod

   
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
  replicas: 3
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