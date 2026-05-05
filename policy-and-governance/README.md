# Policy and Governance — CTS Information Security Policy Library

## Overview

This section contains a complete information security policy library developed as the capstone project for the Bachelor of Science in Cybersecurity program at Kennesaw State University. The library was authored for Cyber Tree Systems (CTS), a fictional business IT solutions company, and covers the full policy hierarchy from enterprise-level governance down to system-specific operational procedures.

The library demonstrates applied knowledge of security policy structure, governance frameworks, and the procedural controls that underpin real-world security operations — competencies directly relevant to SOC analyst and information security roles.

---

## Policy Hierarchy

The library is organized into three tiers, following the standard EISP → ISSP → SYSSP model:

**Enterprise Information Security Policy (EISP)**
Sets the organizational security posture, defines roles and responsibilities, and establishes the authority and scope under which all subordinate policies operate.

**Issue-Specific Security Policies (ISSP)**
Address discrete security domains with targeted controls and procedures:
- Antivirus Policy
- Authentication and Authorization Policy
- Change Control Policy
- Email and Messaging Use and Retention Policy
- Firewall and VPN Security Policy
- Intrusion Detection System Policy

**System-Specific Security Policies (SYSSP)**
Govern the operational procedures for critical systems and response functions:
- Contingency Planning Policy
- Disaster Recovery Policy
- Incident Response Policy
- Systems Management Policy

A Security Directory (index document) maps the full library and serves as the entry point for navigating the policy set.

---

## Relevance to Security Operations

Policy familiarity is a practical SOC skill. Analysts operate within governance frameworks daily — escalation timelines, incident categorization criteria, change control requirements, and data handling rules are all defined at the policy layer. Understanding that layer, not just the tooling above it, is what allows an analyst to work effectively within an organization's security posture rather than around it.

Several documents in this library map directly to the detection and response work demonstrated in the homelab section of this portfolio:

- The **Incident Response Policy** defines the SIRT structure, categorization criteria (Category 1–4), and escalation procedures that mirror real SOC triage workflows.
- The **IDS Policy** specifies signature update cadence, VPN traffic monitoring requirements, and the integration between IDS and firewall that informs how detection infrastructure is designed.
- The **Contingency Planning Policy** covers the governance structure — CPC composition, emergency VPN account controls, failover firewall responsibilities — that organizations rely on when a detection escalates to a full incident.
- The **Change Control Policy** establishes the approval and documentation requirements for firewall and security infrastructure changes, directly applicable to the nftables and pfSense configuration work in the homelab.

---

## Document Notes

All policies are versioned at 2.1. Revision history is included within each document. The library was last reviewed in February 2025 as part of the capstone course submission.

The fictional company name (Cyber Tree Systems) and associated branding are artifacts of the course assignment. The policy content, structure, and governance framework reflect real information security standards and were authored independently.
