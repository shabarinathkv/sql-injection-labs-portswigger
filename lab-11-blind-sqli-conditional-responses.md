# Lab 11 — Blind SQL Injection with Conditional Responses

**Difficulty:** Practitioner
**Technique:** Blind SQLi — boolean-based inference
**Lab URL:** https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses

---

## Vulnerability

The application is vulnerable to blind SQL injection via a tracking cookie. Unlike previous labs, the application does not return query results or errors in the response. Instead, it shows a "Welcome back" message when a condition is true and hides it when false. This subtle difference becomes the signal used to extract data one character at a time.

---

## Steps Taken

1. Noticed a tracking cookie in the request — injected a single quote to test for SQLi
2. Observed that `' AND '1'='1` kept the "Welcome back" message visible — TRUE condition
3. Observed that `' AND '1'='2` made the message disappear — FALSE condition
4. Confirmed blind SQLi — the page behaviour changes based on injected logic
5. Used conditional queries to determine the length of the administrator password
6. Extracted each character of the password one at a time by testing ASCII values
7. Used the extracted password to log in as administrator

---

## Payload Logic

**Confirm vulnerability:**
```
TrackingId=xyz' AND '1'='1   → Welcome back appears (TRUE)
TrackingId=xyz' AND '1'='2   → Welcome back disappears (FALSE)
```

**Check password length:**
```
TrackingId=xyz' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=20--
```
Increment the number until the message appears — that number is the password length.

**Extract each character:**
```
TrackingId=xyz' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a'--
```
Test every character position against every possible character (a-z, 0-9) until the message appears.

---

## Real-World Impact

Blind SQLi is just as dangerous as visible SQLi — the attacker extracts the same data, just more slowly. Every character of every password can be extracted through patient enumeration. In practice, tools like SQLMap automate this process to extract an entire database in minutes despite having no visible output.

---

## Key Takeaway

When the application gives you nothing visible, use its behaviour as your oracle. A message appearing or disappearing, a page loading faster or slower — any consistent difference can be exploited to extract data bit by bit. This is what makes blind SQLi so powerful and why it appears so frequently in real-world assessments.
