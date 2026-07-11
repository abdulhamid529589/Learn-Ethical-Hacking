# CCTV & IP Camera Security — Concepts, Vulnerabilities & Hardening Guide

## https://www.youtube.com/watch?v=MYtXhGa4KQU

> **Course:** Cyber Security — IoT / CCTV Security Assessment
> **Scope note:** This guide covers the _concepts_ side of CCTV security assessment (camera types, protocols, common weaknesses) and **defensive hardening** — for both local (LAN) and internet-facing (WAN) deployments. It intentionally does not include step-by-step exploitation/brute-force scripts; those specifics are the same regardless of whose device is on the other end, so they're left out even in an "own device" context. The methodology section explains how a real, authorized assessment is structured instead.

---

## Table of Contents

1. [Types of CCTV Cameras](#1-types-of-cctv-cameras)
2. [How a CCTV System Works — The Full Picture](#2-how-a-cctv-system-works--the-full-picture)
3. [LAN vs WAN Access — Two Different Attack Surfaces](#3-lan-vs-wan-access--two-different-attack-surfaces)
4. [Key Protocols Used by CCTV Systems](#4-key-protocols-used-by-cctv-systems)
5. [Protocol-by-Protocol Weaknesses](#5-protocol-by-protocol-weaknesses)
6. [The Five Core Vulnerability Categories](#6-the-five-core-vulnerability-categories)
7. [How Your Camera Becomes Reachable From the Internet](#7-how-your-camera-becomes-reachable-from-the-internet)
8. [Auditing Your Own External Exposure (Safely)](#8-auditing-your-own-external-exposure-safely)
9. [Hardening Checklist](#9-hardening-checklist)
10. [How a Real Authorized CCTV Pentest Is Structured](#10-how-a-real-authorized-cctv-pentest-is-structured)
11. [Summary / Quick Revision Table](#11-summary--quick-revision-table)

---

## 1. Types of CCTV Cameras

| Type                    | Connection                                               | Key Trait                                            | Security Note                                                                                                                |
| ----------------------- | -------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Analog**              | Wired cable to a DVR                                     | Oldest technology; transmits data via direct cable   | Low security by design, but also low network exposure — no encryption needed since it's not on a data network                |
| **IP Camera**           | Assigned its own IP address, connects like an IoT device | Enables remote access, live streaming, cloud storage | Because it's on a data network, it inherits **all** normal network attack surface (scanning, brute-force, protocol exploits) |
| **Wireless Camera**     | Connects via Wi-Fi, uses a private IP within the LAN     | Convenient, no cabling                               | Vulnerable to **sniffing** (anyone on the same Wi-Fi/LAN can potentially observe traffic) and **Man-in-the-Middle attacks**  |
| **PTZ (Pan-Tilt-Zoom)** | Remote-controllable camera, wide network accessibility   | Lets an operator remotely change angle/zoom          | Greater remote accessibility = greater exposure; more of the network can potentially interact with it                        |

---

## 2. How a CCTV System Works — The Full Picture

A typical home/small-business CCTV setup looks like this:

```
 [Camera 1] ──┐
 [Camera 2] ──┼──► [DVR] ──► [Router] ──► Internet
 [Camera 3] ──┘        │
                   (private IPs assigned
                    by the router, e.g.
                    192.168.1.x)
```

- Each **IP camera** gets its own **private IP address** (e.g., `192.168.1.2`, `192.168.1.3`) from the **router**.
- Cameras send their recorded/live data to the **DVR** (Digital Video Recorder), which also has its own private IP (e.g., `192.168.1.3`).
- The **router** is the device that assigns all private IPs on the network and also holds **two identities**: a **private IP** (for the local network) and a **public IP** (visible to the internet).
- Anyone on the **same local network** (same Wi-Fi/LAN) as the cameras and DVR can potentially communicate with them directly.

---

## 3. LAN vs WAN Access — Two Different Attack Surfaces

### Scenario A: Local Area Network (LAN)

If a device (attacker or otherwise) is connected to the **same Wi-Fi/network** as the cameras, it can communicate with them directly — this is the scenario covered by network scanning tools like Nmap.

### Scenario B: Wide Area Network (WAN) / Remote Access

If the camera owner wants to view footage **remotely** (e.g., from a mobile app while away from home), the **private IP must somehow become reachable from the internet**. This is achieved through:

1. **Port Forwarding** — a rule configured on the router's admin panel that says: _"Any traffic arriving at my public IP on port X should be forwarded to this specific internal device (e.g., the DVR) on port Y."_ The router acts as a **mediator** between the internet and the internal device.
2. **SIM-connected devices** — some DVRs/cameras have a SIM card and thus their own public IP directly, without needing router-level forwarding.
3. **P2P / Cloud relay services** — many consumer DVR brands use a cloud "P2P" service (accessed via a mobile app and a device ID/QR code) instead of classic port forwarding, which has its own separate set of risks (cloud account security, vendor server compromise, etc.).

**Key takeaway:** LAN and WAN represent **two distinct attack surfaces**. Both are "feasible" to test, but the techniques, tools, and legal/ethical scoping differ — WAN-facing assessment involves your **ISP's public IP address** and, potentially, third-party cloud infrastructure that isn't yours to test.

---

## 4. Key Protocols Used by CCTV Systems

| Protocol                                       | Default Port(s)         | Purpose                                                                                                                                                           |
| ---------------------------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **RTSP** (Real-Time Streaming Protocol)        | 554                     | Live video streaming — lets a remote viewer watch the camera feed                                                                                                 |
| **HTTP / HTTPS**                               | 80 / 443                | Web-based login portal for camera/DVR configuration and management                                                                                                |
| **ONVIF** (Open Network Video Interface Forum) | 8080, 1899 (commonly)   | Cross-brand device communication/discovery — lets cameras from different manufacturers work together on one DVR/NVR                                               |
| **FTP** (File Transfer Protocol)               | 20 (data), 21 (control) | File transfer — used by some systems for storing/retrieving recorded footage                                                                                      |
| **TCP / UDP**                                  | Varies                  | Underlying transport for the main data/streaming connections                                                                                                      |
| **SNMP / SMTP**                                | Varies                  | Basic network management info (SNMP) and mail alerts (SMTP) — usually secondary/lower priority from a camera-hacking perspective, but can leak system information |

**Priority for security review (per the course material):** RTSP and ONVIF are considered the most camera-specific and highest-priority protocols to review, followed by HTTP/HTTPS, then FTP. TCP/UDP and SNMP/SMTP are considered more general/secondary.

---

## 5. Protocol-by-Protocol Weaknesses

### RTSP (Real-Time Streaming Protocol)

- Often has **no authentication**, or weak/default credentials.
- **Public RTSP URLs** can be exposed if port forwarding is configured, meaning the live stream may be viewable by anyone who has (or guesses) the URL — without needing to alter any settings, just to _view_.
- **No encryption** by default — data (including the video stream itself) travels in cleartext, making **stream hijacking/interception** possible for anyone positioned on the network path.

### HTTP / HTTPS (Web Management Portal)

- **Default credentials** are a major weak point.
- **Weak session management** can enable **session hijacking**.
- Being a standard web interface, it inherits typical **web application vulnerabilities**: **CSRF** (Cross-Site Request Forgery) and **XSS** (Cross-Site Scripting) among them.
- **Unpatched/outdated firmware** running the web interface increases the chance of known, publicly documented exploits (CVEs) being applicable.

### ONVIF

- **Weak authentication** (default or weak usernames/passwords).
- **Device discovery exposure** — devices can be publicly discoverable on a network, inviting unauthorized connection attempts.
- **Poor authorization control** — insufficient granularity over who can control what (e.g., viewing vs. full configuration access).

### FTP

- **Weak/default authentication**, including the possibility of **anonymous login** being enabled (no real credentials required at all).
- Data transferred here can be **intercepted** if unencrypted.

### TCP/UDP (general transport)

- Primarily relevant to **sniffing** and **Denial-of-Service (DoS)** style issues rather than direct credential-based compromise.

### SNMP/SMTP

- Vulnerable to **default community strings** (SNMP's "public"/"private" defaults).
- Can lead to **information leakage**, **misconfiguration exposure**, and **credential leakage**; SNMP is also associated with **open relay** issues in some contexts.

---

## 6. The Five Core Vulnerability Categories

Across almost any CCTV/IoT device, compromise typically traces back to one of these five root causes:

1. **Default Credentials** — factory-set usernames/passwords (e.g., `admin`/`admin`, `admin`/`12345`) that are never changed by the end user.
2. **Open Ports & IP Exposure** — the device's IP address (and the services running on it) being reachable and identifiable, giving an entry point to probe further.
3. **Protocol Exploitation** — outdated software versions or misconfigured protocol settings (especially RTSP and ONVIF) being leveraged for unauthorized access.
4. **Firmware Vulnerabilities** — outdated firmware versions with publicly known CVEs that haven't been patched.
5. **Human Error** — by far the most common root cause in practice: reusing/never changing default passwords, clicking malicious links, oversharing Wi-Fi/network access, etc.

---

## 7. How Your Camera Becomes Reachable From the Internet

Understanding _why_ a camera is externally reachable is the foundation of securing it. The main mechanisms are:

| Mechanism                                                            | How It Works                                                                                                                 | Risk                                                                                                                                          |
| -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Port Forwarding**                                                  | Router admin panel rule mapping a public port → an internal device's IP/port                                                 | If the forwarded service (RTSP/HTTP) has weak/default auth, it's now reachable by **anyone on the internet**, not just you                    |
| **UPnP (Universal Plug and Play)**                                   | Some devices automatically configure port forwarding on the router **without your explicit knowledge**                       | Can silently expose services you never intended to be public                                                                                  |
| **DDNS (Dynamic DNS)**                                               | Gives your changing home public IP a fixed hostname (e.g., `mycamera.ddns.net`)                                              | Makes your exposed service easier to _find_ and return to, including by automated internet-wide scanners                                      |
| **Cloud/P2P relay services**                                         | Camera connects outward to the vendor's cloud; your phone app connects to the same cloud — no inbound port forwarding needed | Security now also depends on the **vendor's cloud security**, which is outside your control                                                   |
| **Public search engines for exposed devices (e.g., Shodan, Censys)** | These index internet-facing devices, including CCTV systems with open RTSP/HTTP ports                                        | Anyone can search these engines to find exposed cameras worldwide, including potentially yours, if externally reachable with default settings |

---

## 8. Auditing Your Own External Exposure (Safely)

Since testing "from outside" your own network means interacting with **your ISP's public IP address** — and potentially triggering your ISP's or a third party's monitoring — the safest and most standard way to audit your own external exposure is:

1. **Check what's actually exposed, without attacking anything:**
   - Log in to your **router's admin panel** and review the **Port Forwarding / Virtual Server** section — list every rule and ask "do I actually need this exposed to the internet?"
   - Search your own public IP address (find it by searching "what is my IP" from a browser on your home network) on **Shodan** or **Censys** — these are passive, read-only lookup services that show what internet-wide scanners have already found exposed on your IP, **without you needing to run any scan yourself**.
   - If you want to actively confirm a specific port is open from the _outside_, use a **read-only online port checker** (many free "open port checker" web tools exist) rather than running exploitation tools against your own public IP — this avoids any ambiguity about "attacking" traffic crossing your ISP's network.
2. **If you want to go further and formally pentest your own external exposure** (e.g., running Nmap/Burp Suite-style testing against your own public IP from an external vantage point, such as a cloud VPS you control), treat it like any professional engagement:
   - Confirm with your **ISP's Acceptable Use Policy** whether active security scanning of your own public IP is permitted (some ISPs restrict this).
   - Perform the test from infrastructure you own/control (e.g., a rented VPS), not from a shared or ambiguous network.
   - Keep dated documentation (screenshots, scope notes, "this is my own device, tested on [date] for coursework") in case anything is ever questioned by your ISP.
3. **Never test WAN exposure using default/guessed passwords repeatedly from automated scripts against your live public IP** without controlling for lockouts, logging, and rate limits — many DVR/router firewalls will flag or temporarily block your own home IP, potentially locking you out of your own remote access.

---

## 9. Hardening Checklist

Practical steps to secure a CCTV/DVR/NVR system, directly addressing each vulnerability category above:

### Credentials

- [ ] Change **all default usernames and passwords** immediately after setup (DVR, individual cameras, router admin panel).
- [ ] Use a strong, unique password for each device — not the same password reused across camera, DVR, and router.
- [ ] Disable any **anonymous/guest login** options (especially on FTP, if used).

### Network Exposure

- [ ] Avoid exposing **RTSP or ONVIF ports directly to the internet** via port forwarding — these protocols are the least likely to have strong authentication.
- [ ] If remote viewing is needed, prefer accessing your cameras through a **VPN into your home network** rather than forwarding camera ports directly to the internet.
- [ ] Disable **UPnP** on your router unless you specifically need it and understand what it's exposing.
- [ ] Put cameras/DVR on a **separate VLAN or guest network**, isolated from your main devices (laptops, phones, work devices) — this limits what an attacker can reach even if a camera is compromised.

### Protocols

- [ ] Disable **unused protocols/services** on the DVR/camera admin panel (e.g., disable FTP if you don't use it).
- [ ] Where possible, enforce **HTTPS only** for the web management portal (disable plain HTTP).
- [ ] Change protocol ports from their **well-known defaults** where the device supports it (reduces automated/opportunistic scanning hits, though this is "security through obscurity" and not a substitute for the other steps).

### Firmware

- [ ] Regularly check for and apply **firmware updates** from the manufacturer.
- [ ] Check the manufacturer/model against public **CVE databases** periodically to see if known vulnerabilities exist for your firmware version.

### Human Factors

- [ ] Don't share Wi-Fi credentials broadly; use a **separate guest network** for visitors.
- [ ] Be cautious of phishing links claiming to be from your camera vendor or cloud service.
- [ ] Periodically review who has app/account access to your camera system's cloud account, and remove access that's no longer needed.

---

## 10. How a Real Authorized CCTV Pentest Is Structured

If you want to formally practice offensive testing (beyond passive auditing) — even on your own equipment — structuring it like a professional engagement is good academic practice and keeps you within safe, well-documented boundaries:

1. **Scope definition:** Explicitly list which devices, IP ranges, and ports are in scope (e.g., "my DVR at 192.168.1.3 and my public IP X.X.X.X, ports 80/443/554 only").
2. **Written authorization:** Even for your own equipment, write yourself a short "authorization memo" — device owner, tester, date, scope, and purpose. This is standard practice and builds the habit for real client engagements later, where **this step is legally mandatory**.
3. **Reconnaissance:** Identify live hosts and open ports (e.g., via Nmap) — as covered in earlier parts of this course.
4. **Vulnerability identification:** Match discovered services/versions against known weaknesses (default creds, outdated firmware, protocol misconfig).
5. **Controlled testing:** Attempt only within the agreed scope; for credential testing, prefer **testing with your own known/changed credentials first**, then a **small, controlled wordlist** (not large automated brute-force runs against a live public IP, per the WAN caution in Section 8).
6. **Documentation & remediation:** Record what was found, how, and — most importantly for a real engagement — **how to fix it** (this maps directly to the Hardening Checklist above).
7. **Re-test after remediation:** Confirm the fix actually closes the gap (e.g., default credentials no longer work, RTSP now requires authentication).

---

## 11. Summary / Quick Revision Table

| Concept                               | One-Line Summary                                                                                                                                   |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Camera types**                      | Analog (wired, low exposure), IP (networked, full attack surface), Wireless (Wi-Fi, sniffing/MITM risk), PTZ (remote-controllable, wider exposure) |
| **DVR**                               | Digital Video Recorder — stores footage from cameras and can output/stream it                                                                      |
| **Router's role**                     | Assigns private IPs to all local devices; holds the public IP; performs port forwarding for remote access                                          |
| **LAN attack surface**                | Anyone on the same local network/Wi-Fi as the cameras can potentially communicate with them directly                                               |
| **WAN attack surface**                | Requires port forwarding, a SIM/public IP, or a cloud/P2P relay to make the camera reachable from the internet                                     |
| **RTSP**                              | Video streaming protocol, port 554; often unauthenticated and unencrypted                                                                          |
| **HTTP/HTTPS**                        | Web management portal, ports 80/443; vulnerable to default creds, weak sessions, CSRF/XSS                                                          |
| **ONVIF**                             | Cross-brand device communication, ports ~8080/1899; often weakly authenticated                                                                     |
| **FTP**                               | File transfer, ports 20/21; risk of anonymous/default login                                                                                        |
| **5 core vulnerability categories**   | Default credentials, open ports/IP exposure, protocol exploitation, firmware vulnerabilities, human error                                          |
| **Auditing external exposure safely** | Use Shodan/Censys lookups and read-only port checkers first; avoid active brute-force against your live public IP                                  |
| **Best remote-access alternative**    | Use a VPN into your home network instead of directly port-forwarding camera/DVR ports to the internet                                              |
| **Structured pentest approach**       | Scope → authorization → recon → vulnerability ID → controlled testing → documentation → remediation → re-test                                      |

---

_End of Notes — CCTV & IP Camera Security: Concepts, Vulnerabilities & Hardening_
