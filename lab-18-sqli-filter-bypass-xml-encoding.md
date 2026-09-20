# Lab 18 — SQL Injection with Filter Bypass via XML Encoding

**Difficulty:** Expert
**Technique:** SQLi filter bypass — XML entity encoding to evade WAF
**Lab URL:** https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding

---

## Vulnerability

The application's stock check feature passes user-supplied XML data into a SQL query. A WAF is in place that detects and blocks standard SQL injection keywords like `UNION` and `SELECT`. This lab demonstrates bypassing WAF detection using XML entity encoding to obfuscate the payload while keeping it executable by the database.

---

## Steps Taken

1. Identified the stock check feature sends XML data in a POST request
2. Injected a basic UNION payload — WAF detected and blocked it with a 403 response
3. Used Burp Suite's Hackvertor extension to apply XML entity encoding to the payload
4. Encoded SQL keywords using XML hex entities — the WAF did not recognise them as SQL
5. The database decoded the entities automatically and executed the SQL normally
6. Extracted usernames and passwords from the users table through the bypassed WAF
7. Logged in as administrator

---

## Payload Logic

**Blocked by WAF:**
```xml
<storeId>1 UNION SELECT username||'~'||password FROM users</storeId>
```
WAF detects `UNION SELECT` → 403 Forbidden.

**Bypassed using XML hex encoding:**
```xml
<storeId>1 &#x55;&#x4e;&#x49;&#x4f;&#x4e; &#x53;&#x45;&#x4c;&#x45;&#x43;&#x54; username||'~'||password FROM users</storeId>
```

**What this does:**
- `&#x55;` = `U`, `&#x4e;` = `N` etc. — spelling UNION in XML hex entities
- WAF scans for the string `UNION` — encoded version not recognised
- XML parser decodes the entities before passing to the database
- Database receives and executes `UNION SELECT` normally

---

## WAF Bypass Techniques

| Method | Example for 'U' | Notes |
|--------|----------------|-------|
| XML hex entity | `&#x55;` | Works inside XML context |
| XML decimal entity | `&#85;` | Alternative XML encoding |
| Case variation | `UnIoN` | Works on case-insensitive WAFs |
| Comment insertion | `UN/**/ION` | Works on some WAFs |

---

## Real-World Impact

WAFs are not a substitute for secure coding. A WAF blocking `UNION SELECT` can be bypassed with simple encoding — the underlying vulnerability still exists. WAF bypass is a standard step in professional penetration testing. The only reliable fix is parameterised queries.

---

## Key Takeaway

This is the final lab in the series and its lesson is the most important — no single layer of defence is sufficient. WAFs add protection but must never be the primary defence. Parameterised queries eliminate SQL injection at the source, making bypass techniques irrelevant. Defence in depth means fixing the vulnerability AND adding the WAF — not replacing one with the other.
