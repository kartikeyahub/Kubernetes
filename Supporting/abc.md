1. **Set proper permissions**:
   ```bash
   swapoff -a
   kubeadm reset --force
   ```


2. **Containerd Cleanup Procedures**


##  Nuclear Option (For Stubborn Containers)
```bash
# First stop all Kubernetes services
sudo systemctl stop kubelet

# Stop containerd service
sudo systemctl stop containerd

# Forcefully clean up runtime directories
sudo umount -l /run/containerd/io.containerd.runtime.v2.task/k8s.io/*/rootfs 2>/dev/null || true
sudo rm -rf /run/containerd/*
```

## Restart Services
```bash
sudo systemctl start containerd
sudo systemctl start kubelet
```

## Verification
```bash
# Check runtime directory
sudo ls -la /run/containerd/io.containerd.runtime.v2.task/k8s.io/

# Check for remaining shim processes
ps aux | grep containerd-shim
```

## Complete Reset (Last Resort)
```bash
sudo systemctl stop kubelet containerd
sudo rm -rf /run/containerd/* /var/lib/containerd/*
sudo systemctl start containerd
sudo systemctl start kubelet

```


3. **Initialize cluster**:
   ```bash
   sudo kubeadm init --ignore-preflight-errors=DirAvailable--var-lib-etcd
   ```

4. **Clearing .kube DIR**:
   ```bash
   rm -rf .kube/
   ```

5. **kubeconfig update**:
   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

6. **Install Calico**:
   ```bash
   kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
   ```

7. **Worker Node Join Token - { Master Node Config }**
   ```bash
   sudo kubeadm token create --print-join-command
   ```

8. **Worker Node Join Token - { Master Node Config }**
   ```bash
   sudo su -
   ```