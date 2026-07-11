# Burp Suite — Target, Proxy & Spider Features

### Study Notes: Scoping, Filtering, and Crawling a Target

---

## Index

1. [Overview](#1-overview)
2. [Recap: Verifying Browser-Proxy Connection](#2-recap-verifying-browser-proxy-connection)
3. [The Target Tab](#3-the-target-tab)
   - 3.1 [What Target Solves](#31-what-target-solves)
   - 3.2 [Site Map Sub-Tab](#32-site-map-sub-tab)
4. [Defining Scope (Step-by-Step)](#4-defining-scope-step-by-step)
   - 4.1 [Method A — Add Directly from Site Map](#41-method-a--add-directly-from-site-map)
   - 4.2 [Method B — Capture via Intercept, Then Add to Scope](#42-method-b--capture-via-intercept-then-add-to-scope)
   - 4.3 [Confirming Your Scope](#43-confirming-your-scope)
5. [Filtering the Site Map to Show Only In-Scope Items](#5-filtering-the-site-map-to-show-only-in-scope-items)
6. [Reading the Site Map](#6-reading-the-site-map)
   - 6.1 [Tree Structure](#61-tree-structure)
   - 6.2 [Table Columns](#62-table-columns)
   - 6.3 [Request/Response Panels](#63-requestresponse-panels)
7. [The Spider Feature](#7-the-spider-feature)
   - 7.1 [What Spidering Is (Web Crawling Concept)](#71-what-spidering-is-web-crawling-concept)
   - 7.2 [Step-by-Step: Running Spider on Your Target](#72-step-by-step-running-spider-on-your-target)
   - 7.3 [Resource Management: Pause/Resume](#73-resource-management-pauseresume)
8. [Why URL Discovery Matters for Testing](#8-why-url-discovery-matters-for-testing)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)
10. [Ethical & Legal Notes](#10-ethical--legal-notes)
11. [Hands-On Lab: DVWA / OWASP Juice Shop Setup](#11-hands-on-lab-dvwa--owasp-juice-shop-setup)
12. [Hands-On Lab: Burp Repeater & Intruder](#12-hands-on-lab-burp-repeater--intruder)

---

## 1. Overview

This session covers three interconnected Burp Suite features that form the backbone of manual web testing workflow:

| Feature    | Purpose                                                                                                                       |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **Proxy**  | Intercept, view, and modify live traffic between browser and server                                                           |
| **Target** | Define _which_ host(s) you actually care about (your "scope") and view their captured data as an organized site map           |
| **Spider** | Automatically crawl a target site, discovering every reachable URL/page/file so you have a complete map of the attack surface |

These three work together: **Proxy** captures raw traffic → **Target** organizes and scopes it → **Spider** actively expands it by automatically visiting every discoverable link.

---

## 2. Recap: Verifying Browser-Proxy Connection

Before using Target/Spider, confirm your proxy chain is still working (see previous lesson for full first-time setup):

1. Browser → Settings → search **"proxy"** → **Settings...**
2. Confirm **Manual proxy configuration** is selected with:
   - HTTP Proxy: `127.0.0.1`
   - Port: `8080`
   - **"Also use this proxy for HTTPS"** checked
3. Click OK.

> **Why do this manually instead of using FoxyProxy?** A browser extension like FoxyProxy is convenient for toggling proxies on/off quickly, but if it ever misbehaves or errors out, you won't know how to fix it unless you understand the underlying manual process. Learn the manual method first — automate later once you understand what's happening in the background.

---

## 3. The Target Tab

### 3.1 What Target Solves

When Burp's proxy is active, your browser generates traffic to **many** domains simultaneously — the actual site you're testing, plus analytics, CDNs, ad networks, font providers, tracking pixels, etc. The **Target** tab lets you cut through this noise and focus only on the host(s) you're actually authorized to test.

**Core question Target answers:** _"Out of everything Burp has seen, what do I actually care about?"_

### 3.2 Site Map Sub-Tab

Inside **Target**, the **Site Map** sub-tab shows every domain Burp has observed traffic for, organized as a browsable tree (left panel) plus a flat table (right panel) of individual requests.

---

## 4. Defining Scope (Step-by-Step)

"Scope" = the specific host(s) you've told Burp you're actually testing. Once scoped, you can filter almost every other Burp tool to only show/act on in-scope items.

### 4.1 Method A — Add Directly from Site Map

1. Go to **Target → Site Map**.
2. Scroll the tree/table until you find your target domain (e.g., `cspathshala.com`) — it will already be listed if any traffic to it has passed through Burp.
3. **Right-click** the domain entry → select **"Add to scope."**
4. A confirmation dialog appears (sometimes asking whether to also update the display filter) → click **Yes**.

### 4.2 Method B — Capture via Intercept, Then Add to Scope

Use this when the target hasn't appeared in the Site Map yet:

1. Go to **Proxy → Intercept**, make sure it's toggled **ON**.
2. Reload your target website in the browser.
3. Requests will start appearing in Intercept — many will be unrelated (ads, YouTube embeds, third-party scripts, etc.). Click **Forward** repeatedly to pass these through.
4. Once you see a request whose **Host** header matches your actual target (e.g., `cspathshala.com`), **stop** — don't forward it yet.
5. **Right-click** that captured request → **"Add to scope"** (or first **"Send to Spider,"** covered in §7).
6. Turn Intercept back **OFF** once done, so normal browsing resumes without repeated manual forwarding.

### 4.3 Confirming Your Scope

1. In **Target**, next to **Site Map**, click the **Scope** sub-tab.
2. You should see your added domain(s) listed under the scope rules — this confirms Burp now recognizes that host as your defined target.
3. If prompted with an option like **"Enable 'Show only in-scope items' by default"** or a similar out-of-scope management prompt, you can enable it here for convenience.

---

## 5. Filtering the Site Map to Show Only In-Scope Items

Even after setting scope, the Site Map table will still _display_ all captured domains unless you apply a filter:

1. In **Target → Site Map**, look for the **Filter** bar (usually just above the table, labeled something like _"Filter: Hiding not found items..."_).
2. Click on the filter bar to expand the filter options panel.
3. Check the box **"Show only in-scope items."**
4. Leave all other filter options at default for now (you'll explore MIME-type filters, status-code filters, etc. later).
5. Click anywhere outside the filter panel to collapse it — the filter applies automatically.

**Result:** the Site Map tree/table now shows _only_ your target domain and its associated data — no more YouTube, ad-network, or unrelated traffic cluttering the view.

---

## 6. Reading the Site Map

### 6.1 Tree Structure

- Click the small **arrow/expand icon** next to your domain in the left-hand tree to reveal its structure: subfolders, individual pages (e.g., `robots.txt`, `refund-policy.html`, `payment.html`), script files, CSS files, etc.
- This tree mirrors the actual folder/file structure Burp has _observed_ being requested — it is **not** a guess or a full filesystem listing, only what traffic has actually passed through so far. This is exactly why **Spider** (§7) matters — it actively expands this list beyond what you've manually browsed to.

### 6.2 Table Columns

The right-hand table (when you click a node) shows one row per captured request, with columns typically including:

| Column                | Meaning                                                               |
| --------------------- | --------------------------------------------------------------------- |
| **Host**              | The domain the request was sent to                                    |
| **Method**            | `GET`, `POST`, etc.                                                   |
| **URL**               | The specific path requested                                           |
| **Params**            | Ticked if the request included parameters (query string or POST body) |
| **Status**            | HTTP response status code (see table below)                           |
| **Length**            | Size of the response                                                  |
| **MIME type / Title** | Content type and page title, where available                          |

**HTTP Status Code Quick Reference:**
| Range | Meaning |
|---|---|
| `2xx` (e.g., 200) | Success — page loaded correctly |
| `3xx` (e.g., 301/302) | Redirect |
| `4xx` (e.g., 403, 404) | Client error — Forbidden / Not Found |
| `5xx` (e.g., 500) | Server error |

### 6.3 Request/Response Panels

- Click any row in the table → the lower/side panels show the full **Request** (what your browser/Burp sent) and **Response** (what the server sent back).
- Example: clicking a `robots.txt` entry shows the request asking for that file, and the response panel shows the raw file contents (e.g., `Disallow: /admin`, `Allow: /`, references to a sitemap.xml, etc.) — useful for quickly reviewing crawl directives and discovering disallowed (often _interesting_) paths.

---

## 7. The Spider Feature

### 7.1 What Spidering Is (Web Crawling Concept)

**Web crawling / spidering** = automatically visiting a page, extracting every link found on it, then visiting _those_ links, and repeating recursively — building up a complete map of a website's reachable structure.

**Why it matters for testing:** the more URLs, pages, and parameters you discover, the more surface area you have to test for vulnerabilities. Manually clicking through every page and folder of a website would take enormous time; Spider automates this discovery process in seconds to minutes.

### 7.2 Step-by-Step: Running Spider on Your Target

1. **Before starting:** make sure **Proxy → Intercept is turned OFF.** If Intercept is on, Spider's automated requests will get stuck waiting for manual Forward/Drop action, and you won't see results flow in properly.
2. Go to **Target → Site Map**.
3. **Right-click** your target domain → select **"Spider this host."**
4. Confirm the prompt (e.g., _"Do you want to spider this host?"_) → **Yes**.
5. Spidering begins immediately — you'll see the entry counts and discovered URLs under your domain in the Site Map growing in near real-time.
6. Within seconds, you should see significantly more entries than before: image folders, CGI-bin contents, additional scripts, config-adjacent files, etc. — whatever Spider was able to reach by following links.

### 7.3 Resource Management: Pause/Resume

- Spidering is CPU-intensive and can also generate significant load against the **target server** (since it's actively requesting many pages in quick succession).
- Watch your own machine's CPU usage while Spider runs — heavy sites can noticeably load your system.
- **Best practice:** don't leave Spider running continuously and unattended. Start it, let it gather results for a bit, then **pause** it (via the Spider control panel — Spider status showing running/paused). Resume later as needed.
- This protects both your own machine from overload **and** avoids hammering the target server with sustained automated traffic — especially important as a courtesy/safety practice even in your own lab, and absolutely critical if ever spidering a real authorized engagement target (uncontrolled crawl speed can degrade a production site's performance).

---

## 8. Why URL Discovery Matters for Testing

The core professional logic tying this whole lesson together:

> More discovered URLs → more pages/functions to examine → more potential input points (parameters, forms, file uploads, API endpoints) → more opportunities to find real vulnerabilities.

This is why reconnaissance-style crawling (Spider here, or CLI tools like `gau`/`katana`/`waybackurls` from the earlier recon lesson) is treated as a _foundational_ step before any actual vulnerability testing (SQLi, XSS, auth bypass, etc.) begins. You can't test what you don't know exists.

---

## 9. Quick Reference Cheat Sheet

| Task                         | Steps                                                                                         |
| ---------------------------- | --------------------------------------------------------------------------------------------- |
| Add a target to scope        | Target → Site Map → right-click domain → **Add to scope**                                     |
| Confirm scope                | Target → **Scope** sub-tab                                                                    |
| Show only in-scope traffic   | Target → Site Map → **Filter** bar → check **"Show only in-scope items"**                     |
| Expand site structure        | Click the arrow icon next to the domain in the tree                                           |
| View a request/response      | Click any row in the table → check **Request**/**Response** panels                            |
| Crawl a target automatically | Target → Site Map → right-click domain → **Spider this host** (ensure Intercept is OFF first) |
| Manage Spider load           | Pause/resume Spider periodically instead of running continuously                              |

---

## 10. Ethical & Legal Notes

- **Spidering is active reconnaissance** — it generates real traffic against a real server. Only ever run Spider against:
  - Sites/apps **you personally own**
  - Deliberately vulnerable practice apps (DVWA, OWASP Juice Shop, bWAPP, etc.) run **locally** in your own lab
  - Targets explicitly **in-scope** under a signed bug bounty program or client engagement
- Never spider third-party production sites without authorization — even "just crawling" can be considered unauthorized scanning/access under computer misuse laws in many jurisdictions, and can also degrade server performance for real users.
- Always respect `robots.txt` directives and rate limits when operating under any authorized program's rules of engagement, even though tools like Spider _can_ technically ignore them.

---

## 11. Hands-On Lab: DVWA / OWASP Juice Shop Setup

This section gives you a safe, legal target to practice everything above (Proxy, Target, Scope, Spider) plus the tools in the next section (Repeater, Intruder).

### 11.1 Prerequisites

```bash
# Install Docker on Kali (if not already installed)
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER   # then log out/in, or run `newgrp docker`
```

### 11.2 Option A — OWASP Juice Shop (recommended: modern, realistic, self-contained)

```bash
docker pull bkimminich/juice-shop
docker run -d -p 3000:3000 --name juiceshop bkimminich/juice-shop
```

- Open in your browser: `http://localhost:3000`
- This is a full modern JavaScript web app (Angular/Node) with dozens of intentional vulnerabilities (SQLi, XSS, broken auth, IDOR, etc.) — excellent for realistic Burp practice.
- Built-in **scoreboard** (`http://localhost:3000/#/score-board`) tracks which challenges you've solved — gamified learning.

### 11.3 Option B — DVWA (classic, simpler, PHP/MySQL — good for fundamentals)

```bash
docker pull vulnerables/web-dvwa
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa
```

> Using port `8081` here deliberately, to avoid clashing with Burp's own `8080` proxy port.

- Open in your browser: `http://localhost:8081`
- Default login: `admin` / `password`
- After logging in, go to **DVWA Security** (left sidebar) and set difficulty to **Low** for your first pass through each vulnerability category — increase difficulty as you improve.

### 11.4 Connecting Burp to Your Local Lab App

1. Confirm your browser's proxy is still set to `127.0.0.1:8080` (per §2).
2. Confirm Burp's Proxy listener is active (Proxy → Options → Proxy Listeners).
3. Browse to `http://localhost:3000` (Juice Shop) or `http://localhost:8081` (DVWA).
4. In **Target → Site Map**, find `localhost` in the list → right-click → **Add to scope**.
5. Apply the **"Show only in-scope items"** filter (§5) so your view is clean.
6. Log in / click around the app manually for a minute so Burp captures some real requests (login, registration, search, product pages, etc.).
7. Right-click `localhost` → **Spider this host** to auto-discover the rest of the app's structure.

You now have a fully scoped, fully mapped local target — safe to test against as aggressively as you like, since it's entirely under your own control.

### 11.5 Cleanup / Management Commands

```bash
docker ps                          # see running containers
docker stop juiceshop dvwa         # stop both labs
docker start juiceshop dvwa        # resume later
docker rm -f juiceshop dvwa        # fully remove when done
```

---

## 12. Hands-On Lab: Burp Repeater & Intruder

Natural next steps once you're comfortable with Proxy/Target/Spider. Practice these **only** against your local DVWA/Juice Shop containers from §11.

### 12.1 Repeater — Manually Resend & Modify a Single Request

**Purpose:** take one specific request, tweak it, and resend it as many times as you like — without needing to re-trigger it from the browser each time. This is the primary tool for manual, deliberate testing of a single input point (e.g., trying different payloads in one parameter).

**Step-by-step:**

1. Turn **Proxy → Intercept ON**.
2. In your browser, trigger the request you want to test — e.g., submit the DVWA/Juice Shop **login form**, or load a page with a URL parameter like `http://localhost:8081/vulnerabilities/sqli/?id=1&Submit=Submit`.
3. When the request appears in Intercept, **right-click** it → **"Send to Repeater"** (or use the keyboard shortcut, typically `Ctrl+R`).
4. Forward or drop the original captured request as needed (it already made its way to Repeater regardless).
5. Go to the **Repeater** tab. You'll see your captured request loaded in an editable panel.
6. Edit any part of the request directly — e.g., change `id=1` to `id=1' OR '1'='1` to test for SQL injection, or change a `username`/`password` value.
7. Click **Send** (or the ▶ button).
8. The **Response** panel updates immediately, showing exactly how the server reacted to your modified request.
9. Repeat — change the value again, hit Send again, compare responses. Repeater keeps a history so you can flip back through previous attempts within the same tab.

**Practice exercise (DVWA SQL Injection, Low security):**

- Send the SQLi page's request to Repeater.
- Try payloads like `1`, `1'`, `1' OR '1'='1`, `1' UNION SELECT null,null-- -` one at a time in the `id` parameter, observing how the response (and any SQL error messages) changes each time. This is exactly how manual SQLi discovery/confirmation works.

### 12.2 Intruder — Automated Payload Fuzzing

**Purpose:** automatically send _many_ variations of a request (e.g., trying an entire wordlist against a login form, or a list of SQLi/XSS payloads against a parameter) and compare all the responses at once — far faster than manually testing one value at a time in Repeater.

> **Note:** Burp Community Edition **throttles Intruder's speed** significantly (it's a paid-feature incentive in Professional). For learning purposes and small wordlists against your own local lab, Community is still perfectly usable — it will just be slower on large wordlists.

**Step-by-step — Example: brute-forcing DVWA login (Low security, Brute Force module):**

1. Turn **Proxy → Intercept ON**.
2. In the browser, go to DVWA's **Brute Force** page and submit the login form with any placeholder username/password (e.g., `admin` / `test`).
3. In Intercept, right-click the captured POST request → **"Send to Intruder."**
4. Forward the original request through (or drop it — doesn't matter, it's already copied to Intruder).
5. Go to the **Intruder** tab → **Positions** sub-tab.
6. You'll see the full request with Burp auto-highlighting likely parameter values in `§...§` markers. Click **"Clear §"** to remove all auto-selected markers first, for a clean slate.
7. Manually highlight just the **password** value in the request body → click **"Add §"** to mark it as the injection point (leave `username` as a fixed value like `admin` for this exercise).
8. Set **Attack type** to **Sniper** (single payload position, one wordlist — the default and simplest mode).
9. Go to the **Payloads** sub-tab.
10. Under **Payload type**, keep **Simple list**.
11. Under **Payload Options**, either:
    - Manually type in a few candidate passwords, one per line (`password`, `123456`, `admin`, `letmein`), or
    - Click **Load** and select a wordlist file (e.g., a small custom `.txt`, or `/usr/share/wordlists/rockyou.txt` for a real exercise — but note Community Edition's throttling makes very large lists slow; use a trimmed-down list of a few hundred entries for practice).
12. Click **Start attack** (top-right).
13. A results window opens, showing one row per attempted password, with columns for **Status code**, **Response length**, etc.
14. **Key analysis technique:** sort by **Length** or **Status**. A successful login typically produces a _different_ response length or status code than all the failed attempts (e.g., a redirect to a welcome page vs. an "Invalid credentials" message of consistent length). The odd-one-out row is very likely your successful password.

**Attack type reference (for later, more advanced use):**
| Attack Type | Behavior |
|---|---|
| **Sniper** | One payload set, cycled through a single (or multiple, one-at-a-time) position(s) |
| **Battering ram** | Same single payload inserted into _all_ marked positions simultaneously |
| **Pitchfork** | Multiple payload sets, one per position, stepped through in parallel (e.g., paired username:password lists from a breach dump) |
| **Cluster bomb** | Multiple payload sets, _all combinations_ tried (e.g., every username × every password) — most thorough, but slowest |

### 12.3 Practice Checklist

- [ ] Send a DVWA/Juice Shop login request to Repeater; manually test 3–4 different payloads and record the response differences.
- [ ] Send a parameterized request (e.g., DVWA SQLi `id` parameter) to Repeater; confirm you can identify a SQL error message by altering the input.
- [ ] Send the DVWA Brute Force login form to Intruder; run a Sniper attack with a small password list; identify the successful login by response-length anomaly.
- [ ] Try a **Cluster bomb** attack with a small username list × small password list against the same Brute Force form, to see how combination-testing works.
- [ ] Compare Repeater (manual, deliberate, one-at-a-time) vs. Intruder (automated, bulk) — note when you'd realistically use each in a real assessment (Repeater for confirming/refining a specific finding; Intruder for broad automated coverage).

---

### What's Next

Once comfortable with Target/Proxy/Spider/Repeater/Intruder, the natural next Burp topics to study are:

- **Decoder** (encode/decode data — Base64, URL-encoding, hashing — useful when payloads or tokens are encoded)
- **Comparer** (diff two requests/responses side-by-side — useful for spotting subtle differences Intruder's table view might miss)
- **Extensions/BApp Store** (community-built add-ons, e.g., for JWT manipulation, additional scanners, etc.)
- Moving from DVWA/Juice Shop practice toward a real (authorized) bug bounty program once your fundamentals are solid.
