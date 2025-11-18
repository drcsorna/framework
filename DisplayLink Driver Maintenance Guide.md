# DisplayLink Driver Maintenance Guide

## Quick Reference

**Current Setup:**
- DisplayLink version: 1.14.11
- EVDI version: 1.14.11
- Fedora: 43
- Kernel: 6.17.7-300.fc43.x86_64

---

## When You See DKMS Errors at Boot

### 1. Check What's Wrong

```bash
# Check DKMS status
sudo dkms status

# Check DKMS service errors
sudo journalctl -u dkms.service -b

# Check current kernel
uname -r

# Check DisplayLink package version
rpm -q displaylink
```

### 2. Common Issue: Kernel Too New for EVDI

**Symptoms:**
- Boot error: "dkms.service failed"
- DKMS shows: `evdi/X.XX.XX: added` but not `installed`
- Error log mentions compilation failures

**Cause:** The installed EVDI version is too old for your current kernel.

---

## How to Update DisplayLink Driver

### Option 1: Check for Updates in Your Repos

```bash
# Try updating from installed repos
sudo dnf update displaylink
```

### Option 2: Download New Release from GitHub

1. **Check latest release:**
   - Visit: https://github.com/displaylink-rpm/displaylink-rpm/releases

2. **Download the appropriate package:**
   - For Fedora 43: `fedora-43-displaylink-X.XX.XX-1.github_evdi.x86_64.rpm`
   - If fc43 not available, use fc42 version (usually works fine)
   - **Important:** Use `x86_64`, NOT `aarch64`

3. **Install the new version:**
   ```bash
   cd ~/Downloads
   sudo dnf install ./fedora-4X-displaylink-*.rpm
   ```

4. **Verify:**
   ```bash
   sudo dkms status
   sudo modinfo evdi
   systemctl status displaylink-driver.service
   ```

5. **Reboot:**
   ```bash
   sudo reboot
   ```

---

## Temporary Workaround: Use Older Kernel

If new DisplayLink driver isn't available yet:

```bash
# List installed kernels
rpm -q kernel

# At boot, press ESC to access GRUB menu
# Select "Advanced options"
# Choose an older kernel that works with your current EVDI version
```

---

## Important Notes

### Kernel Auto-Update Configuration

Your system is now configured to automatically boot the latest kernel:
- Config: `/etc/default/grub` has `GRUB_DEFAULT=0`
- This means: Always boots newest kernel (index 0)

### COPR Repository (Optional)

If you want automatic updates, enable COPR:

```bash
# Enable COPR (when fc43 support is added)
sudo dnf copr enable crashdummy/Displaylink

# Then updates work via:
sudo dnf update
```

**Note:** As of Nov 2025, COPR doesn't have fc43 builds yet.

---

## Checking Compatibility

### Before Kernel Updates

```bash
# Check if new kernel is available
sudo dnf check-update kernel

# Check current EVDI/DisplayLink versions
rpm -q displaylink
sudo dkms status

# Search for newer DisplayLink release
# Visit: https://github.com/displaylink-rpm/displaylink-rpm/releases
```

### After Kernel Updates

```bash
# After updating kernel, check DKMS rebuilt successfully
sudo dkms status

# Should show:
# evdi/X.XX.XX, <kernel-version>, x86_64: installed
```

---

## Troubleshooting

### DKMS Build Fails After Kernel Update

```bash
# Check the build log for errors
cat /var/lib/dkms/evdi/<version>/build/make.log

# Common fix: Update to newer DisplayLink/EVDI version
# Follow "How to Update DisplayLink Driver" above
```

### DisplayLink Service Not Starting

```bash
# Check service status
systemctl status displaylink-driver.service

# Check logs
journalctl -u displaylink-driver.service

# Restart service
sudo systemctl restart displaylink-driver.service
```

### Displays Not Detected

```bash
# Check if evdi module is loaded
lsmod | grep evdi

# Check connected DisplayLink devices
lsusb | grep -i displaylink

# Check display detection
xrandr
```

---

## Summary

**Keep these in mind:**
1. Update kernel regularly: `sudo dnf update`
2. Check DKMS status after kernel updates: `sudo dkms status`
3. If DKMS fails, check for newer DisplayLink release on GitHub
4. Your system auto-boots latest kernel - no manual intervention needed
5. Worst case: Boot older kernel until DisplayLink catches up

**Quick health check command:**
```bash
echo "Kernel: $(uname -r)" && \
echo "DisplayLink: $(rpm -q displaylink)" && \
sudo dkms status | grep evdi && \
systemctl is-active displaylink-driver.service
```
