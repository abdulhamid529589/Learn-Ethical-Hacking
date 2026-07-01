# Wireshark & Packet Analysis — Course Notes

## https://youtu.be/eKVFWoMyMH0

> Source: Cybersecurity course video (Packet Sniffing & Network Traffic Analysis with Wireshark)
> Format: Structured notes for revision

---

## 1. Introduction

If you're connected to a WiFi network, someone can potentially see who you're chatting with, what messages you're sending, and what you're using — this is exactly what happens when a hacker connects to a WiFi network and monitors traffic. This is called **Packet Sniffing**, and the tool most commonly used for it is **Wireshark**.

### What This Course Covers

1. What packets are and how data travels across a network
2. How Wireshark works behind the scenes
3. TCP, UDP, DNS, HTTP — seen live in captures
4. Using filters to find exactly what you need
5. Analyzing real traffic

**Prerequisites:** No prior experience needed — just a laptop, curiosity, and basic networking knowledge (which is also covered briefly in this course).

---

## 2. What Is a Packet?

### Key Concept: Data Doesn't Travel as One Big Block

When you send a message or transfer data, it does **not** travel as a single unit to the destination. Instead:

- Your data is **broken into small units called packets**.
- Each packet carries: a **piece of data** + **routing information** + **error-checking information**.
- Typical packet size: **64 to 500 bytes**.

### Multiple Routes

Different packets from the same data transfer can travel via **different routes** through the network to reach the same destination — they travel **independently**.

- At the destination, all packets are **reassembled** in the correct order (Packet 1, Packet 2, Packet 3...) to reconstruct the original data.
- Each packet includes **sender and receiver information** so it knows where to go and where it came from.

### Anatomy of a Packet

A packet is divided into 3 parts:

| Part        | Contents                                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------------------------- |
| **Header**  | Source IP, Destination IP, Protocol used, TTL (Time To Live), Sequence Number, routing instructions           |
| **Payload** | The actual data being sent (the real content, split into small pieces)                                        |
| **Trailer** | Error-checking information — confirms **data integrity** (that the data wasn't tampered with / is legitimate) |

---

## 3. OSI Model — Layers & Protocols (Quick Recap for Wireshark)

| Layer                 | Common Protocols Captured in Wireshark                                          |
| --------------------- | ------------------------------------------------------------------------------- |
| **Application Layer** | HTTP, HTTPS, DNS, FTP — user-facing protocols                                   |
| **Transport Layer**   | TCP, UDP                                                                        |
| **Network Layer**     | IP addressing, TTL, routing                                                     |
| **Data Link Layer**   | MAC address, Ethernet frames                                                    |
| **Physical Layer**    | Frame size, raw bytes — responsible for the physical sender-receiver connection |

> For a full deep-dive into OSI layers and protocols, refer to a dedicated networking course.

---

## 4. How Wireshark Actually Works

### The Problem: Normal NIC Cards Are Limited

Normally, your **NIC (Network Interface Card)** can only see **your own device's traffic** — it cannot see traffic belonging to other devices on the same network.

### How Wireshark Changes This

Wireshark grants the NIC card **special permissions** to see traffic from other devices too — this is what enables **packet sniffing**.

### Key Capabilities Wireshark Enables

1. **Promiscuous Mode** — Normally, a NIC card ignores packets not addressed to it. Wireshark enables Promiscuous Mode, which allows the NIC to **capture packets belonging to other devices** on the same network too.

2. **Monitor Mode** (WiFi) — Captures **all wireless frames in the air**, similar concept but for wireless networks.

3. **libpcap / npcap** — These are the underlying **tools/drivers** that Wireshark depends on. They fetch **raw packets from the OS kernel** and hand them over to Wireshark.

### Simplified Data Flow

```
Target devices on network → traffic captured via libpcap/npcap →
raw packets passed to NIC card → NIC hands packets to Wireshark →
You can view/analyze raw packets (Packet Sniffing)
```

---

## 5. Core Networking Concepts Needed for Wireshark

### TCP vs UDP

|                 | TCP                                                      | UDP                               |
| --------------- | -------------------------------------------------------- | --------------------------------- |
| **Type**        | Connection-oriented                                      | Connectionless                    |
| **Reliability** | Reliable — guarantees delivery, retransmits lost packets | No guarantee of delivery          |
| **Speed**       | Slower (more overhead)                                   | Faster                            |
| **Used for**    | HTTP, HTTPS, FTP, email, file transfers                  | Video streaming, voice calls, DNS |

### Three-Way Handshake (TCP Connection Establishment)

Performed **before any data is shared** over TCP, to establish a connection:

1. **Sender → Receiver:** SYN packet
2. **Receiver → Sender:** SYN + ACK (acknowledgment) packet
3. **Sender → Receiver:** ACK packet

> **Important:** The three-way handshake only happens in **TCP**. UDP has no handshake — this is why UDP is called a **"connectionless protocol."**

### Common Protocols in Wireshark

| Protocol                                       | Notes                                                                                                                       |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **HTTP**                                       | Web-based protocol; data travels **unencrypted** (plain text — readable by anyone intercepting it)                          |
| **HTTPS**                                      | Web-based protocol; data travels **encrypted** (cipher text) — even if packets are captured, content can't be read directly |
| **DNS**                                        | Runs on **UDP**, port **53**. Converts domain names (e.g., google.com) into IP addresses                                    |
| **ARP** (Address Resolution Protocol)          | Converts IP address to MAC address within a local network                                                                   |
| **ICMP** (Internet Control Message Protocol)   | Used for **ping** — no port number used                                                                                     |
| **DHCP** (Dynamic Host Configuration Protocol) | Ports **67 & 68** — assigns IP addresses to new devices joining a network (LAN)                                             |
| **SSH**                                        | Port **22** — used for remote access                                                                                        |
| **FTP**                                        | Port **21** — used for file transfer                                                                                        |

---

## 6. Network Interfaces

A **network interface** is how your computer connects to the network. Wireshark asks you to select one before capturing.

| Interface Type     | Meaning                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| **Ethernet (eth)** | Wired connection                                                                         |
| **Wi-Fi**          | Wireless connection                                                                      |
| **Loopback**       | Local interface — used to see only your own device's traffic (not other network devices) |

> Choosing the correct interface is critical — selecting the wrong one means Wireshark won't capture the traffic you actually want.

---

## 7. Practical: Launching Wireshark (Kali Linux)

```bash
sudo wireshark
```

- `sudo` is used to grant root privileges.
- Wireshark will prompt you to select a **network interface** (e.g., `eth0` for Ethernet).
- Once selected, it **immediately starts capturing** all packets from devices on the local network (including your own).

---

## 8. Live Packet Capture Walkthrough (DNS + TCP Handshake Example)

### Setup

A clean test was done using:

```bash
sudo curl google.com
```

- `sudo` → admin privileges
- `curl` → command-line tool to send an HTTP request to a URL
- The target URL specified: `google.com`

### Step 1: DNS Query

When the request is sent, the device first needs Google's IP address (since it doesn't know it yet):

1. Device sends a **DNS request to the Default Gateway** (since it doesn't have Google's IP directly).
2. Request type: **A record** query for `google.com` (asks for the IPv4 address).
3. Immediately followed by another DNS request for the **AAAA record** (IPv6 address).
4. The Default Gateway replies with:
   - The **A record** (IPv4 address) for google.com
   - The **AAAA record** (IPv6 address) for google.com

**DNS Record Types (Quick Reference):**

| Record   | Stores              |
| -------- | ------------------- |
| **A**    | IPv4 address        |
| **AAAA** | IPv6 address        |
| **MX**   | Mail server address |

### Step 2: Three-Way Handshake (TCP)

Once the device has Google's IP, it communicates **directly** with Google's server:

1. **Device → Google:** SYN packet
2. **Google → Device:** SYN + ACK
3. **Device → Google:** ACK

This confirms the connection is established.

### Step 3: HTTP Request & Redirect

1. Device sends an **HTTP** request (not HTTPS) — because the curl command used `google.com` without specifying `https://`, so it defaults to HTTP/1.1.
2. Google responds with **status code 301 — "Moved Permanently"** — indicating the resource has moved (from HTTP to HTTPS).
3. Device sends an acknowledgment.

> **Note:** HTTP/1.1 is the HTTP version; HTTPS traffic typically shows as HTTP/2.0.

### Step 4: Four-Way Handshake (TCP Connection Termination)

Just as TCP uses a 3-way handshake to **establish** a connection, it uses a **4-way handshake** to **terminate** one:

1. **Device → Google:** FIN packet (device wants to end connection)
2. **Google → Device:** ACK (acknowledging the FIN)
3. **Google → Device:** FIN packet (Google is also ready to terminate) — often combined with a PUSH+ACK
4. **Device → Google:** Final ACK

---

## 9. Wireshark Color Coding

| Color                           | Meaning                                      |
| ------------------------------- | -------------------------------------------- |
| **Light Green**                 | TCP traffic                                  |
| **Light Blue**                  | UDP traffic                                  |
| **Black background + Red text** | TCP errors                                   |
| **Light Yellow**                | ARP communication                            |
| **Light Purple**                | ICMP (ping) traffic                          |
| **Dark Blue**                   | DNS traffic                                  |
| **White**                       | General TCP/HTTP data                        |
| **Red background**              | Serious TCP problems (e.g., retransmissions) |

### Customizing Colors

- **View → Coloring Rules** — view/edit default coloring rules.
- **View → Colorize Packet List** — toggle coloring on/off.
- Right-click a conversation → **Colorize Conversation** → choose a custom color for a specific conversation.
- **View → Colorize Conversation → Reset All** — reset to default colors.

---

## 10. Following a TCP Stream

To view the **actual content** of a conversation between two devices (not just metadata about which packets went where):

**Steps:**

1. Right-click any packet.
2. Go to **Follow → TCP Stream**.

**What it shows:**

- The **full conversation** between two devices in **readable text**.
- Especially useful for **HTTP traffic**, since HTTP data travels unencrypted (plain text) and can be read directly.
- Color coding within the stream view:
  - **Red** = what the client sent (your request)
  - **Purple** = what was received (the response)

---

## 11. Capture Filters vs Display Filters

|               | Capture Filters                             | Display Filters                                         |
| ------------- | ------------------------------------------- | ------------------------------------------------------- |
| **When used** | **Before** capturing starts                 | **After** packets are already captured                  |
| **Purpose**   | Limit what gets recorded in the first place | Find/filter specific packets from already-captured data |

### Capture Filter Examples

```
port 80          # Capture only HTTP traffic
port 53          # Capture only DNS traffic
host <IP address>  # Capture traffic only for a specific device
net <network range>  # Capture traffic for the entire local network range
```

### Display Filter Examples

```
http                       # Show only HTTP requests
dns                        # Show only DNS queries (can also filter by domain name)
tcp.port == 443            # Show HTTPS-related traffic
ip.addr == <IP address>    # Show traffic for a specific device
tcp.analysis.retransmission  # Show only retransmitted TCP packets
```

**How to apply a display filter:**

1. Type the filter into the **Display Filter bar** at the top.
2. If the bar turns **red** → invalid filter syntax.
3. If the bar turns **green** → valid filter.
4. Click the arrow/Enter to apply.
5. Clear the filter bar and press Enter again to see all traffic again (nothing is deleted — just hidden).

---

## 12. Practical Demo: Capturing Login Credentials Over HTTP

> ⚠️ **Legal note:** This kind of packet sniffing must only be performed on **authorized/vulnerable demo systems** — doing this on random real websites is **illegal**. The instructor specifically used an intentionally vulnerable demo website for this demonstration.

### Steps

1. Start packet capture in Wireshark.
2. On the target machine (e.g., a Windows 10 VM), submit a username/password on a **demo/vulnerable login page**.
3. Stop the capture.
4. Apply the display filter: `http`
5. Right-click the relevant `login.jsp` (or similar) POST request → **Follow → TCP Stream**.
6. The credentials appear in plain text within the request, e.g.:
   ```
   uid=admin&password=admin
   ```

**Key takeaway:** This demonstrates exactly why **HTTP is insecure** — anyone sniffing packets on the same network can read credentials submitted over plain HTTP. This is a core reason HTTPS (encrypted) is essential.

---

## 13. Saving & Exporting Results

| Action                         | How                                                              |
| ------------------------------ | ---------------------------------------------------------------- |
| **Save entire capture**        | File → Save As                                                   |
| **Export specific packets**    | File → Export Specified Packets (e.g., export only HTTP traffic) |
| **Share capture for analysis** | Share the `.pcap` file directly                                  |

> Wireshark can open `.pcap` files created by any capture tool (e.g., `tcpdump`), not just its own captures — good cross-tool compatibility.

---

## 14. What to Practice Next

1. **Capture your own browsing traffic** — try the same `curl` technique on different sites.
2. **Analyze sample capture files** — Wireshark's official "Sample Captures" page has ready-made `.pcap` files to practice on.
3. **Learn SSL/TLS in depth** — encryption, decryption, and how handshakes relate to secure connections.
4. **Practice on platforms** like TryHackMe and Hack The Box.
5. **Use vulnerable demo websites** to safely practice credential/packet sniffing in your own Kali VM setup — never on real/unauthorized sites.

---

## Quick-Reference Summary

```
CORE CONCEPTS
- Packet = data + header (routing info) + trailer (error checking)
- Packets travel independently via different routes, reassembled at destination
- Wireshark uses Promiscuous/Monitor Mode + libpcap/npcap to see ALL local network traffic

KEY PROTOCOLS
- TCP  → connection-oriented, reliable, uses 3-way handshake (SYN, SYN-ACK, ACK)
       → terminated via 4-way handshake (FIN, ACK, FIN, ACK)
- UDP  → connectionless, faster, no handshake (used by DNS, streaming, voice)
- HTTP → unencrypted (plain text — readable if sniffed)
- HTTPS → encrypted (cipher text — unreadable if sniffed)
- DNS  → port 53, resolves domain → IP (A = IPv4, AAAA = IPv6, MX = mail server)

WORKFLOW IN WIRESHARK
1. Select correct network interface (eth/WiFi/loopback)
2. Start capture (blue button) → generate traffic → stop capture (red button)
3. Apply filters: capture filters (before) vs display filters (after)
4. Right-click → Follow → TCP Stream to read full readable conversation
5. Use color coding to quickly identify traffic types
6. Save/export results as .pcap for later analysis

LEGAL REMINDER
Only sniff traffic on networks/devices you own or are explicitly authorized to test.
```
