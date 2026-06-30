# Homelab Configuration — schmeck.lab

**Author:** [@aschmeck](https://github.com/aschmeck)

---

## Overview

This document describes the physical and virtual infrastructure underlying all lab work in this portfolio. The environment is built on a personal daily-driver workstation running Debian 13 — not a dedicated lab box — with KVM/QEMU virtualization hosting a segmented network of five VMs. The lab is intentionally designed to mirror the structure of a small enterprise environment: a domain controller, a domain-joined workstation, a SIEM, a firewall, and an attacker machine operating on an isolated network segment.

Every lab in this portfolio was conducted against this environment.

---

## Physical Host

| Attribute | Detail |
|---|---|
| OS | Debian 13 |
| Hypervisor | KVM/QEMU (libvirt) |
| Disk Encryption | LUKS2 |
| Firewall | nftables (deny-by-default) |
| Role | Daily-driver workstation and lab host |

The host machine runs nftables as its primary firewall with a strict deny-by-default policy. Full-disk encryption via LUKS2 is active. The lab environment runs on top of the daily-driver OS — VMs are started and stopped as needed rather than running continuously.

The host configuration and hardening are documented separately in the [Linux Migration project](../../projects/linux-migration-hardening/README.md).

---

## Network Topology

Two virtual networks are managed by libvirt:

| Network | Interface | Type | Purpose |
|---|---|---|---|
| Default NAT | virbr0 | NAT | libvirt default — not used for lab traffic |
| Lab LAN | virbr-lan | Isolated | Lab segment — all VM-to-VM traffic |

pfSense sits between the host and the lab LAN segment, handling WAN connectivity and routing for lab VMs. The isolated `virbr-lan` network ensures lab traffic does not interact with the host's production network interfaces.

---

## VM Inventory

| VM | IP Address | OS | Role |
|---|---|---|---|
| pfSense | — | pfSense | WAN gateway and router for lab segment |
| Kali Linux | 192.168.10.55 | Kali Linux | Attacker / offensive tooling |
| Ubuntu / Splunk | 192.168.10.50 | Ubuntu 24.04 | SIEM — Splunk Enterprise |
| DC01 | 192.168.10.100 | Windows Server 2022 | Domain controller — schmeck.lab |
| WIN11 | 192.168.10.54 | Windows 11 Pro | Domain-joined workstation |

---

## Active Directory Domain

| Attribute | Detail |
|---|---|
| Domain Name | schmeck.lab |
| NetBIOS Name | SCHMECK |
| Forest / Domain Mode | Windows Server 2016 |
| Domain Controller | DC01 — 192.168.10.100 |

**Organizational Units:**
- `Corp-Users` — domain user accounts
- `Corp-Groups` — security groups
- `Workstations` — domain-joined endpoints

**Domain Accounts:**

| Username | Role |
|---|---|
| ssummers | Domain Administrator |
| wworthington | Standard user |
| efrost | Standard user |

Audit policy is configured on DC01 to capture key security subcategories including logon events, account management, object access, and policy changes. Splunk Universal Forwarder is installed on DC01 and WIN11, forwarding Windows Event Logs to Splunk Enterprise on port 9997.

---

## SIEM Configuration

The lab originally ran Wazuh as its SIEM, used during the initial alert operationalization work documented in the Wazuh Failed Logon Alerting project. The environment was subsequently migrated to Splunk Enterprise, which is the active SIEM for all current and future lab work. That migration is documented in the Splunk Homelab series under `services/`.

Splunk Enterprise runs on Ubuntu at `192.168.10.50`. A dedicated `windows` index receives forwarded events from both DC01 and WIN11. Splunk is configured for boot-start on the Ubuntu host and is brought online during active detection work. The Universal Forwarder on each Windows host queues events locally while Splunk is offline, ensuring no telemetry is lost between sessions.


