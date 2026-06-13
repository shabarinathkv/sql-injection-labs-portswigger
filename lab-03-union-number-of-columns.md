# Lab 03 — SQL Injection UNION Attack: Determining Number of Columns

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — column count enumeration
**Lab URL:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. Before data can be extracted using a UNION attack, the attacker must first determine how many columns the original query returns — because a UNION SELECT must match the exact column count of the original query or it will throw an error.

---

## Steps Taken

1. Confirmed SQLi by injecting `'` into the category parameter — server returned an error
2. Used ORDER BY to determine column count — incrementing the number until an error occurs:
   - `'+ORDER+BY+1--` → no error
   - `'+ORDER+BY+2--` → no error
   - `'+ORDER+BY+3--` → no error
   - `'+ORDER+BY+4--` → **error** — confirms the query returns exactly 3 columns
3. Verified using NULL-based UNION:
   - `'+UNION+SELECT+NULL,NULL,NULL--` → success (3 NULLs = 3 columns confirmed)

---

## Payload Used

**ORDER BY method:**
```
?category=Gifts'+ORDER+BY+3--
```

**UNION NULL verification:**
```
?category=Gifts'+UNION+SELECT+NULL,NULL,NULL--
```

**What this does:**
- `ORDER BY N` references the Nth column — if N exceeds the column count, the database throws an error
- `UNION SELECT NULL,NULL,NULL` attempts to append a row — NULL is compatible with any data type, making it safe for probing

---

## Why This Matters

Column count enumeration is always **step one** of a UNION attack. Without knowing the exact number of columns:
- The UNION SELECT will fail
- No data can be extracted

This is a reconnaissance step — it builds the foundation for the next attack (finding which columns display data, then extracting credentials or sensitive information).

---

## Real-World Impact

On its own this step causes no direct harm — but it is the gateway to:
- Extracting usernames and passwords from the database
- Reading data from other tables (orders, payment info, PII)
- Mapping the entire database structure

---

## Key Takeaway

UNION attacks require precision. Understanding column count and data types before attempting extraction is what separates methodical penetration testing from random payload throwing. This lab builds the discipline of enumeration before exploitation.
