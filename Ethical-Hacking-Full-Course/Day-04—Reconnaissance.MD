# Day 04 — Reconnaissance (Recon / Footprinting)

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 03 — EH Methodology & Cyber Kill Chain](#) | Next: [Day 05](#)

---

## 📋 Table of Contents

1. [What is Reconnaissance?](#1-what-is-reconnaissance)
2. [Types of Reconnaissance](#2-types-of-reconnaissance)
   - [Passive Reconnaissance](#passive-reconnaissance)
   - [Active Reconnaissance](#active-reconnaissance)
3. [Passive vs Active — Full Comparison Table](#3-passive-vs-active--full-comparison-table)
4. [Tools Overview](#4-tools-overview)
5. [Recon Workflow (Best Practice)](#5-recon-workflow-best-practice)
6. [Quick Reference Summary](#6-quick-reference-summary)

---

## 1. What is Reconnaissance?

**Reconnaissance** (also called **Footprinting**) is the **first and most critical phase** of a penetration test or security assessment.

> The attacker gathers as much information as possible about the target — its systems, network, technologies, and people — **before attempting any attack.**

### Why is it important?

| Purpose | Detail |
|---------|--------|
| **Understand system architecture** | How is the target built? What technologies does it use? |
| **Identify vulnerabilities** | What weaknesses might exist based on the stack? |
| **Find entry points** | What are the potential ways in? |

> 🏠 **Analogy:** A thief scoping out a building before breaking in — counting doors, windows, checking for CCTV cameras, and noting what the walls are made of.

---

## 2. Types of Reconnaissance

Recon is divided into **two types**:

```
Reconnaissance
├── Passive Reconnaissance  (no direct interaction with target)
└── Active Reconnaissance   (direct interaction with target)
```

---

### Passive Reconnaissance

**Definition:** Gathering information about the target **without directly interacting** with it.

You act as a normal user/observer — the target has no idea you're collecting information.

#### Techniques Used
| Technique | Description |
|-----------|-------------|
| **OSINT (Open Source Intelligence)** | Collecting all publicly available information about the target |
| **Google Dorking** | Using advanced Google search operators to find exposed info |
| **Social Media Analysis** | LinkedIn, Twitter, Instagram — employees, tech stack, org structure |
| **WHOIS Lookup** | Domain registration info, registrant details |
| **DNS Lookups** | Subdomains, mail servers, name servers |
| **Job Postings Analysis** | Reveals tech stack (e.g., "Must know AWS, Django, PostgreSQL") |
| **Website Analysis** | Technologies used (Wappalyzer, BuiltWith) |

#### Example Scenario
> You find an Instagram photo of a company employee where a laptop screen is visible running **Adobe Acrobat X**. You check public exploit databases and find a known CVE for that version. That's passive recon leading to a potential attack vector — without ever touching their systems.

---

### Active Reconnaissance

**Definition:** Gathering information by **directly interacting** with the target system.

You send requests, packets, or probes to the target — this leaves traces and has a higher risk of detection.

#### Techniques Used
| Technique | Tool | Description |
|-----------|------|-------------|
| **Port Scanning** | `nmap` | Discover open/closed/filtered ports on target |
| **Network Mapping** | `nmap` | Map the network topology |
| **Vulnerability Scanning** | `Nessus`, `OpenVAS` | Identify known CVEs and misconfigurations |
| **Network Probing** | `nmap`, `Nessus` | Send probes to enumerate services and versions |
| **Banner Grabbing** | `nmap`, `netcat` | Read service banners to identify software versions |

#### Example Scenario
> You run `nmap` against a target's web server. Your machine sends packets directly to the server, which responds telling you which ports are open. The server's IDS/firewall may log this activity.

---

## 3. Passive vs Active — Full Comparison Table

| Aspect | Passive Reconnaissance | Active Reconnaissance |
|--------|----------------------|----------------------|
| **Definition** | Info gathering without direct target interaction | Info gathering with direct target interaction |
| **Target Interaction** | ❌ None | ✅ Direct |
| **Techniques** | OSINT, Google Dorking, WHOIS, social media | Port scanning, vuln scanning, network probing |
| **Tools** | Google, Shodan, Maltego, theHarvester | `nmap`, `Nessus`, `OpenVAS`, `netcat` |
| **Risk of Detection** | 🟢 Low — appears as normal user traffic | 🔴 High — packets sent directly to target |
| **Information Accuracy** | ⚠️ May be outdated or misleading | ✅ Generally accurate and real-time |
| **Resource Intensity** | 🟢 Low — just needs a browser/search engine | 🔴 Higher — needs tools and a proper setup |
| **Vulnerability Identification** | 🟡 Indirect (inferred from public info) | ✅ Direct (scanners find CVEs explicitly) |
| **Use Case** | Early-stage background gathering; stealth required | Penetration tests needing detailed enumeration |

---

## 4. Tools Overview

### Passive Recon Tools

| Tool | Purpose |
|------|---------|
| **Google** | Google Dorking for exposed files, login pages, configs |
| **Shodan** | Search engine for internet-connected devices and open ports |
| **theHarvester** | Gather emails, subdomains, IPs from public sources |
| **Maltego** | Visual OSINT and link analysis |
| **WHOIS** | Domain ownership and registration info |
| **Wappalyzer / BuiltWith** | Identify web technologies used by a site |
| **LinkedIn / Social Media** | Employee info, org structure, tech stack |

### Active Recon Tools

| Tool | Purpose |
|------|---------|
| **`nmap`** | Port scanning, service detection, OS fingerprinting |
| **`Nessus`** | Comprehensive vulnerability scanner |
| **`OpenVAS`** | Open-source vulnerability scanner |
| **`netcat`** | Banner grabbing, port checking |
| **`Nikto`** | Web server vulnerability scanner |

---

## 5. Recon Workflow (Best Practice)

In real penetration tests, recon is always done in this order:

```
STEP 1: Passive Recon
   └── Collect all publicly available information
   └── OSINT, Google Dorking, WHOIS, social media, job postings
   └── No risk of detection

         ↓

STEP 2: Active Recon
   └── Use gathered info to target specific systems
   └── Port scan, vulnerability scan, service enumeration
   └── Higher detail, higher detection risk
```

> 💡 **Why this order?**  
> Passive recon gives you a map of the terrain safely. Active recon then fills in the technical details. Going straight to active recon wastes time scanning things that passive recon could have eliminated.

---

## 6. Quick Reference Summary

| | Passive | Active |
|-|---------|--------|
| **Interact with target?** | No | Yes |
| **Detection risk** | Low | High |
| **Data accuracy** | Medium | High |
| **Best for** | Stealth, early stage | Detailed enumeration |
| **Key tools** | Google, Shodan, theHarvester | nmap, Nessus, OpenVAS |

---

> ✅ **End of Day 04 Notes**  
> 📌 Next up: **Day 05** — we'll go deeper into one of the recon techniques in detail.  
> 💬 Questions? Drop them in the comments!