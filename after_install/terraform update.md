# Terraform Installation on Fedora - Maintenance Guide

**Date:** 2025-11-27  
**System:** Framework Laptop, Fedora 43 (Rawhide)  
**Issue:** HashiCorp doesn't publish packages for bleeding-edge Fedora versions

---

## Current Configuration

```bash
# Repository file
/etc/yum.repos.d/hashicorp.repo

# Contains hardcoded version
baseurl=https://rpm.releases.hashicorp.com/fedora/41/$basearch/stable
#                                                   ^^ Using Fedora 41 packages on Fedora 43
```

**Installed with:**
```bash
sudo curl -o /etc/yum.repos.d/hashicorp.repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo sed -i 's/$releasever/41/g' /etc/yum.repos.d/hashicorp.repo
sudo dnf makecache
sudo dnf install -y terraform
```

---

## The Problem

1. **DNF5 syntax changed:** Fedora 41+ uses `addrepo --from-repofile=` instead of `--add-repo`
2. **Bleeding-edge Fedora:** HashiCorp publishes packages 1-2 releases behind latest Fedora
3. **Hardcoded version:** Repository file has static version number, doesn't auto-update

---

## Symptoms After Fedora Upgrade

```bash
$ sudo dnf upgrade terraform
Error: Failed to download metadata for repo 'hashicorp'
404 - Not Found
```

**Cause:** Repository still points to old Fedora version (e.g., `41/`) but system upgraded to Fedora 44

---

## Fix: Update Repository Version

### Step 1: Check Available Packages
Visit: https://rpm.releases.hashicorp.com/fedora/

Look for directories like:
```
39/
40/
41/
42/  <- If your system is Fedora 44, try 42 first
```

### Step 2: Update Repository File

```bash
# Check current version in repo file
grep baseurl /etc/yum.repos.d/hashicorp.repo

# Update to newer version (e.g., 41 → 42)
sudo sed -i 's/41/42/g' /etc/yum.repos.d/hashicorp.repo

# Refresh and upgrade
sudo dnf makecache
sudo dnf upgrade terraform
terraform version
```

### Step 3: If New Version Unavailable

Use most recent available version:
```bash
# Example: Fedora 45 system, but only 42 packages exist
sudo sed -i 's/OLD_VERSION/42/g' /etc/yum.repos.d/hashicorp.repo
sudo dnf makecache
sudo dnf upgrade terraform
```

---

## Documentation Options

### Option 1: README.md (Project-Level)
Add section to `homelab-infra/README.md`:
```markdown
## Prerequisites
- Terraform installed using Fedora 41 packages
- After Fedora upgrade: Update `/etc/yum.repos.d/hashicorp.repo`
```

### Option 2: SYSTEM_SETUP.md (Comprehensive)
Create dedicated file documenting all Framework laptop setup steps:
- Terraform installation and maintenance
- Ansible setup
- SSH keys
- Other manual configurations

### Option 3: Calendar Reminder
Set reminder when upgrading Fedora:
- Check https://rpm.releases.hashicorp.com/fedora/
- Update hashicorp.repo
- Test `terraform version`

### Option 4: Automated Check (.bashrc)
Add function to check repo version mismatch on login:
```bash
check_terraform_repo() {
    REPO_VER=$(grep -oP '(?<=fedora/)\d+' /etc/yum.repos.d/hashicorp.repo | head -1)
    SYSTEM_VER=$(rpm -E %fedora)
    [ "$REPO_VER" != "$SYSTEM_VER" ] && echo "⚠️  Terraform repo using Fedora $REPO_VER (system is $SYSTEM_VER)"
}
```

---

## Quick Reference

**Check current setup:**
```bash
cat /etc/yum.repos.d/hashicorp.repo | grep baseurl
rpm -E %fedora  # Shows system Fedora version
terraform version
```

**Update after Fedora upgrade:**
```bash
# 1. Check available versions
curl -s https://rpm.releases.hashicorp.com/fedora/ | grep -oP '\d+/' | sort -V

# 2. Update repo (replace OLD with NEW version)
sudo sed -i 's/OLD/NEW/g' /etc/yum.repos.d/hashicorp.repo

# 3. Upgrade Terraform
sudo dnf makecache && sudo dnf upgrade terraform
```

**Verify installation:**
```bash
terraform version
terraform --help
```

---

## Alternative: Manual Binary Installation

If repository issues persist:
```bash
# Download latest release
TER_VER=$(curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest | grep tag_name | cut -d'"' -f4 | tr -d 'v')
wget https://releases.hashicorp.com/terraform/${TER_VER}/terraform_${TER_VER}_linux_amd64.zip

# Install
unzip terraform_${TER_VER}_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform version
```

**Trade-off:** No automatic updates, manual maintenance required

---

## Future AI Context

**When helping with this issue:**
1. User is on Fedora 43 (Rawhide/bleeding-edge)
2. HashiCorp repos lag behind latest Fedora by 1-2 versions
3. `/etc/yum.repos.d/hashicorp.repo` has hardcoded version number
4. DNF5 syntax is `addrepo --from-repofile=`, not `--add-repo`
5. Check https://rpm.releases.hashicorp.com/fedora/ for available versions
6. Solution is always: update hardcoded version in repo file to latest available

**Don't suggest:**
- Disabling HashiCorp repo (breaks updates)
- Downgrading Fedora
- Complex workarounds

**Do suggest:**
- Checking available versions first
- Using most recent available Fedora version's packages
- Manual binary installation as last resort
