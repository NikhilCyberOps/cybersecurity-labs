# Nemesis CTF — VAPT Walkthrough

**Victim hostname:** `nemesis.lan`  
**Victim IP:** `192.168.31.142`  
**Report:** `Nemesis_CTF_VAPT_Report.pdf`  
**Evidence:** 18 relevant screenshots embedded in the PDF appendix (sensitive areas redacted)

> Authorized isolated-lab use only. Unrelated IPs, contact/address text, and private-key contents are redacted. Treat the key shown in the original evidence as compromised and rotate it if used outside the disposable VM.

## Port inventory

| Port | Service/version | Evidence |
|---|---|---|
| `80/tcp` | Apache httpd `2.4.38 (Debian)` | Nmap; web page displays `/etc/passwd` contents |
| `52845/tcp` | nginx `1.14.2` | Nmap service detection |
| `52846/tcp` | OpenSSH `7.9p1 Debian 10+deb10u2` | Nmap; successful SSH login shown |

## Findings and PoC

### 1. Sensitive file disclosure — TCP/80
**Severity:** High (provisional)  
**Evidence:** browser screenshot renders `/etc/passwd` content. The exact input/route is not clear enough to conclusively label this as LFI or path traversal.

**Validation:** reproduce the same request in the authorized lab, capture the request/response with unrelated details redacted, and confirm that local file contents are no longer returned.

**Remediation:** allowlist file identifiers; reject user-controlled filesystem paths; enforce canonical-path checks; use least privilege.

### 2. SSH private-key exposure — TCP/52846
**Severity:** High  
**Evidence:** private-key material is visible in original screenshots and a later screenshot shows successful SSH login as `thanos` using an identity file. The key itself is redacted in the report.

```bash
nmap -sV -p 52846 192.168.31.142
ssh -p 52846 -i <authorized_key_file> thanos@192.168.31.142
```

**Remediation:** revoke/rotate the key, remove copies, audit `authorized_keys`, restrict source networks, and monitor SSH authentication.

### 3. Privilege escalation through sudo-authorized Nano
**Severity:** Critical  
**Evidence:** `sudo -l` permits `carlos` to run `/bin/nano /opt/priv`; later evidence shows root identity.

```bash
sudo -l
# Supplied evidence: /bin/nano /opt/priv is permitted for carlos
id
whoami
```

**Remediation:** remove Nano from sudoers. Avoid elevated access to interactive editors, pagers, interpreters, or shells. Use `visudo` and narrowly scoped helper commands.

### 4. Apache exposure/version disclosure — TCP/80
```bash
nmap -sV -p 80 192.168.31.142
curl -I http://192.168.31.142/
```
**Evidence:** Apache `2.4.38` is detected. Version disclosure alone does not prove a specific CVE. Patch supported packages, minimize banners, and fix the separately documented file-disclosure behavior.

### 5. nginx exposed on alternate port — TCP/52845
```bash
nmap -sV -p 52845 192.168.31.142
curl -i http://192.168.31.142:52845/
```
**Evidence:** nginx `1.14.2` is detected. The screenshots do not prove a specific nginx CVE. Upgrade, restrict the listener, and review server blocks and file permissions.

### 6. SSH exposure — TCP/52846
**Evidence:** OpenSSH `7.9p1` is detected and an authenticated session is shown. Exposure is confirmed; a separate OpenSSH exploit is not demonstrated.

**Remediation:** patch, restrict network access, revoke exposed keys, disable unused accounts, and monitor logs.

### 7. Python backup/helper scripts — further validation required
**Evidence:** `backup.py` reports a permission error while writing `/tmp/website.zip`; a helper script contains socket/process-spawn behavior. The exact relationship between script versions and the received shell is not fully established.

**Validation:** inspect script permissions/source in the isolated VM and test only with a controlled listener.

**Remediation:** remove unnecessary shell-spawn/debug logic, validate file paths, protect scripts from modification, and restrict outbound connections.

### 8. Root access / flag
**Evidence:** final screenshot shows root access and `FLAG{CTFs_ARE_AW3S0M3}`.

## Remediation checklist

- [ ] Remove sudo permission for Nano and audit sudoers.
- [ ] Revoke/rotate exposed SSH key.
- [ ] Fix web file disclosure and retest.
- [ ] Review Python helper/backup scripts.
- [ ] Patch Apache, nginx, OpenSSH, and the OS.
- [ ] Restrict network access to trusted lab hosts.
- [ ] Review SSH, sudo, web, and outbound connection logs.
- [ ] Restore the lab VM from a clean snapshot after the exercise.

## Evidence limitations

This report is based on the 18 relevant supplied screenshots, not a fresh scan. No CVE is assigned without sufficient evidence. Open ports confirm exposure, not exploitability. Embedded screenshots redact unrelated IPs, contact/address details, and private-key material.

