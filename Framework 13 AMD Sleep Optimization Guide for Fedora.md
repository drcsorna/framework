# Framework 13 AMD Sleep Optimization Guide for Fedora

**Target System:** Framework Laptop 13 (AMD Ryzen 7040 Series)  
**OS:** Fedora 40/41/42  
**Goal:** Optimize s2idle (Modern Standby) for minimal battery drain during suspend

---

## ⚠️ Important: S3 Deep Sleep is NOT Available on AMD Framework

Unlike Intel models, the Framework 13 AMD **does not support traditional S3/Deep Sleep**. The hardware only supports **s2idle (S0ix/Modern Standby)**. Any guides suggesting BIOS settings to enable S3 or kernel parameters like `mem_sleep_default=deep` will not work and should be ignored.

The proper approach is to **optimize s2idle** for best battery life.

---

## Problem Overview

Your system was experiencing:
- High battery drain during suspend (several % per hour)
- System attempting `deep` sleep, failing, then falling back to `s2idle`
- Incorrect power management configuration
- Old kernel lacking AMD optimizations

---

## What We Fixed

### 1. Removed Incorrect Kernel Parameter

**Issue:** The kernel parameter `mem_sleep_default=deep` was set but doesn't work on AMD Framework.

**Fix:**
```bash
# Remove the useless parameter
sudo grubby --remove-args="mem_sleep_default=deep" --update-kernel=ALL
```

**Result:** System no longer wastes time trying to enter unavailable sleep state.

---

### 2. Fixed Power Management Configuration

**Issue:** TLP was enabled, which conflicts with power-profiles-daemon (the correct power manager for AMD Ryzen 7040).

**Fix:**
```bash
# Disable and mask TLP completely
sudo systemctl disable tlp.service
sudo systemctl mask tlp.service

# Install and enable power-profiles-daemon
sudo dnf install power-profiles-daemon
sudo systemctl enable power-profiles-daemon
sudo systemctl start power-profiles-daemon
```

**Why:** AMD Ryzen 7040 series works best with power-profiles-daemon, NOT TLP. This is Framework's official recommendation.

**Verify it's working:**
```bash
powerprofilesctl list
powerprofilesctl get  # Should show 'balanced'
```

---

### 3. Fixed systemd logind Configuration

**Issue:** `/etc/systemd/logind.conf` was missing the `[Login]` section header, causing diagnostic tools to fail.

**Fix:**
```bash
# Backup the file (saved at /etc/systemd/logind.conf.backup)
sudo cp /etc/systemd/logind.conf /etc/systemd/logind.conf.backup

# Fix the file format
echo "[Login]" | sudo tee /etc/systemd/logind.conf > /dev/null
echo "HoldoffTimeoutSec=0" | sudo tee -a /etc/systemd/logind.conf > /dev/null

# Restart logind
sudo systemctl restart systemd-logind
```

**File should look like:**
```ini
[Login]
HoldoffTimeoutSec=0
```

---

### 4. Updated to Latest Kernel

**Issue:** Running old kernel 6.8.5 which lacked AMD Framework optimizations.

**Fix:**
```bash
# Update kernel
sudo dnf update kernel

# Set the latest kernel as default (if not automatic)
sudo grubby --set-default-index=0

# Verify
sudo grubby --default-kernel

# Reboot
sudo reboot

# After reboot, verify kernel version
uname -r  # Should show 6.17.7 or newer
```

**Why:** Newer kernels have significant improvements for AMD Ryzen 7040:
- Better s2idle support
- USB device power management improvements
- Expansion card power state handling
- AMD PMC driver enhancements

---

## Diagnostic Tools Installed

### AMD s2idle Diagnostic Tool

**Purpose:** Analyze suspend behavior and identify wake sources.

**Installation:**
```bash
# Method 1: Via pip (recommended)
sudo pip install amd-debug-tools --break-system-packages

# Method 2: Download script directly
wget "https://gitlab.freedesktop.org/drm/amd/-/raw/master/scripts/amd_s2idle.py" -O ~/amd_s2idle.py
```

**Dependencies:**
```bash
sudo dnf install python3-systemd
```

**Usage:**
```bash
# Run test (suspends laptop for 10 seconds)
sudo amd-s2idle test

# Or if using downloaded script:
sudo python3 ~/amd_s2idle.py test
```

**What it does:**
- Suspends your system for a configurable duration (default 10 seconds)
- Measures battery drain
- Identifies wake sources
- Checks if hardware sleep states were achieved
- Generates detailed report

---

## Current System State

After all fixes:
- ✅ Kernel: 6.17.7-200.fc42.x86_64
- ✅ Power management: power-profiles-daemon (balanced mode)
- ✅ TLP: Disabled and masked
- ✅ Kernel parameters: Clean (no incorrect sleep parameters)
- ✅ logind.conf: Properly formatted

---

## Files Modified/Created

### System Configuration Changes
- `/etc/systemd/logind.conf` - Fixed formatting
- `/etc/systemd/logind.conf.backup` - **Backup file (can be deleted after verification)**
- Kernel boot parameters - Removed `mem_sleep_default=deep`

### Installed Packages/Tools
1. **power-profiles-daemon** - **KEEP** (essential for AMD power management)
2. **amd-debug-tools** (via pip) - **Can be removed after troubleshooting**
3. **python3-systemd** - **KEEP** (useful system dependency)

### Downloaded Files
- `~/amd_s2idle.py` - **Can be deleted after troubleshooting**
- `~/amd-s2idle-report-*.md` - Test reports, **can be deleted** after review

---

## Cleanup Instructions

### After Problem is Resolved

**Remove diagnostic tools:**
```bash
# Remove pip-installed amd-debug-tools
sudo pip uninstall amd-debug-tools

# Remove downloaded script
rm ~/amd_s2idle.py

# Remove test reports (optional)
rm ~/amd-s2idle-report-*.md
```

**Remove backup file:**
```bash
# After verifying /etc/systemd/logind.conf works correctly
sudo rm /etc/systemd/logind.conf.backup
```

**DO NOT REMOVE:**
- power-profiles-daemon (this is your permanent power manager)
- python3-systemd (useful system dependency)
- Kernel updates (keep all recent kernels for safety)

---

## Next Steps: Further Optimization

### 1. Run Diagnostic Test

```bash
sudo amd-s2idle test
```

**Good results:**
- Hardware sleep: >60%
- Battery drain: <0.5W (approximately 2-4% per 8 hours)

**If battery drain is still high, check:**
- Which USB-C expansion cards are installed
- USB device wake sources
- WiFi power management

### 2. Check Current Sleep Statistics

```bash
# View s2idle statistics
sudo cat /sys/kernel/debug/amd_pmc/s2idle_stats

# Check available sleep states
cat /sys/power/mem_sleep  # Should show: [s2idle]
```

### 3. Identify Problem Devices (if needed)

```bash
# List devices that can wake the system
cat /proc/acpi/wakeup

# Check USB devices
lsusb

# Check PCI devices
lspci | grep -i "usb\|sd\|card"
```

### 4. Expansion Card Optimization

**Known issues:**
- Some USB-C expansion cards prevent deep sleep states
- SD card readers can cause wake events
- DisplayPort/HDMI cards may stay active

**Solutions:**
- Remove unused expansion cards during suspend
- Use USB autosuspend for expansion cards
- Update to latest BIOS (check Framework support)

---

## Expected Battery Drain Benchmarks

**Good s2idle performance:**
- 2-4% over 8 hours (~0.3-0.5W)
- Hardware sleep >70%

**Acceptable s2idle performance:**
- 5-8% over 8 hours (~0.7-1.0W)
- Hardware sleep >50%

**Poor s2idle performance (needs fixing):**
- >10% over 8 hours (>1.5W)
- Hardware sleep <30%

---

## Troubleshooting

### System Won't Suspend
```bash
# Check for inhibitors
systemd-inhibit --list

# Check what's preventing sleep
sudo journalctl -b -u systemd-logind | grep -i suspend
```

### System Wakes Immediately
```bash
# Check wake sources
cat /sys/kernel/debug/wakeup_sources

# Run diagnostic to identify culprit
sudo amd-s2idle test
```

### High Battery Drain During Suspend
1. Run `sudo amd-s2idle test` to measure actual drain
2. Check report for devices not entering proper power states
3. Remove/disable problematic expansion cards
4. Check for BIOS updates from Framework

---

## Important Commands Reference

```bash
# Check power profile
powerprofilesctl get

# Set power profile
powerprofilesctl set power-saver  # For battery
powerprofilesctl set balanced     # Default
powerprofilesctl set performance  # For AC power

# Check current kernel
uname -r

# List installed kernels
rpm -q kernel

# Check s2idle stats
sudo cat /sys/kernel/debug/amd_pmc/s2idle_stats

# Run sleep diagnostic
sudo amd-s2idle test
```

---

## Resources

- **Framework Community:** https://community.frame.work/
- **AMD s2idle documentation:** https://gitlab.freedesktop.org/drm/amd/-/blob/master/scripts/amd_s2idle.py
- **Framework Linux Guide:** https://knowledgebase.frame.work/linux
- **Arch Wiki (excellent reference):** https://wiki.archlinux.org/title/Framework_Laptop_13

---

## Summary

**What Was Wrong:**
1. Incorrect kernel parameter trying to enable unavailable S3 sleep
2. TLP conflicting with proper AMD power management
3. Old kernel lacking AMD optimizations
4. Misconfigured logind.conf preventing diagnostics

**What Was Fixed:**
1. Removed incorrect kernel parameters
2. Switched from TLP to power-profiles-daemon
3. Updated to latest kernel (6.17.7+)
4. Fixed system configuration files

**Result:**
System now properly uses s2idle with optimized power management. Further optimization depends on expansion card configuration and specific usage patterns.

**Critical Takeaway:**
Framework 13 AMD uses **s2idle ONLY** - there is no S3 deep sleep option. Focus on optimizing s2idle, not trying to enable non-existent S3 support.
