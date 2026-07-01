# Homelab

This directory documents the infrastructure behind the `schmeck.lab` cybersecurity portfolio environment.

The homelab is the supporting environment used for Active Directory, SIEM, vulnerability assessment, detection engineering, and SOC workflow projects. Major project write-ups now live under [`projects/`](../projects/README.md), while this directory focuses on how the lab itself is built and organized.

---

## Contents

### [`homelab-configuration/`](homelab-configuration/README.md)

Documents the design and build-out of the lab environment: host platform, VM inventory, virtual networking, pfSense gatewaying, Active Directory domain structure, SIEM role, and telemetry architecture.

### [`homelab-configuration/homelab-architecture/`](homelab-configuration/homelab-architecture/README.md)

Contains architecture notes and the current `schmeck.lab` network diagram.

---

## Current Lab Role

The `schmeck.lab` environment supports the following portfolio projects:

- [Active Directory SIEM Build](../projects/active-directory-siem/README.md)
- [SOC Dashboard and Alerting Pipeline](../projects/soc-dashboard-alerting/README.md)
- [Vulnerability Assessment and Defender Visibility Series](../projects/vulnerability-assessment/README.md)
- [Debian 13 Workstation Migration and Hardening](../projects/linux-migration-hardening/README.md)

---

## Environment Summary

| Component | Role |
|---|---|
| Debian 13 host | Daily-driver workstation and KVM/QEMU/libvirt hypervisor |
| pfSense | Lab gateway and firewall |
| Kali Linux | Attacker / offensive tooling / GVM |
| Ubuntu 24.04 | Splunk Enterprise SIEM |
| Windows Server 2022 | DC01 domain controller for `schmeck.lab` |
| Windows 11 | Domain-joined workstation |

The homelab is intentionally small and resource-constrained. VMs are started and stopped as needed, with pfSense normally acting as the persistent gateway for the isolated lab LAN.

---

## Network Summary

| Network | Purpose |
|---|---|
| `virbr0` | libvirt default NAT / pfSense WAN side |
| `virbr-lan` | isolated lab LAN for VM-to-VM traffic |
| `192.168.10.0/24` | internal lab subnet |

The lab is isolated from the host's normal production network. pfSense routes lab traffic and provides gateway services for the internal subnet.

---

## Documentation Boundary

This directory explains the environment.

Project case studies live under [`projects/`](../projects/README.md). This separation keeps the portfolio organized around completed security work while preserving the infrastructure details needed to understand and rebuild the lab.
