# Noob CTF — VAPT Report

## Overview
- **Victim hostname:** `wtf.lan`
- **Victim IP:** `192.168.31.169`
- **PDF:** `Noob_CTF_VAPT_Report.pdf`
- **Scope:** Victim host only; unrelated discovered hostnames/IPs are omitted.
- **Evidence basis:** Supplied screenshots and `report1(8).xlsx`.

## Report format
The PDF now follows the Kioptrix report's table-based finding layout. Each finding has a structured table containing **PORT NO, SERVICE NAME, SEVERITY, VULNERABILITY NAME, CVE ID, DESCRIPTION, REMEDIATION, and PROOF OF CONCEPT**. Evidence screenshots follow the relevant tables.

## Findings
1. **21/TCP — FTP anonymous access/file disclosure.** Screenshot evidence shows anonymous login, directory listing and retrieval of `cred.txt` and `welcome`.
2. **80/TCP — HTTP service/banner exposure.** Apache httpd 2.4.29 (Ubuntu) identified. No specific Apache CVE is claimed from the supplied evidence.
3. **55077/TCP — SSH exposure.** OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 identified. No specific SSH CVE is claimed.
4. **Sudo configuration — excessive privileges.** Evidence shows broad `ALL` command permissions for user `wtf` and a later root shell.
5. **CTF clue recovery — supporting evidence.** Base64, steganography and ROT13 steps are included as a supporting challenge chain, not independent software vulnerabilities.

## PoC commands (authorized lab only)

### Service enumeration
```bash
nmap -sV -sC -p 21,80,55077 192.168.31.169
```

### FTP
```text
ftp 192.168.31.169
# Login: anonymous; blank password if accepted
ls
get cred.txt
get welcome
```

### HTTP
```bash
nmap -sV -p 80 192.168.31.169
curl -I http://192.168.31.169/
```

### SSH
```bash
nmap -sV -p 55077 192.168.31.169
ssh -p 55077 wtf@192.168.31.169
```

### Challenge clues
```bash
echo 'Y2hhbXA6cGFzc3dvcmQ=' | base64 -d
steghide extract -sf funny.bmp
cat user.txt
tr 'A-Za-z' 'N-ZA-Mn-za-m' < user.txt
```

### Sudo verification
```bash
sudo -l
```

## Remediation priorities
- Remove broad `ALL` sudo permissions and apply least privilege.
- Disable anonymous FTP or restrict it to a dedicated non-sensitive directory.
- Remove exposed clues and credentials from public directories.
- Restrict SSH to trusted management hosts and use strong key-based authentication.
- Keep Apache/OpenSSH updated and reduce unnecessary version disclosure.
- Retest after remediation.

## Evidence and privacy
Unrelated host IPs/hostnames are omitted. Personal credentials and challenge tokens are not reproduced. The SSH evidence image masks the non-target IP line. The screenshot identifies `vsftpd 3.0.3`, while the spreadsheet says `vsftpd 2.3.4`; the report uses the screenshot and does not claim the 2.3.4 backdoor.

## Disclaimer
For authorized CTF/lab learning only. Do not test systems without explicit permission.

