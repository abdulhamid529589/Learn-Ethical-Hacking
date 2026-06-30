# Day 06 — Passive Reconnaissance: WHOIS & DNS Reconnaissance

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 05 — Google Dorking](#) | Next: [Day 07](#)

---

## 📋 Table of Contents

1. [WHOIS Records](#1-whois-records)
   - [What is WHOIS?](#what-is-whois)
   - [What Data Can You Extract?](#what-data-can-you-extract)
   - [WHOIS — Practical Demo](#whois--practical-demo)
2. [DNS Reconnaissance](#2-dns-reconnaissance)
   - [What is DNS?](#what-is-dns)
   - [How DNS Works](#how-dns-works)
   - [DNS Record Types](#dns-record-types)
3. [DNS Recon Tools](#3-dns-recon-tools)
   - [host](#tool-1-host)
   - [nslookup](#tool-2-nslookup)
   - [dig](#tool-3-dig)
4. [Tools Comparison](#4-tools-comparison)
5. [Quick Reference Cheat Sheet](#5-quick-reference-cheat-sheet)

---

## 1. WHOIS Records

### What is WHOIS?

When you register a domain (e.g., on GoDaddy), you provide personal/contact details to the domain registrar. **WHOIS** is an open protocol that lets anyone query those registration details from the registrar's database.

> 💡 WHOIS is **passive recon** — you query a public database, never touching the target's server.

**Information stored in WHOIS:**

| Field | Description |
|-------|-------------|
| **Domain Owner / Registrant** | Name of the person or org who registered it |
| **Organization** | Company name |
| **Email** | Admin contact email |
| **Phone** | Admin contact number |
| **Address** | Physical address of the registrant |
| **Registration Date** | When the domain was first registered |
| **Expiry Date** | When the domain will expire |
| **Last Updated** | When details were last changed |
| **Name Servers (NS)** | Which DNS servers the domain uses |
| **Registrar** | Which company sold the domain (e.g., GoDaddy, Namecheap) |
| **WHOIS Server** | Which WHOIS server holds the record |

> 🔒 **WHOIS Privacy Guard:** Domain owners can pay for a privacy guard service that hides their personal details from public WHOIS queries. Many registrars now offer this free by default. If enabled, you'll see the registrar's info instead of the owner's.

---

### What Data Can You Extract?

| Use Case | What WHOIS Reveals |
|----------|--------------------|
| **Investigating a scam site** | Registration date, registrar, possibly owner contact |
| **Recon on a target company** | Email format, org name, address, DNS servers used |
| **Domain expiry tracking** | Expiry date — expired domains can be re-purchased |
| **Finding admin contacts** | Email/phone for social engineering or phishing |

> ⚠️ **Attack Scenario:** If `microsoft.com` expired on May 3, 2025 and wasn't renewed, anyone could purchase it. High-value expired domains sell at auction for large sums.

---

### WHOIS — Practical Demo

#### Method 1: Website
- Go to [https://who.is](https://who.is)
- Enter the target domain (e.g., `microsoft.com`)
- View registrant details, name servers, expiry date, raw data

#### Method 2: Terminal (Kali Linux — built-in)

```bash
whois microsoft.com
```

**Sample output fields:**
```
Domain Name: MICROSOFT.COM
Registrar: MarkMonitor Inc.
Updated Date: 2023-08-XX
Creation Date: 1991-05-02
Expiry Date: 2025-05-03
Name Server: ns1-39.azure-dns.com
Name Server: ns2-39.azure-dns.net
Registrant Organization: Microsoft Corporation
Registrant Address: One Microsoft Way, Redmond, WA, US
Admin Email: admin@domains.microsoft.com
```

---

## 2. DNS Reconnaissance

### What is DNS?

**DNS (Domain Name System)** is a service that translates human-readable domain names into numeric IP addresses (and vice versa).

> 💡 Computers communicate using IP addresses, not domain names. DNS bridges the gap.

**Example:**
```
You type:    google.com
DNS resolves: google.com → 142.250.80.46
Browser connects to: 142.250.80.46
```

**Key facts:**
- Protocol: **UDP**
- Port: **53**
- Can also do **reverse lookup**: IP → Domain name

---

### How DNS Works

```
[You] → type google.com in browser
   ↓
[Browser] → asks DNS Resolver: "What's the IP for google.com?"
   ↓
[DNS Resolver] → queries authoritative DNS server
   ↓
[DNS Server] → returns IP: 142.250.80.46
   ↓
[Browser] → connects to 142.250.80.46
   ↓
[Google's server] → sends back the webpage
```

---

### DNS Record Types

| Record | Full Name | What It Does | Example |
|--------|-----------|--------------|---------|
| **A** | Address | Maps domain → IPv4 address | `google.com → 142.250.80.46` |
| **AAAA** | Quad-A | Maps domain → IPv6 address | `google.com → 2607:f8b0::...` |
| **CNAME** | Canonical Name | Alias — maps one domain name to another | `mail.google.com → google.com` |
| **NS** | Name Server | Specifies which DNS server is authoritative for the domain | `microsoft.com → ns1-39.azure-dns.com` |
| **MX** | Mail Exchange | Identifies the mail server for the domain | `microsoft.com → microsoft-com.mail.protection.outlook.com` |
| **TXT** | Text | Stores arbitrary text — used for verification, SPF records | Google AdSense verification codes |
| **SOA** | Start of Authority | Admin info about the domain — primary NS, zone update settings | Config/admin metadata |

> 💡 **CNAME Example:** `mail.google.com` is a CNAME (alias) for `google.com`. The subdomain `mail` points to the same server as `google.com` via a CNAME record.

> 💡 **TXT Example:** When you connect Google Search Console or AdSense to a website, Google gives you a TXT record to add to your DNS to verify ownership.

---

### Domain Hierarchy

```
. (root)
└── .com / .org / .gov / .net  (Top Level Domains - TLD)
    └── google.com / microsoft.com  (Second Level Domain)
        └── mail.google.com  (Subdomain)
```

- **TLDs** (`.com`, `.org`, `.gov`) have the highest authority
- **Country TLDs** (`.co.in`, `.co.uk`) are slightly lower in hierarchy
- **Subdomains** are mapped via CNAME records

---

## 3. DNS Recon Tools

All three tools below are **built into Kali Linux**. They do similar jobs — use whichever fits your workflow.

---

### Tool 1: `host`

A simple, fast DNS lookup tool for Linux.

#### Get IP address of a domain
```bash
host microsoft.com
```
**Output:**
```
microsoft.com has address 20.112.52.29
microsoft.com has address 20.81.111.85
microsoft.com has IPv6 address 2603:1030:b:3::152
```
> Multiple IPs usually mean the site uses a CDN (e.g., Cloudflare). The real origin IP may be hidden.

#### Get Name Servers (NS records)
```bash
host -t ns microsoft.com
```
**Output:**
```
microsoft.com name server ns1-39.azure-dns.com.
microsoft.com name server ns2-39.azure-dns.net.
microsoft.com name server ns3-39.azure-dns.org.
microsoft.com name server ns4-39.azure-dns.info.
```

#### Get Mail Servers (MX records)
```bash
host -t mx microsoft.com
```
**Output:**
```
microsoft.com mail is handled by microsoft-com.mail.protection.outlook.com.
```

#### Get TXT records
```bash
host -t txt microsoft.com
```

#### Get IPv6 address (AAAA records)
```bash
host -t aaaa microsoft.com
```

#### Syntax Summary
```bash
host <domain>               # A record (IPv4)
host -t ns <domain>         # Name servers
host -t mx <domain>         # Mail servers
host -t txt <domain>        # TXT records
host -t aaaa <domain>       # IPv6 address
host -t cname <domain>      # CNAME records
host -t soa <domain>        # SOA record
```

---

### Tool 2: `nslookup`

Works on **both Linux and Windows** — great for cross-platform use.

#### Basic IP lookup
```bash
nslookup microsoft.com
```

#### Interactive mode (query multiple record types)
```bash
nslookup
> set type=ns
> microsoft.com

> set type=mx
> microsoft.com

> set type=txt
> microsoft.com

> set type=aaaa
> microsoft.com

> set type=cname
> microsoft.com

> exit
```

#### One-line queries (non-interactive)
```bash
nslookup -type=ns microsoft.com
nslookup -type=mx microsoft.com
nslookup -type=txt microsoft.com
nslookup -type=aaaa microsoft.com
nslookup -type=soa microsoft.com
```

> ✅ **Windows users:** `nslookup` works natively in Windows Command Prompt and PowerShell — no installation needed.

---

### Tool 3: `dig`

The most detailed and verbose DNS tool. Preferred by professionals for scripting and deep analysis.

#### Basic lookup
```bash
dig microsoft.com
```

#### Query specific record types
```bash
dig -t mx microsoft.com       # Mail servers
dig -t ns microsoft.com       # Name servers
dig -t txt microsoft.com      # TXT records
dig -t aaaa microsoft.com     # IPv6
dig -t cname microsoft.com    # CNAME
dig -t soa microsoft.com      # SOA record
```

#### Short output (just the answer)
```bash
dig microsoft.com +short
dig -t mx microsoft.com +short
dig -t ns microsoft.com +short
```

#### Reverse DNS lookup (IP → Domain)
```bash
dig -x 20.112.52.29
```

#### Query a specific DNS server
```bash
dig @8.8.8.8 microsoft.com    # Use Google's DNS server
dig @1.1.1.1 microsoft.com    # Use Cloudflare's DNS server
```

---

## 4. Tools Comparison

| Feature | `host` | `nslookup` | `dig` |
|---------|--------|------------|-------|
| **OS** | Linux | Linux + Windows | Linux |
| **Output detail** | Medium | Medium | High (verbose) |
| **Interactive mode** | ❌ | ✅ | ❌ |
| **Scripting friendly** | ✅ | ❌ | ✅ |
| **Built into Kali** | ✅ | ✅ | ✅ |
| **Best for** | Quick lookups | Windows / interactive | Deep analysis / scripting |

---

## 5. Quick Reference Cheat Sheet

### WHOIS
```bash
whois <domain>                  # Full WHOIS lookup via terminal
```
Or visit: [https://who.is](https://who.is)

### host — Common Commands
```bash
host <domain>                   # IPv4 address
host -t ns <domain>             # Name servers
host -t mx <domain>             # Mail servers
host -t txt <domain>            # TXT records
host -t aaaa <domain>           # IPv6 address
host -t soa <domain>            # SOA record
```

### nslookup — Common Commands
```bash
nslookup <domain>               # Basic lookup
nslookup -type=ns <domain>      # Name servers
nslookup -type=mx <domain>      # Mail servers
nslookup -type=txt <domain>     # TXT records
nslookup -type=aaaa <domain>    # IPv6
```

### dig — Common Commands
```bash
dig <domain>                    # Default A record lookup
dig -t mx <domain>              # Mail servers
dig -t ns <domain>              # Name servers
dig -t txt <domain>             # TXT records
dig -t aaaa <domain>            # IPv6
dig -t soa <domain>             # SOA record
dig <domain> +short             # Short output only
dig -x <IP>                     # Reverse lookup
dig @8.8.8.8 <domain>          # Query specific DNS server
```

### DNS Record Quick Reference
```
A     → domain → IPv4
AAAA  → domain → IPv6
CNAME → alias (subdomain) → another domain
NS    → which DNS server handles this domain
MX    → which mail server handles email
TXT   → arbitrary text (verification, SPF, DKIM)
SOA   → admin/config info about the zone
```

---

> ✅ **End of Day 06 Notes**  
> 📌 Next up: More **Passive Reconnaissance** techniques  
> 💡 Try running `whois`, `host`, `nslookup`, and `dig` against any domain you own or a bug bounty target!  
> 💬 Questions? Drop them in the comments!