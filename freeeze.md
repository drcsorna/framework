# Framework 13 AMD GPU Freeze Fix (Fedora 43)

## Problem

System randomly freezes during normal use with these symptoms:
- Screen stays on but nothing responds
- Mouse doesn't move
- Keyboard input ignored
- Requires hard reboot (power button)

## Root Cause

**DMCUB (Display MicroController) firmware crashes** in the AMD GPU driver on kernel 6.17.11.

Log evidence:
```
amdgpu 0000:c1:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error
amdgpu 0000:c1:00.0: [drm] *ERROR* [CRTC:80:crtc-0] flip_done timed out
```

This is caused by PSR (Panel Self Refresh) feature conflicting with the Framework 13 AMD display.

## The Fix

Disable PSR via kernel parameter:

```bash
# Add the PSR disable parameter to all kernels
sudo grubby --update-kernel=ALL --args="amdgpu.dcdebugmask=0x10"

# Verify it was added
sudo grubby --info=ALL | grep args

# Reboot to activate
sudo reboot
```

## Verification

**After reboot, confirm the parameter is active:**
```bash
cat /proc/cmdline | grep dcdebugmask
```

Expected output should include: `amdgpu.dcdebugmask=0x10`

**Check for DMCUB errors (should be zero):**
```bash
journalctl -b | grep -i "DMCUB error"
```

## System Configuration

- **Laptop:** Framework Laptop 13 (AMD Ryzen 7040Series)
- **OS:** Fedora Linux 43 (Workstation Edition)
- **Kernel:** 6.17.11-300.fc43.x86_64
- **GPU:** AMD Radeon 780M Graphics (amdgpu driver)
- **BIOS:** 03.17 (latest)

## Expected Results

- **Before fix:** Random freezes during normal use, multiple DMCUB errors in logs
- **After fix:** No freezes, zero DMCUB errors

## Notes

- This fix **does not conflict** with sleep optimization fixes
- PSR is a power-saving feature, but disabling it has minimal battery impact
- This is a known issue with Framework 13 AMD + Fedora 43 + kernel 6.17.x
- Alternative solutions: kernel upgrade to 6.12.x or downgrade linux-firmware

## Related Documentation

This fix is separate from and compatible with the sleep optimization documented in the Framework 13 AMD Sleep Optimization Guide.

---

**Fix applied:** December 27, 2025  
**Status:** Active and working ✅
