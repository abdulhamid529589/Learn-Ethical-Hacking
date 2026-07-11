# Cybersecurity / Networking Notes — IP Addresses (Concept, Tracking, Types & Rules)

> Instructor: **Ashish Kumar** — Cyber Pathshala
> Topic: Everything about IP addresses from an ethical-hacking/cybersecurity perspective — what they are, how tracking via IP actually works, the 4 IP types (public/private/static/dynamic), and the core "IP Rules" needed to avoid mistakes later in attacks/pentests.

---

## Table of Contents

1. [What Is an IP Address?](#1-what-is-an-ip-address)
2. [Structure of an IP Address](#2-structure-of-an-ip-address)
3. [The General Method for "Reading" Any Address (IP, MAC, IMEI, etc.)](#3-the-general-method-for-reading-any-address-ip-mac-imei-etc)
4. [How Tracking via IP Address Actually Works](#4-how-tracking-via-ip-address-actually-works)
5. [Hands-On: Finding Your Own IP & Its Location Data](#5-hands-on-finding-your-own-ip--its-location-data)
6. [The 4 Types of IP Addresses](#6-the-4-types-of-ip-addresses)
7. [Public vs. Private IP](#7-public-vs-private-ip)
8. [Static vs. Dynamic IP](#8-static-vs-dynamic-ip)
9. [Hands-On: Proving You Have a Dynamic IP](#9-hands-on-proving-you-have-a-dynamic-ip)
10. [The IP Rules (Critical for Ethical Hacking)](#10-the-ip-rules-critical-for-ethical-hacking)

---

## 1. What Is an IP Address?

> **Definition:** _"An IP address (Internet Protocol address) uniquely identifies a device on a network, allowing different devices to communicate across networks and the internet."_

**Analogy:** To courier a book to someone (e.g., "Rohit"), you **must** know their home address — without it, delivery is impossible. Similarly, in networking, when two or more devices communicate, each **must** have a unique address so they can identify and reach one another. That address is the **IP address**.

---

## 2. Structure of an IP Address

**Example IP shown:** `17.172.224.47`

| Property                       | Value                 |
| ------------------------------ | --------------------- |
| Number of "pairs" (octets)     | **4**                 |
| Value range per pair           | **1 – 255**           |
| Size per pair                  | **8 bits (1 byte)**   |
| Total size of the full address | **32 bits (4 bytes)** |

---

## 3. The General Method for "Reading" Any Address (IP, MAC, IMEI, etc.)

> **Instructor's key teaching point:** This methodology applies to **any** unique identifier you'll encounter in cybersecurity — IP address, MAC address, IMEI, etc. — not just IP. Learn the _method_, and you can extract meaningful tracking information from any such address in the future.

**The 2-question method:**

| Question                                        | What It Establishes                                                                                            |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **1. What is this address's use?**              | e.g., for an IP address → its use is to **facilitate communication in networking**                             |
| **2. In what range does this address operate?** | e.g., for an IP address → it works **worldwide** (across the entire internet, not just one city/state/country) |

**Why unique addresses are divided hierarchically:** To ensure **uniqueness** (no duplicates) in a systematic way, a "range" (e.g., the world) is broken into progressively smaller sub-categories:

```
World → Country → State → City → (specific) Address
```

### Applying This to an IP Address's 4 Pairs

| Pair                                   | What It Reveals (per the walkthrough)                                                                                                            |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **1st pair**                           | The **country** (example: India)                                                                                                                 |
| **2nd pair**                           | The **state** (example: Rajasthan)                                                                                                               |
| **3rd pair**                           | The **city** — **and** the **ISP (Internet Service Provider)** the address was assigned to (example: the Jio office in Jaipur, Rajasthan, India) |
| **4th pair** (combined with the above) | Used, together with the ISP's own internal records, to identify the **specific user** the address was assigned to at a given time                |

---

## 4. How Tracking via IP Address Actually Works

**Step-by-step (conceptual) process:**

1. An IP address is captured/logged along with a **specific timestamp** (e.g., "someone hacked our website at 12:05 AM, IP address `17.172.224.47`").
2. This IP + timestamp is reported to a **Cyber Crime Branch** (or similar authority).
3. Using the first 3 pairs, authorities identify the **ISP** (e.g., the Jio office in Jaipur).
4. The ISP is asked: _"Who did you assign this specific IP address to, at this specific date/time?"_
5. The ISP consults its own internal **logs** (referred to as a "lock file" in the transcript — i.e., the ISP's assignment/activity logs) and reveals:
   - The **user** the IP was assigned to at that time.
   - What **activities/websites** that user accessed using that IP during that session.

> ⚠️ **Caveat noted:** There is some security/privacy protection around this process (ISPs don't hand over this data to just anyone) — but structurally, **this is the mechanism** by which IP-based tracking/investigation works.

---

## 5. Hands-On: Finding Your Own IP & Its Location Data

**Steps:**

1. Open any browser.
2. Search: **"what is my IP v4"** (including "v4" in the search is recommended/important).
3. Click any of the resulting websites.
4. **Observed result:** The site not only displays your IPv4 address, but — using the **first 3 pairs** — also identifies the **ISP/network office** location the address was assigned to (example shown: a city network office in Jabalpur, Madhya Pradesh, India).

> **This practically confirms the concept from §3–4:** the first few octets of a public IP genuinely do map back to a specific ISP/region, which is the real starting point for any IP-based tracking or investigation.

---

## 6. The 4 Types of IP Addresses

IP addresses are grouped into **2 pairs of categories** (4 types total):

| Pair       | Types                            |
| ---------- | -------------------------------- |
| **Pair 1** | **Public IP** vs. **Private IP** |
| **Pair 2** | **Static IP** vs. **Dynamic IP** |

---

## 7. Public vs. Private IP

Using the same **2-question method** from §3:

| Question   | Private IP                                       | Public IP                                                |
| ---------- | ------------------------------------------------ | -------------------------------------------------------- |
| **Use?**   | Uniquely identify a device **for communication** | Same — uniquely identify a device for communication      |
| **Range?** | Only within a **Local Area Network (LAN)**       | Across the **entire internet / Wide Area Network (WAN)** |

### The Nickname Analogy

|                       | **Private IP**                                                                      | **Public IP**                                                                                                                         |
| --------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Real-world equivalent | Your **nickname** — used only by close family/friends, within a small/narrow circle | Your **official/legal name** — recognized and usable **anywhere in the world** (school, government documents, other states/countries) |
| Scope of usefulness   | Meaningless outside your small circle/network                                       | Universally usable/recognized regardless of location                                                                                  |

**Practical example given:** In a LAN with 15–50 systems talking to each other, each system needs a **unique private IP** to communicate locally. When you and someone across the world communicate over the internet (e.g., watching this very video), that's only possible because of **public IPs**.

---

## 8. Static vs. Dynamic IP

### Setting Up the Puzzle (Questions Posed to the Audience)

1. **How many billion people are active on the internet today?** → Answer given: **~5.45 billion**.
2. **How many total possible IPv4 address combinations exist?** → Answer given: **~4.3 billion**.

**The apparent contradiction:** There are **more active internet users (5.45B)** than **total possible IPv4 addresses (4.3B)** — yet nobody experiences an error saying "no IP addresses left." **How is this possible**, especially since **IPv6** (which would solve the shortage) still isn't fully/universally implemented everywhere even today?

> **Answer: the Dynamic IP concept** solves this.

### Dynamic IP

- An IP address is assigned to **User A**. If User A goes offline (e.g., travels for a week, powers off their device, or runs out of mobile balance/recharge), that IP address sits **unused**.
- Rather than let it go to waste, the **same IP address gets reassigned to User B** for that period.
- When User A comes back online, they're simply given **a different available IP address** — with **no disruption** to their ability to browse/use the internet.
- **Why this works at scale:** Not all 5.45 billion users are online **simultaneously** — time zone differences (many regions are asleep), depleted mobile balances, powered-off devices, etc., mean the _actual_ concurrent demand for IPs is far lower than the total user count, making address-sharing-over-time feasible.
- **This is the default for most regular user devices** — your home/mobile internet connection typically has a dynamic IP that can change periodically.

### Static IP

- Certain devices — notably **servers** — need to communicate with (serve) people across a **very wide range**, and **cannot** have a constantly-changing address, because too many people rely on reaching that _exact_ same address reliably.
- **Analogy used:** A school's physical address must stay **fixed** — if a school's address changed every few days (Sector 2 → Sector 7 → Sector 6...), students could never reliably find it and attend. In contrast, an individual student's home address changing doesn't disrupt anyone else.
- **Conclusion:** Devices that provide a **consistent public service** (like a hosted website on a server) require a **static IP**, so users worldwide can reliably reach the same address every time.

---

## 9. Hands-On: Proving You Have a Dynamic IP

**Steps (performed on a mobile device):**

1. **Turn off Wi-Fi/hotspot** — ensure the device is using its **SIM's mobile data network** only (not a shared/fixed network).
2. Search **"what is my IP v4"** in any browser → note down (or share, e.g. in comments) just the **last two pairs/octets** of your IP address (for privacy, don't share the full address).
3. Turn on **Airplane Mode** for **~5 seconds**, then turn it back **off**.
   - Rationale: this **fully disconnects** the device's network, meaning the IP address is no longer "in use" during that window — freeing it up to potentially be reassigned.
4. **Reload** the same "what is my IP" website.
5. **Expected result:** Your **mobile IP address will have changed** — proof that your connection uses a **dynamic IP**.

> **Note:** A PC/laptop's IP on a stable, always-connected network may **not** change from this same test (as demonstrated on the instructor's PC) — the behavior/timing of dynamic reassignment depends on the specific network/connection type.

---

## 10. The IP Rules (Critical for Ethical Hacking)

> ⚠️ **The single most important takeaway of this lesson, per the instructor** — essential for avoiding basic mistakes later when trying to find vulnerabilities and compromise devices.

**The Rules:**

1. A **Private IP** can only communicate with **another Private IP**, and only **within the same Local Area Network**.
2. A **Public IP** can only communicate with **another Public IP**, over a **Wide Area Network** (i.e., the internet).
3. ❌ **A Private IP can NEVER communicate directly with a Public IP.**
4. ❌ **A Public IP can NEVER communicate directly with a Private IP.**

### Why This Matters for Ethical Hacking

- When attempting to find vulnerabilities or compromise a target device, you **must** first determine whether the target machine has a **private or public IP**.
- Misjudging this — e.g., assuming you can directly reach a target's private IP from your own public-IP-based machine (or vice versa) — will cause your attack/scan attempts to silently fail, and you may never realize _why_ — leading you to miss or fail more than half of potential attack vectors simply due to this basic addressing misunderstanding.

---

## Quick-Reference Summary

| Concept                          | Key Point                                                                                                                            |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **IP Address**                   | Uniquely identifies a device for network communication; 32 bits = 4 octets (1–255 each)                                              |
| **Reading any address (method)** | Ask: (1) What is its use? (2) What range does it operate in? — applies to IP, MAC, IMEI, etc.                                        |
| **IP-based tracking**            | First 3 octets → ISP/region → ISP's internal logs (with timestamp) → specific user + activity                                        |
| **Public IP**                    | Like your "real/legal name" — works worldwide, used for WAN/internet communication                                                   |
| **Private IP**                   | Like your "nickname" — only meaningful within a LAN                                                                                  |
| **Dynamic IP**                   | Reassigned/shared over time as devices go on/offline — solves the "more users than available IPv4 addresses" problem                 |
| **Static IP**                    | Fixed, unchanging — required for servers/services that need a reliable, constant address                                             |
| **The IP Rules**                 | Private ↔ Private (LAN only); Public ↔ Public (WAN/internet only); Private and Public **never** communicate directly with each other |
