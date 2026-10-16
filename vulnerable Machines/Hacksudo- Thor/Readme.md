# Hacksudo-Thor CTF — VAPT Report with Evidence

## Target
- **Hostname:** `HacksudoThor`
- **Victim IP:** `192.168.31.208`
- **Scope:** Authorized, intentionally vulnerable local CTF machine.

## Deliverables
- `Hacksudo_Thor_CTF_VAPT_Report_With_Evidence.pdf` — detailed field/details PoC tables with the supplied screenshots embedded under the corresponding findings.
- This README — report index and quick reference.

## Evidence-to-finding mapping
- **Images 1–2:** target discovery and Nmap service enumeration.
- **Images 3–4:** HTTP page/source clue and CGI/script enumeration.
- **Images 5–7:** Shellshock Metasploit workflow.
- **Images 8–9:** sudo-permitted `hammer.sh` and execution as `thor`; personal-name/secret-key prompt lines are redacted in the report image.
- **Images 10–13:** shell stabilization, root privilege escalation, root-directory/flag-file evidence, CTF hammer graphic, and target console evidence.

## Findings
| Port / area | Finding | Severity |
|---|---|---:|
| Network | Target discovery and service enumeration | Informational |
| 22/TCP | SSH service exposure; no SSH exploit demonstrated | Low |
| 80/TCP | Apache/CGI endpoint discovery | Medium |
| 80/TCP | Shellshock CGI command execution path | Critical |
| Local | Unsafe sudo execution of `hammer.sh` as `thor` | High |
| Local | Unsafe sudo service invocation to root | Critical |
| 21/TCP | Not confirmed as open FTP from supplied evidence | Informational |

## Redaction and evidence limits
Only victim IP `192.168.31.208` and hostname `HacksudoThor` are retained. Personal-name/secret-key prompt text is redacted in the associated screenshot. Unrelated hostnames/IPs and personal credentials are omitted. Secret flag values are not transcribed. SSH is service exposure only; port 21 is not treated as verified open FTP; CGI discovery alone is not proof of exploitation.

## Recommended remediation order
1. Patch Bash and disable or repair vulnerable CGI endpoints.
2. Remove unsafe sudo rules and correct `hammer.sh` input handling.
3. Remove the unsafe service invocation permission and audit sudoers.
4. Restrict SSH to trusted sources and harden authentication.
5. Re-validate TCP/21 and identify the actual service, if any.

> **Authorization:** These steps are documented for the specified CTF target. Do not run them against systems without explicit permission.

