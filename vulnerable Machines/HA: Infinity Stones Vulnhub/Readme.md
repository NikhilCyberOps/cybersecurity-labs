# HA:Infinity Stones CTF — Detailed PoC Report

## Files
- `HA_Infinity_Stones_CTF_Report.pdf` — step-by-step PoC and validation for each evidence-backed finding.
- This README summarizes the target and report scope.

## Target
- Victim IP: `192.168.31.197`
- CTF title: `HA:Infinity Stones CTF`

## Confirmed TCP ports
| Port | Service |
|---|---|
| 22 | OpenSSH 7.6p1 |
| 80 | Apache httpd 2.4.29 |
| 443 | HTTPS / Apache; certificate date shown expired |
| 8080 | Jetty 9.4.z-SNAPSHOT / Jenkins login |

Port 21 is mentioned in the workbook but not confirmed open by the supplied Nmap screenshot.

## PoCs included in PDF
1. SSH service enumeration and authorized login validation.
2. HTTP service enumeration and Gobuster path validation.
3. HTTPS certificate and TLS configuration checks.
4. Jenkins/Jetty exposure checks; traversal is explicitly marked unconfirmed.
5. Image metadata inspection with ExifTool.
6. CTF clue/hint file retrieval.
7. Authorized offline WPA handshake dictionary test (key redacted).
8. KeePass database audit and entry inspection (passwords redacted).
9. Privilege escalation through the demonstrated sudo rule allowing `/usr/bin/ftp` as root; FTP shell escape; `id`/`whoami` verification. This is a local binary abuse issue, not evidence of a network FTP service.

## Sanitization
- Retained only the victim IP and target name.
- Omitted unrelated LAN IPs, device names and MAC addresses.
- Omitted personal identifiers, passwords, wireless keys and vault secrets.
- Did not guess the final flag because it is not legible in the supplied evidence.

All commands are for the authorized, isolated CTF lab only.

