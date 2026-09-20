# Lab 12 — Blind SQL Injection with Conditional Errors

**Difficulty:** Practitioner
**Technique:** Blind SQLi — error-based inference
**Lab URL:** https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors

---

## Vulnerability

The application is vulnerable to blind SQL injection via a tracking cookie. This lab removes the last visual clue from Lab 11 — there is no "Welcome back" message. Instead, the application returns a 500 error when a database error is triggered and a normal 200 response otherwise. The error itself becomes the binary signal used to infer data.

---

## Steps Taken

1. Injected a single quote into the tracking cookie — server returned a 500 error, confirming SQLi
2. Verified that a valid query returns 200 and an invalid one returns 500
3. Constructed a conditional expression that triggers a divide-by-zero error only when a condition is TRUE
4. Used this to confirm the existence of the administrator account
5. Extracted each character of the password by triggering errors on correct character matches
6. Logged in with the extracted credentials

---

## Payload Logic

**Trigger error on TRUE condition (Oracle syntax):**
```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```
- `CASE WHEN (1=1)` — condition to test
- `TO_CHAR(1/0)` — division by zero triggers a 500 error
- If condition is FALSE — empty string returned, 200 response

**Extract password character by character:**
```
TrackingId=xyz'||(SELECT CASE WHEN (SUBSTR(password,1,1)='a') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
- Error (500) = character matches = TRUE
- No error (200) = character doesn't match = FALSE

---

## Real-World Impact

Error-based blind SQLi works even when developers have disabled all visible output and removed all success/failure messages. As long as the application behaves differently on a database error versus a successful query, an attacker has a signal to exploit.

---

## Key Takeaway

When you have no visible output and no conditional messages — look for error behaviour. A 500 vs 200 response is all you need. Error-based blind SQLi is the technique for situations where every other signal has been removed.
