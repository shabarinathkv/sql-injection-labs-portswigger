# Lab 01 — SQL Injection in WHERE Clause Allowing Retrieval of Hidden Data

**Difficulty:** Apprentice
**Technique:** Boolean filter bypass
**Lab URL:** https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

---

## Vulnerability

The product category filter on the shopping application passes user input directly into a SQL query without sanitisation. This allows an attacker to manipulate the query logic and retrieve data that should be hidden — in this case, unreleased products not visible to regular users.

---

## Steps Taken

1. Opened the application and clicked a product category filter (e.g. "Gifts")
2. Observed the URL parameter: `?category=Gifts`
3. Injected a single quote `'` to test for SQL errors → application returned an error, confirming unsanitised input
4. Constructed a payload to break out of the WHERE clause and force the query to return all rows
5. Appended `--` to comment out the rest of the original query

---

## Payload Used

```
?category=Gifts'+OR+1=1--
```

**What this does:**
- `'` closes the string input
- `OR 1=1` makes the condition always true — returns every row
- `--` comments out anything after (e.g. `AND released=1`), removing the hidden data filter

---

## Original Query (inferred)

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

**After injection:**

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

The `AND released = 1` filter is now commented out — all products including hidden ones are returned.

---

## Real-World Impact

In a real application, this type of vulnerability could allow an attacker to:
- View hidden or draft content not intended for public access
- Bypass business logic filters (e.g. pricing tiers, access levels)
- Confirm SQLi exists as a foothold for deeper attacks (credential extraction, etc.)

---

## Key Takeaway

Never trust user-supplied input in SQL queries. The fix is parameterised queries (prepared statements) — these separate SQL code from data so injected input is always treated as a string, never as executable logic.
