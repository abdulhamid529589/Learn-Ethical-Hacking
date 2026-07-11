# Cyber Security & Ethical Hacking — Course Notes

## https://youtu.be/Y9n2W1ypT1U

> Compiled study notes based on the "Cyber Pathshala" beginner roadmap and foundational lecture series. Organized for quick revision.

---

## 📑 Index / Table of Contents

- [1. Roadmap to Ethical Hacking & Cyber Security](#1-roadmap-to-ethical-hacking--cyber-security)
  - [1.1 What to Learn (in order)](#11-what-to-learn-in-order)
  - [1.2 Qualifications Path](#12-qualifications-path)
  - [1.3 Certifications by Level](#13-certifications-by-level)
  - [1.4 Earning / Career Paths (after Ethical Hacking)](#14-earning--career-paths-after-ethical-hacking)
  - [1.5 Learning Sources](#15-learning-sources)
- [2. Types of Hackers](#2-types-of-hackers)
- [3. Networking Fundamentals](#3-networking-fundamentals)
  - [3.1 What Is Computer Networking?](#31-what-is-computer-networking)
  - [3.2 Types of Networks (by range)](#32-types-of-networks-by-range)
  - [3.3 Key Networking Entities](#33-key-networking-entities)
- [4. IP Addressing](#4-ip-addressing)
  - [4.1 What Is an IP Address?](#41-what-is-an-ip-address)
  - [4.2 Conceptual Breakdown of an IP (for tracing)](#42-conceptual-breakdown-of-an-ip-for-tracing)
  - [4.3 Types of IP Addresses](#43-types-of-ip-addresses)
  - [4.4 IP Communication Rule](#44-ip-communication-rule)
- [5. MAC Addresses](#5-mac-addresses)
- [6. Ports](#6-ports)
  - [6.1 Definition](#61-definition)
  - [6.2 Two Core Rules](#62-two-core-rules)
  - [6.3 Port Categories](#63-port-categories)
  - [6.4 Basic Port-Status Scanning (concept)](#64-basic-port-status-scanning-concept)
- [7. Protocols & Networking Models](#7-protocols--networking-models)
  - [7.1 What Is a Protocol?](#71-what-is-a-protocol)
  - [7.2 OSI Model (7 Layers, read bottom-up)](#72-osi-model-7-layers-read-bottom-up)
  - [7.3 TCP/IP Model (4 Layers)](#73-tcpip-model-4-layers--practicalcompressed-version-of-osi)
  - [7.4 TCP Flags](#74-tcp-flags)
  - [7.5 TCP Three-Way Handshake](#75-tcp-three-way-handshake-connection-establishment)
  - [7.6 TCP Session Termination](#76-tcp-session-termination-4-step-not-called-a-handshake)
  - [7.7 TCP vs UDP](#77-tcp-vs-udp)
- [8. How the Internet Works: Servers, Domains & DNS](#8-how-the-internet-works-servers-domains--dns)
  - [8.1 Servers](#81-servers)
  - [8.2 Domains & Subdomains](#82-domains--subdomains)
  - [8.3 DNS Records](#83-dns-records-configured-via-a-domain-registrars-control-panel)
  - [8.4 DNS Resolution Process (simplified)](#84-dns-resolution-process-simplified)
  - [8.5 Zone Files](#85-zone-files)
- [9. Home Networking: ISPs, Routers & Address Assignment](#9-home-networking-isps-routers--address-assignment)
- [10. Linux Fundamentals](#10-linux-fundamentals)
  - [10.1 Kernel vs OS](#101-kernel-vs-os)
  - [10.2 Open Source](#102-open-source)
  - [10.3 Key Properties of Linux](#103-key-properties-of-linux)
  - [10.4 Linux File System Layout](#104-linux-file-system-layout)
  - [10.5 Essential Commands](#105-essential-commands)
- [11. Information Gathering (Footprinting & Reconnaissance)](#11-information-gathering-footprinting--reconnaissance)
  - [11.1 Why It Matters](#111-why-it-matters)
  - [11.2 Categories of Targets](#112-categories-of-targets)
  - [11.3 Active vs Passive Information Gathering](#113-active-vs-passive-information-gathering)
  - [11.4 Search Engine Techniques](#114-search-engine-techniques)
  - [11.5 Advanced Search Operators ("Google Dorking" basics)](#115-advanced-search-operators-google-dorking-basics)
  - [11.6 OSINT Frameworks & Tools](#116-osint-frameworks--tools-mentioned-conceptually)
- [12. Network Scanning & Enumeration (with Nmap)](#12-network-scanning--enumeration-with-nmap)
  - [12.1 The Five Goals of Network Scanning](#121-the-five-goals-of-network-scanning)
  - [12.2 Practical Workflow Walkthrough](#122-practical-workflow-walkthrough)
  - [12.3 Enumeration — Going Deeper Than Scanning](#123-enumeration--going-deeper-than-scanning)
  - [12.4 Nmap Scripting Engine (NSE)](#124-nmap-scripting-engine-nse)
  - [12.5 Example Finding: Anonymous FTP Login](#125-example-finding-anonymous-ftp-login)
  - [12.6 Summary: Why Enumeration Matters](#126-summary-why-enumeration-matters)
- [13. Practice Lab Environment (Concepts)](#13-practice-lab-environment-concepts)
  - [13.0 Setting Up Metasploitable2 (Step-by-Step)](#130-setting-up-metasploitable2-step-by-step)
  - [13.1 Metasploit Framework (conceptual terms)](#131-metasploit-framework-conceptual-terms)
- [14. Key Takeaways / Ethical Reminders](#14-key-takeaways--ethical-reminders)

---

## 1. Roadmap to Ethical Hacking & Cyber Security

The course is structured around four pillars: **Learn → Certificates → Earn → Sources**

### 1.1 What to Learn (in order)

1. **Networking basics** – IP addressing, protocols, OSI/TCP-IP models (not a full networking/CCNA-level course, just what's relevant to security).
2. **Linux OS & commands** – file system, terminal usage, command-line operations.
3. **Ethical Hacking fundamentals** – footprinting & reconnaissance, scanning, enumeration, vulnerability analysis, exploitation, and securing systems.
4. **Cyber Security domain selection** – after ethical hacking, pick a specialization: network, web, cloud, OS, AI, IoT, mobile, hardware, quantum, etc.

### 1.2 Qualifications Path

| Stage                           | Recommended Path                                                   |
| ------------------------------- | ------------------------------------------------------------------ |
| Before 10th grade               | Focus purely on skills (networking, Linux, ethical hacking basics) |
| 11th–12th                       | Science + Math stream → enables engineering routes                 |
| After 12th (Science)            | B.Tech / M.Tech in Cyber Security                                  |
| After 12th (Non-science)        | BCA / MCA with Cyber Security specialization                       |
| Non-IT Bachelor's               | Master's with a technical Cyber Security specialization            |
| Can't continue formal education | Focus on skills + certifications + diploma programs                |

### 1.3 Certifications by Level

- **Beginner:** eJPT, CEH, Security+, Google Security Certificate
- **Advanced:** OSCP, eCPPT, PenTest+, eWPT
- **Expert:** OSEP, CRTP, CRTE, GPEN (referred to as "GPON" in transcript)

> Certifications should align with your chosen domain (e.g., don't pursue system-hacking certs if your domain is web security).

### 1.4 Earning / Career Paths (after Ethical Hacking)

**Categories of jobs:**

1. **Offensive Security (Red Teaming)** – Junior Pentester, Ethical Hacker, Vulnerability Assessment Analyst, Web App Security Tester, Bug Hunter, Red Team Intern
2. **Defensive Security (Blue Teaming)** – SOC Analyst L1, Security Analyst, Threat Monitoring Analyst, Incident Response Intern, SIEM Analyst
3. **Training & Awareness** – Ethical Hacking Trainer, Cyber Security Instructor, Lab Assistant, Online Instructor
4. **GRC (Governance, Risk & Compliance)** – IT Security Auditor (Junior), Compliance Analyst, Risk Analyst
5. **General IT & Security Support** – IT Support Technician, Network Support Engineer, System Administrator, SOC Intern
6. **Bonus – Freelance/Self-Employment** – Bug bounty (recommended only after gaining experience, not immediately), freelance pentesting, teaching/content creation, security blogging

> Tip: Use keywords like "Junior," "Intern," or "Helper" when job searching as a fresher to avoid roles requiring years of experience.

### 1.5 Learning Sources

- **Books** – recommended starting book: _The Hacker's Playbook 3_
- **Pre-recorded courses**
- **Paid bootcamps / mentorship programs**
- **Blogs, forums, and AI tools** for quick doubt resolution
- **Job platforms:** LinkedIn, Indeed, niche cybersecurity job boards, specialized consultancy firms

> Note: The original material also described methods for obtaining pirated copies of paid books/courses. That content is intentionally omitted here, as it facilitates copyright infringement — please use legitimate sources (official publishers, licensed platforms, free/open resources, or your university library) instead.

---

## 2. Types of Hackers

| Type                           | Intent              | Legality        | Permission                        | Motivation                                       |
| ------------------------------ | ------------------- | --------------- | --------------------------------- | ------------------------------------------------ |
| **White Hat (Ethical Hacker)** | Legal, constructive | Legal           | Has authorization                 | Securing systems, career growth, bug bounties    |
| **Black Hat**                  | Malicious           | Illegal         | No authorization                  | Financial gain, espionage, revenge, disruption   |
| **Grey Hat**                   | Ambiguous / mixed   | Legal gray area | Sometimes with, sometimes without | Fame, public interest, occasional financial gain |

- All three types can have equally deep technical knowledge — the differentiator is how that knowledge is used.
- Example stats mentioned: global cybercrime cost projected around $10.5 trillion by 2025; Google reportedly paid ~$12 million in bug bounties in 2022.

---

## 3. Networking Fundamentals

### 3.1 What Is Computer Networking?

The process by which two or more devices communicate and share data, files, hardware, or software. Key considerations: **information preservation** (backups via shared copies) and **security** of transactions.

### 3.2 Types of Networks (by range)

| Type    | Full Form                 | Range                                             | Example           |
| ------- | ------------------------- | ------------------------------------------------- | ----------------- |
| **LAN** | Local Area Network        | ~15–50 devices, small physical area (e.g., ~100m) | A computer lab    |
| **MAN** | Metropolitan Area Network | Multiple LANs connected across a city/campus      | University campus |
| **WAN** | Wide Area Network         | Global, connects multiple MANs/LANs               | The Internet      |

### 3.3 Key Networking Entities

- **ISP (Internet Service Provider):** Company providing internet access (e.g., regional telecom providers).
- **MAC Address:** Hardware/physical address assigned to a network device at manufacture time.
- **Port:** A communication "route" in/out of a device (1–65535 total).
- **Protocol:** Rules governing safe/secure data transmission.
- **IP Address:** Unique address identifying a device for network communication.

---

## 4. IP Addressing

### 4.1 What Is an IP Address?

A unique identifier for a device on a network, enabling communication across networks/the internet. IPv4 format: four octets (e.g., `17.172.224.47`), each 8 bits (1 byte), total 32 bits.

### 4.2 Conceptual Breakdown of an IP (for tracing)

1. **Purpose:** networking/communication
2. **Range of use:** worldwide
3. **Hierarchical breakdown:** Country → State → City/ISP → assigned to a specific user by the ISP at a point in time

This is conceptually how law enforcement/ISPs can trace an IP address back to an individual account holder using ISP logs for a given time window.

### 4.3 Types of IP Addresses

| Category    | Types                     | Description                                                                                                                  |
| ----------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Scope       | **Public** vs **Private** | Public = usable across the internet (WAN); Private = usable only within a LAN                                                |
| Persistence | **Static** vs **Dynamic** | Static = fixed, used by servers; Dynamic = reassigned as devices connect/disconnect, conserves the limited IPv4 address pool |

**Analogy:** Private IP ≈ a nickname used only among close contacts; Public IP ≈ your official/legal name used everywhere.

**Why dynamic IPs exist:** IPv4 only supports ~4.3 billion addresses, but billions of people use the internet — not everyone is online simultaneously, so ISPs recycle unused IPs.

### 4.4 IP Communication Rule

- Private IP can only talk to Private IP (within the same LAN)
- Public IP can only talk to Public IP (across the WAN)
- Private and Public IPs **cannot** communicate directly with each other

---

## 5. MAC Addresses

- **Full form:** Media Access Control Address (also called hardware/physical address)
- **Structure:** 6 pairs (48 bits / 6 bytes total), using hexadecimal characters (0–9, A–F)
- **Two parts:**
  - First 3 pairs = **OUI (Organizationally Unique Identifier)** → identifies the vendor/manufacturer (e.g., Intel, Samsung)
  - Last 3 pairs = **NIC (Network Interface Controller)** portion → uniquely identifies the specific device
- **Checking your MAC address:**
  - Windows: `getmac` (in Command Prompt)
  - Linux/Mac: `ifconfig` (in Terminal)
- **MAC lookup tools** (e.g., online "MAC lookup" services) can reveal the vendor and approximate manufacture date from the OUI portion.

---

## 6. Ports

### 6.1 Definition

A port is a communication "doorway" (1–65535) that allows data in/out of a device.

### 6.2 Two Core Rules

1. **The port used must be open** (a "listener" must be present) or the packet is dropped/returned.
2. **Only one process can use a given port at a time** — if a port is busy, a second application must wait or use a different port.

### 6.3 Port Categories

| Range       | Name                  | Description                                                                     |
| ----------- | --------------------- | ------------------------------------------------------------------------------- |
| 0–1023      | Well-Known Ports      | Most common protocols/services (HTTP=80, HTTPS=443, FTP=20/21, SSH=22, SMTP=25) |
| 1024–49151  | Registered Ports      | Used for specific application-level tasks                                       |
| 49152–65535 | Dynamic/Private Ports | Frequently reassigned, less reliable for persistent connections                 |

> Security relevance: malware/backdoor authors often prefer registered/dynamic ports over well-known ports to avoid conflicts with existing services (e.g., a payload trying to exfiltrate data over port 80 may fail if a web server is already using it).

### 6.4 Basic Port-Status Scanning (concept)

Using a tool like **Nmap**, you can:

1. Confirm host is reachable (ping/host discovery)
2. Scan a range of ports to see which are open/closed/filtered
3. Identify which service/protocol is using an open port

---

## 7. Protocols & Networking Models

### 7.1 What Is a Protocol?

A protocol is a set of rules governing how data is transmitted safely and reliably between devices — analogous to traffic rules ensuring safe travel.

### 7.2 OSI Model (7 Layers, read bottom-up)

| Layer           | Function                                 | Example Protocols           |
| --------------- | ---------------------------------------- | --------------------------- |
| 7. Application  | User-facing services                     | HTTP, HTTPS, FTP, SMTP, DNS |
| 6. Presentation | Data formatting, compression, encryption | SSL/TLS                     |
| 5. Session      | Establishing/maintaining sessions        | NetBIOS, RPC, SOCKS         |
| 4. Transport    | End-to-end connection                    | TCP, UDP                    |
| 3. Network      | Logical addressing/routing               | IP, ICMP, ARP, DHCP         |
| 2. Data Link    | Framing data for hardware                | Ethernet, ARP               |
| 1. Physical     | Signal transmission over hardware        | Wi-Fi, cabling standards    |

### 7.3 TCP/IP Model (4 Layers – practical/compressed version of OSI)

| Layer          | Combines OSI Layers          | Key Protocols                     |
| -------------- | ---------------------------- | --------------------------------- |
| Application    | App + Presentation + Session | HTTP, FTP, SMTP, DNS, SSH, Telnet |
| Transport      | Transport                    | TCP, UDP                          |
| Internet       | Network                      | IP, ICMP, ARP, DHCP               |
| Network Access | Data Link + Physical         | Ethernet, Wi-Fi, ADSL             |

> Security note: most attacks target the **Application layer** (direct user interaction = most vulnerabilities); session hijacking targets the Session layer; reverse/bind shells operate at the Transport layer.

### 7.4 TCP Flags

| Flag    | Meaning         | Function                                                                       |
| ------- | --------------- | ------------------------------------------------------------------------------ |
| **URG** | Urgent          | High-priority packet, processed immediately                                    |
| **PSH** | Push            | Send buffered data immediately, don't wait                                     |
| **FIN** | Finish          | Request to terminate the connection                                            |
| **ACK** | Acknowledgement | Confirms receipt of a packet                                                   |
| **RST** | Reset           | Forcefully breaks and can re-establish a connection (used to resolve glitches) |
| **SYN** | Synchronize     | Initiates a new connection                                                     |

### 7.5 TCP Three-Way Handshake (Connection Establishment)

1. Client sends **SYN** (requesting connection on a specific port)
2. Server replies **SYN-ACK** (acknowledges + agrees to connect)
3. Client sends **ACK** (connection established)

Sequence numbers (SEQ) ensure packets are reassembled in the correct order.

### 7.6 TCP Session Termination (4-step, NOT called a "handshake")

1. Client sends **FIN** (wants to close connection)
2. Server sends **ACK** (acknowledges the request)
3. Server sends its own **FIN** once its data transfer is complete
4. Client sends final **ACK** — connection closed

### 7.7 TCP vs UDP

| Feature     | TCP                              | UDP                                 |
| ----------- | -------------------------------- | ----------------------------------- |
| Connection  | Connection-oriented (handshake)  | Connectionless                      |
| Speed       | Slower (more overhead)           | Faster                              |
| Reliability | High (guarantees delivery/order) | Low (no guarantees)                 |
| Use case    | Secure, ordered data transfer    | Speed-sensitive, less critical data |

---

## 8. How the Internet Works: Servers, Domains & DNS

### 8.1 Servers

A server is simply a machine with its own **static public IP address**, providing a service (hosting a website, files, etc.) accessible over the WAN.

### 8.2 Domains & Subdomains

- A **domain** (e.g., `example.com`) is essentially a human-friendly name mapped to a server's IP address (since remembering IPs is impractical).
- **Subdomains** (e.g., `blog.example.com`) let you create unlimited named branches under one domain; the base domain extension stays the same.
- **Extensions** (TLDs) include generic (`.com`, `.org`) and country-specific (`.in`, `.jp`, `.ru`) versions.

### 8.3 DNS Records (configured via a domain registrar's control panel)

| Record    | Purpose                                                               |
| --------- | --------------------------------------------------------------------- |
| **A**     | Maps domain → IPv4 address                                            |
| **AAAA**  | Maps domain → IPv6 address                                            |
| **CNAME** | Alias/redirection to another domain or subdomain                      |
| **MX**    | Specifies mail server(s) for the domain                               |
| **TXT**   | Free-text field; commonly used for SPF/DKIM to prevent email spoofing |
| **NS**    | Specifies authoritative name servers for the domain                   |
| **SOA**   | Administrative/authority contact info for the domain                  |

### 8.4 DNS Resolution Process (simplified)

1. User requests a domain (e.g., types it in a browser).
2. **Local/global resolvers** check their cache first.
3. If not cached, the request goes to a **root server**.
4. The root server directs the query to the appropriate **TLD server** (e.g., handling `.com`).
5. The TLD server provides the domain's **name servers**.
6. The name server checks the **zone file** (a public record containing the domain's DNS records) and returns the IP address.
7. The result is cached locally for future requests.

### 8.5 Zone Files

A publicly accessible file (distinct from the private domain-management login) containing a domain's DNS records (A, NS, MX, TXT, etc.), refreshed periodically. This is what DNS infrastructure actually queries.

---

## 9. Home Networking: ISPs, Routers & Address Assignment

- **ISP** reserves a range of public IPs for its customers; your router/broadband gets one static/dynamic public IP from this pool.
- **Router** reserves its own range of private IPs (e.g., `192.168.1.0`–`192.168.1.255`) for devices on the home LAN, keeping the first address (e.g., `192.168.1.1`) for itself — known as the **default gateway** / subnet address.
- **DHCP (Dynamic Host Configuration Protocol):** automatically assigns private IPs to connecting devices (vs. manual configuration by a network admin).
- **Subnetting:** divides a large network into smaller segments to improve manageability, performance, and security (reduces exposure to broadcast storms/DoS-type issues).
- **NAT (Network Address Translation):** allows devices with private IPs to communicate over the public internet by translating between private and public IP addresses at the router.
- **ARP (Address Resolution Protocol):** uses broadcast messages to map IP addresses to MAC addresses and maintains the router's IP-to-MAC address table; also detects when a device disconnects (frees its IP for reuse).

---

## 10. Linux Fundamentals

### 10.1 Kernel vs OS

- **Kernel:** the core layer that communicates directly with hardware (CPU, RAM, disk).
- **Terminal/Shell:** software (e.g., Bash, Zsh) that lets users send commands to the kernel in a language it understands.
- **Linux** itself is technically a _kernel_; **GNU/Linux** is the combination of the GNU tools and the Linux kernel forming a full OS.

### 10.2 Open Source

Source code is made publicly available so a community can inspect, use, and improve it — contrasted with proprietary/closed-source software. Linux distributions (Kali, Parrot, Arch, etc.) are all built on this open, community-driven model.

### 10.3 Key Properties of Linux

- Process isolation (each process runs in its own protected space)
- Permission model (Read / Write / Execute, assigned per user/group)
- High stability, widely used in servers, cloud infrastructure, and DevOps tooling

### 10.4 Linux File System Layout

| Directory | Purpose                                                                           |
| --------- | --------------------------------------------------------------------------------- |
| `/`       | Root partition (equivalent to `C:\` in Windows)                                   |
| `/bin`    | Essential user commands                                                           |
| `/sbin`   | System administration commands/tools                                              |
| `/etc`    | Configuration files for installed software (very important for changing settings) |
| `/home`   | Personal directories for non-root ("guest") users                                 |
| `/root`   | Home directory for the root (admin) user                                          |
| `/lib`    | Shared system libraries                                                           |
| `/usr`    | User-installed programs and their supporting files                                |
| `/var`    | Variable data files used by running software                                      |
| `/tmp`    | Temporary files (can be forensically useful — may retain leftover/deleted data)   |
| `/boot`   | Boot loader files                                                                 |
| `/opt`    | Optional/third-party software                                                     |
| `/media`  | Mount point for removable media (USB, etc.)                                       |
| `/sys`    | System/kernel/hardware information                                                |

### 10.5 Essential Commands

| Command                                | Purpose                                                                  |
| -------------------------------------- | ------------------------------------------------------------------------ |
| `pwd`                                  | Print current working directory                                          |
| `ls`                                   | List files/folders in current (or specified) location                    |
| `cd <folder>`                          | Change directory; `cd ..` goes up one level                              |
| `mkdir <name>`                         | Create a new directory                                                   |
| `cp <source> <destination>`            | Copy a file (`-r` flag needed for folders)                               |
| `mv <source> <destination>`            | Move/rename a file or folder                                             |
| `rm <file>`                            | Remove a file; `rm -r` for folders with contents                         |
| `rmdir <folder>`                       | Remove an _empty_ folder only                                            |
| `chmod`                                | Change file permissions (`+`/`-` with `r`, `w`, `x`)                     |
| `cat <file>`                           | Print file contents to terminal                                          |
| `<command> --help`                     | Quick usage help for a command                                           |
| `man <command>`                        | Full manual page for a command (press `q` to quit)                       |
| `apt update` / `apt install <package>` | Update package lists / install software (Debian-based distros like Kali) |
| `sudo su`                              | Elevate to root/admin privileges                                         |

> Note: Linux is case-sensitive — `Music` and `music` are treated as different names.

> File permission model: by default, new files get only Read and Write; Execute permission must be explicitly granted with `chmod +x`, which is a key reason Linux resists arbitrary code execution by default.

---

## 11. Information Gathering (Footprinting & Reconnaissance)

### 11.1 Why It Matters

Gathering detailed intelligence about a target _before_ attempting any technical attack dramatically increases efficiency — knowing the exact OS/software version lets you target known, relevant vulnerabilities instead of blindly trying generic techniques.

### 11.2 Categories of Targets

1. **Technology** – IP/MAC address, open ports, OS, software & versions, libraries/plugins
2. **Organization** – company details, employee info, financial records, future plans
3. **Individual** – personal details, contact info, routine, qualifications, reputation

### 11.3 Active vs Passive Information Gathering

- **Active:** direct interaction with the target (even from a fake profile/number counts as active — e.g., sending a request, pinging a device).
- **Passive:** indirect observation with no direct interaction (e.g., browsing a public profile without engaging).

### 11.4 Search Engine Techniques

- Never rely only on the first results page — SEO-optimized/paid content often crowds out useful information; dig through subsequent pages.
- Add context keywords (e.g., a profession, city) to narrow results about a specific person or entity.

### 11.5 Advanced Search Operators ("Google Dorking" basics)

Using Google's Advanced Search interface (or manual query syntax) to filter results:

- Exact phrase matching (quotes)
- Exclude terms (`-term`)
- Restrict to a specific site (`site:domain.com`)
- Restrict to a file type (`filetype:pdf`)
- Combine keywords with OR logic

> This is a standard, legitimate OSINT technique used by researchers and security professionals for gathering publicly available information — the same technique used for identifying accidentally-exposed information so it can be reported and fixed.

### 11.6 OSINT Frameworks & Tools (mentioned conceptually)

- **OSINT Framework** – a curated directory of open-source intelligence tools organized by category (username, email, domain, etc.)
- **Shodan** – a search engine for internet-connected devices/services (banner data, open ports, software versions)
- **theHarvester** – an OSINT tool that aggregates emails, subdomains, hosts, and related data from multiple public sources; can use API keys for services like Shodan for richer results

---

## 12. Network Scanning & Enumeration (with Nmap)

> All demonstrations below use a fully isolated, Host-Only/NAT-configured lab pairing a Kali Linux VM (tester) with **Metasploitable2** — a deliberately vulnerable Linux VM that is publicly distributed specifically for training purposes. This is one of the most common teaching setups in cybersecurity courses worldwide. Never point these techniques at systems you don't own or lack explicit written authorization to test.

### 12.1 The Five Goals of Network Scanning

1. **Host Discovery** – confirm the target's IP address is active/reachable
2. **Open Port Discovery** – identify which of the 65,535 ports are open
3. **Service Identification** – determine which protocol/service runs on each open port (e.g., port 80 → HTTP, port 21 → FTP)
4. **Software & Version Identification** – identify the exact software (and version) managing each service (e.g., HTTP on port 80 might be run by Apache 2, version 2.2.8)
5. **OS Fingerprinting / Banner Grabbing** – determine the operating system and version

**Banner grabbing explained:** most software displays an introductory "banner" when it starts or is queried — showing its name, version, developer, and sometimes a short description. Capturing and reading this banner (whether from the OS or a specific service) is called banner grabbing, and it's the basis for goals 4 and 5.

### 12.2 Practical Workflow Walkthrough

**Step 1 – Narrow down candidate hosts on the LAN**

- `netdiscover` — scans the local network range and lists active IP addresses (pre-installed in Kali; LAN-only).
- `ifconfig` — shows your own Kali machine's IP address/interfaces, useful for establishing your subnet range.
- The first address in a subnet (e.g., `x.x.x.1`) is typically reserved by the router itself (default gateway) and can usually be ruled out as a target.

**Step 2 – Broad scan of candidate IPs**
Example concept:

```
nmap -p 1-65535 -v -O <IP_1> <IP_2>
```

- `-p 1-65535` — scan the full port range
- `-v` — verbose output (shows what Nmap is doing step-by-step, not just the final result)
- `-O` — attempt OS detection

By comparing results across multiple candidate IPs (e.g., one returns "all ports closed / OS undetected" while another returns clear open ports and a Linux OS fingerprint), you can identify which host is actually the intended target.

**Step 3 – Full detail scan of the confirmed target**
Example concept:

```
nmap -p 1-65535 -v -sV -O <target_IP>
```

- `-sV` — version detection (retrieves software name + version for each open port)
- `-O` — OS detection (goal 5)

Typical result pattern from a training target (illustrative, not a live target):
| Port | Service | Software/Version |
|---|---|---|
| 21 | FTP | (a legacy FTP daemon version) |
| 22 | SSH | (an OpenSSH version) |
| 80 | HTTP | (an Apache HTTPD version) |
| 23, 25, 53, etc. | Telnet, SMTP, DNS, etc. | (varies) |

The OS detection result plus the scanned hostname/version banner together fulfill goal 5.

### 12.3 Enumeration — Going Deeper Than Scanning

Scanning tells you _what's there_ (IP, open ports, service names, software versions, OS). **Enumeration** uses that data to actively query each identified service for more specific, actionable information: exact configuration details, default/misconfigured accounts, supported commands, and other data a defender needs to lock down (and a tester documents in a findings report).

Analogy used in the course: scanning tells you a door is open; enumeration is walking through that door to see what's inside the house.

### 12.4 Nmap Scripting Engine (NSE)

Nmap ships with a large library of small scripts (`.nse` files), organized by the protocol/service they target (e.g., multiple scripts each for FTP, SSH, HTTP, SMTP, etc.). These live under Nmap's shared scripts directory and can be listed with `ls` once you `cd` into it.

**Two ways to run them:**

**A) Automated / Default Script Scan**

```
nmap -p 1-65535 -v -sV -sC <target_IP>
```

- `-sC` — runs Nmap's default set of NSE scripts automatically against whatever open ports/services are detected.
- Convenient, but only runs a subset of the available scripts per service (e.g., if there are 8 FTP-related scripts, the default scan might only run 1–2 of them).

**B) Manual / Targeted Script Scan**
Lets you run _all_ scripts related to a specific protocol against a specific port:

```
nmap <target_IP> -p <port> -sV --script=<protocol>*
```

Example concept: targeting SMTP specifically —

```
nmap <target_IP> -p 25 -sV --script=smtp*
```

- The port must be specified precisely (not the full 1–65535 range), because running an SMTP script against a port where SMTP isn't running produces errors and wastes time.
- The `*` wildcard runs every script whose name starts with the given protocol prefix (e.g., all `smtp-*.nse` scripts), giving much more thorough enumeration coverage than the default automated scan.

### 12.5 Example Finding: Anonymous FTP Login

One of the FTP-related NSE scripts may report that **anonymous login is allowed** on a target's FTP service. Historically, many FTP server builds shipped with a default "anonymous" account enabled (often no password required, or the word "anonymous" itself used as the password) — a well-documented, legacy misconfiguration.

> **This is a defensive/remediation lesson, not an attack technique.** The correct professional response to finding this is to: disable the anonymous FTP account, upgrade the FTP service to a current, patched version, and enforce proper authentication — exactly what a security assessment report would recommend to the system owner.

### 12.6 Summary: Why Enumeration Matters

Good enumeration is what separates an efficient security assessment from random guesswork. By working systematically — host discovery → port scan → service ID → version ID → OS ID → targeted script-based enumeration — a tester builds a complete, evidence-based picture of a system's attack surface, which is the foundation for writing an accurate vulnerability report and recommending fixes.

---

## 13. Practice Lab Environment (Concepts)

Standard, industry-common lab setup for **safe, legal, isolated practice** (never test tools against systems you don't own or have explicit written permission to test):

- **VirtualBox / VMware** – virtualization software to run isolated virtual machines
- **Kali Linux** – a Debian-based distribution pre-loaded with security testing tools, used for the "attacker" VM
- **Metasploitable2** – an intentionally vulnerable Linux VM published specifically for training purposes (widely used worldwide in university courses and certifications) — never exposed to real networks
- Network mode should typically be set to **Host-Only** or **NAT**, keeping the lab isolated from your real/production network

### 13.0 Setting Up Metasploitable2 (Step-by-Step)

Metasploitable2 is a free, publicly distributed practice VM (commonly hosted on SourceForge/OSBoxes mirrors) that ships with intentionally vulnerable services — including a web stack (DVWA, Mutillidae, phpMyAdmin, etc.) and multiple deliberately weak network services (FTP, SSH, Telnet, SMTP, etc.) — specifically so students can practice web, network, and service-level penetration testing legally and safely.

**Download & Extraction**

1. Search for "Metasploitable2" and download the `.zip` archive from a reputable mirror (e.g., SourceForge).
2. Extract the archive using WinRAR/7-Zip. Inside, you'll find a `.vmdk` (virtual disk) file rather than a full pre-built `.ova`/`.vbox` machine.

**Importing into VirtualBox**

1. Open VirtualBox → **New**.
2. Name the VM (e.g., "Metasploitable2").
3. Choose **Type: Linux**, **Version: Other Linux (64-bit)**.
4. Skip unattended install / ISO selection — there is no installer, since you're attaching a pre-made disk image.
5. Allocate minimal resources (default RAM, 1 CPU) — this is a lightweight, deliberately old, low-resource target VM.
6. For the hard disk step, choose **"Use an existing virtual hard disk file"** → Browse → select the `.vmdk` file extracted earlier.
7. Click **Finish**.

**Network Configuration**

- Set the VM's network adapter(s) to **Host-Only** and/or **NAT**, matching the configuration used on your Kali attacker VM, so both machines can reach each other on an isolated internal network segment — never bridge this VM onto a real/production or public network.

**First Boot & Login**

- Metasploitable2 boots directly to a command-line login prompt (no GUI).
- Default credentials (publicly documented by the VM's own creators, since it's meant purely for training): username `msfadmin`, password `msfadmin` (a `user`/`user` account also exists).
- After logging in, run `ifconfig` to confirm the VM's assigned IP address — this is the target IP you'll use in Nmap/Metasploit exercises from your Kali machine.
- The web applications (DVWA, Mutillidae, phpMyAdmin, etc.) bundled on the VM are accessible by browsing to that IP address from your Kali machine's browser.

> Because Metasploitable2's default credentials and vulnerabilities are publicly documented by design (it exists purely as a training target), none of this constitutes sensitive information — it's the standard "hello world" lab machine used across virtually every introductory pentesting course, CTF, and certification path.

### 13.1 Metasploit Framework (conceptual terms)

| Term          | Meaning                                                         |
| ------------- | --------------------------------------------------------------- |
| **Auxiliary** | Modules that assist with information gathering/enumeration      |
| **Exploit**   | Modules that leverage a discovered vulnerability to gain access |
| **Payload**   | Modules that help maintain access once a system is compromised  |

Basic setup commands: `service postgresql start`, `msfdb init`, `msfconsole`, `db_status` (confirms database connectivity for storing scan results), `db_nmap` (runs Nmap scans directly into the Metasploit database).

> This entire section refers to working exclusively inside a deliberately vulnerable, offline training VM (Metasploitable2) that ships publicly for this exact educational purpose — the same setup used in most university/CEH/OSCP-adjacent coursework.

---

## 14. Key Takeaways / Ethical Reminders

- Always operate strictly within **written authorization** — unauthorized access to systems you don't own is illegal (per the "Black Hat" definition above).
- Practice exclusively in **isolated lab environments** (your own VMs, intentionally vulnerable training machines, or bug bounty programs with explicit scope).
- Foundational knowledge (networking, Linux, OS internals) is what separates efficient, effective security professionals from those relying on guesswork.
- Continuous learning (books, certifications, labs, communities) is essential — this field evolves constantly.

---

_End of notes._
