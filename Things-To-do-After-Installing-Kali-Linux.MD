# 🐧 Kali Linux — Getting Started with Security Tools
> **Study notes** compiled from video tutorial. For educational and ethical use only — always test on systems you own or have explicit written permission to test.

---

## ⚠️ Legal & Ethical Disclaimer

> Unauthorized access to computer systems is **illegal** in most jurisdictions worldwide. All techniques covered here must **only** be practiced on:
> - Your own machines
> - Dedicated lab environments (e.g., VMs, home labs)
> - Systems you have **explicit written permission** to test

---

## Table of Contents

1. [Network Mapping — Nmap](#1-network-mapping--nmap)
2. [GUI Nmap — Zenmap](#2-gui-nmap--zenmap)
3. [SMB Enumeration & File Share Access](#3-smb-enumeration--file-share-access)
4. [Brute Force Login — Hydra](#4-brute-force-login--hydra)
5. [Phishing / Credential Harvesting — SET](#5-phishing--credential-harvesting--social-engineering-toolkit-set)
6. [Web Application Scanning — OWASP ZAP](#6-web-application-scanning--owasp-zap)
7. [Exploitation Framework — Metasploit (msfconsole)](#7-exploitation-framework--metasploit-msfconsole)
8. [OSINT — SpiderFoot](#8-osint--spiderfoot)
9. [Networking Swiss Army Knife — Netcat](#9-networking-swiss-army-knife--netcat)
10. [Password Cracking — Hashcat](#10-password-cracking--hashcat)
11. [Exploit Research — SearchSploit & SQLMap](#11-exploit-research--searchsploit--sqlmap)

---

## 1. Network Mapping — Nmap

**Purpose:** Discover live hosts, open ports, and running services on a network.

### Get Your IP Address First
```bash
ip addr
# Example output: 192.168.0.105/24
```

### Basic Network Scan
Scans all hosts in the subnet range:
```bash
nmap 192.168.0.0/24
```

### Fast Host Discovery (No Port Scan)
Uses ARP ping — just checks which devices are alive, very quick:
```bash
nmap -sn 192.168.0.0/24
```

### Service Version Detection
Identifies what services and versions are running on open ports:
```bash
nmap -sV 192.168.0.229
```

### SMB Share Enumeration Script
Targets port 445 (Windows File Sharing) to list accessible shared folders:
```bash
nmap --script smb-enum-shares -p 445 192.168.0.229
```

### Key Nmap Flags Reference

| Flag | Description |
|------|-------------|
| `-sn` | Ping scan only (no port scan) |
| `-sV` | Detect service versions |
| `-sS` | SYN (stealth) scan |
| `-p <port>` | Scan specific port(s) |
| `--script <name>` | Run an NSE script |
| `-A` | Aggressive scan (OS, version, scripts) |
| `-T4` | Speed up scan (timing template) |

---

## 2. GUI Nmap — Zenmap

**Purpose:** Graphical front-end for Nmap — easier for beginners.

### Launch Zenmap
```bash
zenmap
```

### How to Use
1. Enter the **Target** IP range (e.g., `192.168.0.0/24`)
2. Select a **Profile** from the dropdown:
   - `Quick Scan` — fast, basic
   - `Quick Scan Plus` — adds version detection
   - `Intense Scan` — full scan with OS detection
   - `Comprehensive` — everything
3. Click **Scan**
4. Navigate the **Topology** tab to see a visual network map
5. Click any host on the left panel to see its open ports and services

---

## 3. SMB Enumeration & File Share Access

**Purpose:** Connect to shared Windows network folders (SMB/Samba shares).

### Connect to an Open (No Password) Share
```bash
smbclient //192.168.0.229/no_password_folder
# Press Enter when prompted for password
```

### List Files in the Share
```bash
ls
```

### Connect to a Protected Share (with credentials)
```bash
smbclient //192.168.0.229/share_name -U username
# Then enter password when prompted
```

> **Note:** Public Wi-Fi environments (hotels, airports, cafés) often have devices with misconfigured open shares — a common real-world finding in security audits.

---

## 4. Brute Force Login — Hydra

**Purpose:** Automated login attempts using wordlists to guess credentials.

### Basic SMB Brute Force
```bash
hydra -L usernames.txt -P passwords.txt smb://192.168.0.229
```

### Flag Reference

| Flag | Description |
|------|-------------|
| `-l <user>` | Single username |
| `-L <file>` | Username wordlist file |
| `-p <pass>` | Single password |
| `-P <file>` | Password wordlist file |
| `-t <n>` | Number of parallel threads |
| `-V` | Verbose output |

### Common Protocols Supported
`ssh`, `ftp`, `smb`, `http-get`, `http-post-form`, `rdp`, `telnet`, `mysql`

### Example Output
```
[445][smb] host: 192.168.0.229   login: admin   password: 12345678
1 of 1 target successfully completed, 2 valid passwords found
```

> **Tip:** Common wordlists are bundled with Kali at `/usr/share/wordlists/` — e.g., `rockyou.txt`.

---

## 5. Phishing / Credential Harvesting — Social Engineering Toolkit (SET)

**Purpose:** Create fake login pages to simulate phishing attacks (for awareness training).

### Launch SET
```bash
sudo setoolkit
```

### Navigation Menu (Step-by-Step)
```
1) Social-Engineering Attacks
   └── 2) Website Attack Vectors
          └── 3) Credential Harvester Attack Method
                 ├── 1) Web Templates  ← ready-made pages (Google, Twitter, etc.)
                 ├── 2) Site Cloner   ← clone any existing site
                 └── 3) Custom Import ← use your own template
```

### Web Templates Workflow
1. Select **Web Templates**
2. Enter your **local IP address** for the POST-back listener
3. Choose a template (e.g., `Google`)
4. SET starts a local web server
5. Browse to `http://192.168.0.105` — the fake login page is live
6. Any submitted credentials are captured and printed in the terminal

---

## 6. Web Application Scanning — OWASP ZAP

**Purpose:** Automated web vulnerability scanner — finds XSS, CSRF, misconfigurations, and more.

### Launch ZAP
```bash
# Search in Kali menu → ZAP Proxy, or:
zap
```

### Automated Scan Workflow
1. Open ZAP
2. Click **Automated Scan**
3. Paste the target URL
4. Click **Attack**
5. Monitor the **Alerts** tab for discovered vulnerabilities

### Common Vulnerabilities ZAP Detects

| Vulnerability | Description |
|---------------|-------------|
| XSS (Reflected) | Cross-Site Scripting via URL parameters |
| CSRF Token Absence | Missing anti-CSRF protection |
| Content Security Policy | Missing/weak CSP header |
| Private IP Disclosure | Internal IPs leaked in responses |
| SQL Injection | Database query injection points |

### Using Results
- Each alert shows: **Risk level**, **URL**, **Parameter**, **Evidence**, **Payload**
- Copy the payload from the Evidence tab to manually reproduce/verify the finding
- Use results to write a penetration testing report

---

## 7. Exploitation Framework — Metasploit (msfconsole)

**Purpose:** Search and use pre-built exploit modules against identified services.

### Launch Metasploit
```bash
msfconsole
```

### Basic Workflow

#### Search for a module
```bash
search ftp
search smb
search wordpress
```

#### Use a module
```bash
use auxiliary/scanner/ftp/anonymous
```

#### View required options
```bash
show options
```

#### Set target host
```bash
set RHOSTS 192.168.0.229
```

#### Run the module
```bash
run
```

### Example — FTP Anonymous Access Check
```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS 192.168.0.229
run
# Output: Anonymous READ access: YES
```

### Connect Manually via FTP After Discovery
```bash
ftp 192.168.0.229
# Name: anonymous
# Password: (just press Enter)
ls
```

---

## 8. OSINT — SpiderFoot

**Purpose:** Open-Source Intelligence gathering — collects publicly available information about a target domain, email, IP, or username.

### Launch SpiderFoot (Web UI)
```bash
spiderfoot -l 127.0.0.1:5001
```
Then open browser and navigate to: `http://127.0.0.1:5001`

### Running a Scan
1. Click **New Scan**
2. Enter a **Scan Name** (e.g., "Target Recon")
3. Enter a **Scan Target** (domain, IP, email, username)
4. Click **Run Scan Now**

### What SpiderFoot Collects

| Data Type | Examples |
|-----------|---------|
| Domain info | Subdomains, DNS records, WHOIS |
| Email addresses | Found via public sources |
| HTTP status codes | Live vs. dead pages |
| Internet names | Linked hostnames |
| Correlations | Relationships between data points |

### Tabs to Explore
- **Summary** — overview of all data types found
- **Browse** — drill into specific data categories
- **Correlations** — visualize relationships

---

## 9. Networking Swiss Army Knife — Netcat

**Purpose:** Multi-use tool for banner grabbing, port listening, reverse shells, and file transfer.

### Banner Grabbing
Grabs the service banner from an open port to identify what's running:
```bash
nc 192.168.0.229 21    # Check FTP banner
nc 192.168.0.229 22    # Check SSH banner
nc 192.168.0.229 80    # Check HTTP banner
```

### Setting Up a Listener (for Reverse Shells)
Run this on your machine to wait for incoming connections:
```bash
nc -lvp 4444
```

| Flag | Description |
|------|-------------|
| `-l` | Listen mode |
| `-v` | Verbose output |
| `-p` | Specify port |
| `-n` | Skip DNS resolution |
| `-e` | Execute program on connect |

### Once a Connection is Received
```bash
# In the netcat session, run commands on the remote machine:
whoami
ls
cat /etc/passwd
```

> **Context:** Netcat listeners are commonly used after exploiting a vulnerability to gain a remote shell on the target system.

---

## 10. Password Cracking — Hashcat

**Purpose:** Crack hashed passwords using wordlists or brute force.

### Understanding Password Hashes
When databases store passwords, they're hashed (e.g., MD5, SHA1, bcrypt). Cracking recovers the original plaintext.

Example hash in a database dump:
```
username:5f4dcc3b5aa765d61d8327deb882cf99:user@email.com
          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
          This is the MD5 hash of "password"
```

### Extract Hashes to a File
```bash
cat database_dump.txt | cut -d: -f2 > extracted_hashes.txt
```

### Run Hashcat Against a Wordlist
```bash
hashcat extracted_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Specify Hash Type (Common Modes)

| Mode | Hash Type |
|------|-----------|
| `0` | MD5 |
| `100` | SHA1 |
| `1400` | SHA256 |
| `1800` | SHA512 (Unix) |
| `3200` | bcrypt |

```bash
hashcat -m 0 extracted_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Example Output
```
5f4dcc3b5aa765d61d8327deb882cf99:password
ab56b4d92b40713acc5af89985d4b786:master
```

---

## 11. Exploit Research — SearchSploit & SQLMap

**Purpose:** Find documented exploits for known vulnerabilities and automate SQL injection testing.

### SearchSploit — Finding Exploits

Search the local Exploit-DB copy for known vulnerabilities:
```bash
searchsploit wordpress
searchsploit apache 2.4
searchsploit ftp
```

### View an Exploit
```bash
# Download/view exploit file by its ID
searchsploit -m php/webapps/5486.txt
cat 5486.txt
```

The file typically contains:
- **Vulnerable code** — what the weakness looks like
- **Affected versions** — which versions are impacted
- **Proof-of-Concept payload** — how to trigger the vulnerability

### SQLMap — Automated SQL Injection

Automates finding and exploiting SQL injection vulnerabilities:

```bash
sqlmap -u "http://target.com/page.php?id=1"
```

### Interactive Prompts During Scan
SQLMap will ask questions during the scan:
- `Do you want to skip test payloads specific for other DBMSes?` → `Y`
- `Do you want to include all tests?` → `Y`
- `Do you want to keep testing the others?` → `N` (once found)

### Once Injection is Confirmed
```bash
# Dump the database
sqlmap -u "http://target.com/page.php?id=1" --dump

# List databases
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Get current user
sqlmap -u "http://target.com/page.php?id=1" --current-user
```

---

## 🗂️ Tool Quick Reference Summary

| # | Tool | Purpose | Command to Start |
|---|------|---------|-----------------|
| 1 | **Nmap** | Network/port scanning | `nmap` |
| 2 | **Zenmap** | GUI for Nmap | `zenmap` |
| 3 | **smbclient** | Access SMB/file shares | `smbclient` |
| 4 | **Hydra** | Brute force logins | `hydra` |
| 5 | **SET** | Phishing / credential harvesting | `sudo setoolkit` |
| 6 | **OWASP ZAP** | Web app vulnerability scanner | `zap` |
| 7 | **Metasploit** | Exploit framework | `msfconsole` |
| 8 | **SpiderFoot** | OSINT / recon | `spiderfoot -l 127.0.0.1:5001` |
| 9 | **Netcat** | Listeners, banners, shells | `nc` |
| 10 | **Hashcat** | Password hash cracking | `hashcat` |
| 11 | **SearchSploit** | Exploit database search | `searchsploit` |
| 11 | **SQLMap** | SQL injection automation | `sqlmap` |

---

## 📚 Recommended Next Steps

- Set up a **home lab** using VirtualBox or VMware with intentionally vulnerable VMs:
  - [Metasploitable 2/3](https://sourceforge.net/projects/metasploitable/)
  - [DVWA (Damn Vulnerable Web Application)](https://github.com/digininja/DVWA)
  - [VulnHub](https://www.vulnhub.com/) machines
- Practice on legal platforms:
  - [Hack The Box](https://www.hackthebox.com/)
  - [TryHackMe](https://tryhackme.com/)
  - [PentesterLab](https://pentesterlab.com/)
- Study for certifications:
  - **CompTIA Security+** — foundational
  - **eJPT** (eLearnSecurity Junior Penetration Tester) — practical beginner cert
  - **OSCP** (Offensive Security Certified Professional) — industry standard

---

*Notes compiled for educational purposes. Always practice ethically and legally.*