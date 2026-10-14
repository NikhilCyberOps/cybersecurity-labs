# Earth CTF — Vulnerability Assessment & PoC

## Target
- **Victim hostname:** `earth.local` / `earth.lan`
- **Related virtual host observed:** `terratest.earth.local`
- **Victim IP:** `192.168.31.53`
- **Environment:** Intentionally vulnerable, isolated CTF lab

## Report
Open `Earth-CTF-VAPT-Report-Redacted.pdf` for the full report, findings, PoCs, remediation guidance, and redacted screenshot evidence.

## Confirmed observations
- TCP/22: SSH, OpenSSH 8.6
- TCP/80: HTTP, Apache 2.4.51 (Fedora), mod_wsgi 4.7.1 / Python 3.9
- TCP/443: HTTPS, Apache 2.4.51; the supplied Nmap certificate output shows an expiry date in 2022
- Web path discovery found an administrative path and a `cgi-bin/` response; related-host enumeration exposed `robots.txt` and `testingnotes.txt`
- `testingnotes.txt` reveals implementation details and the `terra` admin username
- The supplied evidence shows a message decoded through a hex-to-bytes + XOR workflow, with repeated plaintext
- The Admin Command Tool demonstrates server-side command execution; the supplied sequence obtains a shell as `apache`, triggers `reset_root`, and reaches root in the CTF

## Proof of Concept (PoC) checklist
1. **Service discovery:** Reproduce the target-only Nmap scan shown in the report.
2. **HTTP/HTTPS service details:** Verify banners and inspect the TLS certificate with `nmap --script ssl-cert`.
3. **Web content exposure:** Review `robots.txt` and `testingnotes.txt` on the authorized related virtual host. Do not treat robots directives as access control.
4. **XOR message weakness:** Follow the included evidence for hex decoding and XOR decoding; repeated output indicates predictable/reused key material.
5. **Command execution:** The screenshots show benign command output returned by the Admin Command Tool. The report records the resulting Apache-context shell as evidence.
6. **Privilege escalation:** The screenshots document the `reset_root` trigger and successful root authentication/root-flag access in the intentionally vulnerable CTF. Do not repeat this chain on any system outside the lab.

## Validation commands
Run only against the authorized victim:
```bash
nmap -sV -sC -Pn -p 22,80,443 192.168.31.53
nmap -Pn -p 443 --script ssl-cert 192.168.31.53
curl -I http://earth.local/
curl -k -i https://terratest.earth.local/robots.txt
curl -k -i https://terratest.earth.local/testingnotes.txt
```

## Remediation priorities
- Remove arbitrary command execution and the `reset_root` trigger mechanism.
- Rotate root/admin credentials and review server logs.
- Replace custom XOR/reused-key encryption with authenticated encryption (for example, AES-GCM) and proper key management.
- Remove internal developer notes and sensitive files from the web root.
- Renew the TLS certificate and verify automatic renewal.
- Patch supported SSH, Apache, Python, and related packages; restrict management services to trusted networks.

## Redaction note
Only the victim identity is retained. Other discovered host IPs, hostnames, device names, and callback/source IPs have been redacted or omitted. The unrelated restaurant onboarding screenshot containing sensitive tax/bank details was excluded.

## Disclaimer
This document describes an authorized, intentionally vulnerable CTF lab. Do not test systems without explicit permission.

## Detailed PoC sections in the PDF
The revised PDF now contains a dedicated step-by-step PoC section covering:
- Target-only Nmap discovery and service inventory
- TCP/22 SSH banner validation
- TCP/80 HTTP service checks
- TCP/443 certificate validation
- Gobuster and direct review of robots.txt/testingnotes.txt
- Hex + XOR message decoding observations
- Admin Command Tool command-execution validation
- Apache shell identity evidence
- reset_root/root-flag privilege-escalation evidence
- Finding-to-evidence summary

All reproduction steps are for the isolated CTF victim only. Other discovered hosts and callback/source IPs remain redacted.
