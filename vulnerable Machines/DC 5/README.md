# DC-5 Vulnerable Machine — VAPT Report

## Overview

This repository entry documents a penetration-testing lab against the **DC-5** vulnerable machine. The report follows the observed attack path from web enumeration to Local File Inclusion (LFI), command execution, reverse-shell access, and local privilege escalation.

- **Victim machine:** DC-5
- **Victim IP:** `192.168.31.244`
- **Web service:** HTTP on TCP port `80`
- **Web server identified in the report:** Nginx `1.6.2`
- **Primary finding:** Local File Inclusion (LFI)
- **Reported severity:** Critical

> Only the DC-5 victim IP is retained in the report. Other IP addresses have been redacted from editable text and screenshots.

## Attack Path Summary

1. **Host and service discovery** — Nmap identified the DC-5 host and the web service on port 80.
2. **Web enumeration** — The website and contact form were explored. The thank-you page showed changing copyright years, leading to further testing of its file parameter.
3. **LFI validation** — The file parameter was manipulated to read local files, including `/etc/passwd`, and the Nginx access log was identified.
4. **Command execution** — The report documents using PHP code through the vulnerable inclusion path and verifying execution with the `id` command.
5. **Reverse shell** — A Netcat listener received a shell, which was upgraded to an interactive TTY.
6. **Privilege escalation** — The SUID-enabled `/bin/screen-4.5.0` binary was investigated with Searchsploit. The report describes using the related local privilege-escalation exploit to obtain root access.
7. **Flag retrieval** — The final step was to inspect `/root` and read `thisistheflag.txt`.

## Vulnerability Details

| Field | Details |
|---|---|
| Vulnerability | Local File Inclusion (LFI) |
| Affected service | HTTP / Nginx |
| Port | `80/TCP` |
| Version noted in report | `1.6.2` |
| Severity noted in report | Critical |
| CVE | Not specified |

## Recommendations

- Validate and sanitize all user-controlled input.
- Use an allowlist for files that the application is permitted to include.
- Avoid including files based directly on user-supplied paths.
- Restrict access to sensitive files and application/server logs.
- Keep the web application and server software updated.

## Files

- `DC5_VAPT_Report.pdf` — Redacted PDF version of the detailed lab report.

## Disclaimer

This write-up is for authorized, educational lab use on the DC-5 vulnerable machine. Do not test these techniques against systems without explicit permission.

