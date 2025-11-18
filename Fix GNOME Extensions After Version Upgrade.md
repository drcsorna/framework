# Fix GNOME Extensions After Version Upgrade

## Problem
After upgrading Fedora/GNOME, extensions show **ERROR** state even with "Disable version validation" enabled.

## Root Cause
- **Metadata check**: Extension doesn't declare support for new GNOME version
- **Code incompatibility**: Extension uses deprecated/changed APIs

"Disable version validation" only bypasses metadata checks, not code incompatibilities.

---

## Solution Steps

### 1. Identify the Extension
```bash
gnome-extensions list --details | grep -B 2 "State: ERROR"
```

Note the extension's UUID (e.g., `my-extension@example.com`).

### 2. Navigate to Extension Directory
```bash
cd ~/.local/share/gnome-shell/extensions/EXTENSION-UUID
```

### 3. Backup Original Files
```bash
cp extension.js extension.js.backup
cp metadata.json metadata.json.backup
```

### 4. Fix Code Incompatibilities
Check GitHub issues or extension reviews for known fixes.

**Common GNOME 49 fix example:**
```bash
# Replace deprecated API calls
sed -i 's/win\.get_maximized() === Meta\.MaximizeFlags\.BOTH/win.is_maximized()/g' extension.js
```

### 5. Update metadata.json
Add current GNOME version to `shell-version` array:

```bash
# Check current GNOME version
gnome-shell --version

# Add version (e.g., 49) to metadata
sed -i 's/"48"/"48",\n    "49"/g' metadata.json

# Verify changes
cat metadata.json
```

### 6. Restart GNOME Shell
Log out and log back in (or reboot).

### 7. Verify Fix
```bash
gnome-extensions list --details | grep -A 2 "EXTENSION-UUID"
```

Status should be **ACTIVE**.

---

## Restore if Broken
```bash
cd ~/.local/share/gnome-shell/extensions/EXTENSION-UUID
cp extension.js.backup extension.js
cp metadata.json.backup metadata.json
```

Log out and back in.

---

## Finding Fixes
1. **Extension's GitHub**: Check Issues/Pull Requests
2. **GNOME Extensions website**: Read recent user reviews
3. **Search**: `"extension name" gnome VERSION fix`

---

## Common API Changes by Version

### GNOME 49
- `win.get_maximized() === Meta.MaximizeFlags.BOTH` → `win.is_maximized()`

### Future versions
Check GNOME API documentation: https://gjs-docs.gnome.org/

---

## Notes
- Always backup before modifying
- Some extensions may need upstream updates
- Check for official updates before manual fixes
