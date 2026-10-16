# VulnCMS — VAPT Report

Companion markdown version of `VulnCMS_VAPT_Report.pdf`. Screenshots referenced below are in `./screenshots/`.

> **Disclaimer:** This documents a penetration test against an intentionally vulnerable machine in a private, isolated lab for training purposes. Not a production system, not internet-facing. Only use these techniques against systems you own or are explicitly authorised to test.

## Target

| Field | Value |
|---|---|
| Hostname | `vuln_cms.lan` |
| IP Address | `192.168.31.69` |
| Objective | Acquire root access on the target machine |
| Engagement type | Black-box, isolated lab network (CTF-style exercise) |
| Result | Full compromise — root access obtained, flag captured |

## Executive Summary

The target hosted a custom web stack (nginx, WordPress, Joomla, Drupal) across five ports. Enumeration found an outdated Drupal 7 instance vulnerable to the pre-authentication SQL injection flaw known as **Drupageddon (CVE-2014-3704)**. Exploiting it gave remote code execution, which exposed a plaintext credential file enabling SSH access, and a misconfigured `sudo` rule on `journalctl` that allowed escalation to full root.

| Metric | Value |
|---|---|
| Open ports | 22, 80, 5000, 8081, 9001 |
| Critical findings | 3 |
| Medium findings | 3 |
| Low findings | 1 |
| Initial access | Drupal 7 "Drupageddon" SQLi (CVE-2014-3704) |
| Privilege escalation | NOPASSWD sudo on `/bin/journalctl` (GTFOBins) |

## 1. Host Discovery

Trimmed to the assessment target only — unrelated hosts on the local network are omitted.

```
$ nmap -sn <lab subnet>
...
Nmap scan report for vuln_cms.lan (192.168.31.69)
Host is up (0.00046s latency).
MAC Address: 00:0C:29:97:7F:C1 (VMware)
...
Nmap done: 256 IP addresses scanned
```

## 2. Port & Service Enumeration

![nmap port scan](./screenshots/01_nmap_port_scan.png)
*Screenshot 1 — `nmap -sC -sV` against 192.168.31.69*

| Port | Service |
|---|---|
| 22/tcp | OpenSSH 7.6p1 (Ubuntu) |
| 80/tcp | nginx 1.14.0, "W3.CSS Template" |
| 5000/tcp | WordPress 5.7.2, "fsociety — Just another WordPress site" |
| 8081/tcp | Joomla! CMS (15 disallowed paths in robots.txt) |
| 9001/tcp | Drupal 7, "fsociety.web" |

Manual browsing confirmed a separate web app per port. `dirb` found nothing of note beyond an `about.html` hint that brute-forcing wasn't the intended path — pointing toward CMS-specific vulnerabilities instead.

## 3. Exploitation — Drupal 7 "Drupageddon" (Port 9001)

`searchsploit drupal 7` confirmed public exploit availability. Metasploit's `exploit/multi/http/drupal_drupageddon` module — implementing SA-CORE-2014-005 / **CVE-2014-3704** — was used.

![Drupageddon module setup](./screenshots/02_drupageddon_module_setup.png)
*Screenshot 2 — RHOST/RPORT set, module options*

> **Note on CVE naming:** the module used is `drupal_drupageddon`, which is **CVE-2014-3704** ("Drupageddon"), not the similarly-named "Drupalgeddon2" RCE (CVE-2018-7600). The original draft of this report cited CVE-2018-7600; it has been corrected here.

![Meterpreter sysinfo](./screenshots/03_meterpreter_sysinfo.png)
*Screenshot 3 — sysinfo confirming a Meterpreter session on host "vuln_cms"*

## 4. Post-Exploitation — Credential Discovery

Directory listing revealed an unusual `misc` directory containing a plaintext `tyrell.pass` file.

![tyrell.pass](./screenshots/04_tyrell_pass_credential_file.png)
*Screenshot 4 — plaintext username/password pair*

## 5. Lateral Movement — SSH Access

![SSH login](./screenshots/05_ssh_login_as_tyrell.png)
*Screenshot 5 — SSH login as "tyrell" using the recovered credentials*

## 6. Privilege Escalation — Misconfigured Sudo

```
tyrell@vuln_cms:~$ sudo -l
...
User tyrell may run the following commands on vuln_cms:
    (root) NOPASSWD: /bin/journalctl
```

![sudo -l](./screenshots/06_sudo_l_journalctl.png)
*Screenshot 6 — NOPASSWD journalctl*

`journalctl` is a documented GTFOBins sudo privilege-escalation primitive: its pager supports a shell escape via `!/bin/sh`.

![GTFOBins root shell](./screenshots/07_gtfobins_root_shell.png)
*Screenshot 7 — `!/bin/sh` inside the journalctl pager returns a root shell*

## 7. Flag Capture

![root.txt](./screenshots/08_root_flag.png)
*Screenshot 8 — contents of root.txt*

```
root.txt: 4359537020406305
```

## 8. Findings

| Port | Finding | Service | Severity | CVE |
|---|---|---|---|---|
| 9001 | Drupal 7 RCE ("Drupageddon" SQL Injection) | HTTP (Drupal) | **Critical** | CVE-2014-3704 |
| 9001 (post-exploit) | Sensitive Credential File Exposure (`tyrell.pass`) | Filesystem | **Critical** | N/A |
| 22 (local) | Privilege Escalation via Misconfigured Sudo (journalctl) | SSH / sudo | **Critical** | N/A |
| 80 | HTTP Information Disclosure | HTTP | Medium | N/A |
| 5000 | WordPress Version Disclosure | HTTP (WordPress) | Medium | N/A |
| 8081 | Joomla CMS Exposed | HTTP (Joomla) | Medium | N/A |
| 22 | SSH Service Detected | SSH | Low | N/A |

See `VulnCMS_VAPT_Report.pdf` for the full per-finding detail (description, remediation, PoC steps) in the template format matched to `report1.xlsx`.

## 9. Recommendations

- Patch/replace the Drupal 7 install — it's end-of-life; migrate and apply all outstanding advisories.
- Never store credentials in plaintext in app-accessible directories; use a secrets manager, rotate `tyrell`'s credential.
- Audit all `sudo` rules against GTFOBins; remove NOPASSWD access to binaries with known shell-escape primitives.
- Suppress version banners on nginx/WordPress/Joomla; remove unused CMS instances.
- Re-test after remediation.

## 10. Conclusion

The host was fully compromised end-to-end: unauthenticated RCE on Drupal 7 → plaintext credential exposure → root via sudo misconfiguration. Each step is independently well-documented and independently remediable.

---

### A note on redaction

Per request, this report keeps only the victim's IP/hostname (`192.168.31.69` / `vuln_cms.lan`). The original `nmap -sn` sweep also enumerated other devices on the home network (a router, a set-top box, a personal laptop, the attacking Kali box) — those entries have been removed from both the PDF and this file, since they aren't part of the target and reveal personal network details. No personal account credentials (email, etc.) belonging to Nikhil Gowda appeared anywhere in the source screenshots — the only credential pair found (`tyrell` / recovered password) is the CTF target's own local account, which is kept since it's the core of the PoC chain.
