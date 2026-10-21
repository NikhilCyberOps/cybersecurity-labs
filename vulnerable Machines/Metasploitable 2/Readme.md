Penetration Testing Report — Metasploitable 2

Vulnerability Assessment & Exploitation Walkthrough

Objective

Acquire root access to the Metasploitable 2 VM — an intentionally vulnerable Ubuntu-based image built by Rapid7 for pentest training. Testing followed: recon/port scan (Nmap) → service enumeration → exploitation (Metasploit/manual) → post-exploitation verification (id/whoami/uname -a).

Executive Summary
Port	Service	Vulnerability	Severity
21	vsftpd 2.3.4	Malicious backdoor (CVE-2011-2523)	Critical
22	OpenSSH 4.7p1	Default creds msfadmin:msfadmin	High
23	Telnet	Cleartext login, weak creds	High
25	Postfix 2.5.5	VRFY user enum, open relay risk	Medium
53	BIND 9.4.2	Outdated DNS, zone transfer risk	Medium
80	Apache 2.2.8/PHP 5.2.4	DVWA, Mutillidae, TWiki, phpMyAdmin	Critical
111	rpcbind	Exposed portmapper	Medium
139/445	Samba 3.0.20-Debian	usermap_script RCE (CVE-2007-2447)	Critical
512-514	rlogin/rsh/rexec	Trust-based auth	High
1099	Java RMI	Insecure class loading RCE	Critical
1524	ingreslock	Pre-existing root backdoor	Critical
2049	NFS	Exports with no access control	Medium
3306	MySQL 5.0.51a	Root, no password	High
3632	distccd	Unauthenticated RCE (CVE-2004-2687)	Critical
5432	PostgreSQL 8.3.0	Default postgres:postgres	High
5900	VNC (RealVNC 4.1.1)	Weak password “password”	High
6667	UnrealIRCd 3.2.8.1	Trojaned backdoor (CVE-2010-2075)	Critical
8180	Apache Tomcat 5.5.9	Default tomcat:tomcat manager creds	High
Each service section includes
Narrative — what the vulnerability is and why it exists
Terminal walkthrough — actual commands used (Nmap/Metasploit/manual), styled as a dark terminal block
PoC table — Vulnerability Name / Port / Service / Version / Severity / CVE ID / Description / Remediation / PoC Steps (same 8-field layout as your report1.xlsx sample)

Example (vsftpd, page 4 of the PDF):

$ msfconsole -q
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 exploit(...) > set RHOSTS 192.168.56.102
msf6 exploit(...) > run
[+] 192.168.56.102:21 - UID: uid=0(root) gid=0(root)
[*] Command shell session 1 opened (...:4444 -> ...:6200)
id
uid=0(root) gid=0(root)

→ PoC table: CVE-2011-2523, Critical, remediation = replace trojaned binary, upgrade, restrict FTP exposure.

This pattern repeats for all 14 services (full walkthroughs for SSH, Samba, distcc, Java RMI, ingreslock, UnrealIRCd, Tomcat, MySQL, PostgreSQL, VNC, NFS, r-services, and web apps).

Conclusion

Overall risk rating: Critical — multiple independent unauthenticated remote-to-root paths exist (vsftpd, Samba, distcc, UnrealIRCd, ingreslock). Recommendations: patch/upgrade all services, remove default accounts, disable legacy r-services/telnet/X11, segment management ports, verify software integrity, and re-test after remediation.
