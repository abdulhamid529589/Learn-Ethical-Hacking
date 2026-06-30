# Day 12 — Scanning: Directory Brute Forcing

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 11](#) | Next: [Day 13](#)

---

## 📋 Table of Contents

1. [What are Server Directories?](#1-what-are-server-directories)
2. [What is Directory Brute Forcing?](#2-what-is-directory-brute-forcing)
3. [Why It Matters](#3-why-it-matters)
4. [HTTP Status Codes You'll See](#4-http-status-codes-youll-see)
5. [Tool 1: GoBuster](#5-tool-1-gobuster)
6. [Tool 2: Feroxbuster](#6-tool-2-feroxbuster)
7. [Wordlists](#7-wordlists)
8. [Tools Comparison](#8-tools-comparison)
9. [What to Do After Finding Directories](#9-what-to-do-after-finding-directories)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. What are Server Directories?

Directories on a web server are **folders** that store and organize files — just like folders on your computer.

**Analogy:**
```
Port       = Door to a room (e.g., port 80 = HTTP room)
Directory  = Wardrobe/cabinet inside that room
Files      = Items stored inside the wardrobe
```

**Examples of common web server directories:**

| Directory | What it typically contains |
|-----------|---------------------------|
| `/css/` | Stylesheet files |
| `/js/` | JavaScript files |
| `/images/` | Image assets |
| `/admin/` | Admin panel (high value target!) |
| `/internal/` | Internal resources (potentially sensitive) |
| `/backup/` | Backup files (may contain source code) |
| `/uploads/` | File upload functionality |
| `/api/` | API endpoints |
| `/.git/` | Git repository (can leak source code) |

---

## 2. What is Directory Brute Forcing?

Directory brute forcing is the process of **systematically guessing directory and file names** on a web server to discover hidden or unlisted resources.

**Why "brute force"?**
- The server doesn't give you a map of its directories
- You try every possible name from a **wordlist** (a list of common directory names)
- For each name, you send a request and check the server's response
- If the server responds with `200 OK` → directory exists!

**Analogy:**
```
No light in a dark room + a wardrobe with unknown contents
→ You reach in and feel for objects one by one
→ "Does this feel like a book? No. Does this feel like a file? Yes!"
→ That's directory brute forcing — try each name, see what exists
```

**What you're sending:**
```
GET http://target.com/admin     → 200? Found it!
GET http://target.com/login     → 404? Doesn't exist.
GET http://target.com/backup    → 301? Redirects somewhere!
GET http://target.com/internal  → 200? Potentially sensitive!
```

---

## 3. Why It Matters

Directory brute forcing during the **scanning phase** can reveal:

| Discovery | Significance |
|-----------|-------------|
| **Login panels** | Brute force credentials, bypass auth |
| **Admin panels** | Direct admin access if credentials are weak |
| **File upload pages** | Upload malicious files (webshells) |
| **Backup files** | Source code, config files with credentials |
| **Internal resources** | Pages not meant to be public-facing |
| **API endpoints** | Undocumented APIs, potential data exposure |
| **Config files** | `.env`, `config.php`, `web.config` with secrets |

> 💡 In the demo: The `/internal/` directory revealed a **file upload functionality** — a potential entry point for uploading malicious files to the server.

---

## 4. HTTP Status Codes You'll See

| Code | Meaning | Action |
|------|---------|--------|
| **200 OK** | Directory/file exists and is accessible | ✅ Investigate! |
| **301 Moved** | Permanent redirect to another URL | ✅ Follow the redirect |
| **302 Found** | Temporary redirect | ✅ Follow the redirect |
| **403 Forbidden** | Exists but you don't have permission | ⚠️ Note it — may bypass later |
| **404 Not Found** | Doesn't exist | ❌ Skip |
| **401 Unauthorized** | Exists but requires authentication | ⚠️ Note it — try credential attacks |
| **500 Server Error** | Server crashed on request | ⚠️ Potential vulnerability |

> ✅ **Focus on 200 and 301 codes** — these confirm real directories/files.  
> ⚠️ **Note 403 codes** — these directories exist but are protected. You may be able to bypass the restriction later.

---

## 5. Tool 1: GoBuster

**GoBuster** is a fast directory brute forcing tool written in Go. Best choice for CTF environments and HTTP targets.

### Installation

```bash
# Pre-installed on Kali Linux

# Install on other systems:
sudo apt install gobuster
# Or via Go:
go install github.com/OJ/gobuster/v3@latest
```

### Basic Directory Brute Force

```bash
gobuster dir -u http://TARGET_IP:PORT -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Full Command Breakdown

```bash
gobuster dir \
  -u http://TARGET_IP:PORT \
  -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt \
  -t 50 \
  -x php,html,txt \
  --status-codes 200,301,302 \
  -o results.txt
```

| Flag | Description |
|------|-------------|
| `dir` | Directory brute forcing mode |
| `-u` | Target URL |
| `-w` | Wordlist path |
| `-t` | Number of threads (default: 10, increase for speed) |
| `-x` | File extensions to append (e.g., `.php`, `.html`, `.txt`) |
| `--status-codes` | Only show these status codes |
| `-o` | Save output to file |
| `-k` | Skip TLS/SSL certificate verification (for HTTPS) |
| `--timeout` | HTTP timeout in seconds |
| `-b` | Blacklist status codes (e.g., `-b 404`) |

### Common Usage Examples

```bash
# Basic scan
gobuster dir -u http://192.168.1.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

# With file extensions
gobuster dir -u http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -x php,html,txt,js

# With port specified
gobuster dir -u http://192.168.1.10:3333 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt

# HTTPS target (skip cert verification)
gobuster dir -u https://target.com -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -k

# Save results to file
gobuster dir -u http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -o gobuster_results.txt

# Faster scan with more threads
gobuster dir -u http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -t 50

# Only show specific status codes
gobuster dir -u http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt --status-codes 200,301
```

### Example Output
```
===============================================================
Gobuster v3.x
===============================================================
[+] Url:      http://192.168.1.10:3333
[+] Wordlist: /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt
===============================================================
/.htaccess            (Status: 403) [Size: 278]
/css                  (Status: 301) [Size: 0]
/images               (Status: 301) [Size: 0]
/internal             (Status: 301) [Size: 0]  ← Interesting!
/js                   (Status: 301) [Size: 0]
===============================================================
```

---

## 6. Tool 2: Feroxbuster

**Feroxbuster** is a recursive directory brute forcing tool. It automatically goes **inside found directories** and scans those too. Best for HTTPS targets.

### Installation

```bash
# Pre-installed on Kali Linux

# Install manually:
sudo apt install feroxbuster
# Or via cargo (Rust):
cargo install feroxbuster
```

### Basic Usage

```bash
feroxbuster --url http://TARGET -w /path/to/wordlist
```

### Full Command with Options

```bash
feroxbuster \
  --url https://target.com \
  --wordlist /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt \
  --threads 50 \
  --depth 3 \
  --extensions php,html,txt \
  --output ferox_results.txt
```

| Flag | Description |
|------|-------------|
| `--url` | Target URL |
| `-w` / `--wordlist` | Wordlist path |
| `-t` / `--threads` | Number of threads |
| `-d` / `--depth` | Recursion depth (default: 4) |
| `-x` / `--extensions` | File extensions to search for |
| `-o` / `--output` | Save output to file |
| `-k` | Ignore TLS certificate errors |
| `--no-recursion` | Disable recursive scanning |
| `--filter-status` | Filter out specific status codes |
| `-C` | Filter status codes (e.g., `-C 404,403`) |

### Common Usage Examples

```bash
# Basic scan
feroxbuster --url http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt

# HTTPS target
feroxbuster --url https://target.com -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -k

# With extensions
feroxbuster --url http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -x php,html,txt

# Non-recursive (faster, like GoBuster)
feroxbuster --url http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt --no-recursion

# Limit recursion depth
feroxbuster --url http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -d 2

# Filter out 404s (cleaner output)
feroxbuster --url http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -C 404

# Save to file
feroxbuster --url http://192.168.1.10 -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt -o ferox_output.txt
```

### Example Output
```
200      GET   12l   45w  target.com/.env           ← Sensitive!
200      GET    5l   20w  target.com/.well-known/security.txt
301      GET    0l    0w  target.com/css
200      GET   35l   150w target.com/css/style.css  ← (recursive find)
301      GET    0l    0w  target.com/js
```

> 🔁 **Recursion:** Feroxbuster found `/css/`, then automatically scanned inside it and found `/css/style.css`. GoBuster does NOT do this by default.

---

## 7. Wordlists

The quality of your scan depends heavily on your **wordlist**.

### Common Wordlists (Kali Linux built-in)

| Wordlist | Path | Size | Use Case |
|----------|------|------|----------|
| **common.txt** | `/usr/share/wordlists/seclist/Discovery/Web-Content/common.txt` | ~4,700 words | Quick first scan — most useful |
| **directory-list-2.3-small.txt** | `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt` | ~87,000 | Deeper scan |
| **directory-list-2.3-medium.txt** | `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` | ~220,000 | Comprehensive scan |
| **big.txt** | `/usr/share/wordlists/seclist/Discovery/Web-Content/big.txt` | ~20,000 | Good balance |

### Install SecLists (if not present)

```bash
sudo apt install seclists
# Wordlists installed to: /usr/share/seclists/
```

### Recommended Wordlist Strategy

```
1st pass: common.txt        → Fast, catches most obvious dirs
2nd pass: big.txt           → More thorough
3rd pass: directory-list-2.3-medium.txt  → Deep scan if needed
```

---

## 8. Tools Comparison

| Feature | GoBuster | Feroxbuster |
|---------|----------|-------------|
| **Language** | Go | Rust |
| **Speed** | ⭐⭐⭐ Very Fast | ⭐⭐⭐ Very Fast |
| **Recursive scanning** | ❌ No (manual) | ✅ Yes (automatic) |
| **HTTPS support** | ✅ Yes (`-k`) | ✅ Yes (`-k`) |
| **Best for** | CTFs, HTTP targets, quick scans | HTTPS targets, deep recursive scans |
| **Pre-installed on Kali** | ✅ Yes | ✅ Yes |
| **Output to file** | ✅ `-o` | ✅ `-o` |
| **Extension bruteforce** | ✅ `-x` | ✅ `-x` |

> 💡 **Instructor's recommendation:**  
> - Use **GoBuster** for CTFs and HTTP targets  
> - Use **Feroxbuster** for HTTPS targets or when you need recursive scanning  
> - When in doubt, try both — each may find different things

---

## 9. What to Do After Finding Directories

```
Found /admin/        → Try to access, check for login panel
Found /internal/     → Check for sensitive files or functionality
Found /uploads/      → File upload vulnerability? Try uploading a webshell
Found /backup/       → Download and inspect — may contain source code/credentials
Found /.git/         → Run: git-dumper to extract the repository
Found /api/          → Enumerate API endpoints further
Found /.env          → Open it — may contain database passwords, API keys
Found /config.php    → Download — may contain DB credentials
Found /login.php     → Brute force credentials or check for SQLi
```

---

## 10. Quick Reference Cheat Sheet

### GoBuster
```bash
# Basic
gobuster dir -u http://TARGET -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt

# With extensions + faster
gobuster dir -u http://TARGET -w WORDLIST -x php,html,txt -t 50

# With port
gobuster dir -u http://TARGET:3333 -w WORDLIST

# HTTPS (skip cert check)
gobuster dir -u https://TARGET -w WORDLIST -k

# Save output
gobuster dir -u http://TARGET -w WORDLIST -o results.txt
```

### Feroxbuster
```bash
# Basic
feroxbuster --url http://TARGET -w /usr/share/wordlists/seclist/Discovery/Web-Content/common.txt

# HTTPS
feroxbuster --url https://TARGET -w WORDLIST -k

# No recursion (faster)
feroxbuster --url http://TARGET -w WORDLIST --no-recursion

# With extensions
feroxbuster --url http://TARGET -w WORDLIST -x php,html,txt

# Filter 404s
feroxbuster --url http://TARGET -w WORDLIST -C 404
```

### Wordlist Quick Pick
```bash
# Fast first scan
/usr/share/wordlists/seclist/Discovery/Web-Content/common.txt

# Deeper scan
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# With file extensions
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
```

---

> ✅ **End of Day 12 Notes**  
> 📌 This is part of the **Scanning Phase** — don't exploit yet, just enumerate everything first.  
> 💡 Practice: Set up a vulnerable machine (TryHackMe / HackTheBox) and run GoBuster against it!  
> 💬 Questions? Drop them in the comments!