## https://youtu.be/cbzYtBFjqtU

# Bug Bounty Beginner Roadmap — Course Notes

> Source: Cybersecurity course video (Bug Bounty Roadmap for Beginners)
> Format: Structured notes for revision

---

## 1. Introduction — The Reality Check

Bug bounty is heavily hyped on social media: "hack and earn money," "work from home," "companies pay you to find bugs." This is _true_, but what's usually left out is the **hard reality**:

- No one tells you how long it takes to get your **first valid bug**.
- No one tells you how many reports get **rejected**.
- No one tells you how much **competition** exists in this field.
- No one tells you how much **patience** is actually required.

### Common Misconceptions vs Reality

| Beginners Think                       | Reality                                                                                                                            |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Work from anywhere, on your own terms | True, but only after real skill-building                                                                                           |
| No corporate politics, no 9–5         | True, but income is irregular                                                                                                      |
| Quick money                           | First valid bug can take **weeks to months**                                                                                       |
| Easy field to enter                   | It's **overcrowded** — many people join because they can't get a job or lack strong skills, so it looks like an "easy" alternative |
| Consistent income                     | **No guaranteed regular income.** You might earn well one month and find nothing the next                                          |

**Key takeaway:** Beginners who lack patience quit early. There is no shortcut — consistency is everything.

---

## 2. What You Actually Need to Start

Three core requirements (more important than any tool, certificate, or expensive hardware):

1. **Patience & Consistency** — Progress is slow in the beginning.
2. **Creativity & Logical Thinking** — Needed to think like an attacker and find non-obvious flaws.
3. **A Habit of Questioning Everything** — Always ask "why does this behave this way?" This mindset is what leads to bug discovery.

You do **not** need:

- A Computer Science degree
- Expensive hardware/tools
- Prior hacking experience

---

## 3. Core Concepts: Bug & Bug Bounty

- **Bug / Vulnerability / Weakness** — A point in a system (e.g., a website) where an attacker can do something unintended, such as accessing a database they shouldn't. All three terms mean the same thing; "vulnerability" is the more technical term.
- **Bug (technical definition)** — Software behaving differently than the developer intended, which an attacker can trigger with unexpected input.
- **Bug Bounty** — Companies legally invite researchers to find and report vulnerabilities in exchange for rewards. **This is 100% legal**, not hacking in the illegal sense — companies explicitly authorize this testing.

### Types of Rewards

Rewards aren't always cash. You may receive:

- Cash
- Hall of Fame recognition
- Swag
- Conference invites

### Reward Ranges by Severity

| Severity | Example Bug                 | Typical Reward Range                            |
| -------- | --------------------------- | ----------------------------------------------- |
| Low      | Minor information leak      | $50 – $300                                      |
| Medium   | Reflected XSS               | $300 – $900(approx.)                            |
| High     | IDOR, Authentication Bypass | $900 – few thousand $                           |
| Critical | RCE, Full System Takeover   | $$ thousands, potentially $1 lakh+ (and beyond) |

_(Reward amounts vary by company/program — these are general industry ranges.)_

---

## 4. The Complete Roadmap (Overview)

| Level            | Focus Areas                                                                    |
| ---------------- | ------------------------------------------------------------------------------ |
| **Beginner**     | Computer fundamentals → Networking → Linux → Command line → Programming basics |
| **Intermediate** | Web security → OWASP Top 10 → Tools & setup → Practice                         |
| **Advanced**     | CTFs/labs practice → Real bug hunting                                          |

> **Rule:** Don't skip phases even if you think you already know them — revise anyway.

---

## Phase 1: Computer Fundamentals

Basic (roughly class 9th–10th level) concepts you must be solid on:

- Hardware vs Software
- Input vs Output
- Operating System vs Kernel (and the difference between them)
- How file systems are organized
- How to install Windows / Linux
- What Linux is
- How to create and manage **Virtual Machines**

### Why Virtual Machines?

You cannot legally attack real/live systems. VMs let you build a **safe, legal practice lab** on your own laptop:

- One VM can act as the "attacker" machine.
- Another VM can act as the "victim" machine (e.g., a vulnerable machine).
- No risk to your main/host system — the VM is isolated.
- **Snapshots**: Save the VM's state so you can roll back if it crashes or breaks — like a save point.

**Tools:** VirtualBox or VMware (both free & beginner-friendly). VMware recommended for beginners.

---

## Phase 2: Networking

**Why it matters:** Every attack starts from the network. If you don't understand the starting point of an attack, you can't detect or reproduce it. This phase **cannot be skipped** — it's the #1 mistake beginners make (jumping straight to tools/hacking).

### OSI Model — Focus on 3 Key Layers

| Layer   | Name                  | Why It Matters                                                                                                           |
| ------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Layer 7 | **Application Layer** | Most important — HTTP, DNS and other application protocols live here. Nearly all web-based attacks happen at this layer. |
| Layer 4 | **Transport Layer**   | TCP, UDP, Ports — how data is delivered from sender to receiver                                                          |
| Layer 3 | **Network Layer**     | IP addressing — how data reaches its destination                                                                         |

### How the Web Works — Must-Know Topics

- **DNS Lookup** — Domain → IP conversion. Go deeper: DNS records (A record, NS record, etc.), name servers.
- **TCP Handshake** — 3-way handshake, 4-way handshake (termination), why UDP has no handshake, speed vs reliability trade-offs (TCP vs UDP).
- **TLS Handshake** — How encryption/security is established.
- **HTTP Request/Response Flow** — How a request travels to the server, how the server responds, and how the browser renders the page.

**Don't stop at basic definitions** (what is IP, what is DNS) — go deeper into how these actually function.

---

## Phase 3: Linux

**Why Linux?**

- It's **open source**.
- Most hacking tools come **pre-installed** (e.g., Kali Linux has 600+ pre-installed tools).

**Important distinction:**

- **Linux = a kernel**, NOT an operating system.
- **Kali Linux = an operating system** built on the Linux kernel.

### What to Learn

- **File system structure** — Linux's directory structure is very different from Windows.
- **Permissions** — Read, Write, Execute (RWX); how to change/grant/revoke permissions.
- **Users** — Regular users vs Admin/root users, and their roles.
- **Package management** — How to install tools/software.

**Distro recommendations:**

- Beginner-friendly starting point: Ubuntu (or similar)
- For hacking-focused work: **Kali Linux** or **Parrot OS**

---

## Phase 4: The Terminal / Command Line Interface (CLI)

Linux distros like Kali are terminal/CLI-based (unlike Windows' GUI). You **must** be comfortable with the terminal — there's no way around it.

### Command Categories to Learn

1. **File & Navigation commands** — `ls`, `cd`, `cat`, etc.
2. **Networking commands** — finding your IP, scanning the network, etc.
3. **Advanced/Power commands** — `grep` (search text inside files), changing permissions, adding new users, etc.

---

## Phase 5: Programming Basics

**Important mindset shift:** As a beginner in bug bounty, **you don't need to _write_ code — you need to _read and understand_ code.** You're not becoming a developer; you're learning to understand already-built systems (websites, tools).

If your coding knowledge is weak, you can:

- Use free resources (YouTube, Google, ChatGPT/AI) to learn basics.
- Use AI to explain code snippets you don't understand.

### Languages to Get Familiar With (basics only — syntax, structure, purpose)

| Language       | Purpose                                                                 |
| -------------- | ----------------------------------------------------------------------- |
| **HTML**       | Defines a web page's structure                                          |
| **JavaScript** | Client-side logic language — most client-side attacks happen through JS |
| **Python**     | Scripting language, useful for automation                               |
| **SQL**        | Needed to understand how queries work (critical for SQL Injection)      |
| **PHP**        | Server-side language — basic understanding needed                       |

**Don't over-invest time here.** Make a plan (e.g., "today I learn X, tomorrow Y") so you don't get stuck trying to "fully learn" development.

---

## Phase 6: Web Security & OWASP Top 10

This is where you start learning to actually attack/assess websites.

### For Every Vulnerability, Learn 4 Things:

1. What is this bug?
2. How do you **find** it?
3. How do you **exploit** it?
4. How do you **fix** it?

### OWASP Top 10 (Industry-standard vulnerability list — your primary hunting checklist)

1. Broken Access Control
2. Cryptographic Failures
3. Injection (SQL Injection, XSS, etc.)
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable & Outdated Components
7. Authentication Failures (Identification & Authentication Failures)
8. Data Integrity Failures (Software & Data Integrity Failures)
9. Logging & Monitoring Failures (Security Logging & Monitoring Failures)
10. SSRF (Server-Side Request Forgery)

**Practice ground:** PortSwigger Web Security Academy (free) — register and solve their labs. Even failed attempts teach you where your gaps are.

### Bugs by Difficulty Level

**Basic (most common for beginners):**

- **XSS (Cross-Site Scripting)** — Injecting scripts. Three types: Reflected, Stored, DOM-based. Impact: cookie theft, session hijacking.
- **IDOR (Insecure Direct Object Reference)** — Changing an ID/reference in a URL and gaining access to another user's data/session. **Always test with two separate accounts** to detect this.

**Intermediate:**

- SQL Injection
- CSRF (Cross-Site Request Forgery)
- Open Redirect

**Advanced (come to these later, after gaining experience):**

- SSRF
- SSTI (Server-Side Template Injection)
- XXE (XML External Entity)
- File Upload Vulnerabilities

> Don't jump straight to advanced bugs. Master basic/intermediate first — advanced bugs will naturally come with experience.

---

## Phase 7: Build Your Own Hacking Lab

### Requirements

- **Virtualization software:** VirtualBox or VMware
- **OS:** Kali Linux or Parrot OS

### Essential Tools

**Burp Suite** (most important tool — cannot be skipped)

- Intercepts requests going from browser to server so you can analyze/modify them.
- Start with the free **Community Edition** as a beginner.

Key Burp Suite modules to learn:
| Module | Purpose |
|---|---|
| Proxy | Intercepts requests (works with FoxyProxy to route traffic through Burp) |
| Repeater | Where you experiment most with requests |
| Intruder | Automates/fuzzes requests |
| Decoder | Encode/decode data |
| Comparer | Spot differences between two responses |
| Collaborator & others | Explore further as needed |

**FoxyProxy** — Browser extension used to route traffic into Burp Suite's proxy.

**SecLists** — GitHub repository of wordlists, extremely useful for brute-force/dictionary attacks.

### Recon / Recommended Tools by Category

| Purpose                | Tools                                            |
| ---------------------- | ------------------------------------------------ |
| Subdomain enumeration  | Sublist3r, Amass, Assetfinder                    |
| Detecting live hosts   | httpx, Naabu, DNSx                               |
| Port scanning          | Nmap, RustScan, Masscan                          |
| Finding hidden URLs    | Waybackurls, GAU (Gospider/Ghost Finder), Katana |
| JavaScript analysis    | LinkFinder, SecretFinder                         |
| Fuzzing                | ffuf, Gobuster, directory search tools, Arjun    |
| Vulnerability scanning | Nuclei, Nikto                                    |
| SQL Injection          | SQLMap                                           |
| XSS detection          | Dalfox                                           |
| Secret leaks           | TruffleHog, GitLeaks                             |
| Exposed services/recon | Shodan (search engine for exposed servers/IPs)   |

### Resources

- **SecLists** — wordlists for every bug type
- **PayloadsAllTheThings** — payloads for each vulnerability type
- **HackTricks** — step-by-step techniques
- **Nuclei Templates** — community scanner templates

---

## Phase 8: Practice Before You Hunt

**Do not jump directly into live bug hunting after learning theory.** Practice first on labs/CTFs.

| Level                  | Platforms                          |
| ---------------------- | ---------------------------------- |
| Beginner-friendly      | TryHackMe, PortSwigger Web Academy |
| Intermediate           | Hacker101 CTF, Hack The Box        |
| Offline practice       | DVWA, OWASP Juice Shop, VulnHub    |
| Vulnerable OS practice | Metasploitable                     |

---

## 5. Starting Real Bug Hunting

### Platforms (Public Programs)

- **Bugcrowd**
- **HackerOne**
- Intigriti
- Open Bug Bounty
- Synack (invite-only)

> Beginners should start with **public** programs. **Private programs are invite-only** and reserved for experienced hunters who get specifically invited.

### Choosing & Working a Program

1. Always choose a **public** program first.
2. Read the **scope** carefully — what's allowed, what's not allowed, which domains are in/out of scope. Test strictly within scope.
3. Build your **reputation** on the platform through consistency and **quality reports**.
4. Once you perform well on public programs, you'll start getting **invited to private programs**.

### What to Look For in a Program (as a Beginner)

- Choose **large-scope** programs (bigger attack surface).
- Hunt on things you know well.
- Spend a reasonable amount of time (1–2 weeks) per target — go deep, but don't get stuck for months on one target. Move on if nothing is found after a fair effort.

### Things to Avoid

- ❌ **Don't target huge companies** (Apple, Google, Meta) as a beginner — these require deep experience.
- ❌ **Don't jump between programs too frequently** — commit meaningful time to each before switching.
- ❌ **Don't rely only on automated tools** — many bugs require manual, creative testing that tools can't find.

---

## 6. Writing a Bug Bounty Report

A strong report should include:

1. **Title** — Bug type + impact
2. **Short Description** — What the bug is and what it affects
3. **Impact** — Clearly explain the real-world consequences
4. **Steps to Reproduce** — Clear, reproducible steps
5. **Proof of Concept (PoC)** — Most important part. Include screenshots or screen recordings proving the bug is real.
6. **Suggested Fix** — Recommendations on how to remediate the bug

> Tip: You can use AI to help format and refine your report.

---

## 7. Mindset & Long-Term Habits

- **Skills first, money follows** — Don't jump straight to bug hunting without building real skills.
- **Compete with yourself, not others** — Don't compare your progress to people with years of experience.
- **Every rejected report is a lesson** — Learn from rejections; don't get demotivated.
- **Rewards will come** — With consistent skill-building and effort, results follow (even if delayed).
- **Build a research/writing habit:**
  - Write your own blogs (e.g., on Medium) documenting labs you've solved.
  - Read other hackers' write-ups and blogs.
  - Watch tutorials on new techniques.
  - Stay updated on trending vulnerabilities, outdated software versions, and new tech releases — outdated versions are often easy wins.

---

## Quick-Reference Roadmap Summary

```
1. Computer Fundamentals (OS, kernel, file systems, VMs)
2. Networking (OSI model, DNS, TCP/TLS handshakes, HTTP)
3. Linux (file system, permissions, users, package mgmt)
4. Terminal / CLI commands
5. Programming basics (HTML, JS, Python, SQL, PHP — read, don't need to write)
6. Web Security + OWASP Top 10 (find/exploit/fix for each bug)
7. Build a hacking lab (VM + Kali/Parrot + Burp Suite + recon tools)
8. Practice on CTFs/labs (TryHackMe, PortSwigger, HTB, DVWA)
9. Start bug hunting on public programs (Bugcrowd, HackerOne)
10. Write high-quality reports → build reputation → get private invites
```
