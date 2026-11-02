# Conti Ransomware Investigation with Splunk (TryHackMe)

**Author:** Nikhil Gowda
**Platform:** TryHackMe, *Conti* room
**Tools:** Splunk Enterprise 8.2.2, Sysmon, IIS logs
**Focus:** SOC Level 1 incident investigation, SIEM log analysis, MITRE ATT&CK mapping

---

## 1. Overview

This report documents my investigation of a simulated Conti ransomware attack against a Microsoft Exchange server. Using only Splunk, I traced the intrusion from initial access through persistence, credential access, and the final ransomware execution.

| Item | Detail |
|---|---|
| Domain | `bellybear.local` |
| Affected host | `WIN-AOQKG2AS2Q7` (Exchange Server) |
| Log sources | Sysmon (`WinEventLog:Microsoft-Windows-Sysmon/Operational`), IIS |
| Activity date in logs | 2021-09-08 |
| Threat | Conti ransomware operators |

> This is a training lab. All hostnames, accounts and credentials below belong to the lab environment.

---

## 2. Threat Background

Researchers have linked Conti operators to exploitation of the following vulnerabilities:

| CVE | Product | CVSS | Notes |
|---|---|---|---|
| CVE-2020-0796 | Microsoft (SMBGhost / CoronaBlue) | 10.0 | Critical RCE / privilege escalation, one of the top exploited vulnerabilities of 2020 |
| CVE-2018-13379 | Fortinet FortiOS | 9.8 | Critical, allows unauthenticated attacks |
| CVE-2018-13374 | Fortinet FortiOS | 8.8 | High, used to compromise web applications |

![CVE insights](screenshots/01-cve-insights.png)

---

## 3. Attack Timeline

| Stage | Evidence | Time (as shown in Splunk) |
|---|---|---|
| Web shell on Exchange | `i3gfPctK1c2x.aspx` in OWA auth directory | 9/8/21 12:52:09 PM (Splunk display time) |
| Local account creation | `net user /add securityninja` | 2021-09-08 20:04:10 UTC |
| Privilege and RDP access | Added to Administrators and Remote Desktop Users | 2021-09-08 20:04:10 to 20:04:11 UTC |
| Ransomware execution | `cmd.exe` running from Administrator's Documents folder | 2021-09-08 20:08:34 UTC |

The web shell timestamp is shown in the Splunk display timezone and the other entries are in UTC, so compare them with care.

---

## 4. Investigation Walkthrough

### 4.1 Web shell on the Exchange server

**Query**
```spl
index=* "i3gfPctK1c2x.aspx"
| highlight Commandline
```

**Finding:** 6 events. Sysmon recorded `attrib.exe` run against a suspicious `.aspx` file inside the Exchange FrontEnd OWA auth directory:

```
attrib.exe -r \\win-aoqkg2as2q7.bellybear.local\C$\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\i3gfPctK1c2x.aspx
```

- User: `NT AUTHORITY\SYSTEM`
- Parent process: `C:\Windows\system32\cmd.exe`
- Working directory: `c:\windows\system32\inetsrv\` (the IIS worker context)

A randomly named `.aspx` file in the OWA auth directory, spawned from the IIS context under SYSTEM, is a strong web shell indicator. Clearing the read-only attribute is typically done to modify or remove a file, so this may be attacker cleanup or file manipulation.

![Web shell attrib command](screenshots/02-webshell-attrib-commandline.png)

### 4.2 Confirming the web shell in IIS logs

**Query**
```spl
index=* sourcetype=iis cs_method="POST" aspx
```

**Finding:** 187 POST events. The `cs_uri_stem` field had 6 values:

| URI | Count | % |
|---|---|---|
| `/ecp/DDI/DDIService.svc/GetList` | 171 | 91.44% |
| `/owa/auth.owa` | 8 | 4.28% |
| `/owa/auth/i3gfPctK1c2x.aspx` | 4 | 2.14% |
| `/ecp/DDI/DDIService.svc/NewObject` | 2 | 1.07% |
| `/ecp/DDI/DDIService.svc/GetObject` | 1 | 0.53% |
| `/owa/lang.owa` | 1 | 0.53% |

The attacker sent 4 POST requests directly to the web shell. The heavy use of the Exchange ECP `DDIService.svc` endpoints is consistent with Exchange exploitation (ProxyShell-style) used to plant the shell.

![IIS POST requests](screenshots/03-iis-post-requests.png)

### 4.3 Process injection (Sysmon Event ID 8)

**Query**
```spl
index=* EventCode=8
| table TargetImage
| reverse
```

**Finding:** 2 CreateRemoteThread events, targeting:

- `C:\Windows\System32\wbem\unsecapp.exe`
- `C:\Windows\System32\lsass.exe`

Remote thread creation into `lsass.exe` is a common step in credential theft. Injection into a legitimate Windows binary such as `unsecapp.exe` points to defence evasion.

![CreateRemoteThread events](screenshots/04-createremotethread-eventcode8.png)

### 4.4 Persistence: new admin account

**Query**
```spl
index=* EventCode=1 net.exe
| table UtcTime CommandLine
| reverse
```

**Finding:** 6 events (3 commands, each logged for `net` and `net1`):

```
net user /add securityninja hardToHack123$
net localgroup administrators securityninja /add
net localgroup "Remote Desktop Users" "securityninja" /add
```

The attacker created a local user `securityninja`, made it a local administrator, and allowed it to log in over RDP, giving persistent remote access.

![net.exe commands](screenshots/05-net-user-persistence.png)

### 4.5 Ransom note files

**Query**
```spl
index=* EventCode=11 cmd.exe
```

**Finding:** 19 file-creation events. The `TargetFilename` field shows `readme.txt` dropped in the Downloads folder of multiple user profiles, for example:

- `C:\Users\.NET v4.5 Classic\Downloads\readme.txt`
- `C:\Users\.NET v4.5\Downloads\readme.txt`
- `C:\Users\Administrator.BELLYBEAR\Downloads\readme.txt`

The same note appearing across many profiles is typical ransomware behaviour.

![Ransom note files](screenshots/06-readme-ransom-note-files.png)

### 4.6 The ransomware binary: a fake `cmd.exe`

**Query: who created the files**
```spl
index=* EventCode=11 cmd.exe
| table UtcTime Image
```

The process writing the ransom notes is `c:\Users\Administrator\Documents\cmd.exe`, first seen at `2021-09-08 20:08:34.900` (19 events).

![File creation by fake cmd.exe](screenshots/08-fake-cmd-file-creation.png)

**Query: process creation and hashes**
```spl
index=* cmd.exe EventCode=1 Image="*\\Users\\*\\Documents\\cmd.exe"
| table Image CommandLine Hashes
```

| Field | Value |
|---|---|
| Image | `C:\Users\Administrator\Documents\cmd.exe` |
| CommandLine | `cmd.exe` |
| MD5 | `290C7DFB01E50CEA9E19DA81A781AF2C` |
| SHA256 | `53B1C1B2F41A7FC300E97D036E57539453FF82001DD3F4ABF07F4896B1F9CA22` |
| IMPHASH | `23F815785DB238377F4513BE54DBA574` |

The legitimate `cmd.exe` lives in `C:\Windows\System32`. A `cmd.exe` running from a user's Documents folder is masquerading as a system binary. The hashes above are good IOCs for threat intel lookups (for example VirusTotal).

![Fake cmd.exe hashes](screenshots/07-fake-cmd-hashes.png)

---

## 5. MITRE ATT&CK Mapping

| Tactic | Technique | Evidence |
|---|---|---|
| Initial Access | T1190 Exploit Public-Facing Application | Exchange ECP/OWA exploitation |
| Persistence | T1505.003 Server Software Component: Web Shell | `i3gfPctK1c2x.aspx` |
| Persistence | T1136.001 Create Account: Local Account | `net user /add securityninja` |
| Privilege Escalation | T1078 / group modification | Added to Administrators group |
| Lateral Movement | T1021.001 Remote Services: RDP | Added to Remote Desktop Users |
| Defense Evasion | T1055 Process Injection | CreateRemoteThread into `unsecapp.exe` and `lsass.exe` |
| Defense Evasion | T1036.005 Masquerading | `cmd.exe` in Documents folder |
| Credential Access | T1003.001 LSASS Memory | Remote thread into `lsass.exe` (likely) |
| Impact | T1486 Data Encrypted for Impact | `readme.txt` notes in 19 locations |

---

## 6. Indicators of Compromise (IOCs)

| Type | Value |
|---|---|
| Web shell | `...\FrontEnd\HttpProxy\owa\auth\i3gfPctK1c2x.aspx` |
| URI | `/owa/auth/i3gfPctK1c2x.aspx` |
| Account | `securityninja` |
| Malicious file | `C:\Users\Administrator\Documents\cmd.exe` |
| MD5 | `290C7DFB01E50CEA9E19DA81A781AF2C` |
| SHA256 | `53B1C1B2F41A7FC300E97D036E57539453FF82001DD3F4ABF07F4896B1F9CA22` |
| Ransom note | `readme.txt` (user Downloads folders) |
| Host | `WIN-AOQKG2AS2Q7` |

---

## 7. Detection and Response Recommendations

**Detection ideas**
- Alert on new `.aspx` files created under Exchange FrontEnd/OWA directories.
- Alert on `net user /add` and `net localgroup administrators ... /add` (Sysmon Event ID 1).
- Alert on Sysmon Event ID 8 where `TargetImage` is `lsass.exe`.
- Alert on `cmd.exe` and other system binary names executing from outside `C:\Windows\System32`.
- Alert on bursts of identical file creation (for example `readme.txt`) across many user profiles.

**Response and hardening**
- Isolate the affected host and remove the web shell.
- Disable and delete the `securityninja` account; reset credentials exposed on the server, since LSASS was targeted.
- Block the file hashes and hunt for them across the environment.
- Patch Exchange and edge devices (including the FortiOS and SMB CVEs listed above).
- Restrict RDP exposure and keep offline, tested backups.

---

## 8. Skills Demonstrated

- Splunk SPL: `table`, `reverse`, `highlight`, field filtering, `sourcetype` searches
- Sysmon event analysis (Event IDs 1, 8, 11)
- IIS log analysis and web shell identification
- Threat hunting across process, file and network-facing logs
- MITRE ATT&CK mapping, IOC extraction, and incident reporting

---
