# Introduction
Kubernetes volumes enable data persistence and sharing between containers in a pod. They decouple storage from pods, allowing applications to maintain state across restarts and reschedules.

## Volume Types

### Ephemeral Volumes
- Tied to pod lifecycle
- Common types:
  - `hostPath`: Mounts a host filesystem path (use cautiously)
  - `emptyDir`: Temporary directory created when pod starts
 

### HostPath
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
  labels:
    app: hostpath-test
spec:
  containers:
  - name: app-container
    image: nginx:alpine
    volumeMounts:
    - name: hostpath-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: hostpath-volume
    hostPath:
      path: /ks/data
```


2. **Check pod status**:
   ```bash
   kubectl get pods -o wide
   ```

3. **Check host directory**:
   ```bash
   ls -la /ks/data
   ```

3. **Connect to Container**:
   ```bash
   kubectl exec -it hostpath-demo -- /bin/sh
   ```
3. **Verify volume mounting**:
   ```bash
   kubectl exec hostpath-demo -- ls /usr/share/nginx/html
   ```




`hostPath` volumes mount files or directories from the host node's filesystem into pods. Use with caution due to security implications.

## Key Configuration Options

| Field            | Recommended Value       | Purpose                                                                 |
|------------------|-------------------------|-------------------------------------------------------------------------|
| `type`           | `Directory`             | Strict directory requirement (must pre-exist)                           |
|                  | `DirectoryOrCreate`     | Auto-creates directory if missing                                      |
|                  | `File`                  | Mounts a single file                                                   |
|                  | `Socket`                | For Unix domain sockets                                                |
| `readOnly`       | `true`                  | Prevents container modifications (when possible)                       |
| `runAsNonRoot`   | `true`                  | Prevents running as root (security best practice)                      |

## When to Use hostPath

✅ **Appropriate Use Cases**:
- Accessing node-specific files (logs, libs, binaries)
- Development/testing environments
- Single-node clusters
- Performance-critical storage needs

❌ **Avoid When**:
- Running multi-node clusters
- Need portable storage across nodes
- Deploying untrusted workloads

-------------------------------------------------------------

# Kubernetes `emptyDir` Volumes

## Overview
`emptyDir` is a temporary storage volume created when a Pod is assigned to a node. It exists for the Pod's lifetime and is deleted when the Pod is removed.

## Key Features

| Characteristic       | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| **Lifetime**         | Created when Pod starts, deleted when Pod terminates                       |
| **Storage Location** | Node filesystem (default) or RAM (`medium: Memory`)                        |
| **Capacity**         | Can be limited with `sizeLimit`                                            |
| **Access**           | Shared between containers in the same Pod                                   |
| **Performance**      | RAM-backed is faster but volatile                                          |

## When to Use

✅ **Appropriate Use Cases**:
- Temporary cache storage (e.g., NGINX cache)
- Sharing files between containers in the same Pod
- Scratch space for batch processing
- Checkpointing for crash recovery

❌ **Avoid When**:
- Data persistence is required
- Sharing data across different Pods
- Storing sensitive data in RAM-backed volumes

## Example Configurations

### Basic Usage
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: basic-emptydir
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
  volumes:
  - name: cache
    emptyDir: {}
```

### RAM-Backed with Size Limit
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ram-emptydir
spec:
  containers:
  - name: processor
    image: alpine
    command: ["sh", "-c", "while true; do echo $(date) >> /data/log; sleep 1; done"]
    volumeMounts:
    - name: mem-vol
      mountPath: /data
  volumes:
  - name: mem-vol
    emptyDir:
      medium: Memory
      sizeLimit: 256Mi
```

## Best Practices

1. **Security**:
   ```yaml
   securityContext:
     runAsNonRoot: true
     readOnlyRootFilesystem: true
   ```

2. **Resource Management**:
   - Always set `sizeLimit` for RAM-backed volumes
   - Monitor usage with `kubectl exec <pod> -- df -h`

3. **Alternatives**:
   - Use `PersistentVolumeClaim` for persistent storage
   - Consider `hostPath` for node-specific temporary storage

## Verification

```bash
# Check volume mounts
kubectl exec basic-emptydir -- ls /var/cache/nginx

# Monitor RAM usage
kubectl exec ram-emptydir -- df -h /data
```


----------------------------------------------------------

### EmptyDir
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: empty-demo
  labels:
    app: empty-test
spec:
  containers:
  - name: app-container
    image: nginx:alpine
    volumeMounts:
    - name: empty-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: empty-volume
    emptyDir: {}
```


3. **Connect to Container**:
   ```bash
   kubectl exec -it empty-demo -- /bin/sh
   ```
3. **Verify volume mounting**:
   ```bash
   df -h #check mountpoint
   ```




------------------------------------------------------------------




# Node Selector

### Secure Configuration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-hostpath
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: alpine
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: restricted-vol
      mountPath: /data
      readOnly: true
  volumes:
  - name: restricted-vol
    hostPath:
      path: /mnt/secure-data
      type: Directory  # Must exist
```

## Best Practices

1. **Security**:
   - Always set `runAsNonRoot: true`
   - Use `readOnly: true` when possible
   - Restrict to specific nodes with `nodeSelector`

2. **Alternatives**:
   ```yaml
   volumes:
     - name: safer-storage
       persistentVolumeClaim:
         claimName: my-pvc
   ```

3. **Cleanup**:
   ```bash
   kubectl delete pod hostpath-example
   rm -rf /data/hostpath  # Clean host directory
   ```
