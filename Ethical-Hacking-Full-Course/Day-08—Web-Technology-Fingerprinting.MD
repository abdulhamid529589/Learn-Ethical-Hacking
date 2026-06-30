# Day 8 — Web Technology Fingerprinting
### Learn Ethical Hacking in 30 Days

---

## 🎯 Objective

Learn how to identify the **backend technologies** of any website using three reconnaissance tools. This is part of the **Footprinting & Reconnaissance** phase in ethical hacking.

---

## 🧠 Why Does This Matter?

When you identify the specific software, frameworks, versions, and infrastructure a website uses, you can:

- Search for known **exploits/CVEs** for specific software versions
- Understand the **attack surface** of a target
- Combine information from multiple tools to build a complete recon dataset
- (For developers) Understand how well-built sites are architected

> ⚠️ **Always perform reconnaissance only on systems you have explicit permission to test.**

---

## 🛠️ Tools Covered

| # | Tool | Type | Best For |
|---|------|------|----------|
| 1 | **Wappalyzer** | Browser Extension | Instant, detailed tech stack detection |
| 2 | **BuiltWith** | Web App | Alternative recon, technology trends |
| 3 | **WhatWeb** | CLI Tool (Kali) | Command-line recon, IP & server info |

---

## 🔧 Tool 1 — Wappalyzer (Browser Extension)

### Installation
1. Open **Firefox** browser
2. Go to **Add-ons / Extensions** → Themes → Extensions
3. Search for **`Wappalyzer`**
4. Click **Add** to install

### What It Detects
- **Web Server** (e.g., Nginx, Apache)
- **CDN** (e.g., jsDelivr, Akamai)
- **Programming Language** (e.g., Java, PHP, Python)
- **JavaScript Libraries** (e.g., React, jQuery, Angular)
- **Platform as a Service** (e.g., Azure, AWS, GCP)
- **Web Application Firewall (WAF)**
- **CMS** (e.g., WordPress, Drupal)
- **Advertising Networks**
- **A/B Testing Tools**
- **Software Versions** ← 🔴 Critical for exploit hunting

### Example Results

**`microsoft.com`**
| Category | Finding |
|----------|---------|
| WAF | Microsoft Azure WAF |
| Language | Java |
| CDN | jsDelivr, Akamai |
| Hosting | Microsoft Azure |
| Advertising | Microsoft Advertising |

**`amazon.in`**
| Category | Finding |
|----------|---------|
| Web Server | Nginx |
| Advertising | Amazon (self-hosted) |

> 💡 **Pro Tip:** Version numbers detected by Wappalyzer (e.g., `Apache 2.4.2`) can be cross-referenced on **CVE databases** to find known vulnerabilities for that specific version.

---

## 🌐 Tool 2 — BuiltWith

### Access
Visit: **[builtwith.com](https://builtwith.com)**

### How to Use
1. Enter the target domain in the search bar (e.g., `youtube.com`, `microsoft.com`)
2. Browse categorized technology results

### What It Detects
- Tracking technologies (e.g., Open Graph Protocol)
- Analytics tools
- Hosting infrastructure
- JavaScript frameworks
- CDN details
- CMS and eCommerce platforms
- Technology trends (how many live sites use a specific tech)

### Notes
- Some **detailed/historical data is behind a paywall**
- Useful as a **cross-reference** when Wappalyzer misses something
- Results may be slightly less real-time than Wappalyzer

---

## 💻 Tool 3 — WhatWeb (CLI)

### About
- **Pre-installed** on Kali Linux — no setup needed
- Command-line based recon tool

### Basic Usage
```bash
whatweb <target-domain>
```

**Example:**
```bash
whatweb microsoft.com
whatweb ndtv.com
```

### What It Returns
- **Country** where the server is located
- **IP Address** of the server
- **Redirect chains** (e.g., HTTP → HTTPS)
- **Server software**
- Sometimes: CMS, framework, error messages

### Example Output (ndtv.com)
```
ndtv.com [200 OK] Country[SINGAPORE], IP[x.x.x.x], ...
```

### Limitations
- May return **limited info** for large/well-protected sites
- Can throw `Timeout` errors on first run — retry if needed
- Some sites actively block fingerprinting requests

---

## 🔁 Workflow: Using All Three Together

```
Target Website
     │
     ├── Wappalyzer      → Tech stack, versions, WAF, CDN
     ├── BuiltWith        → Cross-reference, historical data
     └── WhatWeb (CLI)    → IP address, server location, redirects
          │
          ▼
   Combine all findings
          │
          ▼
   Final Recon Dataset
```

> **Key Insight:** No single tool gives you everything. Use all three and merge the data — what one tool misses, another often catches.

---

## 📝 Summary

| Tool | Strength | Weakness |
|------|----------|----------|
| Wappalyzer | Fast, visual, highly detailed | Browser only |
| BuiltWith | Good cross-reference, trend data | Some features paid |
| WhatWeb | CLI automation, IP/geo info | Limited on hardened sites |

---

## ✅ Key Takeaways

1. **Version numbers are gold** — they directly lead to CVE/exploit research
2. **WAF detection** tells you what defenses are in place
3. **Hosting platform** (Azure, AWS) narrows the attack surface further
4. Always use **multiple tools** and pool the results
5. This phase is called **Footprinting** — gather before you act

---

*Series: Learn Ethical Hacking in 30 Days | Day 8 of 30*