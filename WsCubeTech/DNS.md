# DNS & Domain Name System — Complete Study Notes

## https://youtu.be/WtsedKN6w7g

> **Course:** Cyber Security & Networking Fundamentals
> **Topic:** Understanding Domains, URLs, and DNS (Basic to Advanced)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Anatomy of a URL](#2-anatomy-of-a-url)
   - 2.1 [Protocol (HTTP/HTTPS)](#21-protocol-httphttps)
   - 2.2 [Subdomain](#22-subdomain)
   - 2.3 [Second-Level Domain (SLD)](#23-second-level-domain-sld)
   - 2.4 [Top-Level Domain (TLD)](#24-top-level-domain-tld)
   - 2.5 [Path and Query String](#25-path-and-query-string)
   - 2.6 [Domain vs Subdomain vs Path — Key Differences](#26-domain-vs-subdomain-vs-path--key-differences)
3. [Identifying Real vs Fake Domains](#3-identifying-real-vs-fake-domains)
   - 3.1 [Characteristics of a Real/Legitimate Domain](#31-characteristics-of-a-reallegitimate-domain)
   - 3.2 [Characteristics of a Fake/Phishing Domain](#32-characteristics-of-a-fakephishing-domain)
   - 3.3 [Social Engineering Tactics: Urgency & Fear](#33-social-engineering-tactics-urgency--fear)
4. [What is DNS (Domain Name System)?](#4-what-is-dns-domain-name-system)
   - 4.1 [The Phonebook Analogy](#41-the-phonebook-analogy)
   - 4.2 [Why DNS is Necessary](#42-why-dns-is-necessary)
5. [How DNS Works Behind the Scenes](#5-how-dns-works-behind-the-scenes)
   - 5.1 [Step-by-Step DNS Resolution Process](#51-step-by-step-dns-resolution-process)
6. [DNS Record Types](#6-dns-record-types)
7. [DNS Caching](#7-dns-caching)
   - 7.1 [TTL (Time to Live)](#71-ttl-time-to-live)
   - 7.2 [Why Distributed Name Servers Matter](#72-why-distributed-name-servers-matter)
8. [DNSSEC (Domain Name System Security Extensions)](#8-dnssec-domain-name-system-security-extensions)
   - 8.1 [The Core Problem DNSSEC Solves](#81-the-core-problem-dnssec-solves)
   - 8.2 [Digital Signatures](#82-digital-signatures)
   - 8.3 [How DNSSEC Works — Step by Step](#83-how-dnssec-works--step-by-step)
   - 8.4 [Chain of Trust](#84-chain-of-trust)
   - 8.5 [Authenticity vs Integrity](#85-authenticity-vs-integrity)
   - 8.6 [Important: DNSSEC Does NOT Encrypt](#86-important-dnssec-does-not-encrypt)
9. [Summary / Key Takeaways](#9-summary--key-takeaways)
10. [Quick Revision Table](#10-quick-revision-table)

---

## 1. Introduction

Every time you type a website name like `Google.com` into your browser, your computer needs a way to find the **actual physical server** that hosts that website. Computers don't understand names like "Google" — they only understand numerical addresses (IP addresses). The system responsible for converting a human-readable name into a machine-usable address is called the **Domain Name System (DNS)**.

Understanding DNS is **not optional** for anyone entering cybersecurity or networking — it is one of the foundational building blocks of how the internet works, and it is also one of the most commonly abused systems by attackers (phishing, spoofing, cache poisoning, etc.).

This document covers:

- The structure of a URL (domain, subdomain, path, etc.)
- How DNS resolves a domain name into an IP address, step by step
- The different types of DNS records (A, AAAA, CNAME, NS, MX, PTR)
- DNS caching and TTL
- DNSSEC and how it protects against spoofing
- How to visually distinguish a real domain from a fake/phishing domain

---

## 2. Anatomy of a URL

A **URL (Uniform Resource Locator)** is simply the _address_ of a web page — just like every house has a postal address, every web page has a URL.

Example URL breakdown:

```
https://www.example.com/about?ref=homepage
   |      |      |    |    |         |
Protocol Sub  Domain  TLD  Path   Query String
        Domain
```

### 2.1 Protocol (HTTP/HTTPS)

- **HTTP** = HyperText Transfer Protocol — the basic protocol for transferring web data.
- **HTTPS** = HTTP **Secure** — the "S" indicates that the data transfer is encrypted/secured.
- Many old websites still run on plain HTTP. Modern browsers show a **warning** when you visit such sites.
- **Best practice:** Avoid visiting HTTP-only websites. If you must (e.g., for research), do so inside an **isolated environment** (like a sandboxed VM) to stay safe.

### 2.2 Subdomain

- The subdomain is a **prefix** added before the main domain (e.g., `www`, `cloud`, `blog`, `login`).
- It is **not mandatory** to always be `www` — it changes depending on which section/service of the website you're visiting.
- Example: `cloud.google.com` → `cloud` is the subdomain, showing you are on the Cloud services section of Google.
- Subdomains are created and controlled **by the website's own developers** — you don't need to "buy" a subdomain; it's free and can be created instantly.

### 2.3 Second-Level Domain (SLD)

- This is the **registered domain name** — the actual brand/organization identity.
- Example: `wscube` in `wscube.com`, or `google` in `google.com`.
- This name is **chosen by the founder/owner** of the organization and is **purchased/registered** (e.g., via registrars like GoDaddy) at a cost.
- Because it costs money and effort to register and maintain, **this rarely changes**.

### 2.4 Top-Level Domain (TLD)

- The extension after the domain, e.g., `.com`, `.org`, `.gov`, `.net`.
- Trusted/well-known TLDs include `.com`, `.gov`, `.edu`, `.org`.
- Note: in some course terminology, the subdomain position was also loosely referred to as "third-level domain," while the true TLD (`.com`, `.gov`, etc.) is the actual **top-level domain** — don't get confused between the two.

### 2.5 Path and Query String

- **Path** → the exact _location of the specific page_ within the website (e.g., `/about`, `/login`).
- **Query String** → additional parameters passed to the page (e.g., `?ref=homepage`), often used to pass extra information about how/why the page was accessed.
- Both the **path** and **query string** are created and can be freely changed by the developer, instantly and at no cost.

### 2.6 Domain vs Subdomain vs Path — Key Differences

| Component              | Who Creates/Controls It                      | Cost to Change | Frequency of Change               |
| ---------------------- | -------------------------------------------- | -------------- | --------------------------------- |
| **Domain (SLD + TLD)** | Purchased from a registrar (e.g., GoDaddy)   | Costs money    | Rarely changes                    |
| **Subdomain**          | Created by the organization's own developers | Free           | Can be changed anytime            |
| **Path**               | Created by developers when building the site | Free           | Can be changed anytime, instantly |

**Why this matters for security:** Attackers often exploit vulnerable websites, clone them, or hijack them. Understanding the _real_ structure of a URL helps you detect when a link looks "off" — e.g., when the "brand name" appears in the wrong part of the URL (see Section 3).

---

## 3. Identifying Real vs Fake Domains

This is one of the most **practical cybersecurity skills** you can learn — being able to look at a URL and immediately judge whether it's legitimate or a phishing attempt.

### 3.1 Characteristics of a Real/Legitimate Domain

- ✅ **Short and simple** — no unnecessary extra words.
- ✅ **Exact brand name in the SLD position** (e.g., `paypal.com`, not something with "paypal" buried elsewhere).
- ✅ **Trusted TLD** (`.com`, `.gov`, `.org`, etc.)
- ✅ **Valid SSL certificate** (shows `https://` with a proper lock icon).
- ✅ **Clean URL structure** — easy to read and verify at a glance.

### 3.2 Characteristics of a Fake/Phishing Domain

- ❌ **Brand name stuffed as extra words**, not placed correctly in the domain structure.
  - Example of a fake pattern: `paypal-secure-login.xyz.top` — here, "PayPal Secure Login" is jammed together as if it were the whole organization name, when in a real URL, terms like "secure" or "login" would correctly appear in the **subdomain** or **path**, not merged into the domain name itself.
- ❌ **Suspicious/uncommon TLDs** like `.xyz`, `.top`, or other rarely-used extensions favored by attackers.
- ❌ **Missing or invalid SSL certificate** — may still be plain `http://`, or may have a fake/self-signed certificate that a non-technical user won't notice.
- ❌ **Extra unnecessary words** around the brand name meant to create false trust (e.g., "secure", "verify", "official").

**Key insight:** If you see the _entire brand name plus extra words_ crammed together as one domain name (instead of appearing properly as a subdomain or path), it's a strong red flag for a phishing site.

### 3.3 Social Engineering Tactics: Urgency & Fear

Attackers rarely rely on the fake URL alone — they pair it with **psychological pressure**:

- **Urgency:** "This offer is valid only for today!" / "Complete this within 3 hours!"
- **Fear:** "Your account will be blocked if you don't log in immediately!"
- **Reward bait:** "Share this with 5 friends and win an iPhone!" (fake giveaway/viral scam)

**Why this works:** A person under fear or urgency reacts emotionally rather than analytically — they click the link, land on the fake page, and enter their real credentials, which are then stolen by the attacker. This is a form of **data theft via phishing**, and it can spread further when victims unknowingly share the malicious link with others.

**Golden rule:** Genuine organizations do not create panic-driven deadlines for basic account actions. Always verify the URL structure before entering any credentials.

---

## 4. What is DNS (Domain Name System)?

**DNS (Domain Name System)** is a foundational internet concept that acts like a **distributed directory** — it maps human-readable domain names to machine-readable IP addresses.

### 4.1 The Phonebook Analogy

Imagine:

- You want to talk to your friend **Sameer**, but you don't know his number.
- Your friend **Rahul** does have it — but _you_ don't have Rahul's number memorized either, you just look it up in your phone's contact list.
- You search "Rahul" in your directory → find his number → call him → he connects you to Sameer.

**DNS works exactly like this "directory lookup" process for the internet.** When you type `Google.com`, your computer doesn't inherently understand what "Google.com" means — the computer only understands binary (0s and 1s), not human language. DNS is the "directory" that looks up the human-readable name and returns the machine-readable IP address so your browser can connect to the right server.

### 4.2 Why DNS is Necessary

- Humans cannot memorize the IP addresses of millions of websites.
- DNS converts **human-readable domain names → IP addresses** silently, in the background, within milliseconds.
- **Every single online action** — browsing a website, sending an email, streaming a video — relies on DNS resolution happening behind the scenes.
- This is why DNS is often called **"the phonebook of the internet"** and is considered **the backbone of the internet**.

---

## 5. How DNS Works Behind the Scenes

### 5.1 Step-by-Step DNS Resolution Process

When you type `www.wscube.com` into your browser:

```
 User types URL
        │
        ▼
 [1] Browser initiates a DNS query
        │
        ▼
 [2] Recursive Resolver (your ISP — e.g., Jio, Airtel)
        │   → Checks its CACHE first
        │   → If found in cache → returns IP immediately (query ends here)
        │   → If NOT found → forwards the query further
        ▼
 [3] Root Server
        │   → Stores info about all TLDs (.com, .org, .xyz, etc.)
        │   → Redirects to the appropriate TLD server
        ▼
 [4] TLD Server
        │   → Redirects to the correct Authoritative Server
        ▼
 [5] Authoritative Server
        │   → This is the server belonging to the actual organization
        │   → Returns the correct IP address
        ▼
 [6] Browser connects to that IP and loads the website
```

**Breakdown of each entity:**

| Step | Entity                       | Role                                                                                 |
| ---- | ---------------------------- | ------------------------------------------------------------------------------------ |
| 1    | **Browser**                  | Initiates the DNS query when you type a URL                                          |
| 2    | **Recursive Resolver (ISP)** | First checks its own cache for a recent match; if not found, passes the query onward |
| 3    | **Root Server**              | Holds a directory of all TLDs and redirects to the correct one                       |
| 4    | **TLD Server**               | Redirects to the specific authoritative server for that domain                       |
| 5    | **Authoritative Server**     | The actual organization's server — holds the real, final IP address                  |
| 6    | **Browser**                  | Uses the IP to connect and load the website                                          |

All of this happens **invisibly**, in milliseconds, every single time you visit a website — even though the underlying hierarchy (root → TLD → authoritative) is quite involved.

---

## 6. DNS Record Types

Different types of DNS records serve **specific functions** in directing internet traffic — not all traffic/services can rely on a single record type.

| Record Type      | Full Meaning         | Function                                                                                                                                                | Real-World Analogy                                                                                                                         |
| ---------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **A Record**     | Address Record       | Maps a hostname to an **IPv4** address                                                                                                                  | Direct phonebook entry using an old-style phone number                                                                                     |
| **AAAA Record**  | "Quad-A" Record      | Maps a hostname to an **IPv6** address (introduced because IPv4 addresses are limited and running out; IPv6 offers a virtually unlimited address space) | Same purpose as A record, but for the newer, larger addressing system                                                                      |
| **CNAME Record** | Canonical Name       | Points one domain/subdomain to another domain (an "alias") — used when multiple names should resolve to the same underlying resource                    | Like a person having both an official name (e.g., "Ramu" used in documents) and a nickname (e.g., "Ram") — both refer to the _same_ person |
| **NS Record**    | Name Server Record   | Tells the internet **which server holds the authoritative information** for a domain                                                                    | Like a building manager who knows exactly which apartment (server) a specific resident (domain) lives in                                   |
| **MX Record**    | Mail Exchange Record | Ensures that emails sent to a domain (e.g., `contact@wscube.com`) are correctly routed to the right mail server                                         | The "postal routing" system for a company's email                                                                                          |
| **PTR Record**   | Pointer Record       | The **reverse** of an A record — maps an **IP address back to a hostname**                                                                              | Like using a "reverse phone lookup" (e.g., Truecaller) to find out whose number just called you                                            |

**Key distinction:**

- **A/AAAA records:** hostname → IP address (forward lookup)
- **PTR records:** IP address → hostname (reverse lookup)
- **CNAME:** domain/subdomain → another domain name (alias, not a raw IP)

---

## 7. DNS Caching

**Caching** = temporarily storing data so it doesn't need to be looked up again immediately.

- **DNS caching** speeds up DNS resolution by **reducing the time needed to retrieve DNS records** on repeated lookups.
- If a **Recursive Resolver** (your ISP) has recently resolved a domain, it stores that result temporarily — so the next time you (or anyone using that ISP) requests the same domain, it's served instantly from cache, **without going through the entire root → TLD → authoritative server chain again**.

### 7.1 TTL (Time to Live)

- **TTL** defines **how long** a DNS record stays valid in the cache before it must be refreshed/re-fetched.
- Example: If TTL = 15 minutes, the cached record will be used for 15 minutes; after that, a fresh lookup is required.
- Different records can have different TTL values depending on how frequently their underlying data is expected to change.

### 7.2 Why Distributed Name Servers Matter

Organizations never run their entire DNS infrastructure on a **single server** — the load is split across multiple name servers (e.g., N1, N2, N3, N4).

**Why?**

1. **Reliability:** If a single server goes down, the entire organization's web presence would go down with it. Splitting the load across multiple servers avoids a single point of failure.
2. **Security (DDoS mitigation):** During a **Denial of Service (DoS/DDoS) attack**, a flood of requests hits the servers. If load is distributed, and if caching absorbs repeat queries, less traffic actually reaches the origin server — and the sudden traffic pattern also helps the organization detect that an attack is occurring.

**In short:** DNS caching + distributed name servers = faster resolution **and** better resilience against server overload/attacks.

---

## 8. DNSSEC (Domain Name System Security Extensions)

### 8.1 The Core Problem DNSSEC Solves

By default, DNS has a major weakness: it **blindly trusts** whatever response it receives, without verifying whether that response is genuine. This makes it vulnerable to:

- **DNS Spoofing** — an attacker sends a fake DNS response.
- **Cache Poisoning** — an attacker inserts a fraudulent record into a resolver's cache.

**Consequence:** A user gets silently redirected to a **fake website** without any indication something is wrong. They log in normally — and their credentials go straight to the attacker.

**DNSSEC (Domain Name System Security Extensions)** was introduced specifically to solve this trust problem. In simple terms:

> DNSSEC ensures that the DNS response you receive is **authentic** (not faked) and that the data has **not been tampered with** in transit.

### 8.2 Digital Signatures

DNSSEC verifies authenticity using **digital signatures**.

**Real-world analogy:** An official government document carries an official stamp/signature proving it's genuine. Similarly, DNSSEC attaches a **digital signature** to DNS data, proving that the data genuinely came from the original, authoritative source.

**How a digital signature is created — Public Key Cryptography:**

- Uses **two keys**: a **private key** and a **public key**.
- The **private key** is used to **sign** the data.
- The **public key** is used to **verify** the signature.

### 8.3 How DNSSEC Works — Step by Step

```
[1] User sends a request for a website
        │
        ▼
[2] DNS server responds with the data + a Digital Signature
        │
        ▼
[3] The system verifies that signature using the Public Key
        │
        ├── Signature VALID   → Response accepted, treated as authentic
        │
        └── Signature INVALID → Response REJECTED (protects against spoofing)
```

Instead of blindly trusting every response, the system now **verifies first, trusts second**.

### 8.4 Chain of Trust

DNSSEC builds trust in a **hierarchy**:

```
Root  →  TLD  →  Domain
```

- Each level in the hierarchy **verifies the next level down**.
- This creates a continuous, verifiable **"chain of trust."**
- If **any link in the chain is broken**, the response is considered **untrusted and rejected** — even if it looks superficially valid.

### 8.5 Authenticity vs Integrity

Two crucial concepts underpin DNSSEC:

| Term             | Meaning                                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| **Authenticity** | The data genuinely comes from the real, original source                                                 |
| **Integrity**    | The data has **not been altered/tampered with** in transit — what was sent is exactly what was received |

DNSSEC validates DNS responses by ensuring **both** properties hold true simultaneously.

### 8.6 Important: DNSSEC Does NOT Encrypt

This is a commonly misunderstood point:

> ❗ **DNSSEC does not provide encryption.** The data is **not hidden or made private** by DNSSEC.

DNSSEC's job is purely to **verify correctness and authenticity** — confirming that the data is correct, unaltered, and from a legitimate source. It does **not** conceal the contents of DNS traffic from eavesdroppers. (Encryption of DNS traffic is instead handled by separate technologies like DNS-over-HTTPS/DNS-over-TLS, which are outside the scope of DNSSEC itself.)

---

## 9. Summary / Key Takeaways

1. A **URL** consists of: Protocol → Subdomain → Domain (SLD) → TLD → Path → Query String.
2. **Domains** are purchased and rarely change; **subdomains** and **paths** are freely created/changed by developers.
3. Phishing sites often disguise themselves by placing the brand name incorrectly within the URL and pairing this with urgency/fear-based messaging.
4. **DNS** translates human-readable domain names into machine-readable IP addresses — it is the "phonebook of the internet."
5. DNS resolution follows a hierarchy: **Browser → Recursive Resolver (ISP) → Root Server → TLD Server → Authoritative Server**.
6. Key DNS record types: **A** (IPv4), **AAAA** (IPv6), **CNAME** (alias), **NS** (name server info), **MX** (mail routing), **PTR** (reverse lookup).
7. **DNS Caching**, governed by **TTL**, speeds up repeated lookups and reduces load on origin servers.
8. **DNSSEC** adds **digital signature-based verification** to DNS responses to prevent spoofing/cache poisoning — but importantly, it does **not encrypt** DNS traffic; it only verifies authenticity and integrity.

---

## 10. Quick Revision Table

| Concept                  | One-Line Definition                                                               |
| ------------------------ | --------------------------------------------------------------------------------- |
| **URL**                  | The complete address of a specific web resource                                   |
| **Domain**               | The registered, purchased identity of an organization online                      |
| **Subdomain**            | A free, developer-created prefix to the domain (e.g., `blog.`, `cloud.`)          |
| **Path**                 | The specific location of a page within a website                                  |
| **DNS**                  | The system that translates domain names into IP addresses                         |
| **Recursive Resolver**   | Your ISP's server; checks cache first, then queries further up the chain          |
| **Root Server**          | Holds directory info about all TLDs                                               |
| **TLD Server**           | Directs to the correct authoritative server for a domain                          |
| **Authoritative Server** | The final server holding the real IP address for a domain                         |
| **A / AAAA Record**      | Maps hostname → IPv4 / IPv6 address                                               |
| **CNAME Record**         | Alias — one domain name pointing to another                                       |
| **NS Record**            | Identifies which server is authoritative for a domain                             |
| **MX Record**            | Routes email to the correct mail server                                           |
| **PTR Record**           | Reverse lookup — IP address → hostname                                            |
| **TTL**                  | How long a DNS record stays valid in cache                                        |
| **DNSSEC**               | Security extension that verifies DNS response authenticity via digital signatures |
| **Chain of Trust**       | Hierarchical verification: Root → TLD → Domain                                    |
| **Authenticity**         | Data is from a genuine source                                                     |
| **Integrity**            | Data has not been tampered with                                                   |

---

_End of Notes — DNS & Domain Name System_
