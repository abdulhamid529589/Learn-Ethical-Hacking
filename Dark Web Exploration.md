# 🧅 Dark Web Exploration — Security Researcher's Guide
### Understanding, Navigating, and Researching the Tor Network Safely

> **Purpose:** This guide is for security researchers, ethical hackers, and curious learners
> who want to understand how the dark web works, what exists there, and how to explore
> it safely without exposing yourself or breaking laws.
>
> **Important:** Many parts of the dark web contain illegal content. This guide focuses
> on understanding the technology, legitimate research, and safe exploration.
> Never purchase, download, or interact with illegal content.

---

## 📚 Table of Contents

1. [What is the Dark Web?](#1-what-is-the-dark-web)
2. [How Tor and .onion Sites Work](#2-how-tor-works)
3. [Setting Up Safe Environment](#3-setting-up-safe-environment)
4. [Tor Browser — Complete Setup](#4-tor-browser-complete-setup)
5. [Navigating the Dark Web](#5-navigating-the-dark-web)
6. [Legitimate .onion Sites](#6-legitimate-onion-sites)
7. [Dark Web Search Engines](#7-dark-web-search-engines)
8. [OSINT on the Dark Web](#8-osint-on-the-dark-web)
9. [Dark Web Monitoring](#9-dark-web-monitoring)
10. [Hidden Services — How They Work](#10-hidden-services)
11. [Running Your Own .onion Site](#11-running-your-own-onion-site)
12. [Dark Web Security Research](#12-dark-web-security-research)
13. [What to Avoid](#13-what-to-avoid)
14. [Staying Safe — Complete Checklist](#14-staying-safe)

---

## 1. What is the Dark Web?

### The Three Layers of the Internet

```
SURFACE WEB (4% of internet)
├── Indexed by Google, Bing, etc.
├── Accessible to anyone
└── Examples: Google, YouTube, Wikipedia, news sites

DEEP WEB (96% of internet)
├── NOT indexed by search engines
├── Requires login or direct URL
├── NOT illegal or suspicious!
└── Examples:
- Your email inbox
- Online banking
- Netflix content
- Hospital records
- Corporate intranets
- Academic databases
- Government databases

DARK WEB (small subset of deep web)
├── Requires special software (Tor) to access
├── Intentionally hidden
├── .onion domains (not real DNS)
└── Mix of:
- Legitimate privacy tools
- Journalism and activism
- Academic research
- Illegal marketplaces (avoid!)
- Hacking forums
- Privacy-focused services
```

### Common Misconceptions

```
MYTH: Dark web = only illegal things
TRUTH: Many legitimate services use .onion for privacy
Facebook, BBC, NYT, ProtonMail all have .onion sites

MYTH: Visiting dark web is illegal
TRUTH: Using Tor is legal in most countries
Accessing most .onion sites is legal
What's illegal: buying drugs, CSAM, weapons, etc.

MYTH: You'll get hacked immediately
TRUTH: With proper precautions, exploration is safe
Most danger comes from downloading files or JS exploits

MYTH: Everything is anonymous on dark web
TRUTH: Anonymity requires careful OPSEC
Mistakes can still identify you

MYTH: The dark web is huge
TRUTH: Estimated 60,000-100,000 .onion sites
Most are small, many are offline
Surface web is much larger
```

### Who Uses the Dark Web Legitimately?

```
Journalists:
- Receive documents from whistleblowers securely
- SecureDrop used by NYT, Guardian, Washington Post
- Protect sources in dangerous countries

Activists and dissidents:
- Citizens in authoritarian countries
- Bypass censorship (China, Iran, Russia)
- Organize safely without government surveillance

Privacy-focused individuals:
- People who simply value privacy
- Avoid corporate data collection

Security researchers:
- Study malware and cyber threats
- Monitor threat intelligence
- Understand attack methods

Law enforcement:
- Monitor illegal markets
- Gather intelligence
- Trace criminals

Corporations:
- Monitor if their data is being sold
- Brand protection
- Threat intelligence
```

---

## 2. How Tor Works

### The Onion Routing Explained

```
Normal internet request:
You → ISP → Website
Website sees: Your real IP

Tor request:
You encrypt message 3 times (like onion layers)

Layer 1 (outer):  Encrypted for Guard Node
Layer 2 (middle): Encrypted for Middle Node
Layer 3 (inner):  Encrypted for Exit Node

You → Guard Node (peels layer 1, sees: Middle Node)
→ Middle Node (peels layer 2, sees: Exit Node)
→ Exit Node (peels layer 3, sees: Destination)
→ Website

Nobody has complete picture:
Guard sees: Your IP + Middle Node (not destination)
Middle sees: Guard + Exit (not you, not destination)
Exit sees: Destination + Middle (not your IP)
Website sees: Exit node's IP (not you)
```

### How .onion Addresses Work

```
Regular website:
DNS: example.com → 93.184.216.34 (IP address)
Anyone can look up the IP

.onion site:
NO DNS lookup needed
Address IS the public key of the server
Example: facebookwkhpilnemxj7asber7cybq.onion

When you visit:
1. Tor finds introduction points for that .onion
2. Both you and server build circuits to a rendezvous point
3. Connection established WITHOUT either knowing other's IP!

This is called a "Hidden Service" or "Onion Service"
Server's real IP is never revealed
Your real IP is never revealed
```

### .onion v2 vs v3

```
v2 addresses (OLD, deprecated):
16 characters: abcdefghijklmnop.onion
Example: expyuzz4wqqyqhjn.onion
Weak cryptography (SHA1, 1024-bit RSA)
Being phased out

v3 addresses (CURRENT, secure):
56 characters: facebookwkhpilnemxj7asber7cybq2ibopvx...onion
Strong cryptography (ED25519, SHA3)
Much harder to fake or brute force
All new sites use v3

How to tell: Count the characters in the address
16 chars = v2 (old, less secure)
56 chars = v3 (current standard)
```

---

## 3. Setting Up Safe Environment

### Safety First — Before You Start

```
Minimum safety requirements:
✓ Use Tor Browser (not regular browser through Tor)
✓ Keep Tor Browser updated
✓ Don't enable JavaScript (set to Safest)
✓ Don't download files and open them
✓ Don't log into personal accounts
✓ Use VPN before Tor (optional but good)

Recommended setup:
✓ Tails OS (leaves no trace)
✓ OR Whonix (isolated Tor environment)
✓ VPN before connecting to Tor
✓ All anonymity from previous guide applied
```

### Recommended Environment Options

```
Option 1: Tails OS (BEST for anonymity)
Boot from USB
All traffic through Tor automatically
No trace left on computer
RAM wiped on shutdown
→ Best for serious research

Option 2: Whonix in VirtualBox
Gateway VM routes through Tor
Workstation VM for browsing
Even if compromised, real IP safe
→ Good for long-term research use

Option 3: Parrot OS + Tor Browser + VPN
What you already have
Enable VPN first, then Tor
Use Safest security level in Tor Browser
→ Fine for learning and basic research

DO NOT:
✗ Use regular Chrome/Firefox with Tor proxy
✗ Browse dark web without Tor Browser
✗ Use your daily OS without extra protection
```

---

## 4. Tor Browser — Complete Setup

### Installation and Configuration

```bash
# Install Tor Browser on Parrot OS
sudo apt install torbrowser-launcher
torbrowser-launcher  # Downloads and installs

# Or download directly from torproject.org
wget https://www.torproject.org/dist/torbrowser/[version]/tor-browser-linux64-[version]_ALL.tar.xz

# Verify signature (important!)
gpg --verify tor-browser-linux64-*.asc

# Extract and run
tar -xf tor-browser-linux64-*.tar.xz
cd tor-browser/
./start-tor-browser.desktop
```

### Security Settings — Set to Maximum

```
After opening Tor Browser:

1. Click Shield icon (top right)
→ Advanced Security Settings
→ Security Level: SAFEST

This disables:
- JavaScript (main attack vector!)
- SVG fonts
- Some media formats
- MathML
- CSS cursor customization

2. Settings → Privacy & Security:
✓ Block dangerous and deceptive content
✓ Block dangerous downloads
✓ Warn about unwanted software

3. NoScript (click S icon):
→ Options → Default: BLOCK ALL
Only enable JS if you absolutely must,
for specific trusted sites only
	
	4. Never:
	- Install additional extensions
	- Resize browser window (standardized = anonymous)
	- Enable plugins (Flash, Java, etc.)
	- Open downloaded files in external apps
	```
	
	### Understanding Tor Browser Interface
	
	```
	Address bar: Shows .onion address or normal URL
	Green lock = HTTPS (encrypted to exit node)
	
	Circuit display: Click (i) next to URL
	Shows your 3 Tor nodes:
	[Your Country] → [Node Country] → [Exit Country]
	
	New Identity:    Broom icon = completely new circuit
	Use when you want to "change location"
	
	New Circuit:     Refresh circuit for this site only
	
	Shield icon:     Security level indicator
	Green = Standard
	Yellow = Safer
	Red = Safest (use this!)
	```
	
	### Verify Tor is Working
	
	```
	Visit in Tor Browser:
	https://check.torproject.org
	
	Should show:
	"Congratulations. This browser is configured to use Tor."
	+ The Tor exit node IP (not your real IP)
	
	Also check:
	https://whatismyipaddress.com
	→ Should show a server in another country
	
	https://browserleaks.com/ip
	→ Should show no WebRTC leak
	→ IP should be Tor exit node
	```
	
	---
	
	## 5. Navigating the Dark Web
	
	### How Dark Web URLs Work
	
	```
	.onion addresses look like:
	duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion
	(DuckDuckGo's official .onion)
	
	They are:
	- Case-sensitive
	- Exact (one wrong character = wrong site)
	- Long (v3 = 56 characters)
	- Cannot be guessed or searched in normal DNS
	
	How to find .onion addresses:
	- Dark web wikis and link lists
	- Official sites list their .onion mirrors
	- .onion search engines
	- Security research publications
	- The Hidden Wiki (be careful what you click)
	```
	
	### Basic Navigation Tips
	
	```
	Speed:
	Tor is slow (3 hops worldwide)
	Average page load: 5-30 seconds
	Some pages never load (servers offline)
	Be patient, don't keep reloading
	
	Links:
	NEVER click links without knowing where they go
	Hover over links to see URL before clicking
	Many links lead to scam or illegal sites
	
	HTTP vs HTTPS on .onion:
	.onion sites don't need HTTPS for anonymity
	(connection is encrypted end-to-end through Tor)
	But HTTPS still good practice for integrity
	
	Site reliability:
	Many .onion sites go offline frequently
	Some only online at certain times
	Sites disappear and reappear
	Don't be surprised if a site is down
	
	Loading issues:
	Click the (x) if page hangs for too long
	Request new circuit and try again
	Some sites block Tor exit nodes
	Some require solving CAPTCHAs
	```
	
	---
	
	## 6. Legitimate .onion Sites
	
	### Official Organization .onion Mirrors
	
	These are REAL organizations with OFFICIAL .onion mirrors for privacy:
	
	```
	NEWS AND MEDIA:
	BBC:
	bbcnewsd73hkzno2ini43t4gblxvycyac5aw4gnv7t2rccijh7745uqd.onion
	
	New York Times:
	nytimesn7cgmftshazwhfgzm37qxb44r64ytbb2dj3x62d2lljsciiyd.onion
	
	ProPublica (investigative journalism):
	p53lf57qovyuvwsc6xnrppyply3vtqm7l6pcobkmyqsiofyeznfu5uqd.onion
	
	The Guardian (SecureDrop):
	xp44cagis447k3lpb4wwhcqukix6cgqokbuys24vmxmbzmaq2gjvc2yd.onion
	
	SEARCH ENGINES:
	DuckDuckGo:
	duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion
	
	Ahmia (indexes .onion sites):
	ahmia.fi  (also on regular web)
	juhanurmihxlp77nkq76byazcldy2hlmovfu2epvl5ankdibsot4csyd.onion
	
	SOCIAL MEDIA / COMMUNICATION:
	Facebook:
	facebookwkhpilnemxj7asber7cybq2ibopvx63clq4nfldfuer6znpxid.onion
	(Yes, Facebook has an official .onion!)
	
	Twitter/X:
	twitter3e4tixl4xyajtrzo62zg5vztmjuricljdp2c5kshju4avyoid.onion
	
	EMAIL:
	ProtonMail:
	protonmailrmez3lotccipshtkleegetolb73fuirgj7r4o4vfu7ozyd.onion
	
	Tutanota:
	tutanota.com also accessible via Tor
	
	SECURE DOCUMENT SUBMISSION:
	SecureDrop (multiple news organizations):
	secrdrop5wyphb5x.onion  (directory of all SecureDrop instances)
	Used by: Washington Post, Guardian, NYT, etc.
	Whistleblowers submit documents here securely
	
	PRIVACY TOOLS:
	Tor Project itself:
	2gzyxa5ihm7nsggfxnu52rck2vv4rvmdlkiu3zzui5du4xyclen53wid.onion
	
	EFF (Electronic Frontier Foundation):
	effectiveaztaqzb3dkw5oxtmpiwnnx5c3vl7yqeaxtmwlrpmpwkdnoyd.onion
	```
	
	### Educational and Research .onion Sites
	
	```
	LIBRARIES AND KNOWLEDGE:
	Sci-Hub (research papers, controversial but widely used):
	scihub.onion (various mirrors, search for current)
	
	Library Genesis:
	Books and academic papers
	Various .onion mirrors (search "libgen onion")
	
	Imperial Library:
	kx5thpx2olielkihfyo4jgjqfb7zx7wxr3sd4xzt26ochei4m6f7tayd.onion
	Collection of public domain books
	
	SECURITY RESEARCH:
	Various hacking forums (read-only research):
	Many exist, search via dark web search engines
	Look for: threat intelligence, CVE discussions, tools
	
	PRIVACY AND TECHNOLOGY:
	Riseup (activist communication):
	vww6ybal4bd7szmgncyruucpgfkqahzddi37ktceo3ah7ngmcopnpyyd.onion
	
	i2p (alternative anonymity network):
	Information and resources
	```
	
	---
	
	## 7. Dark Web Search Engines
	
	### How to Search the Dark Web
	
	Unlike Google, dark web search engines don't crawl and index everything automatically. Many .onion sites deliberately hide from search engines.
	
	```
	AHMIA (Best for beginners):
	Clear web: https://ahmia.fi
	.onion: juhanurmihxlp77nkq76byazcldy2hlmovfu2epvl5ankdibsot4csyd.onion
	
	- Filters out illegal content
	- Good for finding legitimate sites
	- Also shows statistics about Tor network
	
	TORCH (Oldest dark web search engine):
	xmh57jrknzkhv6y3ls3ubitzfqnkrwxhopf5aygthi7d6rplyvk3noyd.onion
	- Large index
	- Less filtering
	- More results, mixed quality
	
	HAYSTAK:
	haystak5njsmn2hqkewecpaxetahtwhsbsa64jom2k22z5afxhnpxfid.onion
	- Claims largest index
	- Filters CSAM and other illegal content
	- Good for research
	
	Not Evil:
	notevil.onion
	- Basic search
	- Minimal filtering
	
	DARK SEARCH:
	darksearch.io  (also on regular web)
	- Indexed .onion sites
	- Accessible from regular web too
	```
	
	### Search Tips for Dark Web
	
	```
	Effective searches:
	- Use specific terms (broad terms return too much)
	- Try variations if nothing found
	- Use quotes for exact phrases
	
	What you CAN find with search:
	- Forums and communities
	- News sites
	- Privacy tools
	- Security research resources
	- Whistleblowing platforms
	- Political discussion
	
	What search WON'T show:
	- Sites that block indexing
	- Password-protected areas
	- Dynamically generated pages
	- Newly created sites
	- Illegal marketplaces (deliberately hidden)
	```
	
	---
	
	## 8. OSINT on the Dark Web
	
	Open Source Intelligence gathering on the dark web is a legitimate security research skill.
	
	### What Dark Web OSINT Means
	
	```
	Monitoring for:
	- Your organization's stolen data
	- Leaked credentials
	- Threat actor discussions
	- Planned attacks
	- Vulnerability trading
	- Ransomware victim lists
	- Your company mentioned in forums
	
	This is done by:
	- Corporate security teams
	- Threat intelligence companies
	- Government agencies
	- Security researchers
	- Journalists
	```
	
	### Tools for Dark Web OSINT
	
	```bash
	# OnionSearch - search multiple dark web engines
	pip3 install onionsearch
	
	# Search across all dark web engines
	onionsearch "search term"
	onionsearch "company name breach"
	
	# Hunchly - professional dark web capture tool
	# (paid, but widely used by investigators)
	
	# Maltego - OSINT platform with dark web module
	
	# DarkOwl, Recorded Future, Flashpoint
	# (commercial threat intelligence platforms)
	# They continuously monitor dark web for clients
	
	# Manual research approach:
	# 1. Use Tor Browser
	# 2. Visit known forums
	# 3. Search for specific terms
	# 4. Document findings (screenshots)
	# 5. Report if you find crimes in progress
	```
	
	### Monitoring Your Own Data
	
	```python
	# check_dark_web.py
	# Check if your email appears in breaches
	# Uses Have I Been Pwned API (no dark web access needed)
	
	import requests
	import hashlib
	
	def check_email_breach(email):
	"""Check if email appears in data breaches"""
	url = f"https://haveibeenpwned.com/api/v3/breachedaccount/{email}"
	headers = {
		'hibp-api-key': 'YOUR_API_KEY',  # Get free at haveibeenpwned.com
		'user-agent': 'SecurityResearch'
	}
	
	resp = requests.get(url, headers=headers)
	
	if resp.status_code == 200:
		breaches = resp.json()
		print(f"[!] {email} found in {len(breaches)} breaches!")
		for b in breaches:
			print(f"    - {b['Name']} ({b['BreachDate']})")
			print(f"      Data: {', '.join(b['DataClasses'])}")
			elif resp.status_code == 404:
			print(f"[✓] {email} not found in any known breaches")
			
			def check_password_breach(password):
			"""Check if password appears in breach databases"""
			# Uses k-anonymity (never sends full password!)
			sha1 = hashlib.sha1(password.encode()).hexdigest().upper()
			prefix = sha1[:5]
			suffix = sha1[5:]
			
			url = f"https://api.pwnedpasswords.com/range/{prefix}"
			resp = requests.get(url)
			
			for line in resp.text.splitlines():
				hash_suffix, count = line.split(':')
				if hash_suffix == suffix:
					print(f"[!] Password found in {count} breaches! Change it!")
					return
					
					print("[✓] Password not found in known breaches")
					
					# Check your own email
					check_email_breach("your@email.com")
					
					# Check a password (safe - uses k-anonymity)
					check_password_breach("your_password")
					```
					
					### Paste Sites and Leak Monitoring
					
					```bash
					# Pastebin and similar sites often contain leaked data
					# Monitor them for your information
					
					# Tools:
					# PasteHunter - monitors paste sites
					pip3 install pastehunter
					
					# Manual paste sites to check:
					# https://pastebin.com
					# https://ghostbin.com
					# https://rentry.co
					# https://privatebin.net
					
					# Dark web paste sites:
					# zerobin.net (also on .onion)
					
					# Search for your email/domain in pastes:
					# Many breach dumps end up here first
					
					# Google dork for pastes:
					# site:pastebin.com "yourcompany.com" password
					# site:pastebin.com "your@email.com"
					```
					
					---
					
					## 9. Dark Web Monitoring
					
					### Setting Up Automated Monitoring
					
					```python
					# dark_web_monitor.py
					# Monitor dark web for specific keywords
					# Runs through Tor
					
					import requests
					import time
					import json
					from datetime import datetime
					
					# Configure for Tor
					TOR_PROXY = {
						'http': 'socks5h://127.0.0.1:9050',
						'https': 'socks5h://127.0.0.1:9050'
					}
					
					KEYWORDS_TO_MONITOR = [
						"your-company.com",
"your@email.com",
"YOUR_USERNAME",
# Add what you want to monitor
					]
					
					SEARCH_ENGINES = [
						"http://juhanurmihxlp77nkq76byazcldy2hlmovfu2epvl5ankdibsot4csyd.onion/search/?q=",  # Ahmia
					]
					
					def search_onion(query, engine):
					"""Search a .onion search engine"""
					try:
					url = engine + requests.utils.quote(query)
					resp = requests.get(url, proxies=TOR_PROXY, timeout=60)
					return resp.text
					except Exception as e:
					return None
					
					def monitor_keywords():
					"""Monitor dark web for keywords"""
					results = []
					timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
					
					for keyword in KEYWORDS_TO_MONITOR:
						print(f"[*] Searching for: {keyword}")
						
						for engine in SEARCH_ENGINES:
							content = search_onion(keyword, engine)
							
							if content and keyword.lower() in content.lower():
								print(f"[!] FOUND: '{keyword}' on dark web!")
								results.append({
									'timestamp': timestamp,
									'keyword': keyword,
									'engine': engine,
									'snippet': content[:500]
								})
								
								time.sleep(5)  # Rate limiting
								
								return results
								
								def save_results(results):
								with open('/tmp/dark_web_alerts.json', 'a') as f:
								for r in results:
									f.write(json.dumps(r) + '\n')
									
									# Run monitoring
									print("[*] Starting dark web keyword monitoring...")
									print("[*] Make sure Tor is running (systemctl start tor)")
									
									while True:
										results = monitor_keywords()
										if results:
											save_results(results)
											print(f"[!] {len(results)} alerts saved!")
											else:
												print("[✓] No alerts found in this scan")
												
												print(f"[*] Next scan in 1 hour...")
												time.sleep(3600)
												```
												
												---
												
												## 10. Hidden Services — How They Work
												
												### Technical Deep Dive
												
												```
												Setting up a hidden service (how .onion sites work):
												
												1. Server generates ED25519 key pair
												- Private key: kept secret (= identity of the site)
												- Public key: hashed to create .onion address
												
												2. Server advertises itself to Tor network
												- Chooses 3 Introduction Points (IP)
												- Sends signed service descriptor to HSDir nodes
												- HSDir = Hash-distributed directory nodes
												- They store the service descriptor
												
												3. Client wants to connect:
												a. Downloads service descriptor from HSDir
												(gets list of Introduction Points)
												b. Creates a circuit to a Rendezvous Point (RP)
												c. Sends RP address + one-time secret to server via IP
												d. Server creates circuit to RP
												e. Connection established through RP
												
												4. Neither knows the other's real IP:
												- Client knows only RP address
												- Server knows only RP address
												- RP knows client circuit + server circuit but not IPs
												- Traffic is encrypted end-to-end
												
												The .onion address IS the public key
												You can verify you're talking to the right server
												No DNS = no DNS hijacking possible
												```
												
												### Types of Hidden Services
												
												```
												Category 1: Web servers
												Standard HTTP servers accessible via .onion
												Most common type
												
												Category 2: Email servers
												SMTP/IMAP over Tor
												ProtonMail, others
												
												Category 3: IRC servers
												Anonymous chat networks
												Some still active
												
												Category 4: File sharing
												BitTorrent over Tor
												I2P integration
												
												Category 5: API services
												Programmatic access to services
												Monitoring, intelligence feeds
												
												Category 6: Marketplaces
												(Many illegal - avoid)
												Some legitimate: security tools, legal items
												
												Category 7: Forums
												Discussion communities
												Security research, privacy advocacy
												Political discussion
												```
												
												---
												
												## 11. Running Your Own .onion Site
												
												For research purposes — understand how hidden services work by creating one.
												
												### Simple .onion Web Server
												
												```bash
												# Install Tor and nginx
												sudo apt install tor nginx
												
												# Configure Tor to create hidden service
												sudo nano /etc/tor/torrc
												
												# Add these lines:
												HiddenServiceDir /var/lib/tor/hidden_service/
												HiddenServicePort 80 127.0.0.1:80
												
												# Save and restart Tor
												sudo systemctl restart tor
												
												# Wait 30-60 seconds, then get your .onion address:
												sudo cat /var/lib/tor/hidden_service/hostname
												# Output: something like:
												# xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.onion
												
												# Configure nginx to serve your site
												sudo nano /etc/nginx/sites-available/default
												
												server {
													listen 127.0.0.1:80;
													root /var/www/html;
													index index.html;
												}
												
												# Create a simple page
												echo "<h1>My Research .onion Site</h1>" | \
												sudo tee /var/www/html/index.html
												
												# Start nginx
												sudo systemctl start nginx
												
												# Test it!
												# Open Tor Browser and visit your .onion address
												# (the one from hostname file)
												```
												
												### Running a Python .onion Server
												
												```python
												# onion_server.py
												# Simple Flask server as a hidden service
												
												from flask import Flask, render_template_string
												import subprocess
												
												app = Flask(__name__)
												
												HTML = """
												<!DOCTYPE html>
												<html>
												<head><title>Research Hidden Service</title></head>
												<body>
												<h1>🧅 Research .onion Site</h1>
												<p>This is a test hidden service for security research.</p>
												<p>If you can read this, your Tor connection works!</p>
												<hr>
												<p><small>Accessed via Tor Hidden Service</small></p>
												</body>
												</html>
												"""
												
												@app.route('/')
												def home():
												return render_template_string(HTML)
												
												@app.route('/info')
												def info():
												return {
													'status': 'running',
													'type': 'research hidden service',
													'note': 'For educational purposes only'
												}
												
												if __name__ == '__main__':
													app.run(host='127.0.0.1', port=5000)
													
													# Also configure torrc:
													# HiddenServiceDir /var/lib/tor/research_service/
													# HiddenServicePort 80 127.0.0.1:5000
													```
													
													### Vanity .onion Address (Custom Start)
													
													```bash
													# Generate custom .onion address with specific prefix
													# Example: research........onion
													
													# Install mkp224o (v3 onion vanity generator)
													sudo apt install mkp224o
													# or:
													git clone https://github.com/cathugger/mkp224o.git
													cd mkp224o
													./autogen.sh && ./configure && make
													
													# Generate address starting with "research"
													./mkp224o research
													
													# Warning: this is computationally intensive!
													# 6-char prefix: minutes to hours
													# 8-char prefix: days to weeks
													# 10-char prefix: months to years
													
													# Output: directory with:
													# - hs_ed25519_public_key
													# - hs_ed25519_secret_key
													# - hostname (your .onion address)
													
													# Copy to Tor hidden service directory
													sudo cp -r research*/ /var/lib/tor/my_service/
													sudo chown -R debian-tor:debian-tor /var/lib/tor/my_service/
													sudo chmod 700 /var/lib/tor/my_service/
													
													sudo systemctl restart tor
													sudo cat /var/lib/tor/my_service/hostname
													# research[rest of address].onion
													```
													
													---
													
													## 12. Dark Web Security Research
													
													### What Security Researchers Look For
													
													```
													Threat Intelligence:
													- New malware samples being sold
													- Zero-day exploits available for purchase
													- Ransomware group announcements
													- Data breach dumps
													- Compromised credential lists
													- Attack planning discussions
													
													Defensive Use:
													- Early warning of attacks
													- Understanding attacker tools/techniques
													- Identifying if your org is targeted
													- Recovering stolen data evidence
													
													Research Areas:
													- Cryptocurrency tracing (following money)
													- Forum analysis (who are threat actors?)
													- Malware evolution tracking
													- Market economics of cybercrime
													```
													
													### Safely Analyzing Dark Web Threats
													
													```bash
													# NEVER download malware to your main system
													# Use isolated VM for any suspicious files
													
													# Set up analysis VM (separate from Tor VM)
													# See: Binary RE and Malware Analysis notes
													
													# Document everything with screenshots
													# Use a screenshot tool:
													sudo apt install scrot
													scrot screenshot.png    # Take screenshot
													
													# Take notes on what you find
													# Date, URL, content summary, significance
													# Keep research journal
													
													# Share findings responsibly:
													# - Report active crimes to law enforcement
													# - Responsible disclosure of vulnerabilities
													# - Publish research through proper channels
													
													# Tools for professional dark web research:
													# Hunchly: captures web sessions automatically
													# Maltego: visualize connections
													# i2p: alternative network to research
													```
													
													### Cryptocurrency Tracing (Following the Money)
													
													```
													Dark web transactions mostly use:
													Bitcoin (traceable! all transactions public)
													Monero (much harder to trace)
													Zcash (privacy-focused)
													Litecoin (older markets)
													
													Bitcoin is NOT anonymous:
													All transactions on public blockchain
													Can trace money flows
													Many criminals caught this way
													
													Tools for cryptocurrency research:
													- Blockchain.info: view Bitcoin transactions
													- Chainalysis: professional tracing tool
													- Elliptic: compliance and investigation
													- OXT.me: Bitcoin OSINT
													
													Monero is much harder:
													- Ring signatures hide sender
													- Stealth addresses hide receiver
													- Confidential transactions hide amount
													- Much harder for law enforcement
													
													Research opportunity:
													Transaction analysis is active area of academic research
													Many universities research this
													```
													
													---
													
													## 13. What to Avoid
													
													### Illegal Content — Never Interact With
													
													```
													ABSOLUTELY NEVER:
													✗ Child sexual abuse material (CSAM)
													- Illegal everywhere, severe penalties
													- Even viewing is a crime
													- Report to: cybertipline.org (NCMEC)
													- Report to: iwf.org.uk (Internet Watch Foundation)
													
													✗ Purchasing illegal drugs
													- Illegal even researching purchases
													- Fentanyl and other drugs killing people
													- Law enforcement actively monitors markets
													
													✗ Weapons and explosives
													- Illegal to purchase without proper licensing
													- Many are stings by law enforcement
													
													✗ Hired hitman/violence-for-hire
													- Almost always scams
													- But still conspiracy charges possible
													
													✗ Counterfeit currency/documents
													- Serious federal crimes
													
													✗ Stolen data/credentials
													- Possession can be criminal even if not used
													
													✗ Hacking services (booters, DDoS-for-hire)
													- Illegal to purchase even if not used yourself
													```
													
													### Scams — Dark Web is Full of Them
													
													```
													Common scams:
													Exit scams:
													- Market takes deposits, then disappears
													- Very common with dark web marketplaces
													- Lost millions of dollars
													
													Hitman scams:
													- "Rent a hitman" sites
													- All are scams, take your money
													- Plus you could face conspiracy charges
													
													Fake hacking services:
													- "Hack this person's email for $200"
													- Take money, do nothing
													- Or worse: phish you for your money
													
													Bitcoin doublers:
													- "Send 1 BTC, get 2 BTC back"
													- Always scams
													
													Honeypots (law enforcement):
													- Fake markets/forums run by agencies
													- Designed to catch buyers and sellers
													- Many successful operations (Silk Road, AlphaBay)
													```
													
													### Technical Dangers
													
													```
													Malware risks:
													- Malicious .onion sites serve exploits
													- Drive-by downloads through JavaScript
													- Fake tools containing RATs/malware
													Fix: Keep JavaScript disabled (Safest mode)
													Never download and open files
													
													Phishing:
													- Fake .onion addresses (one character off)
													- Fake login pages to steal credentials
													Fix: Bookmark real addresses
													Verify .onion addresses from multiple sources
													
													Tracking:
													- Some sites try to de-anonymize visitors
													- JavaScript used for browser fingerprinting
													- Some sites embed tracking pixels
													Fix: No JavaScript, use Tor Browser properly
													
													Law enforcement honeypots:
													- Fake markets to catch criminals
													- If you're only browsing, less risk
													- Never transact, never post personal info
													```
													
													---
													
													## 14. Staying Safe — Complete Checklist
													
													### Before Exploring
													
													```
													ENVIRONMENT:
													□ Using Tails OS, Whonix, or at minimum VPN+Tor Browser
													□ Tor Browser security level set to SAFEST
													□ JavaScript completely disabled
													□ VPN connected (if using regular OS)
													□ No personal accounts logged in anywhere
													□ Screenshot tool ready for documentation
													
													MINDSET:
													□ Clear purpose for research (what am I looking for?)
													□ Know what's illegal to access/download
													□ Have plan to report illegal content if found
													□ Not going to buy, sell, or transact anything
													```
													
													### While Exploring
													
													```
													□ Read URLs carefully before clicking
													□ Don't enter any personal information anywhere
													□ Don't download files (extreme danger zone)
													□ Take screenshots for documentation
													□ Note interesting .onion addresses in notes (NOT in browser)
													□ If you find CSAM: close immediately, report to NCMEC
													□ If you find planned attacks: report to FBI IC3 or local law enforcement
													□ Keep notes of your research (for legitimate purpose defense)
													```
													
													### After Exploring
													
													```
													□ Close all Tor Browser windows
													□ If Tails: shutdown (everything wiped)
													□ If regular OS: clear browser history
													□ Secure delete any downloaded files
													□ Review your notes - anything to report?
													□ Log your research activities (date, what you found, why)
													```
													
													### Reporting Illegal Content
													
													```
													If you encounter:
													
													Child sexual abuse material (CSAM):
													→ Report immediately to:
													USA: cybertipline.org (NCMEC)
													UK: report.iwf.org.uk
													International: inhope.org
													FBI: tips.fbi.gov
													
													Planned terrorist attack or violence:
													→ FBI: tips.fbi.gov
													→ Local police emergency line
													→ If imminent: call 911/999/112
													
													Drugs/weapons markets:
													→ DEA: dea.gov/contact-dea
													→ FBI: tips.fbi.gov
													→ Europol: europol.europa.eu/report-crime
													
													Cybercrime in progress:
													→ FBI IC3: ic3.gov
													→ CISA: cisa.gov/report
													→ Your local cybercrime unit
													```
													
													---
													
													## Quick Reference — Legitimate .onion Sites
													
													```
													SEARCH:
													DuckDuckGo: duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion
													Ahmia:      juhanurmihxlp77nkq76byazcldy2hlmovfu2epvl5ankdibsot4csyd.onion
													
													NEWS:
													BBC:        bbcnewsd73hkzno2ini43t4gblxvycyac5aw4gnv7t2rccijh7745uqd.onion
													NYT:        nytimesn7cgmftshazwhfgzm37qxb44r64ytbb2dj3x62d2lljsciiyd.onion
													
													EMAIL:
													ProtonMail: protonmailrmez3lotccipshtkleegetolb73fuirgj7r4o4vfu7ozyd.onion
													
													SOCIAL:
													Facebook:   facebookwkhpilnemxj7asber7cybq2ibopvx63clq4nfldfuer6znpxid.onion
													
													VERIFY TOR:
													check.torproject.org
													
													WHISTLEBLOWING:
													SecureDrop: secrdrop5wyphb5x.onion
													
													PRIVACY:
													Tor Project: 2gzyxa5ihm7nsggfxnu52rck2vv4rvmdlkiu3zzui5du4xyclen53wid.onion
													```
													
													---
													
													## Summary — The Dark Web in Perspective
													
													```
													SIZE:
													~60,000-100,000 active .onion sites
													Most are small, many offline frequently
													Regular web has billions of pages
													
													CONTENT BREAKDOWN (estimates):
													~50% - Legitimate privacy services, forums
													~30% - Scams, abandoned sites
													~15% - Gray area (piracy, etc.)
													~5%  - Genuinely illegal markets
													
													THE TECHNOLOGY ITSELF IS NEUTRAL:
													Tor was created by US Navy for secure communications
													Now funded by US State Department partly
													Used by journalists, activists, military, businesses
													The tool is not the problem — misuse is
													
													YOUR LEGAL POSITION:
													Using Tor: Legal in most countries
													Visiting .onion sites: Legal in most countries
													Downloading illegal content: Illegal everywhere
													Purchasing illegal goods: Illegal everywhere
													Research and documentation: Legal and valuable
													
													WHAT MAKES A GOOD RESEARCHER:
													Clear documented purpose
													No illegal transactions
													Report crimes found
													Publish findings responsibly
													Keep records of your research
													```
													
													---
													
													*The dark web is a fascinating area of the internet that reveals both*
													*the importance of privacy tools and the challenges of online safety.*
													*Approach it with curiosity, caution, and clear ethical boundaries.*
