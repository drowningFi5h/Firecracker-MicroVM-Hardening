```bash
# Create final report
cat > /tmp/syz-report/FINAL_REPORT.md << 'EOF'
# Syzkaller Fuzzing: Hardened vs Non-Hardened VM Report

## Results

### Non-Hardened VM
EOF

# Append results
echo "- **Crashes**: $NH_CRASHES" >> /tmp/syz-report/FINAL_REPORT.md
echo "- **Hangs**: $NH_HANGS" >> /tmp/syz-report/FINAL_REPORT.md

cat >> /tmp/syz-report/FINAL_REPORT.md << 'EOF'

### Hardened VM (with Jailer)
EOF

echo "- **Crashes**: $H_CRASHES" >> /tmp/syz-report/FINAL_REPORT.md
echo "- **Hangs**: $H_HANGS" >> /tmp/syz-report/FINAL_REPORT.md

cat >> /tmp/syz-report/FINAL_REPORT.md << 'EOF'


## Files

- `COMPARISON_REPORT.txt` - Numeric summary
- `crashes-nh/` - Non-hardened crash details
- `crashes-h/` - Hardened crash details
- `hangs-nh/` - Non-hardened hang details
- `hangs-h/` - Hardened hang details

EOF
# Print final report
cat /tmp/syz-report/FINAL_REPORT.md
```

View and Archive Results
```bash
# Display all reports
echo "=== COMPARISON REPORT ==="
cat /tmp/syz-report/COMPARISON_REPORT.txt

echo ""
echo "=== FINAL REPORT ==="
cat /tmp/syz-report/FINAL_REPORT.md

# List all exported crashes
echo ""
echo "=== EXPORTED CRASHES ==="
find /tmp/syz-report -name "log" | head -10

# Create tarball for archiving
tar -czf /tmp/syzkaller-fuzzing-report-$(date +%Y%m%d-%H%M%S).tar.gz /tmp/syz-report/

# Show archive
ls -lh /tmp/syzkaller-fuzzing-report-*.tar.gz
```
