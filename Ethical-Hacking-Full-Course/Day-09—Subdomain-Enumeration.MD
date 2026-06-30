# Day 9 — Subdomain Enumeration
### Learn Ethical Hacking in 30 Days

---

## 🎯 Objective

Learn what subdomains are, why they matter in ethical hacking, and how to enumerate them using dedicated tools during the **Reconnaissance** phase.

---

## 🧠 What is a Subdomain?

A subdomain is a **part (section) of a larger main domain**. It appears **before** the main domain name in a web address.

### Syntax
```
subdomain.maindomain.com

blog.example.com
shop.example.com
api.example.com
```

### Real-World Analogy

> Think of a company like Microsoft as a **building**.
> Instead of cramming every department (HR, Accounts, Reception) into one open floor, they create **separate rooms/cabins** for each department.
> Similarly, instead of putting everything on `microsoft.com`, they create subdomains for separate services.

| Subdomain | Purpose |
|-----------|---------|
| `blog.microsoft.com` | Blog/articles section |
| `shop.microsoft.com` | E-commerce / store |
| `mail.microsoft.com` | Email services |
| `api.microsoft.com` | Developer API |
| `account.paytm.com` | User accounts |

---

## 🔍 What is Subdomain Enumeration?

**Subdomain Enumeration** = The process of finding all subdomains associated with a target domain.

### Why Enumerate Subdomains?

1. **Expand the attack surface** — Each subdomain runs a different service; more targets = more chances to find vulnerabilities
2. **Identify scope** — Understand how large the target organization's infrastructure is
3. **Find forgotten/unpatched services** — Old or neglected subdomains are often vulnerable
4. **Low-hanging fruit** — Subdomains are one of the first places bug bounty hunters look

> 💡 The **main domain** is usually well-secured. Subdomains hosting separate services are far more likely to have vulnerabilities.

---

## 🛠️ Tools Covered

| # | Tool | Type | Notes |
|---|------|------|-------|
| 1 | **Subfinder** | CLI (Kali Linux) | Fast passive enumeration, by ProjectDiscovery |
| 2 | **c99.nl** | Web App | Online subdomain scanner with Cloudflare detection |

---

## 🔧 Tool 1 — Subfinder (CLI)

### About
- Developed by **ProjectDiscovery**
- Pre-available on Kali Linux
- Performs **passive enumeration** (queries public sources, not the target directly)

### Basic Usage
```bash
subfinder -d <target-domain>
```

**Example:**
```bash
subfinder -d microsoft.com
```

### Flag Reference
| Flag | Meaning |
|------|---------|
| `-d` | Specify the target domain |

### Example Output
Running `subfinder -d microsoft.com` returns a long list of subdomains such as:
```
sas-service.microsoft.com
mail.microsoft.com
blog.microsoft.com
...
```

> The output can contain **hundreds of subdomains** depending on the organization size.

---

## 🌐 Tool 2 — c99.nl (Web App)

### Access
Visit: **[c99.nl](https://c99.nl)** → Subdomain Finder

### How to Use
1. Enter your target domain (e.g., `paytm.com`)
2. Click **Start Scan**
3. Browse the results

### What It Returns
- Full list of discovered subdomains
- **Most used IP addresses**
- **Cloudflare status** — tells you if a subdomain is behind Cloudflare protection ← 🔴 Useful!

### Example Output (paytm.com)
```
paytm.com           → Cloudflare: YES
api.paytm.com       → Cloudflare: NO
blog.paytm.com      → Cloudflare: NO
account.paytm.com   → Cloudflare: NO
...
164 subdomains found
```

> 💡 **Cloudflare status matters** — If a subdomain is NOT behind Cloudflare, the real server IP may be exposed, making it an easier target.

---

## 🔁 Subdomain Enumeration Workflow

```
Target Domain (e.g., microsoft.com)
          │
          ├── subfinder -d microsoft.com     → Passive recon, large list
          └── c99.nl scan                    → Online scan + Cloudflare info
                    │
                    ▼
         Compile all subdomains
                    │
                    ▼
         Visit / scan each subdomain
                    │
                    ▼
         Identify vulnerable services
                    │
                    ▼
         Report or exploit (with permission)
```

---

## 📝 Key Concepts Recap

| Term | Meaning |
|------|---------|
| **Subdomain** | A section of a main domain for a specific service |
| **Subdomain Enumeration** | Finding all subdomains of a target |
| **Passive Enumeration** | Querying public data sources without touching the target |
| **Bug Bounty Scope** | Subdomains help define how big a target's attack surface is |
| **Cloudflare** | A WAF/CDN that hides real server IPs — important to detect |

---

## ✅ Key Takeaways

1. **Subdomains = separate services** — each one is an independent target
2. **Main domains are hardened** — subdomains are often neglected and vulnerable
3. **Subfinder** is the go-to CLI tool for fast passive subdomain discovery
4. **c99.nl** adds value with its Cloudflare detection feature
5. Subdomain enumeration is **the first step** in almost every bug bounty hunt
6. Always enumerate subdomains **before** jumping to exploitation

---

*Series: Learn Ethical Hacking in 30 Days | Day 9 of 30*