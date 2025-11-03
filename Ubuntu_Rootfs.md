```bash
# Create rootfs build directory
ROOTFS_DIR="/tmp/ubuntu-rootfs"
sudo mkdir -p "$ROOTFS_DIR"
cd "$ROOTFS_DIR"

# Debootstrap minimal Ubuntu 22.04
SUITE="jammy"
sudo debootstrap --include=systemd,curl,vim,strace,ca-certificates \
    "$SUITE" \
    "$ROOTFS_DIR/root" \
    http://archive.ubuntu.com/ubuntu/

# Enter chroot and configure
sudo chroot "$ROOTFS_DIR/root" /bin/bash << 'CHROOT_EOF'
# Set hostname
echo "firecracker-vm" > /etc/hostname

# Configure networking
cat > /etc/systemd/network/eth0.network << 'NET_EOF'
[Match]
Name=eth0

[Network]
DHCP=yes
NET_EOF

# Create minimal fstab
cat > /etc/fstab << 'FSTAB_EOF'
/dev/vda / ext4 defaults 0 0
FSTAB_EOF

# Configure serial console
systemctl enable serial-getty@ttyS0.service

# Minimal init script for syzkaller
cat > /syz-executor << 'SYZ_EOF'
#!/bin/sh
exec /usr/local/bin/executor
SYZ_EOF
chmod +x /syz-executor

# Set root password (optional, for manual access)
echo "root:root" | chpasswd

# Cleanup
apt-get clean
rm -rf /var/lib/apt/lists/*
exit
CHROOT_EOF

# Convert rootfs to ext4 image
ROOTFS_IMG="/tmp/ubuntu-rootfs/image.ext4"
ROOTFS_SIZE="4G"

# Create sparse image file
dd if=/dev/zero of="$ROOTFS_IMG" bs=1 count=0 seek="$ROOTFS_SIZE"

# Format as ext4
sudo mkfs.ext4 -F "$ROOTFS_IMG"

# Mount and copy rootfs
sudo mount -o loop "$ROOTFS_IMG" /mnt
sudo cp -a "$ROOTFS_DIR/root"/* /mnt/
sudo umount /mnt

# Move to firecracker directory
sudo mv "$ROOTFS_IMG" /srv/firecracker/rootfs/image.ext4
sudo chmod 644 /srv/firecracker/rootfs/image.ext4

# Verify
ls -lh /srv/firecracker/rootfs/image.ext4
```
