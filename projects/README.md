# Projects

This directory contains standalone projects that support or extend the homelab work. Each project is documented with artifacts, configuration files, and the reasoning behind key decisions — including what went wrong and how it was resolved.

---

## Contents

### [`linux-migration-hardening/`](linux-migration-hardening/README.md)
Documents the complete migration of a daily-driver workstation from Windows 11 to Debian 13, treated with the rigor of an enterprise endpoint hardening effort. Covers pre-migration ISO verification, LUKS2 full-disk encryption, nftables firewall design, resolution of a firewall conflict caused by firewalld silently overriding nftables rules, and a systemd-based maintenance suite covering automated updates, log rotation, integrity checks, and scheduled auditing with Lynis and ClamAV. All configuration files, rulesets, and logs are captured as version-controlled artifacts.

### [`wazuh-alert-optimization/`](wazuh-alert-optimization/README.md)
Documents the operationalization of Windows failed logon alerting using Wazuh and Slack. The lab addresses a gap that exists in many SOC environments: detection firing in a SIEM that nobody sees because there is no real-time notification mechanism. Covers alert pipeline configuration, Slack integration using a modern application webhook, threshold tuning, and end-to-end validation. The core finding — that detection without delivery is operationally ineffective — is documented with evidence and framed against real SOC workflow.

### [`osint-smishing-investigation/`](osint-smishing-investigation/README.md)
Documents a structured security analysis of a real-world smishing job scam targeting the author. The case study treats the scam as a user-reported security event and covers evidence preservation, defensive OSINT, infrastructure enrichment, social engineering analysis, MITRE ATT&CK and SOC workflow mapping, attribution limits, and responsible reporting. Sensitive live indicators are redacted so the project demonstrates analyst methodology without exposing personal data or enabling abuse.

