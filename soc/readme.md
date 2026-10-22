# Snapped Phish-ing Line — SOC Level 1 (TryHackMe)

## Summary
Investigation report for the TryHackMe room **Snapped Phish-ing Line** (SOC Level 1 path,
Phishing module). Walks through analyzing a phishing email impersonating an Office 365
"update" notice, pivoting to the attacker's exposed web directory, recovering the phishing
kit, and decoding the room's hidden flag.

- **Difficulty:** Easy
- **Type:** Challenge
- **Scenario:** IT staff at fictitious "SwiftSpend Financial" investigate a phishing report
- **Flag:** `THM{pL4y_w1Th_tH3_URL}`

## What's in the report
1. Room overview & objective
2. Tools used (dmarcian, sha256sum, CyberChef, browser)
3. Walkthrough — DMARC/S/MIME concepts, email analysis, open directory discovery,
   kit hashing, credential-log review, PHP exfiltration-script review, flag decoding
4. Findings summary table
5. Conclusion — how the attack chain worked
6. Mitigation recommendations
7. IOC table
8. References

## Key indicators
| Type | Value |
|---|---|
| Malicious domain | `kennaroads.buzz` |
| Kit archive | `Update365.zip` |
| SHA-256 | `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686` |
| Exfil mailbox | `m3npat@yandex.com` |

## Reference
- TryHackMe room: https://tryhackme.com/room/snappedphishingline
- Kioptrix 1 writeup (uploaded by user) used as the formatting template for this report
