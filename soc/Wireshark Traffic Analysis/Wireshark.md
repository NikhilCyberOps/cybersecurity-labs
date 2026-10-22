# TryHackMe — Wireshark Traffic Analysis | SOC Level 1

## Overview

A network traffic investigation using Wireshark and Nmap in an authorized lab environment.

- **Analysis host:** `192.168.31.172` — Kali Linux
- **Target:** `192.168.31.104` — Metasploitable
- **Tools:** Wireshark, Nmap, Linux CLI
- **Focus:** Service enumeration, cleartext credentials, SMB traffic, HTTP requests, and TCP analysis.

## Key Findings

| Finding | Severity | Evidence |
|---|---|---|
| FTP cleartext authentication | High | FTP stream contains USER/PASS commands and a successful login response |
| Multiple exposed services | High — provisional | Nmap identifies numerous open ports, including Telnet, SMB, bind shell and IRC |
| SMB enumeration | Medium | IPC$ connection and SRVSVC share-enumeration request |
| HTTP `/dav/` request | Informational / Medium | GET request observed; WebDAV vulnerability not confirmed |
| TCP SYN/RST activity | Informational | Connection-probing activity visible in the capture |
| ICMP connectivity | Informational | Echo requests and replies between the lab hosts |

## Wireshark Display Filters

```wireshark
ftp
http.request.method == "GET"
http.request.uri contains "/dav/"
smb || ntlmssp
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.flags.reset == 1
icmp
```

## Attack Analysis

The evidence shows target discovery, a broad exposed service surface, cleartext FTP authentication, SMB share enumeration, an HTTP request to `/dav/`, and repeated TCP SYN/RST activity.

The clearest confirmed security issue is **FTP credentials transmitted without encryption**. Anyone able to capture that session could read the credentials. The exposed legacy services increase the target's attack surface, but the supplied screenshots do not prove that SMB, HTTP, or another service was successfully exploited.

## Mitigation

1. Replace FTP with SFTP or correctly configured FTPS, and rotate exposed credentials.
2. Disable unnecessary services, especially Telnet and any unintended bind shell.
3. Restrict inbound ports using host and network firewalls.
4. Harden SMB, disable SMBv1, and enforce least-privilege share permissions.
5. Disable WebDAV if unnecessary; otherwise restrict access and write methods.
6. Correlate packet evidence with endpoint, firewall, and IDS logs before attributing malicious intent.

## Report

See `TryHackMe_Wireshark_Traffic_Analysis_SOC_Report.pdf` for the detailed findings, proof-of-concept validation, selected screenshots, limitations, and remediation recommendations.

## References

- [TryHackMe — Wireshark: Traffic Analysis](https://tryhackme.com/room/wiresharktrafficanalysis)
- [Jalblas — Wireshark Traffic Analysis Walkthrough](https://www.jalblas.com/blog/thm-wireshark-traffic-analysis-walkthrough/)
- [GitHub — Wireshark Traffic Analysis Write-up](https://github.com/veyronshade/tryhackme-writeups/blob/main/cybersecurity-101/Wireshark-Room/Wireshark%20Traffic%20Analysis.md)

**Disclaimer:** Educational defensive analysis for an authorized lab. Do not test systems without explicit permission.
