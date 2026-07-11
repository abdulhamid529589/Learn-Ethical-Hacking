# Sherians Cyber School — VAPT Series, Part 2: Scanning & Enumeration

**Instructor:** Abhishek Chourasia
**Channel:** Sherians Cyber School
**Topic:** Turning Raw Reconnaissance Data into a Validated Scan List, Host Discovery, Port Scanning Methodology (Nmap Scan Types), Service/Version/OS Detection, NSE Scripting, and Service-Specific Enumeration (SMB, SMTP, FTP, SSH, Web)

> **Note on this document:** The original video was delivered in Hindi (with English technical terms mixed in) as Part 2 of a 3-part VAPT series. This document translates and organizes the lecture into structured English notes for study purposes. All practical/hands-on examples in this document use **generic placeholders** (`<TARGET_IP>`, `target.tld`) or reference **Metasploitable2** and **DVWA** — both of which are widely used, publicly distributed, _intentionally vulnerable_ training virtual machines created specifically for people to legally practice these exact techniques on, in their own local lab environment. No real third-party credentials, private data, or unauthorized targets from the source video are reproduced here. As in Part 1, the instructor repeatedly frames all of this as **authorized, professional security testing methodology** — this document preserves that framing throughout.

---

## 📑 Table of Contents

1. [From Reconnaissance to Scanning: Why Raw Recon Data Isn't Ready to Scan](#1-from-reconnaissance-to-scanning-why-raw-recon-data-isnt-ready-to-scan)
2. [What Recon Actually Gives You](#2-what-recon-actually-gives-you)
3. [Turning Recon Output Into a Scan Target](#3-turning-recon-output-into-a-scan-target)
   - 3.1 [Convert Domains to IP Addresses](#31-convert-domains-to-ip-addresses)
   - 3.2 [Subdomain Validation](#32-subdomain-validation)
   - 3.3 [Removing CDN Noise / Identifying the Real Origin Server](#33-removing-cdn-noise--identifying-the-real-origin-server)
   - 3.4 [Scope Boundary Confirmation](#34-scope-boundary-confirmation)
4. [Target Validation Before Scanning](#4-target-validation-before-scanning)
5. [Building a Structured Target File](#5-building-a-structured-target-file)
6. [Professional Discipline: A Real-World Audit Story](#6-professional-discipline-a-real-world-audit-story)
7. [Practical: Host Discovery](#7-practical-host-discovery)
   - 7.1 [Windows Tools: Angry IP Scanner & Advanced IP Scanner](#71-windows-tools-angry-ip-scanner--advanced-ip-scanner)
   - 7.2 [Linux Tools: arp-scan, netdiscover, fping, Nmap Discovery](#72-linux-tools-arp-scan-netdiscover-fping-nmap-discovery)
8. [Practical: Port Scanning (Nmap Scan Types in Depth)](#8-practical-port-scanning-nmap-scan-types-in-depth)
9. [Practical: Service and Version Detection](#9-practical-service-and-version-detection)
10. [Practical: OS Detection](#10-practical-os-detection)
11. [Practical: Aggressive Scanning and Timing Templates](#11-practical-aggressive-scanning-and-timing-templates)
12. [Practical: Nmap Scripting Engine (NSE) and Vulnerability Scanning](#12-practical-nmap-scripting-engine-nse-and-vulnerability-scanning)
13. [Practical: Manual Banner Grabbing](#13-practical-manual-banner-grabbing)
14. [Practical: Service-Specific Enumeration](#14-practical-service-specific-enumeration)
    - 14.1 [SMB Enumeration](#141-smb-enumeration)
    - 14.2 [SNMP Enumeration](#142-snmp-enumeration)
    - 14.3 [FTP Enumeration](#143-ftp-enumeration)
    - 14.4 [SSH Enumeration](#144-ssh-enumeration)
15. [Practical: Web Enumeration](#15-practical-web-enumeration)
16. [Recommended Practice Machines](#16-recommended-practice-machines)
17. [Summary of Key Takeaways](#17-summary-of-key-takeaways)
18. [Quick-Reference Glossary](#18-quick-reference-glossary)

---

## 1. From Reconnaissance to Scanning: Why Raw Recon Data Isn't Ready to Scan

- Before attempting to exploit any website, server, or network, a tester needs the underlying **knowledge and assets** first — and that's exactly what **scanning and enumeration** provide.
- This lecture goes deep into **how** to properly perform that process, building directly on Part 1's reconnaissance output.

[⬆ Back to top](#-table-of-contents)

---

## 2. What Recon Actually Gives You

Reconnaissance (Part 1) produces a **large volume of raw data**, including:

- A lot of **IP addresses**, **subdomains**, and **CDN** references.
- **DNS servers**, **backup servers**, and assets deployed across multiple locations.
- Some **stale/irrelevant logs** and **inactive subdomains** that may not even be running anymore.
- DNS records, URLs, and more subdomains.

> **Key insight:** This raw output is **not directly suitable for scanning**, because it doesn't give you clean, verified data to work from. A professional VAPT tester or pentester always includes a **target preparation** step before scanning — this isn't necessarily a rigid checklist, but it follows a natural flow:

**Core flow:** Recon data → **Filter** targets → **Validate** targets (which ones can actually be attacked) → Build a **structured scan list** → Begin scanning and further enumeration.

[⬆ Back to top](#-table-of-contents)

---

## 3. Turning Recon Output Into a Scan Target

### 3.1 Convert Domains to IP Addresses

> **Core principle: scanning happens against hosts (IP addresses), not just names.**

**Analogy used:** If you save a contact in your phone as "Abhishek" with a phone number, calling "Abhishek" and calling the number reach the same person — but if you only search your contacts by name, you won't get anywhere without the underlying number actually being stored. Similarly, **you cannot scan a domain name directly** — you need to resolve it to an actual **IP address** first.

- Use **DNS resolution** to map domains/subdomains to server IPs.
- Once you have the server IP, that becomes your actual scanning target, and from there you extract further information needed for exploitation.

> **Enumeration**, in this context, can be understood as: _the process of extracting exactly the information required to perform exploitation._

### 3.2 Subdomain Validation

> A single domain can have **many subdomains** — but not all of them are necessarily worth testing or even still active.

**Subdomain validation accounts for:**

- **Expired records**
- **Parked subdomains** (registered but unused — see below)
- **Sinkholes** (explained below)
- **Tool-generated false positives** — automated tools sometimes surface irrelevant results that need to be manually verified (e.g., by visiting the URL in a browser and checking what's actually there).

**Parked domains:** Some organizations register domain names they intend to use for a future product or idea, but never actually deploy anything on them — the DNS server exists and resolves, but there's no real content/service behind it. (The instructor mentions personally owning around 15–17 such unused domains from past project ideas.)

**Sinkholes:** A DNS server can be configured so that if a malicious-looking request comes in, it gets redirected away from the main/real domain to a different, "safe" domain instead — essentially a defensive technique to protect the real infrastructure from suspicious traffic.

### 3.3 Removing CDN Noise / Identifying the Real Origin Server

- Some targets are protected by a **CDN** (e.g., Cloudflare). If you scan such a target directly, you'll often just get the **CDN's IP address**, not the actual origin server behind it.
- **You need to figure out** whether you're looking at CDN infrastructure or the real host, and if it's CDN-protected, work out how to identify the **actual origin server's IP** for meaningful scanning.
- Broader goal: build a full picture — a kind of "roadmap" — of exactly how the target's network is laid out, and what's deployed where.

### 3.4 Scope Boundary Confirmation

> **Only keep targets that are inside the authorized scope.**

- Before any VAPT engagement, the client typically defines **scope** — which specific domains/subdomains/assets you are and are not permitted to test.
- **Example:** Bug bounty platforms (e.g., a program listing on a platform like HackerOne) explicitly list a **"Scope"** section, defining exactly which subdomains are in-bounds for testing.
- This same principle applies in professional VAPT engagements: the client will specify exactly what's shared/authorized infrastructure, what's explicitly out-of-scope or blacklisted, and how that should be handled/tested for confirmation.

> **Core principle: "Recon gives you quantity, scanning needs quality."** Recon produces a large amount of raw data, but scanning requires filtering that down to only what's genuinely relevant and useful.

[⬆ Back to top](#-table-of-contents)

---

## 4. Target Validation Before Scanning

> **Target validation** means confirming that targets are actually **alive and reachable** before scanning them (e.g., a basic reachability/ping-style check).

**Why this matters:**

- **Avoids wasting time** scanning targets where nothing meaningful will be found anyway.
- **Reduces unnecessary traffic** generated during testing.
- **Prevents noisy/irrelevant results** — building a clear structure ahead of time avoids clutter.
- **Improves scan accuracy** — a scanning tool will simply report whatever data it can get from the internet/network, but you must ensure your scan is actually operating **accurately**.
- **Maintains professional testing discipline** (see the real-world story in Section 6).

**What must be verified:**

- **Domain validation:** Does the domain resolve? Does it respond over HTTP and HTTPS? (If not, it likely isn't active anyway.)
- **IP reachability:** Is the host reachable at the network level? Does it respond to probes (e.g., ping)?
- **Subdomain resolution:** Does the subdomain return a valid DNS record, and does it map to a real host?

> **Only after validation should confirmed live targets move forward into the scanning phase.**

[⬆ Back to top](#-table-of-contents)

---

## 5. Building a Structured Target File

> Before scanning, build a **structured target file** (e.g., `targets.txt`) to keep everything organized — since this information will eventually need to go into a report.

**What to include:**

- Verified domains
- Live domains
- Reachable domains
- In-scope assets

**What NOT to include:**

- Dead hosts
- CDN-only endpoints (no value in scanning these directly)
- Out-of-scope assets
- Duplicate entries

**Suggested file structure — one target per line**, e.g.:

```
main domain
API subsystem
staging environment
<IP address>
```

**Benefits of this approach:**

- **Batch execution** — scan one target, then the next, then the next, automatically, in sequence.
- **Repeatable scans** — re-run the same scan against the same list without redoing setup each time.
- **Easy retesting** — makes repeated verification simple.
- **Team-collaboration friendly** — everyone on a team knows exactly where to find which asset, since it's all documented in one place, rather than everyone tracking things separately.

> This becomes especially important during **professional methodology** work — when auditing a large network, things can quickly become disorganized without this kind of structure.

[⬆ Back to top](#-table-of-contents)

---

## 6. Professional Discipline: A Real-World Audit Story

The instructor shares a personal anecdote to illustrate the importance of **professional conduct** during an on-site audit (around age 17–18, during one of his first internal bank audits):

- When arriving on-site, auditors are typically given a **dedicated room** to work from (similar to how a financial audit team gets its own space).
- When asked to "take access to the network," the instructor initially and naively asked for the **Wi-Fi password** — which was met with a strong reprimand from a senior colleague.
- **The lesson:** Organizations being audited generally do **not** hand over Wi-Fi access directly. Instead, they typically provide a **physical LAN cable** (sometimes with a specific connector type), and even then, the network may use **static IP assignment** (no DHCP) rather than automatically assigning an address.
- **Broader lesson:** There is a proper, professional **manner and etiquette** for how to conduct yourself during an engagement — how you approach people, how you speak, and how you carry yourself — since testers are actively gathering sensitive information and should do so in a way that doesn't make people (understandably) suspicious or uncomfortable. Working calmly and with discipline also reduces the tester's own mistakes.

[⬆ Back to top](#-table-of-contents)

---

## 7. Practical: Host Discovery

> **Goal:** Determine which hosts are actually "alive" within a given IP range or target network, before scanning them further.

### 7.1 Windows Tools: Angry IP Scanner & Advanced IP Scanner

**Angry IP Scanner:**

- Lets you select an IP range (or import a text file of IP addresses) and scan it.
- Can be configured to scan for specific ports (e.g., FTP on 20/21, SSH on 22) using ICMP or other probe types to determine liveness.
- Option: **"Scan alive hosts and hosts with open ports only"** — filters results down to just what's actually useful, which is handy for quickly copying live IPs into a report.
- Result colors: dead/inactive hosts appear differently from currently live/active hosts.

**Advanced IP Scanner:**

- Offers **richer detail** than Angry IP Scanner: MAC address, hardware vendor/manufacturer, hostname, NetBIOS group membership, and general device status.
- Useful for quickly identifying **what kind of device** each live IP actually is (e.g., a laptop, phone, or printer) based on vendor/hostname data.
- The instructor personally prefers this tool over Angry IP Scanner specifically because of this richer functionality.

**A cautionary real-world illustration (network printer discovery):** During the demo, an open network printer was discovered this way and briefly accessed via its web login — illustrating a broader, important point:

> **Why printer access matters in real audits:** In organizations where sensitive documents are frequently printed (e.g., banks — balance sheets, bank statements, confidential records), a printer left open/accessible on the network can be "hijacked" to intercept or access those documents. The instructor notes personally having discovered exposed printers in real banking audits and reporting this finding directly to leadership as a serious risk.

### 7.2 Linux Tools: arp-scan, netdiscover, fping, Nmap Discovery

**Setup note:** For internal network discovery, the tester's VM network adapter is typically set to **Bridged** mode, so it shares the same local network visibility as the host machine.

```bash
# ARP-based discovery — fast, but only works on the *local* network segment
sudo arp-scan --localnet

# netdiscover — similar goal, but continuously listens for devices joining/leaving
# (unlike arp-scan, which is a one-time snapshot)
sudo netdiscover

# netdiscover — passive mode (generates minimal traffic to the network)
sudo netdiscover -p

# netdiscover — scan a specific range (CIDR notation)
sudo netdiscover -r 192.168.1.0/24

# fping — ICMP-based liveness check across a whole range, showing only alive hosts
fping -a -g 192.168.1.1 192.168.1.254 --alive

# fping — save just the list of live hosts to a file for later use
fping -a -g 192.168.1.0/24 --alive > live.txt

# Feed that live-host list directly into an nmap scan
nmap -iL live.txt

# Nmap's own ICMP-based discovery scan (no port scan yet, just host discovery)
nmap -sn 192.168.1.0/24

# Nmap TCP-based discovery — useful when ICMP is blocked by a firewall
# (sends a SYN probe to specific ports to check for a response)
nmap -sn -PS22,80,443 192.168.1.0/24

# Nmap TCP ACK-based discovery — an alternative probe type
nmap -sn -PA 192.168.1.0/24

# Quick single-host discovery scan
nmap 192.168.1.50
```

**Why use multiple discovery methods?** If ICMP (ping) is blocked by a firewall, a host might incorrectly appear "down" using one method — but a different probe type (TCP SYN, TCP ACK, etc.) may still get a response, correctly revealing that the host is actually alive. Having several techniques on hand avoids false conclusions.

**Finding your own network's CIDR range:**

```bash
ip a          # Linux — look for the "inet" line and its /XX suffix
ifconfig      # Windows/older Linux — look for the subnet mask
```

**Quick CIDR-to-host-count reference:**
| CIDR Suffix | Approximate Number of IP Addresses |
|---|---|
| `/24` | ~256 |
| `/16` | ~65,536 |
| `/8` | ~16.7 million (⚠️ never scan a range this large "blindly" — it can overwhelm your machine or the network) |

[⬆ Back to top](#-table-of-contents)

---

## 8. Practical: Port Scanning (Nmap Scan Types in Depth)

> Once live hosts are confirmed, the next step is determining **which ports are open** on each one. There are three possible port states: **open, closed,** and **filtered** (reachable, but something like a firewall is interfering).

The instructor demonstrates these techniques against **Metasploitable2** — a deliberately vulnerable practice VM built specifically for this kind of training (freely downloadable; instructions in Section 16).

```bash
# --- TCP Connect Scan ---
# Completes a full TCP handshake with each port; reliable but "noisier"
# (more easily logged/detected by a monitoring target)
nmap -sT <TARGET_IP>

# --- UDP Scan ---
# No handshake involved; can be slower and less certain,
# but useful since many services (DNS, SNMP, etc.) run over UDP
nmap -sU <TARGET_IP>

# --- NULL Scan ---
# Sends a packet with NO TCP flags set at all
nmap -sN <TARGET_IP>

# --- FIN Scan ---
# Sends a packet with only the FIN flag set
nmap -sF <TARGET_IP>

# --- XMAS Scan ---
# Sends a packet with FIN, PSH, and URG flags all set at once
# (the flags "light up" like a Christmas tree — hence the name)
nmap -sX <TARGET_IP>

# --- Custom TCP flag scan ---
# Manually specify exactly which flags to send
nmap --scanflags URGACKPSHRSTSYNFIN <TARGET_IP>
nmap --scanflags SYN <TARGET_IP>      # equivalent to a SYN scan
nmap --scanflags ACK <TARGET_IP>      # ACK-only probe

# --- IP Protocol Scan ---
# Tests which IP-layer protocols (ICMP, TCP, UDP, GRE, etc.) are supported
nmap -sO <TARGET_IP>

# --- Specify a source interface manually ---
nmap -e eth0 <TARGET_IP>
```

**Why learn so many different scan types?** Firewalls sometimes block **specific** flag combinations (e.g., FIN packets) while allowing others (e.g., SYN) through. Having multiple scan techniques available means that if one approach is blocked or gives no result, another can often still succeed — this flexibility is especially valuable when a target has an active firewall or "blue team" defensive monitoring in place.

**Monitoring your own scan's footprint:** The instructor demonstrates watching a scan's traffic live in **Wireshark** while running it, specifically to visualize how much traffic a scan generates and how easily a defender could notice it (e.g., a flood of packets from one source IP to one destination IP would stand out clearly in a monitoring tool).

[⬆ Back to top](#-table-of-contents)

---

## 9. Practical: Service and Version Detection

> After finding which ports are open, the next goal is identifying **exactly which service and version** is running on each one.

**Analogy used:** First you confirm a house exists (host discovery). Then you find which doors are open (port discovery). Now you need to know **what's behind each door** — what's actually stored there, and what type it is. That's service/version detection.

```bash
# Service and version detection
nmap -sV <TARGET_IP>
```

**Why version numbers matter so much:** An outdated service version may have a **known, publicly documented exploit** available for it. Once you know the exact version (e.g., a specific FTP or SSH daemon version), you can research whether a matching public exploit/payload exists for that specific release — this is a core part of preparing for the exploitation phase covered in Part 3 of the series.

[⬆ Back to top](#-table-of-contents)

---

## 10. Practical: OS Detection

```bash
# Attempt to fingerprint the target's operating system
nmap -O <TARGET_IP>
```

- This relies on **TCP/IP stack fingerprinting** — subtle differences in how different operating systems implement networking allow nmap to make an educated guess (e.g., a specific Linux kernel version range), along with an estimate of the **hop count** to the target and its detected **MAC address** (when on the same local network).
- Nmap also tags a general **"device type"** classification (e.g., general purpose).

[⬆ Back to top](#-table-of-contents)

---

## 11. Practical: Aggressive Scanning and Timing Templates

```bash
# Aggressive scan: combines OS detection, version detection,
# script scanning, and traceroute all in one command
nmap -A <TARGET_IP>

# Explicit combination of the individual pieces "-A" bundles together
nmap -sV -O --script=default --traceroute <TARGET_IP>
```

**Timing templates (`-T0` through `-T5`):** Control how fast/aggressively a scan runs.

| Template  | Speed                | Typical Use                                                           |
| --------- | -------------------- | --------------------------------------------------------------------- |
| `T0`–`T1` | Very slow            | Maximum stealth, avoiding detection                                   |
| `T2`      | Slow                 | Cautious testing                                                      |
| `T3`      | **Default**          | Balanced — the nmap default                                           |
| `T4`      | Fast                 | Common choice when quicker results are acceptable                     |
| `T5`      | Very fast/aggressive | Fastest, but noisiest — highest chance of detection or missed results |

```bash
# Example: run a fast, less time-consuming scan
nmap -T4 -sV -O <TARGET_IP>

# Manually control delay between probes instead of using a template
nmap --scan-delay 1s <TARGET_IP>
```

> **Demonstrated timing difference:** In the lecture, the same scan took roughly **25 seconds at T4** versus a much longer duration at the default template — illustrating a real, measurable trade-off between speed and "noise"/thoroughness.

[⬆ Back to top](#-table-of-contents)

---

## 12. Practical: Nmap Scripting Engine (NSE) and Vulnerability Scanning

> Nmap ships with a large library of scripts (the **Nmap Scripting Engine**, or NSE) covering categories like **discovery, vulnerability ("vuln"), intrusive, auth,** and **safe** scripts.

```bash
# Locate the NSE script library on your system
ls /usr/share/nmap/scripts/

# Search for scripts related to a specific service (example: SMB)
ls /usr/share/nmap/scripts/ | grep smb

# Run ALL vulnerability-detection scripts against a target
nmap --script vuln <TARGET_IP>

# Run all scripts for a specific service category (example: HTTP)
nmap -p 80 --script "http-*" <TARGET_IP>

# Run all SMB-related scripts
nmap -p 139,445 --script "smb-*" <TARGET_IP>

# Run a specific single script
nmap --script smb-os-discovery -p 139,445 <TARGET_IP>
```

**What running a `vuln` scan against Metasploitable2 typically reveals** (a deliberately vulnerable machine, so this is expected/by design):

- References to publicly known vulnerability identifiers.
- Discovered web paths/URLs that may be exploitable (e.g., pages vulnerable to **SQL injection**), to be investigated and exploited in the next phase of the series.
- A discovered login page for **DVWA (Damn Vulnerable Web Application)** — another intentionally vulnerable practice application, often bundled alongside Metasploitable-style training environments, useful for practicing web-application-specific attacks like brute-forcing a login form.

> **Traffic footprint consideration:** Running many NSE scripts at once generates more traffic and takes longer — but as observed via Wireshark in the demo, it can still be a relatively **light** footprint compared to something like a full brute-force attack, though a defensive team could of course still notice it depending on their firewall rules and monitoring sensitivity.

[⬆ Back to top](#-table-of-contents)

---

## 13. Practical: Manual Banner Grabbing

> Beyond automated tools, **manually connecting** to a service and reading its response ("banner") is a valuable, low-noise technique — and helps avoid relying purely on automation, which can sometimes produce false positives.

```bash
# Netcat — connect directly to a port and read whatever banner is returned
nc -nv <TARGET_IP> 21      # FTP
nc -nv <TARGET_IP> 22      # SSH
nc -nv <TARGET_IP> 25      # SMTP
nc -nv <TARGET_IP> 80      # HTTP

# Telnet — an older but still useful tool for manual banner grabbing
telnet <TARGET_IP> 21
telnet <TARGET_IP> 23      # if Telnet service itself is open
telnet <TARGET_IP> 25

# curl — grab HTTP headers/response directly
curl -v http://<TARGET_IP>/

# openssl s_client — connect to an SSL/TLS-enabled service and inspect its certificate
openssl s_client -connect <TARGET_IP>:443
openssl s_client -connect <TARGET_IP>:443 -tls1_2
```

**Reading the results:** A banner often reveals the exact **software name and version** running on that port (e.g., a specific FTP daemon or mail server), which directly feeds into the version-based research described in Section 9.

[⬆ Back to top](#-table-of-contents)

---

## 14. Practical: Service-Specific Enumeration

Once a specific port/service is confirmed open, the next step is enumerating that **service** in depth.

### 14.1 SMB Enumeration

```bash
# enum4linux — comprehensive SMB enumeration (users, groups, RID ranges, shares, OS info)
enum4linux <TARGET_IP>
enum4linux -a <TARGET_IP>          # run all checks

# smbclient — list available shares
smbclient -L //<TARGET_IP>/ -N     # -N = no password (tests for anonymous/null session)

# smbclient — connect using an anonymous account
smbclient //<TARGET_IP>/SHARE_NAME -U ''%''

# smbclient — connect using a "guest" account
smbclient //<TARGET_IP>/SHARE_NAME -U 'guest'%''

# rpcclient — connect via RPC with a null session
rpcclient -U "" <TARGET_IP>
# once connected, useful sub-commands include:
#   srvinfo         (server info)
#   enumdomusers    (enumerate domain users)
#   querydominfo    (domain info)
#   help            (list all available commands)

# Nmap SMB script sweep
nmap -p 139,445 --script "smb-enum-*" <TARGET_IP>
```

**What this typically reveals (on a vulnerable practice target):** Domain/RID ranges, known default account names (Administrator, root, guest, krbtgt, etc.), domain groups, whether a null/blank-credential session is accepted, a list of real usernames, and available shared folders.

### 14.2 SNMP Enumeration

```bash
# snmpwalk — bulk data retrieval using the default "public" community string
snmpwalk -v2c -c public <TARGET_IP>

# snmp-check — a more human-readable summary of the same information
snmp-check <TARGET_IP>

# Nmap SNMP script
nmap -sU -p 161 --script snmp-info <TARGET_IP>
```

- If the port doesn't respond, that simply means the **SNMP service isn't running/exposed** on that host — not every target will have it open.

### 14.3 FTP Enumeration

```bash
# Manually connect and attempt an anonymous login
ftp <TARGET_IP>
# When prompted:
#   Username: anonymous
#   Password: (leave blank, or use a placeholder email address)

# Once connected:
ls              # list files in the current directory
get filename    # download a file for inspection

# Nmap FTP scripts
nmap -p 21 --script ftp-anon <TARGET_IP>
nmap -p 21 --script "ftp-*" <TARGET_IP>
```

- A successful **anonymous login** on an FTP server is itself a notable finding, since it means files may be readable (or even writable) without any authentication at all.

### 14.4 SSH Enumeration

```bash
# Manual banner grab
nc -nv <TARGET_IP> 22

# Nmap SSH-related scripts (e.g., checking supported algorithms/versions)
nmap -p 22 --script "ssh2-enum-algos,ssh-hostkey" <TARGET_IP>
```

- Reveals the SSH server software/version, which — as with any service — should be checked against known vulnerabilities for that specific version.

[⬆ Back to top](#-table-of-contents)

---

## 15. Practical: Web Enumeration

```bash
# subfinder — passive subdomain discovery for a given domain
subfinder -d target.tld

# dirb — directory/content brute-forcing using a wordlist
dirb http://target.tld/
dirb http://target.tld/ /path/to/wordlist.txt

# gobuster — a faster alternative for the same purpose
gobuster dir -u http://target.tld/ -w /path/to/wordlist.txt

# whatweb — fingerprint the web technology stack in use
whatweb target.tld
```

> **Note:** The instructor demonstrates `dirb`/directory-brute-forcing tools conceptually using his own company's public website purely to show the _tool's behavior_ (finding directories that already exist on a site he owns/controls) — this is **not** a demonstration against an unauthorized target. As always, only run these tools against domains you own or are explicitly authorized to test.

[⬆ Back to top](#-table-of-contents)

---

## 16. Recommended Practice Machines

The instructor recommends the following **legally downloadable, intentionally vulnerable** practice environments to reinforce everything covered in this lecture:

| Machine/Resource                           | Focus Area                                                                                                                                                                                                    |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Metasploitable2**                        | General multi-service target (FTP, SSH, SMB, HTTP, SMTP, and more) — the primary machine used throughout this lecture; freely downloadable (search "Metasploitable2 download", official Rapid7-hosted mirror) |
| **DVWA (Damn Vulnerable Web Application)** | Web application vulnerabilities (SQLi, brute force, etc.) — often bundled alongside Metasploitable-style setups                                                                                               |
| A dedicated SMB-focused practice VM        | SMB enumeration/exploitation practice specifically                                                                                                                                                            |
| **TryHackMe**                              | Guided, beginner-friendly rooms organized by topic/service, including network service enumeration                                                                                                             |
| **HackTheBox**                             | A large library of practice machines across all skill levels                                                                                                                                                  |

**Basic setup steps (as demonstrated):**

1. Download the practice VM's compressed file.
2. Extract it.
3. In VMware/VirtualBox, choose **Open** (not "New Virtual Machine") and select the extracted `.vmx` file.
4. Set the VM's network adapter to **Bridged** mode so it shares your local network for discovery/scanning practice, or **NAT/Host-only** for a fully isolated lab.

[⬆ Back to top](#-table-of-contents)

---

## 17. Summary of Key Takeaways

1. Raw **reconnaissance data** (domains, subdomains, IPs, CDN references, old records) is **not directly scannable** — it must first be **filtered, validated, and organized** into a clean target list.
2. **Scanning targets hosts (IP addresses), not domain names** — DNS resolution is the bridge between the two.
3. **Subdomain validation** filters out expired records, parked domains, sinkholes, and tool-generated false positives.
4. **CDN-protected targets** require identifying the real origin server, not just the CDN's own IP.
5. **Scope boundary confirmation** ensures only explicitly authorized assets are tested — critical in any professional engagement.
6. **Target validation** (confirming hosts are alive/reachable) prevents wasted time, unnecessary traffic, and noisy/inaccurate results.
7. A **structured target file** (one target per line, verified/live/in-scope only) enables batch scanning, easy retesting, and team collaboration.
8. **Professional discipline and etiquette** matter during real on-site audits — engagements have proper protocols (e.g., physical network access rather than Wi-Fi credentials) that testers must respect.
9. **Host discovery** can be done via GUI tools (Angry IP Scanner, Advanced IP Scanner) on Windows, or via `arp-scan`, `netdiscover`, `fping`, and `nmap -sn` on Linux — with multiple techniques available in case one probe type (e.g., ICMP) is blocked by a firewall.
10. **Port scanning** has many nmap variants — TCP connect (`-sT`), UDP (`-sU`), NULL (`-sN`), FIN (`-sF`), XMAS (`-sX`), custom flag scans, and IP protocol scans (`-sO`) — useful because firewalls may block some flag combinations while allowing others through.
11. **Service/version detection** (`-sV`) and **OS detection** (`-O`) reveal exactly what's running, which is essential for researching known vulnerabilities in the next phase.
12. **Timing templates** (`-T0` to `-T5`) trade off speed against stealth/thoroughness; `-A` bundles OS detection, version detection, script scanning, and traceroute together.
13. The **Nmap Scripting Engine (NSE)** includes categories like vulnerability scanning (`--script vuln`), and service-specific script sweeps (e.g., `smb-*`, `http-*`) that can reveal known issues and exploitable paths.
14. **Manual banner grabbing** (via `nc`, `telnet`, `curl`, `openssl s_client`) is a valuable, low-noise complement to automated scanning, helping avoid false positives.
15. **Service-specific enumeration** techniques were covered for **SMB** (`enum4linux`, `smbclient`, `rpcclient`), **SNMP** (`snmpwalk`, `snmp-check`), **FTP** (anonymous login testing), and **SSH** (banner/algorithm enumeration).
16. **Web enumeration** tools (`subfinder`, `dirb`/`gobuster`, `whatweb`) help map out subdomains, hidden directories, and the underlying web technology stack.
17. All of this should be **practiced on legal, intentionally vulnerable lab machines** — Metasploitable2, DVWA, and platforms like TryHackMe/HackTheBox — never against systems you don't own or lack explicit authorization to test.

[⬆ Back to top](#-table-of-contents)

---

## 18. Quick-Reference Glossary

| Term                                       | Definition                                                                                               |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| **Target validation**                      | Confirming a target is alive and reachable before scanning it                                            |
| **Parked domain**                          | A registered domain with no active service/content deployed on it                                        |
| **Sinkhole**                               | A DNS configuration that redirects suspicious/malicious requests away from the real infrastructure       |
| **CDN (Content Delivery Network)**         | Infrastructure that can mask a target's true origin server IP                                            |
| **Scope**                                  | The explicitly authorized set of assets a tester is permitted to test                                    |
| **Host discovery**                         | Identifying which hosts on a network/range are currently active                                          |
| **Port state (open/closed/filtered)**      | Whether a port is accepting connections, rejecting them, or being silently blocked (e.g., by a firewall) |
| **TCP connect scan (`-sT`)**               | A full TCP handshake-based port scan                                                                     |
| **SYN scan**                               | A "half-open" scan sending only a SYN packet                                                             |
| **NULL / FIN / XMAS scan**                 | Scans using unusual TCP flag combinations, sometimes used to evade certain firewall rules                |
| **Service/version detection (`-sV`)**      | Identifying the exact software and version running on an open port                                       |
| **OS detection (`-O`)**                    | Fingerprinting a target's likely operating system based on TCP/IP stack behavior                         |
| **Timing template (`-T0`–`-T5`)**          | Nmap settings controlling scan speed vs. stealth                                                         |
| **NSE (Nmap Scripting Engine)**            | Nmap's built-in library of scripts for discovery, vulnerability detection, and more                      |
| **Banner grabbing**                        | Manually connecting to a service to read its identifying response/banner                                 |
| **Metasploitable2**                        | A widely used, intentionally vulnerable practice virtual machine                                         |
| **DVWA (Damn Vulnerable Web Application)** | An intentionally vulnerable web application used for practicing web attack techniques                    |

[⬆ Back to top](#-table-of-contents)

---

_End of Part 2 — Part 3 of the series continues with exploiting the vulnerabilities and findings uncovered during this scanning and enumeration phase._
