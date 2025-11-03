**Config for Non-Hardened VM**
```bash
mkdir -p /srv/syzkaller/configs

cat > /srv/syzkaller/configs/non-hardened.json << 'EOF'
{
  "target": "linux/amd64",
  "http": "127.0.0.1:56741",
  "workdir": "/srv/syzkaller/workdir-nh",
  "kernel_obj": "/srv/firecracker/kernels",
  "image": "/srv/firecracker/rootfs/image.ext4",
  "syzkaller": "/opt/syzkaller",
  "procs": 2,
  "type": "firecracker",
  "vm": {
    "kernel": "/srv/firecracker/kernels/vmlinux",
    "image": "/srv/firecracker/rootfs/image.ext4",
    "image_size": 4294967296,
    "socket": "/tmp/firecracker-syz-nh.socket",
    "cpu_count": 2,
    "mem_mb": 512,
    "timeout": 30,
    "cmdline": "console=ttyS0 root=/dev/vda rw init=/syz-executor"
  },
  "reproduce": false,
  "cover": false,
  "enable_syscalls": [
    "bpf", "futex", "futex2", "openat", "ioctl", "read", "write",
    "mmap", "mprotect", "clone", "socket", "sendto", "bind"
  ]
}
EOF

# Create workdir
sudo mkdir -p /srv/syzkaller/workdir-nh
sudo chmod 755 /srv/syzkaller/workdir-nh

echo "Non-hardened config created"
```

**Config for Hardened VM**
```bash
cat > /srv/syzkaller/configs/hardened.json << 'EOF'
{
  "target": "linux/amd64",
  "http": "127.0.0.1:56742",
  "workdir": "/srv/syzkaller/workdir-h",
  "kernel_obj": "/srv/firecracker/kernels",
  "image": "/srv/firecracker/rootfs/image.ext4",
  "syzkaller": "/opt/syzkaller",
  "procs": 2,
  "type": "firecracker",
  "vm": {
    "kernel": "/srv/firecracker/kernels/vmlinux",
    "image": "/srv/firecracker/rootfs/image.ext4",
    "image_size": 4294967296,
    "socket": "/tmp/firecracker-syz-h.socket",
    "cpu_count": 2,
    "mem_mb": 512,
    "timeout": 30,
    "cmdline": "console=ttyS0 root=/dev/vda rw init=/syz-executor",
    "jailer": {
      "id": "hardened-vm-001",
      "exec_file": "/usr/local/bin/jailer",
      "uid": 1000,
      "gid": 1000,
      "chroot_base_dir": "/srv/jailer/firecracker"
    }
  },
  "reproduce": false,
  "cover": false,
  "enable_syscalls": [
    "bpf", "futex", "futex2", "openat", "ioctl", "read", "write",
    "mmap", "mprotect", "clone", "socket", "sendto", "bind"
  ]
}
EOF

# Create workdir
sudo mkdir -p /srv/syzkaller/workdir-h
sudo chmod 755 /srv/syzkaller/workdir-h

echo "Hardened config created"
```

### Verify Configs
```bash
cat /srv/syzkaller/configs/non-hardened.json
cat /srv/syzkaller/configs/hardened.json
```
