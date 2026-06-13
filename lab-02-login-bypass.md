# Lab 02 — SQL Injection Allowing Login Bypass

**Difficulty:** Apprentice
**Technique:** Comment injection to bypass authentication
**Lab URL:** https://portswigger.net/web-security/sql-injection/lab-login-bypass

---

## Vulnerability

The login form passes the username directly into a SQL query without sanitisation. By injecting SQL comment syntax into the username field, an attacker can truncate the query and bypass the password check entirely — logging in as any user, including administrators.

---

## Steps Taken

1. Opened the login page and tried a normal login to observe behaviour
2. Entered `'` in the username field — application returned an error, confirming SQLi
3. Identified that the query likely checks username AND password in a WHERE clause
4. Constructed a payload to close the username string and comment out the password check
5. Left the password field as anything (it is never checked after injection)

---

## Payload Used

**Username field:**
```
administrator'--
```

**Password field:**
```
anything
```

**What this does:**
- `administrator` targets the admin account directly
- `'` closes the string
- `--` comments out the rest of the query, including `AND password = '...'`

---

## Original Query (inferred)

```sql
SELECT * FROM users WHERE username = 'administrator' AND password = 'anything'
```

**After injection:**

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'anything'
```

The password check is now completely removed. The database returns the administrator row and login succeeds.

---

## Real-World Impact

This is one of the most dangerous SQLi scenarios. In a real application an attacker could:
- Log in as any user — including admins — without knowing the password
- Take full control of the application
- Access all user data, settings, and admin functionality
- Use admin access as a pivot point for deeper system compromise

---

## Key Takeaway

Authentication queries are high-value targets. Parameterised queries are the fix — but beyond that, applications should enforce multi-factor authentication so that even a bypassed password check is not enough to gain access.
