# Domain Proxy — Installation Guide

## Overview

`install-dp.sh` is a three-step installer for the CBRS Domain Proxy (DP).

| Step | Action |
|------|--------|
| 1 | Extract the DP installation package (`.zip`) |
| 2 | Configure `http_proxy` / `https_proxy` environment variables |
| 3 | Register a systemd service that runs `reset.sh` at boot |

---

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| OS | Linux with systemd |
| Privileges | `sudo` / root required for steps 2 and 3 |
| Tools | `unzip`, `bash`, `systemctl` must be installed |
| Package | `Install_SXGDPV100*.zip` (the DP release package) |

---

## Usage

```bash
sudo bash install-dp.sh [ZIP_FILE] [INSTALL_DIR]
```

| Argument | Default | Description |
|----------|---------|-------------|
| `ZIP_FILE` | `Install_SXGDPV100.zip` (current directory) | Path to the DP zip package |
| `INSTALL_DIR` | `/home/test/DP` | Directory where the DP will be installed |

### Examples

```bash
# Use default package name and install to /home/test/DP
sudo bash install-dp.sh Install_SXGDPV100R100C037_20260424.zip

# Specify both zip file and install directory
sudo bash install-dp.sh /tmp/Install_SXGDPV100R100C037_20260424.zip /home/test/DP
```

---

## Step-by-Step Details

### Step 1 — Extract Installation Package

The script extracts the zip package into the install directory using `unzip -o`.  
Existing files will be overwritten.

```
[INFO]  Extracting Install_SXGDPV100R100C037_20260424.zip → /home/test/DP ...
[OK]    Extraction complete: /home/test/DP
```

### Step 2 — Configure Proxy (Optional)

The script prompts for proxy addresses interactively.  
Press **Enter** to skip if no proxy is required.

```
Enter proxy address (e.g. http://proxy.example.com:3128).
Press Enter to skip proxy configuration.
  http_proxy  : http://proxy.example.com:3128
  https_proxy [http://proxy.example.com:3128]:
```

Proxy variables are written to `/etc/profile.d/dp-proxy.sh` and apply to all new login sessions.  
To apply immediately in the current session:

```bash
source /etc/profile.d/dp-proxy.sh
```

### Step 3 — Configure Auto-Start Service

A systemd service unit is created at `/etc/systemd/system/domain-proxy.service`.  
The service runs `bash /home/test/DP/reset.sh` at boot after the network is online.

At the end of this step, the script asks whether to start the service immediately:

```
Start the Domain Proxy service now? [Y/n]:
```

---

## Service Management

After installation, use standard `systemctl` commands to manage the DP service:

```bash
# Check service status
systemctl status domain-proxy

# Start / stop / restart
systemctl start   domain-proxy
systemctl stop    domain-proxy
systemctl restart domain-proxy

# Enable / disable auto-start at boot
systemctl enable  domain-proxy
systemctl disable domain-proxy

# Follow live service logs
journalctl -u domain-proxy -f

# Show last 50 log lines
journalctl -u domain-proxy -n 50
```

---

## File Locations

| File | Purpose |
|------|---------|
| `/home/test/DP/reset.sh` | DP startup script (executed by the service) |
| `/etc/systemd/system/domain-proxy.service` | systemd service unit |
| `/etc/profile.d/dp-proxy.sh` | Proxy environment variables (if configured) |
| `/home/test/DP/logs/allLog/info.log` | DP runtime log |

---

## Troubleshooting

### Service fails to start

```bash
journalctl -u domain-proxy -n 100
```

Check that `reset.sh` is executable and all DP files are present under `/home/test/DP/`.

### Proxy not applied to service

The `EnvironmentFile` directive in the service unit loads `/etc/profile.d/dp-proxy.sh` automatically.  
If the proxy was configured after service installation, reload the daemon and restart:

```bash
systemctl daemon-reload
systemctl restart domain-proxy
```

### Re-running the installer

The installer is safe to re-run. It will:
- Overwrite extracted files (`unzip -o`)
- Overwrite the proxy config file
- Overwrite the service unit and re-enable it

---

## Uninstall

```bash
systemctl stop    domain-proxy
systemctl disable domain-proxy
rm -f /etc/systemd/system/domain-proxy.service
rm -f /etc/profile.d/dp-proxy.sh
systemctl daemon-reload
```

The install directory `/home/test/DP` and its contents are **not** removed automatically.
