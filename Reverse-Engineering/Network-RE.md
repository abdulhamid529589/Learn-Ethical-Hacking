# 🌐 Network Reverse Engineering — Complete Guide

### Protocol Analysis, Packet Capture, and Traffic Decoding

> **Who is this for?** You want to understand network traffic at a deep level — capture and decode packets, reverse engineer unknown protocols, analyze malware C2 communication, intercept encrypted traffic, and build custom network tools.

---

## 📚 Table of Contents

1. [How Networking Really Works](#1-how-networking-really-works)
2. [Wireshark — Deep Packet Inspection](#2-wireshark)
3. [tcpdump — Command Line Capture](#3-tcpdump)
4. [Protocol Analysis](#4-protocol-analysis)
5. [Reversing Unknown Protocols](#5-reversing-unknown-protocols)
6. [TLS/SSL Traffic Analysis](#6-tlsssl-traffic-analysis)
7. [DNS Analysis](#7-dns-analysis)
8. [Detecting Malware C2 Traffic](#8-detecting-malware-c2-traffic)
9. [Network Tools with Python](#9-network-tools-with-python)
10. [Wireless Network Analysis](#10-wireless-network-analysis)
11. [Tools Reference](#11-tools-reference)
12. [Practice Labs](#12-practice-labs)

---

## 1. How Networking Really Works

Before capturing traffic, you must understand the layers it travels through.

### The Network Stack (OSI Model — Simplified)

```
Layer 7: Application    HTTP, DNS, FTP, SMTP — what apps use
Layer 6: Presentation   Encryption (TLS), compression
Layer 5: Session        Connection management
Layer 4: Transport      TCP, UDP — reliability and ports
Layer 3: Network        IP — routing between machines
Layer 2: Data Link      Ethernet, WiFi — local delivery
Layer 1: Physical       Cables, radio waves
```

### What a Packet Looks Like

Every piece of data on a network is wrapped in multiple layers (like envelopes inside envelopes):

```
┌───────────────────────────────────────────────────────────┐
│  Ethernet Header (Layer 2)                                │
│  Source MAC: AA:BB:CC:DD:EE:FF                            │
│  Destination MAC: 11:22:33:44:55:66                       │
├───────────────────────────────────────────────────────────┤
│  IP Header (Layer 3)                                      │
│  Source IP: 192.168.1.100                                 │
│  Destination IP: 93.184.216.34                            │
│  Protocol: TCP (6)                                        │
├───────────────────────────────────────────────────────────┤
│  TCP Header (Layer 4)                                     │
│  Source Port: 54321                                       │
│  Destination Port: 443                                    │
│  Sequence Number: 1234567                                 │
│  Flags: SYN, ACK, PSH, FIN, RST                          │
├───────────────────────────────────────────────────────────┤
│  Application Data (Layer 7)                               │
│  GET /index.html HTTP/1.1                                 │
│  Host: example.com                                        │
└───────────────────────────────────────────────────────────┘
```

### TCP vs UDP

| Feature     | TCP                           | UDP                         |
| ----------- | ----------------------------- | --------------------------- |
| Connection  | Connection-based (handshake)  | Connectionless              |
| Reliability | Guaranteed delivery, ordering | No guarantee                |
| Speed       | Slower                        | Faster                      |
| Use Cases   | HTTP, HTTPS, SSH, FTP         | DNS, VoIP, games, streaming |
| Headers     | Larger (20+ bytes)            | Smaller (8 bytes)           |

### The TCP Handshake

Every TCP connection starts with this 3-way handshake:

```
Client                    Server
│                          │
│──── SYN ────────────────▶│  "I want to connect"
│                          │
│◀─── SYN+ACK ─────────────│  "OK, I'm ready"
│                          │
│──── ACK ────────────────▶│  "Great, let's go"
│                          │
│═══ Data flows both ways ═│
│                          │
│──── FIN ────────────────▶│  "I'm done"
│◀─── FIN+ACK ─────────────│  "OK, me too"
```

**Why this matters for RE:**

- Seeing SYN packets = connection attempts (malware reaching out to C2?)
- Seeing RST packets = connection refused (port closed)
- Seeing lots of SYNs without ACKs = port scan

### Port Numbers

```
Well-known ports (0-1023):
20, 21   → FTP (file transfer)
22       → SSH (encrypted shell)
23       → Telnet (unencrypted shell)
25       → SMTP (email sending)
53       → DNS (domain lookup)
80       → HTTP
110      → POP3 (email retrieval)
143      → IMAP (email)
443      → HTTPS
445      → SMB (Windows file sharing)
3306     → MySQL
3389     → RDP (Windows Remote Desktop)

Suspicious ports (malware often uses):
4444     → Metasploit default
5555     → Android Debug Bridge
6666     → Common backdoor
1337     → "Leet" — hacker humor, often CTFs
31337    → "Elite" — classic backdoor port
8080     → Alt HTTP / proxies
9090     → Common C2 port
```

---

## 2. Wireshark

Wireshark is the most powerful packet analysis tool. It captures and analyzes network traffic in real time.

### Installation and Setup

```bash
# Linux
sudo apt install wireshark
sudo usermod -aG wireshark $USER   # Allow non-root capture
logout && login again

# Windows / Mac: Download from wireshark.org

# Start capture
wireshark
# Select interface (usually eth0 or wlan0)
# Click the blue shark fin to start
```

### Interface Overview

```
Menu bar:     File, Edit, View, Capture, Analyze, Statistics
Toolbar:      Start/Stop capture, open/save files
Filter bar:   Type display filters here
Packet list:  All captured packets (time, source, dest, protocol, info)
Packet details: Tree view of selected packet's decoded fields
Hex dump:     Raw bytes of selected packet
```

### Display Filters (Most Important Skill)

Display filters narrow down which packets you see — critical when you have thousands of packets.

**Basic syntax:**

```
protocol                    → Show only this protocol
field == value              → Exact match
field contains "string"     → Contains substring
field matches "regex"       → Regular expression match
!filter  or  not filter     → Negate
filter1 and filter2         → Both must match
filter1 or filter2          → Either matches
```

**Protocol filters:**

```wireshark
http                        # HTTP traffic
https  (or tls or ssl)      # HTTPS/TLS
dns                         # DNS queries
tcp                         # All TCP
udp                         # All UDP
icmp                        # Ping traffic
arp                         # ARP (MAC resolution)
smb                         # Windows file sharing
ftp                         # FTP
ssh                         # SSH
```

**IP and port filters:**

```wireshark
ip.addr == 192.168.1.100          # Traffic to/from this IP
ip.src == 192.168.1.100           # Only FROM this IP
ip.dst == 8.8.8.8                 # Only TO this IP
ip.addr == 192.168.1.0/24         # Entire subnet

tcp.port == 80                    # Either side on port 80
tcp.dstport == 443                # Destination port 443
tcp.srcport == 4444               # Source port 4444

!(ip.addr == 192.168.1.1)         # Exclude router traffic
```

**Content filters:**

```wireshark
http.request.method == "POST"           # POST requests
http.request.uri contains "login"       # Login requests
http contains "password"                # Any HTTP with "password"
http.response.code == 200               # Success responses
http.response.code == 401              # Unauthorized

dns.qry.name contains "evil"           # DNS for suspicious domains
dns.qry.name matches ".*\.xyz$"        # .xyz TLD (often malware)

tcp.flags.syn == 1 and tcp.flags.ack == 0  # SYN packets (new connections)
tcp.flags.rst == 1                     # Reset connections
```

**Malware hunting filters:**

```wireshark
# Connections to suspicious ports
tcp.dstport == 4444 or tcp.dstport == 6666 or tcp.dstport == 1337

# Connections outside local network
!(ip.dst == 192.168.0.0/16) and !(ip.dst == 10.0.0.0/8) and ip.dst != 127.0.0.1

# Large outbound data (exfiltration?)
tcp.len > 1000 and ip.dst != 192.168.0.0/16

# Frequent beaconing (check time column — regular intervals)
ip.dst == [suspicious-ip]

# Non-standard HTTP ports
http and not (tcp.port == 80 or tcp.port == 8080)
```

### Following Streams

Right-click any packet → Follow → TCP/UDP/HTTP Stream

This reassembles the full conversation and shows it as readable text:

```
GET /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{"username":"admin","password":"secret123"}

HTTP/1.1 200 OK
Content-Type: application/json

{"token":"eyJhbGciOiJIUzI1NiJ9..."}
```

**Color coding:**

- Red/pink = client → server
- Blue = server → client

### Exporting Objects

Wireshark can extract files from traffic:

```
File → Export Objects → HTTP (extracts files transferred over HTTP)
File → Export Objects → SMB (extracts files from Windows shares)
File → Export Objects → DICOM (medical imaging)
```

### Statistics and Analysis

```
Statistics → Protocol Hierarchy    # What protocols are in the capture?
Statistics → Conversations         # Which IPs talked to each other?
Statistics → Endpoints             # All IP addresses in capture
Statistics → IO Graph              # Traffic over time graph
Statistics → DNS                   # DNS query summary
Analyze → Expert Information       # Wireshark's analysis of anomalies
```

### Saving and Sharing Captures

```bash
# Save capture
File → Save As → *.pcap or *.pcapng

# Read pcap from command line
tshark -r capture.pcap

# Filter while reading
tshark -r capture.pcap -Y "http.request.method==POST"

# Export specific fields
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e http.request.uri
```

---

## 3. tcpdump

tcpdump is the command-line packet capture tool — essential on servers where you don't have a GUI.

### Basic Capture

```bash
# Capture all traffic on interface
sudo tcpdump -i eth0

# List available interfaces
sudo tcpdump -D

# Capture to file (always do this for analysis later)
sudo tcpdump -i eth0 -w capture.pcap

# Capture with timestamps and readable output
sudo tcpdump -i eth0 -nn -tttt

# Flags:
# -i      specify interface (-i any = all interfaces)
# -w      write to file
# -r      read from file
# -n      don't resolve IPs to hostnames
# -nn     don't resolve IPs or ports
# -v      verbose (more detail)
# -vvv    very verbose
# -s 0    capture full packets (default truncates)
# -A      print ASCII content
# -X      print hex + ASCII
# -c 100  stop after 100 packets
```

### Capture Filters (BPF Syntax)

```bash
# By host
sudo tcpdump -i eth0 host 192.168.1.100
sudo tcpdump -i eth0 src 192.168.1.100
sudo tcpdump -i eth0 dst 8.8.8.8

# By port
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 port 80 or port 443
sudo tcpdump -i eth0 portrange 8000-9000

# By protocol
sudo tcpdump -i eth0 tcp
sudo tcpdump -i eth0 udp
sudo tcpdump -i eth0 icmp

# By network
sudo tcpdump -i eth0 net 192.168.1.0/24
sudo tcpdump -i eth0 not net 192.168.0.0/16

# Combinations
sudo tcpdump -i eth0 'tcp port 80 and src 192.168.1.100'
sudo tcpdump -i eth0 'tcp port 443 and (src host 1.2.3.4 or dst host 1.2.3.4)'

# Show HTTP content
sudo tcpdump -i eth0 -A -s 0 'tcp port 80'

# Show DNS queries
sudo tcpdump -i eth0 -n 'udp port 53'
```

### Reading and Analyzing pcap Files

```bash
# Read a capture file
tcpdump -r capture.pcap

# Apply filter when reading
tcpdump -r capture.pcap 'tcp port 80'

# Count packets per source IP
tcpdump -r capture.pcap -nn | awk '{print $3}' | sort | uniq -c | sort -rn

# Extract HTTP content
tcpdump -r capture.pcap -A -s 0 'tcp port 80' | grep -E "GET|POST|Host:|Cookie:"

# Show only DNS queries
tcpdump -r capture.pcap -n 'udp port 53' -A | grep -E "A\?|AAAA\?"
```

---

## 4. Protocol Analysis

### Analyzing HTTP in Wireshark

```
Filter: http

Right-click on a request → Follow → HTTP Stream
→ See full request + response as text

Statistics → HTTP → Requests
→ See all URLs accessed

Filter: http.request.method == "POST" and http contains "password"
→ Find login attempts with cleartext passwords!
```

### Analyzing HTTPS/TLS

TLS encrypts content, but you can still see:

```
Filter: tls

What you CAN see without decryption:
- Server name (SNI) in TLS ClientHello
Filter: tls.handshake.extensions_server_name
→ Shows which domains are being accessed!

- Certificate information
Filter: tls.handshake.type == 11
→ Server's certificate (who issued it, domain)

- TLS version (old versions = security issue)
Filter: tls.record.version < 0x0303
→ TLS 1.0 or 1.1 (vulnerable)

What you CANNOT see without the key:
- Actual data being transferred
- HTTP headers and body
```

### Analyzing DNS

DNS reveals a LOT even when traffic is encrypted:

```
Filter: dns

# See all domains being looked up
tshark -r capture.pcap -T fields -e dns.qry.name | sort | uniq

# What Wireshark shows per DNS packet:
# Query: www.example.com Type A  (looking up IPv4)
# Response: www.example.com → 93.184.216.34

# Suspicious patterns:
dns.qry.name contains "dga"         # DGA domains (random-looking names)
dns.flags.rcode == 3                # NXDOMAIN (domain doesn't exist)
dns.resp.ttl < 60                   # Very low TTL (fast-flux DNS = malware!)
```

**Detecting DGA (Domain Generation Algorithm) domains:**

````python
# dga_detector.py
import math
from collections import Counter

def calculate_entropy(domain):
"""High entropy = random-looking = possibly DGA"""
domain_part = domain.split('.')[0]  # Remove TLD
counter = Counter(domain_part)
length = len(domain_part)
entropy = -sum((c/length) * math.log2(c/length)
for c in counter.values())
return entropy

def is_likely_dga(domain):
domain_part = domain.split('.')[0]
entropy = calculate_entropy(domain)

# DGA characteristics:
# - Long domain name (> 10 chars)
# - High entropy (> 3.5) = random-looking
# - No real words
return len(domain_part) > 10 and entropy > 3.5

# Test
domains = [
	"www.google.com",           # Legit
	"xkf9j2mplqrs8vb.com",     # DGA
	"a.b.c.d.e.evil.com",      # Suspicious subdomain chaining
	"api.example.com",          # Legit
]

for d in domains:
	print(f"{d}: {'SUSPICIOUS DGA' if is_likely_dga(d) else 'OK'} (entropy: {calculate_entropy(d):.2f})")
	```

	### Analyzing SMB (Windows File Sharing)

	```
	Filter: smb or smb2

	# See files being accessed
	tshark -r capture.pcap -Y smb2 -T fields -e smb2.filename

	# Look for credential hashes (NTLMv2)
	Filter: ntlmssp
	# Can crack with hashcat!

	# Suspicious: Large file transfers, lateral movement
	Filter: smb2.cmd == 5  # SMB2 Create (file open/create)
	```

	### Analyzing FTP (Cleartext!)

	FTP sends credentials in plaintext:

	```
	Filter: ftp

	# See credentials
	Filter: ftp.request.command == "USER" or ftp.request.command == "PASS"

	# Follow stream to see all files transferred
	Right-click → Follow → TCP Stream
	# You'll see:
	# USER admin
	# PASS password123    ← Credentials!
	# RETR secret_file.txt
	```

	---

	## 5. Reversing Unknown Protocols

	When you capture traffic from an unknown application, how do you figure out the protocol?

	### Step-by-Step Unknown Protocol Analysis

	**Step 1: Identify the transport**
	```
	Is it TCP or UDP?
	TCP → ordered, reliable → good for command protocols
	UDP → unordered, fast → good for streaming, games

	What port is it using?
	Standard port → probably standard protocol
	Unusual port → custom/proprietary protocol

	What's the packet size distribution?
	All same size → likely a streaming or gaming protocol
	Variable sizes → likely a text or command protocol
	```

	**Step 2: Look at the raw bytes**
	```
	In Wireshark: Click packet → Hex dump at bottom
	Look for:
	- Readable text (ASCII range 0x20-0x7E)
	- Magic bytes at start of packets
	- Repeating patterns
	- Length fields (value matches rest of packet length)
	```

	**Step 3: Collect many samples**
	```
	Do different actions in the application:
	- Log in
	- Click different features
	- Send a message
	- Buy something

	Capture a packet for each action.
	Compare packets:
	- What changes? That's the variable data (username, message, etc.)
	- What stays the same? That's headers, magic bytes, protocol structure
	```

	**Step 4: Identify structure**

	A typical custom protocol might look like:

	```
	Byte 0-1:  Magic bytes (always the same, like "AB" or 0x5A4D)
	Byte 2:    Packet type (command ID)
	Byte 3-4:  Payload length
	Byte 5+:   Payload (varies by type)

	Example packets:
	Login:     [AB] [01] [00 0E] [admin\0password\0]
	magic type  len    username + password

	Move:      [AB] [02] [00 08] [00 00 01 F4] [00 00 02 8A]
	magic type  len    x coord       y coord

	Chat:      [AB] [03] [00 0D] [Hello, World!\0]
	magic type  len    message
	```

	### Example: Reversing a Game Protocol

	```python
	# game_protocol_analyzer.py
	import struct
	from scapy.all import rdpcap, Raw

	def analyze_game_packets(pcap_file, game_ip, game_port):
	packets = rdpcap(pcap_file)

	for pkt in packets:
		if pkt.haslayer('TCP') and pkt.haslayer(Raw):
			# Filter game traffic
			if (pkt['IP'].dst == game_ip and pkt['TCP'].dport == game_port) or \
				(pkt['IP'].src == game_ip and pkt['TCP'].sport == game_port):

				data = bytes(pkt[Raw])
				if len(data) < 4:
					continue

					# Try to parse our guessed structure:
					# [2 bytes magic][1 byte type][2 bytes length][payload]

					magic = data[0:2]
					if magic != b'\xAB\xCD':  # Our magic bytes
						continue

						pkt_type = data[2]
						length = struct.unpack('>H', data[3:5])[0]
						payload = data[5:5+length]

						direction = "CLIENT→SERVER" if pkt['IP'].dst == game_ip else "SERVER→CLIENT"

						print(f"\n{direction}")
						print(f"Type: {hex(pkt_type)} ({get_type_name(pkt_type)})")
						print(f"Length: {length}")
						print(f"Payload (hex): {payload.hex()}")

						# Try to parse payload as text
						try:
						text = payload.decode('utf-8')
						print(f"Payload (text): {text}")
						except:
						pass

						# Parse specific packet types
						parse_packet(pkt_type, payload)

						def get_type_name(pkt_type):
						types = {
							0x01: 'LOGIN',
							0x02: 'MOVE',
							0x03: 'CHAT',
							0x04: 'ATTACK',
							0x10: 'SERVER_ACK',
							0x11: 'PLAYER_LIST',
							0x12: 'GAME_STATE',
						}
						return types.get(pkt_type, f'UNKNOWN_{hex(pkt_type)}')

						def parse_packet(pkt_type, payload):
						if pkt_type == 0x02:  # MOVE packet
							if len(payload) >= 8:
								x, y = struct.unpack('>ff', payload[0:8])
								print(f"  Position: x={x:.2f}, y={y:.2f}")

								elif pkt_type == 0x04:  # ATTACK packet
								if len(payload) >= 8:
									target_id, damage = struct.unpack('>II', payload[0:8])
									print(f"  Attack: target={target_id}, damage={damage}")

									elif pkt_type == 0x03:  # CHAT packet
									try:
									msg = payload.decode('utf-8').strip('\x00')
									print(f"  Chat: '{msg}'")
									except:
									pass

									analyze_game_packets('game_traffic.pcap', '192.168.1.200', 7777)
									```

									### Building a Protocol Fuzzer

									Once you understand a protocol, fuzz it to find vulnerabilities:

									```python
									# protocol_fuzzer.py
									import socket
									import struct
									import random

									class ProtocolFuzzer:
									def __init__(self, host, port):
									self.host = host
									self.port = port

									def build_packet(self, pkt_type, payload):
									"""Build a valid packet"""
									header = b'\xAB\xCD'          # Magic
									header += bytes([pkt_type])    # Type
									header += struct.pack('>H', len(payload))  # Length
									return header + payload

									def send_packet(self, packet):
									try:
									s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
									s.settimeout(5)
									s.connect((self.host, self.port))
									s.send(packet)
									response = s.recv(4096)
									s.close()
									return response
									except Exception as e:
									return f"Error: {e}"

									def fuzz_length_field(self, pkt_type):
									"""Try different length values"""
									print(f"Fuzzing length field for packet type {hex(pkt_type)}")

									for length in [0, 1, 255, 256, 65535, 65536, 0xFFFFFFFF]:
										# Build packet with incorrect length
										header = b'\xAB\xCD' + bytes([pkt_type])
										header += struct.pack('>H', length)  # Wrong length
										payload = b'A' * 100  # Fixed payload

										try:
										s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
										s.settimeout(2)
										s.connect((self.host, self.port))
										s.send(header + payload)
										resp = s.recv(4096)
										s.close()
										print(f"  Length {length}: Got response ({len(resp)} bytes)")
										except Exception as e:
										print(f"  Length {length}: {e} ← POSSIBLE CRASH!")

										def fuzz_payload(self, pkt_type, normal_payload):
										"""Fuzz the payload with various inputs"""
										fuzz_inputs = [
											b'A' * 1000,          # Buffer overflow attempt
											b'\x00' * 100,        # Null bytes
											b'\xff' * 100,        # Max bytes
											b'%s' * 50,           # Format string
											b'../../../etc/passwd', # Path traversal
											b"'; DROP TABLE--",   # SQL injection
											b'\n\r\n\r',          # CRLF injection
											b'\x00',              # Single null
											b'',                  # Empty payload
										]

										for fuzz in fuzz_inputs:
											pkt = self.build_packet(pkt_type, fuzz)
											result = self.send_packet(pkt)
											print(f"Payload {fuzz[:20]}...: {result}")

											# Usage
											fuzzer = ProtocolFuzzer('192.168.1.200', 7777)
											fuzzer.fuzz_length_field(0x01)  # Fuzz login packet
											fuzzer.fuzz_payload(0x03, b'hello')  # Fuzz chat packet
											```

											---

## 6. TLS/SSL Traffic Analysis

### Understanding TLS Handshake

Even without decrypting, the TLS handshake reveals a lot:

````

Client Hello →
├── TLS version supported
├── Cipher suites supported
└── SNI (Server Name Indication) ← The domain name!

← Server Hello
├── Chosen TLS version
├── Chosen cipher suite
└── Server certificate

← Certificate
├── Domain name (CN / SAN)
├── Issuer (who signed it)
├── Valid from/to dates
└── Self-signed? (suspicious!)

← Certificate Verify, Finished
Client Finished →

═══ Encrypted application data ═══

```

**What you can extract from TLS without decryption:**
```

Filter: tls.handshake.extensions_server_name
→ All domain names being accessed (even in encrypted traffic!)

Filter: tls.handshake.type == 11
→ All server certificates
→ Right-click → Follow → TLS Stream → see certificate details

Filter: tls.record.version < 0x0303
→ Connections using TLS 1.0 or 1.1 (outdated, vulnerable)

Filter: ssl.handshake.ciphersuite == 0x0035
→ Weak cipher suites being negotiated

```

### Decrypting TLS Traffic

**Method 1: With the private key (server)**
```

Wireshark → Edit → Preferences → Protocols → TLS
Add RSA key:
IP: server IP
Port: 443
Key file: server_private_key.pem

````

**Method 2: With SSLKEYLOGFILE (best method)**

Modern TLS uses Perfect Forward Secrecy (PFS) — per-session keys. To decrypt, log the session keys:

```bash
# Set environment variable to log keys
export SSLKEYLOGFILE=/tmp/ssl_keys.log

# Launch browser with key logging
SSLKEYLOGFILE=/tmp/ssl_keys.log firefox

# In Wireshark:
Edit → Preferences → Protocols → TLS → Master secret log filename
Point to: /tmp/ssl_keys.log

# Now all HTTPS traffic is decrypted in Wireshark!
````

**Method 3: Using Frida to extract keys**

```javascript
// extract_tls_keys.js
// Works on applications that use OpenSSL

var SSL_CTX_new = Module.findExportByName('libssl.so', 'SSL_CTX_new')
var SSL_read = Module.findExportByName('libssl.so', 'SSL_read')
var SSL_write = Module.findExportByName('libssl.so', 'SSL_write')

if (SSL_read) {
  Interceptor.attach(SSL_read, {
    onLeave: function (retval) {
      if (retval.toInt32() > 0) {
        var buf = this.context.rsi // Buffer argument
        var len = retval.toInt32()
        console.log('[TLS READ ' + len + ' bytes]')
        console.log(hexdump(buf, { length: len }))
      }
    },
  })
}

if (SSL_write) {
  Interceptor.attach(SSL_write, {
    onEnter: function (args) {
      var len = args[2].toInt32()
      if (len > 0) {
        console.log('[TLS WRITE ' + len + ' bytes]')
        console.log(hexdump(args[1], { length: len }))
      }
    },
  })
}
```

---

## 7. DNS Analysis

DNS is often overlooked but extremely revealing.

### Why DNS Matters

- Reveals ALL domains a machine communicates with
- Hard to fully encrypt (DNS over HTTPS still has limitations)
- Malware uses DNS for C2 (DNS tunneling, DGA)
- Exfiltration can hide in DNS queries!

### DNS Deep Dive in Wireshark

```
Filter: dns

# What gets revealed per query:
# Query: malware-c2.com Type A
# → Even if connection is blocked, you know the domain!

# See only failed lookups (NXDOMAIN)
Filter: dns.flags.rcode == 3
# Lots of NXDOMAIN = possible DGA malware trying many domains

# Very fast lookups (automating?)
Filter: dns and frame.time_delta < 0.001

# Unusual query types (data hiding?)
Filter: dns.qry.type == 16    # TXT record queries
Filter: dns.qry.type == 28    # AAAA (IPv6) queries
Filter: dns.qry.type == 15    # MX (mail) queries
```

### DNS Tunneling Detection

Malware can exfiltrate data by encoding it in DNS queries:

```
Normal DNS: www.google.com → 142.250.80.46

DNS Tunnel:
hex_encoded_data_chunk1.attacker.com
hex_encoded_data_chunk2.attacker.com
hex_encoded_data_chunk3.attacker.com

The DNS "queries" look like:
48656c6c6f20576f726c64.attacker.com  (hex for "Hello World")
```

**Detecting DNS tunneling:**

````python
# dns_tunnel_detector.py
import binascii
import math
from collections import Counter

def check_dns_tunnel(hostname):
"""Check if DNS query looks like tunneled data"""
subdomain = hostname.split('.')[0]

# Indicators of DNS tunneling:

# 1. Very long subdomain (normal DNS rarely exceeds 30 chars)
if len(subdomain) > 40:
	return True, f"Long subdomain ({len(subdomain)} chars)"

	# 2. High entropy (random-looking = encoded data)
	counter = Counter(subdomain)
	length = len(subdomain)
	if length > 0:
		entropy = -sum((c/length) * math.log2(c/length)
		for c in counter.values())
		if entropy > 4.0:
			return True, f"High entropy ({entropy:.2f})"

			# 3. Looks like base64 or hex
			hex_chars = set('0123456789abcdef')
			if all(c in hex_chars for c in subdomain.lower()) and len(subdomain) > 20:
				return True, "Possible hex encoding"

				base64_chars = set('ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=')
				if all(c in base64_chars for c in subdomain) and len(subdomain) > 30:
					return True, "Possible base64 encoding"

					return False, "Looks normal"

					# Test
					queries = [
						"www.google.com",

"48656c6c6f20576f726c64.evil.com", # hex
"SGVsbG8gV29ybGQh.attacker.com", # base64
"randomlookingxkf9j2mplqrs8vb.example.com", # DGA-like
"api.legitimate-site.com",
]

for q in queries:
	is_tunnel, reason = check_dns_tunnel(q)
	status = "⚠ TUNNEL?" if is_tunnel else "✓ OK"
	print(f"{status} {q} ({reason})")
	```

	---

## 8. Detecting Malware C2 Traffic

### Beacon Detection

Malware periodically "beacons" to its C2 server — like a heartbeat. The regular intervals are a giveaway.

```python
# beacon_detector.py
from scapy.all import rdpcap
from collections import defaultdict
import statistics

def detect_beaconing(pcap_file, threshold_cv=0.3):
"""
Detect beaconing by looking for regular connection intervals.
Low coefficient of variation = regular intervals = beaconing
"""
packets = rdpcap(pcap_file)

# Track connection times per destination
connections = defaultdict(list)

for pkt in packets:
	if 'TCP' in pkt and pkt['TCP'].flags & 0x02:  # SYN packets
		dst_ip = pkt['IP'].dst
		dst_port = pkt['TCP'].dport
		timestamp = float(pkt.time)

		key = f"{dst_ip}:{dst_port}"
		connections[key].append(timestamp)

		print("=== BEACONING ANALYSIS ===\n")

		for destination, times in connections.items():
			if len(times) < 5:  # Need enough samples
				continue

				# Calculate intervals between connections
				intervals = [times[i+1] - times[i] for i in range(len(times)-1)]

				avg_interval = statistics.mean(intervals)
				if avg_interval < 1:  # Skip very frequent normal traffic
					continue

					std_dev = statistics.stdev(intervals) if len(intervals) > 1 else 0
					cv = std_dev / avg_interval if avg_interval > 0 else 1

					# Low CV = regular = possible beaconing
					if cv < threshold_cv and len(times) >= 5:
						print(f"⚠ POSSIBLE BEACON: {destination}")
						print(f"  Connections: {len(times)}")
						print(f"  Avg interval: {avg_interval:.1f}s")
						print(f"  Std deviation: {std_dev:.1f}s")
						print(f"  Coefficient of variation: {cv:.3f}")
						print()

						detect_beaconing('malware_traffic.pcap')
						```

						### C2 Traffic Patterns

						```
						Pattern 1: Regular beaconing
						→ SYN packets every N seconds, very consistent timing

						Pattern 2: Keep-alive HTTP
						→ POST /update.php every 60 seconds
						→ Response: {"cmd": "idle"} or {"cmd": "execute", "command": "..."}

						Pattern 3: DNS beaconing
						→ DNS query for: [random].c2domain.com every few minutes
						→ Response IPs encode commands

						Pattern 4: Social media C2
						→ Connects to Twitter/Pastebin/GitHub
						→ Reads specific account/paste for commands (blends in!)

						Pattern 5: ICMP tunneling
						→ Data hidden in ICMP ping payloads
						→ Filter: icmp and icmp.type == 8
						→ Check payload for non-standard content
						```

						**Hunting C2 with Wireshark:**
						```wireshark
						# Regular POST to same URL
						http.request.method == "POST" and http.request.uri contains "update"

						# User-Agent doesn't match a real browser
						http.user_agent contains "python" or http.user_agent == ""

						# Suspicious domains
						dns.qry.name matches ".*[0-9]{5,}.*"  # Numbers in domain name

						# Data in unusual places
						icmp and data.len > 20   # ICMP with large payload (tunneling!)

						# IRC (botnet communication)
						tcp.port == 6667 and tcp.payload contains "PRIVMSG"
						```

    																						---

## 9. Network Tools with Python

### Scapy — Craft and Send Any Packet

Scapy is Python's packet crafting library — you can build any packet from scratch.

```python
from scapy.all import *

# ============================================
# Basic packet crafting
# ============================================

# Build and send an ICMP ping
pkt = IP(dst="8.8.8.8") / ICMP()
reply = sr1(pkt, timeout=2)
if reply:
	print(f"Got reply from {reply[IP].src}")

# TCP SYN packet (port scan)
pkt = IP(dst="192.168.1.1") / TCP(dport=80, flags="S")
reply = sr1(pkt, timeout=2)
if reply and reply[TCP].flags == "SA":  # SYN+ACK
	print("Port 80 is OPEN")
	elif reply and reply[TCP].flags == "RA":  # RST+ACK
	print("Port 80 is CLOSED")

# UDP packet
pkt = IP(dst="8.8.8.8") / UDP(dport=53) / DNS(rd=1, qd=DNSQR(qname="example.com"))
reply = sr1(pkt, timeout=2)
if reply:
	print(f"DNS answer: {reply[DNSRR].rdata}")

# ============================================
# Port scanner
# ============================================
def port_scan(target, ports):
open_ports = []

for port in ports:
pkt = IP(dst=target) / TCP(dport=port, flags="S")
reply = sr1(pkt, timeout=1, verbose=0)

if reply and reply.haslayer(TCP):
if reply[TCP].flags == 0x12:  # SYN+ACK
open_ports.append(port)
# Send RST to close properly
send(IP(dst=target)/TCP(dport=port, flags="R"), verbose=0)

return open_ports

target = "192.168.1.1"
ports = list(range(1, 1025))
print(f"Scanning {target}...")
open_ports = port_scan(target, ports)
print(f"Open ports: {open_ports}")

# ============================================
# Packet sniffer with analysis
# ============================================
def analyze_packet(pkt):
if pkt.haslayer(DNS) and pkt.haslayer(DNSQR):
print(f"[DNS] {pkt[IP].src} queried: {pkt[DNSQR].qname.decode()}")

if pkt.haslayer(TCP) and pkt.haslayer(Raw):
data = bytes(pkt[Raw])
if b'GET' in data or b'POST' in data:
lines = data.decode('utf-8', errors='ignore').split('\r\n')
print(f"[HTTP] {pkt[IP].src}:{pkt[TCP].sport} → {pkt[IP].dst}")
for line in lines[:3]:
print(f"  {line}")

# Sniff for 30 seconds
sniff(prn=analyze_packet, timeout=30, store=0)
```

### Building a Simple Protocol Parser

```python
# protocol_parser.py
import socket
import struct
import threading

class ProtocolServer:
"""
Simple server that implements a custom protocol for testing:

Packet format:
[2 bytes: magic (0xABCD)]
[1 byte:  type]
[2 bytes: payload length]
[N bytes: payload]
"""

MAGIC = b'\xAB\xCD'

PACKET_TYPES = {
0x01: 'LOGIN',
0x02: 'MESSAGE',
0x03: 'COMMAND',
0x10: 'ACK',
0x11: 'ERROR',
}

def __init__(self, host='0.0.0.0', port=9999):
self.host = host
self.port = port

def parse_packet(self, data):
"""Parse incoming packet"""
if len(data) < 5:
return None

magic = data[0:2]
if magic != self.MAGIC:
return None

pkt_type = data[2]
payload_len = struct.unpack('>H', data[3:5])[0]
payload = data[5:5+payload_len]

return {
'type': pkt_type,
'type_name': self.PACKET_TYPES.get(pkt_type, 'UNKNOWN'),
'length': payload_len,
'payload': payload
}

def build_response(self, pkt_type, payload):
"""Build a response packet"""
packet = self.MAGIC
packet += bytes([pkt_type])
packet += struct.pack('>H', len(payload))
packet += payload
return packet

def handle_packet(self, pkt):
"""Handle parsed packet, return response"""
if pkt['type'] == 0x01:  # LOGIN
username = pkt['payload'].split(b'\x00')[0].decode()
print(f"[LOGIN] User: {username}")
return self.build_response(0x10, b'OK')

elif pkt['type'] == 0x02:  # MESSAGE
msg = pkt['payload'].decode('utf-8', errors='ignore')
print(f"[MSG] {msg}")
return self.build_response(0x10, b'OK')

return self.build_response(0x11, b'UNKNOWN_TYPE')

def handle_client(self, conn, addr):
print(f"[+] Connection from {addr}")
buffer = b''

while True:
data = conn.recv(1024)
if not data:
break

buffer += data

# Parse packets from buffer
while len(buffer) >= 5:
payload_len = struct.unpack('>H', buffer[3:5])[0]
pkt_size = 5 + payload_len

if len(buffer) < pkt_size:
break

raw_pkt = buffer[:pkt_size]
buffer = buffer[pkt_size:]

pkt = self.parse_packet(raw_pkt)
if pkt:
response = self.handle_packet(pkt)
conn.send(response)

conn.close()

def start(self):
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind((self.host, self.port))
s.listen(5)
print(f"[*] Server listening on {self.host}:{self.port}")

while True:
conn, addr = s.accept()
t = threading.Thread(target=self.handle_client, args=(conn, addr))
t.daemon = True
t.start()

# Start server
server = ProtocolServer()
server.start()
```

### Automated Network Reconnaissance

```python
# net_recon.py
import subprocess
import socket
import concurrent.futures
import ipaddress

def ping(ip):
"""Check if host is alive"""
result = subprocess.run(
['ping', '-c', '1', '-W', '1', str(ip)],
capture_output=True
)
return result.returncode == 0

def check_port(ip, port, timeout=1):
"""Check if a specific port is open"""
try:
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.settimeout(timeout)
result = s.connect_ex((str(ip), port))
s.close()
return result == 0
except:
return False

def grab_banner(ip, port, timeout=2):
"""Get service banner"""
try:
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.settimeout(timeout)
s.connect((str(ip), port))
s.send(b'HEAD / HTTP/1.0\r\n\r\n')
banner = s.recv(1024).decode('utf-8', errors='ignore').strip()
s.close()
return banner[:100]
except:
return ''

def scan_network(network_cidr, ports_to_scan=None):
"""Scan entire network"""
if ports_to_scan is None:
ports_to_scan = [21, 22, 23, 25, 53, 80, 443, 445, 3306, 3389, 8080]

network = ipaddress.IPv4Network(network_cidr, strict=False)
results = {}

print(f"Scanning {network_cidr}...")

# Find alive hosts
alive_hosts = []
with concurrent.futures.ThreadPoolExecutor(max_workers=50) as executor:
futures = {executor.submit(ping, ip): ip for ip in network.hosts()}
for future in concurrent.futures.as_completed(futures):
ip = futures[future]
if future.result():
alive_hosts.append(str(ip))
print(f"  [+] Host alive: {ip}")

# Scan ports on alive hosts
for host in alive_hosts:
open_ports = []
for port in ports_to_scan:
if check_port(host, port):
banner = grab_banner(host, port)
open_ports.append({'port': port, 'banner': banner})
print(f"  {host}:{port} OPEN  {banner[:50]}")

results[host] = open_ports

return results

# Usage
results = scan_network('192.168.1.0/24')
```

---

## 10. Wireless Network Analysis

### WiFi Capture Setup

```bash
# Check your WiFi adapter supports monitor mode
iw list | grep -A 10 "Supported interface modes"
# Must show: "monitor"

# Enable monitor mode
sudo ip link set wlan0 down
sudo iw wlan0 set monitor control
sudo ip link set wlan0 up

# Or use airmon-ng (easier)
sudo apt install aircrack-ng
sudo airmon-ng start wlan0
# Creates wlan0mon interface

# Capture all WiFi traffic
sudo wireshark -i wlan0mon
# or
sudo tcpdump -i wlan0mon -w wifi_capture.pcap
```

### Analyzing WiFi in Wireshark

```
Filter: wlan                      # All WiFi frames
Filter: wlan.fc.type == 0         # Management frames (connect/disconnect)
Filter: wlan.fc.type == 2         # Data frames
Filter: eapol                     # WPA handshakes (for cracking!)

# See all SSIDs being broadcast
tshark -r wifi.pcap -Y "wlan.fc.type_subtype == 0x08" \
-T fields -e wlan.ssid

# Capture WPA handshake (for testing on your own network)
# Filter: eapol
# This shows 4-way handshake between client and AP
```

### WPA Handshake Capture

```bash
# Find target network
sudo airodump-ng wlan0mon

# Capture handshake from a specific AP
sudo airodump-ng -c [channel] --bssid [AP_MAC] -w capture wlan0mon

# Wait for a client to connect naturally
# OR force reconnect with deauth (on networks you own!)
sudo aireplay-ng -0 5 -a [AP_MAC] -c [CLIENT_MAC] wlan0mon

# Crack captured handshake (testing only, on your own network!)
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap
```

---

## 11. Tools Reference

| Tool | Use | Install |
|---|---|---|
| **Wireshark** | GUI packet analysis | `apt install wireshark` |
| **tshark** | CLI packet analysis | Included with Wireshark |
| **tcpdump** | CLI packet capture | `apt install tcpdump` |
| **Scapy** | Python packet crafting | `pip install scapy` |
| **nmap** | Network scanning | `apt install nmap` |
| **Zeek (Bro)** | Network analysis framework | `apt install zeek` |
| **NetworkMiner** | Extract files from pcap | networkmniner.net |
| **mitmproxy** | HTTP(S) proxy | `pip install mitmproxy` |
| **aircrack-ng** | WiFi analysis | `apt install aircrack-ng` |
| **ngrep** | grep for network traffic | `apt install ngrep` |
| **strings** | Find text in pcap | `strings capture.pcap` |

### Useful Commands Quick Reference

```bash
# Capture
sudo tcpdump -i eth0 -w cap.pcap
sudo tcpdump -i eth0 -w cap.pcap 'port 80'

# Read and filter
tshark -r cap.pcap -Y "http"
tshark -r cap.pcap -T fields -e ip.src -e ip.dst

# Extract strings from pcap
strings cap.pcap | grep -E "password|token|Bearer"

# Follow streams
tshark -r cap.pcap -q -z follow,tcp,ascii,0

# Extract HTTP objects
tshark -r cap.pcap --export-objects http,./extracted/

# DNS analysis
tshark -r cap.pcap -Y dns -T fields -e dns.qry.name | sort | uniq -c
```

---

## 12. Practice Labs

### Practice Environments

| Resource | URL | What You Get |
|---|---|---|
| **Malware Traffic Analysis** | malware-traffic-analysis.net | Real malware pcap exercises! |
| **PacketTotal** | packettotal.com | Analyze pcap files online |
| **CloudShark** | cloudshark.org | Cloud-based pcap analysis |
| **Wireshark Sample Captures** | wiki.wireshark.org/SampleCaptures | Practice pcaps |
| **PicoCTF** | picoctf.org | Network forensics challenges |
| **HackTheBox** | hackthebox.com | Network RE challenges |

### Suggested Practice Path

```
Week 1: Wireshark Basics
→ Install Wireshark, capture your own traffic
→ Filter by HTTP, DNS, TCP
→ Follow TCP streams, find credentials in cleartext HTTP
→ Download pcap samples from Wireshark wiki

Week 2: Protocol Analysis
→ Download malware pcaps from malware-traffic-analysis.net
→ Identify C2 traffic using filters
→ Find beaconing patterns
→ Extract files from HTTP traffic

Week 3: Python Networking
→ Build a port scanner with Scapy
→ Write a DNS query monitor
→ Parse a pcap file with Python

Week 4: Custom Protocols
→ Pick a simple game or app
→ Capture its traffic
→ Try to understand the protocol structure
→ Build a simple parser

Month 2+:
→ Do CTF network challenges on PicoCTF
→ Try HackTheBox network machines
→ Practice WiFi capture on your own home network
→ Build a beacon detector
```

---

## Quick Reference Cheatsheet

### Wireshark Filters
```
http                                    HTTP traffic
dns                                     DNS queries
tls.handshake.extensions_server_name   HTTPS domain names
tcp.flags.syn == 1                      New connections
!(ip.addr == 192.168.0.0/16)           External traffic
tcp.dstport == 4444                     Suspicious ports
dns.flags.rcode == 3                    Failed DNS lookups
```

### tcpdump Quick Capture
```bash
sudo tcpdump -i eth0 -w out.pcap              # Capture all
sudo tcpdump -i eth0 port 80 -A              # HTTP content
sudo tcpdump -i eth0 'udp port 53' -n        # DNS queries
sudo tcpdump -i eth0 'not port 22' -w out.pcap # Exclude SSH
```

### tshark Analysis
```bash
tshark -r cap.pcap -Y "http.request"          # HTTP requests
tshark -r cap.pcap -T fields -e dns.qry.name  # DNS names
tshark -r cap.pcap -z conv,tcp                # TCP conversations
tshark -r cap.pcap --export-objects http,./   # Extract files
```

---

*Part of the Complete Reverse Engineering Series*
*Next: Firmware RE*
````
