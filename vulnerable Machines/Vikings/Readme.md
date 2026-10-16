# Vikings CTF — VAPT Report

## Target Information

- **Victim Hostname:** `vikings.lan`
- **Victim IP Address:** `192.168.31.51`
- **Environment:** Authorized, intentionally vulnerable CTF lab
- **Attacker Machine:** Kali Linux

## Overview

This project documents the security assessment of the Vikings CTF machine using Nmap, Gobuster, Nikto, Curl, John the Ripper, SSH, and Linux privilege-escalation checks.

The assessment identified exposed web services, information disclosure, directory indexing, a publicly accessible clue file, an encrypted archive with a weak password, SSH access, and successful privilege escalation to root.

The findings and proof-of-concept (PoC) steps are based on the supplied screenshots. Vulnerabilities are documented according to the available evidence.

## Tools Used

- Nmap — Host discovery and service enumeration
- Gobuster — Web directory and file discovery
- Nikto — Web server security checks
- Curl — HTTP response and file validation
- John the Ripper — Archive password recovery
- SSH — Remote access validation
- Linux utilities — SUID enumeration and privilege-escalation validation

## Confirmed Open Ports

| Port | Service | Detected Version |
|---|---|---|
| 22/TCP | SSH | OpenSSH 7.6p1 Ubuntu |
| 80/TCP | HTTP | Apache httpd 2.4.29 Ubuntu |

## Vulnerability Findings

### 1. SSH Service Exposure

**Port:** 22/TCP  
**Severity:** Informational / Low, depending on access controls

**PoC:**

```bash
nmap -sV -p 22 192.168.31.51
ssh floki@192.168.31.51
```

**Evidence:** The screenshots demonstrate an SSH service and successful login using the CTF account.

**Remediation:**
- Restrict SSH access to trusted sources.
- Use SSH keys and disable password authentication where practical.
- Apply supported security updates.
- Enforce strong authentication and login-rate limiting.

### 2. Apache Version Disclosure

**Port:** 80/TCP  
**Severity:** Informational

**PoC:**

```bash
nmap -sV -p 80 192.168.31.51
curl -I http://192.168.31.51/
```

**Evidence:** Nmap identifies Apache httpd 2.4.29 on Ubuntu.

**Remediation:**
- Apply supported vendor security updates.
- Minimize unnecessary server-banner information.
- Review enabled modules and server configuration.

### 3. Missing HTTP Security Headers

**Port:** 80/TCP  
**Severity:** Low to Medium, depending on application context

**PoC:**

```bash
nikto -h http://192.168.31.51
curl -I http://192.168.31.51/
```

**Evidence:** Nikto reports missing `X-Frame-Options` and `X-Content-Type-Options` headers.

**Remediation:**
- Configure an appropriate anti-clickjacking policy, preferably using CSP `frame-ancestors`.
- Set `X-Content-Type-Options: nosniff`.
- Review other security headers based on application requirements.

### 4. Directory Indexing and Content Exposure

**Port:** 80/TCP  
**Affected Path:** `/site/`

**PoC:**

```bash
gobuster dir \
  -u http://192.168.31.51:80 \
  -w /usr/share/wordlists/dirb/big.txt

curl -i http://192.168.31.51/site/
```

**Evidence:** Directory enumeration identifies `/site/`, and the browser screenshot shows its content.

**Remediation:**
- Disable directory listing.
- Remove unnecessary files from the web root.
- Restrict access to sensitive resources.

### 5. Publicly Accessible Clue File

**Port:** 80/TCP  
**Affected Path:** `/site/war.txt`

**PoC:**

```bash
curl -i http://192.168.31.51/site/war.txt
```

**Evidence:** The
