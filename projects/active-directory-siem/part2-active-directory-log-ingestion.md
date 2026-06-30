# Homelab Series — Part 2: Active Directory Deployment and Splunk Universal Forwarder Configuration

**Author:** [@aschmeck](https://github.com/aschmeck)  
**Series:** Homelab Security Operations  
**Part:** 2 of ongoing  
**Previous:** [Part 1 — Splunk Enterprise Deployment on a Segmented KVM Lab Network](part1-splunk-enterprise-deployment.md)  
**Next:** Part 3 — Windows 11 Endpoint Onboarding, Sysmon Deployment, and Attack Simulation (Coming Soon)

---

## Overview

This document covers the deployment of an Active Directory Domain Services (AD DS) environment on Windows Server 2022, the configuration of a realistic organizational unit and user structure, audit policy hardening, and the installation of a Splunk Universal Forwarder to ship Windows Event Logs to the Splunk Enterprise instance deployed in Part 1.

At the conclusion of this document, Windows Security, System, and Application event logs from the domain controller are flowing into Splunk and searchable in real time.

---

## Environment

### Infrastructure

This lab builds on the environment established in Part 1. Refer to that document for host machine specifications and full network topology. The relevant components for this document are:

| Component | Detail |
|---|---|
| Domain Controller | win2k22 — Windows Server 2022 Evaluation |
| Splunk Indexer | ubuntu24.04 — 192.168.10.50 |
| Virtual Network | lab-lan — 192.168.10.0/24 |
| Gateway / DHCP | pfSense — 192.168.10.1 |

### Domain Structure

| Component | Value |
|---|---|
| Domain Name | schmeck.lab |
| NetBIOS Name | SCHMECK |
| Forest / Domain Mode | Windows2016 |
| Domain Controller | DC01.schmeck.lab |
| DC IP Address | 192.168.10.100 (static) |
| FSMO Roles | All roles held by DC01 |

---

## Prerequisites

- Part 1 completed — Splunk Enterprise running at `192.168.10.50:8000` with port 9997 receiving enabled and a `windows` index created
- Windows Server 2022 VM installed and updated
- VM connected to lab-lan and receiving a DHCP lease from pfSense
- Internet access confirmed from the VM prior to AD DS installation

---

## Step 1 — Prepare the Domain Controller

### 1.1 Set a Static IP Address

A domain controller must have a static IP address. DHCP-assigned addresses are not appropriate for a DC — DNS records, Kerberos authentication, and replication all depend on a stable address.

The following commands remove the existing DHCP-assigned address and replace it with a static configuration matching the lease that pfSense had already assigned. DNS is pointed at localhost (`127.0.0.1`) in anticipation of the DC running its own DNS server after promotion.

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.100 -Confirm:$false
Remove-NetRoute -InterfaceAlias "Ethernet" -DestinationPrefix 0.0.0.0/0 -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.100 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
```

Verify the configuration:

```powershell
ipconfig /all
```

Expected: DHCP Enabled — No, IPv4 Address — 192.168.10.100, DNS Servers — 127.0.0.1.

### 1.2 Install the AD DS Role

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

A reboot is required after role installation completes before promotion can proceed.

---

## Step 2 — Promote to Domain Controller

### 2.1 Create a New Forest

The following command creates a new AD forest with the root domain `schmeck.lab`. The `-InstallDns` flag configures the DC to serve as the primary DNS server for the domain — standard practice for a single-DC environment.

A Directory Services Restore Mode (DSRM) password is prompted during execution. This password is used for AD recovery and should be stored securely and separately from standard administrative credentials.

The server reboots automatically after promotion completes. On next login the domain prefix `SCHMECK\` will be present, confirming successful promotion.

```powershell
Install-ADDSForest `
    -DomainName "schmeck.lab" `
    -DomainNetbiosName "SCHMECK" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -Force:$true
```

### 2.2 Verify Domain and Forest Health

```powershell
Get-ADDomain
Get-ADForest
```

Key values to confirm:

- `DNSRoot` — schmeck.lab
- `PDCEmulator`, `RIDMaster`, `InfrastructureMaster` — all pointing to DC01.schmeck.lab
- `DomainMode` — Windows2016Domain
- `GlobalCatalogs` — DC01.schmeck.lab

---

## Step 3 — Build the Organizational Unit Structure

Default AD containers cannot have Group Policy Objects linked to them directly and are not intended for production object placement. A proper OU structure is created to organize domain objects and support GPO targeting in later phases.

```powershell
New-ADOrganizationalUnit -Name "Workstations" -Path "DC=schmeck,DC=lab"
New-ADOrganizationalUnit -Name "Corp-Users" -Path "DC=schmeck,DC=lab"
New-ADOrganizationalUnit -Name "Corp-Groups" -Path "DC=schmeck,DC=lab"
```

> **Note:** The default `Users` container already exists in AD as a built-in container, not an OU. Attempting to create an OU named `Users` at the domain root will fail. `Corp-Users` and `Corp-Groups` avoid this conflict while maintaining a clear naming convention.

---

## Step 4 — Create Users and Groups

### 4.1 Create Domain Users

Three users are created representing realistic personas at different privilege levels. Fictional names are used deliberately — they are unambiguous in log output and clearly not real personal data.

```powershell
# Standard user
New-ADUser -Name "Scott Summers" -GivenName "Scott" -Surname "Summers" `
    -SamAccountName "ssummers" -UserPrincipalName "ssummers@schmeck.lab" `
    -Path "OU=Corp-Users,DC=schmeck,DC=lab" `
    -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
    -Enabled $true

# Standard user
New-ADUser -Name "Warren Worthington" -GivenName "Warren" -Surname "Worthington" `
    -SamAccountName "wworthington" -UserPrincipalName "wworthington@schmeck.lab" `
    -Path "OU=Corp-Users,DC=schmeck,DC=lab" `
    -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
    -Enabled $true

# Privileged IT administrator
New-ADUser -Name "Emma Frost" -GivenName "Emma" -Surname "Frost" `
    -SamAccountName "efrost" -UserPrincipalName "efrost@schmeck.lab" `
    -Path "OU=Corp-Users,DC=schmeck,DC=lab" `
    -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
    -Enabled $true
```

### 4.2 Create Security Groups and Assign Membership

```powershell
New-ADGroup -Name "IT-Admins" -GroupScope Global -GroupCategory Security `
    -Path "OU=Corp-Groups,DC=schmeck,DC=lab"

Add-ADGroupMember -Identity "IT-Admins" -Members "ssummers"
Add-ADGroupMember -Identity "Domain Admins" -Members "ssummers"
```

### 4.3 Verify Structure

```powershell
Get-ADUser -Filter * -SearchBase "OU=Corp-Users,DC=schmeck,DC=lab" | Select Name, SamAccountName, Enabled
Get-ADGroupMember -Identity "IT-Admins" | Select Name
Get-ADGroupMember -Identity "Domain Admins" | Select Name
```

---

## Step 5 — Configure Audit Policy

Windows audit policy controls which security events are written to the Security event log. Without explicit configuration the Security log generates minimal output — insufficient for meaningful SOC work. The subcategories below are enabled to produce the event IDs most relevant to threat detection and incident response.

```powershell
# Logon and session tracking
AuditPol /set /subcategory:"Logon" /success:enable /failure:enable
AuditPol /set /subcategory:"Logoff" /success:enable
AuditPol /set /subcategory:"Account Lockout" /success:enable /failure:enable

# Privilege use and account management
AuditPol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable
AuditPol /set /subcategory:"User Account Management" /success:enable /failure:enable
AuditPol /set /subcategory:"Security Group Management" /success:enable /failure:enable
AuditPol /set /subcategory:"Computer Account Management" /success:enable /failure:enable

# Process creation and directory activity
AuditPol /set /subcategory:"Process Creation" /success:enable
AuditPol /set /subcategory:"Directory Service Access" /success:enable /failure:enable
AuditPol /set /subcategory:"Directory Service Changes" /success:enable

# Policy change tracking
AuditPol /set /subcategory:"Audit Policy Change" /success:enable /failure:enable
```

Verify:

```powershell
AuditPol /get /category:*
```

### Key Event IDs Produced by This Policy

| Event ID | Description |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4634 | Account logoff |
| 4648 | Logon using explicit credentials |
| 4672 | Special privileges assigned to new logon |
| 4720 | User account created |
| 4726 | User account deleted |
| 4728 | Member added to global security group |
| 4740 | Account locked out |
| 4767 | Account unlocked |

---

## Step 6 — Install Splunk Universal Forwarder

### 6.1 Download the Forwarder

The Universal Forwarder MSI is downloaded directly from Splunk's official portal. A free Splunk account is required.

1. Navigate to [splunk.com/en_us/download/universal-forwarder.html](https://www.splunk.com/en_us/download/universal-forwarder.html)
2. Select **Windows** → **64-bit** → copy the direct download link
3. Download to DC01:

```powershell
Invoke-WebRequest -Uri "<official-download-link>" -OutFile "C:\splunkforwarder.msi"
```

> **Note:** Always retrieve the download link directly from Splunk's portal. The link changes with each release and includes a verification token.

### 6.2 Silent Installation

```powershell
msiexec.exe /i C:\splunkforwarder.msi SPLUNKUSERNAME=admin SPLUNKPASSWORD=changeme `
    RECEIVING_INDEXER="192.168.10.50:9997" `
    AGREETOLICENSE=Yes /quiet
```

### 6.3 Configure Windows Event Log Inputs

The MSI installer's `WINEVENTLOG_*` flags do not reliably write `inputs.conf` across all versions. The file is authored manually to ensure correct channel names and index targeting.

```powershell
$inputsConf = @"
[WinEventLog://Security]
index = windows
disabled = 0

[WinEventLog://System]
index = windows
disabled = 0

[WinEventLog://Application]
index = windows
disabled = 0
"@

Set-Content "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf" -Value $inputsConf
```

Restart the forwarder to apply the configuration:

```powershell
Restart-Service SplunkForwarder
Get-Service SplunkForwarder
```

### 6.4 Troubleshooting

If logs do not appear in Splunk, inspect the forwarder log:

```powershell
Get-Content "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" -Tail 50
```

Common issues encountered during this deployment:

- **Typo in channel name** — `splunkd.log` will report `Failed to find Event Log with channel name='<value>'`. Correct the spelling in `inputs.conf` and restart the service.
- **Forwarder connecting but no events flowing** — Confirm `inputs.conf` exists at the correct path and contains valid stanzas. The MSI silent install does not reliably create this file.

---

## Step 7 — Verify Log Ingestion

### 7.1 Generate Test Events

```powershell
# Failed logon — Event ID 4625
net use \\DC01\IPC$ /user:ssummers wrongpassword 2>$null

# Multiple failed logons to trigger lockout — Event ID 4740
1..5 | ForEach-Object {
    net use \\DC01\IPC$ /user:wworthington wrongpassword 2>$null
}

# Unlock account — Event ID 4767
Unlock-ADAccount -Identity "wworthington"

# Create and delete user — Event IDs 4720 and 4726
New-ADUser -Name "Jean Grey" -SamAccountName "jgrey" `
    -Path "OU=Corp-Users,DC=schmeck,DC=lab" `
    -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true
Remove-ADUser -Identity "jgrey" -Confirm:$false
```

### 7.2 Search in Splunk

Navigate to **Search & Reporting** at `http://192.168.10.50:8000` and run:

```
index=windows host=DC01 EventCode IN (4625, 4740, 4720, 4726, 4767)
```

Confirmed event types flowing from DC01 into Splunk:

- Failed logon attempts (4625)
- Account lockout (4740)
- Account unlocked (4767)
- User account created (4720)
- User account deleted (4726)

---

## Operational Notes

### Splunk Forwarder Persistence Queue

The Universal Forwarder maintains a local persistent queue when the Splunk indexer is unreachable. Events generated while ubuntu24.04 is offline are queued on disk and forwarded automatically when the connection is restored. Event timestamps reflect the time of original generation — not delivery time — preserving accurate timeline analysis.

The default queue size is 500MB per forwarder, sufficient for extended offline periods in a lab environment. This allows the Splunk VM to be shut down between sessions without losing event data from Windows VMs that remain active.

---

## Current State

At the completion of this document the following is in place:

- `schmeck.lab` AD forest deployed — DC01 holds all FSMO roles
- Static IP assigned to DC01 (192.168.10.100)
- OU structure: Workstations, Corp-Users, Corp-Groups
- Domain users: ssummers (Domain Admin, IT-Admins), wworthington (standard), efrost (standard)
- Audit policy configured across logon, account management, privilege use, process creation, and directory service categories
- Splunk Universal Forwarder installed on DC01 with manually authored `inputs.conf`
- Security, System, and Application Windows Event Logs flowing into the `windows` index in real time

---

## What's Next

**Part 3** will cover:

- Windows 11 endpoint domain join to schmeck.lab
- Splunk Universal Forwarder deployment on WIN11
- Sysmon deployment and configuration on DC01 and WIN11
- Attack simulation from Kali Linux
- Detection of simulated attacks in Splunk using SPL

---

## Reference

| Resource | URL |
|---|---|
| Splunk Universal Forwarder | https://www.splunk.com/en_us/download/universal-forwarder.html |
| Splunk Windows Event Log Monitoring | https://docs.splunk.com/Documentation/Splunk/latest/Data/MonitorWindowseventlogdata |
| Microsoft Audit Policy Reference | https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/advanced-security-audit-policy-settings |
| AD DS Deployment | https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services |
| Windows Security Event IDs | https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/security-auditing-overview |

---

*Part of the [Homelab Security Operations](https://github.com/aschmeck) series by [@aschmeck](https://github.com/aschmeck)*
