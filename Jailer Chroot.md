```bash
# Create jailer chroot base
JAILER_BASE="/srv/jailer/firecracker"
JAILER_VM_ID="test-vm-001"

sudo mkdir -p "$JAILER_BASE/$JAILER_VM_ID/root"
sudo chmod 755 "$JAILER_BASE" "$JAILER_BASE/$JAILER_VM_ID"

# Copy kernel to chroot
sudo mkdir -p "$JAILER_BASE/$JAILER_VM_ID/root/kernel"
sudo cp /srv/firecracker/kernels/vmlinux "$JAILER_BASE/$JAILER_VM_ID/root/kernel/"
sudo chmod 444 "$JAILER_BASE/$JAILER_VM_ID/root/kernel/vmlinux"

# Copy rootfs to chroot
sudo mkdir -p "$JAILER_BASE/$JAILER_VM_ID/root/rootfs"
sudo cp /srv/firecracker/rootfs/image.ext4 "$JAILER_BASE/$JAILER_VM_ID/root/rootfs/"
sudo chmod 444 "$JAILER_BASE/$JAILER_VM_ID/root/rootfs/image.ext4"

# Create socket directory in chroot (for Firecracker API)
sudo mkdir -p "$JAILER_BASE/$JAILER_VM_ID/root/run"
sudo chmod 755 "$JAILER_BASE/$JAILER_VM_ID/root/run"

# Verify structure
sudo tree "$JAILER_BASE/$JAILER_VM_ID/root" || sudo find "$JAILER_BASE/$JAILER_VM_ID/root" -type f
```
