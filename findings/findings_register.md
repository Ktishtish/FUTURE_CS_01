# Findings Register – zero.webappsecurity.com (Draft)

| ID | Finding | Evidence | Risk |
|----|---------|----------|------|
| F-01 | Banking application served over unencrypted HTTP (ports 80, 8080); no redirect to HTTPS | curl headers; Nmap 05 | High |
| F-02 | Expired TLS certificate (expired 4 May 2022) | Nmap ssl-cert | High |
| F-03 | Obsolete protocols only (SSLv3, TLS 1.0); no TLS 1.2/1.3 | Nmap ssl-enum-ciphers | High |
| F-04 | Weak/export-grade ciphers accepted (EXPORT, RC4, DES, 3DES, MD5); worst grade E | Nmap ssl-enum-ciphers | High |
| F-05 | End-of-life server software (Apache 2.2.6, OpenSSL 0.9.8e) | Nmap -sV, Server header | High |
| F-06 | TLS compression enabled (CRIME risk) | Nmap ssl-enum-ciphers | Medium |
| F-07 | Wildcard cross-origin policy (Access-Control-Allow-Origin: *) | curl / Nmap headers | Medium |
| F-08 | Missing security headers (CSP, X-Frame-Options, X-Content-Type-Options, HSTS, Referrer-Policy) | curl headers | Medium |
| F-09 | Redundant public web port 8080 exposing the application | Nmap 04/05 | Medium |
| F-10 | Detailed server/version banners disclosed (software, OS, modules) | Server headers | Low |
| F-11 | Default web server page exposed on HTTPS | Nmap 443 headers (44 bytes, 2004 date) | Low |
| F-12 | Login credentials submitted over unencrypted HTTP (form posts to /signin.html over http://); confirmed by browser warning | scans/03_login_form.txt; screenshots/08, 09 | High |
**Positive observations:** Only 3 of 1,000 common ports exposed; anti-caching headers correctly set on application pages; 2048-bit RSA key with SHA-256 signature.
**Additional positive observations:** Login form uses POST (credentials not exposed in URLs); no cookies set on public pages before login.

**Limitations:** Session cookie attributes not assessed — cookies are issued post-authentication, which was out of scope.
