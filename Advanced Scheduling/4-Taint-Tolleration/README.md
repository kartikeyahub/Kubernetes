# Kubernetes Taints and Tolerations

## Overview

Taints and tolerations are Kubernetes mechanisms that control pod scheduling on nodes. They ensure that pods are only scheduled on appropriate nodes based on specific requirements.

## Concepts

### Taints
- Applied to **nodes** to repel pods that don't match the taint
- Ensure only specific pods can be scheduled on the node
- Useful for dedicating nodes to special workloads

### Tolerations
- Applied to **pods** to allow scheduling on tainted nodes
- Must match the node's taint for the pod to be scheduled

## Key Components

| Component | Description | Example |
|-----------|-------------|---------|
| **Key** | Unique identifier for the taint | `env`, `disk` |
| **Value** | Value associated with the key | `prod`, `ssd` |
| **Effect** | Behavior enforced by the taint | `NoSchedule`, `PreferNoSchedule`, `NoExecute` |

### Taint Effects
- `NoSchedule`: Pods without toleration won't be scheduled
- `PreferNoSchedule`: System will try to avoid scheduling
- `NoExecute`: Evicts existing pods without toleration

## Examples

# Kubernetes Node Taints Management

## Table of Contents
- [Adding a Taint to a Node](#adding-a-taint-to-a-node)
- [Removing a Taint from a Node](#removing-a-taint-from-a-node)
- [Viewing Existing Taints](#viewing-existing-taints)
- [Common Taint Examples](#common-taint-examples)
- [Best Practices](#best-practices)

## Adding a Taint to a Node

### Using kubectl command
```sh
kubectl taint nodes <node-name> <key>=<value>:<effect>
kubectl taint node mynode123 env=prod:NoSchedule
```

Where:
- `<node-name>`: Name of your node (e.g., `node-1`)
- `<key>`: Taint identifier (e.g., `env`)
- `<value>`: Taint value (e.g., `production`)
- `<effect>`: One of `NoSchedule`, `PreferNoSchedule`, or `NoExecute`

**Examples:**
```sh
# Reserve node for production workloads
kubectl taint nodes node-1 env=production:NoSchedule

# Mark node for maintenance
kubectl taint nodes node-2 maintenance=true:NoExecute

# Prefer not to schedule GPU workloads
kubectl taint nodes gpu-node-1 hardware=gpu:PreferNoSchedule
```

### Using Node Manifest
```yaml
apiVersion: v1
kind: Node
metadata:
  name: node-1
spec:
  taints:
  - key: "env"
    value: "production"
    effect: "NoSchedule"
```
## Viewing Existing Taints

### List all nodes with taints
```sh
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

### Detailed view for a specific node
```sh
kubectl describe node <node-name> | grep Taints
```

### 3. Pod with Toleration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-app
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: env
      operator: Equal
      value: prod
      effect: NoSchedule
```

### 4. NoExecute Taint Example
```sh
kubectl taint node mynode123 disk=ssd:NoExecute
```

### 5. Pod with NoExecute Toleration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-app
spec:
  containers:
    - name: redis
      image: redis
  tolerations:
    - key: disk
      operator: Equal
      value: ssd
      effect: NoExecute
```


## Removing a Taint from a Node

### Basic Removal
```sh
kubectl taint nodes <node-name> <key>-
```

**Examples:**
```sh
# Remove taint with key 'env'
kubectl taint nodes node-1 env-

# Remove specific taint (when multiple exist with same key)
kubectl taint nodes node-1 env=production:NoSchedule-
```

### Removing All Taints
```sh
kubectl taint nodes <node-name> --all
```

## Best Practices

1. Use descriptive keys and values (e.g., `gpu=true`, `env=production`)
2. Start with `PreferNoSchedule` before using `NoSchedule`
3. Use `NoExecute` cautiously as it can disrupt running pods
4. Combine with node affinity for more precise scheduling

## Common Use Cases

- **Dedicated Hardware**: Reserve GPU nodes for machine learning workloads
- **Environment Separation**: Isolate production and development workloads
- **Maintenance Preparation**: Drain nodes by applying `NoExecute` taints
- **Specialized Nodes**: Create pools for SSD, high-memory, or high-CPU nodes

## Troubleshooting

- `kubectl describe node <node-name>` - View taints applied to a node
- `kubectl get pods -o wide` - Check where pods are scheduled
- `kubectl describe pod <pod-name>` - View pod tolerations

## FAQ

**Q: Can a pod tolerate multiple taints?**  
A: Yes, a pod can have multiple tolerations for different taints.

**Q: What happens if I remove a taint from a node?**  
A: The scheduling restriction is immediately lifted, and pods without toleration can be scheduled.

**Q: How do I remove a taint?**  
A: `kubectl taint node <node-name> <key>-` (note the trailing hyphen)
```

