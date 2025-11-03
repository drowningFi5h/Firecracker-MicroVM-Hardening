## Run Syzkaller on Non-Hardened VM
```bash
# Kill any old processes
pkill -f syz-manager || true
pkill -f firecracker || true
sleep 2

# Start fuzzing
sudo /opt/syzkaller/bin/linux_amd64/syz-manager \
  -config=/srv/syzkaller/configs/non-hardened.json
```

### Monitor in Another Terminal
```bash
# Watch for crashes
watch -n 5 'find /srv/syzkaller/workdir-nh -type f -name "*.txt" | wc -l'
tail -f /tmp/syz-manager-nh.log
```

## Run Syzkaller on Hardened VM
```bash
# Kill previous fuzzer
pkill -f syz-manager || true
pkill -f firecracker || true
pkill -f jailer || true
sleep 2

# Start fuzzing hardened VM
sudo /opt/syzkaller/bin/linux_amd64/syz-manager \
  -config=/srv/syzkaller/configs/hardened.json
```

