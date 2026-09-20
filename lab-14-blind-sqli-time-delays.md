# Lab 14 — Blind SQL Injection with Time Delays

**Difficulty:** Practitioner
**Technique:** Blind SQLi — time-based confirmation
**Lab URL:** https://portswigger.net/web-security/sql-injection/blind/lab-time-delays

---

## Vulnerability

The application is vulnerable to blind SQL injection via a tracking cookie. This lab removes every signal used in previous labs — no visible output, no error messages, no behavioural differences in the page response. The only way to confirm injection exists is by measuring how long the server takes to respond. A deliberate time delay injected into the query becomes the confirmation signal.

---

## Steps Taken

1. Injected a single quote into the tracking cookie — no error, no visible change in the page
2. Tried conditional response payloads from Lab 11 — no difference detected
3. Tried error-based payloads from Lab 12 — no error returned
4. Switched to time-based technique — injected a sleep payload
5. Observed the server response was delayed by exactly 10 seconds — confirming SQLi
6. Lab objective achieved — time delay confirmed the injection point exists

---

## Payload Used

```
TrackingId=xyz'||pg_sleep(10)--
```

**What this does:**
- `||` is the string concatenation operator (PostgreSQL)
- `pg_sleep(10)` tells the PostgreSQL database to pause for 10 seconds
- If the server responds after exactly 10 seconds — injection is confirmed
- If the server responds instantly — the payload was not executed

**Database-specific sleep functions:**

| Database | Sleep Payload |
|----------|--------------|
| PostgreSQL | `pg_sleep(10)` |
| MySQL | `SLEEP(10)` |
| MSSQL | `WAITFOR DELAY '0:0:10'` |
| Oracle | `dbms_pipe.receive_message(('a'),10)` |

---

## Why This Lab Matters

This lab is purely about confirmation — not data extraction. Before attempting to extract data via time delays (Lab 15), you need to:
- Confirm the injection point exists
- Identify which sleep function works (reveals the database platform)
- Establish that timing is reliable enough to use as a signal

A 10 second delay is unmistakable. Shorter delays can be caused by network latency and are unreliable for detection.

---

## Real-World Impact

Time-based blind SQLi confirmation means:
- The application is vulnerable even with all output suppressed
- Data extraction is possible using the technique demonstrated in Lab 15
- The database platform is revealed by which sleep function succeeds
- No WAF or output filter can prevent timing-based detection as long as injection exists

---

## Key Takeaway

When every other signal is gone — no output, no errors, no behavioural difference — time is always left. A database that pauses on command is a database that can be controlled. This lab is the gateway to Lab 15 where the same timing channel is used to extract the complete administrator password character by character.
