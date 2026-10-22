# TryHackMe – The Greenholt Phish | SOC Investigation

## 📌 Overview

This project documents a phishing email investigation completed in the TryHackMe **The Greenholt Phish** room.

A sales executive at Greenholt PLC received an unexpected email requesting a money transfer. The message contained an unsolicited attachment and unusual wording, raising suspicion of a phishing attempt.

The objective was to analyse the email, validate the sender information, inspect authentication records, investigate the sending infrastructure, and identify the attachment's true file type.

**Platform:** TryHackMe  
**Room:** The Greenholt Phish  
**Category:** SOC Analysis | Phishing Investigation | Email Forensics  
**Difficulty:** Easy

[Open the TryHackMe room](https://tryhackme.com/room/phishingemails5fgjlzxc)

---

## 🎯 Investigation Objectives

- Analyse the suspicious email and its headers.
- Identify the sender, reply address, and originating IP.
- Investigate the sender infrastructure using WHOIS.
- Review SPF and DMARC records.
- Identify the suspicious attachment and calculate its SHA-256 hash.
- Use VirusTotal to investigate the file's reputation and actual file type.
- Document findings and recommend appropriate incident-response actions.

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Thunderbird | View and inspect the `.eml` email sample |
| Email source/header analysis | Examine sender details, routing information, and authentication results |
| WHOIS lookup | Investigate the originating IP address and its registered organisation |
| SPF lookup | Review the sending domain's authorised mail servers |
| DMARC lookup | Review the domain's email authentication policy |
| Linux terminal | Calculate the attachment's SHA-256 hash |
| VirusTotal | Check file reputation, reported size, and actual file type |

---

## 🔎 Investigation Findings

### 1. Email and Sender Analysis

The email contained several suspicious characteristics:

- An unexpected request involving a money transfer.
- A generic greeting that did not match the recipient's expectations.
- An attachment that the recipient had not requested.
- A reply address that differed from the sender's displayed email address.

These indicators warranted further investigation rather than trusting the displayed sender name alone.

### 2. Email Header and Infrastructure Analysis

The email headers were reviewed to identify the sender address, reply address, originating IP, and delivery information.

| Artifact | Finding |
|---|---|
| Sender display name | `Mr. James Jackson` |
| Sender email | `info@mutawamarine.com` |
| Reply email | `info.mutawamarine@mail.com` |
| Originating IP | `192.119.71.157` |
| IP organisation | `Hostwinds LLC` |

The differing sender and reply addresses are a warning sign. They do not independently prove spoofing, but they should be investigated alongside the message headers and authentication results.

### 3. SPF and DMARC Analysis

The following records were identified in public walkthrough references for the lab:

**SPF record**
```text
v=spf1 include:spf.protection.outlook.com -all
```

**DMARC record**
```text
v=DMARC1; p=quarantine; fo=1
```

SPF specifies which sending servers are authorised by a domain. DMARC defines how receiving systems should handle messages that fail the domain's authentication alignment policy.

**SOC takeaway:** Interpret SPF, DKIM, and DMARC results in the context of the complete message headers. A published DNS record alone does not establish whether the individual email passed authentication.

### 4. Attachment Analysis

The attachment was named:

```text
SWT_#09674321____PDF__.CAB
```

The filename suggested a PDF-related document, but the extension alone could not establish the actual file type.

The SHA-256 hash was calculated using Linux:

```bash
sha256sum 'SWT_#09674321____PDF__.CAB'
```

**SHA-256 hash**
```text
2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f
```

### 5. VirusTotal Investigation

The hash was searched in VirusTotal to investigate the file's reputation and metadata.

| Attribute | Finding |
|---|---|
| Reported file size | `400.26 KB` |
| Actual file type | RAR archive |
| Filename appearance | PDF-related |
| Security concern | Archive disguised with a misleading filename |

The discrepancy between the filename and actual file type is a significant indicator for further investigation. A file's real type should be validated using file signatures and analysis tools, not just its extension.

[Search the SHA-256 hash on VirusTotal](https://www.virustotal.com/gui/file/2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f)

*Note: The hash, size, and file-type findings above are documented in public walkthrough references for this challenge. Detection verdicts can change over time.*

---

## 🚩 Indicators of Compromise (IOCs)

| Type | Value |
|---|---|
| Sender email | `info@mutawamarine.com` |
| Reply email | `info.mutawamarine@mail.com` |
| Originating IP | `192.119.71.157` |
| Attachment | `SWT_#09674321____PDF__.CAB` |
| SHA-256 | `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f` |

These indicators should be validated against the original lab evidence before being used in production security controls.

---

## 🛡️ Recommended Mitigation and Response

1. **Contain the email:** Quarantine the message and prevent other users from receiving or opening the same attachment.
2. **Preserve evidence:** Retain the original `.eml` file, complete headers, attachment hash, and analysis notes.
3. **Search for exposure:** Search mailboxes and endpoint telemetry for the attachment hash, filename, sender, and related message indicators.
4. **Investigate execution:** Check endpoint process, file, and network logs to determine whether the attachment was opened or executed.
5. **Validate the archive:** Analyse the file in an isolated, authorised malware-analysis environment. Do not execute it on a normal workstation.
6. **Review email authentication:** Assess SPF, DKIM, and DMARC results together with the message's routing information.
7. **Block confirmed malicious indicators:** Apply appropriate mail gateway, endpoint, DNS, or network controls after validation.
8. **Escalate if necessary:** If execution or compromise is confirmed, follow the organisation's incident-response process, including endpoint containment and credential resets where justified.

---

## 🧠 Key Learnings

- Email display names are not sufficient to verify a sender's identity.
- A different reply address can indicate possible impersonation or fraud.
- Email headers provide valuable evidence about message origin and delivery.
- SPF and DMARC records help analysts evaluate email authentication.
- SHA-256 hashes support file identification and threat-intelligence searches.
- File extensions can be misleading; verify the actual file type.
- VirusTotal provides useful context, but its verdict should be combined with other evidence.
- SOC conclusions should distinguish confirmed observations from assumptions.

## 📸 Evidence and Documentation

Add your own screenshots from the lab to this repository, such as:

- The suspicious email and its subject.
- Sender and reply-address details.
- Email headers and originating IP.
- SPF and DMARC lookup results.
- Attachment filename and SHA-256 calculation.
- VirusTotal file details.

Suggested folder structure:

```text
Greenholt-Phish/
├── README.md
├── screenshots/
│   ├── 01-suspicious-email.png
│   ├── 02-email-headers.png
│   ├── 03-authentication-records.png
│   ├── 04-attachment-hash.png
│   └── 05-virustotal-analysis.png
└── report/
    └── phishing-analysis-report.pdf
```

Only include screenshots and a PDF report that you have actually saved to your repository.

## ✅ Conclusion

The Greenholt Phish investigation demonstrates a structured approach to phishing triage: examine the message, inspect the headers, investigate sender infrastructure, review email authentication, calculate the attachment hash, and validate the file's true type.

The unsolicited money-transfer request, inconsistent reply address, and misleading attachment filename are suspicious indicators. Together, they support treating the email as a potential phishing attempt and investigating further before deciding the scope of any incident.

## 🔗 References

- [TryHackMe – The Greenholt Phish](https://tryhackme.com/room/phishingemails5fgjlzxc)
- [Walkthrough – The Greenholt Phish (Jalblas)](https://www.jalblas.com/blog/tryhackme-the-greenholt-phish-walkthrough-soc-level-1/)
- [VirusTotal – File hash lookup](https://www.virustotal.com/gui/file/2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f)

---

**Disclaimer:** This project is for educational and defensive cybersecurity purposes. Analyse suspicious files only in authorised, isolated lab environments.

