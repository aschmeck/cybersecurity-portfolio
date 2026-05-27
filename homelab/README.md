# Homelab

This directory contains the documentation for the `schmeck.lab` homelab environment — both the infrastructure that was built and the security work conducted against it.

---

## Contents

### [`homelab-configuration/`](homelab-configuration/README.md)
Documents the design and build-out of the lab environment itself — the network topology, VM inventory, Active Directory domain, and the detection infrastructure (SIEM, log forwarding, and endpoint telemetry) that underpins all lab work. Start here if you want to understand what the environment looks like before diving into the labs.

### [`vulnerability-assessment/`](vulnerability-assessment/README.md)
Documents a structured vulnerability assessment conducted against the lab infrastructure from a Kali Linux attacker VM. Covers network reconnaissance, defender visibility validation, and host-by-host vulnerability scanning. This work establishes the baseline attack surface that all subsequent offensive simulation and detection engineering labs build from.

---

## Relationship Between the Two

The configuration work and the assessment work are intentionally connected. The infrastructure documented in `homelab-configuration/` is the same infrastructure assessed in `vulnerability-assessment/` — the Splunk SIEM and Sysmon telemetry deployed in the configuration phase are what make the defender-side visibility sections of the assessment possible. Building the environment and attacking it are two sides of the same lab.
