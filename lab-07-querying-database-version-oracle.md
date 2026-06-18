# Lab 07 — SQL Injection Attack: Querying the Database Type and Version on Oracle

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — database fingerprinting on Oracle
**Lab URL:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. This lab introduces database fingerprinting — identifying which database platform the application is running on. Knowing the database type is essential because each platform (Oracle, MySQL, MSSQL, PostgreSQL) has different syntax, system tables, and functions. The wrong syntax on the wrong platform means the attack fails entirely.

---

## Steps Taken

1. Confirmed SQLi via single quote injection — server returned an error
2. Determined column count using ORDER BY — confirmed 2 columns
3. Confirmed both columns accept text using NULL probing
4. Identified this as an Oracle database — Oracle requires a FROM clause in every SELECT statement, unlike other databases
5. Queried the Oracle-specific version table `v$version` to extract the database banner
6. Injected the version query into the UNION and observed the database version string in the response

---

## Payload Used

```
?category=Gifts'+UNION+SELECT+BANNER,NULL+FROM+v$version--
```

**What this does:**
- `UNION SELECT` appends a new row to the query result
- `BANNER` is a column in Oracle's `v$version` table that contains the database version string
- `FROM v$version` is Oracle-specific — this table does not exist in MySQL or MSSQL
- `NULL` fills the second column

**Oracle-specific rule:** Every SELECT in Oracle must include a FROM clause. On other databases `SELECT 'test',NULL--` works fine. On Oracle it must be `SELECT 'test',NULL FROM dual--` (using the built-in dummy table `dual`).

---

## What the Response Revealed

The application returned a string similar to:
```
Oracle Database 11g Express Edition Release 11.2.0.2.0
```

This tells the attacker:
- Exact database platform (Oracle)
- Version number
- Which known vulnerabilities apply to this version
- Which attack syntax to use for all further exploitation

---

## Why Database Fingerprinting Matters

Without knowing the database platform, an attacker is guessing syntax and wasting time. Fingerprinting is always done early in an assessment because it determines every subsequent step:

| Database | Version Query | Dummy Table Needed |
|----------|--------------|-------------------|
| Oracle | `SELECT banner FROM v$version` | Yes — `FROM dual` |
| MySQL | `SELECT @@version` | No |
| MSSQL | `SELECT @@version` | No |
| PostgreSQL | `SELECT version()` | No |

---

## Real-World Impact

Knowing the exact database version allows an attacker to:
- Target known unpatched CVEs for that specific version
- Choose the correct syntax for further enumeration
- Identify available functions and features for deeper exploitation
- Adapt out-of-band or time-based techniques to platform-specific methods

---

## Key Takeaway

Fingerprinting is reconnaissance. A methodical attacker always identifies the target platform before attempting data extraction. Skipping this step leads to failed payloads and wasted effort. On Oracle specifically — always remember the mandatory `FROM` clause and the `dual` dummy table.
