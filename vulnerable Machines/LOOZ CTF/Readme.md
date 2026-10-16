# Looz CTF — Vulnerability Assessment & Penetration Test

Boot2root CTF write-up for a self-hosted vulnerable VM, **Looz**, performed on a private home-lab network for SOC/pentest skill-building.

## Target

| | |
|---|---|
| Hostname | `looz.lan` |
| IP address | `192.168.31.120` |
| Objective | Gain root access and capture `root.txt` |

## Files in this package

| File | Purpose |
|---|---|
| `Looz_CTF_Pentest_Report.pdf` | Full write-up: methodology, screenshots/PoC for every step, and a per-vulnerability findings table (port, service, version, severity, CVE, description, remediation, PoC reference). |
| `report1.xlsx` | Vulnerability log template used to track each finding individually (one row of fields per vulnerability: Description, Vulnerability name, Port, Service name, Version, Severity, CVE ID, Remediation, Proof of Concept). Use it to copy/log each of the 7 findings below as a separate tracked entry. |

## Attack chain summary

1. **Host discovery** — `nmap -sn` identified the target at `192.168.31.120`.
2. **Port scan** — `nmap -sV -p-` found SSH (22), nginx (80), MariaDB (3306), and Apache/WordPress (8081); SMB (139/445) closed.
3. **Port 80 recon** — a static site was found; its page source contained an HTML comment leaking a plaintext WordPress username/password.
4. **Port 8081 recon** — `gobuster` confirmed a WordPress install (`wp-admin`, `wp-login.php`, `wp-config.php` reachable).
5. **WordPress takeover** — the leaked credentials logged into `wp-admin`, exposing an outdated WordPress core (7.0.2, update pending) and end-of-life PHP (7.4.20).
6. **SSH brute force** — `hydra` with `rockyou.txt` cracked a weak password for the local user `gandalf`.
7. **Privilege escalation** — a SUID binary (`shell_testv1.0`) in another user's home directory, owned by root, was executed to obtain a root shell.
8. **Flag captured** — `root.txt` → `ab17850978e36aaf6a2b8808f1ded971`

## Findings at a glance

| # | Finding | Port | Severity |
|---|---|---|---|
| 1 | Credentials disclosed in HTML source comment | 80/tcp | Critical |
| 2 | Leaked WordPress admin credentials → full admin takeover | 8081/tcp | Critical |
| 3 | Outdated WordPress core & EOL PHP version | 8081/tcp | Medium |
| 4 | Directory/content enumeration via predictable paths | 80/tcp, 8081/tcp | Low |
| 5 | Weak SSH password, cracked via dictionary attack | 22/tcp | Critical |
| 6 | Misconfigured SUID binary → local privilege escalation to root | local | Critical |
| 7 | Database service exposed to the network | 3306/tcp | Medium |

Full details, remediation steps, and proof-of-concept screenshots for each finding are in `Looz_CTF_Pentest_Report.pdf`.

## Notes

- Any unrelated hosts visible during network discovery (neighboring devices, the attacker box, etc.) have been removed from the report — only the target's IP/hostname is shown.
- Credentials shown throughout (`gandalf` / cracked SSH password, WordPress `john` / leaked password) belong to the vulnerable lab VM, not to any personal account.

