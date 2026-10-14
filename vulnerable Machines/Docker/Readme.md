# Docker CTF Vulnerability Assessment --- PoC README

**Victim hostname:** `vlundocker.lan`\
**Victim IP:** `192.168.31.113`\
**Observed web app:** `http://192.168.31.113:8000/`\
**Scope:** Authorized, isolated Docker CTF lab only.

Other discovered IP addresses and host/device names have been omitted.
The victim hostname and IP are retained.

## How this report uses evidence

The supplied Kioptrix PDF is used as a report/PoC formatting reference
only. The Docker findings below are based on the provided screenshots
and `report1(4).xlsx`. Where evidence does not confirm that a service
belongs to the victim, the PoC is a validation step, not a claim that
the port is open.

## 1. Initial reconnaissance

**Objective:** Determine the victim's actual open ports and service
versions.

**Step 1 --- Run a target-specific scan:**

``` bash
nmap -sV -sC -Pn -p- 192.168.31.113
```

**Step 2 --- Record results only for the victim IP.** Screenshots show
the WordPress site at TCP/8000. Other ports shown during Docker-network
scans must be independently checked against the victim before reporting
them as exposed.

## 2. Outdated WordPress --- High

**Evidence:** WPScan screenshot identifies WordPress 4.8.28 and the
Twenty Seventeen theme.

**PoC steps** 1. Fingerprint WordPress and enumerate versions/users. 2.
Compare detected versions against vendor advisories; do not infer a
specific CVE from a version string alone.

``` bash
wpscan --url http://192.168.31.113:8000/ --enumerate u,v
```

**Observed:** WPScan output reports WordPress 4.8.28.\
**Remediation:** Upgrade WordPress core and themes to supported
releases, then rescan.

## 3. XML-RPC enabled --- Medium

**Evidence:** WPScan reports XML-RPC enabled at `/xmlrpc.php`.

**PoC steps** 1. Request the endpoint. 2. Inspect the HTTP response. 3.
Disable it if not required, or restrict access if it is needed.

``` bash
curl -i http://192.168.31.113:8000/xmlrpc.php
```

**Observed:** The supplied WPScan output detects XML-RPC. Availability
alone is not proof of a standalone CVE.\
**Remediation:** Disable XML-RPC if unused; otherwise restrict access
and monitor authentication attempts.

## 4. WordPress user enumeration --- Low

**Evidence:** WPScan identifies user/author `bob`.

**PoC steps** 1. Make a benign author-pattern request. 2. Compare
redirect/page metadata with the WPScan user-enumeration evidence.

``` bash
curl -i 'http://192.168.31.113:8000/?author=1'
```

**Remediation:** Reduce unnecessary author enumeration and enforce
strong credentials, MFA, and rate limiting.

## 5. Weak WordPress credentials --- High

**Evidence:** Screenshot reports successful XML-RPC authentication for
user `bob` with password `Welcome1`.

**PoC steps** 1. Use the supplied successful-authentication screenshot
as evidence; do not repeat password guessing outside this isolated CTF.
2. Change the password. 3. Verify the old password no longer
authenticates using one controlled test.

``` text
Evidence from screenshot:
Username: bob
Password accepted in lab: Welcome1
```

**Observed:** The screenshot explicitly reports a valid
username/password combination.\
**Remediation:** Rotate the password immediately, review access logs,
enable MFA where available, and rate-limit authentication.

## 6. PHP fatal error / filesystem path disclosure --- Medium

**Evidence:** Browser screenshot of the Twenty Seventeen `404.php` path
shows a PHP fatal error and internal filesystem path.

**PoC steps** 1. Request the path. 2. Record status and response body.
3. Confirm after remediation that internal paths/errors are no longer
disclosed.

``` bash
curl -i http://192.168.31.113:8000/wp-content/themes/twentyseventeen/404.php
```

**Observed:** Screenshot shows a fatal error involving `get_header()`
and a filesystem path. This proves error disclosure, not code execution
via this URL.\
**Remediation:** Disable `display_errors`, log errors server-side, and
update/repair the theme.

## 7. Reverse shell / command execution as `www-data` --- High

**Evidence:** Shell screenshot shows `uid=33(www-data)`,
`gid=33(www-data)`, and `whoami` returning `www-data`.

**PoC steps** 1. Treat the supplied shell screenshot as proof of a shell
session in the lab. The screenshots do not contain a complete
target-specific payload command, so this README does not invent one. 2.
Verify execution context using read-only commands inside the established
lab shell.

``` bash
id
whoami
ip a
```

**Observed:** Command execution as the web-server account inside the
container. The evidence does not establish root or host compromise.\
**Remediation:** Fix the initial web weakness, remove unauthorized
access, rotate exposed credentials, review logs, rebuild the container
from a trusted image, and use least privilege.

## 8. FTP / vsftpd 2.3.4 on TCP/21 --- Critical if confirmed; unverified

**Evidence source:** `report1(4).xlsx` lists FTP/21, vsftpd 2.3.4,
Critical. No supplied screenshot confirms this service on the victim.

**PoC steps** 1. Check whether TCP/21 is open on the victim. 2. Review
the service banner and validate the exact package/build through
authorized lab access. 3. Do not classify the issue as confirmed based
on the spreadsheet or banner alone.

``` bash
nmap -sV -Pn -p 21 192.168.31.113
```

**Observed:** Unverified lead only.\
**Remediation:** If confirmed vulnerable, isolate/disable it and replace
it with a trusted supported package; rescan afterward.

## 9. Per-port validation PoCs

These commands validate whether the listed service is actually exposed
on the victim IP. The Docker-network screenshot alone does not prove all
of these ports belong to the victim.

  Port                       PoC
  -------------------------- ---------------------------------------
  22 / SSH                   `nmap -sV -Pn -p 22 192.168.31.113`
  80 / HTTP                  `nmap -sV -Pn -p 80 192.168.31.113`
  111 / RPCBind              `nmap -sV -Pn -p 111 192.168.31.113`
  443 / HTTPS                `nmap -sV -Pn -p 443 192.168.31.113`
  1024 / RPC-related         `nmap -sV -Pn -p 1024 192.168.31.113`
  3306 / MySQL               `nmap -sV -Pn -p 3306 192.168.31.113`
  8000 / WordPress           `nmap -sV -Pn -p 8000 192.168.31.113`
  8022 / forwarded service   `nmap -sV -Pn -p 8022 192.168.31.113`

Combined inventory:

``` bash
nmap -sV -sC -Pn -p 21,22,80,111,443,1024,3306,8000,8022 192.168.31.113
```

## 10. Remediation priority

1.  Change the weak WordPress password and review authentication logs.
2.  Upgrade WordPress core/theme and disable public PHP error display.
3.  Investigate the `www-data` shell, remediate root cause, and rebuild
    the container if needed.
4.  Validate the spreadsheet's FTP/vsftpd claim; isolate/patch if
    confirmed.
5.  Disable/restrict XML-RPC if not needed.
6.  Confirm ownership of internal SSH/HTTP/MySQL/forwarded ports; close
    or firewall unnecessary services.

## Limitations

The supplied screenshots show multiple internal Docker-network hosts.
Ports observed on those hosts are not claimed as victim-exposed until
validated against `192.168.31.113`. The exact shell payload is not shown
in the supplied evidence, so no payload has been fabricated. All
commands are for the authorized isolated CTF target only.
