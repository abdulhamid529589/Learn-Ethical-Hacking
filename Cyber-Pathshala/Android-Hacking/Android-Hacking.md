# Android APK Pentesting & Reverse Engineering — Course Notes (Part 1)

> **Source:** CPCS Cyber Pathshala — Android/APK Pentesting Course (Part 1 transcript)
> **Scope:** Android OS fundamentals, APK structure, static/dynamic analysis tools, decompilation workflows, lab environment setup, and a guided walkthrough using the intentionally vulnerable **InsecureBankV2** app.
> **Ethics note:** Every technique below is demonstrated here strictly for testing on **your own devices, your own VMs (e.g., Metasploitable2, Android-x86 in VirtualBox), and intentionally vulnerable practice apps (e.g., InsecureBankV2)**. Never run these steps against apps, servers, or devices you do not own or do not have explicit written authorization to test.

---

## Table of Contents

1. [Introduction & Learning Path](#1-introduction--learning-path)
2. [Android OS Fundamentals](#2-android-os-fundamentals)
   - [2.1 What Android Actually Is](#21-what-android-actually-is)
   - [2.2 The Linux Kernel Connection](#22-the-linux-kernel-connection)
   - [2.3 Open Source Software (OSS) Concept](#23-open-source-software-oss-concept)
3. [Understanding APK Files](#3-understanding-apk-files)
   - [3.1 What Is an APK?](#31-what-is-an-apk)
   - [3.2 APK Internal Structure](#32-apk-internal-structure)
   - [3.3 App Types & Storage Locations](#33-app-types--storage-locations)
4. [AndroidManifest.xml Deep Dive](#4-androidmanifestxml-deep-dive)
5. [Core APK Components](#5-core-apk-components)
   - [5.1 Intents](#51-intents)
   - [5.2 Broadcast Receivers](#52-broadcast-receivers)
   - [5.3 Services](#53-services)
   - [5.4 Activities](#54-activities)
6. [Android Architecture Layers](#6-android-architecture-layers)
7. [Mobile Malware Scanning with MVT](#7-mobile-malware-scanning-with-mvt)
8. [App Vulnerability Scanning with MobSF](#8-app-vulnerability-scanning-with-mobsf)
9. [Decompiling APKs — 5 Methods](#9-decompiling-apks--5-methods)
10. [Building an Android Test Lab (VirtualBox)](#10-building-an-android-test-lab-virtualbox)
11. [Connecting & Controlling Devices with ADB](#11-connecting--controlling-devices-with-adb)
12. [Practical Walkthrough: InsecureBankV2 — Insecure Logging Vulnerability](#12-practical-walkthrough-insecurebankv2--insecure-logging-vulnerability)
13. [Kali NetHunter on Stock Android (Termux Method)](#13-kali-nethunter-on-stock-android-termux-method)
14. [Tool Reference Table](#14-tool-reference-table)
15. [Glossary](#15-glossary)
16. [Practice Tasks / Lab Checklist](#16-practice-tasks--lab-checklist)

---

## 1. Introduction & Learning Path

This course (Part 1) builds the **foundation** for Android application penetration testing / bug bounty work. Part 1 covers:

- How Android works internally (OS, kernel, architecture layers)
- APK file structure and manifest analysis
- Reverse engineering / decompiling APKs (multiple methods, from fully manual to fully automated)
- Static and dynamic analysis tooling (MVT, MobSF)
- Building a safe, isolated lab (VirtualBox + Android-x86, Kali Linux, ADB)
- A live vulnerability walkthrough (insecure logging / plaintext credential leakage) using the **InsecureBankV2** training app

Part 2 (referenced but not covered in this transcript) goes deeper into practical exploitation.

> ⚠️ The course frames some content around "cracking," "mod APKs," and bypassing OTP/credential protections. These framings describe **illegal activity if performed against apps/services you don't own or don't have permission to test**. The technical skills (decompiling, static/dynamic analysis, log inspection) are legitimate mobile-AppSec skills when applied to your own apps, VMs, and authorized bug bounty / CTF targets only.

---

## 2. Android OS Fundamentals

### 2.1 What Android Actually Is

- The physical phone is just **hardware** — it provides resources (CPU, memory, sensors, radios) but cannot do anything on its own.
- **Android is the operating system** that runs on top of that hardware, comparable to Linux, Windows, or macOS on a computer.
- The OS's core job: translate **human/user intent** into **hardware-understandable instructions**, and manage the hardware on the user's behalf.

### 2.2 The Linux Kernel Connection

- Android is built on the **Linux kernel**.
- The kernel's job is to talk to hardware directly, but it does so in **machine language**, which isn't practical for end users.
- An abstraction/interface layer (GUI + command line) was built on top of the kernel — this became the operating system.
- Android's OS layer built on the Linux kernel is known as **AOSP — Android Open Source Project**.

### 2.3 Open Source Software (OSS) Concept

| Aspect                       | Closed/Proprietary Software               | Open Source Software (OSS)                                   |
| ---------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| Source code visibility       | Hidden/compiled to bytecode, unreadable   | Publicly available, human-readable                           |
| Distribution                 | Sold, licensed                            | Freely shared (e.g., on GitHub)                              |
| Improvement speed            | Slow — only original developer updates it | Fast — global community contributes changes                  |
| Motive                       | Profit                                    | Community benefit / collective improvement                   |
| Example outcome after 1 year | Minor updates                             | Major feature/technology improvements (e.g., AI integration) |

Android/AOSP follows the OSS model — its source is publicly available, which is why it evolves rapidly and has many maintained versions (currently stewarded by Google).

---

## 3. Understanding APK Files

### 3.1 What Is an APK?

- **APK = Android Package Kit** (also called Android application package).
- Functionally equivalent to a `.exe` on Windows — it's the packaged format for an installable Android app.
- Built using languages such as **Java, Kotlin, C++**.
- **An APK is fundamentally a ZIP archive** with additional signing/security metadata layered on top.

### 3.2 APK Internal Structure

**Practical: inspecting an APK's contents**

```bash
# Rename .apk to .zip, then extract
mv app.apk app.zip
unzip app.zip -d app_extracted
```

Inside the extracted folder you'll typically find:

| Component             | Purpose                                                                             |
| --------------------- | ----------------------------------------------------------------------------------- |
| `AndroidManifest.xml` | Declares app structure, permissions, components (binary XML format by default)      |
| `classes.dex`         | Compiled/secured source code (Dalvik bytecode) — not human-readable until converted |
| `res/` (Resources)    | Images, icons, videos, layout/graphics assets                                       |
| `META-INF/`           | Metadata, certificates, and file signatures                                         |
| `lib/`                | Native libraries (if any)                                                           |

### 3.3 App Types & Storage Locations

| App Type                        | Description                                                                            | Storage Path Pattern          |
| ------------------------------- | -------------------------------------------------------------------------------------- | ----------------------------- |
| **Pre-installed (System) apps** | Shipped with the device by the manufacturer (Camera, Gallery, File Manager, Browser)   | `/data/system/app/<AppName>/` |
| **User-installed apps**         | Installed later by the user via Play Store or sideloading (WhatsApp, Instagram, games) | `/data/data/<AppName>/`       |

> Both ultimately sit under a top-level `data` directory, then branch into `system` (pre-installed) vs. direct app folders (user-installed).

---

## 4. AndroidManifest.xml Deep Dive

The manifest is the **first file analyzed** in any APK pentest — it's the blueprint of the entire app. It contains:

1. **Unique package name** — e.g., `com.vl.flappybird`; uniquely identifies the app and its data folder.
2. **Version information** — Java version, Android Studio version, XML schema versions used to build the app.
3. **Definitions** — declared activities (pages/screens) and what the app is structurally capable of doing.
4. **Permission definitions** — camera, storage, location, contacts, etc. that the app requests from the OS.
5. **Shared User ID** — a unique ID assigned per app/user for internal resource-sharing.
6. **Preferred installation location** — where app data/files are physically stored.
7. **UI information** — launcher icon, graphics/branding assets.
8. **External library/package references** — third-party SDKs used (e.g., Google Maps SDK), useful for identifying supply-chain / third-party attack surface.

**Example manifest skeleton (illustrative, not from a specific real app):**

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.demoapp">

    <application android:icon="@mipmap/ic_launcher" android:label="DemoApp">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="33" />
    <!-- Permission tags would appear here, e.g.: -->
    <!-- <uses-permission android:name="android.permission.CAMERA" /> -->
</manifest>
```

---

## 5. Core APK Components

Four building blocks make up nearly every Android app's internal logic:

### 5.1 Intents

- Internal **messaging system** used for communication between app screens, between apps, or between an app and the OS.
- Two categories in practice: explicit (targeting a specific component) and implicit (broadcast-style requests handled by whichever component can service them).

### 5.2 Broadcast Receivers

- Components that **sit idle and "listen"** for a system-wide or app-wide event, then react.
- Analogy used in the course: like an FM radio receiver picking up a public broadcast, or ARP broadcasting "who has this IP?" and only the relevant host replying.
- Real-world example: **Truecaller** — a broadcast receiver detects an incoming call event and triggers Truecaller's caller-ID overlay before you answer.

### 5.3 Services

- Background components managing ongoing device tasks: **Bluetooth, Wi-Fi, music playback, background updates**, etc. — anything running without a foreground UI.

### 5.4 Activities

- Each app "page" or discrete function is an **Activity** — analogous to a distinct page on a website.
- Examples: `MainActivity` (first screen), `LoginActivity`, `LogoutActivity`, `ForgotPasswordActivity`.

| Component          | Role                                   | Real-world analogy   |
| ------------------ | -------------------------------------- | -------------------- |
| Intent             | Message/data passing                   | Internal mail system |
| Broadcast Receiver | Listens & reacts to system-wide events | Radio receiver       |
| Service            | Manages background/ongoing tasks       | Background daemon    |
| Activity           | A single screen/function               | A webpage            |

---

## 6. Android Architecture Layers

From the **bottom (hardware-closest) to top (user-facing)**:

1. **Linux Kernel** — lowest layer; manages hardware directly via **drivers** (camera driver, Wi-Fi driver, Bluetooth driver, etc.). Considered the most secure and hardest layer to access.
2. **HAL / HIDL (Hardware Abstraction Layer)** — relays hardware requests from upper layers down to the kernel/drivers (e.g., "turn on the camera").
3. **Native Libraries** — system libraries supporting core functions like data storage and memory management (e.g., **SQLite** for local databases).
4. **ART (Android Runtime)** — executes the compiled bytecode (previously handled by **Dalvik**), running the app's actual instructions.
5. **Android Framework** — manages high-level app logic: activities, intents, content providers, broadcast receivers.
6. **Apps Layer** — the actual installed applications (system + user), the topmost layer users interact with.

**Example flow (tap camera shutter button):**

```
User taps "capture"
   → Android Framework triggers CameraActivity
   → ART executes the compiled instruction
   → Native libraries confirm resource availability
   → HAL forwards the hardware request
   → Linux Kernel driver operates the camera hardware
   → Image data flows back up through the same chain
```

> **Pentest tip from the course:** vulnerability analysis typically starts at the **Android Framework** layer (app-level logic, manifest, permissions) and works progressively deeper toward the kernel, which is the hardest and most sensitive layer to reach.

---

## 7. Mobile Malware Scanning with MVT

**MVT = Mobile Verification Toolkit** — scans a device (Android or iOS) against known **Indicators of Compromise (IOCs)** to detect malware/spyware infection signatures.

### Installation (Kali Linux)

```bash
sudo su
pip install mvt
```

Common install errors and fixes (Python package conflicts):

```bash
# If pip refuses to uninstall an old package (example: tld, python-dateutil, pydantic):
locate <package_name>          # find the conflicting package folder
cd /usr/lib/python3/dist-packages/
rm -rf <old_package_folder>    # remove only the specific conflicting version
pip install mvt                # re-run install; repeat for each conflict until it completes
```

> This is a normal part of Python environment troubleshooting — each error names the exact package to remove, and the fix is always "locate → cd → rm -rf → retry".

### Usage

```bash
mvt android --help              # or: mvt ios --help
mvt android download-iocs       # pulls the latest IOC/signature database
mvt android check-adb           # scans a connected Android device over ADB
```

### Connecting a device for scanning

1. On the Android device: **Settings → About Phone → tap Build Number 7×** to unlock Developer Options.
2. **Settings → System → Developer Options → enable USB Debugging** (and Wireless Debugging if preferred).
3. Connect via USB cable; **allow the USB debugging prompt** on the device.
4. On Kali: `adb devices` to confirm connection (install ADB first if missing: `apt install adb`).
5. Run `mvt android check-adb` — it can create a device backup and generate a compromise report.

> The report can be lengthy/technical — the course suggests pasting it into an AI assistant to summarize findings, which is a reasonable practice as long as you don't paste sensitive personal data from someone else's device.

---

## 8. App Vulnerability Scanning with MobSF

**MobSF (Mobile Security Framework)** performs automated static/dynamic analysis on Android, iOS, and Windows app packages. A hosted web version exists so no local install is required.

### Workflow

1. Go to the MobSF web app and **upload/drag-and-drop** an APK or IPA file.
2. MobSF automatically decompiles and analyzes the binary.
3. **View Report** gives you:
   - App metadata (name, size, hashes/signatures)
   - **Security warnings**, e.g.:
     - Use of insecure APIs
     - Use of insecure random number functions (weak cryptography)
     - Information leakage through login functions or WebView components
   - Extracted **server IP addresses** — useful for identifying C2 infrastructure if analyzing a malicious/unknown sample
   - Extracted **URLs and embedded emails** — useful for footprinting/OSINT
   - Full listing of internal files/folders and their paths

| Use case                                                                                | What MobSF reveals                                                          |
| --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Testing your own app                                                                    | Insecure API usage, weak crypto, leaky logs, embedded secrets               |
| Analyzing a suspicious/unknown APK (only in an isolated lab, never on personal devices) | Attacker infrastructure IPs, suspicious permissions, obfuscation indicators |

---

## 9. Decompiling APKs — 5 Methods

The course teaches 5 approaches, progressing from **fully manual → fully automated**.

### Method 1 — Fully Manual (dex2jar + JD-GUI)

```bash
# 1. Download target APK (example: InsecureBankV2 from GitHub "raw" download)
# 2. Rename and extract
mv InsecureBankv2.apk InsecureBankv2.zip
unzip InsecureBankv2.zip

# 3. Install dex-to-jar converter
apt install dex2jar

# 4. Convert classes.dex to a .jar
d2j-dex2jar classes.dex

# 5. Install a Java bytecode viewer
apt install jd-gui

# 6. Open the generated jar in JD-GUI
jd-gui classes-dex2jar.jar
```

Inside JD-GUI, navigate the package tree (e.g., `com → android → insecurebankv2 → ...`) to read the decompiled Java source in clear text.

### Method 2 — Semi-Automated (Bytecode Viewer)

```bash
apt install bytecode-viewer
bytecode-viewer     # then drag-and-drop the APK directly into the tool
```

Bytecode Viewer automatically handles renaming, extraction, decompilation, and resource decoding (including a readable `AndroidManifest.xml`) in one step.

### Methods 3–5 (mentioned as further automation levels)

The course indicates progressively more automated pipelines exist beyond Bytecode Viewer (e.g., `apktool`, `jadx`, or MobSF's built-in decompilation covered in Section 8) — worth exploring as extensions to this lab:

```bash
# jadx - fast, widely used decompiler (not explicitly named in transcript but a natural next tool)
apt install jadx
jadx-gui InsecureBankv2.apk

# apktool - decodes resources + smali disassembly
apktool d InsecureBankv2.apk -o decoded_output/
```

| Method         | Tooling                  | Effort                       | Output readability              |
| -------------- | ------------------------ | ---------------------------- | ------------------------------- |
| 1. Manual      | `dex2jar` + `jd-gui`     | High (multiple manual steps) | Java source                     |
| 2. Semi-auto   | Bytecode Viewer          | Low (drag & drop)            | Java source + decoded resources |
| 3–5. Automated | `apktool`, `jadx`, MobSF | Very low                     | Java/Smali + resources + report |

---

## 10. Building an Android Test Lab (VirtualBox)

Goal: run a full Android OS as a VM, networked alongside a Kali Linux VM, for safe/isolated testing.

### Steps

1. Install **VirtualBox** (or VMware) on your host OS.
2. Obtain an **Android-x86 ISO/VDI** image (a prebuilt virtual hard disk).
3. In VirtualBox: **New Machine** → name it (e.g., "Android") → OS type: **Linux** → a Linux distribution variant.
4. Skip unattended installation and guest user creation.
5. Allocate resources appropriate to your host: e.g., **2–4 GB RAM**, **2–3 CPU cores**.
6. For the virtual disk step, choose **"Use an existing virtual hard disk"** and browse to the downloaded `.vdi` file (don't create a new blank one).
7. Finish, then start the VM.

### Fixing the "console only / no GUI" issue

```
Power off the VM
→ Settings → Display → Graphics Controller
→ Switch from "VMSVGA" to "VBoxSVGA" (or the alternate working option)
→ Save, then restart the VM
```

### Networking the Android VM with Kali

1. In the Android VM's Wi-Fi settings, confirm it's on a **Host-Only** or virtual network adapter.
2. In Kali's VirtualBox network settings, set **Adapter 1 → NAT** (internet) and **Adapter 2 → Bridged/Host-Only** to match Android's network (so the two VMs can reach each other).
3. Confirm both machines are on the **same IP subnet** via `ifconfig` (Kali) and Wi-Fi → Advanced details (Android).

> Releasing a captured VM cursor: **Right Ctrl** (VirtualBox) or **Ctrl+Alt** (VMware Workstation).

---

## 11. Connecting & Controlling Devices with ADB

**ADB = Android Debug Bridge** — Google's official tool for remotely controlling/debugging an Android device from a computer.

```bash
apt install adb
adb devices                 # list connected devices
adb connect <device_ip>     # connect over network (both must match subnet)
adb disconnect               # disconnect current sessions
adb kill-server              # reset the ADB server if a connection error occurs
adb install <app.apk>        # install an APK to the connected device
adb logcat                    # stream live device logs (crucial for vulnerability hunting)
```

Enabling **USB/Wireless Debugging** on a real device: **Settings → About Phone → tap Build Number ×7 → System → Developer Options → toggle USB Debugging**.

---

## 12. Practical Walkthrough: InsecureBankV2 — Insecure Logging Vulnerability

**InsecureBankV2** is a well-known, intentionally vulnerable Android banking app built specifically for security training (safe and legal to test against).

### Step-by-step

```bash
# 1. Download & extract the training app + its companion server
git clone <InsecureBankV2 repo>   # or download the release zip
unzip InsecureBankV2-master.zip
cd InsecureBankV2-master/AndroLabServer

# 2. The bundled server requires Python 2.7
apt install python2.7

# 3. Install pip for Python 2 if missing
python2.7 get-pip.py

# 4. Install required packages
python2.7 -m pip install setuptools
python2.7 -m pip install -r requirements.txt

# 5. Start the vulnerable app's backend server (listens on port 8888 by default)
python2.7 app.py
```

```bash
# 6. Install the APK on your Android test VM/device
adb install InsecureBankv2.apk

# 7. In the app: Settings (⋮) → Preferences → set Server IP to your Kali machine's IP
#    (verify with `ifconfig` on Kali)
```

### Capturing the vulnerability via logs

```bash
# On Kali:
adb connect <android_vm_ip>
adb logcat
```

- Log in to the app using the training credentials provided in the app's repo README (e.g., a demo username/password pair).
- Watch the `logcat` stream — scroll to the **login activity** entries.
- **Finding:** the username and password appear in **clear text inside the system logs**, even though the transmission itself may look "hidden" to the end user.

**Why this matters (the core AppSec lesson):**

- Logging sensitive data (credentials, OTPs, tokens) in plaintext is a common real-world mobile vulnerability class — it falls under **OWASP Mobile Top 10 → "Insecure Data Storage / Insufficient Cryptography / Information Leakage"**.
- Any process or app on a rooted device (or an attacker with physical/ADB access) could read these logs and harvest credentials.
- **Fix (for developers):** never log PII/credentials; use secure storage (Android Keystore); scrub or disable verbose logging in production builds; ensure ProGuard/R8 strips debug logs.

---

## 13. Kali NetHunter on Stock Android (Termux Method)

A way to get a portable Linux/pentesting environment on an unrooted Android device.

```bash
# 1. Install Termux from F-Droid or the official GitHub releases page
#    (avoid the Play Store version — its repos are often outdated)

# Android settings first (avoid background-kill issues):
# Settings → Apps → Termux → Battery → disable battery optimization
# Settings → Apps → Termux → Permissions → allow Storage

# 2. Inside Termux:
pkg update
apt update
pkg upgrade
apt install wget

# 3. Download the NetHunter installer script
wget -O install https://<installer-script-url>
chmod +x install
./install
# Choose: full / minimal / nano rootfs when prompted
```

After installation:

```bash
nh          # launch NetHunter shell
sudo su     # (blank password by default) - operates within the contained rootfs only, no device root needed
```

### Accessing it graphically via VNC

```bash
nh kex           # or: nh -x, depending on version
kex passwd        # set a VNC password
kex               # start the VNC server (commonly on display :1 / port 5901)
```

Then, install a VNC viewer app (e.g., "NetHunter KeX Client" or any VNC client), connect to `127.0.0.1:5901` with the password you set.

> This gives you a self-contained Kali-like environment for practicing command-line tools directly on a phone — useful for on-the-go recon/practice, though a full VM (Section 10) is better for heavier tasks like MobSF or Burp Suite.

---

## 14. Tool Reference Table

| Tool                              | Purpose                                                   | Platform                 |
| --------------------------------- | --------------------------------------------------------- | ------------------------ |
| `dex2jar`                         | Convert Dalvik bytecode (`classes.dex`) → readable `.jar` | Linux/Kali               |
| `jd-gui`                          | Graphical Java decompiler/viewer                          | Linux/Kali               |
| Bytecode Viewer                   | All-in-one decompiler (drag & drop APK)                   | Linux/Kali               |
| `apktool`                         | Decode resources + disassemble to Smali                   | Linux/Kali               |
| `jadx` / `jadx-gui`               | Fast modern APK-to-Java decompiler                        | Linux/Kali               |
| MVT (Mobile Verification Toolkit) | Malware/spyware IOC scanning                              | Linux/Kali               |
| MobSF                             | Automated static/dynamic mobile app security testing      | Web / self-hosted        |
| ADB                               | Device control, install, logging, debugging               | Linux/Kali/Windows/macOS |
| Termux + NetHunter                | Linux pentest environment on Android itself               | Android (no root needed) |
| VirtualBox/VMware + Android-x86   | Isolated Android test VM                                  | Host OS                  |

---

## 15. Glossary

| Term                               | Definition                                                                   |
| ---------------------------------- | ---------------------------------------------------------------------------- |
| **AOSP**                           | Android Open Source Project — the open-source codebase Android is built from |
| **APK**                            | Android Package Kit — the installable app file format (a structured ZIP)     |
| **ART**                            | Android Runtime — executes compiled app bytecode                             |
| **Activity**                       | A single screen/function within an app                                       |
| **ADB**                            | Android Debug Bridge — CLI tool for controlling/debugging Android devices    |
| **Broadcast Receiver**             | Component that listens for and reacts to system/app-wide events              |
| **Dalvik / DEX**                   | Android's (legacy) bytecode format compiled from Java/Kotlin source          |
| **HAL/HIDL**                       | Hardware Abstraction Layer — relays software requests to kernel drivers      |
| **Intent**                         | Internal messaging mechanism between app components                          |
| **IOC**                            | Indicator of Compromise — a signature/pattern suggesting malware infection   |
| **Manifest (AndroidManifest.xml)** | Declares an app's structure, permissions, and components                     |
| **MobSF**                          | Mobile Security Framework — automated mobile app vulnerability scanner       |
| **MVT**                            | Mobile Verification Toolkit — scans devices for spyware/malware IOCs         |
| **OSS**                            | Open Source Software — publicly available, freely modifiable source code     |
| **Service**                        | Background component managing ongoing tasks (Bluetooth, Wi-Fi, playback)     |
| **VNC**                            | Virtual Network Computing — remote graphical desktop protocol                |

---

## 16. Practice Tasks / Lab Checklist

Use these against **your own apps, InsecureBankV2, and Metasploitable2** only.

- [ ] Set up an Android-x86 VM in VirtualBox networked with your Kali VM.
- [ ] Extract one of your own test APKs and manually identify: package name, permissions requested, declared activities (via manifest).
- [ ] Decompile the same APK using **all three** methods: manual (`dex2jar`+`jd-gui`), Bytecode Viewer, and `jadx`. Compare output readability/speed.
- [ ] Run your own test APK through the **MobSF** web scanner and document every warning it raises.
- [ ] Run **MVT** against your Android test VM and review the generated report.
- [ ] Deploy **InsecureBankV2** + its companion Python server; reproduce the plaintext-credential logging vulnerability using `adb logcat`.
- [ ] Write a short vulnerability report (as if for a bug bounty submission) describing the insecure-logging issue: severity, root cause, proof-of-concept steps, and remediation.
- [ ] (Extension) Point MobSF or `jadx` at a deliberately vulnerable open-source app (e.g., DIVA, GoatDroid) and catalog every OWASP Mobile Top 10 category you can identify.
- [ ] (Extension) Set up Kali NetHunter via Termux on a spare/rooted-not-required Android device and confirm `nh kex` graphical access works.

---

_End of Part 1 notes. Part 2 (referenced in the course) is expected to cover deeper exploitation techniques — recommend creating a follow-up file `Android_APK_Pentesting_Course_Notes_Part2.md` once that transcript is available, to keep the index consistent with your other course files._
