# Homelab Series — Part 3: Windows 11 Endpoint Onboarding and Sysmon Deployment

**Author:** [@aschmeck](https://github.com/aschmeck)  
**Series:** Homelab Security Operations  
**Part:** 3 of ongoing  
**Previous:** [Part 2 — Active Directory Deployment and Splunk Universal Forwarder Configuration](../part2-active-directory/splunk-homelab-part2.md)  
**Next:** Part 4 — Attack Simulation and Detection (Coming Soon)

---

## Objective

This document covers the deployment of the Splunk Universal Forwarder on the Windows 11 domain workstation (WIN11), followed by the installation and configuration of Sysmon v15.20 on both WIN11 and DC01 using the olafhartong sysmon-modular configuration. At the conclusion of this document, both endpoints are delivering Windows Event Logs and Sysmon telemetry into the `windows` index in Splunk Enterprise, providing the detection-grade visibility required for attack simulation and threat hunting in subsequent labs.

---

## Environment

| Component | Details |
|---|---|
| Host OS | Debian 13, KVM/QEMU hypervisor |
| SIEM | Splunk Enterprise 10.2.2 on Ubuntu 24.04 (192.168.10.50) |
| Domain Controller | Windows Server 2022 — DC01 (192.168.10.100) |
| Workstation | Windows 11 Pro — WIN11 (192.168.10.54) |
| Domain | schmeck.lab |
| Forwarder | Splunk Universal Forwarder 10.2.2 |
| Sysmon | Sysmon v15.20 (olafhartong sysmon-modular config) |
| Network | Isolated lab-lan (virbr-lan), no external routing |

---

## Problem Statement

With Splunk Enterprise and Active Directory operational from Parts 1 and 2, the remaining gap was endpoint telemetry coverage on the Windows 11 workstation and process-level visibility on both hosts. Standard Windows Event Logs capture authentication and account management activity but lack the process creation, network connection, and file system telemetry required for meaningful threat detection. Sysmon fills that gap by enriching the event stream with granular host activity mapped to MITRE ATT&CK techniques. This lab closes the visibility gap and establishes the telemetry foundation required for all subsequent detection and simulation work.

---

## Implementation

### Part A — Splunk Universal Forwarder on WIN11

#### Download and Installation

The Splunk Universal Forwarder 10.2.2 MSI was downloaded directly from Splunk's official download page and installed using the GUI installer. The installer was configured with the following options:

- License agreement accepted
- Installation type: On-premises Splunk Enterprise instance
- Service account: Local System
- Deployment Server: left blank (no deployment server in this environment)
- Receiving Indexer: configured post-install via `outputs.conf`

The GUI installer was used in preference to a silent MSI install. In the previous session, the MSI flags `WINEVENTLOG_SEC_ENABLE`, `WINEVENTLOG_SYS_ENABLE`, and `WINEVENTLOG_APP_ENABLE` did not reliably produce a valid `inputs.conf`. Manual file authoring was confirmed as the more reliable approach for this environment.

#### outputs.conf Verification

Following installation, the UF configuration directory was inspected:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\
```

The installer had written a valid `outputs.conf` pointing to the Splunk indexer:

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.10.50:9997

[tcpout-server://192.168.10.50:9997]
```

No modifications to `outputs.conf` were required.

#### inputs.conf — Manual Authoring

A `inputs.conf` was manually authored using `Set-Content` in PowerShell to ensure correct formatting and encoding:

```ini
[WinEventLog://Security]
index = windows
sourcetype = WinEventLog:Security
disabled = false

[WinEventLog://System]
index = windows
sourcetype = WinEventLog:System
disabled = false

[WinEventLog://Application]
index = windows
sourcetype = WinEventLog:Application
disabled = false
```

The SplunkForwarder service was started using `sc.exe` after confirming the configuration was correct. The forwarder log at `C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log` confirmed active connections to `192.168.10.50:9997` with repeated `Connected to idx` entries.

#### Connectivity Verification

Initial Splunk searches returned no results due to a time range mismatch — the index showed a "Latest Event: in 3 hours" offset caused by a timezone discrepancy between the forwarder and the indexer. Setting the time picker to **All Time** confirmed both DC01 and WIN11 were indexed and returning events. Both hosts were visible in:

```spl
index=windows | stats count by host
```

---

### Part B — Sysmon Deployment

Sysmon v15.20 was deployed on both DC01 and WIN11 using the olafhartong sysmon-modular configuration. This config was selected over the SwiftOnSecurity config, which is no longer actively maintained, in favor of sysmon-modular's active development and explicit MITRE ATT&CK technique mapping.

#### Download

On each host, Sysmon was downloaded from the official Microsoft Sysinternals distribution:

```powershell
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "C:\Users\...\Sysmon.zip"
```

The archive was extracted to `C:\Sysmon` and the olafhartong configuration was retrieved directly from the repository:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml" -OutFile "C:\Sysmon\sysmonconfig.xml"
```

#### Installation

Sysmon was installed from an elevated PowerShell prompt with the EULA accepted non-interactively:

```powershell
C:\Sysmon\Sysmon64.exe -i C:\Sysmon\sysmonconfig.xml -accepteula
```

This installs the Sysmon service and driver, which persists across reboots and begins writing events to:

```
Applications and Services Logs\Microsoft\Windows\Sysmon\Operational
```

#### Splunk inputs.conf Update

The Sysmon event log channel was added to `inputs.conf` on both hosts:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = windows
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
disabled = false
renderXml = true
```

The `renderXml = true` flag is required to preserve the structured XML format of Sysmon events, enabling proper field extraction in Splunk.

#### WIN11 — Access Denied Resolution

On WIN11, the SplunkForwarder service initially failed to subscribe to the Sysmon event log channel with `errorCode=5` (Access Denied). This is a known issue when the forwarder runs as the `NT SERVICE\SplunkForwarder` virtual account, which does not have default read access to the Sysmon operational log.

The resolution was to add the forwarder service account to the local Event Log Readers group:

```powershell
net localgroup "Event Log Readers" "NT SERVICE\SplunkForwarder" /add
```

An attempt was also made to modify the channel ACL directly via `wevtutil set-log`, however this did not persist after the service restart. The group membership approach resolved the issue cleanly. The forwarder service was restarted and the error did not recur.

#### Verification

Following the service restart on both hosts, Sysmon events were confirmed flowing into Splunk:

```spl
index=windows sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" | stats count by host
```

Results confirmed both endpoints reporting:

| Host | Sysmon Event Count |
|---|---|
| DC01 | 122 |
| WIN11 | 433 |

The first event observed from DC01 was an Event ID 3 (Network Connection) flagged with MITRE technique T1036 (Masquerading), generated by Windows Defender making an outbound connection to a Microsoft endpoint over port 443. This is an example of the enriched telemetry Sysmon provides beyond standard Windows Event Logs.

---

## Attacker / Defender Perspective

**Defender:**
Sysmon Event ID 1 (Process Creation) is the foundational detection primitive for most endpoint-based detections. Combined with Event ID 3 (Network Connection) and Event ID 10 (Process Access), it enables detection of techniques including process injection, credential dumping via LSASS access, and command-and-control beaconing. The olafhartong config explicitly maps rules to MITRE ATT&CK techniques, meaning each logged event carries technique context directly in the `RuleName` field — visible in the first DC01 event captured in this lab.

**Attacker:**
Sysmon is a high-value evasion target. Common evasion techniques include process name masquerading (T1036) to blend into legitimate process trees, direct system call usage to bypass Sysmon's user-mode hooks, and Sysmon driver tampering if the attacker obtains administrative access. The olafhartong config includes exclusion rules for high-volume benign processes to reduce noise — an attacker aware of the deployed config can potentially exploit these exclusions by staging activity inside excluded process paths. This is a deliberate tradeoff between signal fidelity and log volume that defenders must consciously manage.

MITRE ATT&CK references: T1036 (Masquerading), T1055 (Process Injection), T1003.001 (LSASS Memory), T1562.001 (Disable or Modify Tools)

---

## Outcome

At the conclusion of this document the following is in place:

- Splunk Universal Forwarder 10.2.2 installed and operational on WIN11
- Windows Security, System, and Application event logs from WIN11 flowing into the `windows` index
- Sysmon v15.20 installed on DC01 and WIN11 with the olafhartong sysmon-modular configuration
- Sysmon telemetry from both endpoints indexed in Splunk with structured XML field extraction enabled
- Event Log Readers group membership confirmed on WIN11 to resolve the forwarder access issue
- Full endpoint telemetry stack in place across both Windows hosts, ready for attack simulation

---

## Lessons Learned

- The Splunk UF MSI GUI installer does not reliably configure `inputs.conf` via the deployment server or indexer fields — manual file authoring remains the recommended approach for this environment.
- Sysmon's `NT SERVICE\SplunkForwarder` virtual account requires explicit membership in the Event Log Readers group on Windows 11 to access the Sysmon operational log. The `wevtutil set-log` ACL approach did not persist reliably; group membership was the effective fix.
- Splunk search time range defaults can mask successfully indexed data — always verify with **All Time** when troubleshooting forwarder connectivity before assuming a configuration failure.
- The olafhartong sysmon-modular config supersedes SwiftOnSecurity as the maintained community standard and provides explicit MITRE ATT&CK technique mapping in the `RuleName` field, which is directly useful for detection engineering.

---

## Future Improvements

- Tune the Sysmon configuration to reduce noise from known-benign processes in the lab environment (Windows Defender, Splunk UF itself) to improve signal-to-noise ratio before attack simulation.
- Configure Splunk field extractions or a Technology Add-on (TA) for Sysmon to enable proper field parsing and CIM compliance.
- Implement Sysmon configuration version control so config changes can be tracked alongside detection rule changes.

---

## What's Next

**Part 4** will cover:

- Attack simulation from Kali Linux against the schmeck.lab environment
- Detection of simulated attacks in Splunk using SPL
- Correlation of Sysmon and Windows Event Log telemetry across the kill chain

---

## Reference

| Resource | URL |
|---|---|
| Splunk Universal Forwarder | https://www.splunk.com/en_us/download/universal-forwarder.html |
| Sysmon — Microsoft Sysinternals | https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon |
| olafhartong sysmon-modular | https://github.com/olafhartong/sysmon-modular |
| Splunk Windows Event Log Monitoring | https://docs.splunk.com/Documentation/Splunk/latest/Data/MonitorWindowseventlogdata |
| MITRE ATT&CK | https://attack.mitre.org |

---

*Part of the [Homelab Security Operations](https://github.com/aschmeck) series by [@aschmeck](https://github.com/aschmeck)*
