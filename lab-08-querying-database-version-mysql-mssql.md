# Lab 08 — SQL Injection Attack: Querying the Database Type and Version on MySQL and MSSQL

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — database fingerprinting on MySQL and MSSQL
**Lab URL:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. This lab covers the same objective as Lab 07 — extracting the database version — but on MySQL or MSSQL instead of Oracle. The technique is identical but the syntax is different, reinforcing that penetration testers must adapt their approach to the specific platform they are targeting.

---

## Steps Taken

1. Confirmed SQLi via single quote — server returned an error
2. Determined column count using ORDER BY — confirmed 2 columns
3. Confirmed text-compatible columns using NULL probing
4. Identified this as MySQL/MSSQL — no mandatory FROM clause required
5. Used the `@@version` global variable to extract the database version string
6. Observed the version information returned in the application response

---

## Payload Used

```
?category=Gifts'+UNION+SELECT+@@version,NULL-- -
```

**What this does:**
- `@@version` is a global system variable available in both MySQL and MSSQL
- Returns the full database version string including platform, version number, and OS details
- `-- -` is used as the comment sequence on MySQL (note the space after `--` which MySQL requires)

**Important difference from Oracle:**
- Oracle uses `v$version` table with a mandatory `FROM` clause
- MySQL/MSSQL use `@@version` — no FROM clause needed
- MySQL comment syntax requires a space after `--` or use `#`

---

## MySQL vs MSSQL Syntax Comparison

| Task | MySQL | MSSQL |
|------|-------|-------|
| Version | `@@version` | `@@version` |
| Comment | `-- -` or `#` | `--` |
| String concat | `CONCAT(a,b)` | `a+b` |
| Limit rows | `LIMIT 1` | `TOP 1` |

Both share `@@version` but differ on other syntax — always verify the exact platform before deeper enumeration.

---

## What the Response Revealed

The application returned a string similar to:
```
8.0.27-0ubuntu0.20.04.1
```
or for MSSQL:
```
Microsoft SQL Server 2019 (RTM) - 15.0.2000.5
```

This confirms the platform and version — enabling precise attack path selection for subsequent steps.

---

## Real-World Impact

Same as Lab 07 — knowing the exact version allows an attacker to:
- Target version-specific CVEs and known vulnerabilities
- Select the correct syntax for all further enumeration steps
- Identify available stored procedures (especially dangerous on MSSQL — `xp_cmdshell` allows OS command execution)
- Plan privilege escalation paths specific to the platform

---

## Key Takeaway

The same goal (version extraction) requires completely different syntax on different databases. This is why database fingerprinting is always the first step after confirming injection — and why penetration testers maintain a mental map of syntax differences across platforms. Memorise `@@version` for MySQL/MSSQL and `v$version` for Oracle. These are the two most commonly encountered in real assessments.
