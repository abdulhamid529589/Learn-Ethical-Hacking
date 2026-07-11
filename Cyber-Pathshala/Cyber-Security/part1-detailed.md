# Cybersecurity & Ethical Hacking — Foundations Module

### Networking, Linux, and Reconnaissance Concepts

_Compiled study notes — for lab use on your own systems and Metasploitable2 VM only_

---

## Table of Contents

1. [Career Roadmap Overview](#1-career-roadmap-overview)
2. [Hacker Classifications](#2-hacker-classifications)
3. [Networking Fundamentals](#3-networking-fundamentals)
4. [IP Addressing](#4-ip-addressing)
5. [MAC Addressing](#5-mac-addressing)
6. [Ports](#6-ports)
7. [Protocols & the TCP/IP Model](#7-protocols--the-tcpip-model)
8. [TCP Deep Dive: Flags & Handshakes](#8-tcp-deep-dive-flags--handshakes)
9. [OSI vs TCP/IP Model](#9-osi-vs-tcpip-model)
10. [How the Internet Works: Servers, Domains, DNS](#10-how-the-internet-works-servers-domains-dns)
11. [Home Networking: ISP, Router, NAT, DHCP, ARP](#11-home-networking-isp-router-nat-dhcp-arp)
12. [Linux Fundamentals](#12-linux-fundamentals)
13. [Linux File System](#13-linux-file-system)
14. [Linux Command Line Basics](#14-linux-command-line-basics)
15. [Information Gathering (Footprinting & Reconnaissance)](#15-information-gathering-footprinting--reconnaissance)
16. [Network Scanning — Concepts & Goals](#16-network-scanning--concepts--goals)
17. [Enumeration — Concepts](#17-enumeration--concepts)
18. [Suggested Practical Lab Plan](#18-suggested-practical-lab-plan)

---

## 1. Career Roadmap Overview

The course frames a learning path around four pillars:

| Pillar           | Focus                                                                                             |
| ---------------- | ------------------------------------------------------------------------------------------------- |
| **Learn**        | Networking → Linux → Ethical Hacking modules → choose a Cybersecurity domain                      |
| **Certificates** | Beginner (eJPT, CEH, Security+) → Advanced (OSCP, eCPPT, eWPT) → Expert (OSEP, CRTP, CRTE, GPEN)  |
| **Earn**         | Offensive (red team), Defensive (blue team), GRC, Training/Awareness, General IT/Security Support |
| **Sources**      | LinkedIn, job boards, books, structured courses, labs                                             |

**Recommended learning order:**

1. Networking basics (IP/protocols/OSI-TCP-IP model)
2. Linux OS & command line
3. Ethical hacking curriculum: footprinting → scanning → enumeration → vulnerability analysis → exploitation → post-exploitation/securing
4. Pick a specialization domain (web, network, cloud, mobile, IoT, AI security, etc.)

**Beginner-friendly job title keywords to search for:** _Junior_, _Intern_, _Trainee_, _Associate_, _Helper_ — avoid titles that assume years of experience.

---

## 2. Hacker Classifications

| Type          | Intent              | Legality        | Permission             | Motivation                                                    |
| ------------- | ------------------- | --------------- | ---------------------- | ------------------------------------------------------------- |
| **White Hat** | Ethical / defensive | Legal           | Authorized             | Career growth, bug bounties, securing systems                 |
| **Black Hat** | Malicious           | Illegal         | Unauthorized           | Financial gain, espionage, sabotage, revenge                  |
| **Gray Hat**  | Mixed/ambiguous     | Legal gray area | Sometimes unauthorized | Fame, "public interest" disclosure, occasional financial gain |

**Key distinction:** All three groups can possess _equal technical skill_ — the differentiator is **how that knowledge is applied**, not how much of it exists.

- **White hat** activities: penetration testing, vulnerability assessment, bug bounty programs.
- **Gray hat** risk: publicly disclosing a vulnerability before the vendor patches it creates legal exposure even if the _intent_ was to force a fix.
- **Espionage** = surveillance for political/state motives. **Sabotage** = intentional disruption to cause harm.

> Ethical takeaway: legality hinges on **authorization**. Testing your own systems/websites and an isolated lab VM (Metasploitable2) is what keeps this legal — never test systems, networks, or accounts you don't own or have explicit written permission to test.

---

## 3. Networking Fundamentals

**Definition:** Computer networking = two or more devices communicating and sharing data, files, folders, hardware, or software.

### Network Types (by range)

| Type    | Full Form                 | Range                 | Example                 |
| ------- | ------------------------- | --------------------- | ----------------------- |
| **LAN** | Local Area Network        | ~15–50 devices, ~100m | Office, computer lab    |
| **MAN** | Metropolitan Area Network | Multiple LANs joined  | University campus, city |
| **WAN** | Wide Area Network         | Global                | The Internet            |

_(Other types like PAN, CAN exist but are less relevant to offensive security fundamentals.)_

### Core Networking Entities

| Entity          | Full Form                 | Role                                               |
| --------------- | ------------------------- | -------------------------------------------------- |
| **ISP**         | Internet Service Provider | Supplies internet access (e.g., telecom carriers)  |
| **MAC Address** | Media Access Control      | Unique hardware/physical address                   |
| **Port**        | —                         | Communication route in/out of a device (1–65535)   |
| **Protocol**    | —                         | Rules governing safe/structured communication      |
| **IP Address**  | Internet Protocol Address | Unique logical address for networked communication |

---

## 4. IP Addressing

### Definition

An IP address uniquely identifies a device on a network so devices can locate and communicate with each other — analogous to a mailing address.

### IPv4 Structure

- Format: four octets, e.g. `172.217.194.100`
- Each octet: 0–255 (8 bits / 1 byte)
- Total size: 32 bits

### Conceptual breakdown of an IP (how geolocation/tracing works)

1. **1st octet range** → broadly narrows to country/region block
2. **2nd octet** → narrows to state/ISP allocation
3. **3rd octet** → narrows to city + the **ISP** that owns the block
4. **Full address + ISP logs** → the ISP can (with legal process, e.g. a law enforcement request) map the address to the specific subscriber account active at that timestamp

This is the basis of how IP addresses assist law-enforcement tracing — the address itself only reveals ISP/region; identifying the actual person requires the ISP's subscriber logs (a legal process, not something the address discloses on its own).

### Types of IP Addresses

| Type           | Scope                          | Analogy                                                                                                               |
| -------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| **Private IP** | Usable only within a LAN       | Your nickname (only your household knows it)                                                                          |
| **Public IP**  | Usable across the WAN/Internet | Your legal name (works anywhere)                                                                                      |
| **Static IP**  | Fixed, doesn't change          | Needed for servers (consistent address so clients can always find them)                                               |
| **Dynamic IP** | Reassigned when unused         | Efficient reuse — IPv4 has ~4.3 billion addresses but ~5+ billion active global users, so idle addresses get recycled |

### The IP Communication Rule

> A **private IP can only talk directly to another private IP** (within its LAN).
> A **public IP can only talk directly to another public IP** (across the WAN).
> A private IP **cannot** directly communicate with a public IP or vice versa — this is why **NAT** (Network Address Translation) exists at your router (see §11).

**Practical check:** Search "what is my IP" in a browser to see your public IP and its ISP/region attribution — this demonstrates the concept live.

---

## 5. MAC Addressing

### Definition

A **hardware/physical address** burned into every network-capable device (NIC, Wi-Fi adapter, etc.) at manufacture time.

### Structure

- 6 pairs of hex characters (0–9, A–F), e.g. `F4:7B:09:XX:XX:XX`
- 48 bits / 6 bytes total
- Split into two halves:
  - **First 3 pairs (OUI – Organizationally Unique Identifier):** identifies the **manufacturer/vendor** (Intel, Samsung, etc.)
  - **Last 3 pairs (NIC-specific):** identifies the **individual device**

### Checking your MAC address

```bash
# Windows
getmac

# Linux / macOS
ifconfig
# or
ip link show
```

### MAC Lookup (Practical)

Search "MAC address lookup" tools online, paste the OUI (first 3 pairs) → returns the manufacturer and often manufacture-date metadata. Combined with vendor sales/billing records (again, only accessible via legal process), this is how device-level tracing works in principle.

> Note: MAC addresses are traditionally described as "permanent," but tools exist to **spoof/change** them at the OS level — relevant later for anonymity-related coursework, and something you can safely practice changing on your own lab machine's virtual NIC.

---

## 6. Ports

### Definition

Ports are the **logical communication channels** (1–65535) a device uses to send/receive specific types of traffic. Combined with an IP address, a port specifies _exactly_ which application/service on a machine a packet is meant for.

### The Two Port Rules

1. **The port must be open** (a listener/service must be bound to it) or the packet is dropped — like knocking on a door nobody is behind.
2. **Only one process can bind to a given port at a time.** If Software A is using port 8080, Software B cannot also bind to 8080 simultaneously — you'll get a "port already in use" error.

### Port Ranges

| Range       | Name                      | Description                                                                     |
| ----------- | ------------------------- | ------------------------------------------------------------------------------- |
| 0–1023      | **Well-Known Ports**      | Reserved for common core services (HTTP=80, HTTPS=443, FTP=21, SSH=22, SMTP=25) |
| 1024–49151  | **Registered Ports**      | Assigned to specific applications by IANA, less contested                       |
| 49152–65535 | **Dynamic/Private Ports** | Temporary, used for short-lived/ephemeral connections                           |

### Common Well-Known Ports

| Port  | Service   |
| ----- | --------- |
| 20/21 | FTP       |
| 22    | SSH       |
| 23    | Telnet    |
| 25    | SMTP      |
| 53    | DNS       |
| 80    | HTTP      |
| 443   | HTTPS/SSL |

**Security relevance:** Malware/backdoor payloads that try to bind to a well-known port (e.g., 80) often _fail_ to exfiltrate data because that port is already legitimately occupied by a running service (e.g., a web server). This is why offensive tooling typically favors the **registered port range**, which is more likely to be free — a key reason to understand port allocation theory before doing any payload/listener work in your lab.

### Checking open ports locally

```bash
# See what's actively listening on your own machine
sudo ss -tulnp
# or
sudo netstat -tulnp
```

---

## 7. Protocols & the TCP/IP Model

**Protocol** = a defined rule set ensuring data is transmitted safely, reliably, and in a format both ends understand — like traffic rules for data.

### The Four-Layer TCP/IP Model (practical, compressed version of OSI)

| Layer                    | Purpose                                | Example Protocols                                    |
| ------------------------ | -------------------------------------- | ---------------------------------------------------- |
| **Application Layer**    | User-facing services & data formatting | HTTP, HTTPS, FTP, SMTP, POP3, DNS, SSH, Telnet, SNMP |
| **Transport Layer**      | Establishes end-to-end connections     | **TCP**, **UDP**                                     |
| **Internet Layer**       | Addressing & routing                   | IP, ICMP, ARP, DHCP                                  |
| **Network Access Layer** | Physical/hardware transmission         | Ethernet, Wi-Fi, ADSL                                |

### TCP vs UDP

|                 | TCP                                                | UDP                                                                     |
| --------------- | -------------------------------------------------- | ----------------------------------------------------------------------- |
| **Full name**   | Transmission Control Protocol                      | User Datagram Protocol                                                  |
| **Connection**  | Connection-oriented (3-way handshake)              | Connectionless                                                          |
| **Reliability** | Guaranteed delivery, ordered                       | Best-effort, no guarantee                                               |
| **Speed**       | Slower (overhead of handshake/ack)                 | Faster                                                                  |
| **Use case**    | Web, email, file transfer — where accuracy matters | Streaming, DNS queries, VoIP — where speed matters more than perfection |

---

## 8. TCP Deep Dive: Flags & Handshakes

### The Six Key TCP Flags

| Flag    | Full Name       | Function                                                                              |
| ------- | --------------- | ------------------------------------------------------------------------------------- |
| **SYN** | Synchronize     | Initiates a new connection request                                                    |
| **ACK** | Acknowledgement | Confirms receipt of a packet                                                          |
| **PSH** | Push            | Sends buffered data immediately, without waiting                                      |
| **FIN** | Finish          | Requests graceful connection termination                                              |
| **URG** | Urgent          | Marks a packet as high priority, process immediately                                  |
| **RST** | Reset           | Forcibly terminates and can restart a connection (used to recover from broken states) |

### The Three-Way Handshake (Connection Establishment)

```
Client                      Server
  |------ SYN (seq=10) ------>|
  |<--- SYN-ACK (seq=142,     |
  |      ack=11) -------------|
  |------ ACK (ack=143) ----->|
   [connection established — data transfer begins]
```

1. Client sends **SYN**: "I want to talk, is this port open?"
2. Server replies **SYN+ACK**: "Got your request, yes, let's connect."
3. Client replies **ACK**: "Confirmed — connection established."

**Sequence numbers** ensure that data broken into multiple packets can be reassembled in the correct order at the destination.

### Session Termination (four-step, NOT called a "handshake")

```
Client                      Server
  |------ FIN --------------->|
  |<----- ACK -----------------|
  |<----- FIN ------------------|
  |------ ACK ----------------->|
   [connection closed]
```

**Practical lab exercise:** Run `sudo tcpdump -i <interface> tcp` or open Wireshark while browsing to a site you own, and watch the SYN → SYN-ACK → ACK handshake and FIN/ACK teardown happen live. This is one of the best ways to internalize this concept.

---

## 9. OSI vs TCP/IP Model

### OSI Model (7 layers, theoretical, read bottom-up)

| #   | Layer        | Function                                      | Example Protocols     |
| --- | ------------ | --------------------------------------------- | --------------------- |
| 7   | Application  | User-facing apps                              | HTTP, FTP, SMTP       |
| 6   | Presentation | Formatting, encryption (SSL/TLS), compression | SSL, TLS              |
| 5   | Session      | Establishes/manages sessions                  | NetBIOS, RPC, SOCKS   |
| 4   | Transport    | End-to-end connection                         | TCP, UDP              |
| 3   | Network      | Logical addressing/routing                    | IP, ICMP, ARP, DHCP   |
| 2   | Data Link    | Frames, MAC addressing, hardware delivery     | Ethernet, ARP         |
| 1   | Physical     | Raw bit transmission (signals)                | Cabling, radio, Wi-Fi |

### How TCP/IP compresses OSI

| TCP/IP (4-layer) | Combines OSI Layers                          |
| ---------------- | -------------------------------------------- |
| Application      | Application + Presentation + Session (7,6,5) |
| Transport        | Transport (4)                                |
| Internet         | Network (3)                                  |
| Network Access   | Data Link + Physical (2,1)                   |

**Security relevance by layer:**

- **Application layer** — highest attack surface (direct user interaction): most web app vulnerabilities live here.
- **Presentation layer** — encoding/encryption weaknesses.
- **Session layer** — session hijacking attacks target this layer.
- **Transport layer** — reverse shells / bind shells operate here.
- **Lower layers (Network, Data Link, Physical)** — generally considered more hardened/harder to exploit remotely, though ARP spoofing and similar attacks target Layer 2/3.

---

## 10. How the Internet Works: Servers, Domains, DNS

### Servers

A **server** is simply a machine with its own **static public IP address**, hosting a service (website, files, etc.) that others can reach over the WAN.

### Domains & Subdomains

- A **domain** (e.g., `example.com`) is a human-readable **name mapped to an IP address** — because remembering numeric IPs at scale isn't practical.
- **Subdomains** (e.g., `blog.example.com`, `mail.example.com`) let you create unlimited named subdivisions under one domain, always sharing the parent domain's suffix.
- **Extensions/TLDs** (`.com`, `.org`, `.in`, `.jp`) denote category or country.

### DNS Records (set in your domain registrar's control panel)

| Record    | Purpose                                                                          |
| --------- | -------------------------------------------------------------------------------- |
| **A**     | Maps domain → IPv4 address                                                       |
| **AAAA**  | Maps domain → IPv6 address                                                       |
| **CNAME** | Alias/redirect to another domain name                                            |
| **MX**    | Specifies mail server(s) for the domain                                          |
| **TXT**   | Free-form text; commonly used for **SPF** (anti-spoofing/anti-spam verification) |
| **NS**    | Specifies authoritative Name Servers managing the domain's DNS                   |
| **SOA**   | Administrative contact / authority info for the domain                           |
| **PTR**   | Reverse DNS lookup (IP → hostname)                                               |
| **SRV**   | Specifies location of specific services                                          |

### DNS Resolution Flow (how a browser finds a website)

```
User types "example.com"
      ↓
Local/browser cache? → if cached, use it
      ↓ (if not cached)
Local Resolver / ISP Resolver → checks cache
      ↓ (if not found)
Root DNS Server → "I don't have it, ask the TLD servers for .com"
      ↓
TLD Server (.com) → "I don't have the IP, but here are the Name Servers for example.com"
      ↓
Name Server (from domain's NS records) → looks in the Zone File
      ↓
Zone File → contains the A record → returns the IP address
      ↓
Browser connects directly to that IP address
      ↓
(Result is cached locally for faster future lookups)
```

### Zone Files

A **zone file** is the public-facing, periodically-synced copy of a domain's DNS records (A, MX, TXT, NS, etc.), served by the authoritative name servers so the entire internet can resolve the domain — without needing login access to the registrar's private control panel.

> Attack note: **DNS zone transfer** misconfigurations (allowing anyone to request a full zone file dump via `AXFR`) are a classic reconnaissance target — you can safely test this against your own domains/lab DNS server using tools like `dig axfr @nameserver domain.com`.

**Practical lab exercise:** Buy/use a domain you own (or a free lab domain), configure A/MX/TXT records yourself, and observe propagation using:

```bash
dig example.com A
dig example.com MX
dig example.com TXT
dig example.com NS
whois example.com
```

---

## 11. Home Networking: ISP, Router, NAT, DHCP, ARP

### The Chain of Addressing

1. **ISP** reserves a block of public IPs and assigns one to your **router/broadband modem**.
2. Your **router** reserves its own block of **private IPs** (e.g., `192.168.1.0–192.168.1.255`) for devices on your LAN, keeping the first address (`192.168.1.1`) for itself — this is the **default gateway / subnet address**.
3. Devices connecting to the router are assigned private IPs from that pool.

### DHCP (Dynamic Host Configuration Protocol)

Automatically assigns IP addresses, subnet mask, gateway, and DNS settings to devices joining the network — eliminating manual configuration.

- **Subnet mask** — defines which portion of an IP is the network vs host portion (defines the usable address range).
- **Subnetting** — dividing a large network into smaller segments improves performance and security (limits broadcast domain size, contains potential attacks).

### NAT (Network Address Translation)

Since private IPs can't talk directly to public IPs, your router performs NAT: it translates outbound private-IP traffic to use its own public IP, and translates inbound replies back to the correct private IP on your LAN — this is what lets multiple devices share one public IP.

### ARP (Address Resolution Protocol)

Maps **IP addresses to MAC addresses** within a LAN using a broadcast: _"Who has 192.168.1.10? Tell 192.168.1.1."_ The device holding that IP replies with its MAC address, and the router updates its ARP table accordingly.

```bash
# View your ARP table
arp -a
# or
ip neigh
```

> **Attack note:** ARP has no built-in authentication, which is what makes **ARP spoofing/poisoning** possible (an attacker sends forged ARP replies to redirect traffic through their own machine — a classic MITM technique). Safe to practice with `arpspoof`/`ettercap` on your own isolated lab network only, never on shared/production Wi-Fi.

---

## 12. Linux Fundamentals

### Kernel vs OS vs Terminal

- **Kernel** — the core layer that directly interfaces with hardware (CPU, RAM, disk). Linux, strictly speaking, _is_ a kernel, not a full OS.
- **Terminal / Shell** — software (bash, zsh, fish) that lets a human send commands to the kernel using a defined programming language/syntax.
- **GNU/Linux** — the combination of the open-source GNU toolset with the Linux kernel, forming a complete OS. Different **distributions** (Kali, Parrot, Ubuntu, Arch) package this differently.

### Why "Open Source" Matters for Security Tools

Open-source software's source code is publicly visible and modifiable, allowing rapid community-driven improvement — which is why security tooling (Nmap, Metasploit, most of the Kali toolkit) evolves so quickly compared to closed proprietary software.

### Why Linux Is Favored for Security Work

- **Process isolation** — each process runs in its own memory space; a crash in one doesn't take down the system.
- **Granular permission model** — read/write/execute permissions per user/group, enforced strictly by default.
- **Stability** — can run for long periods without degradation.
- **Industry ubiquity** — most cloud infrastructure, servers, and DevOps tooling (Docker, AWS backends, etc.) runs on Linux.

---

## 13. Linux File System

Linux uses a single unified tree rooted at `/` (unlike Windows' drive-letter system).

| Directory        | Contents                                                                                  |
| ---------------- | ----------------------------------------------------------------------------------------- |
| `/bin`           | Essential user command binaries (ls, cd, etc.)                                            |
| `/sbin`          | System administration binaries (reboot, disk tools)                                       |
| `/etc`           | **Configuration files** for installed software — critical for admin/pentest work          |
| `/home`          | Per-user data directories for non-root ("guest") accounts                                 |
| `/root`          | Home directory for the root (admin) account                                               |
| `/lib`           | Shared system libraries                                                                   |
| `/usr`           | User-installed (non-default) applications and their data                                  |
| `/var`           | Variable/changing data — logs, mail queues, app runtime data                              |
| `/tmp`           | Temporary files — often forensically valuable, may retain traces of deleted/modified data |
| `/boot`          | Boot loader files                                                                         |
| `/opt`           | Optional/third-party software, self-contained installs                                    |
| `/media`, `/mnt` | Removable media mount points                                                              |
| `/sys`           | Kernel and hardware/system info                                                           |

### User Account Types

- **Guest/standard user** — limited permissions, cannot modify system-critical files.
- **Root/admin user** — full system control; equivalent to Windows "Administrator."

### File Permissions Model

Every file has three possible permissions: **R**ead, **W**rite, **e**X**ecute** — applied to _owner_, _group_, and _others_. By default, new files get read/write but **not** execute — this is a core Linux security default (a downloaded file won't auto-run just because of its extension, unlike some Windows behaviors).

```bash
# View permissions
ls -l

# Grant/revoke permissions
chmod +x filename     # add execute permission
chmod -w filename     # remove write permission
chmod +rwx filename   # add all three
chmod 755 filename    # numeric form: owner=rwx, group/other=r-x
```

---

## 14. Linux Command Line Basics

```bash
pwd                     # print current working directory
ls                      # list files/folders in current directory
ls -l                   # long listing with permissions, owner, size, date
cd foldername           # change into a directory
cd ..                   # move up one directory
cd /                    # go to root
mkdir foldername        # create a new directory
cp source destination         # copy a file
cp -r source destination      # copy a directory recursively
mv source destination         # move/rename a file or folder
rm filename              # remove a file
rm -rf foldername        # remove a folder and its contents (recursively, forced)
cat filename              # print file contents to screen
./scriptname             # execute a script/program (must have execute permission)
sudo su                  # elevate to root shell (use with care)
```

### Getting Help (Essential Habit)

```bash
command --help       # quick usage summary
man command           # full manual page (press q to exit)
```

### Package Management (Debian/Kali — APT)

```bash
sudo apt update          # refresh package index (do this first, always)
sudo apt upgrade         # upgrade all installed packages
sudo apt install toolname   # install a new tool
sudo apt remove toolname    # uninstall a tool
```

> **Practical tip:** `/etc/apt/sources.list` defines where APT pulls packages from — a good first thing to inspect/understand on a fresh Kali VM if package installs are failing.

---

## 15. Information Gathering (Footprinting & Reconnaissance)

### Why It Matters

The course's own illustrative comparison: an experienced attacker who skips recon and just throws known exploits at a target can spend a week failing, while an attacker who spends even half a day identifying the exact OS/software version in use can often succeed in a single, well-chosen attempt. **Reconnaissance dramatically improves efficiency and precision.**

### Categories of Targets

- **Technology** — servers, IPs, open ports, OS, software versions, CMS/frameworks, libraries.
- **Organization** — company structure, employees, financials, public goals.
- **Individual** — name, contact info, public digital footprint, role, habits.

### Active vs Passive Recon

| Type        | Description                                                 | Example                                                     |
| ----------- | ----------------------------------------------------------- | ----------------------------------------------------------- |
| **Passive** | No direct interaction with the target; purely observational | Browsing a public LinkedIn/social profile without engaging  |
| **Active**  | Direct interaction/engagement, even under a pseudonym       | Sending a connection request, pinging a host, port scanning |

> Active recon against a target's live infrastructure (scanning, pinging) should **only** be done against systems you own or are explicitly authorized to test.

### OSINT Technique: Advanced Search Operators ("Google Dorking" — legitimate use)

Search engines rank for SEO/commercial intent by default, not necessarily what's most factually relevant — so advanced operators narrow results precisely:

| Operator          | Effect                             |
| ----------------- | ---------------------------------- |
| `"exact phrase"`  | Forces an exact match              |
| `-word`           | Excludes a term from results       |
| `site:domain.com` | Restricts results to one site      |
| `filetype:pdf`    | Restricts to a file type           |
| `intitle:word`    | Word must appear in the page title |
| `inurl:word`      | Word must appear in the URL        |

**Practical exercise:** Practice OSINT techniques on your **own** name, company, or website to see exactly what's publicly discoverable about you/your organization — this is a completely safe and highly instructive exercise, and a real part of professional OSINT/red-team work (finding your own exposure before an attacker does).

### OSINT Framework Concept

Structured directories of free OSINT tools exist online, organized by data type (username lookup, email lookup, domain WHOIS, subdomain discovery, etc.) — useful as a jumping-off point once you're ready to explore individual tools.

```bash
# WHOIS lookup — safe against any public domain, including your own
whois example.com

# DNS record enumeration
dig example.com ANY
```

### theHarvester (OSINT aggregation tool)

A Kali-included tool that aggregates OSINT data (emails, subdomains, hosts) about a target domain from multiple public sources (some requiring free API keys, e.g., Shodan).

```bash
theHarvester -d yourdomain.com -b all -l 100
```

- `-d` = target domain
- `-b` = data source(s) — use `-b all` or specific sources
- `-l` = limit result count (helps avoid API rate-limiting)

**Only run this against domains/organizations you own or are authorized to assess.**

---

## 16. Network Scanning — Concepts & Goals

Network scanning is **active reconnaissance** performed against a target IP to map its attack surface. Below are the five classic goals, using **Nmap** (Network Mapper) as the reference tool.

### The Five Goals of Scanning

| #   | Goal                                    | What You Learn                                                     |
| --- | --------------------------------------- | ------------------------------------------------------------------ |
| 1   | **Host Discovery**                      | Is this IP address active/alive?                                   |
| 2   | **Port Discovery**                      | Which of the 65,535 ports are open?                                |
| 3   | **Service Identification**              | Which protocol/service runs on each open port (HTTP, FTP, SSH...)? |
| 4   | **Software/Version Detection**          | What software (and exact version) is running that service?         |
| 5   | **OS Fingerprinting / Banner Grabbing** | What operating system (and version) is the host running?           |

**"Banner grabbing"** specifically refers to capturing the introductory/identifying text a service displays on connection (software name, version, sometimes OS) — a low-effort way to learn a lot about a target.

### Nmap Command Structure (Reference)

```bash
# Basic host discovery on a local subnet
nmap -sn 192.168.1.0/24

# Full port scan with service/version detection + OS detection
nmap -p 1-65535 -v -sV -O <target-ip>

# Default script scan (automated enumeration scripts)
nmap -sC -sV -p- <target-ip>

# Targeted script scan for one service (e.g., all SMTP-related scripts)
nmap -p 25 --script "smtp*" <target-ip>
```

**Flag reference:**
| Flag | Meaning |
|---|---|
| `-sn` | Ping scan only (host discovery, no port scan) |
| `-p` | Specify port(s) — `-p-` means all 65535 |
| `-v` | Verbose output |
| `-sV` | Service/version detection |
| `-O` | OS detection |
| `-sC` | Run Nmap's default safe script set |
| `--script <name/pattern>` | Run specific NSE (Nmap Scripting Engine) script(s) |

Nmap's scripts live at `/usr/share/nmap/scripts/` and are named by protocol prefix (`ftp-*`, `http-*`, `smtp-*`, `ssh-*`) — inspecting this folder is a great way to see the full breadth of what's testable per-service.

```bash
ls /usr/share/nmap/scripts/ | grep ftp
```

> **Lab-only reminder:** Run these scans exclusively against your Metasploitable2 VM, your own lab network, or infrastructure you explicitly own. Unauthorized scanning of third-party systems is illegal in most jurisdictions even without exploitation.

---

## 17. Enumeration — Concepts

**Enumeration** = "advanced" or "deeper" information gathering that goes beyond basic scanning — using discovered open ports/services as an entry point to extract more specific, often sensitive, data (usernames, misconfigurations, banner details, default credentials exposure, etc.).

- **Nmap's NSE scripts** (`-sC` or targeted `--script`) are one enumeration mechanism.
- **Metasploit Framework "auxiliary" modules** serve a similar advanced-enumeration purpose within that toolset, separate from "exploit" modules (which act on a discovered weakness) and "payload/post" modules (which maintain access after successful exploitation).

### Conceptual terminology in Metasploit (for later modules)

| Term          | Role                                                  |
| ------------- | ----------------------------------------------------- |
| **Auxiliary** | Information-gathering / enumeration modules           |
| **Exploit**   | Modules that leverage a specific vulnerability        |
| **Payload**   | Code delivered/executed after successful exploitation |

_(Detailed exploitation methodology — module selection, payload staging, post-exploitation — deserves its own dedicated, hands-on module once you're comfortable with the recon/scanning foundations above. Best learned interactively against Metasploitable2 directly in your terminal, following Metasploit's own `help`, `show options`, and `info` commands for the exact syntax of whatever module you're using.)_

---

## 18. Suggested Practical Lab Plan

Since you already have your own websites and a Metasploitable2 VM, here's a structured way to turn this document into hands-on practice:

### Phase 1 — Networking Fundamentals

- [ ] Run `ip a` / `ifconfig` on your host and Kali VM; identify your private IP, MAC, subnet.
- [ ] Use `arp -a` to see your ARP table; correlate MACs to devices you recognize.
- [ ] Capture a TCP handshake with `tcpdump`/Wireshark against your own web server.
- [ ] Look up your public IP and confirm ISP/region attribution via a "what is my IP" service.

### Phase 2 — DNS & Domains

- [ ] If you own a domain, inspect its A/MX/TXT/NS records via `dig`.
- [ ] Attempt (and expect to fail, if configured correctly) a DNS zone transfer against your own DNS server: `dig axfr @ns1.yourdomain.com yourdomain.com`.

### Phase 3 — Linux Mastery

- [ ] Practice file permission changes (`chmod`) on test files.
- [ ] Explore `/etc` on your Kali VM to find configs for 3 tools you've installed.
- [ ] Write a basic shell script, `chmod +x` it, and run it with `./script.sh`.

### Phase 4 — Recon & OSINT (on yourself/your own assets only)

- [ ] Run `theHarvester` against a domain you own.
- [ ] Practice 5 different Google dork operators searching for your own public footprint.
- [ ] Document what you find — this is literally what a professional "external recon" engagement produces.

### Phase 5 — Scanning (Metasploitable2 target)

- [ ] Host discovery: `nmap -sn <lab-subnet>`
- [ ] Full port + service scan against Metasploitable2's IP.
- [ ] Run `-sC` default scripts, then manually target scripts for each discovered service (FTP, SSH, SMTP, HTTP).
- [ ] Document every open port, service, and version in a table (this is literally a pentest report skeleton).

### Phase 6 — Enumeration & Beyond

- [ ] Once comfortable with scanning, move into a **dedicated Metasploit Framework module** covering `msfconsole` workflow, module search, and safe exploitation practice against Metasploitable2's known-vulnerable services — happy to build that as a follow-up doc once you've worked through this foundation.

---

_Document compiled for personal university coursework and authorized lab practice only (own systems, own websites, Metasploitable2 VM). Always confirm explicit written authorization before testing any system you do not own._
