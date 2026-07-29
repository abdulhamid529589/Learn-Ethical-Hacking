# Dark Web, Tor Network & Anonymous Browsing

### Cybersecurity Student Guide — Educational & Defensive Focus

> **Legal Notice:** This guide is for cybersecurity education. Tor browser is completely legal software. Accessing the dark web itself is legal in most countries. What is illegal is accessing illegal content (CSAM, weapon markets, stolen data markets) regardless of which network you use. This guide focuses on understanding the technology, staying safe, and defensive knowledge.

---

## Table of Contents

1. [What Is the Dark Web?](#1-what-is-the-dark-web)
2. [How Tor Network Works](#2-how-tor-network-works)
3. [Setting Up Tor Safely](#3-setting-up-tor-safely)
4. [Security Configuration](#4-security-configuration)
5. [.onion Sites — What They Are](#5-onion-sites--what-they-are)
6. [Legitimate Uses of Tor](#6-legitimate-uses-of-tor)
7. [Threat Modeling — Who Are You Hiding From?](#7-threat-modeling--who-are-you-hiding-from)
8. [OPSEC — Operational Security](#8-opsec--operational-security)
9. [Dark Web for Security Research](#9-dark-web-for-security-research)
10. [Defensive Knowledge — What Attackers Use It For](#10-defensive-knowledge--what-attackers-use-it-for)
11. [Lab Exercise — Safe Tor Setup in VM](#11-lab-exercise--safe-tor-setup-in-vm)

---

## 1. What Is the Dark Web?

### 1.1 The Three Layers of the Internet

The internet is commonly divided into three layers:

```
SURFACE WEB (4% of internet)
├── Indexed by Google, Bing, etc.
├── Publicly accessible websites
├── Social media, news, e-commerce
└── What most people use daily

DEEP WEB (90%+ of internet)
├── NOT indexed by search engines
├── NOT necessarily illegal
├── Examples:
│   ├── Your email inbox
│   ├── Online banking pages
│── ├── Hospital patient records
│   ├── Corporate intranets
│   ├── Paid databases (academic journals)
│   └── Private cloud storage
└── Accessed with normal browsers, just requires login

DARK WEB (<1% of internet)
├── Intentionally hidden, not indexed
├── Requires special software (Tor, I2P)
├── Mix of:
│   ├── Legitimate: Privacy forums, journalism, whistleblowing
│   └── Illegal: Drug markets, stolen data, etc.
└── .onion domain suffix
```

**Key point:** The deep web is NOT the dark web. Your Gmail inbox is deep web. Your hospital records are deep web. These are normal, legal, private things.

### 1.2 Dark Web vs Deep Web — Common Misconception

Most media conflates "dark web" and "deep web." They are different:

| Feature        | Deep Web                | Dark Web                              |
| -------------- | ----------------------- | ------------------------------------- |
| Access method  | Normal browser + login  | Tor browser required                  |
| Examples       | Gmail, banking, Netflix | .onion sites                          |
| Legal status   | Completely normal       | Legal to access; some content illegal |
| Size           | Enormous (90%+)         | Very small                            |
| Search engines | Not indexed (private)   | Special search engines (.onion)       |

### 1.3 What Actually Exists on Dark Web

**Legitimate content:**

- Privacy-focused forums and communities
- Whistleblower platforms (SecureDrop — used by major newspapers)
- Journalism resources
- Political dissident communities in authoritarian countries
- Academic and security research resources
- Dark web versions of legitimate sites (BBC, DW News, Facebook all have .onion versions)

**Illegal content (what you must avoid):**

- Illegal marketplaces
- Stolen credentials/credit card databases
- CSAM (strictly illegal everywhere — serious prison sentences)
- Counterfeit documents
- Hacking services

**As a cybersecurity student:** Understanding that these exist is important. Accessing illegal content is not. Your goal is defensive knowledge.

---

## 2. How Tor Network Works

### 2.1 The Problem Tor Solves

On the normal internet:

```
You → ISP → Website
(Website sees your real IP)
(ISP sees which websites you visit)
(Anyone monitoring network sees your traffic)
```

### 2.2 Onion Routing — The Core Concept

Tor uses "onion routing" — your traffic is wrapped in multiple layers of encryption, like an onion:

```
Your Data
    ↓
[Encrypted with Exit Node key]
    ↓
[Encrypted with Middle Node key]
    ↓
[Encrypted with Guard Node key]
    ↓
Sent to Guard Node (Entry Node)

Guard Node:
- Knows: Your real IP + Middle Node IP
- Does NOT know: Final destination, content
    ↓
Middle Node (Relay):
- Knows: Guard Node IP + Exit Node IP
- Does NOT know: Your real IP, destination, content
    ↓
Exit Node:
- Knows: Middle Node IP + Final destination
- Does NOT know: Your real IP
    ↓
Website/Service
- Sees: Exit Node IP (not your real IP)
```

### 2.3 Circuit Building Process

```
Step 1: Tor client downloads consensus file from Directory Servers
        (List of all Tor relays worldwide — 6000+ relays)

Step 2: Client selects 3 relays:
        Guard (Entry) → Relay (Middle) → Exit

Step 3: Client negotiates encryption keys with each hop separately
        (Each hop only knows adjacent hops)

Step 4: Traffic travels through 3-hop circuit

Step 5: New circuit built every 10 minutes
```

### 2.4 .onion Hidden Services — How They Work

.onion sites are different from normal Tor usage:

```
Normal Tor:
You → Tor circuit → Exit Node → Clearnet website
(Exit node knows the destination)

.onion Hidden Service:
You → Tor circuit → Rendezvous Point ← Tor circuit ← .onion server
(Neither side knows the other's real IP)
(No exit node needed — stays within Tor network)
```

**How a .onion address is generated:**

```python
# Simplified explanation:
# .onion address is derived from the public key of the hidden service

import hashlib

# Generate key pair
private_key = generate_ed25519_keypair()
public_key = private_key.public_key()

# Derive .onion address
onion_address = base32(public_key_hash) + ".onion"
# Result: something like 3g2upl4pq6kufc4m.onion
```

This means .onion addresses ARE cryptographic identifiers — they prove you're talking to the right server.

### 2.5 Tor vs VPN vs Proxy — Comparison

| Feature              | Tor                   | VPN                         | Proxy          |
| -------------------- | --------------------- | --------------------------- | -------------- |
| Anonymity            | High (3 hops)         | Medium (trust VPN provider) | Low            |
| Speed                | Slow                  | Fast                        | Fast           |
| Cost                 | Free                  | Paid (usually)              | Free/Paid      |
| Trust required       | No single party       | VPN provider                | Proxy operator |
| Exit node visibility | Exit sees destination | VPN sees all                | Proxy sees all |
| .onion access        | Yes                   | No                          | No             |
| IP hidden from ISP   | Yes (ISP sees Tor)    | Yes                         | Partially      |

---

## 3. Setting Up Tor Safely

### 3.1 Installation

**On Kali Linux (your lab):**

```bash
# Method 1: Using apt
sudo apt update
sudo apt install tor torbrowser-launcher

# Launch Tor browser:
torbrowser-launcher

# Method 2: From official Tor Project
# Download from: https://www.torproject.org/download/
# Verify signature before installing
```

**Verify download signature (important):**

```bash
# Download Tor Browser + signature file (.asc)
# Import Tor Project signing key:
gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org

# Verify the downloaded file:
gpg --verify tor-browser-linux64-13.0_en-US.tar.xz.asc tor-browser-linux64-13.0_en-US.tar.xz

# Should say: "Good signature from ..."
# If BAD signature: Do NOT use this file — tampered download
```

**On Windows:**

- Download from torproject.org ONLY (not third-party sites)
- Run installer
- Verify signature using Gpg4win

### 3.2 First Launch Configuration

```
When Tor Browser opens:
1. Click "Connect" (auto-connect to Tor network)
   OR
   If Tor is blocked in your country → "Configure Connection"
   → Use bridges (obfuscated relays that look like normal traffic)

2. Wait for connection (usually 10–30 seconds)
3. Green onion icon = connected to Tor
```

---

## 4. Security Configuration

### 4.1 Security Level Settings

The transcript mentioned this — it's critical:

```
Click Shield icon (top right) → Advanced Security Settings

Security Levels:
┌─────────────────────────────────────────────────────────┐
│ Standard    → All features enabled                      │
│              (fastest but least private)                 │
│                                                          │
│ Safer       → JavaScript disabled on non-HTTPS sites    │
│              Some fonts blocked                          │
│                                                          │
│ Safest      → JavaScript disabled EVERYWHERE            │
│              Most secure, some sites break              │
│              RECOMMENDED for dark web browsing          │
└─────────────────────────────────────────────────────────┘
```

**Why disable JavaScript?**
JavaScript can be used to:

- Reveal your real IP address (WebRTC leaks)
- Fingerprint your browser
- Execute malicious code
- Bypass Tor anonymity

The FBI famously de-anonymized Tor users by exploiting a Firefox JavaScript vulnerability in 2013.

### 4.2 Permission Blocking

As the transcript mentioned — block all permission requests:

```
Tor Browser → Settings (hamburger menu) → Privacy & Security

Under Permissions, click Settings next to each:
• Location → Block new requests
• Camera → Block new requests
• Microphone → Block new requests
• Notifications → Block new requests
• Autoplay → Block audio and video

Why: Malicious sites request these to gather device information
     or attempt deanonymization
```

### 4.3 Additional Settings

```
In Tor Browser Settings:

General:
• Do NOT make Tor Browser your default browser
• Do NOT allow popups

Privacy & Security:
• History: Never remember history
• Cookies: Clear on close
• Block dangerous content: Enable
• Block dangerous downloads: Enable

NEVER:
• Install browser extensions/add-ons (breaks fingerprinting protection)
• Maximize the browser window (screen size is fingerprint data)
• Login to personal accounts (Google, Facebook, etc.)
• Torrent over Tor (bypasses anonymity)
• Open downloaded files while connected to Tor
```

### 4.4 WebRTC Leak Test

WebRTC can leak your real IP even through Tor:

```bash
# Test for WebRTC leaks:
# 1. Connect to Tor
# 2. Visit: https://ipleak.net (in Tor browser)
# 3. Check: Does it show your real IP anywhere?
# 4. Also check: https://browserleaks.com

# If leak detected:
# Tor Browser should block WebRTC by default
# If not: about:config → media.peerconnection.enabled → false
```

---

## 5. .onion Sites — What They Are

### 5.1 Finding .onion Sites

The transcript mentioned OSINT Framework — that's a legitimate resource:

**OSINT Framework (osintframework.com):**

- Legitimate OSINT (Open Source Intelligence) tool
- Used by security researchers and investigators
- Has links to various resources including dark web search engines

**Dark Web Search Engines (legitimate):**

```
Ahmia (ahmia.fi) — indexes clearnet-accessible .onion sites
                   Can use on normal browser to find .onion URLs
                   Filters out illegal content

DuckDuckGo on Tor — duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion
                    Privacy-focused search engine

Torch — dark web search engine (results unfiltered — be careful)
```

### 5.2 Legitimate .onion Sites for Education

These are real, legitimate .onion sites operated by known organizations:

```
The Tor Project:         http://2gzyxa5ihm7nsggfxnu52rck2vv4rvmdlkiu3zzui5du4xyclen53wid.onion
BBC News (Tor mirror):   https://www.bbcnewsd73hkzno2ini43t4gblxvycyac5yjdud4dxhj5gucngv36ad.onion
DW (Deutsche Welle):     https://www.dwnewsgngmhlplxy6o2twtfgjnrnjxbegbwqx6wnotdhkzt562tszfid.onion
The New York Times:      https://www.nytimesn7cgmftshazwhfgzm37qxb44r64ytbb2dj3x62d2lljsciiyd.onion
ProPublica:              https://p53lf57qovyuvwsc6xnrppyply3vtqm7l6pcobkmyqg2tf3qjblaipyd.onion
Facebook (yes, really):  https://www.facebookwkhpilnemxj7asber7cybul79eqt1hcwmse7ry5bsb2gv2d7byd.onion
SecureDrop:              Many news organizations run SecureDrop over Tor for anonymous tips
```

**Why do these organizations have .onion sites?**

- Users in countries where these sites are blocked can still access them
- Extra layer of privacy for readers
- Protects journalists' sources
- Best practice for privacy-respecting organizations

### 5.3 SecureDrop — Whistleblower Platform

SecureDrop is used by major news organizations to securely receive leaked documents:

```
Users: Washington Post, Guardian, NY Times, Al Jazeera, many others
Purpose: Whistleblowers submit documents without revealing identity
How: Tor + SecureDrop system + end-to-end encryption
Legal: Completely legal for both journalists and whistleblowers (in most countries)
Used by: Edward Snowden's documents came via this type of system
```

As a security student, understanding SecureDrop is valuable — it shows real-world application of Tor technology.

---

## 6. Legitimate Uses of Tor

### 6.1 Who Uses Tor Legitimately

```
Journalists:
• Communicate with sources in dangerous countries
• Research sensitive stories without ISP logging
• Submit/receive documents securely

Activists and Dissidents:
• Countries like China, Iran, Russia block many websites
• Tor bypasses censorship
• Communicate without government surveillance

Abuse Survivors:
• Research support resources without leaving browser history
• Communicate securely away from abusers

Law Enforcement:
• Investigate dark web crime
• Undercover operations online

Security Researchers:
• Study malware samples
• Monitor threat intelligence
• Understand attacker techniques

Regular Privacy Users:
• Avoid advertiser tracking
• Prevent ISP from selling browsing data
• General privacy preference
```

### 6.2 Tor in Your Country — Bangladesh Context

In Bangladesh:

- Tor is legal to use
- Some websites may be blocked by BTRC (Bangladesh Telecommunication Regulatory Commission)
- Tor can bypass such blocks for educational and research purposes
- Using Tor to access illegal content remains illegal regardless

### 6.3 Corporations Using Dark Web Technologies

```
Banks and Financial Institutions:
• Monitor dark web for stolen customer data
• Threat intelligence gathering
• Check if company credentials are for sale

Companies:
• Monitor if company data was breached and posted
• Track counterfeit goods being sold
• Brand protection research

Government Agencies:
• Cybercrime investigation
• Counterterrorism research
• Law enforcement operations
```

---

## 7. Threat Modeling — Who Are You Hiding From?

### 7.1 What Is Threat Modeling?

Before using privacy tools, ask: **Who is your adversary?**

```
Different adversaries require different protections:

LEVEL 1 — Casual Advertiser/Tracker:
Adversary: Google, Facebook, ISP selling data
Tools needed: VPN + private browser (Tor optional)
Realistic for: Most people who just want privacy

LEVEL 2 — ISP / Local Network Monitor:
Adversary: Campus network, coffee shop, ISP
Tools needed: VPN or Tor
Realistic for: Students, travelers

LEVEL 3 — Corporate Surveillance:
Adversary: Employer monitoring, data brokers
Tools needed: Tor + good OPSEC
Realistic for: Research journalists, activists

LEVEL 4 — Law Enforcement:
Adversary: Police, agencies
Tools needed: Tor + VPN + VM + physical security + air gap
Realistic for: Whistleblowers in dangerous situations

LEVEL 5 — Nation-State Actor:
Adversary: Intelligence agencies (NSA, FSB, etc.)
Tools needed: Extreme OPSEC + custom solutions
Realistic for: High-value targets (not typical users)
```

**For cybersecurity students:** You're typically at Level 1–2 for research purposes.

### 7.2 Tor Limitations — What It Doesn't Protect Against

```
Tor CANNOT protect against:
• User mistakes (logging into personal accounts)
• Malware on your device
• Timing correlation attacks (nation-state level)
• Exit node surveillance (for clearnet sites — use HTTPS)
• Social engineering
• Physical surveillance
• Compromised Tor exit nodes (use .onion to avoid exit nodes)

Tor DOES protect against:
• IP tracking by websites
• ISP monitoring of which sites you visit
• Network-level surveillance
• Censorship
```

---

## 8. OPSEC — Operational Security

### 8.1 OPSEC Basics

OPSEC (Operational Security) is protecting information that could identify you.

**The 5-step OPSEC process:**

```
1. IDENTIFY critical information
   What do you not want revealed?
   (Your real identity, location, activities)

2. ANALYZE threats
   Who might want this information?
   What capabilities do they have?

3. ANALYZE vulnerabilities
   What could reveal your critical information?
   (Browser fingerprint, writing style, metadata)

4. ASSESS risks
   How likely is exposure? How bad would it be?

5. APPLY countermeasures
   What steps reduce the risk?
```

### 8.2 Common OPSEC Failures

```
TECHNICAL FAILURES:
• Using Tor + personal accounts (Google, Facebook)
• Downloading files and opening them while connected to Tor
• Not updating Tor Browser (security patches important)
• Installing browser extensions that fingerprint you
• Using Tor from a fixed location always

BEHAVIORAL FAILURES:
• Using the same username across dark/clearnet
• Writing style analysis (stylometry) can identify authors
• Time correlation: posting at unusual hours narrows location
• Reusing email addresses
• Mixing anonymous and real activities on same device

REAL EXAMPLE: Silk Road founder (Ross Ulbricht)
• Used his real email in an early forum post asking about coding
• FBI found this and connected it to his dark web persona
• Single OPSEC failure → life sentence
```

### 8.3 Metadata — The Hidden Data

Metadata can reveal identity even when content is protected:

```python
# Example: An image file contains metadata
from PIL import Image
import piexif

# Load an image
img = Image.open('photo.jpg')
exif_data = piexif.load(img.info['exif'])

# What metadata can reveal:
# GPS coordinates (exact location where photo was taken)
# Device make and model (iPhone 14 Pro, Samsung Galaxy S23)
# Date and time
# Camera settings
# Sometimes: Software version, editing software used

# Remove metadata before sharing:
# Linux:
# exiftool -all= filename.jpg    # removes all metadata
# mat2 filename.jpg              # privacy-focused metadata removal
```

**Always strip metadata from files before sharing anonymously.**

---

## 9. Dark Web for Security Research

### 9.1 Threat Intelligence Gathering

Security professionals legitimately monitor dark web for:

```
What researchers look for:
• Company credentials for sale (breached employee accounts)
• Stolen customer databases
• Company intellectual property
• Zero-day exploits being sold
• New malware samples being offered
• Ransomware group announcements
• APT (Advanced Persistent Threat) forums

Why this matters for defenders:
• Early warning of breaches (before company realizes)
• Understanding attacker capabilities
• Monitoring for leaked internal data
• Tracking ransomware operators
```

### 9.2 Have I Been Pwned — Dark Web Monitoring

```
haveibeenpwned.com (clearnet, legitimate service by Troy Hunt):
• Checks if your email is in known breach databases
• Many breaches found from dark web marketplaces
• Free for individuals
• Used by security teams for monitoring

API usage:
import requests

def check_email(email):
    url = f"https://haveibeenpwned.com/api/v3/breachedaccount/{email}"
    headers = {"hibp-api-key": "your_api_key"}
    response = requests.get(url, headers=headers)

    if response.status_code == 200:
        breaches = response.json()
        return f"Found in {len(breaches)} breaches"
    elif response.status_code == 404:
        return "Not found in any breach"
```

### 9.3 Dark Web Monitoring Tools for Organizations

```
Commercial services:
• Recorded Future — threat intelligence platform
• DarkOwl — dark web data collection
• Flashpoint — intelligence for risk management
• Digital Shadows — external threat intelligence

Open source:
• OnionScan — scanning .onion sites for misconfigurations
• TorBot — dark web OSINT tool
• Ahmia — dark web search engine API

For learning:
• TryHackMe has dark web OSINT rooms
• OSINT Framework has resources for dark web research
```

### 9.4 Reading Dark Web Data Legally

As a security researcher you can:

- Monitor breach announcements
- Read threat actor forums that are publicly accessible
- Study malware samples in controlled environments
- Track ransomware group activity

You cannot:

- Purchase stolen data even for research (receiving stolen property)
- Download CSAM under any circumstances
- Interact with illegal markets
- Use threat actor tools against real systems without authorization

---

## 10. Defensive Knowledge — What Attackers Use It For

Understanding attacker behavior helps defenders.

### 10.1 Cybercriminal Dark Web Activities

```
Ransomware Groups:
• Operate leak sites (.onion) where they post stolen data
• Negotiate ransoms through .onion chats
• Recruit affiliates through dark web forums
• Sell ransomware-as-a-service

Initial Access Brokers (IABs):
• Hack into corporate networks
• Sell access to ransomware groups
• Post on dark web forums: "Access to [Company] network, 500 employees, $5000"

Credential Markets:
• Sell combo lists (email:password from breaches)
• Corporate VPN credentials
• RDP access to servers
• Bank account access

Malware Services:
• Malware-as-a-Service (MaaS)
• Botnets for rent (DDoS)
• Custom malware development
```

### 10.2 How Law Enforcement Takes Down Dark Web Markets

Understanding this helps you see the limits of dark web anonymity:

```
Silk Road (2013):
• Ross Ulbricht used real email in early forum post
• Careless OPSEC led to identification
• FBI agent located server through mistake in CAPTCHA system

AlphaBay (2017):
• Admin used same username (Alpha02) on clearnet forums
• Used personal email for early site accounts
• Server found in Canada through legal process

Hansa Market (2017):
• Dutch police took over the market covertly
• Ran it for a month, gathering user data
• Then announced seizure — 10,000 users identified

Common takedown methods:
• OPSEC mistakes by operators
• Informants (human intelligence)
• Server seizure (hosting misconfigurations)
• Financial tracking (cryptocurrency is NOT anonymous)
• Exit node surveillance
• Timing correlation attacks
```

### 10.3 Bitcoin Is NOT Anonymous

This is a critical misconception:

```
Bitcoin is PSEUDONYMOUS, not anonymous:

All transactions are on public blockchain:
• Every wallet address visible
• Every transaction permanently recorded
• Amount, timestamp, sender, receiver (by address) all public

Chain analysis can trace:
• Coinbase (exchange) → wallet → dark web market → cash out
• Chainalysis, CipherTrace used by FBI, Europol
• Many dark web operators traced through Bitcoin

Monero (XMR):
• Designed for privacy (ring signatures, stealth addresses)
• Harder to trace, but not impossible
• Used by many dark web markets now

Key point for defenders:
Cryptocurrency transactions leave permanent trails
Law enforcement has sophisticated blockchain analysis tools
Many criminals caught through financial forensics
```

---

## 11. Lab Exercise — Safe Tor Setup in VM

### 11.1 Setup — Isolated VM Environment

**As mentioned in the transcript and your existing cybersecurity lab:**

```bash
# Step 1: Create dedicated VM for Tor research
# In VirtualBox: New VM
# OS: Kali Linux or Tails OS
# Network: NAT (not bridged — reduces footprint)
# Snapshot: Take clean snapshot before any research

# Step 2: Install Tor Browser in VM
sudo apt update
sudo apt install torbrowser-launcher
torbrowser-launcher  # First run downloads and installs

# Step 3: Verify Tor is working
# Visit in Tor Browser: https://check.torproject.org
# Should say: "Congratulations. This browser is configured to use Tor."
```

### 11.2 Tails OS — Better for Serious Research

Tails OS is an entire operating system designed for privacy:

```
Why Tails is better than just Tor Browser:
• Runs from USB — leaves no trace on computer
• All traffic automatically routed through Tor
• RAM only — nothing written to disk
• Amnesic — everything erased on shutdown
• Used by journalists, activists, security researchers worldwide

Download: tails.boum.org
Verify signature (ALWAYS verify Tails download)
Write to USB: balenaEtcher or dd command

Usage:
• Boot from USB
• Connect to network
• All traffic automatically through Tor
• Session disappears on shutdown
```

### 11.3 Exercise — Verifying Tor Anonymity

```bash
# In Tor Browser:

# Test 1: Check your IP
# Visit: https://whatismyip.com
# Should show Tor exit node IP, not your real IP

# Test 2: Verify Tor routing
# Visit: https://check.torproject.org
# Should confirm Tor is active

# Test 3: Check for leaks
# Visit: https://ipleak.net
# Check: IP, DNS, WebRTC — none should reveal your real IP

# Test 4: Check fingerprint
# Visit: https://coveryourtracks.eff.org
# See how unique your browser fingerprint is
# Tor Browser should show similar fingerprint to other Tor users

# Test 5: View your circuit
# Click padlock icon in Tor Browser URL bar
# "View circuit" shows your 3-hop path
# Note: Country of each relay
```

### 11.4 Understanding the Circuit Display

```
Example circuit:
You → [Germany - Guard] → [Netherlands - Middle] → [USA - Exit] → Website

What you see:
• Guard node: First node, knows your real IP
• Middle node: Relay, anonymizes between guard and exit
• Exit node: Last node, connects to final destination

Change circuit:
• Click shield icon → "New Circuit for this Site"
• Tor → "New Identity" (restarts browser, new circuit for everything)
```

### 11.5 Research Exercise — OSINT Framework

```
Visit: osintframework.com (clearnet — normal browser)
Navigate: OSINT Framework → Dark Web → Tor

Explore the categories:
• .onion link directories
• Dark web search engines
• Security research resources

Practice: Find legitimate .onion sites for:
• A major news organization
• The Tor Project itself
• SecureDrop for a known newspaper

Document: URL structure of .onion addresses
Observe: Length and format of .onion v3 addresses (56 characters + .onion)
```

---

## 12. Quick Reference — Tor Commands

```bash
# Install Tor service (not browser):
sudo apt install tor

# Start Tor service:
sudo systemctl start tor
sudo systemctl enable tor  # Start on boot

# Check Tor is running:
sudo systemctl status tor

# Use Tor with curl (test):
curl --socks5-hostname localhost:9050 https://check.torproject.org/api/ip

# Use torsocks to route applications through Tor:
sudo apt install torsocks
torsocks wget https://website.com  # Routes wget through Tor
torsocks curl https://website.com  # Routes curl through Tor

# Tor Browser launcher:
torbrowser-launcher

# Check Tor log:
sudo journalctl -u tor

# Tor configuration file:
sudo nano /etc/tor/torrc

# Add entry node/exit node constraints (advanced):
# EntryNodes {us} — only use US entry nodes
# ExitNodes {de} — only use German exit nodes
# StrictNodes 1 — enforce the above

# Hidden service setup (host your own .onion):
# In /etc/tor/torrc add:
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:8080
# After restart: cat /var/lib/tor/hidden_service/hostname
# That's your .onion address
```

---

## Summary — Key Takeaways

```
TECHNOLOGY:
✓ Tor uses 3-hop onion routing for anonymity
✓ .onion sites are end-to-end anonymous (no exit node)
✓ JavaScript is the biggest deanonymization risk
✓ Bitcoin is traceable — not true anonymity

SAFETY RULES:
✓ Always use VM for research (not your main system — as transcript says)
✓ Set security level to Safest
✓ Block all permission requests
✓ Never login to personal accounts over Tor
✓ Never open downloaded files while connected
✓ Strip metadata from any files

LEGITIMATE USE:
✓ Journalists use Tor — BBC, NY Times have .onion sites
✓ SecureDrop is critical whistleblower infrastructure
✓ Security researchers monitor dark web legitimately
✓ Privacy is a valid reason to use Tor

LEGAL BOUNDARIES:
✓ Tor browser = legal everywhere (almost)
✓ Dark web access = legal
✓ Illegal content = illegal regardless of network
✓ Cryptocurrency is NOT anonymous — transactions are traced

FOR CYBERSECURITY CAREER:
✓ Understanding Tor helps you defend against threats
✓ Threat intelligence includes dark web monitoring
✓ Many companies pay for dark web monitoring services
✓ OPSEC knowledge is valuable in corporate security roles
```

---

_This guide covers the educational and defensive aspects of dark web technology. The Tor Project is a legitimate non-profit organization that develops privacy tools. Journalists, human rights workers, and security professionals use these tools daily for legitimate purposes._
