# Debian 13 Workstation Migration and Hardening

**Author:** [@aschmeck](https://github.com/aschmeck)

---

## Objective

Document the complete migration of a daily-driver workstation from Windows 11 to Debian 13, treating the process with the rigor of an enterprise endpoint hardening effort. The goal was a transparent, encrypted, and maintainable environment with a verifiable security baseline — not just a working installation.

---

## Environment

| Attribute | Detail |
|---|---|
| Host OS | Debian 13 (Trixie) |
| Desktop | Xfce |
| Disk Encryption | LUKS2 |
| Firewall | nftables |
| Auditing | Lynis, ClamAV |
| Automation | systemd timers |

---

## What Was Done

### Pre-migration validation
Hardware profiling was performed under Windows before installation to confirm driver compatibility and avoid post-install surprises. The Debian ISO was verified against its SHA-256 hash before writing to USB. A full system image backup was taken before wiping the drive.

### Installation and encryption
Debian 13 was installed with LUKS2 full-disk encryption on the root volume. Secure Boot was confirmed active post-install alongside the encrypted boot sequence.

### Firewall hardening and conflict resolution
nftables was deployed with a strict deny-by-default policy — loopback and established traffic permitted, all inbound dropped, SSH restricted to localhost only.

Despite correct rules, port 22 still appeared open on external scans. Investigation revealed that `firewalld`, active by default, was injecting its own nftables chains with an early-order `accept` rule for TCP/22 that evaluated before the drop rule. `firewalld` was removed to eliminate the conflict, and interface-specific filtering was applied to enforce SSH isolation. External scans confirmed the port as filtered while local access remained intact.

### Post-install hardening
sysctl tuning was applied for kernel-level hardening. ClamAV and Lynis were installed and configured. SSH daemon state was locked down. All configuration was captured as versioned artifacts.

### Automated maintenance suite
A systemd timer suite was implemented to handle ongoing system hygiene without manual intervention:

| Frequency | Task |
|---|---|
| Daily | Package updates and upgrades |
| Daily | Log rotation and cleanup |
| Daily | Disk usage monitoring |
| Daily | Configuration backups |
| Weekly | Temporary file cleanup |
| Weekly | ClamAV malware scan and Lynis audit |
| Weekly | Service health checks |
| Weekly | Package integrity verification |
| Monthly | Log and config archiving |

---

## Key Finding

The firewalld conflict is the most significant troubleshooting moment in this project. It demonstrates a real-world problem that would affect any administrator deploying nftables on a system where firewalld is present — silent rule injection that bypasses an otherwise correct deny-by-default policy without producing any obvious error. The diagnosis, root cause, and resolution are documented in full in [`initial-setup.md`](initial-setup.md).

---

## Artifacts

All configuration files, logs, and audit outputs are stored in [`artifacts/`](artifacts/README.md). Key evidence includes:

| Artifact | Description |
|---|---|
| `install_log.txt` | Installer summary |
| `hash_verification.txt` | SHA-256 verification of Debian ISO |
| `dmesg_init.txt` | Initial boot log |
| `apt_first_run.log` | First update and upgrade output |
| `nftables_final.conf` | Finalized firewall ruleset |
| `sysctl_hardening.txt` | Applied kernel parameters |
| `ssh_daemon_state.txt` | SSH daemon configuration state |
| `clamav_scan.log` | ClamAV scan output |
| `final_system_audit.txt` | Combined Lynis and ClamAV summary |
| `systemd_timer_list.txt` | Active maintenance timer inventory |
| `metrics_summary.md` | System resource and security metrics |
