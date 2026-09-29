# Scope & Rules of Engagement

**Program / Track:** Cyber Security (CS) — Task 01
**Assessor:** KEZIAH TISHER
**Assessment type:** Passive, non-intrusive external vulnerability assessment
**Dates:** 26 September 2026 – 26 October 2026

## In scope
- Target: http://zero.webappsecurity.com
- Authorization: Publicly provided by OpenText (formerly Micro Focus) as an intentionally vulnerable demo banking application for security testing

## Permitted activities
- Basic port and service discovery (Nmap, default timing)
- HTTP security header and cookie inspection
- OWASP ZAP passive scanning (spider + passive rules only)
- Manual review via browser DevTools

## Prohibited activities
- Exploitation of any identified weakness
- Authentication bypass or credential attacks
- Active scanning, fuzzing, or injection testing
- Denial-of-Service or high-rate traffic
- Accessing, modifying, or exfiltrating data

## Methodology reference
OWASP Web Security Testing Guide (WSTG) v4.2 — Information Gathering (WSTG-INFO) and Configuration Testing (WSTG-CONF)
