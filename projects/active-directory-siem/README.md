# Active Directory SIEM Build

This project documents the build-out of the telemetry foundation for the `schmeck.lab` homelab: Splunk Enterprise, Windows Server 2022 Active Directory, Splunk Universal Forwarders, Windows Event Logs, and Sysmon endpoint telemetry.

The goal was to create a monitored Windows domain environment capable of supporting later detection engineering, vulnerability assessment validation, SOC dashboarding, and simulated attack investigations.

This project is separate from the SOC dashboard and alerting work. The purpose here is to build and validate the monitored environment. The dashboard project uses this telemetry for operational SOC visibility and alerting.

## Series

1. [Part 1 — Splunk Enterprise Deployment](part1-splunk-enterprise-deployment.md)
2. [Part 2 — Active Directory Log Ingestion](part2-active-directory-log-ingestion.md)
3. [Part 3 — Endpoint Sysmon Telemetry](part3-endpoint-sysmon-telemetry.md)

## Environment

- Debian 13 host running KVM/QEMU/libvirt
- pfSense gateway and network segmentation
- Ubuntu 24.04 Splunk Enterprise SIEM
- Windows Server 2022 domain controller
- Windows 11 domain workstation
- Splunk Universal Forwarders
- Sysmon endpoint telemetry

## Skills Demonstrated

- Splunk Enterprise deployment
- Active Directory deployment
- Windows Event Log forwarding
- Splunk Universal Forwarder configuration
- Sysmon deployment and troubleshooting
- Endpoint telemetry validation
- SIEM ingestion troubleshooting
- Detection engineering readiness

## Related Project

The telemetry built here is used in the [SOC Dashboard and Alerting Pipeline](../soc-dashboard-alerting/README.md) project.
