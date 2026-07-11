# Burp Suite — Proxy Setup & Core Workflow

### Study Notes: Understanding the Intercepting Proxy

---

## Index

1. [What Is Burp Suite?](#1-what-is-burp-suite)
2. [Networking Background: Request/Response Packets](#2-networking-background-requestresponse-packets)
3. [What Is a Proxy Address?](#3-what-is-a-proxy-address)
4. [Why HTTPS Needs a Certificate](#4-why-https-needs-a-certificate)
5. [Lab Setup Guide](#5-lab-setup-guide)
   - 5.1 [Starting Burp Suite](#51-starting-burp-suite)
   - 5.2 [Confirming the Proxy Listener](#52-confirming-the-proxy-listener)
   - 5.3 [Configuring Browser Proxy Settings](#53-configuring-browser-proxy-settings)
   - 5.4 [Installing Burp's CA Certificate](#54-installing-burps-ca-certificate)
   - 5.5 [Removing/Replacing an Old Certificate (Troubleshooting)](#55-removingreplacing-an-old-certificate-troubleshooting)
6. [Core Proxy Concepts](#6-core-proxy-concepts)
   - 6.1 [Intercept On vs. Off](#61-intercept-on-vs-off)
   - 6.2 [Forward vs. Drop](#62-forward-vs-drop)
   - 6.3 [Intercepting Server Responses](#63-intercepting-server-responses)
   - 6.4 [HTTP History](#64-http-history)
7. [Reading a Request Packet](#7-reading-a-request-packet)
8. [Quick Reference Cheat Sheet](#8-quick-reference-cheat-sheet)
9. [Common Errors & Fixes](#9-common-errors--fixes)
10. [Ethical & Legal Notes](#10-ethical--legal-notes)

---

## 1. What Is Burp Suite?

Burp Suite is a **web application penetration testing tool** built around an **intercepting proxy**. It sits between your browser and the target web server, giving you full visibility and control over every HTTP/HTTPS request and response that passes between them.

Editions: **Community** (free, manual features only), **Professional** (paid, adds automated scanning, Intruder without rate limits, extensions, etc.).

**Core use case:** authorized web application security testing — inspecting, modifying, replaying, and analyzing traffic to find vulnerabilities (input validation flaws, auth issues, logic flaws) in applications you own or are contracted to test.

---

## 2. Networking Background: Request/Response Packets

Before understanding Burp, it helps to understand the normal request/response flow:

```
User A (Private IP + MAC)
   │
   ▼
Home Router / Broadband (Public IP + Private IP + MAC)
   │
   ▼
Target Server, e.g. cs.com (Public IP, e.g. 4.4.4.4)
```

- A **private IP** can never talk directly to a **public IP** — traffic must pass through the router, which has both a private IP (to talk to devices on the LAN) and a public IP (to talk to the internet).
- **Request packet**: what your machine sends out asking for something ("I want to access cs.com").
- **Response packet**: whatever the server sends back — could be "yes, here's the page," "no, access denied," "site is down," etc. Any reply, regardless of content, is a response packet.

**Why this matters for testing:** to test a web application properly, you need to see and control the exact contents of both the request going out and the response coming back — not just what the browser renders. That's the gap Burp Suite fills.

---

## 3. What Is a Proxy Address?

Burp Suite listens on a **local proxy address**, by default:

```
127.0.0.1:8080
```

- `127.0.0.1` = localhost (your own machine)
- `8080` = the port Burp's proxy listener is bound to

When you point your browser at this address, **all your browser's traffic routes through Burp first**, before going to the router/internet. Burp then has three choices for each packet:

1. **Forward it unchanged** to the server
2. **Modify it**, then forward the modified version
3. **Drop it** — the packet never reaches the server, and the browser will show a connection error/keep loading indefinitely

This is what makes Burp so powerful for testing: you can alter parameters, headers, cookies, or body data on the fly and observe how the application responds.

---

## 4. Why HTTPS Needs a Certificate

Here's the gap in the flow that causes confusion for beginners:

- Your **request** goes: Browser → Burp → Router → Server (fine, no trust issue — you're just relaying your own outbound request).
- The **response** comes back: Server → Router → Burp → Browser.

The problem: for **HTTPS** traffic, the browser expects the response to be cryptographically signed by a certificate authority (CA) it trusts. Since Burp is sitting in the middle (a deliberate "man-in-the-middle" for testing purposes), it re-signs traffic using **its own CA certificate** (`cacert.der`/`cert.cer`). Unless your browser is told to trust Burp's CA certificate, it will reject the HTTPS traffic and throw certificate warnings/errors, or simply refuse to load HTTPS sites.

**Fix:** download Burp's CA certificate and manually install/trust it in your browser's certificate store. Covered step-by-step below.

---

## 5. Lab Setup Guide

### 5.1 Starting Burp Suite

**Community Edition (Kali/Linux):**

```bash
burpsuite
```

Or launch it from the Kali applications menu (search "burpsuite").

**Professional Edition:** navigate to your install folder and run the executable/jar, e.g.:

```bash
cd ~/Desktop/BurpSuitePro
./BurpSuitePro
```

On first launch: choose **Temporary Project** (or a saved project if you want persistence), then **Use Burp defaults** for configuration (unless you have a saved config profile).

### 5.2 Confirming the Proxy Listener

1. Go to the **Proxy** tab.
2. Click **Options** (or **Proxy Settings**, depending on Burp version).
3. Under **Proxy Listeners**, confirm there's an entry for:
   - Interface: `127.0.0.1:8080`
   - **Running** checkbox is ticked (this means the listener is active and ready to accept browser traffic).

If you need Burp accessible from another device on your network (e.g., testing from a phone), you can bind the listener to `0.0.0.0:8080` — but for a single-machine lab setup, `127.0.0.1:8080` is standard and safer (not exposed to the network).

### 5.3 Configuring Browser Proxy Settings

Using Firefox as the example (recommended for pentesting since it isolates proxy config from OS-wide settings):

1. Open Firefox → **Settings** (☰ menu → Settings, or `about:preferences`).
2. Scroll to **Network Settings** → click **Settings...**
3. Select **Manual proxy configuration**.
4. Set:
   - **HTTP Proxy**: `127.0.0.1` **Port**: `8080`
   - Check **"Also use this proxy for HTTPS"** (ensures both HTTP and HTTPS traffic route through Burp)
5. Click **OK**.

> 💡 **Tip:** Many testers use the **FoxyProxy** browser extension to quickly toggle between "Burp proxy on" and "direct/no proxy" without manually re-editing these settings each time.

### 5.4 Installing Burp's CA Certificate

1. With the proxy configured, open a **new browser tab**.
2. Navigate to: `http://burp` (⚠️ must be plain `http`, no `https`, no trailing slash/path — this is a special address Burp itself intercepts and responds to).
3. You'll see Burp's built-in page with a **"CA Certificate"** download link/button. Click it.
4. This downloads a file like `cacert.der` (Firefox often labels it as a numbered cert if you've downloaded it before, e.g., `cacert(3).der`).
5. In Firefox, go to **Settings** → search **"certificates"** → **View Certificates**.
6. Go to the **Authorities** tab → click **Import**.
7. Select the downloaded certificate file → **Open**.
8. When prompted "Do you want to trust this CA to identify websites?" → check the box → **OK**.
9. Click **OK** again to close the certificate manager.

At this point, Burp is fully wired into your browser: all HTTP/HTTPS traffic routes through the proxy, and your browser trusts Burp's re-signed HTTPS certificates.

### 5.5 Removing/Replacing an Old Certificate (Troubleshooting)

If you've installed Burp's certificate before (e.g., a previous project/reinstall generated a new CA cert) and now get certificate errors:

1. Firefox → **Settings** → search **"certificates"** → **View Certificates**.
2. Go to **Authorities** tab → scroll to find the old entry (often filed under **"PortSwigger"**, Burp's parent company — that's the name to search for).
3. Select it → **Delete or Distrust** → confirm.
4. Re-download a fresh certificate from `http://burp` (as in section 5.4) and re-import it following the same import steps.

This "remove old cert → install fresh cert" cycle is the standard first troubleshooting step whenever Burp/browser HTTPS interception starts throwing errors.

---

## 6. Core Proxy Concepts

### 6.1 Intercept On vs. Off

Located at **Proxy → Intercept**, with a toggle button reading **"Intercept is on"** / **"Intercept is off."**

- **Intercept ON**: every request the browser sends gets paused inside Burp before continuing to the server. You must manually **Forward** or **Drop** each one. Useful when you specifically want to catch and modify a particular request (e.g., a login POST, a parameter you want to tamper with).
- **Intercept OFF**: traffic still flows _through_ Burp (so it still gets logged in HTTP History), but nothing is paused — browsing works normally. This is the default/idle state you should leave Burp in unless you're actively intercepting something specific, otherwise you'll be forwarding dozens of unrelated packets (analytics, favicon requests, ad calls, etc.) just to get to the one you care about.

### 6.2 Forward vs. Drop

While a request is paused under Intercept:

- **Forward**: sends the (possibly edited) packet on to the server as normal.
- **Drop**: the packet is discarded and never reaches the server. The browser tab will typically show a connection error (e.g., "request was dropped") or hang/reload indefinitely, since it's waiting for a response that will never come.

This Forward/Drop/Edit cycle is the fundamental mechanism for **manual request tampering** — modifying parameters, headers, or cookies before they reach the server to test how the application handles unexpected input.

### 6.3 Intercepting Server Responses

By default, Burp **does not** pause responses coming back from the server — only outbound requests. To intercept a response as well:

1. With the relevant request visible/paused in the Intercept tab, **right-click** on it.
2. Select **"Do intercept"** → **"Response to this request."**
3. Now when you Forward that specific request, Burp will pause again once the server's **response** comes back, letting you inspect/modify it before it reaches your browser.

This is a one-time flag per request — it doesn't turn on global response interception, which is intentional (otherwise every single response, including images/CSS/JS, would pause your browsing).

### 6.4 HTTP History

**Proxy → HTTP History** logs **every** request and response that has passed through Burp, whether or not Intercept was on at the time. This is where you'll do most of your actual analysis:

- Click any row to see the full **Request** and **Response** for that transaction.
- Useful views: **Pretty**, **Raw**, **Hex**, **Params** (parsed parameters), depending on Burp version — lets you inspect data in whatever format is most useful.
- This history becomes your reference list when deciding which requests to send to other Burp tools (Repeater, Intruder, etc. — covered in later lessons).

---

## 7. Reading a Request Packet

Example structure of a captured HTTPS request in Burp:

```
GET / HTTP/1.1
Host: cspathshala.com
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml...
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Cookie: session=abc123...
Connection: keep-alive
```

Key fields to understand:
| Field | Meaning |
|---|---|
| `GET` (or `POST`, etc.) | HTTP method — what action is being requested |
| `Host` | Which domain/site this request is meant for |
| `User-Agent` | Browser/client identification string |
| `Accept` / `Accept-Language` / `Accept-Encoding` | Content negotiation headers |
| `Cookie` | Session/auth tokens sent with the request |
| `Content-Length` | Size of the request body (relevant for POST requests) |

The **Response** tab for the same entry shows the server's reply — status code (`200 OK`, `403 Forbidden`, `500 Internal Server Error`, etc.), response headers, and the body (HTML/JSON/etc.).

---

## 8. Quick Reference Cheat Sheet

| Task                           | Where                                                                                   |
| ------------------------------ | --------------------------------------------------------------------------------------- |
| Start Burp                     | `burpsuite` (Community) or run installed binary (Pro)                                   |
| Check proxy listener is active | Proxy → Options → Proxy Listeners                                                       |
| Set browser proxy              | Browser Settings → Network Settings → Manual proxy → `127.0.0.1:8080`                   |
| Trust Burp's cert              | Visit `http://burp` → download CA cert → import under browser's Certificate Authorities |
| Toggle packet interception     | Proxy → Intercept → "Intercept is on/off"                                               |
| Let a paused packet through    | Click **Forward**                                                                       |
| Block a paused packet          | Click **Drop**                                                                          |
| Also pause the server's reply  | Right-click request → Do intercept → Response to this request                           |
| Review all past traffic        | Proxy → HTTP History                                                                    |

---

## 9. Common Errors & Fixes

| Symptom                                                                  | Likely Cause                                                                                   | Fix                                                                                                                               |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Browser says "connection not secure" / cert warnings on every HTTPS site | Burp's CA cert not trusted by browser                                                          | Re-do §5.4 (download + import cert)                                                                                               |
| HTTPS sites won't load at all through Burp                               | Old/mismatched cert installed, or proxy not actually set                                       | Remove old cert (§5.5), re-verify proxy settings (§5.3)                                                                           |
| Page keeps reloading forever                                             | Intercept is ON and the request is sitting paused, unforwarded                                 | Go to Proxy → Intercept and click **Forward** (or turn Intercept off)                                                             |
| Page shows "request dropped" style error                                 | You clicked **Drop** on that request                                                           | Reload the page to generate a fresh request; Forward it this time                                                                 |
| Can't reach `http://burp` page at all                                    | Proxy not actually configured in browser                                                       | Re-verify §5.3 exactly — check IP is `127.0.0.1` and port is `8080`, and "also use for HTTPS" is checked                          |
| Traffic isn't appearing in Burp at all                                   | Browser bypassing proxy (e.g., using a different profile, or an extension overriding settings) | Confirm you're using the correct browser profile; disable conflicting proxy extensions; test with FoxyProxy for reliable toggling |

---

## 10. Ethical & Legal Notes

- Burp Suite is a legitimate, industry-standard tool used daily by professional penetration testers and bug bounty hunters — but its interception/tampering capabilities mean it must **only be pointed at applications you own or have explicit written authorization to test** (e.g., your own local dev sites, deliberately vulnerable practice apps like **DVWA**, **OWASP Juice Shop**, **bWAPP**, or in-scope bug bounty targets).
- Never proxy traffic to third-party production websites without authorization, even "just to look" — passively viewing traffic through an intercepting proxy against a site you don't have permission to test can still constitute unauthorized access/testing depending on jurisdiction and the site's terms of service.
- When you're done testing, revert your browser's proxy settings back to **"No proxy"** / **system default** so your normal browsing isn't routed through Burp (and so Burp doesn't log unrelated personal browsing traffic).
- Good practice for a personal lab: keep a **separate browser profile** (or a dedicated browser like a Firefox "pentest" profile) configured with the Burp proxy, and use your normal browser/profile for everyday browsing — this avoids accidentally routing sensitive personal traffic (banking, email, etc.) through an intercepting proxy.

---

### Suggested Next Practice Steps (Your Own Lab Only)

1. Install **DVWA** or **OWASP Juice Shop** locally (Docker is the easiest route) and point Burp at it.
2. Practice toggling Intercept, forwarding/dropping requests, and reviewing HTTP History end-to-end.
3. Try flagging "Do intercept → Response to this request" on a login POST to see both the request and response for a single transaction.
4. Once comfortable with the proxy basics here, move on to Burp's **Repeater** (resend/modify a single request repeatedly) and **Intruder** (automated fuzzing) tools in your next study session — natural next steps from this proxy foundation.
