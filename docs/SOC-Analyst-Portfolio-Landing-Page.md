# SOC Analyst Portfolio — Start Here

**Aidan Schmeckpeper** · [GitHub: @aschmeck](https://github.com/aschmeck)  
**B.S. Cybersecurity — Kennesaw State University** · **CompTIA Security+**

---

## Why this page exists

This page is a focused entry point into my cybersecurity portfolio for SOC Analyst, Blue Team, and entry-level security operations roles.

The full repository contains several projects, notes, archived labs, and supporting artifacts. This page highlights the work most relevant to security monitoring, detection engineering fundamentals, vulnerability management, incident-style investigation, and defensive operations.

The short version: I built a self-hosted Active Directory homelab, instrumented it with Splunk, generated attacker-style activity from Kali Linux, investigated what the SIEM could and could not see, and documented the operational gaps I found along the way.

---

## Core environment

All primary portfolio work was completed in a self-built homelab, not a guided cloud sandbox or pre-configured training VM.

| Component | Purpose |
|---|---|
| Debian 13 host | Daily-driver workstation and KVM/QEMU hypervisor |
| pfSense | Lab gateway, routing, DHCP, and network segmentation |
| Windows Server 2022 | Domain controller for `schmeck.lab` |
| Windows 11 | Domain-joined workstation |
| Ubuntu 24.04 | Splunk Enterprise SIEM server |
| Kali Linux | Authorized attacker and scanning platform |
| Splunk Universal Forwarder | Windows Event Log and Sysmon forwarding |
| Sysmon | Endpoint telemetry for process and network activity |
| GVM / Greenbone | Vulnerability scanning and host assessment |

[View full homelab documentation](homelab/homelab-configuration/README.md)

---

## Best review path

For a recruiter, hiring manager, or analyst reviewing this portfolio, I recommend starting with these three areas.

### 1. SOC Dashboard and Alerting

[Lab 4 — SOC Dashboard and Alerting](homelab/homelab-configuration/lab4-soc-dashboard-alerting.md)

This is the strongest direct SOC Analyst piece in the portfolio.

What it demonstrates:

- Built a Splunk SOC dashboard from Windows Event Log and Sysmon telemetry
- Created dashboard panels for failed logons, lockouts, privileged logons, scan indicators, process execution, outbound connections, and high-value events
- Configured scheduled alerts with Slack delivery for repeated failed logons, account lockouts, and Domain Admin logon activity from a workstation
- Investigated an unplanned finding: 1,500+ failed logons surfaced on first dashboard load
- Identified SIEM scheduler and ingestion backlog issues caused by Splunk queue behavior
- Documented alert tuning decisions and false positive patterns

Why it matters:

A SIEM that only stores logs is not operationally useful. This project shows the work of turning telemetry into triage visibility, alert logic, and investigation workflow.

---

### 2. Vulnerability Assessment Series

[Start with the Vulnerability Assessment Conclusion](homelab/vulnerability-assessment/va-lab-conclusion.md)

This series documents reconnaissance, unauthenticated scanning, authenticated scanning, SIEM visibility, and attacker/defender interpretation across the lab environment.

Recommended order:

1. [Part 1 — Initial Reconnaissance and Defender Visibility Gap](homelab/vulnerability-assessment/part1-reconnaissance/va-lab-part1.md)
2. [Part 2 — Unauthenticated GVM Scan: DC01 and WIN11](homelab/vulnerability-assessment/part2-unauthenticated-scan/va-lab-part2.md)
3. [Part 3 — Authenticated Scanning](homelab/vulnerability-assessment/part3-authenticated-scanning/va-lab-part3.md)
4. [Part 4 — Authenticated Linux Scanning](homelab/vulnerability-assessment/part4-authenticated-linux-scanning/va-lab-part4.md)
5. [Conclusion — Attack Surface and Detection Lessons](homelab/vulnerability-assessment/va-lab-conclusion.md)

What it demonstrates:

- Ran Nmap reconnaissance against Windows and Linux lab hosts
- Identified that Windows Filtering Platform auditing was not enabled, making scan traffic invisible to Splunk
- Remediated the visibility gap and validated scan detection through Splunk searches
- Ran unauthenticated and authenticated GVM scans against DC01, WIN11, and the Splunk VM
- Compared what attackers learn before and after obtaining credentials
- Treated the SIEM itself as an attack surface requiring assessment and hardening

Why it matters:

This is not just vulnerability scanning. It connects vulnerability management to detection engineering: what the attacker sees, what the defender logs, what the SIEM misses, and what needs to be fixed.

---

### 3. Splunk Homelab Build Series

This series documents the infrastructure behind the SOC and vulnerability work.

Recommended order:

1. [Part 1 — Splunk Enterprise Deployment on a Segmented KVM Lab Network](homelab/homelab-configuration/part1-splunk-deployment/splunk-homelab-part1.md)
2. [Part 2 — Active Directory Deployment and Splunk Universal Forwarder Configuration](homelab/homelab-configuration/part2-active-directory/splunk-homelab-part2.md)
3. [Part 3 — Windows 11 Endpoint Onboarding and Sysmon Deployment](homelab/homelab-configuration/part3-windows-endpoint-sysmon/splunk-homelab-part3.md)

What it demonstrates:

- Built a segmented KVM/QEMU lab network with pfSense routing and isolation
- Deployed Splunk Enterprise on Ubuntu 24.04
- Created a Windows Server 2022 Active Directory domain
- Configured Windows audit policy for security monitoring
- Installed Splunk Universal Forwarder on Windows hosts
- Deployed Sysmon using a maintained community configuration
- Troubleshot forwarding, access permissions, and time range issues in Splunk

Why it matters:

A lot of portfolios show screenshots of tools already running. This series shows the actual build, configuration, breakage, and troubleshooting required to make the tools useful.

---

## Additional project: Debian workstation migration and hardening

[Debian 13 Workstation Migration and Hardening](projects/linux-migration/README.md)

This project documents the migration of my daily-driver workstation from Windows 11 to Debian 13 with security hardening applied as if it were an enterprise endpoint.

Highlights:

- LUKS2 full-disk encryption
- Secure Boot validation
- nftables deny-by-default firewall policy
- SSH restricted to localhost
- Investigation of a real `firewalld` and nftables rule conflict that caused port 22 to remain externally visible
- Lynis, ClamAV, sysctl hardening, and automated systemd maintenance timers

Why it matters:

This project shows endpoint hardening, Linux administration, troubleshooting, and evidence-driven documentation outside the Windows/Splunk lab environment.

---

## Skills demonstrated

| Area | Evidence in portfolio |
|---|---|
| SOC triage | Splunk dashboard, failed logon investigation, alert tuning |
| SIEM operations | Splunk deployment, forwarder configuration, dashboarding, alerting |
| Detection engineering fundamentals | WFP visibility gap, scan behavior detection, Sysmon telemetry |
| Vulnerability management | Nmap, GVM, unauthenticated vs. authenticated scan analysis |
| Windows security | AD DS, audit policy, Windows Event IDs, Sysmon, domain authentication |
| Linux administration | Debian hardening, nftables, systemd timers, OpenSSH hardening |
| Network security | pfSense, segmented lab network, firewall behavior, RPC/DNS exposure |
| Incident-style documentation | Findings, root cause, remediation, attacker/defender perspective |

---

## What makes this portfolio different

This portfolio is built around investigation rather than tool collection.

Each major project asks the same practical questions:

- What happened?
- What evidence supports that conclusion?
- What could an attacker do with this information?
- What would a defender see?
- What did the tooling miss?
- What should be remediated or tuned?

The result is a portfolio focused on operational reasoning. Tools matter, but the real value is knowing what the output means, when it is wrong, and what to do next.

---

## Full repository

[View the complete cybersecurity portfolio](README.md)

The full repository includes additional project documentation, policy and governance work, learning notes, and archived lab material.
