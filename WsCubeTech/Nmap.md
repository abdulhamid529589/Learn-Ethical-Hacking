# Nmap — Complete Crash Course Notes (Network Scanning)

## https://youtu.be/V4fhsPlPMWE

> **Course:** Cyber Security & Ethical Hacking — Network Scanning Phase
> **Topic:** Nmap (Network Mapper) — Basic to Advanced

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [The 5 Phases of Ethical Hacking](#2-the-5-phases-of-ethical-hacking)
3. [What is Nmap?](#3-what-is-nmap)
4. [Understanding Ports](#4-understanding-ports)
   - 4.1 [Types of Ports](#41-types-of-ports)
   - 4.2 [Common Ports & Services](#42-common-ports--services)
   - 4.3 [Port Number Ranges](#43-port-number-ranges)
5. [TCP Flags](#5-tcp-flags)
6. [TCP Three-Way Handshake](#6-tcp-three-way-handshake)
7. [Port States in Nmap](#7-port-states-in-nmap)
8. [Target Specification](#8-target-specification)
9. [Port Specification](#9-port-specification)
10. [Host Discovery](#10-host-discovery)
    - 10.1 [Practical: Finding Live Hosts](#101-practical-finding-live-hosts)
11. [Nmap Scan Types](#11-nmap-scan-types)
    - 11.1 [TCP Connect Scan (-sT)](#111-tcp-connect-scan--st)
    - 11.2 [TCP SYN Scan (-sS)](#112-tcp-syn-scan--ss)
    - 11.3 [UDP Scan (-sU)](#113-udp-scan--su)
    - 11.4 [Stealth Scans: NULL, FIN, Xmas](#114-stealth-scans-null-fin-xmas)
    - 11.5 [ACK Scan (-sA)](#115-ack-scan--sa)
    - 11.6 [Scan Type Flags — Quick Reference](#116-scan-type-flags--quick-reference)
12. [OS Detection](#12-os-detection)
13. [Service Version Detection](#13-service-version-detection)
14. [Timing Templates](#14-timing-templates)
15. [Firewall / IDS Evasion Techniques](#15-firewall--ids-evasion-techniques)
16. [Output Formats](#16-output-formats)
17. [Putting It All Together — Full Command Example](#17-putting-it-all-together--full-command-example)
18. [Legal & Ethical Use](#18-legal--ethical-use)
19. [Summary / Quick Revision Table](#19-summary--quick-revision-table)

---

## 1. Introduction

Before any hacker or security professional scans a network, the very first step is **Reconnaissance** — gathering information about the target. Once basic recon is done, the next step is **Network Scanning**, and the most powerful tool for this job is **Nmap**.

This crash course covers, from scratch to advanced level:

- **Host Discovery** — which devices are active on a network
- **Port Scanning** — which ports are open, closed, or filtered
- **OS Detection** — what operating system the target is running
- **Service Version Detection** — exactly which service (and version) is running on each port
- Real commands with detailed explanations of what happens internally

---

## 2. The 5 Phases of Ethical Hacking

Nmap is used in the **second phase** of ethical hacking. The five phases are:

| Phase                    | Description                                                                     |
| ------------------------ | ------------------------------------------------------------------------------- |
| **1. Reconnaissance**    | Gathering initial information about the target                                  |
| **2. Network Scanning**  | Scanning the target's network, ports, OS, services (this is where Nmap is used) |
| **3. Exploitation**      | Actually attacking the target to gain access                                    |
| **4. Post-Exploitation** | What to do _after_ successfully gaining access                                  |
| **5. Reporting**         | Documenting everything done, results found, and remediation/patching advice     |

---

## 3. What is Nmap?

- **Nmap = Network Mapper**
- It is an **open-source network discovery and security auditing tool**.
  - _Open source_ means its source code is publicly available — anyone can view, modify, and use it according to their needs.
- Created by **Gordon Lyon** in **1997**.
- Used by: **system administrators, penetration testers, network engineers** worldwide.
- **Core capabilities:**
  - Host Discovery
  - Port Scanning
  - Service Detection
  - Operating System Fingerprinting

**Simple analogy:** Imagine you (the attacker/security professional) are on a LAN with multiple other connected devices. Nmap lets your machine scan those other devices to find out things like: which operating system they're running, which ports are open/closed, and what services are active on them.

**Prerequisite:** To understand network scanning properly, you must already understand basic networking concepts (ports, protocols, subnetting, etc.).

---

## 4. Understanding Ports

A **port** is like a "door" into a device — to establish any connection with a device, you need to go through a specific port.

### 4.1 Types of Ports

1. **Physical Ports** — physical connection points (e.g., Ethernet ports)
2. **Virtual/Logical Ports** — software-level ports used for network communication (what Nmap scans)

### 4.2 Common Ports & Services

| Port | Protocol | Service                      | Notes                          |
| ---- | -------- | ---------------------------- | ------------------------------ |
| 21   | TCP      | FTP (File Transfer Protocol) | Unencrypted file transfer      |
| 22   | TCP      | SSH (Secure Shell)           | Encrypted remote access        |
| 23   | TCP      | Telnet                       | Unencrypted remote access      |
| 25   | TCP      | SMTP                         | Email sending                  |
| 53   | TCP/UDP  | DNS                          | Domain Name resolution         |
| 80   | TCP      | HTTP                         | Unencrypted web traffic        |
| 443  | TCP      | HTTPS                        | Encrypted web traffic          |
| 445  | TCP      | SMB                          | File/printer sharing (Windows) |
| 3306 | TCP      | MySQL                        | Database service               |

### 4.3 Port Number Ranges

Total ports available: **65,535**

| Range             | Category                  | Description                                                                                   |
| ----------------- | ------------------------- | --------------------------------------------------------------------------------------------- |
| **0 – 1023**      | **Well-Known Ports**      | Reserved for common, standard services (HTTP, HTTPS, SSH, FTP, etc.)                          |
| **1024 – 49151**  | **Registered Ports**      | Registered for specific applications/services (e.g., MySQL, specific apps)                    |
| **49151 – 65535** | **Dynamic/Private Ports** | Temporary ports, used e.g. when a system needs to share data momentarily — these change often |

---

## 5. TCP Flags

Understanding TCP flags is **essential** before learning Nmap scan types, because Nmap scans work by sending specific combinations of these flags and interpreting the responses.

| Flag     | Full Form      | Meaning / Purpose                                                                    |
| -------- | -------------- | ------------------------------------------------------------------------------------ |
| **SYN**  | Synchronize    | "I want to connect" — sent when a device wants to initiate a connection              |
| **ACK**  | Acknowledgment | "Yes, I have received your previous packet/flag"                                     |
| **RST**  | Reset          | "Close this connection immediately"                                                  |
| **FIN**  | Finish         | "I want to end this connection gracefully" — used when all data transfer is complete |
| **PSH**  | Push           | "Deliver this data right now" — used to bypass buffering and push data immediately   |
| **URG**  | Urgent         | Marks data as high priority, to be sent/processed immediately                        |
| **NULL** | —              | No flags set at all                                                                  |

---

## 6. TCP Three-Way Handshake

The **three-way handshake** is how any standard TCP connection is established between two devices.

```
   My Machine (Attacker)                Target Device
          │                                   │
          │ ───────── SYN ──────────────────► │   "I want to connect"
          │                                   │
          │ ◄──────── SYN + ACK ────────────  │   "Got you, ready"
          │                                   │
          │ ───────── ACK ──────────────────► │   "Connected"
          │                                   │
```

1. **Step 1:** My device sends a **SYN** packet → "I want to create a connection"
2. **Step 2:** Target device replies with **SYN + ACK** → "I acknowledge, and I also want to connect"
3. **Step 3:** My device sends a final **ACK** → "Connection established"

This exact handshake is used (or deliberately _not_ completed) in different Nmap scan types, which is what differentiates them.

---

## 7. Port States in Nmap

| State          | Meaning                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------- |
| **Open**       | The target application is actively **listening** on this port ("Yes, I am listening")             |
| **Closed**     | The host is up/reachable, but **no application is listening** on this port — not ready to connect |
| **Filtered**   | A **firewall** is blocking traffic to this port, so Nmap cannot determine its true state          |
| **Unfiltered** | The port is **reachable**, but Nmap still **cannot determine** whether it's open or closed        |

**How Nmap determines port state (example logic):**

- Sends a **SYN** packet → if target replies with **SYN + ACK** → port is **OPEN**
- Sends a **SYN** packet → if target replies with **RST + ACK** → port is **CLOSED**
- If a firewall blocks the packet entirely → port is **FILTERED**

---

## 8. Target Specification

How to specify _what_ to scan in Nmap:

| Method              | Syntax Example                              | Description                                                                                                      |
| ------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Single IP**       | `nmap 192.168.1.5`                          | Scans one specific host                                                                                          |
| **Range**           | `nmap 192.168.1.1-254`                      | Scans a range of hosts (last octet varies, e.g., 1 to 254)                                                       |
| **CIDR Notation**   | `nmap 192.168.1.0/24`                       | Scans an entire subnet — `/24` means the first 3 octets (24 bits = 8+8+8) stay fixed, only the last octet varies |
| **Hostname/Domain** | `nmap scanme.nmap.org`                      | Scans a domain name directly (Nmap resolves it via DNS)                                                          |
| **File Input**      | `nmap -iL targets.txt`                      | Scans a list of IPs/hostnames stored in a text file                                                              |
| **Random Hosts**    | `nmap -iR 100`                              | Scans 100 random hosts on the internet                                                                           |
| **Exclude Hosts**   | `nmap 192.168.1.0/24 --exclude 192.168.1.5` | Scans a range but excludes specific IP(s)                                                                        |

---

## 9. Port Specification

How to specify _which ports_ to scan:

| Method             | Syntax Example    | Description                                       |
| ------------------ | ----------------- | ------------------------------------------------- |
| **Single Port**    | `-p 80`           | Scans only port 80                                |
| **Multiple Ports** | `-p 22,25,53`     | Scans only the specified list of ports            |
| **Range**          | `-p 1-1024`       | Scans ports 1 through 1024 (all well-known ports) |
| **All Ports**      | `-p-`             | Scans **all 65,535** ports                        |
| **Top N Ports**    | `--top-ports 100` | Scans the top 100 most common ports               |

> **Important interview fact:** If you don't specify any port flag at all, Nmap by default scans the **top 1000 most common ports**.

---

## 10. Host Discovery

Host discovery = finding out which devices ("hosts") on a network are actually **live/active**, before doing any port scanning.

| Flag      | Purpose                                                                                                                     |
| --------- | --------------------------------------------------------------------------------------------------------------------------- |
| **`-sn`** | Performs **only host discovery** (Ping Scan) — no port scanning at all. Fast, clean output showing only which IPs are live. |
| **`-Pn`** | **Skips host discovery** entirely and goes directly to port/service scanning — treats all hosts as if they are online.      |

### 10.1 Practical: Finding Live Hosts

Example workflow (in a local lab environment, attacker machine = Kali Linux, target = victim VM):

```bash
# Check your own IP and subnet mask
ifconfig

# Scan the whole subnet for live hosts (host discovery only, no port scan)
nmap 192.168.1.0-255 -sn
```

- Running this without `-sn` performs full host discovery **plus** default port scanning — which takes longer.
- Using `-sn` gives you a **quick, clean list of live IPs only**.
- **Tip:** IPs ending in `.0`, `.1`, `.254`, or `.255` are usually **virtual router/network addresses** — typically not your actual target, so they can generally be ignored when hunting for a specific victim machine.

Once you identify your target's IP, you can move on to scanning specific ports/services on it.

---

## 11. Nmap Scan Types

### 11.1 TCP Connect Scan (`-sT`)

**Concept:** Completes a **full TCP three-way handshake** with the target.

**Mechanism:**

1. Nmap sends **SYN**
2. Target replies **SYN + ACK**
3. Nmap sends **ACK** (handshake complete → connection established)
4. Nmap **immediately terminates** the connection right after (since the goal was only to check port status, not communicate)

| Aspect          | Details                                                                  |
| --------------- | ------------------------------------------------------------------------ |
| **Speed**       | Slow (full handshake takes time)                                         |
| **Reliability** | Highly reliable (standard protocol compliance)                           |
| **Stealth**     | Low — easily logged/detected by security systems                         |
| **Privileges**  | Does **not** require root/admin privileges — good for unprivileged users |
| **Best for**    | Users without raw packet access                                          |
| **Flag**        | `-sT`                                                                    |

**How it detects port state:**

- **Open port:** SYN → SYN+ACK → ACK → then Nmap sends **RST** to close (handshake succeeded)
- **Closed port:** SYN → target replies directly with **RST** (no handshake forms)

### 11.2 TCP SYN Scan (`-sS`)

Also called **Half-Open Scan** or **Stealth Scan**.

**Concept:** Does **NOT** complete the full three-way handshake.

**Mechanism:**

1. Nmap sends **SYN**
2. Target replies **SYN + ACK** → Nmap interprets this as "port is open" and immediately sends **RST** to tear down the connection **without** sending the final ACK
3. Handshake is deliberately left incomplete

| Aspect         | Details                                                |
| -------------- | ------------------------------------------------------ |
| **Speed**      | Faster than TCP Connect Scan                           |
| **Stealth**    | Higher — significantly less likely to be logged        |
| **Privileges** | **Requires root/admin privileges** (raw socket access) |
| **Best for**   | Initial network scanning / reconnaissance              |
| **Flag**       | `-sS`                                                  |

**How it detects port state:**

- **Open port:** SYN → SYN+ACK received → Nmap sends RST (handshake never fully completes)
- **Closed port:** SYN → target replies with **RST + ACK**

### Comparison: TCP Connect Scan vs SYN Scan

| Feature                   | TCP Connect Scan (`-sT`) | TCP SYN Scan (`-sS`) |
| ------------------------- | ------------------------ | -------------------- |
| Full handshake completed? | ✅ Yes                   | ❌ No (half-open)    |
| Root privileges required? | ❌ No                    | ✅ Yes               |
| Speed                     | Slower                   | Faster               |
| Detectability in logs     | Higher                   | Lower                |

### 11.3 UDP Scan (`-sU`)

Used specifically to scan **UDP ports** (as opposed to TCP ports).

**Why it's the slowest scan** (despite UDP itself being a "fast" protocol):

- UDP has **no handshake mechanism**
- No confirmation/acknowledgment packet is ever returned
- Nmap has to **wait for a timeout** to decide the result, since there's no clear response

**Mechanism:**

- Nmap sends a UDP packet
- **No response** → port is marked **Open or Filtered** (ambiguous)
- **ICMP (port unreachable) response** → port is **Closed**
- **UDP response received** → port is marked **Open**

| Aspect    | Details                               |
| --------- | ------------------------------------- |
| **Speed** | Very slow (slowest of all scan types) |
| **Flag**  | `-sU`                                 |

### 11.4 Stealth Scans: NULL, FIN, Xmas

These three scans are collectively called **Stealth Scans**, used specifically to try to **bypass firewalls**.

**Why they can bypass firewalls:** Firewalls have rule-sets designed mainly to block **SYN packets from unknown devices** (since SYN indicates an attempt to _establish_ a new connection). These stealth scans avoid sending SYN packets altogether, sending unusual flag combinations that many firewall rule-sets don't explicitly account for — potentially "confusing" the firewall into allowing them through.

| Scan          | Packet(s) Sent                | Open Port Response | Closed Port Response | Flag  |
| ------------- | ----------------------------- | ------------------ | -------------------- | ----- |
| **FIN Scan**  | FIN flag                      | No response        | RST                  | `-sF` |
| **NULL Scan** | No flags at all (null packet) | No response        | RST                  | `-sN` |
| **Xmas Scan** | FIN + PSH + URG flags         | No response        | RST                  | `-sX` |

> Note: For all three, an **open port gives no response** at all — this creates _ambiguity_ (Nmap can't fully distinguish "open" from "filtered" in these cases), which is exactly why they're useful for evading detection but less definitive as diagnostic tools.

### 11.5 ACK Scan (`-sA`)

**Purpose:** Unlike the other scans, ACK scan does **not** tell you whether a port is open or closed. Instead, it identifies whether the target's firewall is **Stateful** or **Stateless**.

**Stateful vs Stateless Firewalls:**

| Type                   | Behavior                                                                                                     |
| ---------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Stateful Firewall**  | Remembers/tracks ongoing connections in memory (e.g., "this device already has a SYN-based connection open") |
| **Stateless Firewall** | Treats every packet **individually** — has no memory of prior packets/connections                            |

**Mechanism:**

- Nmap sends an **ACK** packet
- **Target replies with RST** → port is **Unfiltered** (no firewall blocking this state check)
- **No response at all** → port is **Filtered** (firewall is actively blocking/dropping the packet)

| Aspect                           | Details                              |
| -------------------------------- | ------------------------------------ |
| **Detects open/closed ports?**   | ❌ No                                |
| **Detects firewall rules/type?** | ✅ Yes                               |
| **Use case**                     | Firewall detection & network mapping |
| **Flag**                         | `-sA`                                |

### 11.6 Scan Type Flags — Quick Reference

| Scan Type                       | Flag  |
| ------------------------------- | ----- |
| TCP SYN Scan (Half-Open)        | `-sS` |
| TCP Connect Scan                | `-sT` |
| UDP Scan                        | `-sU` |
| NULL Scan                       | `-sN` |
| FIN Scan                        | `-sF` |
| Xmas Scan                       | `-sX` |
| ACK Scan                        | `-sA` |
| Ping Scan (host discovery only) | `-sn` |

---

## 12. OS Detection

**Purpose:** Determine which operating system the target device is running (Windows / Linux / macOS, etc.).

Nmap sends specially crafted packets and analyzes the responses using several fingerprinting details:

| Signal Used            | What It Tells Nmap                                                                                     |
| ---------------------- | ------------------------------------------------------------------------------------------------------ |
| **TTL (Time To Live)** | How many "hops" (routers) a packet can pass through before expiring — this default value differs by OS |
| **TCP Window Size**    | Each OS has a different data-handling pattern, reflected in its default TCP window size                |
| **IP ID Pattern**      | Each OS assigns packet IDs differently — some sequential, some random — helping fingerprint the OS     |

**Commands:**

| Flag                 | Meaning                                                                                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`-O`** (capital O) | Performs OS Detection                                                                                                                                         |
| **`-A`** (capital A) | **Aggressive Scan** — combines OS detection, version detection, script scanning, and traceroute all in one; takes more time but gathers much more information |

> ⚠️ **Careful:** lowercase `-o` (output) and uppercase `-O` (OS detection) are **completely different flags** in Nmap — don't confuse them.

---

## 13. Service Version Detection

**Purpose:** Once open ports are found, this determines **exactly which service and version** is running on each port (e.g., "FTP — vsftpd 3.0.3").

- Works by sending **protocol-specific probes** to gather this information.
- **Flag:** `-sV`

**Why version matters (practical use):** Once you know the exact version of a running service, you can check online (e.g., via Google/CVE databases) whether that version is **outdated**. If it is outdated, it likely already has known **vulnerabilities** — and a corresponding **exploit** may already exist that can be used to gain access to the target device, without needing to research vulnerabilities from scratch.

**Version Intensity** (controls scan thoroughness vs. speed):

| Intensity Value | Behavior                                 |
| --------------- | ---------------------------------------- |
| **0**           | Fastest, least thorough                  |
| **9**           | Slowest, most accurate/thorough          |
| **7 (default)** | Balanced — neither too fast nor too slow |

Usage: `-sV --version-intensity 9`

---

## 14. Timing Templates

Timing templates control **how fast or slow** a scan runs — slower scans are stealthier (less likely to be detected by firewalls/IDS), while faster scans are quicker but more easily detected.

| Template | Flag  | Speed                         | Detectability                                |
| -------- | ----- | ----------------------------- | -------------------------------------------- |
| **T0**   | `-T0` | Very slow ("Paranoid")        | Very low chance of detection                 |
| **T1**   | `-T1` | Slow ("Sneaky")               | Low chance of detection                      |
| **T2**   | `-T2` | Polite — reduces network load | Moderate                                     |
| **T3**   | `-T3` | **Default** — balanced        | Moderate                                     |
| **T4**   | `-T4` | Aggressive — fast             | More easily detectable                       |
| **T5**   | `-T5` | Fastest ("Insane")            | Very easily detectable; can be less accurate |

**Trade-off:** Faster templates (T4/T5) = quicker results but higher detection risk. Slower templates (T0/T1) = takes much longer but far less likely to trip firewall/IDS alerts.

---

## 15. Firewall / IDS Evasion Techniques

Several techniques can help scans avoid detection by firewalls or Intrusion Detection Systems (IDS):

### a) Stealth Scans

Using FIN, NULL, or Xmas scans instead of SYN/Connect scans (see Section 11.4) — since they don't trigger typical "new connection" firewall rules.

### b) Slow Timing Templates

Using `-T0` or `-T1` to send packets very slowly, making the scan far less likely to trigger detection thresholds.

### c) Packet Fragmentation

**Concept:** Splits packets into smaller ("tiny") fragments so a firewall has a harder time inspecting/recognizing them as a scan.

| Flag           | Purpose                                                                                                      |
| -------------- | ------------------------------------------------------------------------------------------------------------ |
| `-f`           | Fragments packets into small pieces by default                                                               |
| `--mtu <size>` | Sets a custom **Maximum Transmission Unit** — e.g., `--mtu 24` limits each fragment to a maximum of 24 bytes |

Example: `nmap -f --mtu 24 <target>`

### d) Idle / Zombie Scan (`-sI`)

- Hides the attacker's real IP address entirely.
- Uses a **third-party "zombie" host** — the scan appears to come from the zombie's IP, not the attacker's.
- The attacker's own IP **never directly touches** the target.

### e) Source Port Manipulation

- Many firewalls are configured to **trust traffic from specific well-known ports** — e.g., port **53 (DNS)**.
- By manually setting your scan's **source port to 53**, the firewall may treat the traffic as trusted DNS traffic and allow it through.

### f) Randomizing Host Order (`--randomize-hosts` type behavior)

- Normally, when scanning a subnet (e.g., `/24`), Nmap scans hosts **sequentially** (.1, .2, .3, .4…).
- If sequential scanning is being detected/blocked, you can instead have Nmap pick IPs to scan in a **random order** — making the scan pattern harder for a firewall/IDS to recognize as a systematic sweep.

---

## 16. Output Formats

| Flag      | Output Format                                                                                         |
| --------- | ----------------------------------------------------------------------------------------------------- |
| **`-oN`** | Normal text format                                                                                    |
| **`-oX`** | XML format                                                                                            |
| **`-oG`** | Grepable format (easy to parse with grep/scripts)                                                     |
| **`-v`**  | Verbose mode — shows detailed real-time info about what's happening in the background during the scan |

**Bonus tip — Visualizing XML reports:**
Nmap's raw XML output can be converted into a clean, graphical **HTML report** using the `xsltproc` tool (referred to generically as "the XSLT tool" in the transcript):

```bash
xsltproc nmap_scan.xml -o nmap_scan.html
```

Opening the resulting `.html` file in a browser gives a neatly formatted table showing **Port, State, Service, Reason, Product, Version**, and extra info for each result — much easier to read than raw XML.

---

## 17. Putting It All Together — Full Command Example

A comprehensive scan command combining multiple flags learned above:

```bash
nmap -sT -sU -p- -A -v <target_IP> -oX nmap_scan.xml
```

**Breakdown:**

| Flag                | Purpose                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------- |
| `-sT`               | Scan all TCP ports (TCP Connect Scan)                                                       |
| `-sU`               | Also scan UDP ports                                                                         |
| `-p-`               | Scan **all** 65,535 ports (not just the default top 1000)                                   |
| `-A`                | Aggressive scan — auto-enables OS detection, version detection, script scanning, traceroute |
| `-v`                | Verbose output — shows live scan progress in the terminal                                   |
| `<target_IP>`       | The target to scan                                                                          |
| `-oX nmap_scan.xml` | Save output in XML format to `nmap_scan.xml`                                                |

> Reminder: lowercase `-o` = output-related flags (`-oN`, `-oX`, `-oG`); uppercase `-O` = OS detection.

---

## 18. Legal & Ethical Use

⚠️ **Critical reminder:** Scanning any network or device **without proper authorization is illegal.**

You may only scan:

- **Your own systems/devices** that you personally own
- Systems where you have **explicit written authorization** (e.g., authorized penetration testing contracts)
- **Bug bounty programs** you are officially enrolled in
- A **local, isolated lab environment** set up specifically for practice

Always operate legally and ethically — practice only in controlled lab environments or with clear authorization.

---

## 19. Summary / Quick Revision Table

| Concept                      | One-Line Summary                                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------- |
| **Nmap**                     | Open-source network discovery & security auditing tool (created 1997 by Gordon Lyon)              |
| **Ports**                    | 65,535 total — divided into Well-Known (0–1023), Registered (1024–49151), Dynamic (49151–65535)   |
| **TCP Flags**                | SYN (connect), ACK (acknowledge), RST (reset/close), FIN (finish), PSH (push now), URG (urgent)   |
| **3-Way Handshake**          | SYN → SYN+ACK → ACK, used to establish TCP connections                                            |
| **Port States**              | Open (listening), Closed (not listening), Filtered (firewall blocking), Unfiltered (undetermined) |
| **`-sn`**                    | Host discovery only, no port scan                                                                 |
| **`-Pn`**                    | Skip host discovery, go straight to port scan                                                     |
| **`-sT`**                    | TCP Connect Scan — full handshake, reliable, slower, no root needed                               |
| **`-sS`**                    | TCP SYN Scan — half-open, fast, stealthier, needs root                                            |
| **`-sU`**                    | UDP Scan — slowest, no handshake, relies on timeouts                                              |
| **`-sN` / `-sF` / `-sX`**    | Stealth scans (Null/FIN/Xmas) — used to evade firewalls                                           |
| **`-sA`**                    | ACK Scan — detects stateful vs stateless firewalls, not port state                                |
| **`-O`**                     | OS Detection                                                                                      |
| **`-A`**                     | Aggressive scan (OS + version + scripts + traceroute)                                             |
| **`-sV`**                    | Service version detection                                                                         |
| **`--version-intensity`**    | 0 = fastest, 9 = most accurate, 7 = default                                                       |
| **Timing Templates (T0–T5)** | T0 = slowest/stealthiest, T5 = fastest/most detectable, T3 = default                              |
| **`-f` / `--mtu`**           | Packet fragmentation for firewall evasion                                                         |
| **`-sI`**                    | Idle/Zombie scan — hides attacker's real IP                                                       |
| **`-oN` / `-oX` / `-oG`**    | Output formats: Normal / XML / Grepable                                                           |
| **`-v`**                     | Verbose mode                                                                                      |
| **Legal use**                | Only scan your own systems or systems you're explicitly authorized to test                        |

---

_End of Notes — Nmap Crash Course_
