Installation Directory Structure
```
/srv/firecracker/
├── kernels/
│   ├── vmlinux                      # Kernel binary
└── rootfs/
    └── image.ext4                   # Ubuntu rootfs image

/srv/jailer/
└── firecracker/
    └── test-vm-001/
        └── root/
            ├── kernel/
            │   └── vmlinux
            ├── rootfs/
            │   └── image.ext4
            └── run/                 # For Firecracker socket

/srv/syzkaller/
└── workdir/                         # Syzkaller fuzzing output

/usr/local/bin/
├── firecracker                      # Firecracker binary
└── jailer                           # Jailer binary

/opt/syzkaller/
├── bin/linux_amd64/
│   ├── syz-manager
│   └── syz-fuzzer
└── executor/
    └── executor                     # Static executor binary
```

## Execution Order:

- Host Setup.md
- Linux Kernel Setup.md
- Ubuntu_Rootfs.md
- Jailer Chroot.md
- Firecracker VM Boot (Unguarded).md
- Jailer-Guarded VM Boot.md
- Syzkaller.md
- Syzkaller Configs.md
- Run_Syzkaller.md
- Result.md
- comparison.md



