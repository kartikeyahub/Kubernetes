## Introduction
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
   ls -la /data/hostpath-demo
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
