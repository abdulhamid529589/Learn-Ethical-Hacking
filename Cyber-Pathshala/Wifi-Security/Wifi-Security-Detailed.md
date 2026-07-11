# Wi-Fi Security, Threats & Hacking Methodology — Course Notes

> **Source:** CPCS Cyber Pathshala — Wi-Fi Hacking Methodology transcript
> **Scope:** Wi-Fi networking fundamentals, terminology, monitor-mode capabilities, WPA/WPA2 handshake capture, deauthentication, offline password cracking, and Wi-Fi-adapter/driver setup.
> **Scope of legal use:** Every technique documented here is legal **only** against (a) your own router/access point, (b) a lab AP you own and control, or (c) a network you have explicit written authorization to test (e.g., a signed pentest/bug-bounty scope). Capturing handshakes, deauthenticating devices, or cracking passwords on a neighbor's or stranger's network — even "just to see if it works" — is unauthorized access to a computer network and is illegal in most jurisdictions regardless of intent. Treat every example below as "run this against your own test AP."

---

## Table of Contents

1. [How Networking & Wi-Fi Actually Work](#1-how-networking--wi-fi-actually-work)
2. [Core Wi-Fi Terminology](#2-core-wi-fi-terminology)
3. [Managed Mode vs. Monitor Mode](#3-managed-mode-vs-monitor-mode)
4. [What Monitor Mode Lets an Attacker Do](#4-what-monitor-mode-lets-an-attacker-do)
5. [Setting Up a Wi-Fi Adapter for Monitor Mode](#5-setting-up-a-wi-fi-adapter-for-monitor-mode)
6. [Practical: Scanning Networks with airodump-ng](#6-practical-scanning-networks-with-airodump-ng)
7. [Practical: Targeted Capture & the WPA Handshake](#7-practical-targeted-capture--the-wpa-handshake)
8. [Deauthentication Attack (Wi-Fi Jamming)](#8-deauthentication-attack-wi-fi-jamming)
9. [Offline Password Cracking](#9-offline-password-cracking)
   - [9.1 Dictionary Attack with aircrack-ng + rockyou.txt](#91-dictionary-attack-with-aircrack-ng--rockyoutxt)
   - [9.2 Leaked-Password / Breach Wordlists](#92-leaked-password--breach-wordlists)
   - [9.3 Brute-Force Generation with Crunch](#93-brute-force-generation-with-crunch)
10. [Password-Reuse Pattern Analysis (Concept Only)](#10-password-reuse-pattern-analysis-concept-only)
11. [Choosing & Troubleshooting a Wi-Fi Adapter](#11-choosing--troubleshooting-a-wi-fi-adapter)
12. [Defensive Takeaways](#12-defensive-takeaways)
13. [Tool & Command Reference](#13-tool--command-reference)
14. [Glossary](#14-glossary)
15. [Practice Tasks / Lab Checklist](#15-practice-tasks--lab-checklist)

---

## 1. How Networking & Wi-Fi Actually Work

- An **ISP** (e.g., a broadband provider) supplies internet access into a home, typically via a **router/modem**.
- Devices reach that connection two ways:
  - **Wired** — an Ethernet cable.
  - **Wireless** — Wi-Fi, broadcast from the router's built-in radio.
- When you enable a **hotspot** on a router (or phone), you assign it a **name**, and it already has a hardware **MAC address**. It may also carry a **private IP** (LAN), a **public IP** (internet-facing), and `127.0.0.1` (loopback).
- A client device (phone/laptop) has its own Wi-Fi radio (built-in or a USB adapter). It scans for nearby hotspots, the user selects one, enters a password, and the device sends a small authentication packet to the hotspot. If the password matches, the connection is established.

This basic flow — scan, select, authenticate, connect — is exactly what gets analyzed (and attacked) at the protocol level in the sections below.

---

## 2. Core Wi-Fi Terminology

| Term               | Meaning                                                                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ESSID**          | The human-readable **name** of the Wi-Fi network/hotspot (e.g., "HomeNetwork")                                                                                      |
| **BSSID**          | The **MAC address of the access point/hotspot** itself                                                                                                              |
| **Station ID**     | The **MAC address of a client device** connected to a hotspot                                                                                                       |
| **ENC**            | Encryption/security type in use — WEP, WPA, WPA2, WPA3                                                                                                              |
| **CH (Channel)**   | The specific radio-frequency "lane" the AP transmits on — conceptually similar to an FM radio station's frequency slot                                              |
| **Beacon**         | Frames the AP broadcasts periodically to announce its presence and capabilities                                                                                     |
| **PWR (Power)**    | Signal strength indicator; values closer to 0 (e.g., -30) = closer/stronger signal, more negative (e.g., -85) = farther/weaker                                      |
| **IV**             | Initialization Vector — part of the encryption scheme, relevant to WEP-era cryptographic weaknesses                                                                 |
| **Handshake file** | The captured exchange of packets that occurs when a client authenticates to an AP — contains the cryptographic material needed to attempt an offline password crack |
| **PSK**            | Pre-Shared Key — the password-based authentication method used in WPA/WPA2-Personal                                                                                 |

---

## 3. Managed Mode vs. Monitor Mode

| Mode             | Description                                                                                                                                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Managed Mode** | Default mode for everyday use — connects to a network, shows nearby SSIDs, handles normal data transfer. This is what your adapter uses day-to-day.                                                                      |
| **Monitor Mode** | A special mode (needs Wi-Fi-adapter chipset support) that lets the adapter **passively read all wireless traffic in range**, even for networks it isn't connected to, and (with the right tools) inject crafted packets. |

Monitor mode is what security tools like the `aircrack-ng` suite require to capture handshakes or send deauthentication frames.

---

## 4. What Monitor Mode Lets an Attacker Do

Once in monitor mode, a Wi-Fi adapter can:

1. **See detailed metadata for every AP in range** — ESSID, BSSID, channel, encryption type, signal strength — without ever connecting.
2. **See which client devices (stations) are connected to which AP** — including their MAC addresses — again, without being connected to that network.
3. **Inject packets**, including:
   - **Deauthentication ("deauth") frames** — force a connected client to disconnect from its AP (this is the mechanism behind Wi-Fi "jamming" of a specific device).
   - Capture the **WPA/WPA2 handshake** that occurs when a deauthenticated (or newly connecting) client re-authenticates.

> This is powerful and also why unauthorized use is a serious violation — an attacker doesn't need your password or even a connection to disrupt your network or capture your authentication exchange.

---

## 5. Setting Up a Wi-Fi Adapter for Monitor Mode

**Identify your wireless interface:**

```bash
ifconfig            # or: ip a
iwconfig             # shows only wireless interfaces + their current mode
```

**Enable monitor mode (automated method via airmon-ng):**

```bash
sudo airmon-ng start wlan0
```

**Stop background services that might interfere / silently flip you back to managed mode:**

```bash
sudo airmon-ng check kill
```

> This will also drop your normal internet connectivity while your adapter is dedicated to monitor mode — expected behavior.

**Verify the mode switch:**

```bash
iwconfig
```

---

## 6. Practical: Scanning Networks with airodump-ng

```bash
sudo airodump-ng wlan0
```

This streams a live table of every AP in range, showing BSSID, PWR, channel, encryption, ESSID, and the stations (clients) connected to each — useful for reconnaissance on **your own lab APs** to understand what's discoverable about a network before you test it.

Press `Ctrl+C` to stop and review the captured table.

---

## 7. Practical: Targeted Capture & the WPA Handshake

Once you've identified **your own test AP's** BSSID and channel from the scan above, focus capture on just that target and write output to a file:

```bash
sudo airodump-ng --bssid <YOUR_AP_BSSID> -c <channel> -w capture_file wlan0
```

- `--bssid` restricts capture to one AP.
- `-c` locks to that AP's channel.
- `-w` writes all captured packets (including any handshake) to files named `capture_file-01.cap`, plus `.kismet.csv`, `.log.csv`, etc.

If a client is already connected, you can simply wait for a natural reconnect — or trigger one deliberately (Section 8) to capture the handshake faster on your own test rig.

A successful capture shows a **"WPA handshake"** notice in the airodump-ng header.

---

## 8. Deauthentication Attack (Wi-Fi Jamming)

Using `aireplay-ng` against **your own AP and your own test client device**:

```bash
sudo aireplay-ng --deauth 100 -a <AP_BSSID> -c <CLIENT_MAC> wlan0
```

- `--deauth 100` — send 100 deauthentication packets.
- `-a` — the AP's BSSID (who the packets claim to be "from").
- `-c` — the specific client's MAC address to disconnect (omit to target all clients broadly).

When the client reconnects, the handshake is captured in the file started in Section 7. Stop with `Ctrl+C` once "WPA handshake" is confirmed.

> **Why this matters defensively:** deauth frames in WPA/WPA2 are largely unauthenticated at the management-frame level, which is exactly why **WPA3's Protected Management Frames (PMF)** were introduced — they cryptographically protect against exactly this kind of forced-disconnect attack. Testing this against your own WPA2 vs. WPA3 AP side-by-side is a great way to see the improvement in action.

---

## 9. Offline Password Cracking

All cracking happens **offline**, against the `.cap` file already captured — it does not touch the live network again.

### 9.1 Dictionary Attack with aircrack-ng + rockyou.txt

```bash
# Locate and extract Kali's built-in wordlist if not already extracted
cd /usr/share/wordlists
gunzip rockyou.txt.gz   # or unzip, depending on distro packaging

# Run the crack
aircrack-ng capture_file-01.cap -w /usr/share/wordlists/rockyou.txt
```

- Without `-w` (a wordlist), `aircrack-ng` can only tell you the encryption type — it cannot crack WPA/WPA2 without a wordlist (this is unlike old WEP, which had direct cryptographic weaknesses).
- If the password exists in the wordlist, it's displayed once found.

### 9.2 Leaked-Password / Breach Wordlists

Security researchers often supplement `rockyou.txt` with **breach-corpus wordlists** (aggregated leaked credentials from past breaches like Adobe, LinkedIn, etc., available through sources such as SecLists on GitHub). These often outperform generic wordlists because they reflect passwords real humans actually chose.

```bash
aircrack-ng capture_file-01.cap -w /path/to/leaked-passwords.txt
```

### 9.3 Brute-Force Generation with Crunch

For exhaustive brute-forcing (only practical for short/known-length passwords due to storage/time):

```bash
crunch 8 15 abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$ | aircrack-ng -e <ESSID> -w - capture_file-01.cap
```

- `crunch 8 15 <charset>` generates candidate passwords between 8 and 15 characters using the given character set.
- The `|` (pipe) streams crunch's output directly into `aircrack-ng` as its wordlist (`-w -` means "read wordlist from stdin") — avoiding the need to store a multi-petabyte wordlist file, since full 8–15 character keyspaces are computationally enormous.
- This is realistically only feasible for **known partial patterns** or very short passwords — full keyspace brute force at this length is not practical on a single machine in reasonable time.

| Method                | Speed     | Best for                                                     |
| --------------------- | --------- | ------------------------------------------------------------ |
| `rockyou.txt`         | Fast      | Common/weak passwords                                        |
| Leaked-password lists | Medium    | Passwords reused from other breaches                         |
| Crunch brute-force    | Very slow | Short, fully unknown passwords, or narrowing a known pattern |

---

## 10. Password-Reuse Pattern Analysis (Concept Only)

The transcript describes a real and well-documented password-hygiene problem: many people reuse the **same base password with minor, predictable variations** across sites (e.g., `Name123`, `Name@456`, `Name@789`).

**The security lesson (this is a defensive awareness point, not a how-to for targeting a specific person):**

- If any of a person's passwords are exposed (e.g., via a data breach, or via Windows' locally-stored Wi-Fi credentials, retrievable on a device you own via `netsh wlan show profile "<SSID>" key=clear`), an attacker can sometimes infer the **pattern** and predict other passwords, including ones never breached.
- This is precisely why security guidance insists on **unique, unrelated passwords per site** (ideally via a password manager) rather than "same password, different suffix."
- From a blue-team/awareness-training angle, this is worth demonstrating on **your own accounts/devices** to show colleagues or students how quickly a weak pattern falls apart — not as a method for profiling a real third party's password habits.

---

## 11. Choosing & Troubleshooting a Wi-Fi Adapter

**Buying tips:**

- Confirm **Linux/Kali driver support** before purchasing — check the listing description or manufacturer's GitHub for stated compatibility.
- Popular chipsets used in pentesting-friendly adapters include Atheros AR9271 and Ralink/MediaTek RT3070/MT7601U-based models (verify current recommendations, as chipset support changes over time).

**Driver troubleshooting, in order of effort:**

```bash
# 1. Check if it's recognized at all
lsusb

# 2. Try installing via APT first
sudo apt install <driver-package-name>

# 3. If not packaged, search GitHub for a driver repo matching the chipset name from lsusb,
#    then follow its README (usually: clone, make, make install)
git clone <driver-repo-url>
cd <driver-repo>
make && sudo make install

# 4. If the adapter only supports an older kernel/Linux version,
#    consider running an older Kali release in a separate VM rather than downgrading your main system.
```

**Common runtime error — network silently drops out of monitor mode:**

```bash
sudo airmon-ng check kill    # stop conflicting background services (also drops your internet — expected)
sudo airmon-ng start wlan0   # re-enter monitor mode cleanly
```

---

## 12. Defensive Takeaways

Understanding these attacks is most valuable when translated into hardening steps for **your own network**:

| Attack                                | Defensive Mitigation                                                                                                                                                         |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Deauthentication / jamming            | Use **WPA3** where possible — Protected Management Frames (PMF) cryptographically prevent forged deauth frames                                                               |
| Handshake capture + offline cracking  | Use a **long, high-entropy passphrase** (15+ random characters or a multi-word passphrase) — dictionary/leaked-password lists and even crunch brute-force become impractical |
| Password-reuse pattern inference      | Use a **password manager** to generate unrelated, unique passwords per account/network                                                                                       |
| Rogue monitoring of connected clients | Enable **MAC address randomization** on client devices (most modern phones do this by default) to reduce tracking of a specific device across networks                       |

---

## 13. Tool & Command Reference

| Tool                                    | Purpose                                                                                    |
| --------------------------------------- | ------------------------------------------------------------------------------------------ |
| `airmon-ng`                             | Enable/disable monitor mode, stop conflicting services                                     |
| `airodump-ng`                           | Passive Wi-Fi scanning and targeted packet/handshake capture                               |
| `aireplay-ng`                           | Packet injection — deauthentication attacks                                                |
| `aircrack-ng`                           | Offline WPA/WPA2 handshake password cracking                                               |
| `crunch`                                | Custom wordlist / brute-force candidate generation                                         |
| `iwconfig` / `ifconfig`                 | Inspect and configure wireless interfaces                                                  |
| `netsh wlan show profile ... key=clear` | (Windows, on a device you own) reveal saved Wi-Fi passwords locally stored on that machine |

---

## 14. Glossary

| Term                                  | Definition                                                                                |
| ------------------------------------- | ----------------------------------------------------------------------------------------- |
| **AP**                                | Access Point — the device broadcasting a Wi-Fi network                                    |
| **BSSID**                             | MAC address of an access point                                                            |
| **ESSID**                             | The network name shown to users                                                           |
| **Deauth frame**                      | A management frame that forces a client to disconnect from an AP                          |
| **Handshake**                         | The captured authentication exchange between client and AP, needed for offline cracking   |
| **Monitor mode**                      | Adapter mode allowing passive capture of all nearby wireless traffic and packet injection |
| **PMF (Protected Management Frames)** | WPA3 feature that cryptographically protects management frames (like deauth) from forgery |
| **PSK**                               | Pre-Shared Key — password-based Wi-Fi authentication                                      |
| **Station**                           | A client device connected to an AP                                                        |
| **Wordlist**                          | A file of candidate passwords used in dictionary-based password cracking                  |

---

## 15. Practice Tasks / Lab Checklist

Perform every task below **only against a router/AP you own** (e.g., a spare home router flashed to a test SSID, or a dedicated lab AP) — never against a neighbor's or public network.

- [ ] Set up a Wi-Fi adapter with confirmed monitor-mode support in your Kali VM; verify with `iwconfig`.
- [ ] Run `airodump-ng` and document ESSID, BSSID, channel, and encryption type for your own test AP.
- [ ] Capture a WPA2 handshake from your own AP + test client, using a deliberate deauth to force reconnection.
- [ ] Crack the resulting `.cap` file with `rockyou.txt` — record how long it takes with an intentionally weak password.
- [ ] Re-run the same crack with a strong 15+ character passphrase set on your test AP, and confirm it does **not** crack with `rockyou.txt` in a reasonable time.
- [ ] If you have access to a WPA3 test AP, repeat the deauth step and document how PMF changes (or blocks) the outcome.
- [ ] Write a short report (as if for a client) comparing WPA2 vs. WPA3 resilience to deauth and offline cracking, with a hardening recommendation.

---

_End of Wi-Fi hacking methodology notes._
