# Homelab Series — Part 1: Splunk Enterprise Deployment on a Segmented KVM Lab Network

**Author:** [@aschmeck](https://github.com/aschmeck)  
**Series:** Homelab Security Operations  
**Part:** 1 of ongoing  
**Next:** [Part 2 — Windows Server 2022 Active Directory Setup and Splunk Universal Forwarder Deployment](#)

---

## Overview

This document covers the deployment of a single-node Splunk Enterprise instance within a segmented KVM/QEMU homelab environment running on a Debian 13 host. The lab is designed to simulate a small enterprise network with an isolated attack surface, a pfSense firewall providing segmentation, and Splunk as the central SIEM.

This write-up is intended to be repeatable. All configuration decisions are documented with rationale so that the environment can be reproduced from scratch.

---

## Environment

### Host Machine

| Component | Detail |
|---|---|
| OS | Debian 13 (Trixie) |
| Kernel | 6.12.43+deb13-amd64 |
| CPU | Intel Core i5-10351 (4c/8t) |
| RAM | 16 GB |
| Storage | 500 GB NVMe SSD |
| Hypervisor | QEMU/KVM via libvirt |
| Firewall | nftables (inet filter table) |
| Encryption | LUKS2 full-disk |

### Virtual Machines

| VM | vCPUs | RAM | Role | Network |
|---|---|---|---|---|
| pfsense-firewall | 1 | 1 GB | Network gateway, DHCP, DNS | WAN: default (NAT), LAN: lab-lan |
| ubuntu24.04 | 4 | 6 GB | Splunk Enterprise indexer/search head | lab-lan |
| win2k22 | 2 | 6 GB | Windows Server 2022 DC (Part 2) | lab-lan |
| win11 | 2 | 4 GB | Windows 11 endpoint (Part 2) | lab-lan |
| kali-linux | 1 | 2 GB | Attack platform | default (NAT) |

### Network Topology

Two virtual networks are defined in libvirt:

**default (NAT) — virbr0 — 192.168.122.0/24**  
Provides NAT-based internet access. pfSense WAN and Kali Linux are attached here. libvirt manages DHCP, DNS, and masquerading for this network natively via its own nftables table (`table ip filter`, `table ip nat`).

**lab-lan (isolated) — virbr-lan — 192.168.10.0/24**  
An isolated internal network with no IP address or DHCP configured at the libvirt level. pfSense LAN, Splunk (Ubuntu), win2k22, and win11 are attached here. pfSense owns all DHCP, DNS, and routing decisions for this segment. Internet access for lab-lan is intentionally controlled at the pfSense firewall level to support a vulnerable target environment.

```
Internet
    |
  virbr0 (192.168.122.1) — NAT managed by libvirt
    |
  pfSense WAN (vtnet0 — 192.168.122.84)
    |
  pfSense LAN (vtnet1 — 192.168.10.1)
    |
  virbr-lan (192.168.10.0/24)
    |
    ├── ubuntu24.04 / Splunk (192.168.10.50)
    ├── win2k22 (DHCP from pfSense)
    └── win11 (DHCP from pfSense)

  virbr0 (separate branch)
    └── kali-linux (DHCP from libvirt — independent internet path)
```

---

## Prerequisites

- KVM/QEMU and libvirt installed and operational on the host
- pfSense installed and configured with WAN on `default` network and LAN on `lab-lan`
- pfSense DHCP server active on LAN interface (192.168.10.0/24)
- Ubuntu 24.04 LTS VM installed with a static or DHCP-assigned IP on lab-lan
- Internet access available from within the Ubuntu VM (via pfSense)
- Splunk account at [splunk.com](https://www.splunk.com) for download access

---

## Step 1 — Prepare the Ubuntu VM

### 1.1 Verify System Resources

Before installation, confirm available disk, memory, and OS version meet Splunk's minimum requirements.

```bash
df -h
free -h
cat /etc/os-release
```

**Minimum requirements for this deployment:**

| Resource | Minimum | This Lab |
|---|---|---|
| RAM | 4 GB | 6 GB |
| Disk | 20 GB free | 33 GB free |
| OS | Linux x86_64 | Ubuntu 24.04.4 LTS |

### 1.2 Remove Conflicting Services

If a previous SIEM (e.g., Wazuh) is installed on the same host, it must be fully removed before Splunk installation. Running both simultaneously will cause severe memory contention.

```bash
sudo systemctl stop wazuh-manager wazuh-indexer wazuh-dashboard
sudo systemctl disable wazuh-manager wazuh-indexer wazuh-dashboard
sudo apt remove --purge wazuh-manager wazuh-indexer wazuh-dashboard -y
sudo apt autoremove -y
sudo rm -rf /var/ossec
sudo rm -rf /etc/wazuh-*
```

Verify memory baseline after removal:

```bash
free -h
```

A clean Ubuntu 24.04 system with no SIEM running should have approximately 4+ GB available on a 6 GB VM.

---

## Step 2 — Download Splunk Enterprise

Splunk Enterprise is downloaded directly from Splunk's official download portal. A free Splunk account is required.

1. Navigate to [splunk.com/en_us/download/splunk-enterprise.html](https://www.splunk.com/en_us/download/splunk-enterprise.html)
2. Select **Linux** → **.tgz** for the x86_64 package
3. Copy the wget download link provided on the download page
4. On the Ubuntu VM, download the package:

```bash
wget -O splunk-linux-x86_64.tgz "<paste-official-download-link-here>"
```

> **Note:** Always retrieve the download link directly from Splunk's portal. Do not use third-party mirrors or cached URLs — the link changes with each release and includes a verification token.

---

## Step 3 — Install Splunk Enterprise

### 3.1 Extract to /opt

```bash
sudo tar -xvzf splunk-linux-x86_64.tgz -C /opt
```

### 3.2 Create a Dedicated Service Account

Splunk should not run as root. A dedicated system user with no login shell is created to own the Splunk process.

```bash
sudo useradd -r -s /bin/false splunk
sudo chown -R splunk:splunk /opt/splunk
```

### 3.3 Start Splunk and Accept License

```bash
sudo -u splunk /opt/splunk/bin/splunk start --accept-license
```

During first start, an admin username and password will be prompted. These credentials are used to access the Splunk web UI.

### 3.4 Enable Boot Start

Configure Splunk to start automatically on system boot, running as the splunk user:

```bash
sudo /opt/splunk/bin/splunk enable boot-start -user splunk
```

---

## Step 4 — Verify Web UI Access

From a browser on the Debian host, navigate to:

```
http://192.168.10.50:8000
```

Log in with the admin credentials created during Step 3.3. Confirm the Splunk dashboard loads successfully.

> **Note:** Port 8000 must be reachable from the host. Because the host is not directly on lab-lan, traffic to 192.168.10.50 routes through pfSense. Confirm pfSense LAN firewall rules permit this traffic if the UI is unreachable.

---

## Step 5 — Configure Splunk to Receive Data

### 5.1 Enable Receiving Port

Universal Forwarders send data to the Splunk indexer over TCP port 9997 by default.

In the Splunk web UI:

**Settings → Forwarding and Receiving → Configure Receiving → New Receiving Port**

- Port: `9997`
- Save

### 5.2 Create a Windows Index

A dedicated index is created for Windows Event Log data to keep it isolated from other log sources added in later phases.

**Settings → Indexes → New Index**

| Field | Value |
|---|---|
| Index Name | `windows` |
| Index Data Type | Events |
| All other settings | Default |

---

## Current State

At the completion of this document, the following is in place:

- Splunk Enterprise running on ubuntu24.04 at `192.168.10.50:8000`
- Receiving port 9997 active and listening
- `windows` index created and ready to receive data
- Boot-start configured — Splunk survives reboots
- lab-lan isolated from direct internet access at the host nftables level
- pfSense managing all DHCP, DNS, and controlled internet access for lab-lan

---

## What's Next

**Part 2** will cover:

- Windows Server 2022 Active Directory domain setup on win2k22
- Splunk Universal Forwarder deployment on win2k22 and win11
- Windows Event Log input configuration (Security, System, Application channels)
- Sysmon deployment and configuration for enhanced endpoint telemetry
- Verification of log flow into the `windows` index

---

## Reference

| Resource | URL |
|---|---|
| Splunk Enterprise Download | https://www.splunk.com/en_us/download/splunk-enterprise.html |
| Splunk Universal Forwarder | https://www.splunk.com/en_us/download/universal-forwarder.html |
| Splunk Documentation | https://docs.splunk.com |
| libvirt Networking | https://libvirt.org/formatnetwork.html |
| pfSense Documentation | https://docs.netgate.com/pfsense/en/latest/ |

---

*Part of the [Homelab Security Operations](https://github.com/aschmeck) series by [@aschmeck](https://github.com/aschmeck)*
