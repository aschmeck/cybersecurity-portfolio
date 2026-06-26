# Services

This directory documents the deployment and configuration of the core security services and infrastructure underpinning the `schmeck.lab` homelab environment. This includes the SIEM, the Active Directory domain, endpoint telemetry tooling, and the log forwarding pipeline that connects them.

---

## Contents

| Write-up | Description | Status |
|---|---|---|
| [splunk-homelab-part1.md](./splunk/splunk-homelab-part1.md) | SIEM deployment and initial configuration | ✅ Complete |
| [splunk-homelab-part2.md](./splunk/splunk-homelab-part2.md) | Active Directory deployment and endpoint log forwarding — DC01 | ✅ Complete |
| [splunk-homelab-part3.md](./splunk/splunk-homelab-part3.md) | Domain join, log forwarding, and Sysmon deployment — WIN11 | ✅ Complete |
| [splunk-homelab-part4.md](./splunk/splunk-homelab-part4.md) | SOC dashboard and alerting — Slack delivery pipeline | ✅ Complete |

---

## Series Overview

This series documents the full build-out of the detection infrastructure used in all subsequent lab work. Each part covers deployment, configuration, and validation — including the troubleshooting decisions and lessons learned along the way, not just the steps that worked.

**Part 1** establishes the SIEM on Ubuntu 24.04, creates the index for Windows event data, configures the receiving port, and validates the instance is ready to accept forwarded telemetry.

**Part 2** deploys Active Directory on Windows Server 2022, standing up the `schmeck.lab` domain with organizational units and user accounts. It then configures log forwarding from DC01 to the SIEM, with a manually authored `inputs.conf` — a deliberate choice over installer flag-based configuration, which proved unreliable in testing.

**Part 3** completes the log forwarding deployment on WIN11, joins the workstation to the `schmeck.lab` domain, and deploys Sysmon on both Windows endpoints for enhanced endpoint telemetry.

**Part 4** builds the operational SOC dashboard in Splunk Classic Dashboard, configures three scheduled alerts with Slack delivery (brute force, account lockout, DA account misuse), and validates the full pipeline against simulated attack conditions. Includes an unplanned finding: credential scanning activity from the VA lab surfaced on first dashboard load and investigated end-to-end.

---

## Note on SIEM History

Wazuh was the original SIEM used in this lab environment, used during the initial alert operationalization work documented in the [Wazuh Alert Optimization project](../../../projects/wazuh-alert-optimization/). The decision to migrate to a new SIEM was made to better align with enterprise SOC tooling and to support the detection engineering and threat hunting work planned for subsequent labs.
