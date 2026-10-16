AIWeb CTF — VAPT Write-up
Target
- Victim hostname: aiweb2host.lan
- Victim IP: 192.168.31.175
- Environment: Authorized CTF / vulnerable lab VM
- Evidence: Five screenshots supplied for this report
Privacy note: unrelated discovered hosts/IPs were removed from the host-discovery screenshot. Personal credentials belonging to Nikhil Gowda are not included. The lab password hash shown in evidence is deliberately not reproduced.

Summary
The evidence shows two open TCP ports:
- 22/tcp — SSH: OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
- 80/tcp — HTTP: Apache web server; page title indicates a File Manager application
The primary confirmed issue is path traversal / local file disclosure in viewing.php. The supplied screenshots show the endpoint reading /etc/passwd and /etc/apache2/.htpasswd. No successful SSH login, password cracking, remote code execution, or privilege escalation is demonstrated.
Findings
F-01 — Path traversal / Local File Inclusion (LFI)
Severity: High
Evidence: A file_name parameter containing parent-directory traversal segments is used to access /etc/passwd. The response shows local Linux account entries.
PoC (abbreviated):
http://192.168.31.175/viewing.php?file_name=[repeated ../ traversal]/etc/passwd
Impact: May disclose system files, usernames, configuration, and secrets readable by the web-server process.
Remediation:
- Do not accept arbitrary filesystem paths from request parameters.
- Map fixed IDs to allowlisted files.
- Canonicalize paths and ensure resolved files remain within a dedicated content directory.
- Run the web process with least privilege.
- Add regression tests for traversal attempts.
F-02 — Apache password-file disclosure
Severity: High
Evidence: The same file-viewing behavior returns content from /etc/apache2/.htpasswd. The hash is intentionally omitted from this README.
Impact: Exposed hashes may be subjected to offline guessing; any associated or reused secret should be treated as compromised.
Remediation: Restrict file permissions, remove sensitive files from web-readable paths, rotate the affected secret, invalidate reused credentials, and fix the LFI.
F-03 — SSH exposed on TCP/22
Severity: Informational–Medium (configuration-dependent)
Evidence: Nmap identified OpenSSH 7.6p1 Ubuntu 4ubuntu0.3. The screenshots do not prove a successful login or a specific SSH exploit.
Remediation: Patch OpenSSH, prefer key-based authentication, disable direct root login, restrict source IPs, and monitor authentication logs.
F-04 — HTTP exposed on TCP/80
Severity: Medium when considered with the confirmed LFI
Evidence: Nmap identifies HTTP/Apache and a File Manager title. Gobuster discovers application paths such as index.php, signup.php, logout.php, css/, srv/, webadmin, and server-status, with varied HTTP status codes.
Interpretation: 200 indicates a successful response; 301/302 indicates redirection; 401/403 indicates authentication or access restrictions. These codes alone do not establish a vulnerability.
Remediation: Remove unused routes, restrict admin/status endpoints, disable directory listing, and enforce authorization server-side.
F-05 — Potentially sensitive paths discoverable
Severity: Medium pending validation
Evidence: Gobuster reports dotfiles and administrative paths, some returning 403. A 403 response does not prove file contents are exposed.
Remediation: Keep dotfiles/backups outside the document root, audit deployment artifacts, and restrict sensitive endpoints.
Reproduction workflow (authorized lab only)
1. Discover the target on the isolated lab network.
2. Enumerate services:nmap -sV -sC -p- 192.168.31.175
3. Enumerate common web paths:gobuster dir -u http://192.168.31.175:80 -w /usr/share/wordlists/dirb/big.txt -x php
4. In the lab, verify the file-viewing endpoint using a harmless test file first. Do not access systems or files outside the authorized target.
5. Capture the HTTP request and response, redact secrets, and document the remediation retest.
Evidence included in PDF
The accompanying AIWeb_CTF_VAPT_Report.pdf includes the supplied screenshots. The host-discovery image is redacted to remove unrelated hosts/IPs while retaining the victim target.
Limitations
This report is based on screenshots rather than a fresh assessment. Credential validity, CVE applicability, exploitability beyond file disclosure, and privilege escalation were not independently verified.
Disclaimer
For educational use only. Test only systems you own or have explicit permission to assess.
