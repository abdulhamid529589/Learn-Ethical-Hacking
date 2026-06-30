# Day 24 — Anonymity: Staying Anonymous While Hacking

> 📚 Part of the **"Learn Ethical Hacking in 30 Days"** series.  
> Previous: [Day 23 — Clearing Tracks](#) | Next: [Day 25](#)

---

## 📋 Table of Contents

1. [Why Anonymity Matters](#1-why-anonymity-matters)
2. [Proxies](#2-proxies)
3. [VPN (Virtual Private Network)](#3-vpn-virtual-private-network)
4. [Tor (The Onion Router)](#4-tor-the-onion-router)
5. [Proxy vs VPN vs Tor — Full Comparison](#5-proxy-vs-vpn-vs-tor--full-comparison)
6. [Tool: ProxyChains](#6-tool-proxychains)
7. [Tool: macchanger (MAC Address Spoofing)](#7-tool-macchanger-mac-address-spoofing)
8. [Quick Reference Cheat Sheet](#8-quick-reference-cheat-sheet)

---

## 1. Why Anonymity Matters

Before hacking (recon, scanning, exploitation), your real IP address can be logged by:
- Target web servers
- Firewalls and IDS/IPS systems
- ISPs (Internet Service Providers)
- Network administrators

**Goal:** Mask your true identity so your activities cannot be traced back to you.

> ⚠️ **In authorized penetration testing:** Anonymity tools help simulate real-world attacker behavior. Always have written permission before testing any system.

---

## 2. Proxies

### What is a Proxy?

A proxy is a **middleman server** that sits between you and the internet. Instead of connecting directly to a website, your request goes through the proxy server first.

```
Without proxy:
[You] → [ISP] → [Website]
         ↑ website sees YOUR IP

With proxy (forward):
[You] → [Proxy Server] → [Website]
                  ↑ website sees PROXY's IP, not yours
```

> 🧑‍🤝‍🧑 **Analogy:** Instead of delivering a letter yourself, you give it to a friend to deliver. The recipient sees your friend's identity, not yours.

### Types of Proxies

| Type | Description | Use Case |
|------|-------------|----------|
| **Forward Proxy** | Client-side — forwards your requests to the internet | Anonymize client traffic, bypass geo-blocks |
| **Reverse Proxy** | Server-side — sits in front of servers, handles incoming requests | Load balancing, hide internal servers from internet |

### Forward Proxy Sub-types

| Type | Description |
|------|-------------|
| **Data Center Proxy** | IP belongs to a data center (AWS, GCP, etc.) — easily identified as non-residential |
| **Residential Proxy** | IP belongs to a real home internet connection — looks like normal user traffic |

> 💡 **Residential proxies** are harder to detect and block because the traffic looks like it comes from a real home user. However, they are expensive.

### Proxies for Hacking — Limitations

- Most proxies provide no encryption — your traffic is still visible
- Transparent proxies can leak your real IP via headers
- Not reliable for true anonymity in high-stakes scenarios

---

## 3. VPN (Virtual Private Network)

### What is a VPN?

A VPN creates an **encrypted tunnel** between your device and a VPN server. All traffic passes through this tunnel before reaching the internet.

```
Without VPN:
[You] → [ISP] → [Internet]
         ↑ ISP can see which sites you visit

With VPN:
[You] → [Encrypted Tunnel] → [VPN Server] → [Internet]
    ↑                    ↑
  VPN client        ISP cannot see inside tunnel
  encrypts          (only sees encrypted blob)
```

### How VPN Works

1. VPN client software is installed on your device
2. When you connect, an **encrypted tunnel** is created between you and the VPN server
3. All your traffic goes through this tunnel → decrypted at the VPN server → forwarded to the internet
4. Websites see the **VPN server's IP**, not your real IP
5. Your ISP sees encrypted traffic going to the VPN server — not what websites you visit

### VPN for Hacking — Limitations

- The VPN provider **can see your traffic** (it decrypts at their server)
- If law enforcement contacts the VPN company, logs may be handed over
- Payment method can be traced (credit card, bank details)
- VPN IPs are publicly known — target servers may block them

> 💡 **You're shifting trust from your ISP to your VPN provider.** If the VPN company cooperates with law enforcement, you can be identified. Several hackers have been caught this way.

### VPN Best Use Cases

- Secure browsing on public WiFi (coffee shop, hotel)
- Protect against ISP surveillance
- Bypass geo-restrictions
- Encrypt traffic on untrusted networks (MITM prevention)

---

## 4. Tor (The Onion Router)

### What is Tor?

**Tor** (The Onion Router) is a **free, open-source** anonymity network that routes your traffic through multiple volunteer-operated nodes around the world.

### How Tor Works

Traffic passes through **3 nodes** (minimum):

```
[You] → [Entry Node] → [Middle Node] → [Exit Node] → [Website]
```

Each hop:
- **Removes one layer of encryption** (like peeling an onion)
- Only knows the **previous** and **next** hop
- Does NOT know the full route

```
Entry Node knows:  Your IP → but not the destination
Middle Node knows: Entry Node → Exit Node, but not you or destination
Exit Node knows:   The destination → but not your real IP
```

**Result:** No single node knows both who you are AND where you're going.

### Why Tor is Hard to Trace

- 3+ layers of encryption (one per node)
- Nodes run by volunteers in different countries
- No logs kept (ideally)
- The source and destination are never known to the same party

> 🔑 **Historical note:** Tor has never been broken cryptographically. Users who have been caught on Tor made **operational security (OpSec) mistakes** — revealing their identity through other means (posting personal info, using real payment methods, etc.) — not because Tor itself was compromised.

### Tor Limitations

- **Slow** — traffic bounces through multiple nodes with multiple encryption/decryption operations
- Exit nodes are publicly listed — some websites block Tor exit IPs
- Not suitable for high-bandwidth activities (streaming, large file transfers)

---

## 5. Proxy vs VPN vs Tor — Full Comparison

| Aspect | Proxy | VPN | Tor |
|--------|-------|-----|-----|
| **How it works** | Forwards requests as middleman | Encrypted tunnel to VPN server | Routes through 3+ nodes with layered encryption |
| **Anonymity** | Masks IP only — not guaranteed | Masks IP, but VPN provider can see traffic | Highest anonymity — no single point knows full route |
| **Encryption** | Usually none | ✅ Encrypts all traffic | ✅ Multi-layer encryption |
| **ISP can see?** | Your IP goes to proxy | Encrypted blob going to VPN server | Encrypted blob going to Tor entry node |
| **Provider can see?** | Proxy server sees your traffic | ✅ VPN provider can see decrypted traffic | ❌ No single node sees full picture |
| **Speed** | Fast | Good (slight overhead) | Slow (multiple hops) |
| **Setup complexity** | Easy (browser settings) | Easy (install client app) | Medium (Tor Browser or configure system) |
| **Best for** | Bypassing geo-blocks | Secure browsing, public WiFi | Maximum anonymity, sensitive activities |
| **For hacking** | ⚠️ Risky (may leak IP) | ⚠️ Risky (provider has logs) | ✅ Best option (no central point of failure) |
| **Cost** | Free or paid | Usually paid | Free |

### Recommendation for Hacking

```
🥇 Best: Tor (or ProxyChains + Tor)
   → No central provider to subpoena
   → Cryptographically strong anonymity
   → Slow but most secure

🥈 Acceptable: VPN (with strict no-logs policy, good jurisdiction)
   → Better than nothing, but you're trusting the VPN company

🥉 Avoid: Free/unknown proxies
   → May leak your real IP
   → Often transparent (no actual anonymity)
```

---

## 6. Tool: ProxyChains

**ProxyChains** is a Kali Linux built-in tool that routes any command's network traffic through a chain of proxies (including Tor).

### Installation

```bash
# Pre-installed on Kali Linux
proxychains4 --help

# Install on other systems:
sudo apt install proxychains4
```

### Configuration File

```bash
sudo nano /etc/proxychains4.conf
```

### Chain Types (configure in the conf file)

| Chain Type | How it works | When to use |
|-----------|-------------|-------------|
| **dynamic_chain** | Goes through proxies in order; skips dead ones | ✅ Recommended — resilient |
| **strict_chain** | Goes through ALL proxies in order; fails if any one is dead | When you need guaranteed routing |
| **random_chain** | Picks a random proxy from the list per connection | Testing IDS detection |

### Setting up ProxyChains with Tor

**Step 1: Edit the config file**

```bash
sudo nano /etc/proxychains4.conf
```

**Step 2: Uncomment `dynamic_chain` (remove the `#`)**

```
# Remove # from:
dynamic_chain
```

**Step 3: Add Tor as a proxy at the bottom of the file**

```
socks5  127.0.0.1  9050
```

Format: `<type> <ip> <port> [username] [password]`

| Field | Value |
|-------|-------|
| Type | `socks5` |
| IP | `127.0.0.1` (localhost — Tor runs locally) |
| Port | `9050` (Tor's default port) |

**Step 4: Save and exit**

Press `Ctrl+X` → `Y` → `Enter`

**Step 5: Start the Tor service**

```bash
sudo systemctl start tor
sudo systemctl status tor   # verify it's running (should say "active")
```

### Using ProxyChains

**Syntax:**
```bash
proxychains4 <command>
```

**Browse Firefox anonymously through Tor:**
```bash
proxychains4 firefox &
```

Then visit `https://whatismyipaddress.com` — you'll see Tor's IP, not yours.

**Run nmap through Tor (anonymized scan):**
```bash
proxychains4 nmap -sT -p 22 scanme.nmap.org
```

> ⚠️ **Note:** Some websites block Tor exit nodes. If a site blocks you, it's because they detect Tor IPs (publicly listed) and have blacklisted them.

### Adding Custom Proxies

In `/etc/proxychains4.conf`, add multiple proxies:

```
# Format: type  host  port  [user]  [pass]
socks5  127.0.0.1  9050        ← Tor
socks4  proxy2.example.com  1080
http    proxy3.example.com  8080  username  password
```

ProxyChains will route through them in order (dynamic_chain skips dead ones).

---

## 7. Tool: macchanger (MAC Address Spoofing)

### What is a MAC Address?

**MAC (Media Access Control) address** is a unique hardware identifier assigned to your network interface card (NIC).

| IP Address | MAC Address |
|-----------|------------|
| Changes (DHCP assigns dynamically) | Permanent (hardcoded in hardware) |
| Layer 3 (Network layer) | Layer 2 (Data link layer) |
| Used for internet routing | Used for local network communication |
| Can be from anywhere | Tied to your physical device |

**Why spoof MAC?**
- When you connect to a WiFi/router, your MAC address is logged
- Forensic investigators can trace the physical device from the MAC
- Some networks use **MAC filtering** (whitelist specific MACs) — spoofing bypasses this

### Check Current MAC Address

```bash
ip addr
# or
ifconfig
```

Look for the line showing `ether` — that's your MAC address.

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    link/ether 08:00:27:22:06:06 brd ff:ff:ff:ff:ff:ff
                 ↑
           your MAC address
```

### macchanger Commands

```bash
# Show current MAC address
macchanger --show eth0

# Set a completely random MAC address
sudo macchanger -r eth0

# Set a specific MAC address
sudo macchanger -m 00:11:22:33:44:55 eth0

# Reset to original/permanent MAC
sudo macchanger -p eth0
```

### Step-by-Step: Spoof MAC Address

```bash
# Step 1: Bring the interface down (required before changing MAC)
sudo ip link set eth0 down

# Step 2: Change to random MAC
sudo macchanger -r eth0

# Step 3: Bring the interface back up
sudo ip link set eth0 up

# Step 4: Verify the change
ip addr show eth0
```

**Example output:**
```
Current MAC:   08:00:27:22:06:06 (original)
Permanent MAC: 08:00:27:22:06:06
New MAC:       36:52:96:aa:3e:08 (random)
```

### Use Cases

| Scenario | Why spoof MAC |
|----------|--------------|
| **Connected to target network** | Your real MAC gets logged in router — spoof to hide device identity |
| **MAC filtering bypass** | Clone an authorized device's MAC to gain network access |
| **Forensic evasion** | Prevent physical device identification after the fact |

---

## 8. Quick Reference Cheat Sheet

### ProxyChains Setup (with Tor)

```bash
# 1. Edit config
sudo nano /etc/proxychains4.conf

# 2. Enable dynamic_chain (uncomment it)
# 3. Add at the bottom:
socks5 127.0.0.1 9050

# 4. Start Tor
sudo systemctl start tor
sudo systemctl status tor

# 5. Use any tool through ProxyChains
proxychains4 firefox &
proxychains4 nmap -sT -p 80 target.com
proxychains4 curl https://whatismyipaddress.com
```

### MAC Address Spoofing

```bash
# View current MAC
ip addr show eth0

# Random MAC spoof
sudo ip link set eth0 down
sudo macchanger -r eth0
sudo ip link set eth0 up
ip addr show eth0      # verify

# Specific MAC
sudo macchanger -m 00:11:22:33:44:55 eth0

# Restore original
sudo macchanger -p eth0
```

### Anonymity Tool Decision Guide

```
Bypassing geo-blocks only?
  → Simple Proxy or VPN

Secure browsing on public WiFi?
  → VPN

High anonymity for sensitive research?
  → Tor Browser

Running hacking tools anonymously?
  → ProxyChains + Tor

Connected to target local network?
  → Spoof MAC with macchanger first
```

### ProxyChains Chain Types

```
dynamic_chain  → Skips dead proxies, continues with live ones (recommended)
strict_chain   → All proxies must work, fails if any is dead
random_chain   → Picks random proxy per connection (IDS testing)
```

---

> ✅ **End of Day 24 Notes**  
> 📌 Next up: More advanced ethical hacking topics  
> 💡 Practice: Run `proxychains4 firefox` and check your IP on whatismyipaddress.com  
> ⚠️ Use these tools only for legitimate security research and authorized penetration testing  
> 💬 Questions? Drop them in the comments!