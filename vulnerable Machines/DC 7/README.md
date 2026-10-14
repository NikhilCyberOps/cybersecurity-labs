# DC:7 — VAPT Report

## Overview
This report documents a Vulnerability Assessment and Penetration Test (VAPT) performed against the **DC:7** boot2root virtual machine (VulnHub).

- **Target system:** DC7
- **Target IP address:** `192.168.31.107`
- **Assessment date:** 03/07/2026
- **Prepared by:** Nikhil Gowda (Cybersecurity Analyst / Fresher)

> Note: all non-target IP addresses referenced in the walkthrough (attacker host, listener addresses, etc.) have been redacted to `XXX.XXX.XXX.XXX` in this version of the report. Only the victim machine's IP is retained.

## Methodology / Walkthrough Summary
1. **Reconnaissance** — Aggressive Nmap scan against the target to identify open ports and running services.
2. **Enumeration** — Explored the HTTP service; the DC:7 welcome page hinted at looking "outside the box," pointing to a GitHub account (`@DC7User`). A public `staffdb` repository exposed Drupal database credentials (`dc7user` / `MdR3xOgB7#dW`) in `config.php`.
3. **Initial Access** — Used the leaked credentials to SSH into the target as `dc7user`.
4. **Post-Exploitation** — Found an inbox message (`mbox`) referencing a cron-run backup script, `/opt/scripts/backup.sh`, which invoked Drush (the Drupal CLI). Used Drush to reset the Drupal admin password.
5. **Web Application Exploitation** — Logged into the Drupal admin console and installed a malicious PHP module (`Manage → Extend → Install new module`) to obtain a reverse shell as `www-data`.
6. **Privilege Escalation** — `www-data` had write access to `/opt/scripts/backup.sh`. Used `msfvenom` to generate a malicious payload, overwrote the script, and caught a root shell via a Netcat listener once the script was triggered.
7. **Result** — Full root compromise of the target; final flag captured.

## Findings

| # | Port | Service | Vulnerability | Severity |
|---|------|---------|----------------|----------|
| 1 | 22 | SSH (OpenSSH 7.4p1) | Outdated SSH version / exposed port configuration | Medium |
| 2 | 80 | HTTP (Apache 2.4.25, Drupal) | Authenticated Drupal RCE via malicious PHP module upload | Critical |

### Finding 1 — Outdated OpenSSH (Medium)
- **Description:** Target runs OpenSSH 7.4p1, an outdated version potentially affected by disclosed vulnerabilities (information disclosure, user enumeration, etc.).
- **Remediation:** Upgrade OpenSSH to the latest supported version; apply vendor security patches; disable legacy protocols/weak ciphers; restrict access via firewall/allowlists; prefer key-based authentication.

### Finding 2 — Drupal Remote Code Execution via Module Upload (Critical)
- **Description:** With valid administrator credentials, the Drupal application allowed installation of a new PHP module, which was abused to upload and execute arbitrary PHP code, resulting in a reverse shell as `www-data`.
- **Proof of Concept:** Logged into `/user/login` with admin credentials → `Manage → Extend → Install New Module` → uploaded malicious PHP module → triggered payload → obtained shell as `www-data`.
- **Remediation:** Keep Drupal core/modules up to date; restrict module installation to trusted admins only; disable PHP execution in upload directories; enforce strong passwords + MFA for admin accounts; audit administrative actions; apply least privilege to web server permissions.

## Files in this delivery
- `DC_7.pdf` — Full VAPT report (converted from the original Word document, with non-victim IP addresses redacted from the text).
- `README.md` — This summary.

## Notes / Limitations
- The redaction above covers the **text** of the report only. The report also contains terminal/browser **screenshots** (embedded images) that may still visually show the original attacker/listener IP addresses and other local network hosts, since those are baked into the image pixels rather than the document's text. Let me know if you'd like those screenshots redacted (blacked out) as well — that requires editing each image individually.
