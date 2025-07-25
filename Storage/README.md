```
## Introduction
Kubernetes volumes enable data persistence and sharing between containers in a pod. They decouple storage from pods, allowing applications to maintain state across restarts and reschedules.
```
## Volume Types

### Ephemeral Volumes
- Tied to pod lifecycle
- Common types:
  - `emptyDir`: Temporary directory created when pod starts
  - `hostPath`: Mounts a host filesystem path (use cautiously)

### Persistent Volumes (PV)
- Cluster-wide storage resources
- Lifecycle independent of pods
- Can be provisioned:
  - **Statically**: Admin creates PVs manually
  - **Dynamically**: Using StorageClasses

## Core Concepts

### Persistent Volume Claims (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

### Storage Classes
Define storage "profiles":
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

## Supported Storage Backends
| Type              | Examples                          |
|-------------------|-----------------------------------|
| Cloud Providers   | AWS EBS, GCE Persistent Disk      |
| Network Storage   | NFS, CephFS, GlusterFS            |
| Local Storage     | hostPath, local volumes           |
| Special Purpose   | ConfigMap, Secret, CSI drivers    |

## Usage Example
1. Create StorageClass (if dynamic provisioning)
2. Create PVC
3. Mount in pod:
```yaml
volumes:
  - name: app-data
    persistentVolumeClaim:
      claimName: my-pvc
```

## Best Practices
- Use dynamic provisioning for most cases
- Set appropriate reclaim policies (Retain/Delete)
- Match AccessModes to workload requirements
- Monitor volume usage with `kubectl top pod`
- Consider CSI drivers for advanced storage features

---
*For detailed documentation, see [Kubernetes Storage Docs](https://kubernetes.io/docs/concepts/storage/)*
```
