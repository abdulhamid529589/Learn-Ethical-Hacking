# Sherians Cyber School — VAPT Series, Part 1: Foundations, Methodology & Reconnaissance

**Instructor:** Abhishek Chourasia
**Channel:** Sherians Cyber School
**Topic:** Introduction to Vulnerability Assessment and Penetration Testing (VAPT) — Mindset, Terminology, Methodology, Types of Testing, and Passive/Active Reconnaissance Techniques

> **Note on this document:** The original video was delivered in Hindi (with English technical terms mixed in) as Part 1 of a 3-part VAPT series. This document translates and organizes the lecture into structured English notes for study purposes. The instructor repeatedly and explicitly frames this content as **authorized, professional security testing** — not unauthorized hacking — and this document preserves that framing throughout. Real domains/IPs used as live demo targets in the original video have been generalized to "a target domain/IP" in these notes, and no credentials, leaked data, or sensitive discovered content from the demo is reproduced here — only the **techniques and tool names**, which are standard, publicly documented methodology taught in any professional pentesting/OSCP-style course.

---

## 📑 Table of Contents

1. [Series Overview: What the 3-Part VAPT Series Covers](#1-series-overview-what-the-3-part-vapt-series-covers)
2. [What VAPT Actually Is (and Isn't)](#2-what-vapt-actually-is-and-isnt)
3. [Why Organizations Perform VAPT](#3-why-organizations-perform-vapt)
4. [Common Beginner Misconceptions](#4-common-beginner-misconceptions)
5. [How Cybersecurity Differs From Other Fields](#5-how-cybersecurity-differs-from-other-fields)
   - 5.1 [Bugs vs. Vulnerabilities](#51-bugs-vs-vulnerabilities)
   - 5.2 [Why Traditional Troubleshooting Fails for Security](#52-why-traditional-troubleshooting-fails-for-security)
6. [Categories of Penetration Testing](#6-categories-of-penetration-testing)
   - 6.1 [Black Box Testing](#61-black-box-testing)
   - 6.2 [White Box Testing](#62-white-box-testing)
   - 6.3 [Gray Box Testing](#63-gray-box-testing)
7. [Core VAPT Terminology](#7-core-vapt-terminology)
8. [VAPT Methodology Overview](#8-vapt-methodology-overview)
9. [Types of Penetration Testing](#9-types-of-penetration-testing)
   - 9.1 [Network Penetration Testing](#91-network-penetration-testing)
   - 9.2 [Web Application Penetration Testing](#92-web-application-penetration-testing)
   - 9.3 [Mobile Application Penetration Testing](#93-mobile-application-penetration-testing)
   - 9.4 [Social Engineering Penetration Testing](#94-social-engineering-penetration-testing)
   - 9.5 [Physical Penetration Testing](#95-physical-penetration-testing)
10. [Report Writing](#10-report-writing)
    - 10.1 [Understanding the Audience](#101-understanding-the-audience)
11. [Reconnaissance: The Foundation of Everything](#11-reconnaissance-the-foundation-of-everything)
    - 11.1 [Passive Reconnaissance](#111-passive-reconnaissance)
    - 11.2 [Active Reconnaissance](#112-active-reconnaissance)
12. [Three Levels of Information Gathered](#12-three-levels-of-information-gathered)
    - 12.1 [Organization-Level Information](#121-organization-level-information)
    - 12.2 [Network-Level Information](#122-network-level-information)
    - 12.3 [System-Level Information](#123-system-level-information)
13. [Practical Demonstration: Browser-Based Reconnaissance](#13-practical-demonstration-browser-based-reconnaissance)
    - 13.1 [Google Dorking](#131-google-dorking)
    - 13.2 [Useful Browser Extensions](#132-useful-browser-extensions)
14. [Practical Demonstration: DNS Enumeration](#14-practical-demonstration-dns-enumeration)
15. [Practical Demonstration: Port Scanning (TCP Recon)](#15-practical-demonstration-port-scanning-tcp-recon)
16. [Practical Demonstration: SMB Enumeration](#16-practical-demonstration-smb-enumeration)
17. [Practical Demonstration: SMTP Enumeration](#17-practical-demonstration-smtp-enumeration)
18. [Practical Demonstration: SNMP Enumeration](#18-practical-demonstration-snmp-enumeration)
19. [Summary of Key Takeaways](#19-summary-of-key-takeaways)
20. [Quick-Reference Glossary](#20-quick-reference-glossary)

---

## 1. Series Overview: What the 3-Part VAPT Series Covers

This video is the first installment of a three-part series on **VAPT (Vulnerability Assessment and Penetration Testing)**, intended to explain how the cybersecurity industry actually operates professionally — not "hacking for fun."

**Part 1 (this video) — Foundation & Mindset:**

- Why VAPT matters and how the field differs from others.
- Core terminology needed before attempting any testing.
- VAPT methodology.
- Scanning and information gathering (reconnaissance).
- Understanding testing **scope** (network vs. web vs. other targets).
- Analyzing discovered weaknesses (loopholes) and their potential business impact.

**Part 2 — Exploitation & Post-Exploitation:**

- Mindset for exploiting web applications and systems.
- What to do _after_ successfully exploiting a target (post-exploitation).
- Handling situations where a vulnerability has since been patched.
- Live demonstrations.

**Part 3 — Reporting, Career & Practice:**

- Reporting skills, including use of AI tools for report writing.
- Risk rating and prioritization (comparing findings to decide what to fix first).
- Solving intermediate-level CTF (Capture the Flag) challenges.
- Career guidance: building a resume and portfolio.

> By the end of the series, the goal is to be prepared to perform a standard security audit of an organization, network, or server.

[⬆ Back to top](#-table-of-contents)

---

## 2. What VAPT Actually Is (and Isn't)

> **VAPT is not about hacking for fun or randomly breaking into systems.**

- **VAPT = Vulnerability Assessment and Penetration Testing** — a **structured security process** used by organizations to:
  1. **Identify** security risks.
  2. **Validate** whether those risks are real.
  3. **Prioritize** them for remediation.
- After identifying and prioritizing, issues are addressed systematically, one at a time.

**Why VAPT is performed in the real world:**

- To understand **how an attacker would think** — not how defenders think. The goal is to reason as an adversary would: how would they get in, how would they exfiltrate data?
- To **discover weaknesses before criminals do.**
- To **reduce business and data risk** — since a data breach directly impacts the business.

**Illustrative example — the 2023 Ferrari data leak:**

- A relatively small number of people own Ferraris (very wealthy individuals), so a leak of Ferrari customer data (potentially including financial/banking details) is meaningful — attackers could infer a great deal about a person's net worth just from knowing they own a Ferrari, giving attackers a specific motive/intent.

[⬆ Back to top](#-table-of-contents)

---

## 3. Why Organizations Perform VAPT

Recap of the core process: **identify → validate → prioritize risks**, then work through them systematically.

[⬆ Back to top](#-table-of-contents)

---

## 4. Common Beginner Misconceptions

The instructor explicitly pushes back on misinformation from "influencers":

- **Myth:** "Cybersecurity jobs aren't available for freshers." — **False.** (The instructor notes getting a job at age 16 as a counterexample.)
- **Myth:** VAPT is just running a tool and exploiting everything possible ("get a shell, get a screenshot, done — system hacked").
- **Reality:** VAPT involves **deliberately choosing what to test and what not to test**, since many client organizations specify particular scope boundaries (certain systems are explicitly in-scope, others explicitly out-of-scope).
- Testers must also understand **why a particular vulnerability actually matters** in context — the same vulnerability can carry very different levels of importance depending on the organization (see Section 6.3 discussion of context-dependent risk).
- Testers must know when to **stop at the right point**, without causing unnecessary damage — not everything that _can_ be accessed or escalated _should_ be, during a professional engagement.

[⬆ Back to top](#-table-of-contents)

---

## 5. How Cybersecurity Differs From Other Fields

- In cybersecurity, you must **think like an attacker** — this is different from most other technical fields, where the mindset is fundamentally about **building** things, not defending them.
- Across most fields, people collaborate to make things work well. In cybersecurity specifically, the job is to ensure that whatever developers/operations build **runs securely**.

### 5.1 Bugs vs. Vulnerabilities

> **A bug** is usually a mistake — an error in logic, or something that causes an application to misbehave (unintended behavior), without necessarily being a security issue.

- Example given: in the classic Snake game, the snake growing when it eats itself would be a bug/quirky behavior, not a vulnerability.

> **A vulnerability** is a weakness that can be **intentionally abused**, leading to unauthorized access, impact, and security consequences.

- **Key distinction:** Not every bug is a vulnerability, but every vulnerability is dangerous, because someone can deliberately exploit it.
- **Common misconception addressed:** People often think "vulnerability" just means "weakness" in general. Not quite — a vulnerability specifically refers to a weakness that **someone else can take advantage of**. (Not every personal weakness is something others can exploit — the same logic applies to systems.)

### 5.2 Why Traditional Troubleshooting Fails for Security

**Traditional IT troubleshooting approach:**

1. Investigate what's happening with your application/network/server.
2. Fix the identified issue.

**Why security doesn't work this way:**

- A system can **appear to work perfectly**, with **no visible errors**, while something is quietly wrong underneath. Security problems don't always announce themselves the way functional bugs do.
- **Security problems are deliberately created** (by an attacker), not accidental — and attackers **adapt and try again repeatedly**. This ongoing, intentional, adaptive nature is what makes cybersecurity fundamentally different from standard IT troubleshooting.

[⬆ Back to top](#-table-of-contents)

---

## 6. Categories of Penetration Testing

There are three major categories: **Black Box, White Box,** and **Gray Box.**

### 6.1 Black Box Testing

> **Black box testing** means the tester has **no prior knowledge of the internal architecture** — no access to source code, no detailed system knowledge. The tester is essentially treated as a complete outsider.

**Focus of black box testing:**

- Test all **external exposure** — what could a hacker discover/access from outside, with zero insider knowledge?
- Evaluate **realistic external attack paths.**
- Understand what an outsider could discover and exploit.

### 6.2 White Box Testing

> **White box testing** means the tester has **full knowledge** of the system architecture, application details, services in use, and (for applications) the **source code**.

- **The goal here is different:** it's **not** to simulate an outside attacker, but to **deeply analyze security flaws** with full access/context — as if evaluating what someone with legitimate internal access to the company could potentially do.

**What white box testing enables:**

- **Thorough/full coverage** — essentially everything about the target can be examined.
- **Faster identification of critical issues**, since the tester already has authorized, direct access to everything relevant.
- **Validation of security at the design and code level** — meaning security issues can potentially be caught and fixed **while the system is still being built**, before it's ever exposed to the public.

### 6.3 Gray Box Testing

> **Gray box testing** provides the tester with **limited but useful information** — a middle ground between black box and white box.

- Example: the tester might be given an application name and a regular **user account**, but not full administrative access or source code — a "partial system."
- **Why this matters:** Gray box testing simulates a **semi-trusted user** — for example, an existing employee within the company who decides to act maliciously (an "insider threat" scenario — "a Vibhishan from within," referencing the mythological insider-betrayal figure).
- **Benefits:** Allows testing of **realistic attack scenarios** and achieving good coverage, **without needing full disclosure** of everything about the system.

[⬆ Back to top](#-table-of-contents)

---

## 7. Core VAPT Terminology

| Term                       | Definition                                                                                                                                                                                                                                                |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Asset**                  | Anything that has value and must be protected — not limited to systems/servers; can be any data, application, service, or component supporting information-related activities (even something as mundane as a security guard's logbook, if it has value). |
| **Mindset for assets**     | Before testing anything, always first identify **what actually needs protection.**                                                                                                                                                                        |
| **Vulnerability**          | A weakness or flaw present within an asset — can be technical, logical, or configuration-related. Importantly, a vulnerability **does not mean a system is already compromised** — it only means there is a **possibility** of compromise.                |
| **Threat**                 | A potential danger to an asset — an actor/factor with the _capability_ to exploit an asset or a vulnerability. A threat exists independently of vulnerabilities, but only becomes _effective_ once a vulnerability is actually present for it to exploit. |
| **Exploit**                | A method or technique used to take advantage of a vulnerability. An exploit transforms a _theoretical_ weakness into an _actual_ compromise — it is the bridge between a vulnerability and unauthorized access.                                           |
| **Risk**                   | The potential damage or loss resulting from a _successful_ exploitation.                                                                                                                                                                                  |
| **Attack surface**         | The total set of points where an attacker could potentially attack — the larger the attack surface, the more opportunities exist for compromise.                                                                                                          |
| **Attack vector**          | The specific path/technique used to carry out an attack.                                                                                                                                                                                                  |
| **PoC (Proof of Concept)** | A demonstration of exactly how a particular vulnerability was exploited.                                                                                                                                                                                  |
| **Impact**                 | The consequence/effect after a vulnerability has been exploited.                                                                                                                                                                                          |
| **False positive**         | When a vulnerability is reported as present, but it does not actually exist in reality.                                                                                                                                                                   |

> **Important nuance emphasized:** A vulnerability existing does **not** guarantee it will be exploited; and even if exploited, the impact is **not** guaranteed to be severe. These are separate, sequential considerations — not automatic escalations.

[⬆ Back to top](#-table-of-contents)

---

## 8. VAPT Methodology Overview

> Professional VAPT methodology is **controlled, repeatable, and explainable** — meaning testers can perform it themselves, test it, and later present/justify it to others.

**Where reconnaissance fits into the overall flow:**

1. **Understand the system** — how it's structured.
2. **Identify the attack surface.**
3. **Shortlist weak points** worth focusing on.
4. **Exploit** where vulnerabilities are found, where impact could be significant, and where the scope clearly justifies testing that particular path.

**Why reporting is not the same as hacking:**

- Even a highly sophisticated exploitation attempt is worthless professionally if it **cannot be clearly reported** to a CISO or the relevant report recipient. If you can't explain what you did and why it matters, the technical accomplishment doesn't count for much in a business/client context.

[⬆ Back to top](#-table-of-contents)

---

## 9. Types of Penetration Testing

### 9.1 Network Penetration Testing

Focuses on discovering flaws/weaknesses across an entire network environment. Divided into two sub-types:

**External network penetration testing:**

- Targets **publicly facing** assets — public IP addresses and publicly available resources.
- Involves an **attack simulation**: what could someone sitting outside the organization actually do/access/compromise?
- Commonly used to test **firewalls, perimeter security,** and **exposed services** (services that are already publicly available by design, which therefore don't need to be separately "discovered" — but still need to be checked for security issues).

**Internal network penetration testing:**

- Performed **from inside** the organization's network (as if already connected internally) — testing what becomes accessible once inside.
- Includes discovering things like exposed printers, cameras/CCTV systems, and other internally reachable devices (the instructor references a prior video on basic networking tricks where a printer and some CCTV system information were discovered this way).
- Evaluates **insider threats** and **compromised internal systems** — e.g., a "clever" employee who has opened up things they shouldn't have.
- Access may come via **VPN** or by **physically visiting** the organization.

### 9.2 Web Application Penetration Testing

- Focuses specifically on testing things like usernames, passwords, credit card information, personal information, and financial information — anything related to a **web application**.
- Treated as its **own separate, significant category** within cybersecurity, with its own dedicated weight/focus within an overall VAPT engagement.

### 9.3 Mobile Application Penetration Testing

- Similarly, if the target scope includes a mobile app (Android or iOS), that also needs to be tested as part of an engagement.

### 9.4 Social Engineering Penetration Testing

> Described as the instructor's **personal favorite** part of VAPT.

- Fundamentally about **targeting humans** to extract information from them, rather than targeting technical systems directly.
- **May be conducted as part of a network test**, targeting employees or an organization's systems.
- **Goal:** Get target employees to perform some kind of action that results in them inadvertently handing over information themselves.

**Common techniques named:**

- **Phishing**
- **Spear phishing**
- **Browser-based attacks**
- **Keyloggers**
- **Network/traffic sniffing**

**Illustrative real case shared by the instructor:**

- A scammer was contacted on Telegram. To locate the scammer, the instructor needed to obtain their IP address (e.g., via an IP-logging/"Grabify"-style link) to trace their location. By repeatedly complimenting the scammer's video content, the scammer became curious and asked for a link — the instructor sent a disguised link, successfully obtaining the needed information to locate them.

### 9.5 Physical Penetration Testing

- Covers physical security elements: **locks, access cards, RFID systems, surveillance (CCTV), and other physical hardware** — anything tangible, digital or otherwise, that can be physically interacted with.

[⬆ Back to top](#-table-of-contents)

---

## 10. Report Writing

**Core qualities of a good report:**

- **Simple, clear, and easy to understand.**
- **Properly formatted** — most companies provide a standard template to follow.
- **Well-structured, logical, and organized.**
- **Consistent writing style and tone** throughout.

> **Core principle:** A good report should require **minimal effort from the reader** to understand — the reader should immediately grasp that something is genuinely wrong, without having to work hard to figure it out.

### 10.1 Understanding the Audience

Different audiences read a VAPT report from different perspectives, and the report must serve all of them:

**Executive audience (CEO, CFO, and other C-suite leaders):**

- Not necessarily technical people — top leadership.
- They will read the **executive summary**: overall risk level, business impact, and high-level recommendations (e.g., "here's the budget needed to fix this").

**Management audience (CISOs, security managers):**

- Reviews overall **security posture/strength** — what's good, what's weak, what risk level applies to different findings.
- Focuses on **remediation priorities** — what needs to be fixed, by when, and when patches/fixes will be released.

**Technical audience (developers, technical/SOC teams):**

- Wants to see **whether the vulnerability is actually real**, with proof — the specific technical path/issue, and how to actually **fix it**.

> **A good VAPT report is structured so that executives understand the risk, management understands the priorities, and technical staff know exactly what to fix.**

[⬆ Back to top](#-table-of-contents)

---

## 11. Reconnaissance: The Foundation of Everything

> **Reconnaissance ("recon") is described as the single most valuable skill for any attacker or tester** — the better the recon, the dramatically easier everything else becomes.

- **Reconnaissance is the first phase of VAPT**, where attackers and testers alike focus on **information gathering.**
- **Why it matters:** Having a solid, well-prepared set of information about a target makes eventual exploitation attempts significantly easier.

There are two fundamental approaches to gathering this information:

### 11.1 Passive Reconnaissance

> **Passive reconnaissance** means gathering information **without directly interacting with the target at all.**

**Analogy used:** If you're interested in getting to know someone, rather than approaching them directly, you might first ask their **friend** about them — without ever contacting the person of interest directly (no visiting their social media, no direct interaction). You're extracting information about the target **through an intermediary**, so the target has no idea they're even being looked into.

- **Technically:** No packets are sent directly to the target's own infrastructure.
- **Mindset:** Behave like a **silent observer**, not an active attacker.
- **Definition:** Collecting information **without directly interacting** with the target.

**Types of information typically gathered passively:**

- Domains, subdomains, IP ranges (indirectly).
- **Tech stack** — frameworks, CMS platforms, cloud hosting details.
- Employee names, contact numbers, and social media profiles.
- **Public documents** — press releases, news coverage, balance sheets/financial filings.
- Previously **exposed credentials** (from past breaches), court records, and **DNS records.**

**Why this matters:** This builds a **mental map** of the organization, reduces "noise" before any active scanning begins (since you already have context), can be applied in real-world red teaming, and forms the **base for the overall attack surface.**

### 11.2 Active Reconnaissance

> **Active reconnaissance** means **directly interacting** with the target system to extract **technical details.**

**Analogy used:** Continuing the earlier analogy — rather than only asking a friend about someone, you now **directly approach the person yourself** (e.g., reaching out to them directly on Instagram).

**Mindset:** You are now **confirming/validating the assumptions** made during passive reconnaissance.

**What is typically discovered during active recon:**

- Which **hosts/systems are alive.**
- Which **ports are open**, and which **services** are running on them, including **version numbers.**
- Basic-to-intermediate details about the **operating system.**
- Number of **folders/files**, **API endpoints**, application behavior, and network responses/banners.

**Why active recon matters:**

- Converts theoretical assumptions into **verified, validated data.**
- Reveals **actual attack vectors.**
- Fulfills the prerequisites needed before attempting real exploitation.
- Helps validate what's needed to properly **measure impact** later.
- **Avoids "blind exploitation attempts"** — i.e., randomly guessing where to attack. With proper active recon, you know **exactly** where a specific attack is likely to succeed, rather than guessing.
- Makes the overall testing process **structured and logical**, rather than haphazard.

[⬆ Back to top](#-table-of-contents)

---

## 12. Three Levels of Information Gathered

Information collected during reconnaissance falls into three broad categories:

### 12.1 Organization-Level Information

- Organization **name, brand, subsidiaries, business model, industry domain**, and **physical locations.**
- **Email formatting conventions** and known **email domains.**
- Whether the organization has a **global presence.**
- **Why it matters:** This is highly sensitive information that helps profile the organization (e.g., what technologies they hire for, indirectly revealing their tech stack), and it's especially useful for planning **social engineering** approaches. It also provides important **context** for later technical findings.
- Includes learning what **technologies** the organization's job postings reference (revealing likely tech stack), and what **third-party services/vendors** they use — helping indirectly identify the attack surface.

### 12.2 Network-Level Information

- **Public IP ranges**, domains and subdomains, **DNS records**, network blocks, **ASN (Autonomous System Number) details** (revealing network configuration).
- **Cloud services** in use, **VPN/proxy gateways**, **firewalls** in use.
- Any **third-party hosted assets.**
- **Why it matters:** This establishes the **technical boundaries** of the engagement — critically, it also reveals what is explicitly **out of scope** (things that must not be touched at all). It forms the base for later **scanning and enumeration.**

### 12.3 System-Level Information

- **Operating system, web server, web application framework, CMS,** and **database technologies** in use.
- **Software versions** in use — important because **outdated software versions represent a strong signal for exploitation** (a strong recommendation is made here to always keep your own personal devices updated, as a general best practice).
- **Security headers, configurations,** and **application entry points.**
- **Why it matters:** This reveals **potential vulnerability classes** and informs which **tools** should be used. It specifically helps **avoid "blind exploitation attempts"** (explained above in Section 11.2) by making the testing process **structured and logical.**

[⬆ Back to top](#-table-of-contents)

---

## 13. Practical Demonstration: Browser-Based Reconnaissance

Before diving into tools, the instructor identifies what kind of information should be gathered about any target organization: **domain, subdomains, IP/IP ranges, OS, services and their versions, employee names/addresses, revenue information, tech stack,** and any discoverable **files/directories** within a given website or IP address.

> **Important note from the instructor:** Different tools/techniques are demonstrated across _different_ example targets throughout the video purely for **teaching purposes** (to show a variety of results) — this does not mean every technique should always be scattered across different targets in real practice. When performing an actual engagement, testing should be applied consistently to one authorized target, using a proper, structured methodology.

### 13.1 Google Dorking

> **Google Dorking (also known as Google Hacking)** is a reconnaissance technique that uses **advanced Google search operators** to locate information, files, or vulnerabilities on websites that were not intended to be publicly discoverable in that way (even though the content itself may technically be publicly accessible).

- **Important distinction stressed by the instructor:** This is **not hacking** — no unauthorized access or system compromise is taking place. It is simply using advanced search syntax to surface publicly indexed content that a normal, basic search wouldn't easily reveal.

**Common Google Dork operators demonstrated:**

| Operator                | Purpose                                                                                               |
| ----------------------- | ----------------------------------------------------------------------------------------------------- |
| `site:`                 | Restrict results to a specific domain or top-level domain (e.g., `.gov`, `.edu`)                      |
| `inurl:`                | Find pages where the URL contains a specific keyword (e.g., `admin`)                                  |
| `intitle:`              | Find pages where the page title contains a specific keyword (e.g., `login`)                           |
| `filetype:` (or `ext:`) | Find files of a specific extension (e.g., `pdf`, `sql`, `xls`, `db`, `cfg`/`conf`)                    |
| `intext:`               | Find pages containing specific text within the visible page content (e.g., a keyword like "password") |
| `index of`              | Find open/exposed directory listings                                                                  |

**Demonstrated use cases:**

- Combining operators (e.g., `site:` + `inurl:admin` + `intitle:login`) to locate **admin login pages** on a specific category of site.
- Using `filetype:pdf` combined with `site:.gov` to find publicly indexed PDF documents.
- Using `filetype:sql` or `ext:sql` to locate potential **database dump or backup files.**
- Using `filetype:cfg`/`conf` to locate exposed **configuration files.**
- Searching for **staging environments** (test/development versions of a website) via `inurl:staging` type queries.
- Using `intext:password` to locate pages where the word "password" appears in visible text — which can sometimes surface accidentally exposed credential listings.
- Searching **GitHub** for exposed secrets shared inadvertently by developers.

> **Why this matters for organizations:** This technique can directly surface a company's **login pages** and other **sensitive information** that was never intended to be easily searchable — which is why the instructor emphasizes that many organizations are genuinely shocked to learn this kind of information can be found this way. It is _not_ hacking — it is surfacing information that is technically public but not meant to be easily discoverable.

#### Practical: Sample Query Patterns (Cheat Sheet)

These follow the same pattern demonstrated in the lecture. Replace `target.tld` with an **authorized** target's actual domain (never a domain you don't have permission to test):

| Goal                                                  | Example Query                                                  |
| ----------------------------------------------------- | -------------------------------------------------------------- |
| Find login/admin pages on a specific domain           | `site:target.tld inurl:admin intitle:login`                    |
| Find PDFs indexed under a `.gov` domain               | `site:.gov filetype:pdf`                                       |
| Find possible SQL dump/backup files                   | `site:target.tld filetype:sql`                                 |
| Find exposed configuration files                      | `site:target.tld filetype:cfg OR filetype:conf`                |
| Find open directory listings (any files)              | `intitle:"index of" site:target.tld`                           |
| Find open directory listings for a specific file type | `intitle:"index of" filetype:pdf`                              |
| Find pages mentioning "password" in visible text      | `intext:password site:target.tld`                              |
| Find a staging/test environment                       | `site:target.tld inurl:staging` OR `inurl:test` OR `inurl:dev` |
| Find exposed `.git` metadata                          | `intitle:"index of" ".git"`                                    |
| Find login portals generally (no domain restriction)  | `intitle:login inurl:admin`                                    |

**Combining operators (the core technique):** Stack multiple operators in a single query to narrow results down to exactly what you need — e.g.:

```
site:target.tld inurl:admin intitle:login
```

This reads as: _"search only within target.tld, where the URL contains 'admin', and the page title contains 'login'."_ Each additional operator shrinks the result set, which is the whole point — going from thousands of generic results down to a small, highly relevant list.

> **Practice tip:** Before testing your actual authorized target, practice these operators on your own personal website, a lab environment (e.g., a deliberately vulnerable VM), or publicly known "test yourself" resources like the **Google Hacking Database (GHDB)**, so you understand what kind of results each operator pattern tends to surface.

### 13.2 Useful Browser Extensions

| Tool/Extension                       | Purpose                                                                                                                                                                                              |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Netcraft** (extension/site report) | Generates a report on a website: hosting provider/country, domain registration date and registrar, SSL/TLS details, DNS admin info, top-level domain, and general web technology/tagging information |
| **Wappalyzer**                       | Identifies the technology stack a website is built with                                                                                                                                              |
| **S3 bucket-finding extensions**     | Identify publicly exposed/open Amazon **S3 storage buckets** associated with a site                                                                                                                  |
| **Shodan (browser extension)**       | Shows IP address, hostname, and which ports are open/closed for the current site                                                                                                                     |
| **Cookie editor extensions**         | Allow inspecting/importing/exporting cookies from the current site                                                                                                                                   |
| **Wayback Machine**                  | View historical/archived versions of a website — useful for finding older content, previously exposed information, or information no longer visible on the current live site                         |

**Additional OSINT/recon tools and resources mentioned:**

| Tool                                                                                                                             | Purpose                                                                                                                                                                                    |
| -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Shodan** (web search engine)                                                                                                   | A specialized search engine for internet-connected devices/services; can be searched by target name, vulnerability type, or general keywords to find exposed devices                       |
| **Censys (`.io`)**                                                                                                               | An alternative to Shodan, also useful for email verification, leaked email discovery, etc.                                                                                                 |
| **`.git` exposure checks**                                                                                                       | Checking for exposed `.git` folders (which contain a website's full development history/version tree) — often accidentally left publicly accessible                                        |
| **Security Headers (web tool)**                                                                                                  | Checks whether a website's HTTP security headers are properly configured; also shows raw headers (e.g., server type)                                                                       |
| **Online vulnerability scanners** (e.g., checking for known CVEs like a certain "Poodle"-style SSL vulnerability, or Heartbleed) | Scan a specific target/port for known vulnerabilities in protocols like OpenSSL                                                                                                            |
| **Google Hacking Database (GHDB)**                                                                                               | A curated public database of known Google Dork queries that have previously surfaced things like exposed password files, SSH keys, and configuration files on real (already-indexed) sites |

[⬆ Back to top](#-table-of-contents)

---

## 14. Practical Demonstration: DNS Enumeration

> **DNS enumeration** involves discovering all the DNS records associated with a domain — where they exist, and how many variants/types there are.

**Common DNS record types covered:**

| Record Type                | Purpose                                              |
| -------------------------- | ---------------------------------------------------- |
| **A record**               | Maps a hostname to an **IPv4** address               |
| **AAAA record** ("quad-A") | Maps a hostname to an **IPv6** address               |
| **CNAME record**           | Maps an alias name to a canonical (true) domain name |
| **MX record**              | Directs mail to a specific mail server               |

**Tools demonstrated:**

| Tool                 | Purpose                                                                                                                                                                                                                                                                                                        |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`host`** (command) | Basic DNS lookup for a domain; can be filtered by record type (e.g., `host -t mx example.com` to find mail servers specifically)                                                                                                                                                                               |
| **`nslookup`**       | Cross-platform DNS lookup tool (available on Windows); shows resolved IP, DNS server used, and authoritative answer status                                                                                                                                                                                     |
| **`dig`**            | A more detailed DNS lookup tool, showing headers and additional record information; supports many operators for more advanced queries                                                                                                                                                                          |
| **`dnsrecon`**       | A Python-based DNS reconnaissance tool for a given target; can reveal hosting provider and other DNS-related details                                                                                                                                                                                           |
| **`dnsenum`**        | A multi-threaded tool to enumerate DNS information for a domain and discover non-contiguous IP blocks; can find live hosts, host addresses, name servers, MX records, attempt AXFR zone transfers, and discover extra subdomains via search-engine scraping (can be enhanced with API keys for better results) |
| **`traceroute`**     | Shows the path (hop by hop) that a request takes to reach a target, including intermediate routers/ISPs and timing                                                                                                                                                                                             |

**Related concept — DNS spoofing/cache poisoning:**

- Broadly refers to techniques involving sending DNS-related requests using a fake/spoofed IP address or server, potentially to protect the requester's own identity during recon or to manipulate DNS resolution.
- **DNS cache checking:** Some techniques (e.g., varying lookup types/non-recursive queries) can reveal whether a DNS record is already cached somewhere, which can sometimes be leveraged toward account/subdomain/domain takeover techniques if a stale or misconfigured record is found (both "inclusive" and "non-inclusive"/opposite lookup approaches exist for this).

#### Practical: Example Commands

Replace `target.tld` with an authorized target's domain:

```bash
# Basic lookup — returns A, AAAA, and MX records
host target.tld

# Look up only MX (mail server) records
host -t mx target.tld

# Look up only NS (name server) records
host -t ns target.tld

# Cross-platform lookup (works on Windows too)
nslookup target.tld

# Detailed lookup with dig, including full header info
dig target.tld

# Query a specific record type with dig
dig target.tld MX
dig target.tld TXT
dig target.tld NS

# Attempt a zone transfer (only works if misconfigured — flags a real finding if it succeeds)
dig axfr target.tld @ns1.target.tld

# Automated DNS recon — pulls hosting provider, records, and more
dnsrecon -d target.tld

# Multi-threaded enumeration: live hosts, name servers, MX records,
# attempts zone transfer, and can scrape search engines for subdomains
dnsenum target.tld

# Trace the network path to a target, hop by hop
traceroute target.tld      # Linux/macOS
tracert target.tld         # Windows
```

**Reading the output — what to look for:**

- Multiple **A records** for the same hostname can indicate load balancing across several servers.
- An unexpected **CNAME** pointing to a third-party service (e.g., a cloud storage or SaaS provider) that is no longer active can be a sign of a potential **subdomain takeover** vulnerability.
- A successful **zone transfer (AXFR)** is a significant finding — it means the entire DNS record set for a domain can be dumped in one request, which should never be possible from an untrusted source.

[⬆ Back to top](#-table-of-contents)

---

## 15. Practical Demonstration: Port Scanning (TCP Recon)

**Network discovery tools (for finding live hosts on a network):**

| Tool                    | Purpose                                                                                                                                     |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Angry IP Scanner**    | Scans a given IP range and reports host status: red = dead/inactive, blue = previously reachable but now inactive, green = currently active |
| **Advanced IP Scanner** | Similar to Angry IP Scanner, but with richer details: MAC address, device manufacturer, IP address, device name, and status                 |
| **Masscan**             | A very fast port scanner; supports specifying IP ranges and specific ports to scan                                                          |
| **RustScan**            | Another fast port scanner (built in the Rust language), supporting both TCP and UDP port scanning                                           |

**Netcat (`nc`) for basic TCP connection testing:**

- Can be used to manually test connectivity to a specific IP and port range (e.g., checking whether ports in the 3388–3390 range are open/responsive), with configurable connection timeout and verbosity settings.

**Nmap fundamentals:**

| Flag/Concept                                      | Purpose                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`nmap <IP>`**                                   | Basic scan showing open ports                                                                                                                                                                                                                                                                                                                        |
| **`-sT`**                                         | TCP connect scan                                                                                                                                                                                                                                                                                                                                     |
| **`-sU`**                                         | UDP scan                                                                                                                                                                                                                                                                                                                                             |
| **`-p-`**                                         | Scan **all** 65,535 ports                                                                                                                                                                                                                                                                                                                            |
| **`-O`**                                          | Attempt OS detection                                                                                                                                                                                                                                                                                                                                 |
| **`iptables`** (companion tool, not part of nmap) | Used to measure/observe how much traffic (bytes/packets) a particular scan actually generates — useful for understanding a scan's "footprint," since **generating minimal, necessary traffic/logs is an important professional consideration during an authorized audit**, to avoid unnecessarily alerting defenders or triggering excessive logging |

> **Professional consideration highlighted:** A single full nmap scan can generate on the order of 1,000+ requests to the target. Understanding and being able to estimate this traffic volume matters during real engagements, where minimizing unnecessary noise/log generation is good practice.

#### Practical: Example Commands

Replace `<TARGET_IP>` with an IP address you are **authorized** to test (e.g., your own lab VM's address):

```bash
# Basic scan of the top common ports
nmap <TARGET_IP>

# TCP connect scan
nmap -sT <TARGET_IP>

# UDP scan (slower — UDP has no handshake, so results can be less certain)
nmap -sU <TARGET_IP>

# Scan a specific port only
nmap -p 80 <TARGET_IP>

# Scan a specific range of ports
nmap -p 1-1000 <TARGET_IP>

# Scan ALL 65,535 ports (thorough, but slow and "noisy")
nmap -p- <TARGET_IP>

# Attempt OS detection
nmap -O <TARGET_IP>

# Service/version detection on open ports
nmap -sV <TARGET_IP>

# Common "quick recon" combo: version detection + default scripts + OS guess
nmap -sV -sC -O <TARGET_IP>

# Fast scan of the ~100 most common ports only
nmap -F <TARGET_IP>

# Verbose output (see progress as it happens)
nmap -v <TARGET_IP>
```

**Other host/port discovery tools:**

```bash
# Angry IP Scanner / Advanced IP Scanner — GUI tools, just enter an IP range
# e.g., 192.168.1.1-192.168.1.254

# Masscan — extremely fast scanner across large ranges
masscan -p80 192.168.1.0/24 --rate=1000

# RustScan — fast scanner, often paired with nmap for detailed follow-up
rustscan -a <TARGET_IP> -- -sV -sC
```

**Measuring your own scan's traffic footprint (professional practice):**

```bash
# Set up a basic iptables counter rule before scanning
sudo iptables -N SCANCOUNT
sudo iptables -A SCANCOUNT -d <TARGET_IP> -j ACCEPT
sudo iptables -I OUTPUT -d <TARGET_IP> -j SCANCOUNT

# Run your scan, then check how many packets/bytes were sent
sudo iptables -L SCANCOUNT -v -n
```

**Reading the output — what matters:**

- **Open** ports are actively accepting connections.
- **Closed** ports respond but have nothing listening.
- **Filtered** ports gave no response at all — usually means a firewall is silently dropping packets, which itself is useful information (it tells you a security control is present).
- Always cross-reference an open port against the **service/version** nmap reports (`-sV`) — an old, unpatched version is a strong signal worth investigating further (see Section 12.3).

[⬆ Back to top](#-table-of-contents)

---

## 16. Practical Demonstration: SMB Enumeration

> **SMB (Server Message Block) enumeration** involves gathering information about SMB-related services running on a target — such as file sharing, file transfer, and printer sharing.

**Why this matters:**

- The instructor notes personally observing real-world cases (e.g., in banking environments) where **networked printers** were left open/exposed. If confidential documents are sent to such a printer and the underlying SMB service has a flaw, that traffic/data could potentially be intercepted ("sniffed") by anyone with network access.
- SMB enumeration typically aims to discover: **shared folders, default/existing usernames, the operating system in use,** and the **SMB protocol version.**

**Tools demonstrated:**

| Tool                                                                         | Purpose                                                                                                                                                                                    |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Nmap SMB scripts** (`--script smb-enum-shares`, `--script smb-enum-users`) | Enumerate shared folders and user accounts on ports 139 and 445                                                                                                                            |
| **Nmap `--script smb-os-discovery`**                                         | Attempt to determine the target's operating system via SMB                                                                                                                                 |
| **`smbclient`**                                                              | An FTP-like client for accessing SMB/CIFS shares directly; if a share allows anonymous/no-password access (`-N`), this can indicate a configuration flaw                                   |
| **`enum4linux`**                                                             | A Linux-based tool for enumerating SMB details (workgroup/domain info, NetBIOS status, default account names like Administrator/Guest/krbtgt/root, etc.) directly by supplying a target IP |
| **CrackMapExec** (referenced as "crackmap")                                  | A tool for further SMB-related enumeration and testing                                                                                                                                     |
| **`smbmap`**                                                                 | Checks what read/write permissions are available on discovered SMB shares for a given target                                                                                               |

**Key takeaway:** If an **SMB null session** (unauthenticated access) is found to be possible, a tester can potentially enumerate user lists, view shared files, and gather general system information — this is a common early step once nmap has revealed an open SMB port on a target.

#### Practical: Example Commands

Replace `<TARGET_IP>` with an authorized lab/target IP. Ports **139** and **445** are the two SMB-relevant ports:

```bash
# Confirm SMB ports are open first
nmap -p 139,445 <TARGET_IP>

# Enumerate shared folders
nmap --script smb-enum-shares -p 139,445 <TARGET_IP>

# Enumerate users
nmap --script smb-enum-users -p 139,445 <TARGET_IP>

# Combine both scripts in one scan
nmap --script smb-enum-shares,smb-enum-users -p 139,445 <TARGET_IP>

# Attempt OS discovery via SMB
nmap --script smb-os-discovery -p 139,445 <TARGET_IP>

# See ALL available smb-* nmap scripts
ls /usr/share/nmap/scripts/ | grep smb

# Connect to a share with smbclient, no password (tests for a null session)
smbclient -L //<TARGET_IP>/ -N

# Connect to a specific discovered share
smbclient //<TARGET_IP>/SHARE_NAME -N

# Linux-based SMB enumeration (workgroup, domain, default accounts, NetBIOS info)
enum4linux <TARGET_IP>
enum4linux -a <TARGET_IP>      # "-a" = run all enumeration checks

# Check read/write permissions on any discovered shares
smbmap -H <TARGET_IP>

# Anonymous/guest login check with smbmap
smbmap -H <TARGET_IP> -u anonymous -p ""

# CrackMapExec — broader SMB enumeration/testing (also supports credential checks)
crackmapexec smb <TARGET_IP>
```

**Reading the output — what matters:**

- A **null session** (connecting with `-N` / no credentials and it works) is itself a notable finding — it means basic enumeration is possible without any authentication at all.
- Shared folders with **write** access open to anonymous/guest users are a high-priority finding, since they could allow an attacker to plant malicious files.
- Default account names appearing (e.g., `Administrator`, `Guest`) confirm the OS/domain structure and can inform later password-guessing attempts (in later phases of an authorized engagement, not during recon itself).

[⬆ Back to top](#-table-of-contents)

---

## 17. Practical Demonstration: SMTP Enumeration

> **SMTP (Simple Mail Transfer Protocol) enumeration** focuses on discovering valid email addresses/users hosted on an organization's mail servers.

**Why it matters:**

- Prevents/detects scenarios where malicious actors could be extracting internal information via mail systems and leaking it externally.
- Helps assess how vulnerable an organization might be to **phishing attacks**, and tests general password strength (e.g., via **password spraying** — checking whether any employees are using weak, easily guessed passwords) or brute-force attempts against the mail server.
- These techniques fall under **active reconnaissance**, since the target's own server will register that this interaction occurred.

**Concepts covered:**

- Distinguishing **valid vs. invalid users** on a mail server.
- Checking whether a given account/username actually **exists**, versus encountering **fake/nonexistent users.**

**Tools/techniques demonstrated:**

| Tool/Port                                   | Purpose                                                                                                                |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Nmap SMTP enumeration scripts** (port 25) | Enumerate mail server details, including whether a domain uses a specific mail provider (e.g., visible via MX records) |
| **Netcat on port 25**                       | Manually test/connect to an SMTP service directly                                                                      |
| **Common SMTP-related ports**               | 25, 465, and 587                                                                                                       |

**What can be learned from SMTP enumeration:**

- Whether a specific username is **valid.**
- **Naming patterns** used for email addresses (e.g., first-name/last-initial conventions).
- Department-based email address structures.
- Presence of **admin accounts, service accounts,** and accounts with elevated/manager-level access.

#### Practical: Example Commands

Replace `<TARGET_IP>` with an authorized lab/target IP. Common SMTP ports: **25** (unencrypted), **465** (SMTPS), **587** (submission):

```bash
# Confirm the mail port is open and grab the service banner
nmap -p 25 <TARGET_IP>
nmap -sV -p 25,465,587 <TARGET_IP>

# Run nmap's SMTP-related enumeration/vulnerability scripts
nmap --script smtp-commands -p 25 <TARGET_IP>
nmap --script smtp-enum-users -p 25 <TARGET_IP>
nmap --script smtp-open-relay -p 25 <TARGET_IP>

# Manually connect to the mail server banner with netcat
nc -nv <TARGET_IP> 25

# Once connected, an SMTP server understands commands like:
#   HELO test.local
#   MAIL FROM:<test@test.local>
#   RCPT TO:<someuser@target.tld>
# The server's response to RCPT TO can reveal whether a given
# mailbox/username exists (this is the "user enumeration" technique).
```

**Reading the output — what matters:**

- A verbose banner (showing exact mail server software/version) helps identify known vulnerabilities for that specific version.
- If the server confirms/denies `RCPT TO` addresses differently for real vs. fake usernames, that's a **user enumeration** weakness.
- An **open relay** (a server that will forward mail for any sender/recipient without authentication) is a serious, high-priority finding — it can be abused for spam/phishing at scale.

[⬆ Back to top](#-table-of-contents)

---

## 18. Practical Demonstration: SNMP Enumeration

> **SNMP (Simple Network Management Protocol) enumeration** focuses on extracting information about **routers, switches, printers,** and other networked devices.

**Why this matters:** Understanding system details is a prerequisite to any further investigation — without basic system information, a tester cannot proceed to discover more sensitive/credential-related information or move further "into" a system.

**Information typically sought via SNMP enumeration:**

- **Running services**, **network configuration**, **usernames**, and each device's specific **role** (since departments within a company often run different systems/networks).
- Whether the SNMP service is running at all (commonly on **UDP port 161**).
- Whether **community strings** (SNMP's basic access credentials) are set to something insecure like default **"public"**/"private" values.
- Broader system details: hostname, OS details, network interfaces, IP addresses, running processes, installed software, user accounts, and routing/ARP table information.

**Tools demonstrated:**

| Tool                                                             | Purpose                                                                                                              |
| ---------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **`snmpwalk`** (with `-v2c`, community string `public`)          | Bulk-retrieves a large amount of information from a target's SNMP service in one operation                           |
| **`snmpcheck`**                                                  | A similar tool for pulling detailed device information (filesystem, memory/disk size and description, etc.) via SNMP |
| **Nmap UDP scan (`-sU`) with `-p 161` and `--script snmp-info`** | Detects and extracts SNMP-related information via nmap's scripting engine                                            |

#### Practical: Example Commands

Replace `<TARGET_IP>` with an authorized lab/target IP. SNMP runs on **UDP port 161** by default:

```bash
# Confirm SNMP is open (UDP scan)
nmap -sU -p 161 <TARGET_IP>

# Run nmap's SNMP info-gathering script
nmap -sU -p 161 --script snmp-info <TARGET_IP>

# Bulk-retrieve information with snmpwalk (v2c, default "public" community string)
snmpwalk -v2c -c public <TARGET_IP>

# Query a specific object identifier (OID) branch only, e.g. system info
snmpwalk -v2c -c public <TARGET_IP> system

# snmp-check — a friendlier, more readable summary of the same data
snmp-check <TARGET_IP>

# Test whether the "private" community string is also accepted (read-write access
# would be a much more serious finding than read-only "public" access)
snmpwalk -v2c -c private <TARGET_IP>
```

**Reading the output — what matters:**

- If `public`/`private` community strings work at all, that's already a finding — SNMP should ideally require a non-default, authenticated (SNMPv3) configuration.
- **Read-write** access (via a working "private" string) is far more serious than read-only, since it could allow reconfiguring the device, not just reading its state.
- Data returned can include hostname, OS, uptime, network interfaces, running processes, installed software, and routing tables — all useful context for the next phase of testing.

[⬆ Back to top](#-table-of-contents)

---

## 19. Summary of Key Takeaways

1. **VAPT (Vulnerability Assessment and Penetration Testing)** is a structured, authorized security process — identify, validate, and prioritize risks — not random, unauthorized hacking.
2. Cybersecurity differs fundamentally from other technical fields because security problems are **intentionally created by adversaries** who adapt and retry, and systems can **appear fine on the surface while being compromised underneath** — traditional "investigate and fix" troubleshooting doesn't fully apply.
3. **Bugs** are unintentional errors; **vulnerabilities** are weaknesses that can be **deliberately exploited** — not every bug is a vulnerability, but every vulnerability carries real risk.
4. Testing falls into three categories: **Black box** (no prior knowledge, simulating an outsider), **White box** (full internal knowledge/source code access, focused on deep analysis), and **Gray box** (limited access, simulating a semi-trusted insider).
5. Core terminology to know: **asset, vulnerability, threat, exploit, risk, attack surface, attack vector, PoC, impact,** and **false positive.**
6. Five major types of penetration testing: **network** (external and internal), **web application, mobile application, social engineering,** and **physical.**
7. A good VAPT **report** must be clear, well-structured, and tailored to three audiences: **executives** (risk/business impact), **management** (priorities/remediation), and **technical staff** (exact fixes needed).
8. **Reconnaissance** is the foundational phase of any engagement, split into **passive** (no direct interaction with the target — domains, tech stack, employee info, public documents, leaked data) and **active** (direct interaction — live hosts, open ports, service versions, OS details).
9. Information gathered spans three levels: **organization-level** (branding, locations, email formats, business context), **network-level** (IP ranges, DNS records, cloud/VPN/firewall usage, scope boundaries), and **system-level** (OS, web server, frameworks, software versions, configurations).
10. Practical reconnaissance techniques covered in detail: **Google Dorking** (`site:`, `inurl:`, `intitle:`, `filetype:`, `intext:`), browser extensions (**Netcraft, Wappalyzer, Shodan, Wayback Machine**), **DNS enumeration** (`host`, `nslookup`, `dig`, `dnsrecon`, `dnsenum`, `traceroute`), **port scanning** (Angry IP Scanner, Advanced IP Scanner, Masscan, RustScan, Nmap), **SMB enumeration** (`smbclient`, `enum4linux`, `smbmap`, Nmap SMB scripts), **SMTP enumeration** (Nmap SMTP scripts, Netcat on port 25), and **SNMP enumeration** (`snmpwalk`, `snmpcheck`, Nmap SNMP scripts).
11. Throughout, the instructor repeatedly stresses that all of this must be performed **legally and with proper authorization** — this is professional security testing methodology, not a guide for unauthorized access to systems you don't have permission to test.

[⬆ Back to top](#-table-of-contents)

---

## 20. Quick-Reference Glossary

| Term                       | Definition                                                                                                                   |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **VAPT**                   | Vulnerability Assessment and Penetration Testing — a structured process to identify, validate, and prioritize security risks |
| **Black box testing**      | Testing with no prior knowledge of internal architecture, simulating an outside attacker                                     |
| **White box testing**      | Testing with full knowledge of architecture and source code, for deep security analysis                                      |
| **Gray box testing**       | Testing with limited, partial access, simulating a semi-trusted insider                                                      |
| **Asset**                  | Anything of value that requires protection                                                                                   |
| **Vulnerability**          | A weakness that can be intentionally exploited                                                                               |
| **Threat**                 | A potential danger capable of exploiting a vulnerability                                                                     |
| **Exploit**                | A method/technique used to take advantage of a vulnerability                                                                 |
| **Risk**                   | Potential damage/loss resulting from successful exploitation                                                                 |
| **Attack surface**         | The total set of points where an attacker could potentially attack a system                                                  |
| **Attack vector**          | The specific path/technique used to carry out an attack                                                                      |
| **PoC (Proof of Concept)** | A demonstration of how a vulnerability was exploited                                                                         |
| **False positive**         | A reported vulnerability that does not actually exist                                                                        |
| **Reconnaissance (recon)** | The information-gathering phase of a security engagement                                                                     |
| **Passive reconnaissance** | Gathering information without directly interacting with the target                                                           |
| **Active reconnaissance**  | Gathering information by directly interacting with the target system                                                         |
| **Google Dorking**         | Using advanced Google search operators to surface hidden but technically public information                                  |
| **DNS enumeration**        | Discovering DNS records (A, AAAA, CNAME, MX, etc.) associated with a domain                                                  |
| **Port scanning**          | Identifying which ports are open on a target and what services run on them                                                   |
| **SMB enumeration**        | Gathering information about file/printer sharing services on a target                                                        |
| **SMTP enumeration**       | Discovering valid email addresses/users on a mail server                                                                     |
| **SNMP enumeration**       | Extracting device/network information via the Simple Network Management Protocol                                             |
| **Nmap**                   | A widely used network scanning tool for port discovery, service detection, and OS fingerprinting                             |
| **Shodan**                 | A search engine for internet-connected devices and exposed services                                                          |

[⬆ Back to top](#-table-of-contents)

---

_End of Part 1 — Part 2 of the series continues with exploitation and post-exploitation techniques; Part 3 covers reporting, risk prioritization, CTF practice, and career guidance._
