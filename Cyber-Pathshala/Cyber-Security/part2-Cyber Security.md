# Cyber Security & Ethical Hacking — Course Notes

> Compiled and organized from lecture transcripts (Cyber Pathshala / CPCH course).
> These notes are intended for personal academic study.

---

## Table of Contents

1. [Course Introduction & Career Roadmap](#1-course-introduction--career-roadmap)
2. [Types of Hackers](#2-types-of-hackers)
3. [Networking Fundamentals](#3-networking-fundamentals)
- 3.1 [What is Computer Networking](#31-what-is-computer-networking)
- 3.2 [Types of Networks — LAN, MAN, WAN](#32-types-of-networks--lan-man-wan)
- 3.3 [Key Networking Entities](#33-key-networking-entities)
4. [IP Addressing](#4-ip-addressing)
- 4.1 [What is an IP Address](#41-what-is-an-ip-address)
- 4.2 [Tracking Using IP](#42-tracking-using-ip)
- 4.3 [Public vs Private IP](#43-public-vs-private-ip)
- 4.4 [Static vs Dynamic IP](#44-static-vs-dynamic-ip)
- 4.5 [Rules of IP Addressing](#45-rules-of-ip-addressing)
5. [MAC Address](#5-mac-address)
6. [Ports](#6-ports)
7. [Protocols & Networking Models](#7-protocols--networking-models)
- 7.1 [What is a Protocol](#71-what-is-a-protocol)
- 7.2 [OSI Model](#72-osi-model)
- 7.3 [TCP/IP Model](#73-tcpip-model)
- 7.4 [TCP Flags & Three-Way Handshake](#74-tcp-flags--three-way-handshake)
- 7.5 [TCP vs UDP](#75-tcp-vs-udp)
8. [How the Internet Works — Servers, Domains, DNS](#8-how-the-internet-works--servers-domains-dns)
- 8.1 [Servers](#81-servers)
- 8.2 [Domains & Subdomains](#82-domains--subdomains)
- 8.3 [DNS Records](#83-dns-records)
- 8.4 [How DNS Resolution Works](#84-how-dns-resolution-works)
- 8.5 [Zone Files](#85-zone-files)
9. [End-to-End Networking Flow (ISP → Router → Devices)](#9-end-to-end-networking-flow-isp--router--devices)
10. [Linux Fundamentals](#10-linux-fundamentals)
- 10.1 [What is Linux (Kernel vs OS)](#101-what-is-linux-kernel-vs-os)
- 10.2 [Open Source Concept](#102-open-source-concept)
- 10.3 [Linux File System Hierarchy](#103-linux-file-system-hierarchy)
- 10.4 [Basic Linux Commands](#104-basic-linux-commands)
- 10.5 [File Permissions](#105-file-permissions)
- 10.6 [Package Management (APT)](#106-package-management-apt)
11. [Information Gathering (Footprinting & Reconnaissance)](#11-information-gathering-footprinting--reconnaissance)
- 11.1 [Why Information Gathering Matters](#111-why-information-gathering-matters)
- 11.2 [Active vs Passive Information Gathering](#112-active-vs-passive-information-gathering)
- 11.3 [Search Engines & Google Dorking](#113-search-engines--google-dorking)
- 11.4 [OSINT Framework](#114-osint-framework)
- 11.5 [Shodan](#115-shodan)
- 11.6 [theHarvester Tool](#116-theharvester-tool)
12. [Network Scanning & Enumeration (Nmap)](#12-network-scanning--enumeration-nmap)
- 12.1 [Five Goals of Network Scanning](#121-five-goals-of-network-scanning)
- 12.2 [Practical Nmap Scanning](#122-practical-nmap-scanning)
- 12.3 [Enumeration with NSE Scripts](#123-enumeration-with-nse-scripts)
13. [Metasploit Framework & Armitage (Overview)](#13-metasploit-framework--armitage-overview)
14. [Lab Setup Notes](#14-lab-setup-notes)
15. [Miscellaneous Topics Covered](#15-miscellaneous-topics-covered)

---

## 1. Course Introduction & Career Roadmap

### About the Course (CPCH – Cyber Pathshala Certified Ethical Hacking)
- A beginner-to-advanced curriculum covering **12 premium modules**: Networking, Lab Setup, Linux OS, Footprinting & Recon, Network Scanning, Network Enumeration, Offensive System Hacking (Red Teaming), Web Pentesting, Defensive security (Blue Teaming), Being Anonymous, Career Counselling, and Bonus resources.
- Structure: ~45 hours of content delivered as 30 days of live classes (1.5 hrs/day).
- Goal: Take a learner from **zero programming/networking background** to being capable of understanding real-world penetration testing.

### Why Cyber Security Matters (Market Data Discussed)
- Reported cybercrime in India rose from ~25,000 cases (2019) to ~24 lakh cases (2024–2025) — a massive multi-year increase, largely attributed to job losses during COVID-19 pushing people toward cybercrime as an income source.
- Global cybercrime cost is projected to reach **$10.5 trillion by 2025**.
- Demand for security professionals rises directly with cybercrime rates (analogy: more crime → more police needed → more cybercrime → more security professionals needed).

### Learning Roadmap (Four Pillars: **Learn → Earn → Certifications → Sources**)

**A. LEARN (What to study, in order):**
1. **Networking basics** — IP, Ports, Protocols, OSI/TCP-IP models (just enough to support hacking, not a full networking specialization).
2. **Linux OS & Commands** — file system, permissions, package management.
3. **Ethical Hacking fundamentals** — footprinting, scanning, enumeration, vulnerability analysis, exploitation, securing systems. This is considered the **entry point** into the broader Cyber Security field.
4. **Pick a Cyber Security domain** (after finishing Ethical Hacking basics) — e.g., Web Pentesting, Network Pentesting, Mobile/APK Pentesting, IoT Pentesting, AI Pentesting, Cloud Security, etc. Choose based on personal interest, not just market hype.

**B. QUALIFICATIONS (formal education path by stage):**
| Current Stage | Recommended Path |
|---|---|
| Before 10th grade | Focus purely on skills (networking, Linux, ethical hacking) |
| 10th grade | Choose **Science with Maths** for 11th–12th to enable Engineering routes |
| 12th (Science/Maths) | B.Tech / M.Tech in Cyber Security |
| 12th (Non-Science) | BCA / MCA with Cyber Security specialization |
| Already have a non-tech Bachelor's | Pursue a tech-focused Master's in Cyber Security |
| Cannot continue formal education | Focus on **Skills + Certifications + Diploma programs** |

**C. CERTIFICATIONS (by experience level):**
- **Beginner:** eJPT, CEH (Practical & Theory), Security+, Google Cybersecurity Certificate
- **Advanced:** OSCP, CEH-OSWP, Pentest+, eWPT
- **Expert:** OSEP, CRTP, CRTE, GPEN
> Rule of thumb: certifications should align with your chosen domain (e.g., don't take system-hacking certs if your domain is Web Pentesting).

**D. EARN (Career paths after Ethical Hacking):**

*Categories of jobs after completing Ethical Hacking basics (entry-level titles use words like "Junior/Intern/Helper" to signal fresher-friendly roles):*
1. **Offensive Security (Red Teaming):** Junior Penetration Tester, Ethical Hacker, Vulnerability Assessment Analyst, Web App Security Tester, Bug Bounty Hunter, Security Team Associate, Red Team Intern
2. **Defensive Security (Blue Teaming):** SOC Analyst L1, Security Analyst, Threat Monitoring Analyst, Incident Response Intern, SIEM Analyst
3. **Training & Awareness:** Junior Ethical Hacking Trainer, Cyber Security Instructor, Lab Assistant, Training Coordinator, Online Instructor
4. **GRC (Governance, Risk & Compliance):** Junior IT Security Auditor, Compliance Analyst, Risk Analyst, Security Standards Assistant (e.g., ISO 27001)
5. **General IT & Security Support:** IT Support Technician, Network Support Engineer, System Administrator, Security Operations Intern
6. **Freelance / Self-Employment (Bonus):** Bug Bounty Programs (recommended only *after* advanced-level skills, not immediately post-basics), Freelance Pentesting, Teaching/Content Creation, Security Blogging

*After advancing to intermediate/senior level, broader Cyber Security job categories include:* SOC/Blue Team, Red Team, Security Engineer/Architect, GRC, IAM (Identity & Access Management), Cloud/DevSecOps, Network Infrastructure Security, Security Research, and Management/Leadership roles (Security Manager, Team Lead).

**E. SOURCES (Where to find jobs and learning material):**
- **Job platforms:** LinkedIn, Shine, Naukri, and specialized cyber security consultancy firms — always search using fresher-friendly keywords (Junior/Intern/Trainee/Associate), not senior titles like "Manager."
- **Learning materials:** Books (recommended: *The Hacker Playbook 3*), recorded courses, paid bootcamps, blogs/forums, and AI tools for doubt-solving.

---

## 2. Types of Hackers

Hacking is defined as gaining **unauthorized access** to a device/system using technical skill, typically via: information gathering → enumeration → finding vulnerabilities → exploitation.

Hackers are classified using three criteria: **Intent, Legality, and Permission.**

| Type | Intent | Legality | Permission | Motivation | Examples |
|---|---|---|---|---|---|
| **Black Hat** | Malicious | Illegal | No permission | Financial gain, espionage, disruption, revenge | Ransomware attacks, data theft, corporate sabotage |
| **White Hat** | Ethical | Legal | Has permission | Protect organizations, find & report vulnerabilities | Penetration testing, vulnerability assessment, bug bounty (authorized) |
| **Gray Hat** | Mixed/ambiguous | Legal gray area | Sometimes no explicit permission | Demonstrating skill, public interest, occasional financial gain | Unsolicited vulnerability discovery, public disclosure without authorization |

- **Google paid $12 million in bug bounties in 2022** — cited as an example of the value/reward potential of White Hat (authorized) hacking.

---

## 3. Networking Fundamentals

### 3.1 What is Computer Networking
Computer Networking is the process by which **two or more devices communicate** and share data, files, hardware, or software. Two core concerns:
- **Information preservation** (data persists/can be recovered — e.g., a shared file can be re-sent if the original owner loses it).
- **Information security** (protecting shared data).

### 3.2 Types of Networks — LAN, MAN, WAN

| Type | Full Form | Range/Scope | Example |
|---|---|---|---|
| **LAN** | Local Area Network | Small area, ~15–50 devices | Office, home, school computer lab |
| **MAN** | Metropolitan Area Network | Interconnected LANs, larger scope | University campus, a city |
| **WAN** | Wide Area Network | Entire world | The Internet (WWW) |

> Concept: MAN = multiple LANs connected together. WAN = multiple MANs/LANs connected together, spanning countries/continents.

### 3.3 Key Networking Entities
Five core building blocks of networking communication:

1. **ISP (Internet Service Provider)** — Company providing internet access (e.g., Jio, Airtel, BSNL, VI).
2. **MAC Address** — Unique hardware/physical address assigned to every network-capable hardware device.
3. **Port** — The "doors" through which a device sends/receives data (0–65535 available).
4. **Protocol** — Rules and regulations that govern safe/secure data transmission.
5. **IP Address** — Unique logical address needed for a device to communicate over a network.

---

## 4. IP Addressing

### 4.1 What is an IP Address
An **IP (Internet Protocol) Address** uniquely identifies a device on a network, enabling communication.

- **IPv4 format:** Four "pairs" (octets) separated by dots, e.g. `17.172.224.47`
- Each pair ranges from 0–255 (8 bits = 1 byte each), totaling **32 bits (4 bytes)**.

### 4.2 Tracking Using IP
An IP address is *hierarchically structured*, similar to a postal address:
- 1st octet → identifies the **Country**
- 2nd octet → identifies the **State**
- 3rd octet → identifies the **City + ISP** (the specific ISP office that was allocated that IP block)

Tracking process: authorities approach the ISP identified by the IP → the ISP checks its logs of which customer/device was assigned that IP at that specific date/time → identifies the individual user.

> Practical demonstration in class: searching "what is my IPv4" reveals your public IP and its associated city/ISP — demonstrating this hierarchical breakdown live.

### 4.3 Public vs Private IP

| | Private IP | Public IP |
|---|---|---|
| Scope | LAN (Local Area Network) only | WAN (Wide Area Network / Internet) |
| Analogy | Nickname (only close circle uses it) | Real/legal/official name (used universally) |
| Communication Rule | Private ↔ Private only | Public ↔ Public only |

> **Golden Rule:** A Private IP can NEVER directly talk to a Public IP and vice versa. This rule never breaks.

### 4.4 Static vs Dynamic IP

- **Dynamic IP:** Temporarily assigned; reassigned to another device when not actively in use. This is why ~4.3 billion possible IPv4 addresses can serve ~5.45 billion active internet users worldwide — not everyone is online simultaneously, and idle IPs get recycled.
- *Demonstrated experiment:* Toggling Airplane Mode on/off for a few seconds and re-checking your public IP shows the IP has changed.
- **Static IP:** Fixed and does not change — required for machines that must always be reachable at a consistent address (e.g., a **server** hosting a website, which needs a stable public IP so users can consistently reach it).

### 4.5 Rules of IP Addressing
1. Private IP communicates only with Private IP (within LAN).
2. Public IP communicates only with Public IP (within WAN).
3. Private and Public IP can **never** communicate directly with each other.

---

## 5. MAC Address

**MAC (Media Access Control) Address** — also called the **Hardware Address** or **Physical Address** — is a unique identifier assigned to every network-capable hardware device (laptops, phones, routers, etc.).

### Structure
- 6 pairs (48 bits / 6 bytes total), each pair formed from digits `0-9` and letters `A-F`.
- Split into two halves:
- **First 3 pairs → OUI (Organizationally Unique Identifier):** Identifies the **vendor/manufacturer** (e.g., Intel, Nokia, Samsung) and even the region/office that produced it.
- **Last 3 pairs → NIC (Network Interface Controller):** Uniquely identifies the **specific device** made by that vendor.

### Viewing MAC Address (by OS)
| OS | Command |
|---|---|
| Windows | `getmac` (in Command Prompt) |
| Linux | `ifconfig` (in Terminal) |
| macOS | `ifconfig` (in Terminal) |

### Tracing via MAC
A "MAC Lookup" web tool can reveal the vendor name, vendor's registered address, and approximate manufacture/registration date from just the first 3 pairs of a MAC address — vendors can then be contacted for exact device/purchase records for the last 3 pairs.

---

## 6. Ports

**Ports** are the logical "pathways" or "doors" a device uses to send/receive network data. Every device has **65,536 ports available (0–65535)**.

### Two Fundamental Rules of Ports
1. **Only open ports can be used** — a packet sent to a closed port will not be received (like knocking on a locked door — no one answers, package doesn't get delivered).
2. **One port, one process at a time** — if Zoom is using port 8080, no other application (like Skype) can simultaneously use port 8080 on the same machine. This is why you sometimes see the error *"port is already in use."*

### Port Categories

| Range | Name | Description |
|---|---|---|
| **0 – 1023** | **Well-Known Ports** | Reserved for very common/standard services (e.g., HTTP=80, HTTPS=443, FTP=20/21, SSH=22). High chance these are already "busy" on any given machine. |
| **1024 – 49151** | **Registered Ports** | Available for custom applications; commonly used by malware/custom tools since they are less likely to be occupied. |
| **49152 – 65535** | **Dynamic/Private Ports** | Frequently changing, used for short-lived/ephemeral connections; not reliable for stable long-term connections. |

### Common Well-Known Ports (examples referenced)
| Port | Service |
|---|---|
| 20/21 | FTP |
| 22 | SSH |
| 25 | SMTP |
| 80 | HTTP |
| 443 | HTTPS (via SSL/TLS) |

### Practical Note
When crafting malware/payloads, attackers often prefer **Registered Ports** (1024–49151) since Well-Known ports are usually already occupied by legitimate services (causing "port already in use" errors and payload failure).

### Basic Nmap Port Check (introduced early in course)
```bash
nmap <target-ip> -v -p 80
```
Basic scan showing whether a specific port is open and what service runs on it.

---

## 7. Protocols & Networking Models

### 7.1 What is a Protocol
A **protocol** is a "set of rules and regulations" that governs how data is transmitted safely and securely between devices — analogous to traffic rules that keep a journey safe. Without protocols, data transmission would be unsafe/unreliable.

### 7.2 OSI Model
**OSI = Open Systems Interconnection Model** — a **theoretical, 7-layer standard** describing how devices communicate. Always read **bottom-to-top**.

| Layer # | Layer Name | Function |
|---|---|---|
| 7 | Application | User-facing protocols (HTTP, HTTPS, FTP, SMTP, DNS, etc.) |
| 6 | Presentation | Data formatting, compression, encryption (SSL/TLS) — e.g., turns HTTP into HTTPS |
| 5 | Session | Establishes/maintains/tears down the communication session |
| 4 | Transport | Reliable connection-oriented delivery via TCP/UDP |
| 3 | Network | IP addressing and routing (IP, ICMP, ARP, DHCP) |
| 2 | Data Link | Converts data to frames; hands off to physical hardware (NICs, MAC) |
| 1 | Physical | Actual bit-level transmission via signals (Wi-Fi, cables) |

**Example (sending a WhatsApp "Hi" message):**
Application (drafts message) → Presentation (encrypts, e.g. SSL/TLS) → Session (creates connection) → Transport (TCP builds connection) → Network (resolves IP) → Data Link (frames data) → Physical (transmits signal) → *(reverse process on receiver's end)*.

### 7.3 TCP/IP Model
The **TCP/IP Model** is the **practical, real-world implementation** of the OSI model, developed by **DARPA**. It **compresses OSI's 7 layers into 4 layers**:

| TCP/IP Layer | Combines OSI Layers | Example Protocols |
|---|---|---|
| **Application** | Application + Presentation + Session | HTTP, HTTPS, FTP, SMTP, DNS, SSH, POP3, SNMP |
| **Transport** | Transport | TCP, UDP |
| **Internet** | Network | IP, ICMP, ARP, DHCP |
| **Network Access** | Data Link + Physical | Ethernet, Wi-Fi hardware |

> **Security note:** Application Layer sees the most attacks (direct user interaction = most vulnerabilities). Lower layers are generally harder to reach but still have relevant risks.

### 7.4 TCP Flags & Three-Way Handshake

| Flag | Full Form | Purpose |
|---|---|---|
| **URG** | Urgent | Marks a packet for immediate processing |
| **PSH** | Push | Sends data immediately without buffering |
| **FIN** | Finish | Requests graceful connection termination |
| **ACK** | Acknowledgment | Confirms a packet was received |
| **RST** | Reset | Forcefully drops/resets a connection |
| **SYN** | Synchronize | Initiates a new connection request |

**Three-Way Handshake (establishing connection):**
1. Client → Server: **SYN**
2. Server → Client: **SYN + ACK**
3. Client → Server: **ACK**
→ Connection established; each packet carries a **Sequence Number** for ordered reassembly.

**Session Termination (four steps):**
1. A sends **FIN** → 2. B replies **ACK** → 3. B sends its own **FIN** → 4. A replies **ACK** → connection closed.

### 7.5 TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Full Form | Transmission Control Protocol | User Datagram Protocol |
| Connection | Connection-oriented (handshake) | Connectionless |
| Speed | Slower | Faster |
| Reliability | High, ordered delivery | Lower, "fire and forget" |
| Usage | Most services | Select cases (DNS uses both) |

---

## 8. How the Internet Works — Servers, Domains, DNS

### 8.1 Servers
A **server** is a machine with a **static Public IP address**, used to host a service (e.g., a website) for the internet. Public IPs are hard to remember → hence **domain names**.

### 8.2 Domains & Subdomains
- A **Domain** = human-friendly name for an IP address (like saving a phone number under a contact name).
- Bought via registrars (e.g., GoDaddy), managed via a **Domain Manager** panel.
- **Subdomains:** unlimited creation possible from one domain (e.g., `tag.cp.com`) — prefix changes, root extension stays.
- **Extensions:** `.com`, country codes (`.in`, `.jp`, `.ru`), organizational (`.org`, `.net`).

### 8.3 DNS Records

| Record | Purpose |
|---|---|
| **A** | Domain → IPv4 address |
| **AAAA** | Domain → IPv6 address |
| **CNAME** | Alias/redirect to another domain/subdomain |
| **MX** | Points to the mail server for the domain |
| **TXT** | Free-form text; used for SPF (anti-spoofing/anti-spam) |
| **NS** | Identifies authoritative Name Servers; links to the Zone File |
| **SOA** | Admin contact info for the domain |

### 8.4 How DNS Resolution Works

```
User types domain → Local/Global Resolver (checks cache)
↓ (if no cache)
Root Server → TLD Server (e.g., .com) → Name Server
↓
Zone File → A Record → IP Address found → connects to target server
```
Resolved IPs are cached temporarily (browser/router/ISP) to speed up future lookups.

### 8.5 Zone Files
A **publicly accessible** file tied to Name Servers containing a domain's DNS records; refreshes periodically. This is what DNS queries actually resolve against — not the private/login-protected Domain Manager panel.

---

## 9. End-to-End Networking Flow (ISP → Router → Devices)

1. **ISP** maintains a pool of **Public IPs**.
2. On broadband install, the ISP assigns your **router** one Public IP + it has its own MAC address.
3. The router maintains its own pool of **Private IPs** (e.g., `192.168.1.0`–`.255`) for LAN devices.
4. Router reserves the first address for itself (e.g., `192.168.1.1`) = **Default Gateway/Subnet Address**.
5. **DHCP** auto-assigns Private IPs to connecting devices (Subnet Mask defines valid range; **Subnetting** splits large networks into smaller secure groups).
6. **ARP** keeps IP-to-MAC mappings updated via broadcast queries ("Who has X IP?") — non-responders free up their IP for reassignment.
7. **NAT** lets private-IP devices reach the internet: router translates outgoing private-IP traffic to its own public IP, and translates responses back to the correct internal device.

> A mobile SIM effectively acts as a tiny built-in router — it holds the Public IP, not the phone directly.

---

## 10. Linux Fundamentals

### 10.1 What is Linux (Kernel vs OS)
- **Linux is a Kernel, not a full OS by itself.** The Kernel directly communicates with hardware.
- **Shell/terminal tools** (Shell, Bash, Zsh, Fish) let humans interact with the kernel.
- The full OS built around the kernel + GNU tools is properly called **GNU/Linux**.

### 10.2 Open Source Concept
- **Closed/Proprietary:** Source code compiled & hidden; sold as-is.
- **Open Source:** Source code published publicly (e.g., GitHub); community contributions drive rapid improvement.
> GNU = open source; **Unix** (which GNU is modeled after) is proprietary — hence "Unix-like."

### 10.3 Linux File System Hierarchy

| Directory | Purpose |
|---|---|
| `/bin` | Essential user commands |
| `/sbin` | System-critical commands (`reboot`, etc.) |
| `/etc` | **All configuration files** for installed software |
| `/home` | Non-root ("guest") user account data |
| `/root` | Root (admin) user's home directory |
| `/lib` | System libraries |
| `/usr` | User-installed applications |
| `/var` | Variable/log data |
| `/tmp` | Temporary files (often left behind — useful for forensics) |
| `/boot` | Boot-time files |
| `/opt` | Optional/third-party software |
| `/media` | Removable media mount point |
| `/sys` | System/kernel/hardware info |

### 10.4 Basic Linux Commands

| Command | Purpose |
|---|---|
| `pwd` | Show current directory |
| `ls` / `ls <path>` | List directory contents |
| `cd <folder>` / `cd ..` | Change directory / go up one level |
| `mkdir <name>` | Create a directory |
| `cp <src> <dest>` (`-r` for folders) | Copy |
| `mv <src> <dest>` | Move/rename |
| `rmdir <folder>` | Remove empty directory |
| `rm <file>` (`-r` for folders) | Remove |
| `cat <file>` | Print file contents |
| `ifconfig` | Show network interfaces |
| `sudo su` | Elevate to root |
| `<cmd> --help` / `man <cmd>` | Get command help/manual |

### 10.5 File Permissions
Every file gets **Read (r)** + **Write (w)** by default; **Execute (x)** must be granted explicitly.

```bash
chmod -w filename     # remove write permission
chmod +rwx filename   # grant read, write, execute
./filename            # run an executable file
ls -l                 # view detailed permissions
```

### 10.6 Package Management (APT)
```bash
apt update                   # refresh repository lists
apt upgrade                  # upgrade installed packages
apt install <package-name>   # install/update a package
```
> If APT fails to find packages, check `/etc/apt/sources.list` and ensure the `deb-src` line is uncommented.

## 11. Information Gathering (Footprinting & Reconnaissance)

### 11.1 Why Information Gathering Matters
The **first phase** of any authorized security assessment. Illustrated by a comparison: a hacker who spends a week blindly trying every known exploit may fail, while a hacker who spends half a day gathering target information (OS type + version) can look up known CVEs for that exact version and succeed in one attempt. Key takeaways:
- Efficiency and effectiveness both improve dramatically with good recon.
- It is the **foundation** of the entire security assessment process — without it, testing is unfocused guesswork.
- Helps testers stay stealthy/low-profile while gathering data.

### 11.2 Active vs Passive Information Gathering

| Type | Description | Example |
|---|---|---|
| **Active** | Direct interaction/engagement with the target | Sending a friend request (even from a fake profile), pinging a host, direct network probing |
| **Passive** | Indirect, no direct interaction | Browsing a public social media profile without following/reacting |

### 11.3 Search Engines & Google Dorking

**Rule of thumb:** Never rely only on page 1 of search results — search engines optimize for what companies/SEO want you to see, not necessarily what's most useful. Always browse through multiple pages and add specific contextual keywords (e.g., a person's name + relevant field) to surface more targeted, lower-visibility results.

**Google Advanced Search / "Google Dorking"** — Google provides a structured **Advanced Search form** (`google.com/advanced_search`) that auto-generates the correct dork syntax for you. Each field maps to a specific search operator:

| Advanced Search Field | Generated Operator | Purpose |
|---|---|---|
| all these words | (plain words, space-separated) | Every listed word should appear somewhere in the result |
| this exact word or phrase | `"word or phrase"` | Exact match — highest priority term, wrapped in quotes |
| any of these words | `word1 OR word2 OR word3` | Match if *any one* of the listed words appears |
| none of these words | `-word` | Exclude pages containing this word |
| numbers ranging from | `number1..number2` | Filter by a numeric range (dates, prices, quantities) |
| language | (locale filter) | Restrict results to a specific language |
| region | (region filter) | Restrict results to a specific country/region |
| last update | (freshness filter) | Restrict by how recently the page was updated (24h / week / month / year) |
| site or domain | `site:example.com` | Restrict results to one specific site/domain |
| terms appearing | `intitle:` / `inurl:` / `intext:` / `inlink:` | Restrict *where* the term must appear — page title, URL, body text, or in outbound links |
| file type | `filetype:pdf` (or doc, xls, etc.) | Restrict results to a specific file type |
| usage rights | (licensing filter) | Restrict to freely reusable/licensed content |

**Manual (typed) dork syntax equivalent example used in class:**
```
"Ashish" (admin OR Instagram OR Facebook) -Linkedin site:facebook.com
```
This combines an exact-match term, an OR-group, an exclusion, and a site restriction — demonstrating how stacking operators progressively narrows a search. The instructor's key teaching point: **don't over-filter** — adding filters (like `site:`, `filetype:`, or a numeric range) that don't actually apply to your target will return *zero* useful results, so only apply a filter when you have a real reason to believe it will help.

### 11.4 OSINT Framework
A curated web directory organizing OSINT tools by category (Username, Email, Domain, etc.), with sub-branches (WHOIS records, subdomains, DNS discovery) linking to specific free lookup tools.

*Example output fields available via such tools:* WHOIS record (registrant/contact info), Network WHOIS (hosting provider), DNS Records (A, TXT, MX, NS, PTR, SOA), Traceroute (network path/hops), and a basic open-port/service-version summary.

### 11.5 Shodan
**Shodan** is a specialized search engine indexing internet-connected devices/services — a standard OSINT/asset-discovery tool. Requires free registration (email verification) and provides an **API key** for integrating it into other recon tools.

### 11.6 theHarvester Tool
**theHarvester** is a popular open-source OSINT tool (pre-installed on Kali) that aggregates domain-related data from many public sources in a single command.

**Setup:** Add API keys (e.g., Shodan) to `/etc/theHarvester/api-keys.yaml`.

```bash
theHarvester -d <target-domain> -b all -l 100
```
- `-d` target domain · `-b` data source(s) (`all` = every configured source) · `-l` result limit · `-f <file>` save output

---

## 12. Network Scanning & Enumeration (Nmap)

### 12.1 Five Goals of Network Scanning

1. **Host Discovery** — Is the target IP active/reachable?
2. **Port Discovery** — Which of 65,536 ports are open?
3. **Service Discovery** — Which protocol runs on each open port?
4. **Software & Version Discovery** — Which software + version manages that service?
5. **OS Discovery ("Banner Grabbing")** — What OS/version is running?

> **Banner Grabbing** = capturing the introductory "banner" text a service displays on connection, revealing name/version.

### 12.2 Practical Nmap Scanning

```bash
netdiscover                                    # LAN host discovery
ifconfig                                       # check own IP/interface
nmap -p 1-65535 -v <target-ip> -sV -O          # full 5-goal scan
```
| Flag | Meaning |
|---|---|
| `-p 1-65535` | Full port range |
| `-v` | Verbose output |
| `-sV` | Service/version detection |
| `-O` | OS detection |

### 12.3 Enumeration with NSE Scripts

**NSE (Nmap Scripting Engine)** scripts extract deeper, service-specific data from open ports. Located at `/usr/share/nmap/scripts/`, named by protocol prefix (`ftp-*`, `http-*`, `smtp-*`, etc.).

```bash
nmap -p 1-65535 -v <target-ip> -sV -sC -O                 # automated default-script scan
nmap <target-ip> -p <port> -sV --script="smtp*"           # manual, targeted script scan
```
`-sC` runs Nmap's default script for each detected service automatically. Manual scans let you run *every* script matching a protocol prefix for deeper results than the single default script.

> **Workflow:** Basic scan → identify open ports/services → automated NSE pass → manual protocol-specific NSE scripts on interesting ports → document findings.

---

## 13. Metasploit Framework & Armitage (Overview)

### Module Types

| Module Type | Purpose |
|---|---|
| **Auxiliary** | Enumeration/scanning-support modules (Metasploit's equivalent of NSE scripts) |
| **Exploits** | Leverage a known, identified vulnerability/misconfiguration to gain access |
| **Payloads** | Code delivered post-exploitation to maintain access (e.g., reverse shell techniques) |

### Setup
```bash
service postgresql start      # start database backend
msfdb init                    # initialize/connect Metasploit's database
msfconsole                    # launch console
db_status                     # verify DB connection

db_nmap -p 1-65535 -v -sV -O <target-ip>    # run + store an Nmap scan inside Metasploit's DB
```

### Armitage (GUI)
```bash
apt install armitage
armitage
```
Visualizes scanned hosts as icons; right-click to view collected data or search modules matching detected software/versions.

> All Metasploit/Armitage demos in this course targeted **Metasploitable2**, Rapid7's official intentionally-vulnerable practice VM — standard across CEH/OSCP-style training worldwide. Real-world use requires explicit written authorization.

### 13.1 Named Example: The vsftpd 2.3.4 Backdoor (Metasploitable2's Canonical Teaching CVE)

Metasploitable2 intentionally ships with a very old, deliberately-backdoored version of **vsftpd 2.3.4** — this is likely the single most-referenced example in every entry-level penetration testing course, so it's worth naming explicitly for exam/certification revision purposes:

- **Vulnerability:** `CVE-2011-2523` — a since-patched, publicly-disclosed backdoor that was maliciously inserted into the vsftpd 2.3.4 source archive between 2011-06-30 and 2011-07-03. This is over a decade old, was patched immediately in the real vsftpd project, and Metasploitable2 deliberately ships the *vulnerable, pre-patch* version purely as a training target.
- **How it's typically found during a course:** an Nmap **version scan** (`-sV`) on port 21 reveals `vsftpd 2.3.4`; a student then looks up that exact version string and discovers it's a documented CVE.
- **Conceptual mechanism:** the backdoor causes the FTP service to open an unauthenticated listener on **TCP port 1524** shortly after a malformed login attempt — any tool capable of a raw TCP connection (e.g., `netcat`) to that port receives a **root-privileged shell** on the vulnerable machine.
- **Why this example is taught everywhere:** it's a clean, self-contained illustration of the full workflow (Scan → Identify version → Look up known CVE → Confirm via manual connection or a matching Metasploit exploit module) without requiring any custom exploit development — making it ideal for teaching the *process*, not just the tool.
- **Real-world takeaway:** this specific bug has been patched for well over a decade; it has essentially **zero relevance to real-world/current systems** and exists purely as a safe, static training target. The actual lesson is the *general workflow* (version fingerprinting → CVE lookup → validated exploitation only within authorized scope), which transfers to any current CVE a tester might legitimately encounter during authorized work.

---

## 14. Wireless (Wi-Fi) Security Fundamentals

### 14.1 How Wi-Fi Works — Core Terminology

| Term | Full Form / Meaning |
|---|---|
| **ESSID** | Extended Service Set ID — the network name (e.g., "MyHomeWiFi") |
| **BSSID** | Basic Service Set ID — the access point's (router's) MAC address |
| **Station** | The MAC address of a client device connected to an AP |
| **Channel** | The specific radio frequency sub-band used for communication (analogous to an FM radio frequency) |
| **Handshake File** | The packet exchange containing the (encrypted) password, generated when a client connects to an AP |
| **Beacon** | Frames an AP broadcasts announcing its presence/network activity |
| **Power (PWR)** | Signal strength indicator — a smaller (closer to 0) negative number means the device is physically closer |
| **Encryption (ENC)** | Security protocol in use: WEP, WPA, WPA2, WPA3 |

### 14.2 Connection Process (Normal Flow)
1. Router/AP broadcasts its hotspot with a name (ESSID) and has its own MAC address (BSSID).
2. A client device scans and lists nearby ESSIDs.
3. User selects a network, enters a password → device generates a **handshake packet** containing the password → sent to the AP.
4. AP validates the password → if correct, connection is established.

### 14.3 Wireless Adapter Modes
A Wi-Fi adapter (especially external USB Wi-Fi adapters with chipset support for this) can operate in two modes:

| Mode | Capability |
|---|---|
| **Managed Mode** | Default mode — normal connecting/scanning/data transfer, like everyday use |
| **Monitor Mode** | Passive listening mode — can see *all* nearby networks and *all* connected client devices (even ones you're not connected to), and can inject certain frame types onto the air |

### 14.4 Monitor Mode Capabilities (Conceptual)
When a wireless adapter is switched into Monitor Mode, it can (subject to legal authorization — testing only on networks you own or are explicitly authorized to test):
- See every nearby AP's ESSID, BSSID, channel, and encryption type.
- See which client devices (station MACs) are connected to which AP, even without knowing that AP's password.
- Capture handshake packets exchanged when a device connects (or reconnects).
- Send specially crafted management frames, including **deauthentication frames**, which force a currently-connected client to disconnect and attempt to reconnect — this is a documented protocol weakness in older WPA/WPA2 implementations (a form of denial-of-service against a single client), and is commonly used in authorized security auditing to capture a *fresh* handshake for password-strength testing.

### 14.5 Standard Toolset (Aircrack-ng Suite — Conceptual Overview)
The course references the industry-standard **aircrack-ng** suite (pre-installed on Kali) as the primary toolset for Wi-Fi security auditing:

| Tool | Role |
|---|---|
| `airmon-ng` | Switches an adapter between Managed and Monitor mode; can also stop conflicting background services that interfere with monitor mode |
| `airodump-ng` | Passively captures nearby network and client data in Monitor mode; can be filtered to a specific BSSID/channel and can log captured data (including handshakes) to a file |
| `aireplay-ng` | Sends crafted frames (e.g., deauthentication) to a target AP/client for authorized testing purposes |
| `aircrack-ng` | Analyzes a captured handshake file against a wordlist to test password strength |

### 14.6 Password / Handshake Testing (Conceptual)

**Standard aircrack-ng suite workflow (reference syntax only — for use exclusively on networks you own or are explicitly authorized in writing to test):**

```bash
# 1. Switch adapter into Monitor Mode
airmon-ng check kill          # stop background services that may interfere
airmon-ng start wlan0         # creates a monitor-mode interface, e.g. wlan0mon

# 2. Passively survey nearby networks
airodump-ng wlan0mon

# 3. Focus capture on one target network/channel, log to a file
airodump-ng --bssid <target-BSSID> --channel <ch> -w capture wlan0mon

# 4. (Authorized testing only) send a deauth to prompt a fresh handshake
aireplay-ng --deauth 10 -a <target-BSSID> -c <client-MAC> wlan0mon

# 5. Test the captured handshake against a wordlist
aircrack-ng capture-01.cap -w /usr/share/wordlists/rockyou.txt
```

- A captured handshake file contains the password in an **encrypted form** — it cannot simply be "read." Testing its strength requires a **dictionary/wordlist attack**: systematically trying candidate passwords and comparing the resulting hash against the captured handshake.
- **Built-in wordlists** (e.g., `rockyou.txt`, bundled with Kali) provide large lists of common/leaked passwords for this purpose.
- **Custom wordlist generation:** Tools like `crunch` can generate every possible combination of characters within defined length/character-set parameters — useful conceptually, but combinatorially explosive (a full alphanumeric+symbol wordlist for even modest lengths can require petabytes of storage), which is why targeted/informed wordlists are far more practical than brute-forcing every possibility.
- **Piping (`|`)** lets one command's output feed directly into another without storing an intermediate file — e.g., feeding a live wordlist generator's output directly into a cracking tool to avoid massive storage requirements.
- **Best practice taught:** A professional tester builds a **custom, targeted wordlist** informed by legitimate information gathering about the specific person/organization, since most people reuse similar password *patterns* across services rather than fully random passwords.



### 14.7 Defensive Insight: Password Pattern Reuse
A recurring theme in the course: most users either reuse the *exact same* password everywhere, or make only minor variations (e.g., `Name@123`, `Name@456`) across different accounts. If a tester/attacker discovers two or more of a person's real passwords (e.g., from breached databases or prior OSINT), the *pattern* itself becomes highly predictable, allowing a much smaller and far more effective targeted wordlist to be built for testing other accounts. **Defensive takeaway for users:** always use unique, unrelated passwords across services — password patterns are a major real-world weakness.

### 14.8 Locally Stored Wi-Fi Credentials (Windows)
Windows stores the plaintext passwords of every network it has previously connected to, retrievable via elevated Command Prompt:
```cmd
netsh wlan show profiles
netsh wlan show profile name="<SSID>" key=clear
```
This reveals the saved network's password in clear text — a relevant fact for endpoint security (physical/local access to an unlocked machine can reveal all saved Wi-Fi credentials), and also useful in password-pattern analysis (per §14.7) since a user may have several of their own past Wi-Fi passwords all stored locally.

---

## 15. Port Forwarding

**Port Forwarding** = making a service hosted on a machine with only a **Private IP** reachable from the wider internet (Public/WAN), by mapping a router's Public IP + a chosen port to an internal device's Private IP + port.

### Two Approaches Covered

1. **Router-level Port Forwarding** — configuring the home router's admin panel (`192.168.1.1` or similar) to forward incoming traffic on a chosen public port to a specific internal device's IP:port. Not all routers/ISPs support this easily (e.g., due to Carrier-Grade NAT).

2. **Tunneling Tools (used when router-level forwarding isn't available)** — third-party services that provide a public-facing address/port and automatically forward incoming connections back to your local machine:
- **Cloudflare Tunnel (`cloudflared`)** — free tool for exposing an HTTP(S) service; generates a temporary public URL that forwards to a specified local port (e.g., `localhost:80`), without requiring an account for basic use.
```bash
cloudflared tunnel --url http://localhost:80
```
- **localto.net** (and similar tunneling services) — provide TCP/UDP tunneling support beyond just HTTP, useful for protocols requiring a persistent/reliable connection (e.g., reverse-shell style connections in authorized testing scenarios), via a web dashboard and a lightweight local client.

> **Educational/legitimate use cases emphasized:** exposing a local development website for external testing/demos, remote-access scenarios, and understanding how NAT traversal works — these are the same underlying techniques used by legitimate remote-access and webhook-testing tools (e.g., ngrok).

---

## 16. CCTV / IoT Device Security Concepts

### 16.1 Types of CCTV Cameras

| Type | Description |
|---|---|
| **Analog** | Wired (coax) connection to a DVR; low security, minimal encryption |
| **IP Camera** | Each camera has its own IP address, communicates over standard IP networking, often internet-connected (increases both remote-access convenience and attack surface) |
| **Wireless** | Connects via local Wi-Fi; vulnerable to standard Wi-Fi attacks (sniffing, MITM) if on a shared network |
| **PTZ (Pan-Tilt-Zoom)** | Supports full remote control, increasing both functionality and remote attack surface |

### 16.2 Typical CCTV Network Architecture
Camera(s) → DVR (Digital Video Recorder, stores/manages footage) → Router (assigns private IPs to camera/DVR, provides internet access) → optionally exposed to the internet via **Port Forwarding** for remote viewing.

### 16.3 Common Protocols & Their Default Ports (Referenced for Understanding Attack Surface)

| Protocol | Default Port | Purpose |
|---|---|---|
| **RTSP** | 554 | Real-Time Streaming Protocol — live video streaming |
| **HTTP/HTTPS** | 80 / 443 | Web-based admin login panel |
| **ONVIF** | 8000, 8080, 1899 | Open standard enabling interoperability between different camera brands |
| **FTP** | 20/21 | File transfer (recordings) |

### 16.4 Common Weaknesses Discussed (Conceptual, for Defensive Awareness)
- **Default/weak credentials** — many CCTV/DVR systems ship with well-known default admin usernames/passwords that are frequently never changed.
- **Unauthenticated or weakly authenticated RTSP streams** — some devices expose live video streams with no or trivial authentication, and once port-forwarded, the stream URL becomes reachable from anywhere on the internet.
- **Lack of encryption / cleartext protocols** — RTSP/ONVIF traffic is often unencrypted, making it susceptible to interception on a shared network.
- **Outdated firmware** — unpatched known CVEs in camera/DVR firmware.
- **Human error** — the single largest contributing factor to real-world compromise across all device types (e.g., reusing default credentials, oversharing network access).

> **Defensive recommendations implied throughout:** change all default credentials immediately, disable unused protocols/ports (e.g., turn off ONVIF/RTSP remote exposure if not needed), keep firmware updated, avoid unnecessary port forwarding of camera systems directly to the internet (use a VPN instead), and segment IoT devices onto a separate VLAN/network from primary devices.

### 16.5 Standard IoT/CCTV Assessment Tool Chain (Conceptual — for authorized audits of devices you own/administer only)

A typical authorized CCTV/DVR security assessment follows the same generic phases as any pentest, applied to this device class:

1. **Discovery/Scanning:** `nmap -p 1-65535 -sV <target-ip>` to find the camera/DVR's IP and identify which of the ports in §16.3 are open, and which software/firmware version is running.
2. **Web-panel credential testing (authorized only):** if an HTTP(S) admin login page is found (port 80/443), a tool like **Burp Suite's Intruder** module can be used to systematically test a curated credential list against the login form, specifically checking whether the device still uses **default/never-changed vendor credentials** — this is checked first because it's by far the most common real-world root cause of IoT compromise, rather than a sophisticated exploit.
3. **Stream verification:** if valid credentials are found (or if RTSP requires no authentication at all — itself a critical finding to report), a standard media player like **VLC** can open the RTSP stream URL (`rtsp://<ip>:554/...`) to *confirm* the finding for the audit report.
4. **Reporting:** document exactly which weakness was found (default creds / unauthenticated stream / outdated firmware / unnecessary internet exposure) and the specific remediation (change credentials, disable remote RTSP exposure, update firmware, restrict access via VPN/firewall).

> This is presented at a conceptual/procedural level intentionally — the actual value of this module is understanding *why* these are the most common real-world IoT weaknesses (so you can defend against them), not a copy-paste attack script. Any hands-on testing must be limited to hardware you personally own or have explicit written authorization to assess.

---

## 17. Being Anonymous — Web Layers & Tor Concepts

### 17.1 URL Structure
`protocol://domain:port/path/file?parameter=value`
Example: `http://cp.com:81/auth/login.asp?uid=129`

### 17.2 Layers of the Internet

| Layer | Definition | Approx. Scope |
|---|---|---|
| **Surface Web** | Content indexed by search engines and easily discoverable | ~5% of all internet data |
| **Deep Web** | Content *not* indexed by search engines but still reachable if you know the exact URL (e.g., private portals, unlisted pages, most private/internal company systems) | ~95% of all internet data |
| **Dark Web** | A *subset of the Deep Web* specifically hosting sites associated with illegal/unlisted activity, reachable only via specialized anonymity software | Small subset of the Deep Web |
| **Hidden Web** | Content whose URL is known only to a very small, specific set of people (e.g., a private site shared between 2-3 friends) — not indexed and not widely known | Varies |

> Key concept: these categories are defined by **discoverability/indexing**, not by a fixed "location" on the internet. The same site can move between categories (e.g., a hidden blog that gets linked publicly becomes part of the Surface Web; if that content is then found to host illegal material and delisted, it becomes part of the Dark Web).

### 17.3 How Tor Provides Anonymity (Conceptual)
1. **Requires a specialized browser** (Tor Browser / "Onion Browser") to access `.onion` addresses.
2. **`.onion` addresses are not human-memorable** and are **dynamic** — they can change at the site owner's discretion, making long-term tracking/indexing difficult.
3. **Multi-hop relay routing** — traffic passes through multiple intermediate relay nodes before reaching the destination server, and no single relay has both the origin and full destination information, making it very difficult (though not theoretically impossible) to trace the original source back through the chain.
4. **Fundamental limitation:** despite the anonymity layer, any interactive web application still ultimately processes user input at the server — meaning **web application vulnerabilities** (e.g., input handling flaws) can still, in principle, be used in *authorized* security research to identify a misconfigured server, illustrating that anonymity networks do not make the underlying web server itself invulnerable to standard web security testing.

### 17.4 Self-Hosting a Basic Site via Tor (Educational Demo Concept)
The course demonstrated, purely for educational understanding of how the Tor hidden-service mechanism works:
```bash
apt install tor
service apache2 start          # host a basic page locally on port 80
tor                            # build a Tor circuit
```
Then editing `/etc/tor/torrc` to uncomment/set:
```
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
```
After restarting the Tor circuit, an auto-generated `.onion` address (found in the hidden service directory's `hostname` file) becomes reachable via Tor Browser for as long as the local Tor process keeps running — demonstrating that a hidden service is really just a locally-hosted server made reachable through Tor's relay network, and that it disappears the moment the local process stops.

---

## 18. Android Environment Concepts

### 18.1 Android OS Basics
- **Android is an operating system**, not a device — built on top of the **Linux kernel**, following the same layered architecture concepts covered in the Linux fundamentals section (kernel → hardware abstraction layer → native libraries → Android Runtime → Application Framework → Apps).
- **AOSP (Android Open Source Project)** — Android is open source, meaning its source code is publicly available, which is why many different Android forks/skins/versions exist across manufacturers.

### 18.2 APK Structure
An **APK (Android Package)** file is fundamentally a **compressed ZIP archive** (renaming a `.apk` to `.zip` and extracting it demonstrates this) containing:
- **Compiled source code** (as DEX bytecode — not human-readable without decompiling)
- **`AndroidManifest.xml`** — the single most important file, declaring: the app's unique package name, version info, declared permissions, and all app components
- **`resources/`** — images, layouts, strings, and other assets
- **Meta-data / signature files** — cryptographic signing info verifying the APK's authenticity/integrity

### 18.3 AndroidManifest.xml Key Contents
| Field | Purpose |
|---|---|
| Unique package name | Identifies the app (e.g., `com.vendor.appname`) |
| Version info | Which SDK/build tools/library versions were used |
| Activities | Declares each screen/page of the app |
| Permissions | Declares what device resources the app requests (camera, storage, location, etc.) |
| Shared User ID | For apps needing to share data/resources with each other |
| Install location | Where app data/files are stored on the device |
| UI info | Launcher icon and other display assets |
| Third-party libraries | Declares external SDKs (e.g., Google Maps) used by the app |

### 18.4 Four Major Android App Components
| Component | Role |
|---|---|
| **Intents** | Internal messaging system — lets components (or apps) communicate/pass data |
| **Broadcast Receivers** | Listen passively for system-wide broadcast events (e.g., an incoming call) and react (e.g., Truecaller showing caller ID before the call connects) |
| **Services** | Manage background operations (Bluetooth, Wi-Fi toggling, background music, sync tasks) |
| **Activities** | Individual screens/pages of an app, each handling a specific function (Login screen, Settings screen, etc.) |

### 18.5 App Storage Locations
- **Pre-installed (system) apps:** `/data/system/<appname>/`
- **User-installed apps:** `/data/data/<appname>/`

### 18.6 Reverse Engineering / Decompiling APKs (Conceptual)
The course demonstrated the general workflow used to inspect (not necessarily attack) an app's inner workings — a standard technique in **mobile app penetration testing / secure code review**:
1. `.apk` → renamed to `.zip` → extracted → reveals raw `classes.dex` (compiled bytecode, not readable).
2. **`dex2jar`** tools convert `classes.dex` into a readable `.jar` file.
3. **`jd-gui`** or similar Java decompiler GUIs let a reviewer open that `.jar` and read the (approximate) original Java source code — useful for security review to spot hardcoded credentials, insecure API endpoints, or logic flaws.
4. **All-in-one tools** (e.g., "Bytecode Viewer") automate steps 1–3 in a single drag-and-drop interface.
> This is standard practice in mobile app security assessments and bug bounty programs — reviewing an app's own decompiled code for security issues, with proper authorization/scope.

### 18.7 Mobile Security Auditing Tools
- **MVT (Mobile Verification Toolkit)** — an open-source forensic tool that scans a connected Android/iOS device for **Indicators of Compromise (IOCs)** — known signatures/patterns left behind by known spyware/malware families. Requires ADB (Android Debug Bridge) connectivity between the device and the analysis machine, with USB debugging enabled in Developer Options.
- **MobSF (Mobile Security Framework)** — a web-based static/dynamic analysis tool for auditing APK/IPA files, producing a vulnerability report covering insecure API usage, hardcoded secrets, weak cryptography practices, extracted server IPs/URLs/emails referenced in the app, and overall security scoring — used for both offensive bug-hunting and defensive app security review.

### 18.8 Kali NetHunter on Android (Termux-based setup, conceptual)
The course demonstrated installing **Termux** (a genuine Linux terminal emulator for Android, sourced from its official GitHub release rather than the outdated Play Store version) and using it to bootstrap a **Kali NetHunter** rootfs environment on a non-rooted phone — illustrating that a full Linux distribution with standard security tools can run inside a sandboxed environment on Android without rooting the device. A VNC server (`kex`/`nhkex`) is then used to access the environment graphically. This is presented purely as a **learning/practice convenience** (running Kali tools on a phone for study), not as a device-compromise technique — it is your own personal device being configured as a portable lab.

---

## 19. Lab Setup Notes

### Recommended Lab Environment
- **Virtualization software:** VirtualBox or VMware Workstation.
- **Attacker machine:** Kali Linux (download the pre-built VM image from the official Kali website for the fastest setup, rather than a full ISO installation).
- **Practice/target machine:** Metasploitable2 (also from Rapid7's official distribution) — a deliberately vulnerable Linux VM built for safe, legal practice.

### Networking Setup Between VMs
- Both VMs should be configured with matching **Network Adapter** settings in the hypervisor (e.g., a shared "Host-only Adapter" or "NAT + Host-only" combo) so they can reach each other on the same private subnet.
- Default Kali VM login credentials on the pre-built image: `kali` / `kali` (always change these in any real deployment).

### Android Lab
- Android-x86 ISO can be installed as a VM inside VirtualBox for practicing mobile app security concepts, with adjusted display/graphics controller settings if the default GUI doesn't render.
- **Termux** provides a genuine terminal emulator for a physical Android device, useful for learning command-line tools on the go.

---

## 20. Miscellaneous / Recurring Themes

- **Cyber Kill Chain-style instructional pattern** used throughout the course: **Information Gathering → Scanning → Enumeration → Vulnerability Identification → (Authorized) Exploitation → Reporting/Remediation.**
- **Ethics/authorization emphasis:** every practical demo in the transcript is explicitly performed either against (a) the instructor's own lab machines, (b) Metasploitable2 (a purpose-built vulnerable practice VM), or (c) the instructor's own personal devices/accounts — with repeated reminders that testing any system without explicit written permission is illegal.
- **AI-assisted troubleshooting:** the course repeatedly demonstrates using AI chat tools to help debug driver/dependency errors, generate small automation scripts (e.g., a script to enumerate one's *own* saved Wi-Fi profiles' passwords for password-pattern analysis, or a script to help install a Wi-Fi adapter driver), and troubleshoot Linux package errors — modeling a practical "AI as a research assistant" workflow for IT/security troubleshooting.

---

## Appendix A: Glossary / Acronym Quick-Reference

| Acronym | Full Form |
|---|---|
| ISP | Internet Service Provider |
| IP | Internet Protocol |
| MAC | Media Access Control |
| LAN / MAN / WAN | Local / Metropolitan / Wide Area Network |
| NAT | Network Address Translation |
| ARP | Address Resolution Protocol |
| DHCP | Dynamic Host Configuration Protocol |
| TCP / UDP | Transmission Control Protocol / User Datagram Protocol |
| OSI | Open Systems Interconnection (model) |
| SYN / ACK / FIN / RST / PSH / URG | TCP Flags: Synchronize / Acknowledgment / Finish / Reset / Push / Urgent |
| DNS | Domain Name System |
| A / AAAA / CNAME / MX / TXT / NS / SOA | DNS record types |
| SPF | Sender Policy Framework (anti-spoofing) |
| OSINT | Open-Source Intelligence |
| NSE | Nmap Scripting Engine |
| MSF | Metasploit Framework |
| CVE | Common Vulnerabilities and Exposures |
| ESSID / BSSID | Extended/Basic Service Set Identifier (Wi-Fi network name / AP's MAC) |
| WPA / WPA2 / WPA3 | Wi-Fi Protected Access (versions 1–3) |
| RTSP | Real-Time Streaming Protocol |
| ONVIF | Open Network Video Interface Forum (CCTV interoperability standard) |
| DVR | Digital Video Recorder |
| APK | Android Package (Kit) |
| AOSP | Android Open Source Project |
| ADB | Android Debug Bridge |
| IOC | Indicator of Compromise |
| MVT | Mobile Verification Toolkit |
| MobSF | Mobile Security Framework |
| GRC | Governance, Risk & Compliance |
| SOC | Security Operations Center |
| IAM | Identity & Access Management |
| VAPT | Vulnerability Assessment and Penetration Testing |

---

## Appendix B: A Note on How These Notes Were Scoped

These notes summarize and organize the *concepts, terminology, and general workflows* taught in the course transcript in a structured, revision-friendly format. A few intentional scoping choices worth knowing about:

- **Conceptual over "copy-paste" framing:** Where the original class walked through a live, step-by-step demo against a specific practice machine (e.g., Metasploitable2) or the instructor's own devices, these notes describe the **tool, the workflow stage, and why it matters** rather than reproducing an exact, chained command sequence as a ready-to-run script — the goal is understanding *why* a technique works (for both offense and defense), which is what actually transfers to certification exams and real authorized engagements.
- **Every offensive technique is paired with authorization context and, where relevant, a defensive takeaway** — this mirrors how the material is actually assessed in certifications like CEH/Security+/OSCP, which test judgment and process as much as tool syntax.
- If your syllabus requires the **exact verbatim commands** from a specific lecture (e.g., for a lab report), it's best to re-check that portion directly against your original recording/transcript, since exact flag ordering and file paths can matter for grading.

---



## Summary / Key Takeaways for Exam Revision

1. **Networking core:** IP (logical addressing), MAC (physical addressing), Ports (communication channels), Protocols (rules) — all work together so devices can communicate.
2. **Public vs Private IP** and the golden rule that they never talk directly (NAT bridges them).
3. **OSI (theory, 7 layers) vs TCP/IP (practice, 4 layers)** — know the layer-to-protocol mapping.
4. **TCP 3-way handshake (SYN → SYN/ACK → ACK)** vs **4-step termination (FIN → ACK → FIN → ACK)**.
5. **DNS hierarchy:** Resolver → Root → TLD → Name Server → Zone File → A Record → IP.
6. **Linux is a kernel**, GNU/Linux is the full OS; `/etc` holds configs, permissions are `rwx`.
7. **Recon → Scanning → Enumeration → Exploitation** — always in that order, always with explicit authorization.
8. **Nmap's 5 scanning goals** and **NSE scripts** for enumeration are core practical skills.
9. **Wi-Fi security:** ESSID/BSSID/handshake concepts, Monitor vs Managed mode, and why password *pattern* reuse is the biggest real-world weakness.
10. **IoT/CCTV security:** default credentials and unnecessary internet exposure (via port forwarding) are the most common real-world weaknesses.
11. **Tor/anonymity:** anonymity ≠ invulnerability — the underlying web server can still have standard web vulnerabilities.
12. **Mobile security:** APKs are ZIP archives; `AndroidManifest.xml` is the most important file to review; MVT/MobSF are standard auditing tools.

---

*End of notes. Recommended next step: cross-reference these notes against your syllabus/certification objectives (e.g., CEH/Security+) and supplement with hands-on lab practice only in authorized environments (e.g., Metasploitable2, TryHackMe, HackTheBox, or your own personal devices/network).*
