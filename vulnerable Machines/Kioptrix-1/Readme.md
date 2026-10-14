
# Kioptrix Level 1 — Vulnerable Machine Lab

## Overview
- **Platform:** VulnHub
- **Category:** Vulnerability Assessment & Penetration Testing (VAPT)
- **Environment:** Kali Linux and an isolated vulnerable virtual machine

## Objective
Identify exposed services, investigate security vulnerabilities, and document the testing evidence and remediation recommendations.

## Tools Used
- Kali Linux
- Nmap
- Metasploit Framework

## Practical Work
- Discovered the target IP and verified connectivity.
- Scanned open ports and enumerated running services.
- Investigated the Samba service and researched a potential buffer overflow vulnerability.
- Documented exploitation evidence from the lab.
- Reviewed exposed services, including SSH, HTTP/HTTPS, and RPC.

## Key Findings
The report discusses a potential Samba buffer overflow and several service exposure or configuration concerns. Findings and severity ratings should be verified against the evidence before being treated as confirmed vulnerabilities.

## Remediation
- Update vulnerable software and apply security patches.
- Restrict access to unnecessary services and ports.
- Harden SSH and web server configurations.
- Disable unused services and validate fixes after remediation.

## Report
See [`Kioptrix1.pdf`] for the detailed assessment and screenshots.

## Disclaimer
This assessment was performed in an intentionally vulnerable lab environment for educational purposes.
