# NFS Server Setup on Ubuntu

This guide provides step-by-step instructions to install and configure an NFS server on Ubuntu, allowing shared directories to be mounted on client machines.

---

## Prerequisites

- Ubuntu Server (20.04, 22.04, or later)
- Root or sudo privileges
- Basic knowledge of Linux command line

---

## Step 1: Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 2: Install NFS Kernel Server

```bash
sudo apt install nfs-kernel-server -y
```

---

## Step 3: Create Shared Directory

Create a directory to be shared over NFS:

```bash
sudo mkdir -p /var/nfs/general
```

Set appropriate permissions:

```bash
sudo chown nobody:nogroup /var/nfs/general
sudo chmod 777 /var/nfs/general
```

---

## Step 4: Configure Exports

Edit the `/etc/exports` file to specify shared directories and permissions:

```bash
sudo nano /etc/exports
```

Add the following line (modify IP addresses as needed):

```plaintext
/var/nfs/general 192.168.1.0/24(rw,sync,no_subtree_check)
```

- Replace `192.168.1.0/24` with your client subnet or specific IPs.
- Use `*` to allow all clients (less secure).

Save and close the file.

---

## Step 5: Export Shared Directory

Apply the export configuration:

```bash
sudo exportfs -ra
```

Verify the export:

```bash
sudo exportfs -v
```

---

## Step 6: Restart NFS Service

```bash
sudo systemctl restart nfs-kernel-server
```

Ensure the service is active:

```bash
sudo systemctl status nfs-kernel-server
```

---

## Step 7: Configure Firewall (Optional)

Allow NFS through UFW firewall:

```bash
sudo ufw allow from 192.168.1.0/24 to any port nfs
```

Adjust the IP range as necessary.

---

## Step 8: Mount NFS Share on Client

On the client machine, install NFS utilities:

```bash
sudo apt install nfs-common -y
```

Create a mount point:

```bash
sudo mkdir -p /mnt/nfs/general
```

Mount the NFS share:

```bash
sudo mount 192.168.1.28:/var/nfs/general /mnt/nfs/general
```

Replace `192.168.1.28` with your server's IP address.

---

## Troubleshooting

- Ensure the server's `/etc/exports` is correctly configured.
- Check server status:

```bash
sudo systemctl status nfs-kernel-server
```

- Verify exported directories:

```bash
sudo exportfs -v
```

- Confirm directory permissions.

---

## Notes

- For persistent mounts, add an entry to `/etc/fstab` on the client:

```plaintext
192.168.1.28:/var/nfs/general /mnt/nfs/general nfs defaults 0 0
```

---

## License

This setup guide is provided as-is. Use at your own discretion.

---

Would you like me to generate this as a downloadable file or customize it further?