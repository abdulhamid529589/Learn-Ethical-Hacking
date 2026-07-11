# Ethical Hacking Notes — Metasploit Framework (MSF) & Armitage

> Instructor: **Ashish Kumar** — Cyber Pathshala
> Topic: **MSF (Metasploit Framework)** and its GUI counterpart **Armitage** — setup, core terminology, database integration, scanning, and a full exploitation walkthrough.
> Environment: Kali Linux (attacker) vs. a victim/target VM (example IP: `192.168.56.103`)

---

## Table of Contents

1. [What Is MSF / Armitage?](#1-what-is-msf--armitage)
2. [Core MSF Terminology: Auxiliaries, Exploits, Payloads](#2-core-msf-terminology-auxiliaries-exploits-payloads)
3. [Installing Armitage](#3-installing-armitage)
4. [Starting MSF Console & Understanding the Banner](#4-starting-msf-console--understanding-the-banner)
5. [Connecting MSF to Its Database](#5-connecting-msf-to-its-database)
6. [Full Reboot-Safe Startup Sequence](#6-full-reboot-safe-startup-sequence)
7. [Scanning the Victim via `db_nmap`](#7-scanning-the-victim-via-db_nmap)
8. [Launching Armitage & Viewing the Victim](#8-launching-armitage--viewing-the-victim)
9. [Hands-On Exploitation: VSFTPD Backdoor Command Execution](#9-hands-on-exploitation-vsftpd-backdoor-command-execution)
10. [Post-Exploitation: Interacting with the Shell](#10-post-exploitation-interacting-with-the-shell)
11. [Quick Command Reference](#11-quick-command-reference)

---

## 1. What Is MSF / Armitage?

| Tool                           | Description                                                                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **MSF (Metasploit Framework)** | A powerful penetration-testing framework used for network scanning, advanced enumeration/data collection, exploiting systems, gaining full control of a system, and maintaining long-term access |
| **Armitage**                   | The **graphical (GUI) version** of the Metasploit Framework — same underlying capability, visualized                                                                                             |

**Capabilities covered in this module:**

- Network scanning
- Advanced enumeration / information gathering
- Exploiting vulnerabilities on a target system
- Gaining complete control of a system
- Maintaining persistent access (e.g., via reverse shells / "lifetime passwords")

---

## 2. Core MSF Terminology: Auxiliaries, Exploits, Payloads

_(Analogy note: just as **Nmap** calls its enumeration helper programs "scripts," MSF has its own naming convention for its program categories.)_

| Term          | Purpose                                                                                                                                                           |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Auxiliary** | Programs that assist with **advanced information gathering / enumeration** (MSF's equivalent of Nmap "scripts")                                                   |
| **Exploit**   | Programs that use discovered vulnerabilities (e.g., default credentials, open ports, misconfigurations found during enumeration) to **fully compromise** a device |
| **Payload**   | Code that runs **after** access is gained, used to **maintain long-term access** to the compromised device (e.g., via **reverse shells**)                         |

**MSF's scale (from the console banner, at time of recording):**
| Category | Count |
|---|---|
| Exploits | 25,575 |
| Auxiliary modules ("olympias" — likely a transcription artifact for "auxiliary") | 1,317 |
| Payloads ("polls") | 1,680 |
| Post modules | 432 |
| Encoders | 49 |
| NOPs ("knobs") | 13 |
| Evasion techniques | 9 |

---

## 3. Installing Armitage

```bash
sudo su                 # obtain root permissions
apt update               # always update first, so the package can be found
apt install armitage     # confirm with 'y' when prompted
```

⚠️ **Troubleshooting note from the walkthrough:** If installation stalls with a network/connection error (e.g., "could not connect to any link"), first verify actual internet connectivity is working (e.g., by trying to load a website like YouTube in a browser) before assuming the tool itself is broken. Once connectivity is confirmed, simply **re-run the install command**.

---

## 4. Starting MSF Console & Understanding the Banner

```bash
msfconsole
```

- Launches the **Metasploit Framework console/environment**.
- On startup, it displays a **banner page** showing the software name, version, and a summary of loaded module counts (exploits, auxiliaries, payloads, post modules, encoders, NOPs, evasion techniques — see table in §2).

---

## 5. Connecting MSF to Its Database

**Why this matters:** MSF needs a database to **store and manage** all data collected/generated during a session (scan results, host info, etc.). This is handled via **PostgreSQL**.

**Step 1 — Check current DB connection status (inside `msfconsole`):**

```bash
db_status
```

- If not connected, it will report **no connection**.

**Step 2 — Exit MSF and initialize the database:**

```bash
exit                          # exits msfconsole (stay in root — don't exit root)
msfdb init                    # "msfdb" = Metasploit Framework database tool; "init" = initiate/create the connection
```

- This creates and connects the MSF database to the PostgreSQL service running on the machine.

**Step 3 — Re-verify the connection:**

```bash
msfconsole
db_status
```

- Should now confirm: **connected to the database tool**.

> ✅ Once `db_status` confirms connection, MSF/Armitage is ready for full use — including storing enumeration/scan data properly.

---

## 6. Full Reboot-Safe Startup Sequence

⚠️ **Important:** After completing the Armitage setup, **reboot the machine**. Without a reboot, the tool may have trouble reading/processing data correctly going forward.

**After every reboot, follow this sequence to avoid errors:**

```bash
sudo su                              # 1. Get root permissions
service postgresql start             # 2. Start the PostgreSQL database service
msfdb reinit                         # 3. Reinitialize the MSF database (recreates it if needed, resolving potential issues)
msfconsole                           # 4. Launch the MSF console
db_status                            # 5. Confirm the database connection succeeded
```

---

## 7. Scanning the Victim via `db_nmap`

**Goal:** Run an Nmap scan **directly from within MSF**, so the results are automatically stored in the MSF/Armitage database (rather than scanning separately and importing later).

```bash
db_nmap -sV -p- -O 192.168.56.103
```

| Flag              | Meaning                                                                                                                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `db_nmap`         | Runs an Nmap scan **and stores the results directly into MSF's connected database**                                                                                                                                 |
| `-sV` (capital V) | **Version scan** — also requests **verbose** output so scan progress is visible _(the transcript notes "-V" for verbose alongside the version scan intent — treat this as version detection + progress visibility)_ |
| `-p-`             | Scan **all ports** (0–65535) — the `-p` flag followed by a dash as a shortcut for the full port range                                                                                                               |
| `-O` (capital O)  | **OS detection** — attempt to identify the target's operating system                                                                                                                                                |
| `192.168.56.103`  | The victim/target machine's IP address                                                                                                                                                                              |

**Result:** All discovered victim data (open ports, services, OS, etc.) is saved directly into the MSF/Armitage-connected database, ready to be visualized once Armitage is launched.

---

## 8. Launching Armitage & Viewing the Victim

**Steps:**

1. Open a **new terminal tab** (keep the scan/db running in the original).
2. Obtain **root permissions** again in the new tab.
3. Once the scan from §7 completes, run:
   ```bash
   armitage
   ```
4. Armitage's GUI opens and **automatically displays the scanned victim device** (using the data already stored in the database from the `db_nmap` scan) — example shown: victim at `192.168.56.103`.

**Inspecting the victim in Armitage:**

- **Right-click the victim device icon → Services** — displays all data collected so far (open ports, service banners, etc.).
- **Hovering over the device** shows quick info such as the detected **OS version** (example: Linux `2.6.x`).

> **Key benefit demonstrated:** Everything shown graphically in Armitage could equally be done from the MSF command line — Armitage is purely a **visual/GUI convenience layer** over the same underlying framework and data.

---

## 9. Hands-On Exploitation: VSFTPD Backdoor Command Execution

**Discovery:** From the scan results (equivalent to what an Nmap script/enumeration would reveal), a known vulnerable service was identified: **VSFTPD Backdoor Command Execution** — a well-known exploit module matching the target's detected FTP service/version.

**Launching the exploit in Armitage:**

1. **Drag-and-drop** (or **double-click**) the matching exploit module (`vsftpd backdoor command execution`) onto/against the victim device.
2. Armitage **auto-populates most exploit configuration fields** based on the stored scan data.

**⚠️ Manual correction required — the `LHOST` field:**

- By default, the **LHOST** (local/attacker host) field may auto-populate using the wrong network interface's IP (e.g., an `eth1`-associated address instead of the correct one).
- **Fix:** Manually change `LHOST` to the **correct Kali (attacker) machine IP** — example used: `192.168.56.102`.

**Field summary:**
| Field | Meaning | Example Value |
|---|---|---|
| **LHOST** | The **attacker's (Kali) IP address** — where the reverse connection should come back to | `192.168.56.102` |
| **RHOST** | The **victim's IP address** — the target being exploited | `192.168.56.103` |
| **RPORT** | The **victim's port** the vulnerable service is running on | (as detected by the scan) |

3. Click **Launch**.
4. **Success indicator:** The victim device's **icon/graphic in Armitage visually changes** to indicate it has been **compromised** — different appearance from an un-exploited host.

---

## 10. Post-Exploitation: Interacting with the Shell

**Steps after successful exploitation:**

1. **Right-click** the now-compromised victim device.
2. **Shell → Interact** — opens an interactive command shell/session on the compromised machine.

**Verifying the compromise is genuine (proof-of-access check):**

```bash
ifconfig
```

- Confirm that the IP address shown in the output matches the victim's actual known IP (example: `192.168.56.103`) — proving you now have a working shell **on the target machine itself**, not your own attacker machine.

**Purpose from a defensive/ethical-hacking standpoint:**

> This process helps identify the exact flaw (in this case, an outdated/vulnerable **FTP server version**) so that it can be **remediated** — e.g., by **upgrading the FTP software version** to a patched release, closing off this specific attack vector.

**Further exploration encouraged:** Armitage's **Host** menu and various scan/module options offer many more capabilities beyond this single walkthrough — the instructor encourages continued hands-on exploration of the tool.

---

## 11. Quick Command Reference

| Command                              | Purpose                                                                                                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `apt update && apt install armitage` | Install Armitage                                                                                              |
| `msfconsole`                         | Launch the MSF console                                                                                        |
| `db_status`                          | Check whether MSF is connected to its database                                                                |
| `exit`                               | Exit `msfconsole` (without necessarily exiting root shell)                                                    |
| `msfdb init`                         | Initialize/create the MSF database connection (first-time setup)                                              |
| `service postgresql start`           | Start the PostgreSQL service (needed after every reboot)                                                      |
| `msfdb reinit`                       | Reinitialize the MSF database (post-reboot / troubleshooting)                                                 |
| `db_nmap -sV -p- -O <target-ip>`     | Run an Nmap scan (version detection, all ports, OS detection) with results saved directly into MSF's database |
| `armitage`                           | Launch the Armitage GUI (uses the same MSF database)                                                          |
| `ifconfig` (inside a gained shell)   | Verify you're operating on the target machine, by confirming its IP address                                   |

---

## Key Takeaways

- **MSF = command-line framework; Armitage = its GUI**, both backed by the **same underlying database and modules**.
- Three core MSF module categories: **Auxiliary** (enumeration), **Exploit** (compromise using found vulnerabilities), **Payload** (maintain access post-compromise).
- MSF requires a working **PostgreSQL-backed database connection** (`msfdb init` / `msfdb reinit` + `db_status`) to store and use scan/session data properly — and a **reboot** after initial Armitage setup is recommended.
- `db_nmap` lets you scan a target **directly into** MSF's database, so Armitage can immediately visualize the results.
- When launching an exploit, **always double-check auto-populated fields like `LHOST`** — Armitage may default to the wrong network interface.
- A successful exploit changes the target's **visual state** in Armitage, and grants an interactive shell — always verify actual access (e.g., via `ifconfig`) rather than assuming success from the GUI indicator alone.
- The end goal in an ethical/defensive context is identifying **exactly which vulnerability** (e.g., outdated FTP software) enabled the compromise, so it can be **patched/upgraded**.
