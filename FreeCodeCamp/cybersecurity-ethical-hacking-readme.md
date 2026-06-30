# Cybersecurity & Ethical Hacking with Kali Linux — Complete Course Notes

> A beginner-friendly, hands-on guide covering penetration testing, Linux fundamentals, wireless security, Wireshark, and Nmap.

---

## Table of Contents

1. [Course Overview](#1-course-overview)
2. [Kali Linux Fundamentals](#2-kali-linux-fundamentals)
   - [Opening the Terminal](#21-opening-the-terminal)
   - [Basic Commands](#22-basic-commands)
   - [File Navigation with `ls` and `cd`](#23-file-navigation-with-ls-and-cd)
   - [Text Editing with Nano](#24-text-editing-with-nano)
   - [The `cat` Command](#25-the-cat-command)
   - [Creating Directories with `mkdir`](#26-creating-directories-with-mkdir)
   - [Searching with `grep`](#27-searching-with-grep)
   - [Word Count with `wc`](#28-word-count-with-wc)
   - [Output Redirection](#29-output-redirection)
   - [Piping](#210-piping)
   - [Copying Files and Directories with `cp`](#211-copying-files-and-directories-with-cp)
   - [Deleting Files and Directories with `rm`](#212-deleting-files-and-directories-with-rm)
3. [Linux User Management](#3-linux-user-management)
   - [Types of Users](#31-types-of-users)
   - [The Root User](#32-the-root-user)
   - [The `sudo` Command](#33-the-sudo-command)
4. [Networking Basics](#4-networking-basics)
   - [The `ip addr` Command](#41-the-ip-addr-command)
5. [Package Management with `apt`](#5-package-management-with-apt)
   - [Installing Packages](#51-installing-packages)
   - [Removing Packages](#52-removing-packages)
6. [Nmap — Network Scanning](#6-nmap--network-scanning)
7. [Wireless Security](#7-wireless-security)
   - [Setting Up Your Wireless Adapter](#71-setting-up-your-wireless-adapter)
   - [Managed vs Monitor Mode](#72-managed-vs-monitor-mode)
   - [Enabling and Disabling Monitor Mode](#73-enabling-and-disabling-monitor-mode)
   - [Scanning Wi-Fi Networks with Airodump-ng](#74-scanning-wi-fi-networks-with-airodump-ng)
   - [Scanning 5 GHz Networks](#75-scanning-5-ghz-networks)
   - [The Four-Way Handshake](#76-the-four-way-handshake)
   - [Capturing the Four-Way Handshake](#77-capturing-the-four-way-handshake)
   - [Deauthentication Attack](#78-deauthentication-attack)
   - [Cracking Wi-Fi Passwords with Aircrack-ng](#79-cracking-wi-fi-passwords-with-aircrack-ng)
8. [Wireshark](#8-wireshark)
   - [Overview and Interface](#81-overview-and-interface)
   - [Display Filters](#82-display-filters)
   - [Capture Filters](#83-capture-filters)
   - [Detecting Deauthentication Attacks in Wireshark](#84-detecting-deauthentication-attacks-in-wireshark)

---

## 1. Course Overview

This course teaches **cybersecurity and ethical hacking from scratch** using **Kali Linux**. No prior knowledge is required. Topics covered include:

- Linux command-line fundamentals
- Linux administration (users, sudo, package management)
- Wireless (Wi-Fi) penetration testing and defense
- Wireshark for traffic analysis
- Nmap for network reconnaissance
- Password cracking techniques

**Instructor:** Sanim Malu — Cybersecurity Consultant and Reverse Engineer

> ⚠️ **Legal Disclaimer:** All techniques demonstrated in this course are for educational and ethical purposes only. Always obtain proper written authorization before testing any system or network you do not own.

---

## 2. Kali Linux Fundamentals

**Kali Linux** is an operating system purpose-built for penetration testing and cybersecurity tasks. It comes with 300+ pre-installed hacking and forensic tools, making it the industry standard for ethical hackers, security researchers, and pen testers.

---

### 2.1 Opening the Terminal

There are three ways to open the terminal in Kali Linux:

| Method | Steps |
|--------|-------|
| Keyboard shortcut | Press `Ctrl + Alt + T` |
| Application menu | Click Applications → Search "terminal" → Click Terminal Emulator |
| Taskbar | Click the terminal icon in the taskbar |

**Increase font size for readability:**
Go to **File → Preferences → Appearance → Change** → Increase font size (e.g., 13).

---

### 2.2 Basic Commands

#### `whoami` — Display current logged-in user
```bash
whoami
```

#### `hostname` — Display the system name
```bash
hostname
```

#### `date` — Print current date and time
```bash
date
```

#### `pwd` — Print Working Directory (show current location in file system)
```bash
pwd
```

#### `clear` — Clear the terminal screen
```bash
clear
```

#### `history` — Show all previously executed commands
```bash
history
```

> 💡 **Tip:** The `$` symbol in the terminal prompt represents a **regular (non-root) user**. The `#` symbol represents the **root user**.

---

### 2.3 File Navigation with `ls` and `cd`

#### `ls` — List files and directories

```bash
ls                    # List contents of current directory
ls -a                 # List all files including hidden ones (files starting with .)
ls desktop            # List contents of a specific directory
ls desktop /etc       # List contents of multiple directories
ls -l                 # Long listing (detailed view with size, date, permissions)
ls -lh                # Long listing with human-readable file sizes (KB, MB, GB)
ls -lha               # Combine all: long, human-readable, including hidden
```

> 💡 **Color coding:** Blue = directories, White = files (all file types)

#### `cd` — Change Directory

```bash
cd desktop            # Navigate into the Desktop directory
cd ..                 # Go one directory back
cd ../..              # Go two directories back
cd                    # Go directly to home directory (no argument)
cd -                  # Go back to the previous working directory
```

---

### 2.4 Text Editing with Nano

**Nano** is a lightweight, terminal-based text editor for creating, opening, and modifying text files.

#### Opening / Creating a file
```bash
nano                          # Open blank nano editor
nano filename.txt             # Open or create a specific file
```

If the file exists → it opens it. If it doesn't → nano creates it.

#### Key Nano Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + G` | Open help/documentation |
| `Ctrl + O` | Write out / Save file |
| `Ctrl + X` | Exit nano |
| `Enter` | New line |

#### Workflow Example
```bash
nano dummy.txt        # Create/open dummy.txt
# Type your content
# Press Ctrl+O → Enter file name → Press Enter to save
# Press Ctrl+X to exit
```

---

### 2.5 The `cat` Command

`cat` stands for **concatenation**. It is one of the most used Linux commands, capable of:
- Viewing file contents
- Creating files
- Appending content to files
- Merging multiple files

#### View file contents
```bash
cat dummy.txt             # Display contents of a file
cat -n dummy.txt          # Display contents with line numbers
cat dummy.txt d1.txt      # Display contents of multiple files
cat /etc/passwd           # View the user accounts file
```

#### Create a file
```bash
cat > newfile.txt         # Create a file and write content (Ctrl+C to exit)
```

> ⚠️ Using `>` **overwrites** existing content.

#### Append content to an existing file
```bash
cat >> existingfile.txt   # Append new content without overwriting
```

#### Merge/concatenate multiple files into one
```bash
cat d1.txt dummy.txt > merged.txt   # Merge two files into merged.txt
```

> 💡 The original files are **not modified** — their contents are only copied into the new file.

---

### 2.6 Creating Directories with `mkdir`

`mkdir` stands for **make directory**.

```bash
mkdir dummy                    # Create a single directory
mkdir d1 d2                    # Create multiple directories at once
mkdir -p sunny/dimalu          # Create nested directories (parent/child)
```

> The `-p` flag stands for **parent** — it creates intermediate directories as needed.

---

### 2.7 Searching with `grep`

`grep` stands for **Global Regular Expression Print**. It searches for text patterns within files — extremely useful for analyzing large logs and files.

```bash
grep "user" out.txt            # Search for the word "user" in out.txt
grep -n "user" out.txt         # Search and show line numbers
grep "dimalu" /etc/passwd      # Search for a user in the system accounts file
```

> 💡 Matching text is highlighted in red. If no match is found, grep produces no output.

**Practical use case:** Check if a user account exists:
```bash
grep "username" /etc/passwd
```

---

### 2.8 Word Count with `wc`

`wc` stands for **word count**. It counts lines, words, and characters in a file.

```bash
wc out.txt          # Show lines, words, and characters
wc -w out.txt       # Count words only
wc -l out.txt       # Count lines only
wc -c out.txt       # Count characters only
```

**Real-world example:** Count how many user accounts exist on the system:
```bash
wc -l /etc/passwd
```

---

### 2.9 Output Redirection

Output redirection allows you to save command output to a file instead of printing it to the terminal.

| Operator | Behavior |
|----------|----------|
| `>` | Redirect output, **overwrite** existing content |
| `>>` | Redirect output, **append** to existing content |

```bash
ls > out.txt            # Save ls output to out.txt (overwrites)
date > date.txt         # Save current date/time to date.txt
date >> date.txt        # Append current date/time to date.txt
```

> ⚠️ Using `>` on an existing file **deletes** its old contents. Use `>>` to preserve them.

---

### 2.10 Piping

**Piping** sends the output of one command directly as input to another command. It is represented by the `|` (pipe) character.

```bash
# Send output of cat to grep
cat out.txt | grep "user"

# Check if a directory exists among many
ls | grep "desktop"

# Count user accounts
cat /etc/passwd | wc -l
```

> 💡 Most Linux commands are designed to receive input via piping. This allows you to chain powerful operations without intermediate files.

---

### 2.11 Copying Files and Directories with `cp`

`cp` stands for **copy**.

```bash
# Copy a file (creates a copy with a new name)
cp dummy.txt dummy_copy.txt

# Copy a file to another directory
cp dummy.txt /home/dimalu/downloads/

# Copy a file to another directory with a new name
cp dummy.txt /home/dimalu/downloads/dummy_copy.txt

# Copy a directory (requires -r flag)
cp -r demo demo2

# Copy a directory to another location
cp -r demo /home/dimalu/downloads/

# Copy multiple files/directories at once
cp -r file1.txt dir1 /home/dimalu/downloads/
```

> ⚠️ The `-r` flag (recursive) is **required** when copying directories.

---

### 2.12 Deleting Files and Directories with `rm`

`rm` stands for **remove**.

```bash
# Remove a file
rm dummy.txt

# Remove multiple files
rm file1.txt file2.txt

# Remove a file in a specific directory
rm /home/dimalu/downloads/dummy.txt

# Remove a directory (requires -r flag)
rm -r demo

# Force remove (bypasses confirmation prompts)
rm -f protected_file.txt

# Force remove a write-protected directory
rm -rf protected_directory/
```

> ⚠️ **Be very careful with `rm -rf`** — it permanently deletes files and directories without any warning. There is no recycle bin or undo.

---

## 3. Linux User Management

### 3.1 Types of Users

Linux (and Windows) have three types of users:

#### 1. Regular Users (Standard / Normal Users)
- Can perform basic tasks: browse internet, create files, play media
- **Cannot** perform administrative tasks (install software, modify system files)
- Each user has their own isolated home directory
- Represented by the `$` symbol in the terminal

#### 2. Root User (Administrative User / Superuser)
- Has **complete control** over the system
- Can install/uninstall applications, modify any system file, manage all users
- Created automatically during OS installation
- Represented by the `#` symbol in the terminal

#### 3. System Users
- Created during OS/application installation
- Run services and daemons in the background
- Not used for day-to-day tasks
- Cannot log in directly (no login shell)

---

### 3.2 The Root User

The **root user** is the most powerful account on a Linux system. It can perform any operation without restriction.

> ⚠️ **Why logging in as root is dangerous:**
> - Commands execute without warnings — even destructive ones
> - Accidental deletion of critical system files can render the system unusable
> - Even experienced users avoid logging in as root unless absolutely necessary

---

### 3.3 The `sudo` Command

`sudo` stands for **Super User Do**. It allows regular users to run specific commands with root-level privileges — without logging in as root.

**Syntax:**
```bash
sudo <command>
```

**Examples:**
```bash
sudo apt install nmap       # Install a package (requires admin rights)
sudo fdisk -l               # List disk partitions
sudo airmon-ng              # Run wireless tools that require root
```

**Check if a user has sudo privileges:**
```bash
id username
```
Look for `sudo` in the `groups=` section of the output.

> 💡 Only the user account created during OS installation has sudo access by default. Users added later do not have sudo access unless explicitly granted.

---

## 4. Networking Basics

### 4.1 The `ip addr` Command

Used to display information about all network interfaces (Ethernet, wireless, loopback).

```bash
ip addr show      # Full command
ip addr           # Same result
ip a              # Shorthand
```

**Output breakdown:**

| Field | Meaning |
|-------|---------|
| `lo` | Loopback interface (localhost `127.0.0.1`) — not a physical adapter |
| `eth0` | Ethernet interface — your wired network card |
| `wlan0` | Wireless interface — your Wi-Fi card |
| `inet` | IPv4 address |
| `inet6` | IPv6 address |
| `ether` | MAC address (hardware address) |
| `brd` | Broadcast address |

> 💡 Virtual machines cannot access the host's built-in wireless card. An **external USB wireless adapter** is required for wireless penetration testing inside a VM.

---

## 5. Package Management with `apt`

`apt` (Advanced Package Tool) is used to install, update, and remove software packages.

### 5.1 Installing Packages

```bash
# Step 1: Always update repositories first
sudo apt update

# Step 2: Install the desired package
sudo apt install <package-name>

# Example
sudo apt install htop
```

> 💡 Run `apt update` before installing to ensure you get the latest available version.

### 5.2 Removing Packages

```bash
# Remove the main application
sudo apt remove <package-name>

# Remove leftover dependencies no longer needed
sudo apt autoremove
```

> Always run `apt autoremove` after `apt remove` to clean up orphaned dependencies.

---

## 6. Nmap — Network Scanning

**Nmap** (Network Mapper) is an advanced information-gathering tool used to discover open ports, running services, and vulnerabilities on target systems.

```bash
# View all available options
nmap --help

# Ping scan — check if a host is up
nmap -SP scanme.nmap.org

# Default scan (top 1000 ports)
nmap scanme.nmap.org

# Scan using IP address
nmap 45.33.32.156
```

**Understanding scan results:**

| Port State | Meaning |
|------------|---------|
| `open` | Port is actively accepting connections |
| `filtered` | Port is likely protected by a firewall |
| `closed` | Port is accessible but no service is listening |

**Common services by port:**

| Port | Service | Notes |
|------|---------|-------|
| 22 | SSH | Encrypted remote access |
| 21 | FTP | Unencrypted file transfer |
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted web traffic |
| 25 | SMTP | Email transmission |

> ⚠️ **Legal Warning:** Scanning networks or systems without explicit written permission is **illegal** in most jurisdictions. Use `scanme.nmap.org` (provided by Nmap) for practice, or use your own systems.

---

## 7. Wireless Security

### 7.1 Setting Up Your Wireless Adapter

A **dedicated external USB wireless adapter** is required for Wi-Fi penetration testing inside a virtual machine.

**Recommended compatible cards** (Kali Linux compatible, no manual drivers needed):
- Alpha series cards with Atheros chipset
- Cards supporting monitor mode and packet injection

**Steps to connect your adapter to Kali Linux VM:**

1. Plug in the wireless adapter
2. In VirtualBox: **Devices → USB → Select your wireless card**
3. Wait 15–20 seconds for setup
4. Verify detection:
```bash
ip a
# Look for wlan0 in the output
```

**Verify detection with Airmon-ng:**
```bash
sudo airmon-ng
```

---

### 7.2 Managed vs Monitor Mode

| Mode | Description |
|------|-------------|
| **Managed Mode** (default) | Standard mode for connecting to Wi-Fi networks and accessing the internet |
| **Monitor Mode** | Captures ALL wireless traffic within range — required for Wi-Fi penetration testing |

> 💡 In monitor mode, you cannot connect to Wi-Fi networks. It is exclusively used for packet capture and analysis.

---

### 7.3 Enabling and Disabling Monitor Mode

#### Check for interfering processes and kill them
```bash
sudo airmon-ng check kill
```

#### Enable monitor mode
```bash
sudo airmon-ng start wlan0
```
This creates a new interface: `wlan0mon`

#### Verify monitor mode is active
```bash
ip a                  # Look for wlan0mon
iwconfig              # Should show "Mode:Monitor"
```

#### Disable monitor mode (return to managed)
```bash
sudo airmon-ng stop wlan0mon
```

#### Restart network manager after returning to managed mode
```bash
sudo systemctl restart NetworkManager
```

> **Note:** `N` in `NetworkManager` and `M` must both be uppercase.

---

### 7.4 Scanning Wi-Fi Networks with Airodump-ng

**Airodump-ng** is part of the Aircrack-ng suite. It captures wireless packets and discovers nearby Wi-Fi networks.

#### Scan all networks
```bash
sudo airodump-ng wlan0mon
```

#### Understanding the output columns

| Column | Meaning |
|--------|---------|
| `BSSID` | MAC address of the wireless access point/router |
| `PWR` | Signal strength (lower = stronger: <40 strong, 40–60 average, >70 weak) |
| `Beacons` | Beacon packets sent by the AP (~10/second) announcing its presence |
| `#Data` | Number of data packets captured |
| `#/s` | Data packets per second (last 10 seconds average) |
| `CH` | Channel number the network is broadcasting on |
| `MB` | Maximum speed supported |
| `ENC` | Encryption type (WPA2, WPA3, OPN) |
| `CIPHER` | Encryption algorithm (CCMP for WPA2) |
| `AUTH` | Authentication method (PSK = Pre-Shared Key/password) |
| `ESSID` | Name (SSID) of the Wi-Fi network |

#### Bottom section: Connected devices

| Column | Meaning |
|--------|---------|
| `BSSID` | MAC of the router the device is connected to |
| `STATION` | MAC address of the connected client device |
| `Not associated` | Device is searching for a network but not connected |

#### Save captured packets to a file
```bash
sudo airodump-ng wlan0mon --write captured_packets
```

Files saved with extensions: `.cap`, `.csv`, `.netxml`, `.kismet.csv`

> The `.cap` file is the one used for password cracking and can be opened in Wireshark.

#### Open captured file in Wireshark
```bash
wireshark captured_packets-01.cap
```

---

### 7.5 Scanning 5 GHz Networks

By default, Airodump-ng scans only the **2.4 GHz** band. Use the `--band` option for 5 GHz.

> ⚠️ Your wireless adapter must **support 5 GHz** for this to work.

```bash
# Scan 5 GHz only
sudo airodump-ng --band a wlan0mon

# Scan 2.4 GHz only (explicit)
sudo airodump-ng --band g wlan0mon

# Scan both 2.4 GHz and 5 GHz simultaneously
sudo airodump-ng --band ag wlan0mon
```

**Band reference:**

| Band Value | Frequency | Channel Range |
|------------|-----------|---------------|
| `a` | 5 GHz | 36–165 |
| `g` | 2.4 GHz | 1–14 |
| `b` | 2.4 GHz (older 802.11b) | 1–14 |

> 💡 Scanning multiple bands increases CPU/RAM load and may slow down scanning.

---

### 7.6 The Four-Way Handshake

The **four-way handshake** is the process that occurs when a client device connects to a Wi-Fi network. It consists of an exchange of **4 packets** between the client and the wireless access point (router).

**Purpose:**
- Generates **session keys** (encryption keys) used to encrypt all data during the session
- Establishes a secure, encrypted channel between the device and the router

**Why it matters for hacking:**
- The handshake contains all information needed to begin password cracking
- The encryption keys are derived from: Wi-Fi password + SSID + MAC addresses + nonce values
- By capturing the handshake, attackers can attempt to crack the password offline

**How it can be captured:**
- A new device connects to the network
- An existing device disconnects and reconnects

---

### 7.7 Capturing the Four-Way Handshake

#### Step 1: Discover target network details
```bash
sudo airodump-ng wlan0mon
# Note down: BSSID (MAC address) and CH (channel number)
```

#### Step 2: Monitor target network and save packets
```bash
sudo airodump-ng --bssid <TARGET_MAC> -c <CHANNEL> --write handshake wlan0mon
```

**Example:**
```bash
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF -c 9 --write handshake wlan0mon
```

#### Step 3: Wait for a device to connect (or force reconnection via deauth attack)

When a device connects, you'll see:
```
WPA handshake: AA:BB:CC:DD:EE:FF
```

#### Step 4: Verify the handshake in Wireshark
```bash
wireshark handshake-01.cap
```
In the Wireshark filter bar, type:
```
EAPOL
```
A valid handshake shows **exactly 4 EAPOL packets**.

---

### 7.8 Deauthentication Attack

#### What is it?

A **deauthentication attack** forcibly disconnects devices from a Wi-Fi network by sending fake deauthentication frames. Since most devices reconnect automatically, this triggers a new four-way handshake — which can then be captured.

#### How it works

1. Attacker spoofs the router's MAC address
2. Sends deauthentication frames to connected devices on behalf of the router
3. Devices disconnect (they cannot verify the frame's authenticity)
4. Devices reconnect automatically → four-way handshake is captured

#### Step 1: Set your card to the target channel
```bash
sudo iwconfig wlan0mon channel <CHANNEL_NUMBER>
```

#### Step 2: Launch the deauthentication attack with Aireplay-ng
```bash
# Send 100 deauth packets
sudo aireplay-ng --deauth 100 -a <TARGET_MAC> wlan0mon

# Send unlimited deauth packets (stop manually with Ctrl+C)
sudo aireplay-ng -0 0 -a <TARGET_MAC> wlan0mon

# Send a specific number
sudo aireplay-ng -0 50 -a <TARGET_MAC> wlan0mon
```

#### Alternative: Using MDK4
```bash
sudo mdk4 wlan0mon d -c <CHANNEL_NUMBER>
```

> ⚠️ A deauthentication attack is a **denial of service** — it prevents devices from using the network while active. Using this on unauthorized networks is **illegal**.

---

### 7.9 Cracking Wi-Fi Passwords with Aircrack-ng

#### Understanding the attacks

**Dictionary Attack:**
- Uses a pre-built wordlist of common/leaked passwords
- Hashes each password and compares it to the captured handshake
- Fast, but only works if the password is in the wordlist

**Brute Force Attack:**
- Tries every possible character combination
- Guaranteed to crack any password eventually
- Very slow for complex passwords

#### The rockyou.txt Wordlist

Kali Linux includes **rockyou.txt** — one of the most popular wordlists, containing **14+ million real-world leaked passwords**.

```bash
# Navigate to wordlists directory
cd /usr/share/wordlists

# Unzip rockyou.txt (it comes compressed)
sudo gunzip rockyou.txt.gz

# Count passwords in the list
cat rockyou.txt | wc -l

# Preview first 100 passwords
cat rockyou.txt | head -n 100
```

#### Crack the Wi-Fi password
```bash
sudo aircrack-ng <handshake.cap> -w /usr/share/wordlists/rockyou.txt
```

**Full example:**
```bash
sudo aircrack-ng handshake-01.cap -w /usr/share/wordlists/rockyou.txt
```

If successful, you'll see:
```
KEY FOUND! [ your_wifi_password ]
```

#### Password strength guide

| Password Type | Crackability |
|--------------|--------------|
| Numbers only | Very easy — cracked in minutes |
| Common words | Easy — likely in rockyou.txt |
| Mixed words + numbers | Moderate |
| Numbers + letters + symbols + uppercase/lowercase | Very hard to near impossible |

> 💡 **Defense Tip:** Use a long password (12+ characters) with a mix of uppercase, lowercase, numbers, and special characters. This makes dictionary attacks virtually useless and brute force impractical.

---

## 8. Wireshark

### 8.1 Overview and Interface

**Wireshark** is a free, open-source network packet analyzer used for:
- Real-time network traffic monitoring
- Security threat detection (hacking attempts, unauthorized access)
- Network troubleshooting (dropped packets, slow performance)
- Protocol analysis for developers

#### Interface Panels

| Panel | Description |
|-------|-------------|
| **Packet List** | All captured packets listed chronologically |
| **Packet Details** | Detailed breakdown of the selected packet |
| **Packet Bytes** | Raw hexadecimal/binary data of the selected packet |
| **Filter Bar** | Where you type display/capture filters |
| **Status Bar** | Summary of captured and dropped packets |

#### Column meanings

| Column | Meaning |
|--------|---------|
| `No.` | Packet number |
| `Time` | Time the packet was captured |
| `Source` | IP address of the sender |
| `Destination` | IP address of the receiver |
| `Protocol` | Protocol type (TCP, UDP, DNS, HTTP, etc.) |
| `Length` | Packet size in bytes |
| `Info` | Brief summary of the packet |

#### Useful Toolbar Buttons

| Button | Action |
|--------|--------|
| Red square | Stop current capture session |
| First button | Start a new capture session |
| Third button | Restart current session |
| Capture Options | Configure interfaces and filters before capture |
| Open file | Open a previously saved `.pcap` or `.pcapng` file |
| Save | Save current captured packets |
| Find | Search packets by string, hex value, or filter |
| Arrow keys | Navigate packets (up = first, down = last) |

#### Changing time display format
Go to **View → Time Display Format → Time of Day → Seconds**

---

### 8.2 Display Filters

Display filters are applied **after** capture — they show/hide packets in the packet list without deleting any data.

#### Filter by protocol
```
http          # HTTP traffic
dns           # DNS traffic
tcp           # TCP traffic
udp           # UDP traffic
arp           # ARP packets
tls           # Encrypted HTTPS traffic (TLS)
icmp          # ICMP/ping traffic
```

#### Filter by port number
```
tcp.port == 80       # HTTP (port 80)
tcp.port == 443      # HTTPS (port 443)
tcp.port == 22       # SSH (port 22)
tcp.port eq 21       # FTP (eq is the same as ==)
```

#### Filter by IP address
```
ip.addr == 192.168.1.100        # All traffic to/from this IP
ip.addr == 192.168.1.1 or ip.addr == 192.168.1.5    # Multiple IPs
not ip.addr == 192.168.1.100    # Exclude a specific IP
```

#### Combining filters with operators

| Operator | Symbol | Example |
|----------|--------|---------|
| OR | `or` or `\|\|` | `dns or arp` |
| AND | `and` or `&&` | `tcp and ip.addr == 192.168.1.1` |
| NOT | `not` or `!` | `not tcp` / `!tcp` |

#### Search for keyword in packets
```
dns contains "youtube"          # Find DNS packets mentioning YouTube
tcp contains "payload"          # Find TCP packets with the word "payload"
```

> 💡 **Filter color codes:** Green background = valid filter, Pink background = invalid filter

---

### 8.3 Capture Filters

Capture filters are set **before** starting a session — they control what Wireshark captures in the first place.

> ⚠️ Capture filter syntax is **different** from display filter syntax.

```
port 80                    # Capture only HTTP traffic
port 21                    # Capture only FTP traffic
port 22 or port 21         # Capture SSH and FTP
tcp                        # Capture only TCP traffic
udp                        # Capture only UDP traffic
host 192.168.1.100         # Capture traffic to/from a specific IP
```

**Where to enter capture filters:**
- On the welcome screen, in the filter box below the interface list
- In **Capture → Capture Options → selected interface filter box**

---

### 8.4 Detecting Deauthentication Attacks in Wireshark

#### Step 1: Put your wireless card in monitor mode
```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
```

#### Step 2: Start Wireshark on the monitor interface
Open Wireshark → Double-click `wlan0mon`

#### Step 3: Stop capture and apply the deauthentication filter
In the filter bar, type:
```
wlan.fc.type_subtype == 12
```

> Subtype `12` = Deauthentication frames  
> Subtype `11` = Authentication frames

#### Step 4: Analyze results

- **Normal behavior:** A router sends at most ~50 deauthentication frames during maintenance
- **Attack indicator:** Thousands of deauthentication frames in a short period = active deauthentication attack

#### Step 5: Identify the attacker's MAC address
Click on any deauthentication packet → Expand **802.11 Deauthentication flags** → Look at **Transmitter address**

> ⚠️ The transmitter MAC will be spoofed to match the router's MAC address. The real attacker is the one sending these frames on the router's behalf.

---

## Quick Reference: Essential Commands

```bash
# Navigation
ls -lha                          # List all files with details
cd /path/to/directory            # Change directory
pwd                              # Show current location

# File operations
cat file.txt                     # View file
nano file.txt                    # Edit file
cp source dest                   # Copy file
rm file.txt                      # Delete file
mkdir -p dir/subdir              # Create directories

# Searching
grep -n "keyword" file.txt       # Search in file with line numbers
ls | grep "name"                 # Search in directory listing

# System
sudo command                     # Run as root
id username                      # Check user/group info
ip a                             # Show network interfaces

# Package management
sudo apt update                  # Refresh package list
sudo apt install package         # Install a package
sudo apt remove package          # Remove a package
sudo apt autoremove              # Clean up unused dependencies

# Wireless
sudo airmon-ng check kill        # Kill interfering processes
sudo airmon-ng start wlan0       # Enable monitor mode
sudo airodump-ng wlan0mon        # Scan all Wi-Fi networks
sudo airmon-ng stop wlan0mon     # Disable monitor mode

# Password cracking
sudo aircrack-ng handshake.cap -w /usr/share/wordlists/rockyou.txt
```

---

## Learning Path Recommendations

After completing this course, consider progressing through:

1. **TryHackMe** — Beginner-friendly, guided rooms for hands-on practice
2. **HackTheBox** — Intermediate to advanced CTF-style machines
3. **Bug Bounty Platforms** — Real-world vulnerability disclosure programs (HackerOne, Bugcrowd)

---

*Course by Sanim Malu | Notes compiled from full course transcript*
