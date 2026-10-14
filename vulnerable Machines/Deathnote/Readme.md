# Deathnote CTF — VAPT Lab Report

This entry documents an authorized assessment of the intentionally vulnerable **Deathnote CTF** machine. The accompanying PDF contains detailed findings plus a dedicated step-by-step PoC appendix with commands, observed results, screenshots, and remediation advice.

## Target
- **Victim hostname:** `deathnote.lan` / `deathnote.vuln`
- **Victim IP:** `192.168.31.43`
- **Scope:** Victim machine only
- **Environment:** Isolated CTF lab

Other discovered hostnames and IP addresses have been redacted from evidence where identified.

## Tools Used
- Nmap — host discovery and service/version detection
- Dirsearch — web content discovery
- Browser/source view — WordPress metadata inspection
- CyberChef and Brainfuck interpreter — decoding challenge clues
- SSH and Linux utilities — credential validation and privilege review

## Attack Path Summary
1. Nmap identified the victim at `192.168.31.43`.
2. Service detection confirmed TCP/22 (OpenSSH 7.9p1 Debian) and TCP/80 (Apache httpd 2.4.38 Debian).
3. Web enumeration discovered `/wordpress/`, `/wordpress/wp-login.php`, `/robots.txt`, and `/manual/`.
4. `robots.txt` pointed to `/important.jpg`, which contained a clue toward `user.txt` and the site's hint section.
5. A Brainfuck clue and encoded text were decoded. The recovered lab credential belonged to user `kira`.
