```bash
# Create kernel build directory
mkdir -p /tmp/kernel-build
cd /tmp/kernel-build

# Download Linux kernel (5.15.x recommended for Firecracker)
KERNEL_VERSION="5.15.160"
wget -q "https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-${KERNEL_VERSION}.tar.xz"
tar -xf "linux-${KERNEL_VERSION}.tar.xz"
cd "linux-${KERNEL_VERSION}"

# Copy minimal Firecracker-compatible config
cat > .config << 'EOF'
CONFIG_LOCALVERSION="-firecracker"
CONFIG_SERIAL_8250=y
CONFIG_SERIAL_8250_CONSOLE=y
CONFIG_DEVTMPFS=y
CONFIG_DEVTMPFS_MOUNT=y
CONFIG_TMPFS=y
CONFIG_EXT4_FS=y
CONFIG_EXT4_FS_POSIX_ACL=y
CONFIG_HAVE_KVM=y
CONFIG_KVM=y
CONFIG_KVM_INTEL=y
CONFIG_NET=y
CONFIG_INET=y
CONFIG_NETDEVICES=y
CONFIG_VIRTIO=y
CONFIG_VIRTIO_BLK=y
CONFIG_VIRTIO_NET=y
CONFIG_VIRTIO_PCI=y
CONFIG_VIRTIO_CONSOLE=y
CONFIG_9P_FS=y
CONFIG_9P_FS_RDONLY=y
CONFIG_NET_9P=y
CONFIG_NET_9P_VIRTIO=y
CONFIG_UNIX=y
CONFIG_BINFMT_ELF=y
CONFIG_MODULES=y
CONFIG_BLK_DEV_INITRD=y
CONFIG_RD_GZIP=y
CONFIG_HAVE_EFFICIENT_UNALIGNED_ACCESS=y
CONFIG_PRINTK=y
CONFIG_PANIC_ON_OOPS=1
EOF

# Build kernel
make -j$(nproc) vmlinux

# Copy vmlinux to firecracker directory
sudo cp vmlinux /srv/firecracker/kernels/vmlinux

# (Optional) Also build initramfs
make -j$(nproc) Image.gz
sudo cp arch/x86/boot/Image.gz /srv/firecracker/kernels/

echo "[*] Kernel built and copied to /srv/firecracker/kernels/"
```
```
