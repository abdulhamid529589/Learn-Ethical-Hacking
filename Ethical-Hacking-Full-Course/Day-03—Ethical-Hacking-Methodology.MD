# Day 03 — Ethical Hacking Methodology & Cyber Kill Chain

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 01](#) | [Day 02](#) | Next: [Day 04](#)

---

## 📋 Table of Contents

1. [Ethical Hacking Methodology](#1-ethical-hacking-methodology)
   - [Phase 1: Reconnaissance & Footprinting](#phase-1-reconnaissance--footprinting)
   - [Phase 2: Scanning](#phase-2-scanning)
   - [Phase 3: Gaining Access](#phase-3-gaining-access)
   - [Phase 4: Maintaining Access](#phase-4-maintaining-access)
   - [Phase 5: Clearing Tracks](#phase-5-clearing-tracks)
2. [Cyber Kill Chain Methodology](#2-cyber-kill-chain-methodology)
   - [Step 1: Reconnaissance](#step-1-reconnaissance)
   - [Step 2: Weaponization](#step-2-weaponization)
   - [Step 3: Delivery](#step-3-delivery)
   - [Step 4: Exploitation](#step-4-exploitation)
   - [Step 5: Installation](#step-5-installation)
   - [Step 6: Command & Control (C2)](#step-6-command--control-c2)
   - [Step 7: Actions on Objectives](#step-7-actions-on-objectives)
3. [Key Differences: EH Methodology vs Cyber Kill Chain](#3-key-differences)
4. [Quick Reference Summary](#4-quick-reference-summary)

---

## 1. Ethical Hacking Methodology

### What is it?

When hacking a target, you **never go in directly** — you follow a structured process called the **Ethical Hacking Methodology**. This ensures a systematic, professional, and repeatable approach to penetration testing.

> 🏠 **Analogy:** Think like a thief planning to break into a building — but legally, with permission, to find security gaps.

The methodology has **5 phases**:

---

### Phase 1: Reconnaissance & Footprinting

| Item | Detail |
|------|--------|
| **Goal** | Gather as much information about the target as possible |
| **Also called** | Recon, Footprinting, Information Gathering |
| **Interaction with target** | Minimal to none (especially in Passive Recon) |

#### Types of Reconnaissance

| Type | Description | Example |
|------|-------------|---------|
| **Passive Recon** | Gathering info *without* directly interacting with the target | Searching WHOIS, Google dorking, LinkedIn scraping |
| **Active Recon** | Directly interacting with the target | Port scanning, sending requests to the server |

#### Common Techniques
- Google Dorking
- WHOIS lookups
- DNS enumeration
- Social media OSINT
- Job postings analysis (reveals tech stack)

> 🏠 **Analogy:** The thief scopes out the building — counts the doors, windows, and possible entry points.

---

### Phase 2: Scanning

| Item | Detail |
|------|--------|
| **Goal** | Identify vulnerabilities in the discovered attack surface |
| **Builds on** | Information gathered during Recon |

#### Common Techniques & Tools

| Technique | Tool | Purpose |
|-----------|------|---------|
| Network Mapping | `nmap` | Discover live hosts and open ports |
| Port Scanning | `nmap`, `masscan` | Find open/closed/filtered ports |
| Vulnerability Scanning | `Nessus`, `OpenVAS` | Find known CVEs and weaknesses |

> 🏠 **Analogy:** The thief now checks each door and window — looking for one that's unlocked, has a weak lock, or can be forced open.

---

### Phase 3: Gaining Access

| Item | Detail |
|------|--------|
| **Goal** | Exploit identified vulnerabilities to get into the system |
| **Also called** | Exploitation Phase |

#### Common Attack Techniques

| Attack | Description |
|--------|-------------|
| **SQL Injection** | Injecting malicious SQL queries to dump/manipulate databases |
| **Buffer Overflow** | Overwriting memory buffers to execute arbitrary shellcode |
| **Phishing** | Tricking users into clicking malicious links or downloading payloads |

> 🏠 **Analogy:** The thief uses the unlocked window or picks the weak lock to get inside the building.

---

### Phase 4: Maintaining Access

| Item | Detail |
|------|--------|
| **Goal** | Keep persistent access to the compromised system |
| **Why** | So the attacker doesn't have to repeat the exploitation process each time |

#### Common Persistence Techniques

| Technique | Description |
|-----------|-------------|
| **Rootkits** | Malicious programs that hide attacker presence and maintain control |
| **Hidden User Accounts** | Creating a backdoor admin account for future logins |
| **Backdoors** | Installing remote access tools that phone home to the attacker |
| **Tunneling** | Using encrypted tunnels (e.g., SSH tunnels) to maintain covert access |

> 🏠 **Analogy:** The thief leaves a door slightly open (or makes a copy of the key) so they can come and go freely without breaking in again.

---

### Phase 5: Clearing Tracks

| Item | Detail |
|------|--------|
| **Goal** | Remove all evidence of the attack to avoid detection |
| **Why** | Prevent forensic investigators and security teams from tracing activity back |

#### Common Techniques

| Technique | Description |
|-----------|-------------|
| **Log Modification/Deletion** | Alter or wipe system logs that record all activities |
| **Deleting Evidence** | Remove malware, tools, or files used during the attack |
| **Obfuscating Malware** | Make malicious code harder to analyze (hiding C2 IPs, domain names) |

> 🏠 **Analogy:** The thief deletes CCTV footage, wipes fingerprints, and removes the ladder they used — leaving no trace.

---

## 2. Cyber Kill Chain Methodology

### What is it?

The **Cyber Kill Chain** was developed by **Lockheed Martin**, inspired by military targeting models. It maps out the **complete lifecycle of a cyberattack** — from the attacker's first step to achieving their goal.

> 💡 Used heavily in **Red Team** operations and threat intelligence.

It has **7 steps**:

---

### Step 1: Reconnaissance

Same as Ethical Hacking Methodology — gather target information.

**Specific intel collected:**
- Open ports and services
- Employee names and roles (LinkedIn, etc.)
- Technologies in use (web server, firewall, CMS)
- Email formats and contacts

---

### Step 2: Weaponization

| Item | Detail |
|------|--------|
| **Goal** | Create a malicious payload tailored to the target's identified vulnerabilities |
| **Tool example** | `msfvenom` (Metasploit Framework) |

**What happens here:**
- Attacker crafts a **malware/payload** based on recon findings
- Payload is paired with an **exploit** that matches the target's vulnerability
- Common form: a PDF, Word doc, or image file with an **embedded malicious payload**

---

### Step 3: Delivery

| Item | Detail |
|------|--------|
| **Goal** | Transmit the weaponized payload to the target |

#### Delivery Methods

| Method | Example |
|--------|---------|
| **Phishing Email** | HR-themed email with malicious attachment |
| **USB Drop** | Leave infected USBs in parking lots — hoping employees plug them in |
| **Watering Hole Attack** | Compromise websites frequently visited by target employees |
| **Physical Mail** | Send infected USB or media to the target's office |

---

### Step 4: Exploitation

| Item | Detail |
|------|--------|
| **Goal** | Trigger the payload; exploit a vulnerability to execute code |
| **Trigger** | User opens the malicious file / visits compromised site |

Once the victim interacts with the payload → **malware executes → attacker gains access**.

---

### Step 5: Installation

| Item | Detail |
|------|--------|
| **Goal** | Establish a persistent foothold on the compromised system |
| **Equivalent to** | "Maintaining Access" in EH Methodology |

**Techniques used:**
- Installing **backdoors**
- Setting up **Remote Access Tools (RATs)**
- Creating **scheduled tasks** or **registry entries** for persistence

---

### Step 6: Command & Control (C2)

| Item | Detail |
|------|--------|
| **Goal** | Remotely control the compromised system without detection |
| **How** | Establish a covert communication channel between attacker and victim machine |

**How it works:**
- Malware on victim "phones home" to the attacker's **C2 server**
- Attacker sends commands through this channel
- Traffic is often disguised to look like normal web/DNS traffic to avoid detection

---

### Step 7: Actions on Objectives

| Item | Detail |
|------|--------|
| **Goal** | Carry out the actual mission / final goal of the attack |

#### Possible Objectives

| Objective | Example |
|-----------|---------|
| **Data Theft** | Exfiltrating sensitive files, credentials, PII |
| **Ransomware** | Encrypting files and demanding payment |
| **System Destruction** | Wiping disks, bricking infrastructure |
| **Espionage** | Long-term silent monitoring of the target |

---

## 3. Key Differences

| Aspect | Ethical Hacking Methodology | Cyber Kill Chain |
|--------|----------------------------|-----------------|
| **Created by** | General security community | Lockheed Martin |
| **Inspired by** | Penetration testing practice | Military targeting doctrine |
| **Steps** | 5 | 7 |
| **Primary use** | Penetration Testing | Red Teaming & Threat Intelligence |
| **Focus** | Testing a system's defenses | Modeling a real attacker's full campaign |
| **Weaponization step** | Not explicitly separated | Explicit (Step 2) |
| **C2 step** | Implied in Maintaining Access | Explicit (Step 6) |

---

## 4. Quick Reference Summary

### Ethical Hacking Methodology — 5 Phases

```
[1] Reconnaissance  →  [2] Scanning  →  [3] Gaining Access
        ↓
[4] Maintaining Access  →  [5] Clearing Tracks
```

### Cyber Kill Chain — 7 Steps

```
[1] Recon → [2] Weaponization → [3] Delivery → [4] Exploitation
                    ↓
        [5] Installation → [6] C2 → [7] Actions on Objectives
```

---

> ✅ **End of Day 03 Notes**  
> 💬 Questions? Drop them in the comments or message the instructor.  
> 👍 Found this helpful? Like and share the series!