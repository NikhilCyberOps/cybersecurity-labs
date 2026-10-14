# DC-6 – Capture The Flag (CTF) | VAPT Report

## 📌 Overview
This repository documents my penetration testing walkthrough of the **DC-6 vulnerable machine**, covering reconnaissance, web application enumeration, initial access, and privilege escalation.

The assessment was performed in a controlled lab environment for educational purposes.

## 🎯 Target Information

- **Target:** DC-6
- **Victim IP:** `192.168.31.164`
- **Operating System:** Linux
- **Web Server:** Apache
- **CMS:** WordPress 5.1.1
- **Open Ports:** 22 (SSH), 80 (HTTP)

## 🔍 Methodology

### 1. Network Reconnaissance
Identified the DC-6 target machine using Nmap and confirmed that SSH and HTTP services were accessible.

### 2. Port and Service Enumeration
Performed service detection to identify the services and versions running on the target.

### 3. Web Application Enumeration
- Identified the WordPress website.
- Used WPScan to enumerate WordPress users.
- Identified the Activity Monitor plugin as a potential attack surface.

### 4. Initial Access
- Investigated the WordPress login interface.
- Performed password testing against identified accounts in the lab.
- Investigated the vulnerable Activity Monitor plugin.
- Obtained a reverse shell on the target machine.

### 5. Privilege Escalation
- Investigated local users and permissions.
- Identified a script that could be executed as another user.
- Enumerated sudo privileges.
- Identified Nmap as a potential privilege-escalation vector.
- Achieved root access in the lab environment.

## 🚨 Vulnerabilities Identified

### 1. WordPress Activity Monitor Plugin Remote Code Execution
- **Severity:** Critical
- **CVE:** CVE-2018-7422 (as recorded in the report)
- **Impact:** Potential remote command execution and unauthorized system access.

### 2. Weak Credentials and Password Reuse
- **Severity:** Medium
- **CVE:** N/A
- **Impact:** Credentials obtained from the WordPress application enabled SSH access.

### 3. Privilege Escalation Through Sudo Misconfiguration
- **Impact:** Excessive permissions associated with Nmap enabled escalation to root.

## 🛡️ Recommendations

- Upgrade WordPress to a supported, secure version.
- Remove or update vulnerable plugins.
- Enforce strong, unique passwords for every account.
- Enable multi-factor authentication where supported.
- Restrict repeated login attempts.
- Disable SSH password authentication where practical and use SSH keys.
- Review sudo permissions and apply the principle of least privilege.
- Monitor authentication events and suspicious processes.
- Regularly conduct vulnerability assessments and patch management.

## 🧰 Tools Used

- Kali Linux
- Nmap
- WPScan
- SearchSploit
- SSH
- Linux command-line utilities

## 📄 Report

The detailed walkthrough, evidence screenshots, and vulnerability findings are available in:

**[DC-6 VAPT Report.pdf](./DC-6-VAPT-Report.pdf)**

## ⚠️ Disclaimer

This documentation is intended for educational purposes and authorized security testing only. Do not perform these techniques against systems without explicit permission.

## 👨‍💻 Author

**Nikhil Gowda**  
Cybersecurity Analyst | Aspiring SOC Analyst
