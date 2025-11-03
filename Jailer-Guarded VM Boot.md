```bash
# Set up jailer VM config
JAILER_BASE="/srv/jailer/firecracker"
JAILER_VM_ID="test-vm-001"
JAILER_UID=1000
JAILER_GID=1000

# Create non-root user for jailer (optional but recommended)
sudo useradd -m -u $JAILER_UID -s /sbin/nologin firecracker || true

# Create config inside chroot
CONFIG_PATH="$JAILER_BASE/$JAILER_VM_ID/root/vm-config.json"
cat | sudo tee "$CONFIG_PATH" << 'JSON_EOF'
{
  "machine-config": {
    "vcpu_count": 2,
    "mem_size_mib": 512,
    "ht_enabled": false
  },
  "boot-source": {
    "kernel_image_path": "/kernel/vmlinux",
    "boot_args": "console=ttyS0 root=/dev/vda rw",
    "initrd_path": ""
  },
  "drives": [
    {
      "drive_id": "rootfs",
      "path_on_host": "/rootfs/image.ext4",
      "is_root_device": true,
      "is_read_only": false
    }
  ]
}
JSON_EOF

# Launch via jailer
sudo /usr/local/bin/jailer \
  --id "$JAILER_VM_ID" \
  --exec-file /usr/local/bin/firecracker \
  --uid $JAILER_UID \
  --gid $JAILER_GID \
  --chroot-base-dir "$JAILER_BASE" \
  -- \
  --config-file /vm-config.json \
  --api-sock /run/firecracker.socket
```
