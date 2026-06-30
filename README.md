# Cybersecurity Portfolio

**Aidan Schmeckpeper** · [@aschmeck](https://github.com/aschmeck)  
**B.S. Cybersecurity — Kennesaw State University**  
**CompTIA Security+**

---

## About This Portfolio

This repository documents my hands-on progression through cybersecurity operations, detection engineering, vulnerability assessment, incident investigation, endpoint hardening, and security policy work.

The projects here are built around one central question:

> **How would I investigate, validate, or defend against this in a real environment?**

Most of the work was conducted in a self-built homelab running on physical hardware, not in a guided lab or preconfigured sandbox. The environment was designed, deployed, broken, repaired, scanned, monitored, and documented from scratch. That includes the useful failures: firewall conflicts, broken log visibility, scheduler backlogs, scan authentication issues, misconfigured audit policy, and noisy alerts that needed tuning.

The goal of this portfolio is not just to show tool familiarity. It is to show operational reasoning: what happened, what evidence was collected, what the data meant, what failed, what was fixed, and what a defender or attacker would do next.

---

## Featured Projects

### [SOC Dashboard and Alerting Pipeline](projects/soc-dashboard-alerting/)

Built an operational Splunk SOC dashboard across a self-built Active Directory homelab. The dashboard surfaces authentication anomalies, account lockouts, privileged logons, suspicious process execution, network indicators, and high-value security events. Scheduled alerts were configured with Slack delivery for repeated failed logons, account lockouts, and Domain Admin account activity from a workstation.

**Skills demonstrated:** Splunk, SPL, Windows Event Logs, Sysmon telemetry, alert design, dashboard design, Slack alert routing, false positive analysis, SOC triage workflow.

---

### [Vulnerability Assessment and Defender Visibility Series](projects/vulnerability-assessment/)

Conducted a multi-part vulnerability assessment against the `schmeck.lab` environment using Nmap and Greenbone Vulnerability Manager. The series compares unauthenticated and authenticated scan results, documents host-by-host attack surface, validates what scan activity looks like in Splunk, and identifies a Windows Filtering Platform audit gap that initially made network scans invisible to the SIEM.

**Skills demonstrated:** Nmap, GVM/OpenVAS, authenticated scanning, vulnerability analysis, Windows Filtering Platform logging, Splunk validation, attack surface analysis, remediation planning.

---

### [Splunk + Active Directory SIEM Build](projects/active-directory-siem/)

Built the telemetry foundation for the homelab: Splunk Enterprise on Ubuntu, Windows Server 2022 Active Directory, Splunk Universal Forwarders, Windows Event Log ingestion, and Sysmon deployment on both the domain controller and workstation. This project establishes the logging and visibility required for later detection and attack simulation labs.

**Skills demonstrated:** Splunk Enterprise, Active Directory, Windows Server, Universal Forwarder configuration, Sysmon, log onboarding, audit policy configuration, endpoint telemetry.

---

### [OSINT Smishing Investigation](projects/osint-smishing-investigation/)

Investigated a real-world smishing attempt using OSINT and browser-based analysis. The project documents the suspicious message, infrastructure, website behavior, indicators, investigative limits, and the type of evidence that would support escalation or blocking in a real SOC workflow.

**Skills demonstrated:** phishing investigation, OSINT, domain/IP review, suspicious site analysis, evidence handling, incident-style reporting.

---

### [Debian 13 Workstation Migration and Hardening](projects/linux-migration-hardening/)

Migrated a daily-driver workstation from Windows 11 to Debian 13 and treated the process as an endpoint hardening project. The work includes LUKS2 full-disk encryption, Secure Boot validation, nftables firewall hardening, firewall conflict resolution, sysctl tuning, ClamAV/Lynis auditing, and a systemd timer-based maintenance suite.

**Skills demonstrated:** Linux administration, endpoint hardening, nftables, firewall troubleshooting, disk encryption, systemd timers, audit artifacts, maintenance automation.

---

### [Wazuh Alert Optimization](projects/wazuh-alert-optimization/)

Operationalized Windows failed logon detection in Wazuh with Slack alerting during an earlier SIEM iteration of the homelab. This project documents the difference between a detection existing and a detection being actionable, including alert tuning and notification workflow design.

**Skills demonstrated:** Wazuh, Windows authentication logs, custom alert tuning, Slack integration, detection operationalization.

---

## Homelab Environment

The core lab environment is `schmeck.lab`, a self-built virtual enterprise network running on a Debian 13 workstation with KVM/QEMU and libvirt.

The environment includes:

- pfSense firewall and gateway
- Isolated `192.168.10.0/24` lab network
- Windows Server 2022 domain controller: `DC01`
- Windows 11 domain workstation: `WIN11`
- Ubuntu Splunk Enterprise SIEM host
- Kali Linux attacker/testing VM
- Splunk Universal Forwarders and Sysmon on Windows hosts

Full infrastructure documentation is available in [`homelab/homelab-configuration/`](homelab/homelab-configuration/README.md).

Network architecture documentation is available in [`homelab/homelab-configuration/homelab-architecture/`](homelab/homelab-configuration/homelab-architecture/README.md).

---

## Skills Represented

This portfolio includes work across:

- Security operations and SOC triage
- SIEM deployment and dashboarding
- Detection engineering and alert tuning
- Windows Event Log and Sysmon analysis
- Active Directory security fundamentals
- Vulnerability assessment and attack surface analysis
- Linux endpoint hardening
- Network segmentation and firewall troubleshooting
- Phishing and smishing investigation
- Security policy and governance documentation
- MITRE ATT&CK mapping from both attacker and defender perspectives

---

## Repository Structure

### [`projects/`](projects/README.md)

Recruiter-facing project case studies. These are the primary portfolio entries and should be read first.

Current projects include:

- [`soc-dashboard-alerting/`](projects/soc-dashboard-alerting/)
- [`vulnerability-assessment/`](projects/vulnerability-assessment/)
- [`active-directory-siem/`](projects/active-directory-siem/)
- [`osint-smishing-investigation/`](projects/osint-smishing-investigation/)
- [`linux-migration-hardening/`](projects/linux-migration-hardening/)
- [`wazuh-alert-optimization/`](projects/wazuh-alert-optimization/)

### [`homelab/`](homelab/README.md)

Supporting infrastructure documentation for the `schmeck.lab` environment. This includes network architecture, VM roles, SIEM infrastructure, and lab configuration notes.

### [`policy-and-governance/`](policy-and-governance/README.md)

Academic security policy library developed as part of the B.S. Cybersecurity program at Kennesaw State University. Includes EISP, ISSP, and SYSSP policy documents covering incident response, disaster recovery, contingency planning, IDS, firewall/VPN security, authentication, change control, and related governance topics.

### [`learning-notes/`](learning-notes/tryhackme/README.md)

Training notes and write-ups from TryHackMe and related learning exercises. These are reference notes, not polished portfolio case studies.

### [`archive/`](archive/README.md)

Deprecated material from earlier lab iterations, retained for continuity and historical context.

---

## Certifications and Background

- **CompTIA Security+**
- **B.S. Cybersecurity** — Kennesaw State University
- Ongoing hands-on practice through self-directed labs, TryHackMe, HackTheBox, and portfolio-driven security projects

---

## Contact

- GitHub: [@aschmeck](https://github.com/aschmeck)
