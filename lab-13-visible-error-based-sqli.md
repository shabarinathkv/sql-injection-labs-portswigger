# Lab 13 — Visible Error-Based SQL Injection

**Difficulty:** Practitioner
**Technique:** Error-based SQLi — data extraction via verbose error messages
**Lab URL:** https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based

---

## Vulnerability

The application is vulnerable to SQL injection via a tracking cookie. This lab exploits a misconfiguration where detailed database error messages are displayed directly in the browser. By crafting input that forces the database to include sensitive data inside the error message itself, credentials can be extracted without any UNION-based technique.

---

## Steps Taken

1. Injected a single quote into the tracking cookie — application returned a verbose database error in the browser
2. Recognised this as a critical misconfiguration — error messages should never be visible to users
3. Crafted a payload using the `CAST()` function to force the database to include query output inside the error
4. Observed the administrator password printed directly in the error message
5. Logged in with the extracted credentials

---

## Payload Used

```
TrackingId=xyz' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
```

**What this does:**
- `SELECT password FROM users LIMIT 1` retrieves the first password from the users table
- `CAST(... AS int)` attempts to convert the password string into an integer
- Since a password cannot be converted to an integer, the database throws a type conversion error
- The error message includes the value it tried to convert — the actual password

**Example error returned:**
```
ERROR: invalid input syntax for type integer: "s3cur3p4ssw0rd"
```

---

## Real-World Impact

Verbose error messages in production are a standalone vulnerability. Combined with injectable input — full credential extraction is possible in a single request with no UNION attacks or blind techniques needed. Classified as both Injection (OWASP A03) and Security Misconfiguration (OWASP A05).

---

## Key Takeaway

Error messages are for developers — never for production browsers. Always configure applications to log errors server-side and show only generic messages to users. A single verbose error combined with a SQL injection point is enough for complete credential extraction in one request.
