# Day 07 — Passive Reconnaissance: Social Media OSINT

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 06 — WHOIS & DNS Recon](#) | Next: [Day 08](#)

---

## 📋 Table of Contents

1. [Why Social Media for Recon?](#1-why-social-media-for-recon)
2. [What to Look For](#2-what-to-look-for)
3. [LinkedIn Recon](#3-linkedin-recon)
   - [Company Page](#company-page)
   - [Job Postings](#job-postings)
   - [Employee Profiles](#employee-profiles)
   - [Finding All Employees via Google Dork](#finding-all-employees-via-google-dork)
4. [Instagram Recon](#4-instagram-recon)
5. [Targeting Strategy](#5-targeting-strategy)
6. [Tool: Sherlock — Username OSINT](#6-tool-sherlock--username-osint)
7. [Attack Scenarios from Social Media Recon](#7-attack-scenarios-from-social-media-recon)
8. [Quick Reference Cheat Sheet](#8-quick-reference-cheat-sheet)

---

## 1. Why Social Media for Recon?

Social media platforms are a goldmine of **freely available, publicly posted** information about a target company and its employees. This makes it pure **passive recon** — no interaction with the target's systems whatsoever.

**What attackers look for:**
- Who works there (names, roles, departments)
- What technologies/software the company uses
- New joiners (easier to phish — less security awareness)
- Physical access clues (ID cards, office photos)
- Personal details useful for social engineering

---

## 2. What to Look For

| Category | What to Find | Why It Matters |
|----------|-------------|----------------|
| **Employee names & roles** | HR, developers, admins, new joiners | Choose the best target for phishing |
| **Tech stack** | Job postings listing required skills | Build a tailored/targeted exploit |
| **Internal software** | Laptop screens in photos, job descriptions | Identify specific CVEs to exploit |
| **ID cards / badges** | Photos employees post online | Potential physical access cloning |
| **Office photos** | Layout, desks, monitors, screens | Physical security assessment |
| **Personal interests** | Pets, hobbies, events | Build rapport for social engineering pretext |
| **New employees** | "Just joined" posts | Lower security awareness = easier target |

---

## 3. LinkedIn Recon

LinkedIn is the **most valuable** platform for corporate recon.

---

### Company Page

Go to the target company's official LinkedIn page and check:

| Section | What You Learn |
|---------|---------------|
| **About** | Company size, HQ location, industry, specialties |
| **Employee count** | Scale of the organization |
| **Posts** | Events, tools they use, conferences they attend |
| **Associated members** | Total people connected to the company |

**Example findings from microsoft.com LinkedIn:**
- 10,000+ employees
- HQ: Redmond, Washington
- Specialties: Business software, developer tools, cloud services
- Leadership: CEO (Satya Nadella), CFO (Amy Hood), CTO (Kevin Scott)

---

### Job Postings

Job postings **reveal the company's technology stack** — this is critical for building targeted attacks.

**What to look for in job descriptions:**
- Programming languages required (Python, C++, Java, etc.)
- Frameworks and platforms (Azure, AWS, Kubernetes, etc.)
- Internal tools mentioned
- Department-specific roles (tells you the org structure)

> 🎯 **Why this matters:** If a job posting says "must know Adobe Acrobat X" and there's a public CVE for that version, you now have a potential exploit tailored to their environment.

**How to browse job postings:**
1. Go to the company's LinkedIn page
2. Click **Jobs** tab
3. Read each posting's **Responsibilities** and **Requirements** sections carefully

---

### Employee Profiles

Searching employee profiles gives you:

| Info | What to Do With It |
|------|-------------------|
| **Full name** | Use in spear phishing email ("Hi [Name]") |
| **Job title & department** | Know who has access to what |
| **Work history** | Build a believable pretext |
| **Education** | Reference shared connections |
| **Activity / Posts** | Find personal interests for rapport-building |
| **Recent job change** | New employees are prime phishing targets |

**Who to target (priority order):**
1. 🎯 **New joiners** — less security-aware, more likely to click
2. 🎯 **HR / Recruiters** — regularly open attachments from strangers (CVs)
3. 🎯 **IT support staff** — have elevated privileges
4. ⚠️ **Senior/experienced employees** — harder to trick, but have more access

---

### Finding All Employees via Google Dork

Use this Google Dork to list all employees of a target company from LinkedIn:

```
site:linkedin.com "microsoft.com"
```

**What this returns:** All LinkedIn profiles that mention `microsoft.com` — employees who've listed the company in their profile.

You can refine further:
```
site:linkedin.com "microsoft.com" "senior engineer"
site:linkedin.com "microsoft.com" "HR"
site:linkedin.com "microsoft.com" "new"
```

---

## 4. Instagram Recon

Instagram is useful for finding **visual information** employees accidentally share.

**What to search:**
```
#microsoft
#microsoftlife
#newjob + "microsoft"
#microsoft + "new joiner"
```

**What to look for in photos:**

| Photo Type | Intelligence You Gain |
|-----------|----------------------|
| **ID card / badge visible** | Badge design, format, colors — can be cloned for physical access |
| **Laptop screen in background** | Software versions, internal tools, open files |
| **Office desk photo** | Security camera placement, office layout |
| **"New job" celebration posts** | Identifies new joiners — prime phishing targets |
| **Team photos** | Names, faces, department info |
| **Personal hobby posts** | Pretext for social engineering contact |

> 🐕 **Example from the video:** An HR employee posted a photo of their dog. An attacker can use this as a social engineering pretext — reach out about a "dog event" or common dog owner interest, build rapport, then leverage that relationship later.

---

## 5. Targeting Strategy

```
High Value Targets (for initial access):
┌─────────────────────────────────────────────────────────┐
│ Priority 1: NEW JOINERS                                 │
│  → Less security awareness                              │
│  → More likely to click unfamiliar links/attachments   │
│  → Don't know all internal processes yet               │
├─────────────────────────────────────────────────────────┤
│ Priority 2: HR / RECRUITERS                             │
│  → Regularly open CVs/attachments from strangers       │
│  → Easy pretext: fake job application                  │
├─────────────────────────────────────────────────────────┤
│ Priority 3: IT / HELPDESK                               │
│  → Have elevated system access                         │
│  → Can be socially engineered via "tech support" angle │
├─────────────────────────────────────────────────────────┤
│ Lower Priority: SENIOR / EXPERIENCED STAFF              │
│  → Have high privileges but harder to trick            │
│  → More likely to recognize phishing attempts          │
└─────────────────────────────────────────────────────────┘
```

---

## 6. Tool: Sherlock — Username OSINT

**Sherlock** searches for a given username across **hundreds of websites** simultaneously — to find all social media accounts of a target person.

### Installation

```bash
# Install via pip
pip install sherlock-project

# Or clone from GitHub
git clone https://github.com/sherlock-project/sherlock.git
cd sherlock
pip install -r requirements.txt
```

> ✅ Sherlock comes **pre-installed on Kali Linux**.

### Basic Usage

```bash
sherlock <username>
```

**Example:**
```bash
sherlock billgates
```

**What it does:**
- Searches 300+ websites (Twitter/X, Reddit, Instagram, GitHub, TikTok, etc.)
- Reports which sites have an account with that username
- Marks sites where the username is **not found** vs **found**

### Useful Flags

```bash
sherlock <username>                     # Basic search across all sites
sherlock <username> --timeout 10        # Set request timeout (default is 30s)
sherlock <username> --output results.txt  # Save output to file
sherlock <username> --print-found       # Only print sites where found
sherlock <username> --site twitter      # Search specific site only
sherlock user1 user2 user3              # Search multiple usernames at once
```

### Example Output
```
[*] Checking username billgates on:
[+] GitHub: https://github.com/billgates
[+] Reddit: https://reddit.com/user/billgates
[-] Instagram: Not Found
[+] Twitter: https://twitter.com/billgates
...
```

### Workflow

```
1. Find target's username from LinkedIn / Instagram / any public profile
        ↓
2. Run: sherlock <username>
        ↓
3. Get all platforms where they have an account
        ↓
4. Visit each profile for more personal details (hobbies, location, habits)
        ↓
5. Use gathered info to build a convincing social engineering pretext
```

---

## 7. Attack Scenarios from Social Media Recon

### Scenario 1: Phishing a Recruiter via Fake CV
```
Recon:    Found "Diana" — Technical Recruiter at Microsoft (LinkedIn)
          She posted she's looking for Software Engineers in Korea
Vector:   Send a phishing email with a "CV" attachment
Payload:  CV file (PDF/DOCX) with embedded malware
Result:   Diana opens the CV → malware executes → attacker gets access
Why it works: Recruiters are conditioned to open attachments from strangers
```

### Scenario 2: Targeting a New Joiner
```
Recon:    Found "Victoria" — just joined Microsoft Feature Studio (LinkedIn/Instagram)
          She posted a "new job" celebration photo with team members
Vector:   Send a phishing email that looks like an internal onboarding email
Payload:  "Please complete your onboarding setup here" → fake login page
Result:   Victoria enters credentials → attacker captures them
Why it works: New employees expect onboarding emails, less suspicious
```

### Scenario 3: Badge Cloning for Physical Access
```
Recon:    Found a LinkedIn post by "Janvi Thakkar" (Software Engineer)
          She posted a photo showing her ID badge clearly
Action:   Screenshot the badge design, colors, format, logo placement
Result:   Clone/duplicate the badge using a card printer
Why it works: Physical security bypassed using publicly posted info
```

### Scenario 4: Social Engineering via Personal Interest
```
Recon:    Found an HR employee posted photos of her dog on Instagram
Vector:   Send a DM/email related to dogs ("dog adoption event", "lost dog nearby")
Goal:     Build personal rapport → later leverage for phishing / info extraction
Why it works: Personal connection lowers guard
```

---

## 8. Quick Reference Cheat Sheet

### LinkedIn Recon Checklist
```
□ Company About page (size, HQ, specialties)
□ Company job postings (tech stack, tools used)
□ Employee list (via Google dork below)
□ New joiners (search #newjob #microsoft)
□ HR / recruiter profiles
□ Posts mentioning internal events or software
```

### Instagram Recon Checklist
```
□ Search #[companyname] and #[companyname]life
□ Look for ID badges in photos
□ Look for laptop screens showing software
□ Look for "new job" posts (new joiners)
□ Note personal interests (pets, hobbies) for pretext
```

### Google Dork — Find All Employees
```bash
site:linkedin.com "target-company.com"
site:linkedin.com "target-company.com" "HR"
site:linkedin.com "target-company.com" "engineer"
site:linkedin.com "target-company.com" "new"
```

### Sherlock — Username Search
```bash
sherlock <username>                       # Search username across 300+ sites
sherlock <username> --print-found         # Show only found results
sherlock <username> --output out.txt      # Save results to file
sherlock user1 user2                      # Multiple usernames at once
```

### Platform Priority for Corporate Recon
```
LinkedIn   → Employees, roles, tech stack, job postings
Instagram  → Photos, badges, office visuals, personal info
Twitter/X  → Public statements, tech preferences, event attendance
GitHub     → Code repos, leaked credentials, internal tools
Facebook   → Personal info, friend network, family details
```

---

> ✅ **End of Day 07 Notes**  
> 📌 Next up: More **Passive / Active Reconnaissance** techniques  
> ⚠️ **Reminder:** Only perform recon on targets you have explicit written permission to test, or on yourself as practice.  
> 💬 Questions? Drop them in the comments!