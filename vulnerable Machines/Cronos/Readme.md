# Cronos — CTF Vulnerability Assessment

## Overview
- **Target:** `chronos.local`
- **Target IP:** `192.168.31.242`
- **Environment:** Kali Linux and an isolated vulnerable virtual machine
- **Assessment Type:** Vulnerability Assessment and Penetration Testing (VAPT)

## Objective
Identify exposed services, investigate web application behavior, validate observations using practical evidence, and document remediation recommendations.

## Tools Used
- Nmap
- Burp Suite
- CyberChef
- Netcat
- Linux command-line utilities
- Web browser and JavaScript source inspection

## Scope and Methodology
1. Host discovery and target identification.
2. Port scanning and service-version enumeration.
3. Web application source-code inspection.
4. Investigation of the `/date?format=...` endpoint.
5. HTTP request and response analysis using Burp Suite.
6. Shell identity verification and application directory enumeration.
7. Documentation of evidence and remediation recommendations.

## Ports Identified

| Port | Service | Observation |
|---|---|---|
| 22/tcp | OpenSSH | SSH service identified by Nmap |
| 80/tcp | Apache HTTP | Web server identified by Nmap |
| 8000/tcp | Node.js / Express | Web application and date endpoint investigated |

## Key Observations
- The web application contained obfuscated JavaScript referencing a date endpoint.
- Endpoint tests returned `Permission Denied` and `Something went wrong`.
- Shell evidence showed execution as `www-data` (UID 33).
- Application directories and a backend dependency manifest were inspected.

*Note: These observations are based on the collected lab evidence. The exact root cause of the initial shell access and specific service vulnerabilities require further validation.*

## Detailed Report and PoC

📄 **[View Cronos-CTF-Report.pdf](https://github.com/NikhilCyberOps/cybersecurity-labs/blob/main/vulnerable%20Machines/Cronos/Cronos-CTF-Report.pdf)**

The PDF contains separate port-wise findings, proof-of-concept evidence, screenshots, limitations, and remediation recommendations.

## Skills Demonstrated
- Network reconnaissance and service enumeration
- Web application investigation
- HTTP request analysis
- Linux shell and application enumeration
- Technical documentation and remediation planning

## Disclaimer
This assessment was performed in an authorized CTF training environment for educational purposes. Do not test systems without permission.

