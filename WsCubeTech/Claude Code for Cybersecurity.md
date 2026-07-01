# Claude Code for Cybersecurity — Course Notes

## https://youtu.be/q3U85bHJYSQ

> Source: Cybersecurity course video (Claude Code, Agentic AI & Automation for Security Work)
> Format: Structured notes for revision

---

## 1. The Problem: Why This Topic Matters

Imagine you're a cybersecurity analyst doing reconnaissance on a target. Normally this means:

- Google searches, LinkedIn searches
- Finding subdomains
- Taking screenshots, writing notes, writing reports
- Ending up with 15+ browser tabs open without realizing it

This is where **agentic AI** — specifically **Claude Code** — comes in. It's not a hacking tool or a "magic vulnerability finder," but it _is_ extremely useful for cybersecurity, bug bounty, SOC operations, threat hunting, and automation.

### What This Session Covers

1. What Claude Code actually is
2. How it's different from ChatGPT / regular AI
3. What "agentic AI" means
4. Key concepts: Skills, MCPs, Hooks
5. How security professionals use it in real workflows

**Note:** No coding requirement to understand these concepts — just conceptual understanding.

---

## 2. Legal & Ethical Boundaries (Must Understand First)

Before learning automation, you must understand the legal boundaries of using it.

### Ethical vs Malicious Hacking

The core difference is **permission**.

|            | Ethical Hacking              | Malicious Hacking     |
| ---------- | ---------------------------- | --------------------- |
| Permission | You have explicit permission | No permission         |
| Legality   | Legal                        | Illegal — a crime     |
| Tools used | Can be the same tools        | Can be the same tools |

> **Penalty in India:** Testing without permission can lead to **up to 3 years of imprisonment**.

### What You CAN Legally Do

1. **Bug Bounty Programs** — Platforms like HackerOne, Bugcrowd formally invite you to test their systems.
2. **Practice Platforms** — Hack The Box, TryHackMe, DVWA — learning built from scratch legally.
3. **Your Own Lab** — Build a lab using a hypervisor (e.g., VMware) with:
   - An **attacker machine** (VM 1)
   - A **target machine** (VM 2)
   - You attack your own target VM to find and exploit vulnerabilities — for practice only.
4. **Authorized Penetration Testing** — A company hires you, a **contract** is signed defining your exact attack surface (scope). You must strictly follow that contract.

### Key Rule on Scope

- If you **intentionally** go outside the agreed attack surface → **illegal**.
- If you **accidentally** stumble onto a vulnerability outside scope → you can report it, but must do so **professionally** and separately.

> **Golden Rule:** Never test on random websites without permission — it is harmful and illegal.

---

## 3. What is Claude Code? (vs Regular AI like ChatGPT/Browser Claude)

### The Core Difference

| Capability                     | ChatGPT / Browser Claude                | Claude Code                   |
| ------------------------------ | --------------------------------------- | ----------------------------- |
| Answer questions               | ✅ Yes                                  | ✅ Yes                        |
| Open your files                | ❌ No                                   | ✅ Yes                        |
| Run programs/commands          | ❌ No                                   | ✅ Yes                        |
| Read program output            | ❌ No                                   | ✅ Yes                        |
| Fix its own mistakes           | ❌ No (you must point it out each time) | ✅ Yes (adapts automatically) |
| Organize results automatically | ❌ No                                   | ✅ Yes                        |

### Simple Analogy

- **ChatGPT / Browser Claude** = a smart friend who gives you advice — but **you** have to do the actual work.
- **Claude Code** = like **hiring an employee/intern** — it sits at your desk, uses your computer, runs your tools, and actually gets the work done. You train it once, and it performs the manual tasks for you.

---

## 4. Browser Claude vs Claude Code — Technical Differences

| Aspect                   | Browser Claude                                       | Claude Code                                                                                        |
| ------------------------ | ---------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Where it runs**        | On Anthropic's servers                               | Locally, on your own computer                                                                      |
| **Data location**        | Data briefly goes to Anthropic's servers (encrypted) | Data stays on your local machine only                                                              |
| **Can run programs**     | ❌ No                                                | ✅ Yes                                                                                             |
| **Best for**             | Quick questions, learning concepts, general code     | Automation, sensitive data, repetitive/advanced tasks (using it for quick questions is "overkill") |
| **Access to your files** | ❌ No                                                | ✅ Yes, but **only with your permission**                                                          |

### How Data Travels

**Browser Claude:**

1. You type a question → data is encrypted → sent to Anthropic's servers
2. Claude processes it on Anthropic's servers
3. Encrypted answer sent back to you
4. Fine for general questions/public info, **but NOT ideal for confidential/client data** — since it briefly leaves your system.

**Claude Code:**

1. You type a command
2. Data **never leaves your computer**
3. Everything happens in your **local environment**
4. **Better for:** sensitive records, confidential client information, internal company scanning.

---

## 5. Security Concerns & Safeguards

Common concern: _"If Claude Code can access files and run on my system, can it access everything without my permission?"_

### How Claude Code Handles This

- It can **only** access what **you explicitly grant permission** for.
- You can configure it to **ask for approval before every action**.
- Claude has **built-in safety guardrails** — for sensitive tasks, it will **not** proceed automatically; it will confirm with you and show warnings first.
- **Everything runs on your computer** — nothing is sent to the cloud during execution.

### Best Practices

- **Limit access** to your project folders only.
- **Never share sensitive data** with any AI (well-established security principle).
- You decide exactly: which folders it can access, which tasks it can perform, which files it can touch.

---

## 6. The Three "Powers" of Claude Code

1. **It Acts** — Runs actual programs on your computer; doesn't just explain things, it does the work.
2. **It Adapts** — If something breaks, it reads the error, tries again, and fixes its own mistakes.
3. **It Automates** — This is the **biggest power**. It performs repetitive tasks consistently — used across Blue Teaming, Red Teaming, GRC, and more, not just bug bounty.

---

## 7. Claude Models Comparison

Three models, differing in speed, intelligence, and cost:

| Model      | Speed     | Intelligence                    | Cost      | Best For                                                                              |
| ---------- | --------- | ------------------------------- | --------- | ------------------------------------------------------------------------------------- |
| **Haiku**  | Very fast | Good (lower relative to others) | Cheap     | Quick tasks                                                                           |
| **Sonnet** | Medium    | Good                            | Moderate  | **Recommended for bug bounty/recon work** — fast enough, smart enough, cost-effective |
| **Opus**   | Slower    | Very high (best)                | Expensive | Complex reasoning tasks                                                               |

**Analogy:**

- Sonnet = your everyday **family car** (used for daily/regular work)
- Opus = a **BMW** you use occasionally for something complex

**Why Sonnet for bug bounty recon:** It balances speed, cost, and intelligence well enough to handle typical scanning/recon edge cases without being overkill.

---

## 8. Key Terminology (Must Know Before Using Claude Code)

### 1. Skills

A **reusable instruction manual**. You write instructions once (e.g., "when I say run recon, do subdomain finding and map out X, Y, Z, then save it"), save it as a **Skill**, and afterward simply say **"run recon skill"** — Claude repeats the exact same process without you re-writing instructions each time.

> **Analogy:** Like writing down a recipe once — anyone (or the AI) can reuse that recipe without you re-explaining it.

### 2. Headless Mode

Normally, an AI/assistant might check in with you after **every single step** ("Is this okay? Is this okay?") — which becomes tedious.

**Headless Mode** = Claude executes your full plan **without interrupting you at every step**; it completes the task and you review the results at the end.

- Useful for **overnight scanning** — set it running, and it works independently without needing constant confirmations.

### 3. Context

Claude's **working memory** of your conversation — essentially how much text/history it can "remember" at once.

- After extended chatting (many hours), Claude may **forget** what was said earlier.
- **Solution:** Start a fresh conversation every 1–2 hours for a clean/fresh memory when doing long work sessions.

### 4. MCP (Model Context Protocol)

How Claude **connects to other tools** — think of it like a **USB port**.

- Without MCP: Claude is smart but **isolated**.
- With MCP: Claude becomes "a smart person with access to your computer's tools" — e.g.:
  - **File system** — to save reports
  - **Terminal** — to run tools
  - **Web browser** (optional) — to read web content

### 5. Hooks

**Automatic triggers → automatic actions.**

Examples:

- When a scan finishes → **automatically save the results**
- If an error happens → **automatically retry** the task
- When a report is complete → **automatically move it** to a designated folder

> Set it up once, and it runs in the background automatically.

---

## 9. What You Can Build: Security Reconnaissance Assistant (Project Concept)

**Concept:** A "Security Reconnaissance Assistant" — essentially **recon automation**.

### Purpose

Automate the **repetitive reconnaissance phase** of bug bounty hunting — the part that normally takes hours of manual work (running tools, collecting scattered results, organizing findings, writing reports).

### What It CAN Do

- Automate running standard recon tools (e.g., Nmap, Sublist3r)
- Collect scattered results from multiple tools into one place
- Organize findings in a structured way
- Generate readable reports
- Save time on repetitive information gathering
- Reduce human error in running commands

### What It CANNOT Do

- ❌ It is **not an automatic/magic vulnerability finder**
- ❌ It **cannot replace your analysis and judgment** — AI can never replace a human's strategic thinking
- ❌ It doesn't guarantee findings — results vary by target (some targets have findings, some don't)

---

## 10. Realistic Expectations

- Automation handles the **boring parts**: scanning, information gathering, organizing, reporting.
- **Your role** remains: **analysis, strategy, and actually identifying real security issues** — since a large volume of data still needs human review.
- Results vary per target — not every target will yield findings.
- Legal requirements (see Section 2) still fully apply.
- **Your skill in recon + your analysis is what makes a good bug bounty hunter** — the tool just removes repetitive manual work.

---

## 11. Claude Code Use Cases Beyond Bug Bounty

Claude Code isn't limited to bug bounty hunting — it can automate **any repetitive task**:

| Team/Area          | Use Case                              |
| ------------------ | ------------------------------------- |
| **Red Team**       | Automate recon and scanning           |
| **Blue Team**      | Automate log analysis, threat hunting |
| **DevOps**         | Automate deployment checks            |
| **Compliance/GRC** | Automate report generation            |

---

## 12. Summary — Claude Code Capabilities Recap

**Can do:**

- Automate running standard recon tools
- Collect/organize scattered results from multiple tools
- Generate structured, readable reports
- Save time on repetitive information gathering
- Reduce human error in command execution

**Cannot do:**

- Magically find vulnerabilities on its own
- Replace human analysis and judgment

---

## Quick-Reference Summary

```
1. Legal boundary: Permission = Ethical | No permission = Illegal (up to 3 yrs jail in India)
2. Claude Code vs ChatGPT: Claude Code can OPEN FILES, RUN PROGRAMS, READ OUTPUT, FIX ERRORS — ChatGPT cannot
3. Browser Claude → runs on Anthropic servers (data leaves your system)
   Claude Code → runs locally (data stays on your machine) — better for sensitive data
4. 3 Powers: Acts, Adapts, Automates
5. Models: Haiku (fast/cheap) | Sonnet (balanced — best for recon) | Opus (powerful/expensive)
6. Key terms: Skills (reusable instructions) | Headless Mode (no step-by-step confirmation)
   | Context (working memory) | MCP (connects to tools, like USB) | Hooks (auto triggers/actions)
7. Project idea: Security Recon Assistant — automates repetitive recon, NOT a vulnerability finder
8. Your job remains: analysis, strategy, and judgment — AI cannot replace that
```
