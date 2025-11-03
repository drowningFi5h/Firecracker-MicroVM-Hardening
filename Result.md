```bash
# Create report directory
mkdir -p /tmp/syz-report

# Count crashes in non-hardened
NH_CRASHES=$(find /srv/syzkaller/workdir-nh/crashes -type d -mindepth 1 2>/dev/null | wc -l)
NH_HANGS=$(find /srv/syzkaller/workdir-nh/hangs -type d -mindepth 1 2>/dev/null | wc -l)

# Count crashes in hardened
H_CRASHES=$(find /srv/syzkaller/workdir-h/crashes -type d -mindepth 1 2>/dev/null | wc -l)
H_HANGS=$(find /srv/syzkaller/workdir-h/hangs -type d -mindepth 1 2>/dev/null | wc -l)

# Create report
cat > /tmp/syz-report/COMPARISON_REPORT.txt << EOF
======================================
SYZKALLER FUZZING COMPARISON REPORT
======================================
Generated: $(date)

--- NON-HARDENED VM ---
Crashes Found:  $NH_CRASHES
Hangs Found:    $NH_HANGS
Total Issues:   $((NH_CRASHES + NH_HANGS))

Workdir:  /srv/syzkaller/workdir-nh
Config:   /srv/syzkaller/configs/non-hardened.json

--- HARDENED VM (with Jailer) ---
Crashes Found:  $H_CRASHES
Hangs Found:    $H_HANGS
Total Issues:   $((H_CRASHES + H_HANGS))

Workdir:  /srv/syzkaller/workdir-h
Config:   /srv/syzkaller/configs/hardened.json

--- ANALYSIS ---
Issue Reduction: $((NH_CRASHES + NH_HANGS - H_CRASHES - H_HANGS)) issues prevented by hardening
Percentage Reduction: $(echo "scale=2; ((($NH_CRASHES + $NH_HANGS) - ($H_CRASHES + $H_HANGS)) / ($NH_CRASHES + $NH_HANGS + 1)) * 100" | bc)%

======================================
EOF

# Print report
cat /tmp/syz-report/COMPARISON_REPORT.txt
```

Extract Detailed Crash Information
```bash
# Create export directories
mkdir -p /tmp/syz-report/{crashes-nh,crashes-h,hangs-nh,hangs-h}

# Export non-hardened crashes
for crash_dir in /srv/syzkaller/workdir-nh/crashes/*/; do
    [ -d "$crash_dir" ] || continue
    crash_name=$(basename "$crash_dir")
    mkdir -p "/tmp/syz-report/crashes-nh/$crash_name"
    [ -f "$crash_dir/log" ] && cp "$crash_dir/log" "/tmp/syz-report/crashes-nh/$crash_name/"
    [ -f "$crash_dir/description" ] && cp "$crash_dir/description" "/tmp/syz-report/crashes-nh/$crash_name/"
    [ -f "$crash_dir/reproducer.c" ] && cp "$crash_dir/reproducer.c" "/tmp/syz-report/crashes-nh/$crash_name/"
done

# Export hardened crashes
for crash_dir in /srv/syzkaller/workdir-h/crashes/*/; do
    [ -d "$crash_dir" ] || continue
    crash_name=$(basename "$crash_dir")
    mkdir -p "/tmp/syz-report/crashes-h/$crash_name"
    [ -f "$crash_dir/log" ] && cp "$crash_dir/log" "/tmp/syz-report/crashes-h/$crash_name/"
    [ -f "$crash_dir/description" ] && cp "$crash_dir/description" "/tmp/syz-report/crashes-h/$crash_name/"
    [ -f "$crash_dir/reproducer.c" ] && cp "$crash_dir/reproducer.c" "/tmp/syz-report/crashes-h/$crash_name/"
done

# Create index
ls -lah /tmp/syz-report/

echo ""
echo "Report exported to: /tmp/syz-report/"
echo "Contents:"
tree /tmp/syz-report/ || find /tmp/syz-report/ -type f | head -20
```
