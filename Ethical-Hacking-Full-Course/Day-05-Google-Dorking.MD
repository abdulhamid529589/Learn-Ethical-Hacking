# Day 05 — Passive Reconnaissance: Google Dorking

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 04 — Reconnaissance](#) | Next: [Day 06](#)

---

## 📋 Table of Contents

1. [What is Google Dorking?](#1-what-is-google-dorking)
2. [Google Dork Operators](#2-google-dork-operators)
   - [site:](#site)
   - [filetype:](#filetype)
   - [intitle:](#intitle)
   - [intext:](#intext)
   - [inurl:](#inurl)
   - [index of](#index-of)
3. [Combining Operators (Power Queries)](#3-combining-operators-power-queries)
4. [Real-World Attack Scenarios](#4-real-world-attack-scenarios)
5. [Google Hacking Database (GHDB)](#5-google-hacking-database-ghdb)
6. [Quick Reference Cheat Sheet](#6-quick-reference-cheat-sheet)

---

## 1. What is Google Dorking?

**Google Dorking** (also called **Google Hacking**) is a passive reconnaissance technique that uses **advanced Google search operators** to find:

- Hidden or sensitive data on websites
- Exposed configuration files
- Publicly accessible files (PDFs, Excel sheets, text files)
- Accidentally exposed vulnerabilities
- Login pages, admin panels, and directories

> 💡 Google Dorking is **100% passive** — you are only using Google's search engine. You never directly interact with the target's server.

> ⚠️ **Legal Note:** Only use Google Dorking on targets you have **explicit permission** to test. Unauthorized use may be illegal.

---

## 2. Google Dork Operators

---

### `site:`

**Purpose:** Restrict search results to a specific domain only.

**Syntax:**
```
site:target.com
```

**Examples:**
```
site:microsoft.com
site:microsoft.com "confidential"
site:microsoft.com "login"
```

**What it does:**
- Returns only pages hosted on `microsoft.com`
- Eliminates noise from other domains
- Reveals all indexed pages of the target

---

### `filetype:`

**Purpose:** Search for specific file types within a domain or across the web.

**Syntax:**
```
filetype:EXTENSION
site:target.com filetype:EXTENSION
```

**Common extensions to look for:**

| Extension | What you might find |
|-----------|-------------------|
| `pdf` | Internal documents, manuals, reports |
| `xls` / `xlsx` | Spreadsheets, possibly with credentials or user data |
| `txt` | Config notes, password files, readme files |
| `doc` / `docx` | Internal Word documents |
| `xml` | Config files, API responses |
| `sql` | Database dumps (very sensitive) |
| `log` | Server/application logs |
| `bak` | Backup files (may contain source code or configs) |

**Examples:**
```
site:microsoft.com filetype:pdf
site:microsoft.com filetype:xlsx passwords
site:microsoft.com filetype:txt
```

---

### `intitle:`

**Purpose:** Search for pages where the **page title** contains a specific word.

**Syntax:**
```
intitle:"keyword"
```

**Examples:**
```
intitle:"admin panel"
intitle:"login page"
intitle:"confidential"
site:microsoft.com intitle:"confidential"
```

**What it does:**
- The `<title>` tag of a web page is what appears in the browser tab
- If a page's title has "admin" or "login", it's likely an admin/login page
- Very useful for finding admin panels and internal pages

---

### `intext:`

**Purpose:** Search for pages where the **body content** contains a specific word or phrase.

**Syntax:**
```
intext:"keyword"
```

**Examples:**
```
intext:"call of duty"
site:microsoft.com intext:"internal use only"
site:microsoft.com intext:"password"
```

**What it does:**
- Searches the actual visible text content of indexed pages
- Useful when the keyword might not be in the title or URL

---

### `inurl:`

**Purpose:** Search for pages where the **URL itself** contains a specific word.

**Syntax:**
```
inurl:"keyword"
```

**Examples:**
```
inurl:"admin"
inurl:"login"
inurl:"config"
site:microsoft.com inurl:"admin"
```

**What it does:**
- Finds pages whose URL path contains your keyword
- Common use: finding admin panels (`/admin`), login pages (`/login`), config endpoints

---

### `index of`

**Purpose:** Find web servers with **directory listing enabled** — meaning you can browse folders like a file explorer.

**Syntax:**
```
intitle:"index of" "keyword"
```

**Examples:**
```
intitle:"index of" "movies"
intitle:"index of" "passwords"
intitle:"index of" ".env"
intitle:"index of" "backup"
```

**What it does:**
- Apache/Nginx servers sometimes accidentally leave directory listing enabled
- This exposes all files in a folder publicly
- Can reveal backups, configs, scripts, logs — anything stored in that directory

> 🔴 **This is a serious misconfiguration.** Finding `index of` on a target means they have an exposed directory that should be private.

---

## 3. Combining Operators (Power Queries)

The real power of Google Dorking comes from **combining multiple operators**:

| Goal | Dork Query |
|------|-----------|
| Find all PDFs on a domain | `site:target.com filetype:pdf` |
| Find Excel files mentioning passwords | `site:target.com filetype:xlsx passwords` |
| Find PDFs mentioning passwords | `site:target.com filetype:pdf passwords` |
| Find admin login pages | `site:target.com inurl:admin intitle:login` |
| Find exposed config files | `site:target.com filetype:xml config` |
| Find exposed .env files | `intitle:"index of" ".env"` |
| Find SQL dumps | `site:target.com filetype:sql` |
| Find log files | `site:target.com filetype:log` |
| Find backup files | `site:target.com filetype:bak` |
| Find exposed directories | `site:target.com intitle:"index of"` |
| Find pages with "confidential" in title | `site:target.com intitle:"confidential"` |
| Find internal-only pages | `site:target.com intext:"internal use only"` |

---

## 4. Real-World Attack Scenarios

### Scenario 1: Finding Sensitive Documents
```
site:targetcompany.com filetype:pdf
```
→ Returns all PDFs indexed by Google from the target's domain  
→ May reveal internal reports, employee handbooks, technical specs, vendor contracts

---

### Scenario 2: Finding Exposed Credentials
```
site:targetcompany.com filetype:xlsx password
site:targetcompany.com filetype:txt password
```
→ Finds spreadsheets or text files that mention "password"  
→ Sometimes these are accidentally uploaded credential sheets

---

### Scenario 3: Finding Admin Panels
```
site:targetcompany.com inurl:admin
site:targetcompany.com intitle:"admin login"
```
→ Reveals admin panels that may have weak authentication

---

### Scenario 4: Directory Listing Exposed
```
site:targetcompany.com intitle:"index of"
```
→ Finds folders where directory listing is enabled  
→ Could expose backups, source code, config files, private documents

---

### Scenario 5: Photo OSINT → Software Version → CVE
> Example from the video:  
> You find an Instagram photo of a company employee where their laptop screen shows **Adobe Acrobat X** running.  
> You search for known CVEs for that version on `exploit-db.com`.  
> You now have a potential exploit — found entirely through passive recon.

---

## 5. Google Hacking Database (GHDB)

The **Google Hacking Database (GHDB)** is a public repository of ready-made Google Dork queries, maintained by **Exploit-DB**.

- 🔗 **URL:** [https://www.exploit-db.com/google-hacking-database](https://www.exploit-db.com/google-hacking-database)
- Contains **7000+ dork entries** categorized by type
- Categories include: Files containing passwords, Sensitive directories, Error messages, Login portals, Vulnerable servers, and more

**How to use:**
1. Go to the GHDB link above
2. Search for your category (e.g., "passwords", "admin", "config")
3. Copy the dork query
4. Run it in Google with your target's domain added via `site:`

---

## 6. Quick Reference Cheat Sheet

| Operator | Syntax | Use Case |
|----------|--------|----------|
| `site:` | `site:target.com` | Restrict results to one domain |
| `filetype:` | `filetype:pdf` | Find specific file types |
| `intitle:` | `intitle:"admin"` | Search in page titles |
| `intext:` | `intext:"password"` | Search in page body text |
| `inurl:` | `inurl:"login"` | Search in page URLs |
| `"exact phrase"` | `"internal use only"` | Search exact string |
| `index of` | `intitle:"index of"` | Find exposed directories |

### Combining Template
```
site:TARGET.COM filetype:EXTENSION "KEYWORD"
site:TARGET.COM inurl:ENDPOINT intitle:TITLE
```

### Common High-Value Dorks
```
site:target.com filetype:pdf
site:target.com filetype:xlsx password
site:target.com filetype:sql
site:target.com filetype:log
site:target.com filetype:bak
site:target.com inurl:admin
site:target.com intitle:"index of"
site:target.com intext:"internal use only"
site:target.com filetype:xml config
intitle:"index of" ".env"
```

---

> ✅ **End of Day 05 Notes**  
> 📌 Next up: More **Passive Reconnaissance** techniques  
> 💡 Practice these dorks on your own domain or on bug bounty programs that allow it!  
> 💬 Questions? Drop them in the comments!