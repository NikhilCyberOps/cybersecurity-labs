# TryHackMe – Shadow Trace | SOC Level 1

## 📌 Overview

This project documents my investigation of suspicious activity in the TryHackMe **Shadow Trace** room. The investigation focuses on analysing security alerts, inspecting a suspicious Windows executable, extracting Indicators of Compromise (IOCs), and understanding potential malware delivery behaviour.

**Platform:** TryHackMe  
**Room:** Shadow Trace  
**Category:** SOC / Blue Team / Threat Hunting  
**Tools Used:** PE-bear, CyberChef, PowerShell, Windows security logs and alert evidence

---

## 🎯 Objectives

- Investigate suspicious activity reported by security alerts.
- Analyse executable metadata and imported libraries.
- Calculate and document the SHA-256 hash of a suspicious file.
- Extract suspicious URLs and domains from available evidence.
- Decode encoded indicators using CyberChef.
- Recommend detection, containment, and remediation measures.

---

## 🔍 Investigation Findings

### 1. Suspicious Executable Analysis

The file `windows-update.exe` was inspected using PE-bear.

**Evidence observed:**
- PE file header identifies the executable as **64-bit AMD64**.
- The file contains suspicious URL-related strings.
- The imported library `WS2_32.dll` is present, which provides Windows networking functionality. Its presence alone does not prove malicious behaviour.

### 2. SHA-256 Hash Identification

The SHA-256 hash was calculated using PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 .\windows-update.exe
```

**SHA-256:**

```text
B2A88DE3E3BCFAE4A4B38FA36E884C586B5CB2C2C283E71FBA59EFDB9EA64BFC
```

This hash can be used to identify the same file in endpoint investigations and authorised threat-intelligence searches.

### 3. Suspicious URL and Domain Analysis

The investigation identified the following indicators in the available evidence and walkthrough material:

| Indicator | Description |
|---|---|
| `tryhatme.com` | Domain associated with a suspicious update-related URL |
| `responses.tryhatme.com` | Domain identified during the investigation |
| `reallysecureupdate.tryhatme.com` | Domain associated with a decoded executable-download URL |
| `https://tryhatme.com/dev/main.exe` | URL decoded from Base64 in the walkthrough evidence |
| `https://reallysecureupdate.tryhatme.com/update.exe` | URL revealed through decimal-to-text decoding |
| `http://tryhatme.com/update/security-update.exe` | URL string observed during executable inspection |

**Important:** These are investigation indicators, not proof that every URL was successfully contacted or that every payload was executed.

### 4. Encoded Data Analysis

CyberChef was used to decode suspicious encoded content into readable text. The analysis revealed an executable-download URL associated with `reallysecureupdate.tryhatme.com`.

Encoded URLs can conceal indicators from casual inspection. SOC analysts should preserve the original data, document the decoding method, and validate decoded results against supporting logs.

### 5. Potential Attack Behaviour

Based on the available evidence, the activity is consistent with a possible suspicious software-update or payload-delivery chain:

1. Security alerts identify suspicious activity.
2. The analyst inspects the executable and its metadata.
3. Static strings reveal update-related URLs.
4. Encoded content is decoded to expose additional indicators.
5. The analyst records the IOCs and recommends investigation and containment.

This is a **tentative reconstruction**. The available evidence does not independently establish successful execution, persistence, privilege escalation, or data exfiltration.

---

## 🛡️ Recommended Mitigation

- Isolate a potentially compromised endpoint according to the incident-response procedure.
- Preserve the suspicious executable, its hash, relevant alerts, and system logs.
- Search endpoint, DNS, proxy, firewall, and EDR telemetry for the identified domains and URLs.
- Block confirmed malicious indicators through approved security controls, after validation.
- Review process execution and network connection telemetry to determine whether the executable ran or contacted external infrastructure.
- Scan the endpoint with approved security tools and investigate persistence mechanisms if execution is confirmed.
- Reimage or restore the endpoint if compromise is established and remediation requires it.
- Educate users about suspicious update prompts and untrusted downloads.

## 📊 SOC Analyst Skills Demonstrated

- Security alert investigation
- Static executable inspection
- SHA-256 file hashing
- IOC extraction and documentation
- Base64 and decimal-to-text decoding
- Suspicious URL and domain analysis
- Incident-response recommendations
- Evidence-based reporting

## 📸 Evidence

Screenshots documenting the investigation are included in the accompanying report:

**[Download the Shadow Trace SOC Analysis Report](sandbox:/mnt/data/TryHackMe_Shadow_Trace_SOC_Analysis_Report.pdf)**

The report includes the supplied screenshots, evidence references, findings, IOCs, analysis, and mitigation recommendations.

## 📝 Conclusion

The Shadow Trace investigation demonstrates how a SOC analyst can combine alert evidence, executable metadata, file hashing, string inspection, and encoded-data analysis to identify suspicious indicators and develop an investigation plan.

The key lesson is to distinguish **observed evidence from assumptions**: suspicious strings and download URLs warrant investigation, but confirming compromise requires corroborating endpoint and network telemetry.

## 🔗 References

- TryHackMe room: https://tryhackme.com/room/shadowtrace
- Walkthrough reference: https://audittrsi.blogspot.com/2026/02/tryhackme-shadow-trace-writeup.html
- Additional walkthrough: https://hackmd.io/K2N_mUPDSraZKb4nuryQ3w

---

**Disclaimer:** This documentation is for educational and defensive cybersecurity purposes. All analysis should be performed in authorised lab environments.
