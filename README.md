# SQL Injection Labs — PortSwigger Web Security Academy

A documented walkthrough of all 18 SQL Injection labs completed on [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection).

Each writeup follows a consistent format: vulnerability description, steps taken, payload used, and real-world impact. The goal is not just to solve labs — but to understand *why* the exploit works and what it means in a real application.

---

## About Me

Cybersecurity student from India, currently pursuing my degree with a focus on penetration testing and network security. These labs are part of my hands-on journey into offensive security.

- LinkedIn: https://www.linkedin.com/in/sabarinathkv/
- Email: sabarinathkv369@gmail.com

---

## Lab Index

| # | Lab Title | Technique | Difficulty |
|---|-----------|-----------|------------|
| 01 | Lab 01 — [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](lab-01-where-clause-hidden-data.md) | UNION / Filter bypass | Apprentice |
| 02 | Lab 02 — [SQL injection vulnerability allowing login bypass](lab-02-login-bypass.md) | Comment injection | Apprentice |
| 03 | Lab 03 — [SQL injection UNION attack — determining number of columns](lab-03-union-number-of-columns.md) | UNION-based | Practitioner |
| 04 | Lab 04 — [SQL injection UNION attack, finding a column containing text](lab-04-union-find-text-column.md)| UNION-based | Practitioner |
| 05 | Lab 05 — [SQL injection UNION attack, retrieving data from other tables](lab-05-union-retrieve-data-other-tables.md) | UNION-based | Practitioner |
| 06 | Lab 06 — [SQL injection UNION attack, retrieving multiple values in a single column](lab-06-union-multiple-values-single-column.md) | UNION-based | Practitioner |
| 07 | Lab 07 — SQL injection attack, querying the database type and version (Oracle) | Error-based | Practitioner |
| 08 | Lab 08 — SQL injection attack, querying the database type and version (MySQL/MSSQL) | Error-based | Practitioner |
| 09 | Lab 09 — SQL injection attack, listing the database contents (non-Oracle) | Enumeration | Practitioner |
| 10 | Lab 10 — SQL injection attack, listing the database contents (Oracle) | Enumeration | Practitioner |
| 11 | Lab 11 — Blind SQL injection with conditional responses | Blind / Boolean-based | Practitioner |
| 12 | Lab 12 — Blind SQL injection with conditional errors | Blind / Error-based | Practitioner |
| 13 | Lab 13 — Visible error-based SQL injection | Error-based | Practitioner |
| 14 | Lab 14 — Blind SQL injection with time delays | Blind / Time-based | Practitioner |
| 15 | Lab 15 — Blind SQL injection with time delays and information retrieval | Blind / Time-based | Practitioner |
| 16 | Lab 16 — Blind SQL injection with out-of-band interaction | Out-of-band | Expert |
| 17 | Lab 17 — Blind SQL injection with out-of-band data exfiltration | Out-of-band | Expert |
| 18 | Lab 18 — SQL injection with filter bypass via XML encoding | Filter bypass | Expert |

---

## Key Concepts Covered

- **In-band SQLi** — UNION-based and error-based techniques
- **Blind SQLi** — Boolean-based, time-based, and out-of-band methods
- **Database enumeration** — Extracting table names, column names, credentials
- **Filter bypass** — Obfuscation and encoding to evade WAF rules
- **Impact assessment** — Understanding attacker goals in real-world scenarios

## Tools Used

- Burp Suite (Community Edition)
- Manual payload crafting
- HTTP request interception and manipulation

---

## What I Learned

SQL injection remains one of the most critical vulnerabilities in web applications (OWASP Top 10 — A03:2021). Through these labs I developed the ability to:

1. Identify SQLi entry points through error observation and response analysis
2. Enumerate database structure without triggering obvious errors
3. Extract sensitive data using both visible and blind techniques
4. Think like an attacker — and understand how defenders should patch these flaws

---

*Completed as part of self-directed learning in offensive web security. All testing performed in isolated lab environments only.*
