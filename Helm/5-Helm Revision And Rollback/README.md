# Helm Revision and Rollback

## What is a Revision in Helm?

In Helm, a **revision** refers to a specific version of a release. Every time you install, upgrade, or rollback a Helm chart, it creates a new revision. These revisions are stored in the Kubernetes cluster and allow you to track changes and revert to previous versions if needed.

## How to View Revisions

```bash
# List all revisions of a release
helm history my-release

# Example output:
REVISION    UPDATED                     STATUS      CHART         APP VERSION     DESCRIPTION     
1           Mon Jan 15 10:30:00 2024    deployed    myapp-1.2.0   1.0.0           Install complete
2           Mon Jan 15 11:45:00 2024    deployed    myapp-1.2.1   1.0.0           Upgrade complete
3           Mon Jan 15 12:15:00 2024    failed      myapp-1.2.1   1.0.0           Upgrade failed
4           Mon Jan 15 12:30:00 2024    deployed    myapp-1.2.0   1.0.0           Rollback to 1
```

## How to Rollback to Different Versions

### Basic Rollback Syntax
```bash
helm rollback <RELEASE_NAME> <REVISION_NUMBER>
```

### Rollback Examples

**Example 1: Rollback to the previous revision**
```bash
# Rollback to revision 2
helm rollback my-release 2

# Output:
Rollback was a success! Happy Helming!
```

**Example 2: Rollback to a specific older revision**
```bash
# Check available revisions first
helm history my-release

# Rollback to revision 1
helm rollback my-release 1
```

**Example 3: Rollback with custom timeout**
```bash
# Rollback with a 5-minute timeout
helm rollback my-release 3 --timeout 5m
```

**Example 4: Rollback and wait for completion**
```bash
# Rollback and wait for all resources to be ready
helm rollback my-release 2 --wait
```

**Example 5: Rollback with atomic option (rollback on failure)**
```bash
# If the rollback fails, revert the changes
helm rollback my-release 4 --atomic
```

## Complete Workflow Example

```bash
# 1. Install a chart (creates revision 1)
helm install my-app ./my-chart --version 1.0.0

# 2. Upgrade to a new version (creates revision 2)
helm upgrade my-app ./my-chart --version 1.1.0

# 3. Check revision history
helm history my-app

# 4. Discover the new version has issues, rollback to revision 1
helm rollback my-app 1

# 5. Verify the rollback was successful
helm status my-app
helm get values my-app
```

## Advanced Rollback Scenarios

**Rollback and keep history:**
```bash
# By default, rollback creates a new revision
helm rollback my-release 3
```

**Force rollback (if previous rollback failed):**
```bash
helm rollback my-release 2 --force
```

**Rollback with specific values:**
```bash
# You can override values during rollback
helm rollback my-release 2 --set image.tag=stable
```

## Checking Rollback Status

```bash
# Check current status
helm status my-release

# View current values
helm get values my-release

# See what changed between revisions
helm diff revision my-release 2 3
```

## Important Notes

1. **Revisions are stored in Kubernetes secrets** (by default)
2. **Rollback creates a new revision** - it doesn't delete the failed revision
3. **Some resources may not rollback cleanly** (like PersistentVolumes)
4. **Use `--cleanup-on-fail`** during upgrades to automatically clean up failed installations
5. **The number of stored revisions is configurable** with `--history-max` during install/upgrade

## Troubleshooting Rollbacks

If a rollback fails, you can:
```bash
# Check why it failed
helm status my-release

# View logs of specific resources
kubectl logs deployment/my-app-deployment

# Force the rollback if necessary
helm rollback my-release 2 --force
```

This revision system makes Helm a powerful tool for managing Kubernetes applications with proper change tracking and recovery capabilities.