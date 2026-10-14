DarkHole V2 — VAPT Report
Overview

This report documents a black-box Vulnerability Assessment and Penetration Test (VAPT) performed against the DarkHole V2 boot2root virtual machine.

Target system: DarkHole V2 (hostname: darkhole.lan)
Target IP address: 192.168.31.242
Assessment window: 21 Jul 2026 – 25 Jul 2026
Prepared by: Nikhil Gowda (Cybersecurity Analyst / Fresher)

All IP addresses, hostnames and MAC addresses belonging to machines other than the target — including the tester's own attack host and unrelated devices seen during network discovery — have been redacted from the report (shown blacked-out in screenshots, or omitted). Only the victim's IP (192.168.31.242) and name (DarkHole V2 / darkhole.lan) are retained throughout.

Attack Chain Summary
Recon — Nmap host discovery + service/version scan identified the target and flagged an exposed http-git endpoint on port 80.
Web enumeration — Gobuster revealed dashboard.php, login.php, config/, etc.
Source disclosure — git-dumper pulled the full .git repository, exposing source code and commit history.
Credential leak — git diff against the history showed a previously hardcoded admin credential later replaced with (still-vulnerable) DB-backed login.
SQL Injection — dashboard.php?id= was injectable (time-based blind + UNION); sqlmap dumped the users and ssh tables, including plaintext passwords.
Initial access — SSH'd in using the dumped SSH credentials, landing as a low-privileged user.
Lateral movement — Local enumeration found a second account and a root-owned cron job running an internal PHP service (/opt/web, port 9999, localhost-only) with an unsanitised cmd parameter passed to system().
RCE — Exploited the command injection to get code execution as the second local account.
Privilege escalation — Escalated to root and confirmed full compromise via the root flag.
Findings (by severity)
#	Finding	Port/Service	Severity
1	Exposed .git repository (source code & history disclosure)	80/tcp (HTTP)	High
2	Hardcoded credentials recoverable via git history	80/tcp (HTTP)	High
3	SQL Injection in dashboard.php (id parameter)	80/tcp (HTTP)	Critical
4	Plaintext password storage in database	80/tcp / MySQL	Medium
5	Outdated OpenSSH 8.2p1 exposed	22/tcp (SSH)	Low
6	OS Command Injection in internal PHP service	9999/tcp (localhost-only)	Critical
7	Privilege escalation to root	Local / OS	Critical

Each finding in the PDF includes a description, proof-of-concept screenshots (redacted of non-target IPs/hosts), and remediation guidance — covering SSH exposure, the exposed git repo, the SQL injection, the plaintext credentials, the command injection, and the final privilege escalation to root.

The report also includes a Section 6 — Vulnerability Details, with one Field/Details table per finding (PORT NO, SERVICE NAME, SEVERITY, VULNERABILITY NAME, CVE ID, DESCRIPTION, REMEDIATION, PROOF OF CONCEPT) for quick reference, alongside the full narrative walkthrough with screenshots in Section 5.

Files in this delivery
DarkHole_V2_VAPT_Report.pdf — Full VAPT report with executive summary, methodology, findings table, and per-vulnerability PoC walkthroughs built from the supplied screenshots.
README_DarkHole_V2.md — This summary.
Notes / Limitations
Screenshot redaction was done by blacking out or cropping out the non-target IPs/hostnames/MAC addresses found in the original screenshots (e.g. the tester's Kali host IP, other LAN devices seen during the nmap -sn sweep, and a stray prior-session IP). A couple of names/emails that appeared as part of the discovered evidence itself (e.g. a git commit author, and a dumped database record) were left in place since they are part of the vulnerability proof, not incidental network noise — let me know if you'd like those scrubbed too.
The final jump from the second local account to root was not fully captured step-by-step in the supplied screenshots (only the resulting root shell/flag was shown), so the report notes that the exact escalation technique wasn't documented in detail rather than guessing at it.
