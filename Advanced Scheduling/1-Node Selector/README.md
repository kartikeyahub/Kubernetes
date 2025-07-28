# Node Selector and HostPath Example

## Node Selector Overview

A Node Selector is a simple way to constrain Pods to nodes with specific labels. It allows you to assign Pods to particular nodes based on node characteristics like hardware, location, or other custom attributes.

### Key Points:
- Works by matching node labels with Pod specifications
- Basic form of node selection (for more complex requirements, use node affinity)
- Requires nodes to be pre-labeled with the desired attributes

## HostPath Overview

HostPath is a volume type that mounts a file or directory from the host node's filesystem into your Pod. This is useful for:
- Accessing node-specific system files
- Sharing data between containers on the same node
- Development and testing scenarios

⚠️ **Warning**: HostPath volumes can present security risks and should be used carefully in production environments.

## Example: Pod with Node Selector and HostPath


```yaml
apiVersion : v1
kind : Pod 
metadata :
  name : poda
spec :
  containers :
  - name : con1
    image : nginx
    volumeMounts:
    - name : hvol
      mountPath: /usr/share/nginx/html
  volumes:
  - name : hvol
    hostPath:
      path : /ks/data
      type : Directory
  nodeSelector :
    disktype : ssd  # Must exist
```

## Deployment with Node Selector and HostPath

```yaml
apiVersion : apps/v1
kind : Deployment
metadata :
  name : dp1
spec : 
  replicas : 5
  selector : 
    matchLabels :
      env : dev
  template : 
    metadata :
      labels :
        env : dev
    spec :
      containers :
      - name : con1
        image : nginx
        volumeMounts:
        - name : hvol
          mountPath: /usr/share/nginx/html
      volumes:
      - name : hvol
        hostPath:
          path : /ks/data
          type : Directory
      nodeSelector :
        disktype : ssd  # Must exist
```

## Prerequisites

1. **Label your nodes**:
   ```bash
   kubectl label nodes <node-name> disktype=ssd
   kubectl label nodes <node-name> environment=production
   ```

2. **Verify node labels**:
   ```bash
   kubectl get nodes --show-labels
   ```

3. **Create directory on node** (if not using DirectoryOrCreate type):
   ```bash
   # SSH into your node first
   mkdir -p /data/app-volume
   ```
4. **Remove the node labels** :
   ```bash
   kubectl label nodes <node-name> disktype-
   ```


## Best Practices

- Use NodeSelectors for simple scheduling requirements
- For complex scheduling, consider using Node Affinity instead
- Be cautious with HostPath volumes in production
- When using HostPath, consider:
  - Restricting access with readOnly: true
  - Using specific types like DirectoryOrCreate
  - Implementing appropriate Pod Security Policies

## Cleanup

To remove the example pod:
```bash
kubectl delete pod node-selector-hostpath-example
```

To remove node labels:
```bash
kubectl label nodes <node-name> disktype-
kubectl label nodes <node-name> environment-
```