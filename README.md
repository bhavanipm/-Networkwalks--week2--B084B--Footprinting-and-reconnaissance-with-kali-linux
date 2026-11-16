# Reconnaissance & Fingerprinting Assessment: networkwalks.com

![Type](https://img.shields.io/badge/type-reconnaissance-blue)
![Platform](https://img.shields.io/badge/platform-Kali%20Linux-557C94)
![Scope](https://img.shields.io/badge/scope-authorized%20only-green)
![Status](https://img.shields.io/badge/status-complete-brightgreen)

A non-intrusive reconnaissance and technology-fingerprinting assessment of **networkwalks.com**, carried out with six standard Kali Linux tools. The full write-up, with screenshots, analysis, severity ratings and remediation advice, is in [`Task2_Internship.pdf`](./Task2_Internship.pdf).

> **Authorization:** Testing was performed with permission from Networkwalks. Only passive and low-impact queries were used. No exploitation, brute-forcing or denial-of-service activity took place.

---

## Table of Contents

- [Objective](#objective)
- [Scope and Environment](#scope-and-environment)
- [Tools and Commands](#tools-and-commands)
- [Key Findings](#key-findings)
- [Positive Observations](#positive-observations)
- [Recommendations](#recommendations)
- [Skills Demonstrated](#skills-demonstrated)
- [Repository Structure](#repository-structure)
- [How to Reproduce](#how-to-reproduce)
- [Limitations](#limitations)
- [Disclaimer](#disclaimer)
- [Author](#author)

---

## Objective

Collect and analyze publicly exposed information about a target domain to understand its attack surface before any deeper testing:

1. Domain registration details
2. Web technologies in use
3. DNS resolution and records
4. HTTP security headers
5. Web Application Firewall (WAF) presence
6. Full DNS enumeration

The work follows the reconnaissance phase of the **PTES** and the information-gathering stage (**WSTG-INFO**) of the **OWASP Web Security Testing Guide**.

## Scope and Environment

| Item | Detail |
|---|---|
| Target | `networkwalks.com` (resolves to `192.232.216.135`) |
| Test type | Reconnaissance only (no exploitation) |
| Operating system | Kali Linux |
| DNS resolver | `8.8.8.8` |
| Assessment date | 15 November 2026 |
| Authorization | Given by Networkwalks |

## Tools and Commands

| # | Task | Tool | Command |
|---|---|---|---|
| 1 | Domain registration | whois | `whois networkwalks.com` |
| 2 | Technology fingerprinting | whatweb | `whatweb networkwalks.com` |
| 3 | IP resolution | nslookup | `nslookup networkwalks.com` |
| 4 | HTTP response headers | curl | `curl -I https://networkwalks.com` |
| 5 | WAF detection | wafw00f | `wafw00f https://networkwalks.com` |
| 6 | DNS enumeration | dnsrecon | `dnsrecon -d networkwalks.com` |

## Key Findings

| ID | Finding | Severity | Source |
|---|---|---|---|
| F-01 | `Strict-Transport-Security` and `Content-Security-Policy` headers are missing | **Medium** | Task 4 |
| F-02 | WordPress core and Download Manager plugin versions are disclosed in the meta generator tag, and two scans reported different versions | Low | Task 2 |
| F-03 | Weak `Referrer-Policy`, partial `Permissions-Policy`, no `SameSite` on the session cookie, no `X-Frame-Options` or `X-Content-Type-Options` | Low | Task 4 |
| F-04 | Public email address and WordPress REST API (`wp-json`) discovery links exposed | Low | Tasks 2, 4 |
| F-05 | DNSSEC is unsigned | Low | Tasks 1, 6 |
| F-06 | Name servers advertise their BIND version (`9.16.23-RH`) | Low | Task 6 |
| F-07 | DMARC record not confirmed in the dnsrecon output (SPF is present); verify `_dmarc.networkwalks.com` | To verify | Task 6 |

### Task highlights

**1. whois:** Registrar is GoDaddy; name servers are at HostGator. All four registrar locks are set (delete, renew, transfer, update). Created 2019-11-06, expires 2027-11-06. DNSSEC is unsigned.

**2. whatweb:** Apache web server with the version hidden, WordPress with the Download Manager plugin, jQuery 3.7.1 and Google Tag Manager. HTTP redirects to HTTPS. The two runs reported different WordPress and plugin versions (7.0.4 / 3.3.58 versus 7.1.4 / 3.3.76), so both are documented in the report.

**3. nslookup:** A single IPv4 address (`192.232.216.135`) and no IPv6 record, via a non-authoritative answer from Google DNS.

**4. curl -I:** HTTP/2 over HTTPS. The `__wpdm_client` cookie has `Secure` and `HttpOnly` but no `SameSite`. Headers such as `x-nginx-cache` and `x-endurance-cache-level` reveal the hosting stack.

**5. wafw00f:** wafw00f v2.4.2 detected a **ModSecurity (SpiderLabs) WAF** in front of the site using 2 requests. WAF detection is heuristic, so the result is treated as indicative rather than final.

**6. dnsrecon:** 8 records found.
- SOA and NS records point to HostGator name servers.
- The A record and the MX record (`mail.networkwalks.com`) share `192.232.216.135`, so web and mail run on the same host.
- An SPF record is present and ends in a strict `-all`.
- cPanel autodiscover SRV records reveal the mail hosting platform.
- A Google site-verification TXT record is published.
- The DNSSEC query returned no answer.

## Positive Observations

- HTTPS is enforced with a 301 redirect and HTTP/2 is in use.
- A WAF (ModSecurity) is present.
- All four registrar locks are enabled.
- The Apache version string is hidden.
- The session cookie sets `Secure` and `HttpOnly`.
- SPF is published with a strict `-all` policy.

## Recommendations

| Priority | Action |
|---|---|
| High | Add HSTS (`max-age=31536000; includeSubDomains`) and a Content-Security-Policy |
| Medium | Add `X-Frame-Options` and `X-Content-Type-Options: nosniff`; tighten `Referrer-Policy` and `Permissions-Policy` |
| Medium | Remove the WordPress generator tag, keep core and Download Manager patched, and confirm which version is live |
| Medium | Publish a DMARC policy (start with `p=none`, then move to `quarantine` or `reject`) and confirm DKIM |
| Low | Add `SameSite=Lax` to the `__wpdm_client` cookie |
| Low | Enable DNSSEC where supported and hide the BIND version on the name servers |
| Low | Replace the public email with a contact form or alias, and restrict REST API user enumeration |

## Skills Demonstrated

- Passive and active reconnaissance with whois, whatweb, nslookup, curl, wafw00f and dnsrecon
- HTTP security header analysis (HSTS, CSP, X-Frame-Options, Referrer-Policy and others)
- DNS and email-security review (SPF, DMARC, DNSSEC, SRV and MX records)
- WAF detection and interpretation of heuristic results
- Risk rating, MITRE ATT&CK and OWASP mapping, and remediation writing
- Professional security reporting with evidence and an authorization statement

## Repository Structure

```
.
├── README.md
├── Task2_Internship.pdf        # Full assessment report
└── screenshots/
    ├── 01_whois.png
    ├── 02_whatweb.png
    ├── 03_nslookup.png
    ├── 04_curl_headers.png
    ├── 05_wafw00f.png
    └── 06_dnsrecon.png
```

## How to Reproduce

Run the commands on a target you **own or have written permission to test**. Practice targets include `scanme.nmap.org` and `testphp.vulnweb.com`.

```bash
whois example.com
whatweb example.com
nslookup example.com
curl -I https://example.com
wafw00f https://example.com
dnsrecon -d example.com
```

Install any missing tools on Kali with `sudo apt install whois whatweb dnsutils curl wafw00f dnsrecon`.


## Limitations

- Results reflect a single point in time; DNS, headers and hosting change often.
- Only low-impact queries were used, and WAF or rate limiting can hide real behavior.
- WAF and technology detection are heuristic and may produce false positives or negatives.
- Version numbers should be checked against vendor advisories before being cited as vulnerabilities.

## Disclaimer

This repository is for educational and portfolio purposes. All testing was performed with permission from the asset owner. Do not run these tools against systems you do not own or have explicit authorization to test.

## Author

**K Bhavani**, Cyber Security & Ethical Hacking Intern

- LinkedIn: www.linkedin.com/in/bhavani-k-692a463a0
- Email:bhavani08021947@gmail.com

## References

- Waqas Karim CCE from Networkwalks
- MITRE ATT&CK: Reconnaissance (TA0043)
- PTES Technical Guidelines
- OWASP Web Security Testing Guide (WSTG-INFO)
