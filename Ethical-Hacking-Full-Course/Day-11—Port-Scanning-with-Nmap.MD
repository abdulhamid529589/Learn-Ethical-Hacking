# Day 11 — Port Scanning with Nmap
### Learn Ethical Hacking in 30 Days

---

## 🎯 Objective

Understand **ports** and **protocols** in computer networking from the ground up, then master **active port scanning** using `nmap` — the most powerful and widely-used network scanning tool in ethical hacking.

> 📌 **Phase Transition:** From this video onward, we move from **Passive Reconnaissance** (gathering info without touching the target) → **Active Scanning** (directly interacting with the target to discover open ports, services, and vulnerabilities).

---

## 🧱 Phase Context: Where We Are in the Hacking Methodology

```
Phase 1 — Reconnaissance (Passive)
    ├── Day 8:  Tech Stack Fingerprinting   (Wappalyzer, BuiltWith, WhatWeb)
    ├── Day 9:  Subdomain Enumeration       (Subfinder, c99.nl)
    └── Day 10: WAF Detection               (wafw00f)

Phase 2 — Scanning (Active)  ← YOU ARE HERE
    └── Day 11: Port Scanning               (Nmap)
```

---

## 🧠 Concept 1 — What are Ports?

### Official Definition
> Ports are **doorways and entry points** that allow different types of data to flow in and out of a device or network. Each port is associated with a **specific service or application**, so different types of data are directed to the correct program.

### Simple Analogy — The House 🏠

> Imagine your computer is a **house**.
>
> The house has **one main front door** (your IP address — the address of the house).
> But inside the house, there are **many separate rooms**, each with its own door:
>
> | Room | Door (Port) | Service |
> |------|------------|---------|
> | Kitchen | Port 20/21 | FTP — File Transfer |
> | Main Hall | Port 80 | HTTP — Web browsing |
> | Secure Room | Port 22 | SSH — Remote login |
> | Post Box | Port 25 | SMTP — Email sending |
> | Vault | Port 443 | HTTPS — Secure web |
>
> The kitchen door leads only to the kitchen — you can't reach the bedroom through it.
> Similarly, **Port 80 leads only to the HTTP web service** — you can't reach SSH through it.

### Key Points About Ports
- There are **65,535 total ports** on any device (0–65535)
- Ports **0–1023** are called **Well-Known Ports** — reserved for standard services
- Ports **1024–49151** are **Registered Ports** — used by software applications
- Ports **49152–65535** are **Dynamic/Private Ports** — used temporarily by client apps
- A port can be in one of three states:
  - **Open** — A service is actively listening on this port
  - **Closed** — No service is running, but the port is reachable
  - **Filtered** — A firewall is blocking access to the port (Nmap can't determine its state)

---

## 🔄 Concept 2 — What are Protocols?

### Official Definition
> Protocols are a **set of rules and standards** that define how data is transmitted and received between devices. They ensure that computers, smartphones, and servers can communicate with each other in a way that **everyone understands — regardless of manufacturer or device type**.

### Simple Analogy — Traffic Rules 🚦

> Think about traffic rules on a road:
> - **Red light = Stop** → applies to every vehicle, every driver, everywhere
> - **Green light = Go** → same universal rule
> - **Two-wheeler → must wear helmet**
> - **Four-wheeler → must wear seatbelt**
>
> Nobody needs to "agree" on these rules mid-journey. They're pre-established. Every road user follows them automatically.
>
> Similarly, **network protocols are pre-established rules** that every device on a network follows automatically — whether it's a Windows PC, iPhone, Android, Linux server, or a router.

---

## ⚡ Concept 3 — TCP vs UDP (The Two Core Protocols)

### TCP — Transmission Control Protocol

**What it is:** A **connection-oriented** protocol. Before sending data, TCP establishes a full connection between sender and receiver (called the **3-Way Handshake**: SYN → SYN-ACK → ACK).

**Key characteristics:**
- Data delivery is **guaranteed** — if a packet is lost, TCP resends it
- Data arrives **in order** — packets are reassembled in the correct sequence
- **Slower** than UDP due to the overhead of error-checking and acknowledgments
- Best for situations where **accuracy matters more than speed**

**Real-world examples of TCP use:**
- Loading a webpage (HTTP/HTTPS) — every byte of HTML must arrive correctly
- Logging into a server via SSH — commands must be received exactly
- Sending an email (SMTP) — the full message must arrive intact
- File download/upload (FTP) — files must not be corrupted

---

### UDP — User Datagram Protocol

**What it is:** A **connectionless** protocol. UDP fires data packets at the destination without first establishing a connection, and without checking if they arrived.

**Key characteristics:**
- **No guarantee** of delivery — packets can be dropped and are not resent
- **No ordering** — packets may arrive out of sequence
- **Much faster** than TCP — no handshake overhead
- Best for situations where **speed matters more than perfect accuracy**

**Real-world examples of UDP use:**
- **Video streaming** (YouTube, Netflix) — a dropped frame is better than buffering
- **Online gaming** — real-time position data must be sent instantly; a missed packet is acceptable
- **DNS lookups** (Port 53) — quick single request/response, no need for full connection
- **VoIP calls** — a tiny glitch in audio is better than a lag

---

### TCP vs UDP — Side-by-Side Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless (no handshake) |
| **Reliability** | ✅ Guaranteed delivery | ❌ No guarantee |
| **Order** | ✅ Data arrives in order | ❌ May arrive out of order |
| **Speed** | Slower | Faster |
| **Error Checking** | Yes | Minimal |
| **Use Case** | When accuracy is critical | When speed is critical |
| **Examples** | HTTP, HTTPS, SSH, FTP, SMTP | DNS, Video streaming, Gaming, VoIP |

---

## 📋 Concept 4 — Common Ports & Protocols (Must Memorize)

These are the **7 most important ports** mentioned in the video. You will see these constantly in Nmap scan results during pentesting.

| Port | Protocol | Service Name | Full Form | Detailed Use Case |
|------|----------|-------------|-----------|-------------------|
| **20** | TCP | FTP Data | File Transfer Protocol | Carries actual **file data** during an active FTP session. Once the connection is established via port 21, all file content flows through port 20. |
| **21** | TCP | FTP Control | File Transfer Protocol | Used for **commands and control** in FTP. You first connect to port 21 to authenticate and issue commands (e.g., list directory, initiate transfer). The actual data then flows via port 20. |
| **22** | TCP | SSH | Secure Shell | **Encrypted remote login** and command execution. All data is encrypted — no man-in-the-middle attack is possible. Used by sysadmins to remotely manage servers. |
| **25** | TCP | SMTP | Simple Mail Transfer Protocol | **Sending emails** between mail servers. When you hit "Send" on Gmail, SMTP on port 25 handles the transmission of your email to the recipient's mail server. |
| **53** | UDP | DNS | Domain Name System | **Resolves domain names to IP addresses**. When you type `google.com`, a DNS query goes out on port 53 to convert that name to `142.250.x.x`. Uses UDP because it's a quick one-shot query. |
| **80** | TCP | HTTP | HyperText Transfer Protocol | **Unencrypted web browsing**. Every website URL starting with `http://` uses port 80. Data is sent in plain text — visible to anyone intercepting the traffic. |
| **443** | TCP | HTTPS | HTTP Secure | **Encrypted web browsing**. Every website URL starting with `https://` uses port 443. Data is encrypted using SSL/TLS — the padlock icon in your browser confirms this. |

### FTP Two-Port Mechanism Explained

```
Client                          FTP Server
  │                                 │
  │──── Port 21: "I want to connect" ──→│   ← CONTROL channel
  │←─── Port 21: "Connected. Authenticated." ─│
  │                                 │
  │──── Port 21: "Send me file.txt" ──→│   ← COMMAND via control
  │                                 │
  │←─── Port 20: [file data flowing] ──│   ← DATA via port 20
```

> Port 21 = the intercom/phone line (you talk through it)
> Port 20 = the delivery belt (actual goods travel through it)

---

## 🛠️ Tool — Nmap (Network Mapper)

### About Nmap
- Full name: **Network Mapper**
- Created by **Gordon Lyon (Fyodor)**
- **Pre-installed on Kali Linux** — no setup needed
- The **most widely used port scanner** in ethical hacking and penetration testing
- Can perform: host discovery, port scanning, service/version detection, OS detection, script-based vulnerability scanning, and traceroute

### Launch Help Menu
```bash
nmap
```
Running just `nmap` shows the full help page with all available options grouped by category.

---

## 🗂️ Nmap Help Menu — Categories Explained

When you run `nmap`, you'll see these option groups:

### 1. Host Discovery
Finds which hosts (IP addresses) are alive on a network.
```
-sL    List Scan          → Just list targets, don't scan
-sn    Ping Scan          → Check which hosts are online without port scanning
-Pn    Skip host discovery → Assume host is up, go straight to port scanning
```
> `-Pn` is very useful when the target **blocks ping requests** — it skips the alive-check and scans anyway.

### 2. Scan Techniques (Port Scanning Methods)
```
-sS    TCP SYN Scan       → Half-open, stealthy
-sT    TCP Connect Scan   → Full connection (noisier, no root needed)
-sU    UDP Scan           → Scans UDP ports
-sW    TCP Window Scan    → Variant of SYN scan
-sM    Maimon Scan        → Unusual scan technique
```

### 3. Port Specification & Scan Order
```
-p 22,80,443     → Scan only specific ports
-p 1-1000        → Scan a port range
-F               → Fast mode: top 100 common ports only
--top-ports 1000 → Scan top 1000 most common ports
```

### 4. Service/Version Detection
```
-sV    → Detect service versions running on open ports
```

### 5. Script Scanning (NSE — Nmap Scripting Engine)
```
-sC    → Run default scripts
--script=<name>   → Run a specific script
--script=vuln     → Run all vulnerability detection scripts
```

### 6. OS Detection
```
-O     → Detect the operating system of the target
```

### 7. Timing & Performance
```
-T0    → Paranoid (slowest, most stealthy)
-T1    → Sneaky
-T2    → Polite
-T3    → Normal (default)
-T4    → Aggressive (faster)
-T5    → Insane (fastest, may miss results)
```

### 8. Firewall / IDS Evasion
```
-f     → Fragment packets (split into small pieces to evade detection)
-D     → Decoy scan (mask your IP with fake IPs)
--data-length    → Append random data to packets
```
> ⚠️ These evasion techniques work less effectively against modern production-grade firewalls but can be useful on internal networks.

### 9. Output Formats
```
-oN <file>    → Normal output saved to file
-oX <file>    → XML output
-oG <file>    → Grepable output
-oA <file>    → Save in all formats simultaneously
```

### 10. Miscellaneous
```
-6     → Enable IPv6 scanning
-A     → Aggressive scan (combines multiple flags) ← MOST IMPORTANT
-v     → Verbose (more output detail)
-h     → Help
```

---

## 🔧 Nmap Flags — Complete Quick Reference

| Flag | Full Name | What It Does | Notes |
|------|-----------|-------------|-------|
| `-sS` | TCP SYN Scan | Sends SYN packets, doesn't complete handshake | Stealthy — fewer logs created |
| `-sT` | TCP Connect Scan | Full TCP connection | Noisier, doesn't need root |
| `-sU` | UDP Scan | Scans UDP ports | Slower than TCP scans |
| `-sV` | Version Detection | Identifies service versions on open ports | Critical for exploit research |
| `-sC` | Default Script Scan | Runs built-in NSE scripts | Can detect misconfigs & vulns |
| `-O` | OS Detection | Guesses target OS | Not always 100% accurate |
| `-Pn` | No Ping | Skips host discovery | Use when target blocks pings |
| `-p` | Port Specification | Scan specific ports | `-p 80` or `-p 1-1000` |
| `-F` | Fast Scan | Top 100 ports only | Quick overview scan |
| `-T4` | Aggressive Timing | Faster scan speed | Good balance of speed/accuracy |
| `-A` | **Aggressive Mode** | `-sS` + `-sV` + `-sC` + `-O` + traceroute | **Best single flag to use** ⭐ |
| `-oN` | Normal Output | Save results to text file | Good for reporting |
| `-v` | Verbose | Show more scan detail | Helpful while scan is running |

---

## 🖥️ Practical Commands — Step by Step

### Command 1: TCP SYN Scan (Stealthy)
```bash
sudo nmap -sS <target>
```
**What happens internally:**
```
Nmap → sends SYN packet → Target
Target → sends SYN-ACK back → Nmap
Nmap → sends RST (reset) → Target  ← does NOT complete the handshake
```
Because the handshake is never completed, the connection is never fully logged by many systems. This is called a **"half-open scan"** — stealthy and fast.

> ⚠️ Requires `sudo` (root privileges) because it constructs raw packets.

---

### Command 2: Version Detection Scan
```bash
sudo nmap -sV <target>
```
**What it adds:** After finding open ports, Nmap sends additional probes to determine **exactly which software and version** is running on each port.

Example result:
```
22/tcp  open  ssh  OpenSSH 6.6.1p1 Ubuntu
80/tcp  open  http Apache httpd 2.4.18
```

---

### Command 3: Default Script Scan
```bash
sudo nmap -sC <target>
```
**What it does:** Runs Nmap's built-in **NSE (Nmap Scripting Engine)** default scripts. These scripts check for:
- SSL/TLS certificate details
- HTTP headers and titles
- SSH host keys
- Common misconfigurations
- Some known vulnerabilities

---

### Command 4: OS Detection
```bash
sudo nmap -O <target>
```
**What it does:** Analyzes how the target responds to various TCP/IP probes to guess its operating system.

Example result:
```
Running (JUST GUESSING): Oracle VirtualBox (92%), QEMU (87%)
Aggressive OS guesses: Linux 3.x (92%)
No exact OS matches found.
```
> OS detection is probabilistic — treat it as a best guess, not a confirmed fact.

---

### Command 5: Full Combined Manual Scan
```bash
sudo nmap -sS -sV -sC -O <target>
```
> ⚠️ **Warning:** Running all these together dramatically increases scan time. The instructor's scan of `scanme.nmap.org` took **1.5–2 hours** with all flags combined.

---

### Command 6: Aggressive Scan — THE RECOMMENDED COMMAND ⭐
```bash
sudo nmap -A <target>
```

**The `-A` flag is a shortcut that automatically enables:**

| What `-A` Includes | Flag Equivalent |
|-------------------|----------------|
| SYN Port Scan | `-sS` |
| Service Version Detection | `-sV` |
| Default Script Scan | `-sC` |
| OS Detection | `-O` |
| Traceroute | `--traceroute` |

**Examples:**
```bash
# Scan the practice target (legal)
sudo nmap -A scanme.nmap.org

# Scan microsoft.com
sudo nmap -A microsoft.com

# Scan with output saved to file
sudo nmap -A <target> -oN results.txt
```

> 💡 The instructor says: *"Most of the time I use `-A`. It gives you everything in one command without having to remember and combine multiple flags."*

---

## 📊 Reading Nmap Output — Full Breakdown

### Scan of `scanme.nmap.org` (Practice Target)

```
PORT      STATE  SERVICE   VERSION
22/tcp    open   ssh       OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13
9929/tcp  open   nping-echo
31337/tcp open   tcpwrapped
```

### Column-by-Column Explanation

| Column | Value | What It Means |
|--------|-------|--------------|
| **PORT** | `22/tcp` | Port number 22 running on TCP protocol |
| **STATE** | `open` | A service is actively listening here |
| **SERVICE** | `ssh` | Nmap identifies this as SSH based on how it responds |
| **VERSION** | `OpenSSH 6.6.1p1` | The exact version of SSH software running ← 🔴 Critical |

### Port State Values Explained

| State | Meaning | What To Do |
|-------|---------|-----------|
| `open` | Service is running and accepting connections | Investigate — check version for exploits |
| `closed` | Port is reachable but no service is running | Less interesting, may open in future |
| `filtered` | Firewall is blocking Nmap's probes | Cannot determine state; WAF may be present |
| `open\|filtered` | Can't tell if open or filtered | Common in UDP scans |

---

### Scan of `microsoft.com` with `-A`

```
PORT    STATE  SERVICE  VERSION
80/tcp  open   http     Kestrel
443/tcp open   https    Kestrel (SSL)
```

**Script Scan Results from Port 80:**
```
http-server-header: Kestrel
http-title: Did not follow redirect
```

**Script Scan Results from Port 443:**
```
ssl-cert: Subject: commonName=*.microsoft.com
http-server-header: Kestrel
```

**OS Detection:**
```
Running (JUST GUESSING): Oracle VirtualBox (98%), QEMU (94%)
No exact OS matches for host
```

**Key Finding:**
> Both Wappalyzer (Day 8) and Nmap's `-sV` flag confirmed the same thing — **Microsoft uses Kestrel** as their web server. This is a great example of how multiple tools cross-validate each other.

---

## 🔴 The Version → Exploit Pipeline

This is the **most important concept** of this lesson:

```
Step 1: Run Nmap with -sV
           ↓
Step 2: Get version number
        Example: OpenSSH 6.6.1p1
           ↓
Step 3: Search on CVE databases
        → https://cve.mitre.org
        → https://www.exploit-db.com
        → https://nvd.nist.gov
           ↓
Step 4: Find a known exploit for that version
        Example: CVE-2016-6515 (OpenSSH DoS vulnerability)
           ↓
Step 5: Use the exploit to gain access to the server
```

> 🎯 **This is why version detection (`-sV`) is the most valuable scan in a penetration test.** A single outdated service version can be the entry point into an entire network.

---

## ⚠️ Important Warnings & Best Practices

### On Scan Duration
Running all flags manually (`-sS -sV -sC -O`) on a real target can take **1.5 to 2+ hours**.
- Use `-A` instead — it's optimized and much more efficient
- Use `-F` for a quick initial overview (top 100 ports only)
- Use `-T4` to speed up scans (e.g., `sudo nmap -A -T4 <target>`)

### On Stealth
- `-sS` (SYN scan) is **stealthier** than `-sT` (full connect scan) because it never completes the TCP handshake
- Even so, modern IDS/IPS systems can detect SYN scans — don't assume you're invisible
- The firewall evasion flags (`-f`, `-D`) work better on older or internal systems

### On Legal Use
> ⚠️ **Only scan systems you have explicit, written permission to test.**
> Scanning systems without permission is **illegal** in most countries.
> Use `scanme.nmap.org` for practice — it is the official legal Nmap practice target.

---

## 🔁 Complete Recon + Scanning Workflow (Days 8–11)

```
TARGET: example.com
         │
         ├─ [Day 8]  Wappalyzer / BuiltWith / WhatWeb
         │            → Tech stack, frameworks, CDN, WAF hints
         │
         ├─ [Day 9]  Subfinder / c99.nl
         │            → All subdomains (expand attack surface)
         │
         ├─ [Day 10] wafw00f
         │            → Identify WAF (Cloudflare, AWS WAF, etc.)
         │
         └─ [Day 11] sudo nmap -A example.com
                      → Open ports, running services, software versions, OS
                               │
                               ▼
                    ┌─────────────────────────┐
                    │   COMPLETE RECON DATA   │
                    │  • Tech stack known     │
                    │  • Subdomains mapped    │
                    │  • WAF identified       │
                    │  • Ports & versions     │
                    └─────────────────────────┘
                               │
                               ▼
                    Search for CVEs/Exploits
                    for identified versions
```

---

## ✅ Key Takeaways

1. **Ports = doorways** — each connects you to a specific service on a computer
2. There are 65,535 ports; well-known ports are 0–1023
3. **TCP** = reliable, connection-oriented, slower → use for accuracy-critical tasks
4. **UDP** = fast, connectionless, no delivery guarantee → use for speed-critical tasks
5. **Memorize the 7 ports:** 20 (FTP data), 21 (FTP control), 22 (SSH), 25 (SMTP), 53 (DNS/UDP), 80 (HTTP), 443 (HTTPS)
6. **FTP uses two ports** — 21 for control/commands, 20 for actual data transfer
7. `sudo nmap -A <target>` is the **single most useful Nmap command** — covers SYN scan + version + scripts + OS + traceroute
8. **Version numbers are critical** — they are the direct path to CVE exploit research
9. Port state `filtered` = firewall is blocking Nmap → WAF is likely present
10. **Never scan without permission** — use `scanme.nmap.org` for legal practice

---

## 📚 Quick Reference Card

```
┌──────────────────────────────────────────────────────┐
│                NMAP QUICK REFERENCE                  │
├──────────────────┬───────────────────────────────────┤
│ sudo nmap -A T   │ Aggressive scan (all-in-one) ⭐   │
│ sudo nmap -sS T  │ Stealthy SYN scan                │
│ sudo nmap -sV T  │ Version detection                │
│ sudo nmap -sC T  │ Default script scan              │
│ sudo nmap -O  T  │ OS detection                     │
│ sudo nmap -Pn T  │ Skip ping (if host blocks pings) │
│ sudo nmap -F  T  │ Fast scan (top 100 ports)        │
│ sudo nmap -p 22,80,443 T │ Scan specific ports      │
│ sudo nmap -A -oN out.txt T │ Save results to file   │
├──────────────────┴───────────────────────────────────┤
│ Practice target: scanme.nmap.org  (legal & safe)     │
└──────────────────────────────────────────────────────┘

PORT CHEAT SHEET
20 → FTP Data       21 → FTP Control    22 → SSH
25 → SMTP           53 → DNS (UDP)      80 → HTTP
443 → HTTPS
```

---

*Series: Learn Ethical Hacking in 30 Days | Day 11 of 30*