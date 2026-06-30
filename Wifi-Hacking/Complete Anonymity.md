# 🕵️ Complete Anonymity & OPSEC Guide
### Stay Anonymous, Untraceable, and Secure as a Security Researcher

> **Why this matters:** Every action you take online leaves traces. IP addresses, MAC addresses, DNS queries, browser fingerprints, metadata in files, timestamps, account linkages — all of these can identify you. This guide covers how to eliminate every trace.

---

## 📚 Table of Contents

1. [Understanding What Exposes You](#1-understanding-what-exposes-you)
2. [Network Anonymity — Hide Your IP](#2-network-anonymity)
3. [Tor — The Onion Router](#3-tor)
4. [VPN — Virtual Private Network](#4-vpn)
5. [Tor + VPN Combined](#5-tor--vpn-combined)
6. [MAC Address Anonymity](#6-mac-address-anonymity)
7. [DNS Anonymity](#7-dns-anonymity)
8. [Operating System Anonymity](#8-operating-system-anonymity)
9. [Tails OS — The Gold Standard](#9-tails-os)
10. [Whonix — Advanced Anonymity](#10-whonix)
11. [Browser Anonymity and Fingerprinting](#11-browser-anonymity)
12. [Operational Security (OPSEC)](#12-operational-security-opsec)
13. [Secure Communications](#13-secure-communications)
14. [File and Metadata Anonymity](#14-file-and-metadata-anonymity)
15. [Physical Security](#15-physical-security)
16. [Complete Anonymity Checklist](#16-complete-anonymity-checklist)

---

## 1. Understanding What Exposes You

Before hiding, you must know what reveals your identity.

### The Full Exposure Map

```
WHO ARE YOU? Attackers/Investigators piece together:

Layer 1: NETWORK IDENTITY
├── Real IP address          → Reveals city, ISP, sometimes street
├── MAC address              → Reveals device manufacturer
├── DNS queries              → What sites you visit (even with VPN!)
└── WiFi probe requests      → Your device looking for saved networks

Layer 2: DEVICE IDENTITY
├── Browser fingerprint      → Unique combo of settings identifies you
├── OS fingerprint           → nmap can detect your OS remotely
├── Timezone                 → Reveals your country/region
├── Screen resolution        → Part of browser fingerprint
└── Installed fonts/plugins  → Further fingerprinting

Layer 3: BEHAVIORAL IDENTITY
├── Writing style            → Stylometry (AI can identify you)
├── Typing patterns          → Timing between keystrokes
├── Active hours             → When you're online = timezone
└── Habits                   → Same usernames, same mistakes

Layer 4: ACCOUNT LINKAGE
├── Same username everywhere → Links all your accounts
├── Same email               → Ties real identity to accounts
├── Password reuse           → One breach exposes all
└── Payment methods          → Credit card = real identity

Layer 5: METADATA
├── Photo EXIF data          → GPS coordinates, camera model, time
├── Document metadata        → Author name, PC name, edits history
├── File creation times      → Reveals timezone, activity patterns
└── Communication metadata   → Who you talk to, when, how often
```

### How People Get Caught (Real Examples)

```
Case 1: IP Address Logged
- Used home IP to access target
- Target's server logged your IP
- Law enforcement subpoenaed server → got your ISP
- ISP gave your address
Fix: Always use Tor or VPN

Case 2: Forgot to Hide One Thing
- Used VPN for everything
- But DNS queries leaked (DNS leak)
- DNS server logs showed real queries
Fix: DNS leak protection + test regularly

Case 3: OPSEC Mistake
- Used anonymous account for attacks
- Mentioned same account on personal forum
- Cross-referenced → identified
Fix: Total separation of identities

Case 4: Metadata in Files
- Sent a document as "evidence"
- Document had author name in metadata
- Revealed real identity
Fix: Strip all metadata before sharing

Case 5: Same Writing Style
- Anonymous posts identified by word choices
- Unique phrases matched to known person
Fix: Use different writing style, or AI rewriter
```

---

## 2. Network Anonymity — Hide Your IP

Your IP address is your home address on the internet. Every connection you make reveals it.

### What Your IP Reveals

```bash
# Anyone can look up your IP at:
# https://whatismyipaddress.com
# https://ipinfo.io

# Your IP reveals:
# - Country
# - City (usually accurate to 10-50km)
# - ISP name
# - Organization name
# - Sometimes: Street address (with ISP cooperation)
# - Your entire browsing history (ISP sees all DNS queries)
```

### IP Leak Test — Check Your Real Exposure

```bash
# Test what websites see about you
curl https://ifconfig.me           # Your public IP
curl https://ipinfo.io/json        # Full info about your IP

# DNS leak test
curl https://ipleak.net/json       # Tests DNS AND IP leaks

# Check from command line
dig +short myip.opendns.com @resolver1.opendns.com

# WebRTC leak test (browser-based)
# Visit: https://browserleaks.com/webrtc
# WebRTC can reveal real IP even through VPN!
```

### Proxy Chains — Route Through Multiple Servers

```bash
# Install proxychains
sudo apt install proxychains4

# Configure: /etc/proxychains4.conf
sudo nano /etc/proxychains4.conf

# Configuration:
# Comment out "strict_chain"
# Uncomment "dynamic_chain"  ← skips dead proxies
# Or use "random_chain"      ← randomizes order

# Add proxy list at bottom:
[ProxyList]
# Format: type  host    port  [user  pass]
socks5  127.0.0.1  9050       # Tor (local)
socks5  proxy1.example.com  1080
socks5  proxy2.example.com  1080
http    proxy3.example.com  8080

# Use proxychains with any tool
proxychains nmap -sT -Pn target.com
proxychains curl https://example.com
proxychains firefox
proxychains msfconsole

# Chain of proxies:
# You → Proxy1 → Proxy2 → Proxy3 → Target
# Target only sees Proxy3's IP
# Proxy3 only knows Proxy2
# Extremely hard to trace back to you
```

### Free Proxy Sources (for Research)

```python
# proxy_scraper.py — collect free proxies
import requests
from concurrent.futures import ThreadPoolExecutor

def get_free_proxies():
"""Fetch free proxy list"""
sources = [
	'https://www.proxy-list.download/api/v1/get?type=socks5',
'https://api.proxyscrape.com/v2/?request=getproxies&protocol=socks5',
]

proxies = set()
for url in sources:
	try:
	resp = requests.get(url, timeout=10)
	for line in resp.text.strip().split('\n'):
		line = line.strip()
		if ':' in line:
			proxies.add(line)
			except:
			pass
			
			return list(proxies)
			
			def test_proxy(proxy):
			"""Test if proxy is working"""
			try:
			resp = requests.get(
				'https://httpbin.org/ip',
				proxies={'https': f'socks5://{proxy}'},
				timeout=10
			)
			if resp.status_code == 200:
				ip = resp.json()['origin']
				return proxy, ip
				except:
				pass
				return None
				
				print("[*] Fetching proxies...")
				proxies = get_free_proxies()
				print(f"[*] Testing {len(proxies)} proxies...")
				
				working = []
				with ThreadPoolExecutor(max_workers=50) as ex:
				results = list(ex.map(test_proxy, proxies[:200]))
				
				working = [r for r in results if r]
				print(f"\n[+] Working proxies: {len(working)}")
				for proxy, ip in working[:20]:
					print(f"  {proxy} → shows as {ip}")
					```
					
					---
					
					## 3. Tor — The Onion Router
					
					Tor is the gold standard for anonymity. Your traffic is encrypted and bounced through 3 random servers worldwide.
					
					### How Tor Works
					
					```
					Without Tor:
					You → ISP → Website
					Website sees your real IP
					ISP sees what site you visit
					
					With Tor:
					You → [Encrypted] → Guard Node
					→ [Encrypted] → Middle Node
					→ [Encrypted] → Exit Node
					→ Website
					
					Website sees: Exit Node's IP (random server somewhere)
					Guard Node sees: Your IP, but NOT your destination
					Middle Node sees: Nothing useful
					Exit Node sees: Destination, but NOT your IP
					ISP sees: You're using Tor, but NOT what site
					
					Three layers of encryption = "onion"
					Each node peels one layer
					No single node knows both who you are AND where you're going
					```
					
					### Install and Use Tor
					
					```bash
					# Install Tor
					sudo apt install tor
					
					# Start Tor service
					sudo systemctl start tor
					sudo systemctl enable tor  # Start on boot
					
					# Verify Tor is running
					sudo systemctl status tor
					netstat -tlnp | grep 9050  # Tor listens on 9050
					
					# Test Tor connection
					torify curl https://check.torproject.org/api/ip
					# Should return Tor exit node IP, not yours
					
					# Use Tor with any application
					torify wget https://example.com
					torify nmap -sT -Pn target.com
					torify python3 script.py
					
					# Use with proxychains
					# Add to /etc/proxychains4.conf:
					socks5 127.0.0.1 9050
					
					proxychains curl https://ipinfo.io
					# Shows Tor exit node location
					```
					
					### Tor Browser — Complete Anonymity for Web
					
					```bash
					# Download Tor Browser
					sudo apt install torbrowser-launcher
					torbrowser-launcher
					
					# Or download directly:
					# https://www.torproject.org/download/
					
					# Tor Browser includes:
					# - Pre-configured Firefox with Tor
					# - NoScript (blocks JS by default)
					# - Anti-fingerprinting
					# - No cookies between sessions
					# - Prevents WebRTC leaks
					
					# Security levels:
					# Standard    → Normal browsing, some JS allowed
					# Safer       → Disable JS on non-HTTPS
					# Safest      → No JS at all (safest, most anonymous)
					# Set: Shield icon → Advanced Security Settings → Safest
					```
					
					### Tor Configuration
					
					```bash
					# Edit Tor configuration
					sudo nano /etc/tor/torrc
					
					# Useful settings:
					# ===============
					
					# Force exit through specific country (e.g., Germany)
					ExitNodes {de}
					StrictNodes 1
					
					# Avoid exit through specific countries
					ExcludeExitNodes {us},{gb},{au},{ca},{nz}
					# (Five Eyes countries — intelligence sharing alliance)
					
					# Use bridge (hide that you're using Tor from ISP)
					UseBridges 1
					Bridge obfs4 IP:PORT fingerprint
					
					# Control port (for automation)
					ControlPort 9051
					HashedControlPassword [generate with tor --hash-password yourpassword]
					
					# Reload Tor after config changes
					sudo systemctl reload tor
					
					# Get new identity (new circuit = new IP)
					sudo killall -HUP tor
					# or via control port:
					echo 'AUTHENTICATE "password"\r\nSIGNAL NEWNYM\r\nQUIT' | nc 127.0.0.1 9051
					```
					
					### Control Tor with Python
					
					```python
					# tor_controller.py — Automate Tor circuits
					from stem import Signal
					from stem.control import Controller
					import requests
					
					def get_tor_session():
					"""Create requests session through Tor"""
					session = requests.Session()
					session.proxies = {
						'http':  'socks5h://127.0.0.1:9050',
						'https': 'socks5h://127.0.0.1:9050'
					}
					return session
					
					def get_current_ip(session):
					"""Get current Tor IP"""
					try:
					resp = session.get('https://httpbin.org/ip', timeout=15)
					return resp.json()['origin']
					except:
					return None
					
					def change_identity():
					"""Request new Tor circuit (new IP)"""
					with Controller.from_port(port=9051) as ctrl:
					ctrl.authenticate()  # No password if not set
					ctrl.signal(Signal.NEWNYM)
					
					# Demo: Browse through Tor, change IP multiple times
					session = get_tor_session()
					
					for i in range(5):
						ip = get_current_ip(session)
						print(f"[Round {i+1}] Current IP: {ip}")
						
						# Change to new identity
						change_identity()
						import time; time.sleep(5)  # Wait for new circuit
						
						print("\n[+] Each request appeared to come from different location!")
						```
						
						### Tor Limitations
						
						```
						Tor does NOT protect against:
						✗ Exit node sniffing (if using HTTP not HTTPS)
						→ Use HTTPS always with Tor
						✗ Browser fingerprinting (without Tor Browser)
						✗ Malware on your machine
						✗ JavaScript exploits
						✗ Mistakes (logging into personal accounts)
						✗ Timing attacks (rare, needs large resources)
						✗ Compromised entry/exit nodes (rare)
						
						Tor IS slow:
						Traffic bounces through 3 countries
						Expect 10x slower than normal internet
						Not good for large downloads
						Not good for video streaming
						```
						
						---
						
						## 4. VPN — Virtual Private Network
						
						A VPN creates an encrypted tunnel from your device to the VPN server. The target sees the VPN's IP, not yours.
						
						### How VPN Works
						
						```
						Without VPN:
						You (1.2.3.4) → ISP → Target
						Target sees: 1.2.3.4 (your real IP)
						ISP sees: You visiting Target
						
						With VPN:
						You → [Encrypted Tunnel] → VPN Server (5.6.7.8) → Target
						Target sees: 5.6.7.8 (VPN server IP)
						ISP sees: You connected to VPN server (can't see what)
						
						VPN vs Tor:
						VPN: One hop, faster, but VPN provider knows your traffic
						Tor: Three hops, slower, but no single entity knows everything
						```
						
						### Choosing a VPN for Security Research
						
						```
						Requirements for serious anonymity:
						✓ No-logs policy (verified by audit)
						✓ Accepts anonymous payment (crypto, cash)
						✓ Kill switch (cuts internet if VPN drops)
						✓ DNS leak protection
						✓ Based outside Five Eyes countries
						✓ RAM-only servers (logs can't survive reboot)
						
						Recommended (no affiliation, research-based):
						- Mullvad VPN     → Accepts cash/crypto, no account email needed
						- ProtonVPN       → Swiss law, open source, free tier available
						- IVPN            → Strong privacy, accepts Monero
						
						Avoid:
						- Free VPNs       → You are the product, they log everything
						- VPNs that ask for real email to sign up
						- VPNs based in US/UK/Australia/Canada/NZ
						```
						
						### Set Up OpenVPN Manually
						
						```bash
						# Install OpenVPN
						sudo apt install openvpn
						
						# Download config from your VPN provider
						# Usually a .ovpn file
						
						# Connect
						sudo openvpn --config vpn_server.ovpn
						
						# Or as a service
						sudo cp vpn_server.ovpn /etc/openvpn/client.conf
						sudo systemctl start openvpn@client
						sudo systemctl enable openvpn@client
						
						# Verify VPN is working
						curl https://ifconfig.me
						# Should show VPN server IP, not your real IP
						
						# Check for DNS leaks
						curl https://ipleak.net/json | python3 -m json.tool
						# "ip" field should be VPN IP
						# "dns_servers" should be VPN's DNS, not your ISP's
						```
						
						### Kill Switch — Never Expose Real IP
						
						```bash
						# Kill switch: if VPN drops, block ALL internet
						# Prevents accidental IP exposure
						
						# Using iptables:
						
						# Allow traffic only through VPN interface (tun0)
						sudo iptables -F                     # Flush existing rules
						sudo iptables -P INPUT DROP          # Default: drop all incoming
						sudo iptables -P OUTPUT DROP         # Default: drop all outgoing
						sudo iptables -P FORWARD DROP        # Default: drop all forwarded
						
						# Allow loopback
						sudo iptables -A INPUT -i lo -j ACCEPT
						sudo iptables -A OUTPUT -o lo -j ACCEPT
						
						# Allow VPN connection itself (to your VPN server)
						sudo iptables -A OUTPUT -p udp --dport 1194 -j ACCEPT  # OpenVPN port
						sudo iptables -A INPUT -p udp --sport 1194 -j ACCEPT
						
						# Allow all traffic through VPN tunnel
						sudo iptables -A INPUT -i tun0 -j ACCEPT
						sudo iptables -A OUTPUT -o tun0 -j ACCEPT
						
						# Now if VPN drops, tun0 disappears
						# All traffic is blocked → no IP leak!
						
						# Save rules
						sudo iptables-save > /etc/iptables/rules.v4
						
						# Restore on boot
						sudo apt install iptables-persistent
						```
						
						### DNS Leak Prevention
						
						```bash
						# DNS leaks happen when DNS queries bypass VPN
						# Your ISP sees what sites you're visiting!
						
						# Check for DNS leaks:
						# Visit: https://dnsleaktest.com
						# or:
						dig +short whoami.akamai.net       # Should show VPN server location
						
						# Fix DNS leaks:
						
						# Method 1: Use VPN's DNS servers
						# Edit: /etc/resolv.conf
						sudo nano /etc/resolv.conf
						# Add VPN provider's DNS:
						nameserver 10.8.0.1    # VPN gateway (usually)
						
						# Method 2: Use encrypted DNS
						# DNS over HTTPS (DoH) or DNS over TLS (DoT)
						
						sudo apt install dnscrypt-proxy
						
						sudo nano /etc/dnscrypt-proxy/dnscrypt-proxy.toml
						# Set: listen_addresses = ['127.0.0.1:53']
						# Set: server_names = ['cloudflare', 'google', 'mullvad-doh']
						
						sudo systemctl start dnscrypt-proxy
						sudo systemctl enable dnscrypt-proxy
						
						# Update resolv.conf
						echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
						
						# Method 3: Prevent leaks via iptables
						# Only allow DNS through VPN
						sudo iptables -A OUTPUT -p udp --dport 53 ! -o tun0 -j DROP
						sudo iptables -A OUTPUT -p tcp --dport 53 ! -o tun0 -j DROP
						```
						
						---
						
						## 5. Tor + VPN Combined
						
						Maximum anonymity: combine both.
						
						### Configuration 1: VPN → Tor (Recommended)
						
						```
						You → VPN → Tor → Target
						
						Benefits:
						✓ ISP sees VPN traffic (not Tor — some ISPs block/flag Tor)
						✓ VPN provider sees you use Tor, not what you do
						✓ Target sees Tor exit node
						✓ Even if VPN logs, they see "Tor traffic" only
						
						How to set up:
						1. Connect to VPN first
						2. Then start Tor
						3. All Tor traffic goes through VPN
						```
						
						```bash
						# Step 1: Connect to VPN
						sudo openvpn --config vpn.ovpn &
						sleep 10
						curl ifconfig.me  # Should show VPN IP
						
						# Step 2: Start Tor (now routes through VPN)
						sudo systemctl start tor
						
						# Step 3: Route traffic through Tor
						proxychains curl https://check.torproject.org/api/ip
						# Shows Tor exit node IP
						# Traffic path: You → VPN → Tor → Target
						```
						
						### Configuration 2: Tor → VPN
						
						```
						You → Tor → VPN → Target
						
						Benefits:
						✓ VPN provider cannot see your real IP (they see Tor exit)
						✓ More resistant to VPN provider logging
						
						Harder to set up (requires Whonix or special config)
						```
						
						### Using Tor Bridges (Hide Tor Use From ISP)
						
						```bash
						# Some ISPs/countries block Tor
						# Bridges = unlisted Tor relays that aren't in public directory
						
						# Get bridges from: https://bridges.torproject.org
						# Or via email: bridges@torproject.org
						# Or in Tor Browser: "Get Bridges"
						
						# Add to torrc:
						sudo nano /etc/tor/torrc
						
						UseBridges 1
						ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy
						Bridge obfs4 IP:PORT FINGERPRINT
						
						# obfs4 makes Tor traffic look like random HTTPS traffic
						# Very hard for ISPs to detect and block
						```
						
						---
						
						## 6. MAC Address Anonymity
						
						Your MAC address identifies your exact network card. Routers log it. Networks remember it.
						
						### Change MAC Before Every Session
						
						```bash
						# Install macchanger
						sudo apt install macchanger
						
						# Check current MAC
						ip link show wlan0 | grep ether
						
						# Change to completely random MAC
						sudo ip link set wlan0 down
						sudo macchanger -r wlan0
						sudo ip link set wlan0 up
						
						# Output:
						# Current MAC: aa:bb:cc:dd:ee:ff (YourDevice Inc.)
						# New MAC:     12:34:56:78:9a:bc (Unknown)
						
						# Change to specific vendor (look less suspicious)
						# Example: appear to be an Apple device
						sudo macchanger -m 00:11:22:33:44:55 wlan0
						
						# Apple MACs start with:
						# 00:11:22  AC:DE:48  A8:5C:2C  F4:F1:5A etc.
						
						# Samsung MACs start with:
						# 00:00:F0  00:12:47  00:15:99  00:17:D5 etc.
						
						# Randomize only last half (less suspicious)
						sudo macchanger -e wlan0
						
						# Check it changed
						macchanger -s wlan0
						
						# Make it persistent on NetworkManager:
						sudo nano /etc/NetworkManager/NetworkManager.conf
						# Add under [device]:
						# [device]
						# wifi.scan-rand-mac-address=yes
						# [connection]
						# wifi.cloned-mac-address=random
						# ethernet.cloned-mac-address=random
						```
						
						### Randomize MAC on Every Boot
						
						```bash
						# Create systemd service to randomize MAC on boot
						cat > /etc/systemd/system/mac-randomize.service << EOF
						[Unit]
						Description=Randomize MAC Address
						Before=network.target
						
						[Service]
						Type=oneshot
						ExecStart=/bin/bash -c 'ip link set wlan0 down && macchanger -r wlan0 && ip link set wlan0 up'
						ExecStart=/bin/bash -c 'ip link set eth0 down && macchanger -r eth0 && ip link set eth0 up'
						RemainAfterExit=yes
						
						[Install]
						WantedBy=multi-user.target
						EOF
						
						sudo systemctl enable mac-randomize
						sudo systemctl start mac-randomize
						```
						
						---
						
						## 7. DNS Anonymity
						
						DNS is the biggest anonymity leak most people ignore.
						
						### The Problem
						
						```
						Normal flow:
						You type: google.com
						Your computer asks DNS server: "What IP is google.com?"
						Default DNS server = your ISP's servers
						ISP logs: [timestamp] [your IP] queried google.com
						ISP can sell this data, give to government, or get hacked
						
						Even with VPN:
						If DNS queries bypass VPN (DNS leak)
						→ ISP still sees everything you visit!
						```
						
						### Solutions
						
						```bash
						# Solution 1: DNS over HTTPS (DoH)
						# Encrypts DNS queries, sends via HTTPS
						# ISP sees HTTPS traffic, not the actual queries
						
						# Install dnscrypt-proxy
						sudo apt install dnscrypt-proxy
						
						# Configure
						sudo nano /etc/dnscrypt-proxy/dnscrypt-proxy.toml
						
						# Set server list (choose privacy-focused ones):
						server_names = ['mullvad-doh', 'cloudflare', 'quad9-doh-ip4-port443-nofilter-pri']
						
						# Enable DoH
						doh_servers = true
						
						# No logging
						require_nolog = true
						
						# Start service
						sudo systemctl start dnscrypt-proxy
						sudo systemctl enable dnscrypt-proxy
						
						# Point DNS to local proxy
						echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
						
						# Verify
						dig google.com  # Should work through encrypted DNS
						
						# Solution 2: Use Tor's DNS
						# Add to /etc/tor/torrc:
						DNSPort 9053
						AutomapHostsOnResolve 1
						
						# Use Tor for DNS:
						echo "nameserver 127.0.0.1" > /etc/resolv.conf
						sudo iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-port 9053
						
						# Solution 3: DNS over Tor (most anonymous)
						# All DNS goes through Tor network
						# Nobody can see what you're looking up
						```
						
						---
						
						## 8. Operating System Anonymity
						
						Your OS itself can leak information. Configuration matters.
						
						### Harden Parrot OS / Kali
						
						```bash
						# 1. Disable unnecessary services that might phone home
						sudo systemctl disable avahi-daemon    # mDNS (broadcasts your name)
						sudo systemctl disable cups           # Printing service
						sudo systemctl disable bluetooth      # Bluetooth (if not needed)
						sudo systemctl disable geoclue        # Location service
						
						# 2. Disable IPv6 (harder to anonymize, often leaks)
						echo "net.ipv6.conf.all.disable_ipv6 = 1
						net.ipv6.conf.default.disable_ipv6 = 1
						net.ipv6.conf.lo.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
						sudo sysctl -p
						
						# Verify IPv6 disabled
						ip addr | grep inet6
						# Should be empty
						
						# 3. Enable firewall
						sudo ufw enable
						sudo ufw default deny incoming
						sudo ufw default deny outgoing   # Block everything by default
						sudo ufw allow out on tun0       # Allow through VPN only
						sudo ufw allow out 1194/udp      # Allow VPN connection itself
						
						# 4. Randomize hostname on each boot
						sudo nano /etc/hostname
						# Change to something generic: "ubuntu" or "localhost"
						
						# 5. Check for processes that call home
						sudo netstat -tlnp   # See what's listening
						sudo ss -tlnp        # Modern version
						
						# 6. Review cron jobs
						crontab -l
						sudo crontab -l
						ls /etc/cron*
						
						# 7. Clear bash history
						unset HISTFILE       # Don't save history this session
						echo "HISTFILE=/dev/null" >> ~/.bashrc  # Never save history
						history -c           # Clear current history
						rm ~/.bash_history   # Delete history file
						```
						
						### Kernel Hardening
						
						```bash
						# Add to /etc/sysctl.conf for security hardening
						
						sudo nano /etc/sysctl.conf
						
						# Networking hardening
						net.ipv4.conf.all.rp_filter = 1           # Reverse path filtering
						net.ipv4.conf.default.rp_filter = 1
						net.ipv4.conf.all.accept_source_route = 0  # No source routing
						net.ipv4.conf.all.accept_redirects = 0    # No ICMP redirects
						net.ipv4.conf.all.send_redirects = 0
						net.ipv4.conf.all.log_martians = 1        # Log suspicious packets
						net.ipv4.icmp_echo_ignore_broadcasts = 1  # Ignore broadcast pings
						net.ipv4.tcp_syncookies = 1               # SYN flood protection
						
						# Privacy
						kernel.randomize_va_space = 2             # ASLR
						kernel.dmesg_restrict = 1                # Restrict kernel logs
						kernel.kptr_restrict = 2                 # Hide kernel pointers
						kernel.yama.ptrace_scope = 1             # Restrict ptrace
						
						sudo sysctl -p
						```
						
						---
						
						## 9. Tails OS — The Gold Standard
						
						Tails is a live OS (runs from USB) designed entirely for anonymity. It leaves NO trace on the computer.
						
						### Why Tails is Special
						
						```
						Properties of Tails:
						✓ Runs entirely from RAM
						✓ Nothing written to hard drive
						✓ ALL traffic routed through Tor automatically
						✓ When you remove USB → everything disappears
						✓ MAC address randomized on boot
						✓ Identical to every other Tails user (no fingerprint)
						✓ Amnesic (no memory between sessions by default)
						✓ Pre-installed with security tools
						
						Used by:
						- Edward Snowden (confirmed)
						- Journalists protecting sources
						- Activists in dangerous countries
						- Security researchers worldwide
						```
						
						### Setting Up Tails
						
						```bash
						# Step 1: Download Tails
						# https://tails.boum.org/install/
						# Verify the download with its signature (important!)
						
						# Step 2: Write to USB (8GB+ recommended)
						# On Linux:
						sudo dd if=tails-amd64-X.X.img of=/dev/sdX bs=16M oflag=direct status=progress
						# Replace /dev/sdX with your USB drive (check with lsblk first!)
						
						# Step 3: Boot from USB
						# Restart computer
						# Press F12/F2/Del for boot menu
						# Select USB drive
						
						# Step 4: Tails boots with:
						# - Random MAC address
						# - Tor connection (wait 1-2 minutes)
						# - Anonymous desktop environment
						# - All traffic through Tor
						
						# Optional: Set up Persistent Storage
						# (encrypted partition for saving files between sessions)
						# Applications → Tails → Persistent Storage
						# Choose what to save: bookmarks, files, tools, etc.
						```
						
						### Tails for Security Research
						
						```
						Pre-installed tools in Tails:
						- Tor Browser          (anonymous browsing)
						- OnionShare           (share files over Tor)
						- Thunderbird + Enigmail (encrypted email)
						- KeePassXC            (password manager)
						- Electrum Bitcoin     (anonymous payments)
						- MAT2                 (metadata cleaner)
						- Aircrack-ng          (WiFi security)
						- Wireshark            (network analysis)
						- GnuPG                (encryption)
						- Kleopatra            (certificate manager)
						
						What Tails CAN'T do:
						- Long-running processes (boots fresh each time)
						- Install permanent software (unless in persistent storage)
						- Work well for complex technical attacks (Kali is better for this)
						- Protect you if you make mistakes (log into real accounts, etc.)
						```
						
						---
						
						## 10. Whonix — Advanced Anonymity
						
						Whonix is two virtual machines that work together — a Gateway (runs Tor) and a Workstation (all traffic forced through Gateway).
						
						### Whonix Architecture
						
						```
						┌─────────────────────────────────────────────────────┐
						│                    Your Computer                     │
						│                                                      │
						│  ┌──────────────────┐      ┌──────────────────────┐ │
						│  │  Whonix Gateway  │      │  Whonix Workstation  │ │
						│  │                  │      │                      │ │
						│  │  Runs Tor        │◄────►│  Your work happens   │ │
						│  │  Only Tor exits  │      │  here                │ │
						│  │  here            │      │                      │ │
						│  │  IP: 10.152.152.10      │  IP: 10.152.152.11   │ │
						│  └──────────────────┘      └──────────────────────┘ │
						│           ↓                                          │
						│    Internet (through Tor only)                       │
						└─────────────────────────────────────────────────────┘
						
						Key property:
						Even if the Workstation is compromised by malware,
it CANNOT leak your real IP because the Gateway
only allows Tor traffic to exit!
```

### Installing Whonix

```bash
# Download Whonix for VirtualBox:
# https://www.whonix.org/wiki/VirtualBox

# Two OVA files:
# Whonix-Gateway-*.ova
# Whonix-Workstation-*.ova

# Import both into VirtualBox:
# File → Import Appliance → select each OVA

# Start Gateway FIRST, then Workstation

# In Workstation:
# Everything automatically goes through Tor
# Even if you run malware, your real IP is safe!

# Update Whonix
sudo whonix_repository enable
sudo apt update && sudo apt upgrade

# Install your tools in Workstation
sudo apt install metasploit-framework nmap
```

---

## 11. Browser Anonymity and Fingerprinting

Even with Tor, your browser can identify you through fingerprinting.

### What is Browser Fingerprinting?

```
Websites collect:
- User Agent string (browser + OS version)
- Screen resolution
- Timezone
- Installed fonts (list varies by person)
- Canvas fingerprint (GPU rendering = unique)
- WebGL fingerprint
- Audio fingerprint
- Installed plugins
- Battery status (old)
- CPU cores
- Memory size

Combined = unique fingerprint
99%+ accurate identification
Works ACROSS different sessions
Even if you change IP!

Test your fingerprint:
https://browserleaks.com
https://coveryourtracks.eff.org
https://amiunique.org
```

### Defeating Fingerprinting

```bash
# Option 1: Tor Browser (best)
# All Tor Browser users have identical fingerprints
# You blend into the crowd
torbrowser-launcher

# Option 2: Firefox with anti-fingerprint config
# Install Firefox
sudo apt install firefox

# Install extensions:
# - uBlock Origin (block trackers)
# - Privacy Badger (block tracking)
# - Canvas Blocker (block canvas fingerprint)
# - No Script (block JavaScript)

# Firefox about:config changes:
# (Type about:config in address bar)

privacy.resistFingerprinting = true    # Best anti-fingerprint setting
privacy.firstparty.isolate = true      # Isolate cookies per site
network.dns.disablePrefetch = true     # No DNS prefetching
browser.send_pings = false             # No hyperlink auditing
geo.enabled = false                    # No geolocation
media.navigator.enabled = false        # No WebRTC device enum
network.http.sendRefererHeader = 0     # No referrer headers
network.http.sendSecureXSiteReferrer = false
dom.battery.enabled = false            # No battery status
webgl.disabled = true                  # No WebGL fingerprint
```

### Browser OPSEC Rules

```
ALWAYS:
✓ Use Tor Browser for sensitive browsing
✓ Use HTTPS everywhere
✓ Clear cookies between sessions
✓ Use private/incognito mode as minimum

NEVER:
✗ Log into personal accounts while anonymous
✗ Use the same browser profile for anonymous + personal
✗ Enable JavaScript when it's not needed
✗ Install random extensions (they can fingerprint you!)
✗ Open files downloaded via Tor (can bypass Tor!)
✗ Resize Tor Browser window (screen size = fingerprint)
✗ Use Google or Bing (use DuckDuckGo onion instead)
```

---

## 12. Operational Security (OPSEC)

OPSEC is about behavior — the hardest part of anonymity.

### The Five OPSEC Steps

```
1. IDENTIFY CRITICAL INFORMATION
What information, if exposed, would identify you?
- Real name, location, IP, accounts, writing style

2. ANALYZE THREATS
Who might want to identify you?
- ISP, adversary, law enforcement, the target

3. ANALYZE VULNERABILITIES
What could expose your critical information?
- Logged IP, metadata in files, account linkage

4. ASSESS RISK
How likely is each vulnerability to be exploited?

5. APPLY COUNTERMEASURES
Take action to eliminate each vulnerability
```

### Separate Identities — The Golden Rule

```
Your Real Life Identity:
Name, face, address, real email
Real social media
Personal devices, home network
↕ MUST NEVER MIX ↕

Research/Anonymous Identity:
Pseudonym (different name)
Anonymous email (Protonmail via Tor)
Anonymous accounts (never linked to real)
Separate devices or VMs
Always through VPN/Tor
Different writing style
```

### Creating a Clean Anonymous Identity

```bash
# 1. Create anonymous email FIRST
# Visit via Tor Browser only:
# - ProtonMail: proton.me (create via Tor)
# - Tutanota: tutanota.com
# - cock.li (anonymous, no phone needed)
# NEVER use Gmail, Yahoo, Outlook for anonymous work

# 2. Create accounts using anonymous email
# GitHub, forums, etc.
# Never reuse usernames from real life

# 3. Use different username conventions
# Real life: john_smith_1990
# Anonymous: r3d_t3am_7 (unrelated to real you)

# 4. Generate anonymous usernames
python3 -c "
import random, string
def gen_username():
adj = ['silent', 'dark', 'ghost', 'shadow', 'cipher', 'void']
noun = ['packet', 'node', 'frame', 'probe', 'flux', 'hex']
num = random.randint(10, 99)
return f'{random.choice(adj)}_{random.choice(noun)}_{num}'

for i in range(5):
	print(gen_username())
	"
	```
	
	### Password Management for Anonymous Accounts
	
	```bash
	# Use KeePassXC for all passwords
	sudo apt install keepassxc
	
	# Create a database for anonymous accounts
	# Store in encrypted persistent storage (Tails) or encrypted drive
	
	# Generate strong passwords for every account
	# Never reuse passwords
	# Never use passwords from real life
	
	# Password generator:
	python3 -c "
	import secrets, string
	alphabet = string.ascii_letters + string.digits + '!@#\$%^&*'
	password = ''.join(secrets.choice(alphabet) for i in range(32))
	print(password)
	"
	```
	
	### OPSEC Mistakes That Get People Caught
	
	```
	Mistake 1: Time Zone / Active Hours
	- If you only post between 9pm-2am in a specific timezone
	- Adversaries can narrow down your location
	Fix: Use random delays, post at unusual hours
	
	Mistake 2: Writing Style
	- Unique phrases, spelling errors, punctuation habits
	- Can be identified even across different usernames
	Fix: Use simple, generic language
	Or use AI to rewrite your text before posting
	
	Mistake 3: Reusing Images
	- Post a photo that you also posted on personal account
	- Reverse image search links them
	Fix: Never reuse images across identities
	
	Mistake 4: Technical Fingerprints
	- Same PGP key across identities
	- Same browser/OS fingerprint
	- Same code style in programs you write
	Fix: Different keys per identity, use Tor Browser
	
	Mistake 5: One Slip is Enough
	- Did everything right for months
	- Then forgot VPN once, connected directly
	- That one log entry = identified
	Fix: Kill switch, always verify before connecting
	
	Mistake 6: Logging Into Real Accounts
	- Using Tor, feeling safe
	- Log into personal Gmail "just once"
	- Now Google knows your Tor exit node = you
	Fix: NEVER mix identities, even "just once"
	```
	
	---
	
	## 13. Secure Communications
	
	### Signal — Encrypted Messaging
	
	```
	Signal properties:
	✓ End-to-end encryption (E2EE)
	✓ Open source, audited
	✓ Minimal metadata collection
	✓ Disappearing messages
	✓ No ads, not for profit
	
	Install on phone:
	Android: F-Droid or Play Store
	iOS: App Store
	
	Desktop:
	sudo snap install signal-desktop
	# or from signal.org
	
	Settings to enable:
	→ Note to Self: enable disappearing messages
	→ Privacy → Screen Lock: ON
	→ Privacy → Screen Security: ON (prevents screenshots)
	→ Privacy → Incognito Keyboard: ON
	→ Payments → Hide balance: ON
	```
	
	### PGP Encryption — For Email
	
	```bash
	# Install GnuPG
	sudo apt install gnupg
	
	# Generate anonymous PGP key
	# IMPORTANT: Use anonymous name, use fake email
	gpg --full-generate-key
	# Select: RSA and RSA
	# Key size: 4096
	# Expiration: 1y (change yearly)
	# Name: Anonymous Researcher (or any pseudonym)
	# Email: anon@protonmail.com (your anonymous email)
	
	# List your keys
	gpg --list-keys
	
	# Export public key (share this)
	gpg --armor --export anon@protonmail.com > my_public_key.asc
	
	# Encrypt a message to someone
	gpg --encrypt --armor -r recipient@email.com message.txt
	# Creates: message.txt.asc (only recipient can read)
	
	# Decrypt message sent to you
	gpg --decrypt message.txt.asc
	
	# Sign a message (proves it came from you)
	gpg --sign --armor message.txt
	
	# Sign + encrypt
	gpg --sign --encrypt --armor -r recipient@email.com message.txt
	```
	
	### OnionShare — Anonymous File Transfer
	
	```bash
	# Install OnionShare
	sudo apt install onionshare
	# or pip3 install onionshare-cli
	
	# Share files anonymously via Tor
	onionshare-cli file1.txt file2.pdf
	# Creates: .onion address
	# Only accessible via Tor
	# No server logs your files
	# Files transferred peer-to-peer through Tor
	
	# Receive files anonymously
	onionshare-cli --receive
	# Creates: .onion address others can upload files to
	
	# Chat anonymously
	onionshare-cli --chat
	# Creates temporary .onion chat room
	```
	
	### Encrypted Email with ProtonMail
	
	```
	ProtonMail (proton.me):
	✓ E2E encrypted (ProtonMail can't read your emails)
	✓ Swiss law (strong privacy)
	✓ No IP logs (with Tor use)
	✓ Anonymous account creation (no phone if using Tor)
	
	Setup:
	1. Open Tor Browser
	2. Go to proton.me/signup
	3. Create account without phone number
	4. Use a pseudonym
	5. Access only via Tor Browser always
	
	For maximum security:
	Use ProtonMail Bridge + local email client
	Add PGP layer on top
	```
	
	---
	
	## 14. File and Metadata Anonymity
	
	Files contain hidden information that can identify you.
	
	### What Metadata Reveals
	
	```
	Photo (JPEG) EXIF data contains:
	- GPS coordinates (exact location!)
	- Camera model (identifies your device)
	- Date and time (your timezone, activity time)
	- Camera settings
	- Software used to edit
	
	Word Document (.docx) metadata:
	- Author name (your real name if Office is set up)
	- Company name
	- Last modified by
	- Revision history
	- Document creation time
	- PC name where it was created
	
	PDF metadata:
	- Author
	- Creator application
	- Creation date
	- Modification date
	
	Real case: A document leaked by a whistleblower
	identified them via document metadata.
	```
	
	### Strip All Metadata
	
	```bash
	# Install MAT2 (Metadata Anonymisation Toolkit)
	sudo apt install mat2
	
	# Check metadata of a file
	mat2 --show document.pdf
	mat2 --show photo.jpg
	
	# Strip metadata from single file
	mat2 photo.jpg
	# Creates: photo.cleaned.jpg (no metadata!)
	
	# Strip from all files in directory
	mat2 *
	
	# Verify metadata removed
	mat2 --show photo.cleaned.jpg
	# Should show: no metadata found
	
	# For images specifically:
	# ExifTool is more powerful:
	sudo apt install libimage-exiftool-perl
	
	# Show all metadata
	exiftool photo.jpg
	
	# Remove ALL metadata
	exiftool -all= photo.jpg
	
	# Remove GPS specifically
	exiftool -GPS:all= photo.jpg
	
	# Process entire directory
	exiftool -all= -r ./photos/
	```
	
	### Secure File Deletion
	
	```bash
	# Regular delete doesn't actually erase data!
	# Data stays on disk until overwritten
	
	# Secure delete single file
	sudo apt install secure-delete
	srm -v sensitive_file.txt
	
	# Secure delete directory
	srm -rv sensitive_directory/
	
	# Overwrite with random data multiple times
	shred -vzu -n 10 sensitive_file.txt
	# -v = verbose
	# -z = add final overwrite with zeros (hides shredding)
	# -u = remove after overwriting
	# -n 10 = 10 passes of random data
	
	# Wipe free space (overwrite deleted files)
	sfill -v /tmp/   # Wipe free space on /tmp
	sfill -v ~/      # Wipe free space on home dir
	
	# For SSDs: shred doesn't fully work due to wear leveling
	# Use full disk encryption instead (data never stored plaintext)
	```
	
	### Encrypted Storage
	
	```bash
	# Create an encrypted container (file that acts like encrypted drive)
	sudo apt install cryptsetup
	
	# Create 1GB encrypted container
	dd if=/dev/urandom of=/tmp/secure_container bs=1M count=1024
	sudo cryptsetup luksFormat /tmp/secure_container
	sudo cryptsetup luksOpen /tmp/secure_container secure_vol
	sudo mkfs.ext4 /dev/mapper/secure_vol
	sudo mount /dev/mapper/secure_vol /mnt/secure/
	
	# Use it like a normal drive
	cp sensitive_files/* /mnt/secure/
	
	# Close it (unmount + encrypt)
	sudo umount /mnt/secure
	sudo cryptsetup luksClose secure_vol
	
	# Now the container looks like random data to anyone
	# Can only be opened with your passphrase
	
	# VeraCrypt (easier GUI alternative)
	sudo apt install veracrypt
	veracrypt  # GUI to create/open encrypted containers
	```
	
	---
	
	## 15. Physical Security
	
	Digital anonymity means nothing if someone can physically access your device.
	
	### Disk Encryption
	
	```bash
	# If your laptop is stolen, disk encryption protects everything
	
	# Full disk encryption (set up during OS install)
	# Parrot OS installer: check "Encrypt the new Parrot installation"
	
	# Encrypt existing disk with LUKS
	sudo apt install cryptsetup-bin
	
	# Check if disk is already encrypted
	sudo dmsetup status
	# or
	lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT
	
	# Encrypt a partition (careful! backs up data first!)
	sudo cryptsetup luksFormat /dev/sdb1
	sudo cryptsetup luksOpen /dev/sdb1 encrypted
	sudo mkfs.ext4 /dev/mapper/encrypted
	sudo mount /dev/mapper/encrypted /mnt/
	
	# BIOS/UEFI password
	# Set in BIOS settings (F2/Del during boot)
	# Prevents booting from USB to bypass OS encryption
	
	# VeraCrypt hidden volume
	# Create two encrypted volumes in one container:
	# - Fake password → shows harmless files
	# - Real password → shows actual sensitive files
	# Can't be proven the hidden volume exists!
	veracrypt
	# Tools → Create Volume → Hidden VeraCrypt Volume
	```
	
	### RAM Encryption / Cold Boot Protection
	
	```bash
	# After computer shuts down, RAM can retain data for minutes
	# "Cold boot attack" = freeze RAM, remove it, read encryption keys
	
	# Protection: enable memory scrubbing on shutdown
	sudo apt install cryptsetup
	
	# Add to /etc/default/grub:
	# GRUB_CMDLINE_LINUX="... mem_sleep_default=deep page_poison=on"
	sudo update-grub
	
	# Use sleep/suspend carefully:
	# Full disk encryption protects when OFF
	# Suspend = RAM still accessible!
	# Hibernate = dumps RAM to disk = need encrypted swap too
	
	# Encrypt swap partition
	sudo apt install cryptsetup
	sudo swapoff -a
	# Edit /etc/crypttab:
	# swap /dev/sda2 /dev/urandom swap,cipher=aes-cbc-essiv:sha256
	```
	
	### Physical Awareness
	
	```
	Rules:
	✓ Lock screen when leaving device (even for 30 seconds)
	✓ Use privacy screen filter (prevents shoulder surfing)
	✓ Be aware of cameras (ATMs, stores, streets)
	✓ Use USB data blocker when charging in public
	✓ Never leave device unattended
	✓ Regularly check for physical keyloggers on keyboards
	
	USB Attacks:
	BadUSB = USB that acts as keyboard, types commands automatically
	Rubber Ducky = popular attack USB device
	Defense: disable USB auto-run, use USBGuard
	
	sudo apt install usbguard
	sudo systemctl enable usbguard
	sudo systemctl start usbguard
	# Now unauthorized USB devices are blocked!
	```
	
	---
	
	## 16. Complete Anonymity Checklist
	
	### Before Every Session
	
	```
	NETWORK:
	□ VPN connected and verified (curl ifconfig.me)
	□ DNS leak test passed (dnsleaktest.com)
	□ Tor running (if needed)
	□ Kill switch active
	□ IPv6 disabled
	□ MAC address randomized
	
	DEVICE:
	□ Using anonymous identity (not real accounts)
	□ Correct browser (Tor Browser for sensitive work)
	□ Metadata stripped from any files you'll share
	□ No personal files on current work environment
	
	OPSEC:
	□ Using pseudonym consistently
	□ Writing style is generic (no unique phrases)
	□ No cross-contamination with real identity planned
	```
	
	### During Session
	
	```
	□ Don't log into real accounts (EVER during anonymous session)
	□ Check VPN/Tor is still connected periodically
	□ Use HTTPS only
	□ Don't download and open files (can bypass Tor)
	□ Don't resize Tor Browser window
	□ No personal information in any communication
	```
	
	### After Session
	
	```
	□ Delete downloaded files (or secure delete)
	□ Clear browser history/cache
	□ Disconnect VPN
	□ If Tails: shutdown (RAM wiped automatically)
	□ If normal OS: clear logs, bash history
	
	# Clear all logs
	sudo journalctl --vacuum-time=1s   # Clear systemd logs
	sudo rm /var/log/*.log             # Clear other logs
	history -c && rm ~/.bash_history   # Clear bash history
	
	# Wipe temporary files
	sudo rm -rf /tmp/*
	sudo rm -rf /var/tmp/*
	```
	
	### Tools Summary Table
	
	| Category | Tool | Purpose |
	|---|---|---|
	| **IP Hiding** | Tor | Route through 3 nodes worldwide |
	| **IP Hiding** | VPN (Mullvad/ProtonVPN) | Encrypt + hide from ISP |
	| **IP Hiding** | Proxychains | Chain multiple proxies |
	| **OS** | Tails OS | Amnesic, leaves no trace |
	| **OS** | Whonix | VM pair, Tor-forced |
	| **DNS** | dnscrypt-proxy | Encrypted DNS queries |
	| **MAC** | macchanger | Randomize hardware address |
	| **Browser** | Tor Browser | Anonymous web browsing |
	| **Comms** | Signal | E2E encrypted messaging |
	| **Comms** | ProtonMail | Encrypted email |
	| **Comms** | OnionShare | Anonymous file transfer |
	| **Files** | MAT2 / ExifTool | Strip metadata |
	| **Files** | VeraCrypt | Encrypted storage |
	| **Delete** | shred / srm | Secure file deletion |
	| **Firewall** | ufw + iptables | Kill switch + filtering |
	| **PGP** | GnuPG | Message encryption |
	
	### Install Everything At Once
	
	```bash
	sudo apt update && sudo apt install -y \
	tor torbrowser-launcher \
	proxychains4 \
	macchanger \
	dnscrypt-proxy \
	mat2 libimage-exiftool-perl \
	secure-delete \
	cryptsetup veracrypt \
	ufw iptables \
	gnupg \
	keepassxc \
	onionshare \
	usbguard \
	firejail \
	fail2ban && \
	pip3 install stem requests[socks]
	
	echo "[+] All anonymity tools installed!"
	```
	
	---
	
	## The Anonymity Pyramid
	
	```
	/\
	/  \
	/ ??  \          ← Impossible (you still exist)
	/________\
	/          \
	/ Tails+Tor  \      ← Highest: NSA-level adversary needed
	/______________\
	/                \
	/ Whonix+VPN+Tor   \   ← Very High: Nation-state level needed
	/____________________\
	/                      \
	/ VPN+Tor+Good OPSEC    \ ← High: Most attackers can't find you
	/________________________\
	\                        /
	\   VPN alone          /  ← Medium: Protects from most
	\____________________/
	\                  /
	\   Nothing      /    ← Low: Everyone can see you
	\______________/
	
	Start at the bottom, add layers until you reach your threat model.
	Most researchers need: VPN + Tor + Good OPSEC + Metadata cleaning
	Journalists/activists: Add Tails or Whonix
	```
	
	---
	
	*Anonymity is not about doing anything wrong.*
	*It is about having the right to privacy, which is a fundamental human right.*
	*The same techniques that protect activists and journalists also protect security researchers.*
