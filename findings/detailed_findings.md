# Detailed Findings – zero.webappsecurity.com

**Program / Track:** Cyber Security (CS) — Task 01
**Assessment type:** Passive, non-intrusive external vulnerability assessment
**Date:** 29 September 2026

---

## Risk Rating Method

Each finding is rated by combining **how likely** it is to be abused with **how much damage** it would cause.

| Rating | Meaning for the business |
|---|---|
| **High** | Customer data or trust is directly at risk; weaknesses are well known and easy to abuse. Fix immediately (within days). |
| **Medium** | Makes attacks easier or removes an important layer of protection. Fix in the next planned update (within weeks). |
| **Low** | Gives attackers useful information but causes no direct harm on its own. Fix during routine maintenance. |

## Summary

| ID | Finding | Risk | Raw findings |
|---|---|---|---|
| VA-01 | Customer logins and banking activity sent without encryption | **High** | F-01, F-12 |
| VA-02 | Secure (HTTPS) website is broken and unusable | **High** | F-02, F-03, F-11 |
| VA-03 | Weak and broken encryption methods accepted | **High** | F-04, F-06 |
| VA-04 | Outdated, unsupported server software | **High** | F-05 |
| VA-05 | Missing browser security protections | **Medium** | F-08 |
| VA-06 | Website data can be read by any other website | **Medium** | F-07 |
| VA-07 | Unnecessary second entrance to the website (port 8080) | **Medium** | F-09 |
| VA-08 | Server reveals its software and versions | **Low** | F-10 |

**Totals:** 4 High · 3 Medium · 1 Low

---

## VA-01 — Customer logins and banking activity sent without encryption

**Risk: HIGH**

**What we found**
The online banking website, including the login page, only works over plain HTTP. When a customer signs in, their username and password travel across the internet as readable text. Visitors are not redirected to a secure version, and Firefox displays its own warning that logins entered on the page "could be compromised."

**Why it matters to the business**
Anyone on the same network as a customer — for example on café, hotel or airport Wi-Fi — could read login details and account activity. This could lead to account takeover, fraud, regulatory penalties and loss of customer trust.

**Evidence**
- `scans/00_zero_initial_headers.txt` — site answers over HTTP with no redirect
- `scans/03_login_form.txt` — login form submits to `/signin.html` over HTTP
- `screenshots/08_login_page_http.png`, `screenshots/09_insecure_login_warning.png`

**How to fix it**
1. Serve the entire banking application over HTTPS only (depends on fixing VA-02, VA-03 and VA-04).
2. Automatically redirect every HTTP request to HTTPS (permanent 301 redirect).
3. Once HTTPS works reliably, enable HSTS (`Strict-Transport-Security`) so browsers never use HTTP again.

**Reference:** OWASP WSTG-CRYP-03 · CWE-319 (Cleartext Transmission of Sensitive Information)

---

## VA-02 — Secure (HTTPS) website is broken and unusable

**Risk: HIGH**

**What we found**
- The website's security certificate **expired on 4 May 2022**.
- The server only supports security protocols retired years ago (SSLv3 and TLS 1.0). Modern browsers refuse to connect: Firefox shows **"Secure Connection Failed."**
- The HTTPS address shows only the web server's default "It works!" test page, not the bank.

**Why it matters to the business**
Customers who try the secure address cannot reach the bank at all, which pushes everyone onto the unencrypted version (VA-01). An expired certificate and an error page also make the bank look neglected and untrustworthy, and train customers to ignore security warnings.

**Evidence**
- `reconnaissance/05_zero_service_detection.nmap` — certificate dates and supported protocols
- `screenshots/05_https_browser_error.png`
- `scans/01_https_root_content.txt` — default page content

**How to fix it**
1. Renew the certificate now, and set up automatic renewal and expiry alerts.
2. Enable only TLS 1.2 and TLS 1.3; disable SSLv3, TLS 1.0 and TLS 1.1.
3. Remove the default test page and serve the real banking application on HTTPS.

**Reference:** OWASP WSTG-CRYP-01 · CWE-298 (Improper Validation of Certificate Expiration) · CWE-327

---

## VA-03 — Weak and broken encryption methods accepted

**Risk: HIGH**

**What we found**
The server accepts encryption methods known to be breakable, including deliberately weakened "export-grade" encryption from the 1990s, RC4, DES/3DES and MD5. It also lets the visitor's browser choose the encryption method (even a weak one) and has compression enabled, which enables an attack that can steal login sessions.

**Why it matters to the business**
Even if customers connected securely, an attacker could force or exploit weak encryption to read or tamper with banking traffic. These weaknesses have public names (POODLE, SWEET32, FREAK, CRIME) and would fail any banking or card-industry (PCI DSS) compliance check.

**Evidence**
- `reconnaissance/05_zero_service_detection.nmap` — `ssl-enum-ciphers` output, weakest grade **E**

**How to fix it**
1. Allow only strong, modern encryption settings (the free Mozilla SSL Configuration Generator provides ready-made settings).
2. Make the server choose the encryption method, not the browser.
3. Turn off TLS compression.

**Reference:** OWASP WSTG-CRYP-01 · CWE-327 (Use of a Broken or Risky Cryptographic Algorithm)

---

## VA-04 — Outdated, unsupported server software

**Risk: HIGH**

**What we found**
The secure web server runs **Apache 2.2.6** and **OpenSSL 0.9.8e**, both released in 2007. Neither has received security updates for years (Apache 2.2 support ended in 2017; OpenSSL 0.9.8 in 2015).

**Why it matters to the business**
Unsupported software collects known, published weaknesses that will never be fixed. It is also the root cause of VA-02 and VA-03: this OpenSSL version cannot support modern encryption at all, so those issues cannot be fully fixed without upgrading.

**Evidence**
- `reconnaissance/05_zero_service_detection.nmap` — version detection
- `Server` response header on port 443

**How to fix it**
1. Upgrade to a currently supported web server (Apache 2.4.x) and OpenSSL (3.x), and confirm the application server (Tomcat) is also on a supported version.
2. Introduce a patch-management routine: check for security updates monthly and apply critical ones promptly.
3. Keep an inventory of software versions so end-of-life dates are tracked.

**Reference:** OWASP Top 10 A06:2021 — Vulnerable and Outdated Components · CWE-1104

---

## VA-05 — Missing browser security protections

**Risk: MEDIUM**

**What we found**
The website does not send the standard instructions that tell browsers how to protect visitors. Missing: `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`, `Referrer-Policy`. OWASP ZAP flagged the same issues independently.

**Why it matters to the business**
Without these, attackers have an easier time tricking customers — for example, hiding the bank's page inside a fake website to capture clicks ("clickjacking") or injecting malicious scripts. These headers are a cheap, widely expected layer of protection.

**Evidence**
- `scans/00_zero_initial_headers.txt`, `screenshots/06_devtools_headers.png`
- `scans/zap_passive_report.html` — Missing Anti-clickjacking Header; X-Content-Type-Options Header Missing

**How to fix it**
Add these headers in the web server configuration:
```
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Strict-Transport-Security: max-age=31536000; includeSubDomains   (after HTTPS is fixed)
```
Test the Content-Security-Policy on a staging site first, as a strict policy can block legitimate page content.

**Reference:** OWASP Secure Headers Project · CWE-693 (Protection Mechanism Failure) · CWE-1021

---

## VA-06 — Website data can be read by any other website

**Risk: MEDIUM**

**What we found**
The server sends `Access-Control-Allow-Origin: *`, which tells browsers that **any** website on the internet may read responses from the bank's site.

**Why it matters to the business**
This is acceptable for public, non-sensitive content, but for a banking site it removes a key browser protection. If sensitive pages or data services use the same setting, a malicious website could read customer information.

**Evidence**
- `scans/00_zero_initial_headers.txt`, `scans/02_login_headers.txt`

**How to fix it**
1. Remove the header where cross-site access is not needed.
2. Where it is needed, list only specific trusted domains instead of `*`.

**Reference:** OWASP WSTG-CLNT-07 · CWE-942 (Permissive Cross-domain Policy)

---

## VA-07 — Unnecessary second entrance to the website (port 8080)

**Risk: MEDIUM**

**What we found**
The full banking website is also available on port 8080, unencrypted — a second public entrance alongside the main one.

**Why it matters to the business**
Every extra entrance must be secured and monitored separately. Secondary ports are often forgotten, miss security updates, or bypass protections applied to the main site.

**Evidence**
- `reconnaissance/04_zero_port_discovery.nmap`, `reconnaissance/05_zero_service_detection.nmap`

**How to fix it**
1. Block port 8080 from the internet with a firewall rule.
2. Configure the application server to accept connections only from the front-end web server, so all visitors go through the single, secured HTTPS entrance.

**Reference:** OWASP WSTG-CONF-01 · CWE-1125 (Excessive Attack Surface)

---

## VA-08 — Server reveals its software and versions

**Risk: LOW**

**What we found**
The server announces its exact software in every response, e.g. `Apache/2.2.6 (Win32) mod_ssl/2.2.6 OpenSSL/0.9.8e mod_jk/1.2.40` and `Apache-Coyote/1.1`. OWASP ZAP flagged this independently.

**Why it matters to the business**
This hands attackers a ready-made list of which known weaknesses to try. It does no harm on its own, but it saves attackers time — especially combined with VA-04.

**Evidence**
- `Server` headers in `scans/00_zero_initial_headers.txt` and Nmap output
- `scans/zap_passive_report.html` — Server Leaks Version Information

**How to fix it**
1. Apache: set `ServerTokens Prod` and `ServerSignature Off`.
2. Tomcat: set a generic value in the connector's `server` attribute.

**Reference:** OWASP WSTG-INFO-02 · CWE-497 (Exposure of Sensitive System Information)

---

## Positive Observations

- Only 3 of the 1,000 most common ports are open; the rest are blocked by a firewall.
- Banking pages tell browsers not to store copies (`Cache-Control: no-store`).
- The login form uses POST, so passwords do not appear in web addresses or logs.
- No cookies are set on public pages before login.
- The certificate uses a strong 2048-bit key with SHA-256.
- Manual checks and OWASP ZAP's automated scan agreed on overlapping findings.

## Limitations

- Passive, non-intrusive testing only. No exploitation, login, or active scanning was performed, so some weaknesses may not have been detected.
- Session cookie protections could not be assessed, because session cookies are only issued after login, which was out of scope.
- The target is an intentionally vulnerable demo site provided for security training; findings are reported as if it were a live bank.
- Initial targets (testphp.vulnweb.com, testhtml5.vulnweb.com) were unreachable from the assessment network on 29 September 2026 and were replaced.
