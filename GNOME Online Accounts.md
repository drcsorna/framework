# GNOME Online Accounts

## Problem
When attempting to add a Google account in GNOME Settings → Online Accounts, the authentication process failed with:
- "Failed to retrieve credentials from the keyring"
- OAuth flow completed in browser but credentials weren't saved
- "Account Action Required - Failed to sign in" notification after reboot

## Root Cause
The gnome-keyring had conflicting/stale Google OAuth credentials registered, causing new authentication attempts to fail with "already registered" errors in the logs.

## Solution

### 1. Install Seahorse (GNOME password manager)
```bash
sudo dnf install seahorse
```

### 2. Remove conflicting keyring entries
```bash
seahorse
```
- Navigate to "Login" keyring
- Delete all entries labeled "GOA google credentials for identity account_*"

### 3. Clear GNOME Online Accounts configuration
```bash
# Stop the daemon
killall goa-daemon

# Remove all GOA config and cache
rm -rf ~/.config/goa-1.0/
rm -rf ~/.cache/goa-1.0/

# Clear dconf settings
dconf reset -f /org/gnome/online-accounts/
```

### 4. Reboot and re-add account
- Reboot system
- Settings → Online Accounts → Add Google account
- When Firefox prompts "Open GNOME OAuth2 Handler?" → Click "Open GNOME OAuth2 Handler"
- Account should now authenticate successfully

## Prevention
Check the "Always allow accounts.google.com to open links" box in Firefox to avoid the handler prompt in the future.
