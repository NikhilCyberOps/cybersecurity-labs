# Web Application VAPT Report — Altoro Mutual (testfire.net)

A Vulnerability Assessment & Penetration Testing (VAPT) report documenting 10 verified security findings on **Altoro Mutual**, a deliberately vulnerable banking demo application published by HCL Technologies for security training.

| | |
|---|---|
| **Target** | https://testfire.net/ (Altoro Mutual) |
| **Assessment date** | September 20, 2026 |
| **Methodology** | Manual + automated VAPT |
| **Prepared by** | Nikhil Gowda |
| **Submitted to** | Rahul Chauhan (Mrwebsecure Infosolutions) |
| **Report type** | Training / assessment report |

📄 **Full report:** [`Nikhil_Gowda_VAPT_Assessment_Report.pdf`](./Nikhil_Gowda_VAPT_Assessment_Report.pdf)

---

## Summary

| Critical | High | Medium | Low | Total |
|:-:|:-:|:-:|:-:|:-:|
| 1 | 4 | 4 | 1 | **10** |

Testing covered authentication, authorization, input handling, session management, error handling, response headers, and brute-force protection.

## Findings

| # | Vulnerability | Category (OWASP) | Severity | CWE |
|:-:|---|---|:-:|---|
| 1 | SQL Injection — authentication bypass on `/login.jsp` | Injection | **Critical** | CWE-89 |
| 2 | Broken Access Control / IDOR / BOLA (`listAccounts`) | Broken Access Control | High | CWE-639, CWE-284 |
| 3 | Cross-Site Request Forgery on `/bank/doTransfer` | Broken Access Control | High | CWE-352, CWE-840 |
| 4 | Reflected XSS (`query` parameter on `/search.jsp`) | Injection | High* | CWE-79 |
| 5 | HTML Injection (`/search.jsp`) | Injection | Medium | — |
| 6 | Information disclosure via verbose stack traces | Security Misconfiguration | Medium | CWE-209 |
| 7 | Hardcoded administrative password in page source | Identification & Authentication Failures | High* | CWE-798, CWE-615 |
| 8 | Sensitive cookie missing `HttpOnly` / `Secure` flags | Cryptographic Failures / Misconfiguration | Medium | CWE-1004, CWE-614 |
| 9 | Missing security headers | Security Misconfiguration | Low | CWE-693, CWE-1021 |
| 10 | Missing rate limiting / brute-force protection | Security Misconfiguration | Medium | CWE-307 |

\* The report preserves the severity values exactly as originally written. Findings 4 and 7 are rated differently in the summary table versus their detailed sections (XSS: High vs. Medium; Hardcoded password: High vs. Critical), and findings 9 and 10 carry similar notes. Both values are retained rather than silently reconciled.

Each finding in the PDF includes the affected endpoint, impact, remediation, CWE ID, step-by-step proof of concept, and screenshot evidence.

## Tools & Techniques

- **Burp Suite** (Proxy, Repeater, Intruder)
- **Browser Developer Tools** (Network, Application/Storage tabs)
- **Page source inspection**
- **Kali Linux** + `curl` for header analysis

## Key Remediation Themes

- **Injection (SQLi, XSS, HTML injection):** parameterized queries, contextual output encoding, input validation
- **Access control & CSRF:** server-side ownership checks on every object reference; per-request anti-CSRF tokens; `SameSite` cookies
- **Authentication:** remove hardcoded credentials, use a secrets manager, add rate limiting / lockout / CAPTCHA
- **Configuration:** generic error pages, `HttpOnly` + `Secure` + `SameSite` cookie flags, standard security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)

## Repository Contents

```
.
├── README.md
└── Nikhil_Gowda_VAPT_Assessment_Report.pdf
```

## Disclaimer

Altoro Mutual / testfire.net is an intentionally vulnerable demo application provided for security testing and education. All testing in this report was performed on that sanctioned target for training purposes. **Do not use these techniques against systems you do not own or have explicit written permission to test.**

## Author

- LinkedIn: www.linkedin.com/in/nikhil-gowda-32a171325
