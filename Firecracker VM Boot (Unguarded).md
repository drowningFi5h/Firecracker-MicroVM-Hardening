```bash
# Create working directory
mkdir -p /tmp/firecracker-test
cd /tmp/firecracker-test

# Create Firecracker socket and API config
SOCKET="/tmp/firecracker-test.socket"
CONFIG="vm-config.json"

# Write machine config
cat > "$CONFIG" << 'JSON_EOF'
{
  "machine-config": {
    "vcpu_count": 2,
    "mem_size_mib": 512,
    "ht_enabled": false
  },
  "boot-source": {
    "kernel_image_path": "/srv/firecracker/kernels/vmlinux",
    "boot_args": "console=ttyS0 root=/dev/vda rw",
    "initrd_path": ""
  },
  "drives": [
    {
      "drive_id": "rootfs",
      "path_on_host": "/srv/firecracker/rootfs/image.ext4",
      "is_root_device": true,
      "is_read_only": false
    }
  ]
}
JSON_EOF

# Start Firecracker
rm -f "$SOCKET"
firecracker --api-sock "$SOCKET" &
FIRECRACKER_PID=$!

sleep 2

# Apply configuration via API (note: order matters!)
curl -X PUT \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d "$(cat $CONFIG | jq '.["machine-config"]')" \
  --unix-socket "$SOCKET" \
  http://localhost/machine

curl -X PUT \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d "$(cat $CONFIG | jq '.["boot-source"]')" \
  --unix-socket "$SOCKET" \
  http://localhost/boot-source

curl -X PUT \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d "$(cat $CONFIG | jq '.["drives"][0]')" \
  --unix-socket "$SOCKET" \
  http://localhost/drives/rootfs

# Boot the VM
curl -X PUT \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d "{}" \
  --unix-socket "$SOCKET" \
  http://localhost/actions?action_type=InstanceStart

# Connect to serial console
socat - UNIX-CONNECT:"$SOCKET" &
sleep 1

echo "[*] VM booting. Press Ctrl+C to stop."
wait $FIRECRACKER_PID
```
