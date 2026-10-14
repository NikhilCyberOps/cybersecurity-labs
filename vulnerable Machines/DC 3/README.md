
# DC-3 — Vulnerability Assessment Report

## Overview
This repository contains a write-up of the DC-3 vulnerable-machine assessment performed in an isolated lab environment.

- **Victim / target IP:** `192.168.31.127`
- **Primary exposed service:** HTTP on TCP port `80`
- **Web application:** Joomla CMS
- **Reported Joomla version:** 3.7
- **Tools shown in the assessment:** Nmap, Searchsploit, SQLmap, John the Ripper, Netcat, and Linux command-line utilities

## Assessment Summary
1. Discovered the target host and identified HTTP on port 80.
2. Enumerated the website and identified Joomla CMS; the report records version 3.7.
3. Investigated a Joomla SQL-injection issue and used SQLmap to enumerate database information in the lab.
4. Recovered Joomla administrator credentials from a password hash, then accessed the administrator interface.
5. Modified a Joomla template in the lab to obtain a reverse shell as the web-server user.
6. Enumerated the Linux host and demonstrated local privilege escalation to root in the vulnerable lab.

## Finding Recorded in the Report
- **Finding:** Outdated Joomla CMS / Joomla SQL injection
- **Port:** `80/TCP`
- **Service:** Apache HTTP Server
- **Version recorded:** `2.4.18`
- **CVE listed in the source report:** `CVE-2017-8917`
- **Severity recorded in the source report:** High

The CVE, affected-version range, and severity should be independently validated against the exact Joomla build before treating them as a final professional assessment finding. Service banners and a version string alone do not prove exploitability.

## Evidence and Report
See the detailed PDF for the screenshots, commands, observations, and proof-of-concept narrative:

[**Open the DC-3 VAPT Report (PDF)**](DC-3-Report.pdf)

## Scope and Safety
This work was performed against a deliberately vulnerable machine in a controlled lab. Do not run these techniques against systems without explicit authorization. Additional IP addresses found in the source screenshots have been redacted; the victim IP `192.168.31.127` is retained.

## Skills Demonstrated
- Host and service discovery with Nmap
- Web application and CMS enumeration
- SQL injection validation and database enumeration
- Password-hash analysis
- Reverse-shell handling and Linux privilege-escalation investigation
- Evidence collection and technical reporting
