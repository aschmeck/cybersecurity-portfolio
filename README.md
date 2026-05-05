# Cybersecurity Portfolio

**Aidan Schmeckpeper** · [@aschmeck](https://github.com/aschmeck)
**B.S. Cybersecurity — Kennesaw State University**
**CompTIA Security+**

---

## About This Portfolio

This repository documents a deliberate, hands-on progression through cybersecurity concepts and practice. The work here spans detection engineering, security operations, vulnerability assessment, offensive security fundamentals, endpoint hardening, and security policy — built and documented in a self-hosted homelab environment running on physical hardware.

Every project in this portfolio was conducted in a self-built environment — not a guided lab or pre-configured sandbox. The infrastructure was designed, deployed, and troubleshot from scratch, which means the failures documented here are real ones: unexpected firewall conflicts, corrupted installers, misconfigured audit policies. The goal throughout has been to build genuine operational understanding — not just familiarity with tools, but the ability to reason about why they work, where they fail, and what an attacker or defender would do next.

Each lab write-up includes an Attacker/Defender Perspective section connecting both sides of the scenario, with MITRE ATT&CK references where applicable. This reflects a core conviction: that effective defenders need to understand how attackers think, and that understanding the attacker's perspective makes every detection, rule, and response decision more deliberate.

---

## Environment

All lab work was conducted against a self-built homelab running on a Debian 13 daily-driver workstation. The environment includes a KVM/QEMU hypervisor, pfSense for network segmentation, a Windows Server 2022 domain controller, a domain-joined Windows 11 workstation, a Splunk Enterprise SIEM, and a Kali Linux attacker VM — all on an isolated lab network segment.

Full environment documentation is in [`homelab/homelab-configuration/`](homelab/homelab-configuration/README.md).

---

## Repository Structure

### `homelab/`
The core of the portfolio. Documents the build-out and use of the `schmeck.lab` homelab environment, organized into two sections:

**`homelab-configuration/`** — Environment documentation, network architecture, and the detection infrastructure build-out (SIEM deployment, Active Directory, Universal Forwarder, Sysmon).

**`vulnerability-assessment/`** — Structured vulnerability assessment of the lab infrastructure conducted from Kali Linux, covering network reconnaissance, defender visibility validation, and host-by-host scanning with Greenbone Vulnerability Manager.

---

### `projects/`
Standalone projects that support or extend the homelab work.

**`linux-migration/`** — Documents the migration of the host workstation from Windows 11 to Debian 13, including full-disk encryption, nftables firewall design, firewall conflict resolution, and a systemd-based maintenance and auditing suite. Treated with the rigor of an enterprise endpoint hardening effort.

**`wazuh-alert-optimization/`** — Documents the operationalization of Windows failed logon alerting using Wazuh and Slack. Demonstrates the gap between detection existing and detection being actionable — and how to close it.

---

### `policy-and-governance/`
A complete information security policy library developed as the capstone project for the B.S. Cybersecurity program at Kennesaw State University. Covers the full EISP → ISSP → SYSSP policy hierarchy for a fictional organization, including incident response, disaster recovery, contingency planning, IDS policy, firewall and VPN policy, and more.

See [`policy-and-governance/README.md`](policy-and-governance/README.md) for context and relevance to security operations work.

---

### `learning-notes/`
Notes and write-ups from TryHackMe rooms and training exercises. Not portfolio pieces — reference material for ongoing skill development.

---

### `archive/`
Deprecated content from an earlier VirtualBox-based lab iteration. Retained for continuity.

---

## Certifications and Background

- **CompTIA Security+**
- **B.S. Cybersecurity** — Kennesaw State University
- Ongoing practice via TryHackMe and HackTheBox
