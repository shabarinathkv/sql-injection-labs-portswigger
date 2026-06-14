# Lab 04 — SQL Injection UNION Attack: Finding a Column Containing Text

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — identifying text-compatible columns
**Lab URL:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. After determining the number of columns returned by the query (from Lab 03), the next step is identifying which of those columns can display string data in the application response. Not all columns are compatible with text — some may hold integers or other data types, causing errors when a string is injected.

---

## Steps Taken

1. Confirmed from Lab 03 that the query returns 3 columns
2. Attempted to inject a string value into each column position one at a time using NULL placeholders for the others
3. Observed the application response for each attempt:
   - `' UNION SELECT 'a',NULL,NULL--` → database error (column 1 not text-compatible)
   - `' UNION SELECT NULL,'a',NULL--` → string appeared in response ✅ (column 2 accepts text)
   - Stopped here — column 2 confirmed as the target

---

## Payload Used

```
?category=Gifts'+UNION+SELECT+NULL,'a',NULL--
```

**What this does:**
- Attempts to append a row with the string `'a'` in column 2
- If the column accepts text, the value appears in the response
- NULL is used for other columns as it is compatible with any data type

---

## Original Query (inferred)

```sql
SELECT name, description, price FROM products WHERE category = 'Gifts'
```

**After injection:**

```sql
SELECT name, description, price FROM products WHERE category = 'Gifts'
UNION SELECT NULL,'a',NULL--'
```

The string `'a'` appears in the description position — confirming column 2 is the output column for future data extraction.

---

## Why This Step Matters

This is still reconnaissance — but it is critical reconnaissance. Without knowing which column renders text:
- Any UNION-based data extraction attempt will either fail silently or throw an error
- You cannot reliably pull usernames, passwords, or any string data

Finding the right column is the last step before actual exploitation begins.

---

## Real-World Impact

On its own this step causes no direct harm. But combined with Labs 03 and 05, it completes the setup for full credential extraction. An attacker who reaches this step has everything they need to begin pulling sensitive data from any table in the database.

---

## Key Takeaway

UNION attacks require three things before data can be extracted: knowing the column count, knowing which columns accept text, and knowing the target table and column names. This lab covers the second piece. Methodical enumeration — not guessing — is what makes the difference between a failed attack and a successful one.
