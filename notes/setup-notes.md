# Setup Notes — Wazuh SIEM Home Lab

Personal notes documenting issues encountered and fixes applied 
during lab setup. Kept for learning reference and transparency.

---

## Environment

```
Host OS      : Windows 10
Hypervisor   : VirtualBox
RAM          : 8GB DDR4 (3GB allocated to Wazuh VM)
Storage      : VMs stored on HDD to preserve SSD space
```

---

## Issue 1 — Wazuh Installer OS Compatibility Error

**Error:**
```
ERROR: The recommended systems are: Red Hat Enterprise Linux 7, 8, 9; 
CentOS 7, 8; Amazon Linux 2; Ubuntu 16.04, 18.04, 20.04, 22.04. 
The current system does not match this list.
```

**Cause:** Wazuh 4.7.5 install script flagged the Ubuntu version.

**Fix:**
```bash
sudo bash wazuh-install.sh -a --ignore-check
```

---

## Issue 2 — Wazuh Indexer Failing to Start (AccessDeniedException)

**Error:**
```
AccessDeniedException: /etc/wazuh-indexer/backup
```

**Cause:** Wrong file permissions on indexer directories 
after installation.

**Fix:**
```bash
sudo mkdir -p /etc/wazuh-indexer/backup
sudo chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/
sudo chown -R wazuh-indexer:wazuh-indexer /var/lib/wazuh-indexer/
sudo chown -R wazuh-indexer:wazuh-indexer /var/log/wazuh-indexer/
sudo chmod 750 /etc/wazuh-indexer/backup
```

**Note:** This fix needs to be reapplied after every reboot 
until services are properly enabled with systemctl enable.

---

## Issue 3 — Low RAM Causing Indexer Crashes

**Symptom:** Wazuh indexer starting then immediately crashing 
with exit code 1.

**Cause:** VM only had 1.9GB RAM. OpenSearch (indexer) requires 
minimum 2GB just for itself.

**Fix:**
- Increased VM RAM to 3GB in VirtualBox settings
- Added 2GB swap space as safety net:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## Issue 4 — Dashboard Not Binding to Port 443

**Symptom:** Dashboard service showing active but nothing 
listening on port 443. Browser showing "site can't be reached".

**Cause:** Two problems found:
1. `opensearch.username` and `opensearch.password` were 
   commented out in config
2. SSL certificate files missing from certs directory

**Fix — Uncomment credentials:**
```bash
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml
# Uncomment and set:
# opensearch.username: "kibanaserver"
# opensearch.password: "YOUR_PASSWORD"
```

---

## Issue 5 — Host-Only Adapter Not Getting IP

**Symptom:** `enp0s8` interface present but showing as DOWN 
with no IP address.

**Cause:** Netplan config was empty after Ubuntu Server install.

**Fix:**
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
```

```bash
sudo netplan apply
```

---

## Issue 6 — Windows Agent Service Name

**Symptom:** `NET START WazuhSvc` returned "service name invalid".

**Cause:** Service name changed in Wazuh 4.7.x.

**Fix:**
```bash
NET START Wazuh
```

---

## Key Lessons Learned

- Allocate minimum 3GB RAM for Wazuh server VM
- Add swap space on RAM-constrained systems
- Service startup order matters: indexer → manager → filebeat → dashboard
- Always verify with `systemctl status` before assuming a service is healthy
