# Tailscale Setup

## Install
```bash
sudo dnf install tailscale
```

## Start & Enable
```bash
sudo systemctl enable --now tailscaled
```

## Authenticate
```bash
tailscale up
```

Visit the URL shown to connect your device.

## Check Status
```bash
tailscale status
```
