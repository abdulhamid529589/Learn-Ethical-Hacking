# Ethical Hacking / Cybersecurity Notes — Module: Footprinting & Reconnaissance

> Instructor: **Ashish Kumar** — Cyber Pathshala
> Topic: **Footprinting and Reconnaissance** (a.k.a. **Information Gathering**) — the **first phase** of ethical hacking / cybersecurity methodology
> Tools covered: Search engines, Google Dorks (Advanced Search Operators), OSINT Framework, Shodan, theHarvester

---

## Table of Contents

1. [Why Information Gathering Matters](#1-why-information-gathering-matters)
2. [What Is Information Gathering / Footprinting / Reconnaissance?](#2-what-is-information-gathering--footprinting--reconnaissance)
3. [Targets of Information Gathering](#3-targets-of-information-gathering)
4. [Types of Information Gathering: Active vs. Passive](#4-types-of-information-gathering-active-vs-passive)
5. [Source 1: Search Engines — The "Never Trust Page One" Rule](#5-source-1-search-engines--the-never-trust-page-one-rule)
6. [Source 2: Google Dorks (Advanced Search Operators)](#6-source-2-google-dorks-advanced-search-operators)
7. [Source 3: The OSINT Framework](#7-source-3-the-osint-framework)
8. [Source 4: Shodan](#8-source-4-shodan)
9. [Source 5: theHarvester (Tool)](#9-source-5-theharvester-tool)
10. [Quick-Reference Summary](#10-quick-reference-summary)

---

## 1. Why Information Gathering Matters

**Core principle:** Footprinting & Reconnaissance is the **very first phase** of ethical hacking. Without understanding your target, you cannot effectively find vulnerabilities or exploit a system.

### Illustrative Case Study — Hacker A vs. Hacker B

|                               | **Hacker A**                                                                                            | **Hacker B**                                                                                                                                                                                           |
| ----------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Experience                    | 10–15 years, highly skilled                                                                             | Similarly experienced                                                                                                                                                                                  |
| Approach                      | Jumped straight to attacking — applied every known technique/exploit directly against the victim device | **First spent time gathering information** about the target before attacking                                                                                                                           |
| Time spent                    | **1 full week** of continuous attack attempts                                                           | **~Half a day** gathering info, then testing                                                                                                                                                           |
| Result                        | **No success** — found no vulnerabilities despite a week of effort                                      | **Success on the first targeted attempt**                                                                                                                                                              |
| What Hacker B did differently | —                                                                                                       | Identified via information gathering that the victim ran **Linux version 2.2.6** → researched known vulnerabilities for that specific version → picked one, tested it → gained full access in one step |

> **Lesson:** Effectiveness and efficiency in real attacks/pentests come from **reconnaissance first**, not from brute-force skill alone. Knowing the exact OS/version turns a blind, exhausting search into a single, targeted, successful strike.

### Key Reasons Footprinting & Recon Are Essential

| Reason                                                          | Explanation                                                                                                                                                                                                                                  |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **You must know your target**                                   | Without knowing the OS, technology stack, or daily routine of a target, you have no realistic starting point to find loopholes/vulnerabilities — worse than even black-box testing (where you at least use tools to gather data with effort) |
| **Staying stealthy**                                            | Reconnaissance (especially the passive kind) lets you gather data **without direct interaction**, keeping you comparatively hidden/anonymous                                                                                                 |
| **It's the entire foundation of cybersecurity/ethical hacking** | Without this phase, you have no data, no direction, and no informed basis for any subsequent testing — everything else depends on it                                                                                                         |

---

## 2. What Is Information Gathering / Footprinting / Reconnaissance?

> **Definition:** _"Collecting intelligence about the target's system, network, and organization."_

- **Footprinting** and **Reconnaissance** are essentially the **elements/components** that make up the broader activity of **Information Gathering**.
- The core question this phase answers: _who/what is my target, and what can I learn about them before I engage further?_

---

## 3. Targets of Information Gathering

Information gathering is performed against **three main categories** of targets:

### 3.1 Technology

| Data Collected                                  |
| ----------------------------------------------- |
| Server IP address                               |
| MAC address                                     |
| Software running on the server                  |
| Open ports                                      |
| Operating system                                |
| Programming languages used (and their versions) |
| Libraries used (and their versions)             |
| Plugins in use                                  |
| Themes in use                                   |

### 3.2 Company / Organization

| Data Collected                                                            |
| ------------------------------------------------------------------------- |
| Company name                                                              |
| Owner's name                                                              |
| Employee details (email addresses, mobile numbers, experience, expertise) |
| Financial records                                                         |
| Future goals/plans of the company                                         |

### 3.3 Individual / Person

| Data Collected                                       |
| ---------------------------------------------------- |
| Name, email address                                  |
| Personal details, contact number                     |
| Government documents                                 |
| Daily routine                                        |
| Qualifications                                       |
| Family background                                    |
| Market reputation/goodwill, general nature/character |

---

## 4. Types of Information Gathering: Active vs. Passive

| Type                              | Definition                                                                                                                                       | Examples                                                                                                                                                                                                                                                                             |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Active Information Gathering**  | Involves **direct interaction/engagement** with the target — even if done anonymously via a fake identity, it still counts as direct interaction | Creating a fake social media profile and sending a friend request; chatting with the target (even under a fake identity); making a fake call or sending an SMS; sending network packets to the target's device that leave a traceable record (e.g., an IP address appearing in logs) |
| **Passive Information Gathering** | Collecting data **indirectly, without any interaction** with the target                                                                          | Creating a fake social account but **not** sending a friend request or reacting to any posts — simply scrolling through and observing a public profile without engaging                                                                                                              |

> **Key distinction:** the presence of _any_ direct engagement (even hidden/anonymous) = active. Pure observation with zero interaction = passive.

---

## 5. Source 1: Search Engines — The "Never Trust Page One" Rule

### The Core Rule

> **Never rely on the first page of search results when gathering high-quality information. Always go beyond page 1 — page 2, page 3, and onward.**

**Why:** Search engines don't show you what _you_ want first — they show you what **advertisers and SEO-optimized companies** want, since businesses use **SEO (Search Engine Optimization)** and digital marketing to rank highly for popular keywords. Example given: searching "ethical hacking" surfaces **paid courses/companies** first, even though free educational content also exists — that free content just isn't SEO-optimized to rank on page 1.

**This rule applies universally, regardless of the search engine used.**

### Practical Technique for Gathering Data on a Person

1. Add **extra context/keywords** beyond just the name — e.g., add "cybersecurity," "ethical hacking," or another relevant field/interest — to narrow and improve result relevance.
2. Expect page 1 to mostly show **what the target wants shown** (e.g., their curated LinkedIn profile, matching photos/bio they control).
3. **Go to page 2, page 3, and beyond** — this is typically where **genuinely valuable/uncontrolled data** starts to surface.
4. **Systematically open every URL, one at a time** — don't skip any — starting from page 2 onward, until you've extracted the most valuable information available.

---

## 6. Source 2: Google Dorks (Advanced Search Operators)

### Concept

Google Dorks = using a search engine's **built-in filters** to tell it _exactly_ what you want, so irrelevant results are excluded — often surfacing exactly what you need right on page 1.

**Two ways to apply these filters:**

1. **Automation method** — using Google's **Advanced Search** form (a GUI that builds the filtered query for you).
2. **Manual method** — typing the filter operators directly into the search bar yourself.

### 6.1 The Advanced Search Form — Field by Field

Accessed by searching **"Advanced Search"** → selecting the Google Advanced Search page.

| Field                         | Purpose                                                                                                                                                                                             | Example Used in Walkthrough                                                                                                                                          |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **All these words**           | General keyword pool — results should relate to these terms                                                                                                                                         | `Ashish Kumar hacking Jodhpur`                                                                                                                                       |
| **This exact word or phrase** | A **must-include** term/phrase — guarantees this exact word appears in every result                                                                                                                 | `Ashish` (wrapped in double quotes in the resulting query)                                                                                                           |
| **Any of these words**        | A looser filter — used when you have multiple candidate terms and aren't sure which will yield the best results; results can match **any one** of them (joined internally with an OR-like operator) | `Admin`, `Instagram`, `Facebook`                                                                                                                                     |
| **None of these words**       | Terms to **exclude** from results                                                                                                                                                                   | `LinkedIn` (to exclude LinkedIn results initially, since that data is already known/expected)                                                                        |
| **Numbers ranging from**      | Used only if a numeric range matters to your search (dates, prices, measurements)                                                                                                                   | _Not used_ — irrelevant for a general person-search                                                                                                                  |
| **Language**                  | Restrict to a specific language, if relevant (shows content **originally written** in that language, not translated)                                                                                | Left as **"Any Language"** — don't assume the target only posts in one language (e.g., a hacker might post data on a Russian-language site)                          |
| **Region**                    | Restrict to a specific geographic area                                                                                                                                                              | Left unrestricted — no way to know in advance which region might hold the best data                                                                                  |
| **Last update**               | Restrict by recency (last 24 hrs / week / month / year)                                                                                                                                             | Left unrestricted — no prior indication of when relevant data was posted                                                                                             |
| **Site or domain**            | Restrict results to a **specific website/domain only**                                                                                                                                              | Demonstrated with `facebook.com` — but explicitly **not used** in the final search, since restricting to one site defeats the purpose of broad information gathering |
| **Terms appearing**           | Where the search terms must appear: in the **page title**, in the **visible text**, in the **URL**, or in **links** (hyperlinks) on the page                                                        | Left as default (anywhere) — to maximize coverage                                                                                                                    |
| **File type**                 | Restrict results to a specific file format (e.g., PDF)                                                                                                                                              | Demonstrated but **not used** — over-restricting (e.g., combining "Facebook" + "PDF only") can eliminate all useful results                                          |
| **Usage rights**              | Restrict to content that's free to reuse/license                                                                                                                                                    | Not relevant/used for reconnaissance purposes                                                                                                                        |

> ⚠️ **General principle demonstrated throughout:** Only apply a filter when you have a genuine, informed reason to restrict results. **Over-filtering** (e.g., combining a specific site + a specific file type unnecessarily) can eliminate all meaningful results entirely.

### 6.2 Manual (Direct Query) Method

The Advanced Search form is really just a **UI ​that generates a query string** — the same effect can be typed manually. From the walkthrough, the generated query illustrated these manual operators:

| Query Syntax Element            | Meaning                                                                |
| ------------------------------- | ---------------------------------------------------------------------- |
| `Ashish Kumar hacking Jodhpur`  | Plain keywords (from "All these words")                                |
| `Admin Instagram Facebook`      | Additional loosely-matched candidate terms (from "Any of these words") |
| `"Ashish"` (double-quoted)      | **Exact phrase/must-include term** (from "This exact word or phrase")  |
| `-LinkedIn` (minus sign prefix) | **Exclude** this term from all results (from "None of these words")    |
| `site:facebook.com`             | Restrict results to **only** this specific site/domain                 |

**Demonstrated result:** With `-LinkedIn` active, results surfaced other exposed data (e.g., email/phone number references) without LinkedIn dominating. Removing the `-LinkedIn` exclusion brought LinkedIn's profile/post data back into the mix — showing how toggling a single operator changes the result set significantly.

---

## 7. Source 3: The OSINT Framework

**Definition:** _"A framework designed to improve intelligence gathering — organizing how to collect data and explaining its value."_

**Structure:** The framework is organized as a set of **branches/categories** based on what type of data you want to look up — e.g.:

- Username
- Email
- Domain
- (and many others)

Selecting a category (e.g., **Domain**) then reveals **sub-options** for the specific kind of data you want, such as:

- **WHOIS record**
- **Subdomain discovery**
- and more

Each sub-option links out to a **curated list of external websites/tools** that can provide that specific type of data.

### Hands-On Walkthrough: Domain Lookup via OSINT Framework

1. Selected **Domain → WHOIS record** category → picked one of the linked tools/websites.
2. Entered a target domain (example used: a placeholder like `test.pap.1vw.com`).
3. Selected multiple options to check simultaneously:
   - **WHOIS record**
   - **DNS record**
   - **Traceroute** (shows how many hops/machines a packet passes through to reach the server, and the IP of each machine along the path)
   - **Network WHOIS record**
   - **Service scan** (reveals open ports and the software/version running on them)
4. Clicked **Go** to run the combined lookup.

**Results obtained:**
| Data Point | What It Revealed |
|---|---|
| **WHOIS record** | Who purchased the domain, under what name, and a phone number |
| **Network WHOIS record** | The **server/hosting provider name** — in this case identified as an **AWS (Amazon)** server |
| **DNS records** | Multiple record types returned: |
| — **A record** | Maps domain to IP address |
| — **MX record** | Mail server info |
| — **NS record** | Nameserver info |
| — **TXT record** | Verifies email and **helps prevent email spoofing** |
| — **SOA record** | Contains an administrative **email address** for the domain, used for contact in case of issues |
| — **PTR record** (Pointer record) | Can be used to point a lookup toward a _different_ identity/location — a technique explored further in a later **Advanced Network Scanning** module |
| **Traceroute** | Identified the source IP the packet was sent from, and traced the path (hop-by-hop machine IPs) to the main server, including the main server's own IP |
| **Service/Port scan** | Found an **open port 80** running **Nginx version 1.19.0** |

> **Instructor's guidance:** This is only a small sample of what the OSINT Framework can surface. It contains a large number of tools and services across many categories — dedicate real time to exploring each branch thoroughly.

---

## 8. Source 4: Shodan

**What it is:** A search engine/platform for discovering internet-connected devices, services, and exposed information (introduced briefly here; deeper usage shown via integration with theHarvester in §9).

### Setting Up an Account (Privacy-Conscious Method)

1. Rather than registering with real personal details, the instructor uses a **disposable/temporary email service** website (name intentionally not disclosed in the transcript) to receive a throwaway inbox.
2. Copied the temporary email address.
3. On Shodan: **Create Account** → filled in a chosen **username**, **password** (+ confirmation), and the **temporary email address**.
4. Skipped subscribing to the newsletter → clicked **Create**.
5. **Verification:** returned to the temporary email inbox → opened the verification email → activated the account.
6. **Logged in** using the chosen username and password.

### Retrieving the API Key

- Went to **My Account** → copied the **API key** shown there (needed for tool integrations, such as theHarvester below).

---

## 9. Source 5: theHarvester (Tool)

**Platform:** Kali Linux.

**Purpose:** An advanced information-gathering/footprinting tool that can query **multiple OSINT sources simultaneously** (including Shodan, once configured with an API key) rather than checking each source manually one at a time.

### 9.1 Launching the Tool

```bash
sudo su          # obtain root permissions
theHarvester     # may show a rename notice
```

- ⚠️ **Naming note from the transcript:** the tool's invocation name has changed — from `theHarvester` to a capitalized **`theHarvester`** variant (the tool itself indicated this rename when the old command was tried) — use whichever name the tool's own prompt/help indicates on your system.

### 9.2 Exploring Available Sources

```bash
theHarvester --help
```

- The `-b` (source/"Sources") option supports **many possible OSINT sources** to query from (Shodan being just one of them).

### 9.3 Adding the Shodan API Key to theHarvester's Config

The Shodan API key must be added to theHarvester's **configuration file**, typically located under `/etc/`:

```bash
cd /etc
ls                          # locate the folder named "theHarvester"
cd theHarvester
ls                          # locate the config file, e.g. "api-keys.yaml"
gedit api-keys.yaml         # or: mousepad api-keys.yaml   (if gedit isn't installed)
```

**Editing steps:**

1. Locate the **`shodan:`** entry in the config file.
2. ⚠️ **Important formatting detail:** leave a **space** after the colon before pasting the API key (i.e., `shodan: <your_api_key>`).
3. Paste the copied Shodan API key.
4. Save and close the file.

### 9.4 Running a Scan

```bash
theHarvester -d test.php.1vw.com -b all -l 100
```

| Flag | Meaning                                                                                                                                                                                                                                                    |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-d` | Target **domain** to investigate (example: `test.php.1vw.com`)                                                                                                                                                                                             |
| `-b` | **Source(s)** to query — `all` tells it to check **every available source**, including Shodan (assuming its API key is configured)                                                                                                                         |
| `-l` | **Limit** on the number of results returned. Default is **500**; the instructor deliberately lowers this to **100** to **avoid getting the (Shodan) API key rate-limited/blocked** — a smaller limit still yields usable results while reducing API strain |
| `-f` | (Mentioned, not demonstrated in depth) Used to **save/export** the results to a file                                                                                                                                                                       |

**Recommendation for better results:** Register for and configure **API keys for every other supported source** as well (following the same pattern shown for Shodan), not just Shodan alone, to maximize the completeness of results returned by theHarvester.

---

## 10. Quick-Reference Summary

| Concept                 | Key Takeaway                                                                                                                                                                                                                             |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Why recon matters**   | Targeted, information-driven attacks succeed far faster and more reliably than blind, skill-only attacks (Hacker A vs. B case study)                                                                                                     |
| **Definition**          | Collecting intelligence about a target's system, network, and organization                                                                                                                                                               |
| **3 target categories** | Technology, Company/Organization, Individual/Person                                                                                                                                                                                      |
| **2 types**             | **Active** (any direct interaction, even under a fake identity) vs. **Passive** (pure observation, zero interaction)                                                                                                                     |
| **Search engine rule**  | Never trust page 1 — SEO/paid content dominates it; real value starts from page 2 onward                                                                                                                                                 |
| **Google Dorks**        | Use Advanced Search filters (or their manual query equivalents: quotes for exact phrase, `-` to exclude, `site:` to restrict domain) — but only apply filters you have real justification for, to avoid over-restricting results to zero |
| **OSINT Framework**     | A categorized directory of recon tools/sources (by username, email, domain, etc.) — supports deep dives like WHOIS, DNS records, traceroute, and service/port scanning                                                                   |
| **Shodan**              | Search engine for exposed devices/services; register privately (disposable email) and retrieve an API key for tool integrations                                                                                                          |
| **theHarvester**        | Aggregates data from many OSINT sources (incl. Shodan) in one command; configure API keys in `/etc/theHarvester/api-keys.yaml`; use `-d` (domain), `-b` (sources), `-l` (result limit), `-f` (save to file)                              |

---

### DNS Record Types Referenced (for quick recall)

| Record  | Purpose                                                                                                                                                            |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **A**   | Maps a domain name to an IPv4 address                                                                                                                              |
| **MX**  | Specifies mail server(s) for the domain                                                                                                                            |
| **NS**  | Specifies the domain's authoritative nameservers                                                                                                                   |
| **TXT** | General text record — commonly used to verify domain ownership / prevent email spoofing (e.g., SPF/DKIM-style verification)                                        |
| **SOA** | Start of Authority — contains administrative info, including a contact email for the domain                                                                        |
| **PTR** | Pointer record — used for reverse DNS lookups; can be leveraged to redirect/obscure where a lookup actually points (explored further in Advanced Network Scanning) |
