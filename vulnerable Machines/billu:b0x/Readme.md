# Billu b0x CTF — VAPT Report

## Target
- **Victim hostname:** `indishell.lan`
- **Victim IP:** `192.168.31.158`
- **Report:** `Billu_b0x_CTF_VAPT_Report.pdf`
- **Scope:** Victim host only; unrelated hostnames and IP addresses are excluded.
- **Evidence:** Supplied screenshots only; no additional live testing was performed.

## Findings (Kioptrix-style PoC tables in PDF)

| Port / area | Finding | Severity | Evidence status |
|---|---|---:|---|
| 80/TCP | Local file disclosure through `test.php` | High | `/etc/passwd` output is visible in screenshot |
| 80/TCP | phpMyAdmin configuration disclosure / database secret exposure | Critical | Configuration source shown; password redacted |
| 22/TCP | Root SSH access / privileged host compromise | Critical | Screenshot shows `uid=0(root)` and `whoami` = root |
| 80/TCP | Legacy Apache version and HTTP hardening observations | Medium | Nmap and Nikto screenshots |
| 80/TCP | Discoverable application paths and test endpoints | Informational–Medium | Gobuster screenshot |

## Proof of Concept commands (authorized lab only)

### Service enumeration
```bash
nmap -sV -sC -p 22,80 192.168.31.158
nikto -h http://192.168.31.158
```

### Directory enumeration
```bash
gobuster dir -u http://192.168.31.158 -w /usr/share/wordlists/dirb/common.txt -x php,txt
```

### File disclosure validation
The evidence shows a request to `test.php` with the `file` parameter set to `/etc/passwd`, followed by local account data in the response. The same evidence sequence shows a request for `/var/www/phpmy/config.inc.php`. For safe retesting, use only the isolated CTF VM, capture the response, and redact secrets and account data before sharing. Avoid publishing database password values.

### Root access evidence
The supplied SSH screenshot shows a successful root session. For validation after remediation, confirm that direct root SSH login is disabled and that `id` / `whoami` return only the intended non-privileged account. Do not include passwords or private keys in the README.

## Remediation priorities
1. Rotate database and reused credentials; review logs and persistence after confirmed root access.
2. Remove the user-controlled file path from `test.php`; use allowlisted identifiers and canonical-path checks.
3. Move secrets outside the document root and deny web access to configuration files.
4. Disable direct root SSH login; restrict SSH to trusted sources and use key-based authentication.
5. Upgrade the legacy OS, Apache and OpenSSH to supported versions.
6. Remove unused test/debug endpoints and restrict phpMyAdmin; disable script execution in upload directories.
7. Retest and capture sanitized evidence.

## Evidence / privacy notes
- Victim identifiers retained: `indishell.lan` and `192.168.31.158`.
- Unrelated IP addresses, hostnames and personal credentials are not included.
- The database password line in the configuration screenshot is redacted in the PDF.
- No specific Apache or SSH CVE is claimed as exploited because the screenshots do not demonstrate one.
- The `web-security-academy.net` avatar screenshot was excluded as unrelated to the target evidence.

> **Authorization:** Use these steps only against the specified CTF VM or another system for which you have explicit permission.

