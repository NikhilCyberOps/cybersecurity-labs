# TryHackMe – PS Eclipse | SOC Level 1 Investigation

## 📌 Overview

This project documents a ransomware investigation performed in the TryHackMe **PS Eclipse** room using Splunk and Windows telemetry.

The investigation focuses on encoded PowerShell execution, attempted Microsoft Defender impairment, suspicious executable downloads, scheduled-task activity, suspicious network indicators, and artifacts associated with the BlackSun ransomware script.

**Platform:** TryHackMe  
**Room:** PS Eclipse  
**Category:** SOC Analysis | SIEM | Threat Hunting | Ransomware Investigation  
**SIEM:** Splunk Enterprise  
**Primary telemetry:** Windows process, file, and network/DNS events

## 🎯 Objectives

- Investigate suspicious PowerShell command lines.
- Decode Base64-encoded commands.
- Identify potential security-control tampering.
- Investigate suspicious executable downloads.
- Analyse scheduled-task creation and execution.
- Correlate DNS activity with suspicious processes.
- Identify ransomware-related artifacts and document IOCs.
- Recommend containment, detection, and recovery measures.

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Splunk Enterprise | Search and correlate Windows event data |
| CyberChef / decoder | Decode Base64-encoded PowerShell commands |
| VirusTotal | Investigate a suspicious script hash |
| Windows Event Logs | Examine process execution and file activity |
| TryHackMe | Controlled SOC investigation lab |

---

## 🔍 Investigation Findings

### 1. Encoded PowerShell Activity

The Splunk investigation identified PowerShell execution using an encoded command.

The decoded command included an attempt to disable Microsoft Defender real-time monitoring and actions to download a suspicious executable and configure a scheduled task.

**Why this matters:** Encoded PowerShell can obscure command activity. Attempts to modify security settings should be investigated and correlated with endpoint protection logs.

### 2. Suspicious Executable Download

The suspicious binary was identified as:

```text
OUTSTANDING_GUTTER.exe
```

The investigation associated the download with an ngrok-hosted address and the Windows Temp directory.

**Observed destination:**

```text
hxxp[://]886e-181-215-214-32[.]ngrok[.]io
```

The address is defanged to reduce accidental navigation.

### 3. Scheduled-Task Activity

The command line showed creation and execution of a scheduled task named:

```text
OUTSTANDING_GUTTER.exe
```

The task was configured to run under:

```text
NT AUTHORITY\SYSTEM
```

The screenshot also shows a trigger based on Application Event ID 777.

Scheduled tasks can be used legitimately, but an unexpected task launching a binary from a writable temporary directory with SYSTEM privileges is a serious warning sign.

**Evidence limitation:** The task's executable path appears inconsistent in the screenshot. The original raw event should be checked before treating the exact path as confirmed.

### 4. Suspicious DNS Activity

The investigation identified the following DNS name associated with the suspicious binary:

```text
9030-181-215-214-32.ngrok.io
```

This may indicate an attempt to communicate with remote infrastructure. A DNS query alone does not prove that a connection succeeded or that command-and-control activity occurred.

### 5. BlackSun PowerShell Script

The investigation identified a PowerShell script initially named:

```text
C:\Windows\Temp\script.ps1
```

The supplied VirusTotal screenshot identifies the script as `BlackSun.ps1`.

**SHA-256:**

```text
E5429F2E44990B3D4E249C566FBF19741E671C0E40B809F87248D9EC9114BEF9
```

The screenshot showed 31 of 54 security vendors flagging the file as malicious at the time of the lookup. Detection results may change over time.

[View the file on VirusTotal](https://www.virustotal.com/gui/file/e5429f2e44990b3d4e249c566fbf19741e671c0e40b809f87248d9ec9114bef9)

### 6. Ransomware-Related Artifacts

The lab evidence identifies the following paths:

**Ransom note**

```text
C:\Users\keegan\Downloads\vasg6b0wmw029hd\BlackSun_README.txt
```

**Wallpaper image**

```text
C:\Users\Public\Pictures\blacksun.jpg
```

The ransom note and wallpaper modification are consistent with ransomware-related activity. The screenshots do not establish the full scope of file encryption.

---

## 🚩 Indicators of Compromise (IOCs)

| Type | Indicator |
|---|---|
| Suspicious executable | `OUTSTANDING_GUTTER.exe` |
| Download hostname | `886e-181-215-214-32.ngrok.io` |
| DNS hostname | `9030-181-215-214-32.ngrok.io` |
| Scheduled task | `OUTSTANDING_GUTTER.exe` |
| PowerShell script | `BlackSun.ps1` |
| SHA-256 | `E5429F2E44990B3D4E249C566FBF19741E671C0E40B809F87248D9EC9114BEF9` |
| Ransom note | `BlackSun_README.txt` |
| Wallpaper | `blacksun.jpg` |

Validate indicators against the original logs and your organisation's threat-intelligence sources before using them in production security controls.

## 🧪 Splunk Queries

### Search for the suspicious executable

```spl
index=* "OUTSTANDING_GUTTER.exe"
```

### Find encoded PowerShell commands

```spl
index=* powershell CommandLine="* -enc *"
```

### Investigate scheduled-task activity

```spl
index=* "schtasks.exe" "OUTSTANDING_GUTTER.exe"
```

### Search for the suspicious DNS name

```spl
index=* QueryName="9030-181-215-214-32.ngrok.io"
```

### Search for ransomware artifacts

```spl
index=* "BlackSun" OR "blacksun.jpg" OR "BlackSun_README.txt"
```

*Note: These are starter searches. Adjust index names and field names to match your lab's event data and field extractions.*

---

## 🧩 Attack Reconstruction

Based on the screenshots and walkthrough references, the likely sequence was:

1. An encoded PowerShell command was executed.
2. The command attempted to disable Microsoft Defender real-time monitoring.
3. PowerShell downloaded `OUTSTANDING_GUTTER.exe` from an ngrok-hosted address.
4. A scheduled task was configured to run the binary as SYSTEM.
5. The suspicious binary was associated with another ngrok hostname.
6. A PowerShell script was identified as `BlackSun.ps1` through hash-based investigation.
7. A ransom note and wallpaper image were found among the lab artifacts.

This sequence is consistent with a ransomware attack involving defense evasion, payload delivery, scheduled-task execution, and ransomware-related impact. Successful execution of every stage should be verified using the original endpoint and network logs.

## 🛡️ Mitigation and Recommendations

- **Containment:** Isolate an affected endpoint if compromise is confirmed and preserve relevant evidence.
- **Defender protection:** Verify real-time monitoring and tamper protection; investigate any unauthorised configuration changes.
- **Scheduled tasks:** Review task creation logs, task XML, executable paths, and execution history. Remove unauthorised tasks after preserving evidence.
- **Malware analysis:** Collect the suspicious files, calculate hashes, and analyse them only in an isolated environment.
- **Network controls:** Investigate and block validated malicious indicators through approved DNS, proxy, firewall, and EDR controls.
- **PowerShell monitoring:** Alert on encoded commands, suspicious downloads, and commands that modify Defender settings.
- **Ransomware recovery:** Determine the scope of encryption and restore affected systems from tested, known-good backups.
- **Threat hunting:** Search for related processes, file writes, scheduled tasks, DNS requests, and ransomware artifacts across other endpoints.

## 🧠 Key Learnings

- Encoded PowerShell commands can hide suspicious execution chains.
- Legitimate administrative tools can be abused by attackers.
- Scheduled tasks may provide persistence or privileged execution.
- DNS telemetry can help connect suspicious processes to remote infrastructure.
- File hashes help correlate suspicious samples with threat-intelligence reports.
- Ransomware response requires containment, evidence preservation, eradication, and recovery.
- A SOC analyst must separate confirmed evidence from inferred attack behaviour.

## 📸 Evidence and Report

The accompanying report includes the supplied screenshots, findings, PoC/validation steps, IOCs, SPL queries, attack reconstruction, and mitigation recommendations.

**[Download the PS Eclipse SOC Investigation Report (PDF)](sandbox:/mnt/data/TryHackMe_PS_Eclipse_SOC_Investigation_Report.pdf)**

Suggested repository structure:

```text
TryHackMe-PS-Eclipse/
├── README.md
├── screenshots/
│   ├── 01-encoded-powershell.png
│   ├── 02-decoded-command.png
│   ├── 03-powershell-events.png
│   ├── 04-suspicious-dns.png
│   ├── 05-virustotal-analysis.png
│   ├── 06-ransomware-artifacts.png
│   └── 07-room-completion.png
└── report/
    └── TryHackMe_PS_Eclipse_SOC_Investigation_Report.pdf
```

Upload your actual screenshots and PDF before linking them from the GitHub repository.

## 🔗 References

- [TryHackMe – PS Eclipse](https://tryhackme.com/room/posheclipse)
- [siunam — PS Eclipse Walkthrough](https://siunam321.github.io/ctf/tryhackme/PS-Eclipse/)
- [Motasem Notes — Ransomware Investigation with Splunk](https://motasem-notes.net/ransomware-investigation-with-splunk-tryhackme-ps-eclipse/)
- [Furkanctiner — PS Eclipse Walkthrough](https://medium.com/@furkanctiner/%EF%B8%8F-ps-eclipse-tryhackme-walkthrough-d691bf004286)
- [GitHub — PS Eclipse Notes](https://github.com/jesusgavancho/TryHackMe_and_HackTheBox/blob/master/PS%20Eclipse.md)

## ✅ Conclusion

The PS Eclipse lab demonstrates how a SOC analyst can use Splunk to investigate an attack involving encoded PowerShell, attempted security-control tampering, suspicious binary downloads, scheduled-task execution, and ransomware-related artifacts.

The evidence is consistent with a BlackSun ransomware chain. The priority in a real incident would be to contain affected systems, confirm execution and encryption scope, remove unauthorised persistence, block validated indicators, and recover from clean backups.

**Disclaimer:** This project is for educational and defensive cybersecurity purposes. Analyse suspicious files only in authorised, isolated lab environments.

