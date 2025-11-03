```bash
# Install Go 
GO_VERSION="1.22.0"
cd /tmp
wget -q "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"
export PATH="$PATH:/usr/local/go/bin"

# Clone and build syzkaller
SYZKALLER_DIR="/opt/syzkaller"
sudo git clone https://github.com/google/syzkaller "$SYZKALLER_DIR"
cd "$SYZKALLER_DIR"
sudo make clean && sudo make

# Build executor
cd executor
sudo gcc -static -O2 -Wall -pthread executor.c -o executor
sudo mv executor /opt/syzkaller/bin/linux_amd64/

# Create syzkaller workdir
sudo mkdir -p /srv/syzkaller/workdir
sudo chmod 755 /srv/syzkaller/workdir

# Copy executor into rootfs for syzkaller to use
sudo mount -o loop /srv/firecracker/rootfs/image.ext4 /mnt
sudo mkdir -p /mnt/usr/local/bin
sudo cp /opt/syzkaller/bin/linux_amd64/executor /mnt/usr/local/bin/
sudo chmod 755 /mnt/usr/local/bin/executor
sudo umount /mnt
```
