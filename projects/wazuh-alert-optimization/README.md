# Lab 1 — Operationalizing Windows Failed Logon Alerts (Wazuh → Slack)

## Objective
Ensure repeated Windows authentication failures are actionable in real time by routing high-signal alerts from Wazuh to an external notification channel.

---

## Environment
- **Endpoint:** Windows 11  
- **SIEM:** Wazuh  
- **Notification Platform:** Slack  
- **Attack Simulation:** Manual brute-force password attempts  

---

## Initial State
Wazuh successfully detected repeated failed Windows logon attempts using an existing rule:

- **Rule ID:** 60122  
- **Event Source:** Windows Security Event Logs  

These events appeared in Wazuh **Alerts** and **Threat Hunting**, but there was no real-time notification mechanism. Detection existed, but operational visibility did not. Alerts required manual dashboard review to be noticed.

---

## Problem Statement
Detection without delivery is ineffective in a SOC environment.  
High-risk authentication failures should notify responders immediately rather than rely on passive monitoring.

---

## Implementation

### 1. Detection Validation
- Generated repeated failed logon attempts on a Windows 11 endpoint.
- Confirmed Wazuh rule `60122` consistently fired and aggregated events over time.

### 2. Alert Delivery Configuration
- Created a Slack integration using a modern Slack application (not deprecated incoming webhooks).
- Configured an alert pipeline to send notifications when repeated failed logons occurred within a defined time window.

### 3. Alert Criteria
- **Trigger:** Repeated Windows failed logon attempts  
- **Time Window:** 5 minutes  
- **Delivery Method:** Slack channel notification  

---

## Validation & Evidence
- Repeated authentication failures produced:
  - Aggregated alerts in Wazuh
  - A real-time Slack notification indicating the alert entered an active state
- Timestamps between Wazuh and Slack aligned, confirming end-to-end alert delivery.

Evidence collected:
- Wazuh alert details showing rule ID and event metadata
- Alert count visualization in Wazuh
- Slack notification confirming alert activation

---

## Outcome
- Authentication brute-force activity is now immediately visible to responders.
- Alerting transitioned from passive detection to an operational SOC workflow.
- No custom detection rules were required; effectiveness was improved through alert delivery and visibility.

---

## Lessons Learned
- Existing detections are often sufficient but underutilized.
- Alert routing is as critical as alert creation.
- SIEM dashboards alone are not an effective notification mechanism.
- SOC effectiveness depends on signal visibility, not alert volume.

---

## Future Improvements
- Restrict Slack notifications to higher-severity authentication alerts.
- Add user- and source IP-based thresholding to reduce noise.
- Correlate failed logon events with firewall or network logs for additional context.

---

## Why This Lab Matters
This lab demonstrates SOC alert operationalization rather than simple tool deployment. It reflects real-world security operations work: identifying workflow gaps, improving alert visibility, and ensuring detections can be acted on in real time.
