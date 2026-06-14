# Lab 05 — SQL Injection UNION Attack: Retrieving Data from Other Tables

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — cross-table data extraction
**Lab URL:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. With the column count and text-compatible column already identified (Labs 03 and 04), this lab demonstrates the full impact of a UNION attack — extracting usernames and passwords directly from a separate users table that the application never intended to expose.

---

## Steps Taken

1. Confirmed from previous labs: query returns 2 columns, both accept text
2. Identified the target: a `users` table containing `username` and `password` columns (hinted by the lab)
3. Constructed a UNION query to append rows from the users table to the product results
4. Injected the payload and observed the response — usernames and passwords appeared in the product listing area
5. Used the extracted administrator credentials to log in and complete the lab

---

## Payload Used

```
?category=Gifts'+UNION+SELECT+username,password+FROM+users--
```

**What this does:**
- Appends rows from the `users` table to the original product query result
- Both `username` and `password` values are returned in the two text-compatible columns
- The application renders them on the page just like normal product data

---

## Original Query (inferred)

```sql
SELECT name, description FROM products WHERE category = 'Gifts'
```

**After injection:**

```sql
SELECT name, description FROM products WHERE category = 'Gifts'
UNION SELECT username, password FROM users--'
```

The application now returns both product rows and user credential rows in the same response — completely unaware that it is leaking sensitive data.

---

## Real-World Impact

This is the most dangerous outcome of UNION-based SQL injection. In a real application an attacker could:
- Extract every username and password from the database in a single request
- Use credentials to take over user accounts including admins
- Access payment data, personal information, private messages, or any other stored data
- Pivot into deeper system access using compromised admin credentials

Even if passwords are hashed, a credential dump of this scale gives an attacker everything they need for offline cracking.

---

## Key Takeaway

This lab connects all the pieces from the previous three labs into one complete attack chain. Column count → text column identification → data extraction. Each step built on the last. This is exactly how a real SQL injection attack progresses — methodical, incremental, and devastating when completed.

The fix is always the same: parameterised queries. They make this entire attack chain impossible by separating SQL logic from user input entirely.
