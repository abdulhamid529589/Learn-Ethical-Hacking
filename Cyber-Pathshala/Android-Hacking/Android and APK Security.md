# Android & APK Security Testing — Course Notes

## https://www.youtube.com/watch?v=q_5p74k6kp4

> Compiled study notes on Android OS fundamentals, APK structure, and mobile application security testing (MAST) methodology. Organized for quick revision.

> **Scope & Ethics Note:** This material covers Android internals and app security assessment techniques (reverse engineering, static/dynamic analysis) for the purpose of **authorized security testing** — testing apps you own, apps you have explicit written permission to test, or apps within a scoped bug bounty program. The same skills taught here (decompiling, reading source code, traffic/log analysis) are foundational to legitimate security research and are taught in every professional mobile-security certification (e.g., certifications covering mobile app pentesting). They should never be used against third-party software without authorization, and should not be used to bypass software licensing/DRM on commercial applications.

---

## 📑 Index / Table of Contents

- [1. Android OS Fundamentals](#1-android-os-fundamentals)
  - [1.1 What Is Android?](#11-what-is-android)
  - [1.2 Open Source & AOSP](#12-open-source--aosp)
- [2. APK File Fundamentals](#2-apk-file-fundamentals)
  - [2.1 What Is an APK?](#21-what-is-an-apk)
  - [2.2 APK Internal Structure (Practical)](#22-apk-internal-structure-practical)
  - [2.3 Application Types & Storage Locations](#23-application-types--storage-locations)
- [3. AndroidManifest.xml Deep Dive](#3-androidmanifestxml-deep-dive)
  - [3.1 What the Manifest Contains](#31-what-the-manifest-contains)
  - [3.2 Example Manifest Walkthrough](#32-example-manifest-walkthrough)
- [4. Core APK Components](#4-core-apk-components)
  - [4.1 Intents](#41-intents)
  - [4.2 Broadcast Receivers](#42-broadcast-receivers)
  - [4.3 Services](#43-services)
  - [4.4 Activities](#44-activities)
- [5. Android Architecture (Layered Stack)](#5-android-architecture-layered-stack)
- [6. Mobile Security Testing Tools](#6-mobile-security-testing-tools)
  - [6.1 MVT — Mobile Verification Toolkit](#61-mvt--mobile-verification-toolkit)
  - [6.2 MobSF — Mobile Security Framework](#62-mobsf--mobile-security-framework)
- [7. APK Decompilation / Reverse Engineering](#7-apk-decompilation--reverse-engineering)
  - [7.1 Why Decompile an APK](#71-why-decompile-an-apk)
  - [7.2 Method 1 — Fully Manual](#72-method-1--fully-manual)
  - [7.3 Method 2 — Semi-Automated (Bytecode Viewer)](#73-method-2--semi-automated-bytecode-viewer)
  - [7.4 Other Common Tools (for reference)](#74-other-common-tools-for-reference)
- [8. Building an Android Test Lab](#8-building-an-android-test-lab)
  - [8.1 Android-in-VirtualBox Setup](#81-android-in-virtualbox-setup)
  - [8.2 Networking Kali ↔ Android](#82-networking-kali--android)
  - [8.3 ADB (Android Debug Bridge) Basics](#83-adb-android-debug-bridge-basics)
- [9. Practical Vulnerability Walkthrough: Insecure Logging](#9-practical-vulnerability-walkthrough-insecure-logging)
  - [9.1 The Target: InsecureBankV2](#91-the-target-insecurebankv2)
  - [9.2 Setting Up the Vulnerable Server](#92-setting-up-the-vulnerable-server)
  - [9.3 Capturing & Analyzing Logs](#93-capturing--analyzing-logs)
  - [9.4 The Finding & Remediation](#94-the-finding--remediation)
- [10. Kali NetHunter on a Stock Android Device](#10-kali-nethunter-on-a-stock-android-device)
- [11. Key Takeaways](#11-key-takeaways)

---

## 1. Android OS Fundamentals

### 1.1 What Is Android?

Android is **not** the physical device — it's the **operating system** running on top of the device's hardware, just as Windows/Linux/macOS run on a PC or laptop.

- The core job of any OS (including Android) is to act as a translator/manager between the user and the hardware.
- **Android is built on top of the Linux kernel** — the same kernel concept covered in general Linux fundamentals (a kernel is the software layer that directly communicates with hardware in machine-level instructions).
- On top of the Linux kernel, Android adds its own frameworks, libraries, and runtime (covered in Section 5).

### 1.2 Open Source & AOSP

- Android is released as **AOSP (Android Open Source Project)** — its source code is publicly available.
- **Open source concept (general):** a developer can either (a) sell compiled/obfuscated software where users can run it but not read the logic, or (b) publish the full source code publicly so a community can inspect, use, and improve it. Community-driven, continuously updated open-source projects tend to evolve much faster than closed proprietary ones because many contributors improve it in parallel.
- Android follows the open-source model; **Google** maintains and coordinates AOSP, but the underlying code is publicly viewable, which is part of why so many custom ROMs/forks exist.

---

## 2. APK File Fundamentals

### 2.1 What Is an APK?

- **APK = Android Package Kit** — the installable application file format on Android (analogous to `.exe` on Windows).
- Apps can be written in **Java, Kotlin, C++**, and other supported languages, which get compiled down into the APK.
- **Key insight:** an APK file is fundamentally just a **ZIP archive** with some added security/signing wrapped around it.

### 2.2 APK Internal Structure (Practical)

Demonstrated by renaming a downloaded `.apk` file to `.zip` and extracting it — this is a standard, harmless way to inspect any APK's structure (used constantly in legitimate app security assessments):

| Component                 | Contents                                                                                                    |
| ------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `AndroidManifest.xml`     | The app's structural blueprint (see Section 3)                                                              |
| `classes.dex`             | The compiled/secured source code (Dalvik Executable bytecode) — not human-readable until reverse-engineered |
| `res/` (Resources folder) | Images, icons, layout files, videos, and other assets                                                       |
| `META-INF/`               | Metadata files, including the app's digital signature information                                           |
| `lib/`                    | Native libraries (compiled C/C++ code, if used)                                                             |

**Practical steps demonstrated:**

1. Download any APK (the course used a sample game APK for demonstration).
2. Rename `filename.apk` → `filename.zip`.
3. Extract the ZIP.
4. Inspect the folders/files listed above.

### 2.3 Application Types & Storage Locations

| Type                            | Description                                                                                        | Typical Location               |
| ------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------ |
| **Pre-installed (system) apps** | Shipped with the device by the manufacturer (Camera, Gallery, default Browser, File Manager, etc.) | `/data/system/app/<app_name>/` |
| **User-installed apps**         | Installed later by the user via Play Store or sideloading (WhatsApp, Instagram, games, etc.)       | `/data/data/<app_name>/`       |

> Knowing these storage paths matters for security assessment because it tells you where to look for an app's local data, cache, shared preferences, and databases during dynamic analysis.

---

## 3. AndroidManifest.xml Deep Dive

The manifest is the **first file examined** in any Android app security assessment — it's the app's blueprint and reveals a huge amount about its structure and attack surface.

### 3.1 What the Manifest Contains

| Field/Section                             | Purpose                                                                                                                                    |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Package name**                          | Unique identifier for the app (e.g., `com.example.appname`); also names its data folder                                                    |
| **Version information**                   | SDK/build tool versions used to compile the app                                                                                            |
| **Component definitions**                 | Declares the app's Activities, Services, Broadcast Receivers, Intents/Intent Filters                                                       |
| **Permission definitions**                | Lists every device permission the app requests (camera, storage, location, contacts, etc.) — critical for assessing privacy/attack surface |
| **Shared User ID**                        | A unique ID that can let multiple apps share data/resources if signed with the same key                                                    |
| **Preferred install location**            | Where the app's data is stored                                                                                                             |
| **UI information**                        | Launcher icon, themes, and other display assets                                                                                            |
| **External library/package declarations** | Third-party SDKs/libraries used (e.g., a Maps SDK) — important for supply-chain/dependency risk review                                     |

### 3.2 Example Manifest Walkthrough

A typical manifest declares, in this general shape:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest package="com.example.app" ...>
    <application android:icon="@drawable/icon" ...>
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
    <uses-sdk android:minSdkVersion="..." />
    <!-- <uses-permission> entries would appear here if the app requests permissions -->
</manifest>
```

- The `<intent-filter>` + `MAIN`/`LAUNCHER` combination is what tells Android which Activity to open when the user taps the app icon.
- Any `<uses-permission>` tags reveal exactly what sensitive device capabilities the app can access — a key checklist item in any mobile app security review.

---

## 4. Core APK Components

Four building blocks make up virtually every Android app's behavior:

### 4.1 Intents

- A **messaging mechanism** for internal communication — between different screens/components within an app, or between one app and another (or the system).
- Used whenever an app needs to trigger an action, pass data, or hand off to another component.

### 4.2 Broadcast Receivers

- A **listener** that waits for a system-wide or app-wide "broadcast" event and reacts when it occurs.
- **Analogy:** like a radio receiver tuned to a frequency — the broadcast is sent out to everyone, but only receivers listening for that specific event respond.
- **Real-world example:** a caller-ID app registers a broadcast receiver for "incoming call" events; the moment that broadcast fires, the receiver triggers the app to show caller information before the call is even answered.

### 4.3 Services

- Background components that manage ongoing tasks without a user interface — e.g., playing music, handling background sync/updates, managing Bluetooth/Wi-Fi state changes.

### 4.4 Activities

- Represents a **single screen or user-facing function** with its own UI and logic — e.g., `MainActivity`, `LoginActivity`, `LogoutActivity`, `ForgotPasswordActivity`.
- Analogous to individual pages of a website; each meaningful screen/action in the app maps to an Activity.

> **Security relevance:** All four components can potentially be exported (made accessible to other apps) via manifest flags. Improperly exported Activities, Services, or Broadcast Receivers are among the most common Android app vulnerabilities — allowing malicious apps to invoke privileged functionality without going through the intended UI/authentication flow.

---

## 5. Android Architecture (Layered Stack)

Read from the bottom up (hardware → user-facing apps):

| Layer                                | Role                                                                                             |
| ------------------------------------ | ------------------------------------------------------------------------------------------------ |
| **Linux Kernel**                     | Directly interfaces with hardware via drivers (camera, Bluetooth, Wi-Fi, mic, etc.)              |
| **HAL (Hardware Abstraction Layer)** | Passes hardware requests from upper layers down to the kernel/drivers (e.g., "start the camera") |
| **Native Libraries**                 | System-level libraries for core functions (e.g., SQLite for local databases, media codecs, etc.) |
| **ART (Android Runtime)**            | Executes the app's compiled bytecode (replaces the older Dalvik runtime)                         |
| **Android Framework**                | Manages high-level app components — Activities, Intents, Content Providers, Broadcast Receivers  |
| **Applications**                     | The actual apps (system + user-installed) that the user interacts with                           |

**Example flow (opening the camera and taking a photo):**

1. User taps the Camera app icon → Android Framework invokes `MainActivity`/`CameraActivity`.
2. User taps "capture" → app logic executes.
3. ART runs the relevant bytecode.
4. The call passes down through native libraries → HAL → Linux kernel → camera driver.
5. Hardware captures the image; the result flows back up through the same layers to the app's UI.

> **Security relevance:** vulnerability research typically starts at the **Framework layer** (highest attack surface, most direct user interaction) and works downward. The **Linux kernel** is the deepest and generally most hardened/hardest-to-reach layer.

---

## 6. Mobile Security Testing Tools

### 6.1 MVT — Mobile Verification Toolkit

**MVT** is a real, well-known open-source forensic tool (originally developed by Amnesty International's Security Lab) used to check a mobile device for **Indicators of Compromise (IOCs)** — i.e., signs that spyware or other known malware has been installed on the device. It's widely used by digital forensics researchers and at-risk users (e.g., journalists, activists) to check whether their own phone has been targeted.

**Setup (Kali Linux):**

```bash
pip install mvt --break-system-packages
```

- Installation commonly hits dependency conflicts (older versions of packages like `tld`, `python-dateutil`, `pydantic`, `setuptools` already present on the system).
- **Resolution pattern:** locate the conflicting package with `locate <package_name>`, navigate to its install directory, remove the old version with `rm -rf <folder>`, then re-run the install — repeating for each conflicting dependency until installation completes cleanly.

**Usage:**

```bash
mvt android download-iocs      # fetches the latest indicator-of-compromise definitions
mvt android check-adb          # scans a connected Android device over ADB for IOCs
```

- Requires connecting the target device via **ADB** (see Section 8.3) with **Developer Options → USB Debugging** enabled on the phone.
- Produces a detailed report — long/technical enough that pasting it into an AI assistant for summarization is a reasonable practical tip for beginners.

> This tool exists specifically to **detect and defend against** malicious spyware on a device — it is a defensive/forensic tool, not an attack tool.

### 6.2 MobSF — Mobile Security Framework

**MobSF** is another well-known, open-source, automated mobile app security testing framework (static + dynamic analysis) used extensively in the industry for Android, iOS, and Windows app assessments.

- A hosted/live version is available in-browser — you simply drag-and-drop an `.apk` or `.ipa` file to scan it.
- **Typical report contents:**
  - App metadata (name, size, file hashes/signatures)
  - Static analysis warnings (e.g., use of insecure APIs, insecure random-number generation, insufficient cryptography, information leakage via login functions or WebView components)
  - Extracted **URLs, IP addresses, and email addresses** embedded in the app binary (useful both for legitimate infrastructure mapping during authorized testing, and for analyzing whether an app you've received is potentially malicious)
  - File/folder structure of the decompiled app

> Both dragging in your own apps for review and analyzing a suspicious APK someone sent you (without executing it) are legitimate, safe, and common use cases for this tool.

---

## 7. APK Decompilation / Reverse Engineering

### 7.1 Why Decompile an APK

To review an app's actual logic for a security assessment (e.g., checking how credentials are handled, whether encryption is implemented correctly, whether hardcoded secrets exist in the code) you need to convert the compiled, unreadable bytecode (`classes.dex`) back into readable source code.

### 7.2 Method 1 — Fully Manual

```bash
apt install dex2jar
apt install jd-gui

# Rename and extract the APK
mv app.apk app.zip
unzip app.zip

# Convert the bytecode to a readable JAR
d2j-dex2jar classes.dex

# Open the resulting JAR graphically
jd-gui
```

- `dex2jar` converts Dalvik bytecode (`classes.dex`) into a standard Java `.jar`.
- `jd-gui` provides a graphical Java decompiler to browse the resulting classes/packages and read the source in clear text.

### 7.3 Method 2 — Semi-Automated (Bytecode Viewer)

```bash
apt install bytecode-viewer
bytecode-viewer
```

- **Bytecode Viewer** automates the entire chain above: drag-and-drop the raw `.apk` file directly, and it handles unzipping, dex-to-jar conversion, and decompilation automatically, presenting the readable source (including the decoded `AndroidManifest.xml`) in one interface.

### 7.4 Other Common Tools (for reference)

The course didn't cover these, but they're standard companions in the same workflow: **APKTool** (for resource/smali decompiling and rebuilding), **JADX** (a modern, very popular APK-to-Java decompiler with a GUI), and **Frida** (for dynamic instrumentation during runtime analysis).

> **Reminder:** these tools are for reading and understanding an app's logic for security assessment or personal educational study on apps you're authorized to test — not for circumventing licensing/DRM on commercial software or redistributing modified copies of someone else's proprietary app.

---

## 8. Building an Android Test Lab

### 8.1 Android-in-VirtualBox Setup

For hands-on practice without needing a physical spare device, you can run an Android x86 image as a VirtualBox VM:

1. Install VirtualBox (or VMware) if not already installed.
2. Create a new VM → OS Type: **Linux** → Version: **Other Linux (64-bit)**.
3. Skip attaching an ISO (no unattended install needed if using a pre-built Android disk image).
4. Allocate RAM/CPU based on your host machine's specs (e.g., 2–4 GB RAM, 2–3 CPU cores is typically sufficient for a lightweight Android VM).
5. For the virtual hard disk step, choose **"Use an existing virtual hard disk file"** and select the downloaded Android disk image.
6. After first boot, if the graphical interface doesn't render correctly, go to **VM Settings → Display → Graphics Controller** and try switching between the available options (e.g., VBoxSVGA vs. the default) until the Android UI displays properly.

### 8.2 Networking Kali ↔ Android

For the Kali (tester) VM and the Android (target) VM to communicate:

- Both VMs' network adapters should be set to the **same mode** — e.g., both on **Host-Only**, or matching NAT/Bridged configurations — so they land on the same virtual subnet.
- Confirm each VM's IP via:
  - Kali: `ifconfig`
  - Android: Settings → Wi-Fi → (tap the connected network) → Advanced/Details → IP address
- Both addresses should share the same subnet prefix (e.g., both `192.168.56.x`) for them to reach each other directly.

### 8.3 ADB (Android Debug Bridge) Basics

ADB is Google's official tool for remotely interacting with an Android device from a computer — used constantly in legitimate app development, debugging, and security testing.

**Enabling on the target device:**

1. Settings → About Phone → tap the Build Number ~7 times → unlocks **Developer Options**.
2. Settings → System → Developer Options → enable **USB Debugging** (and/or **Wireless Debugging**).
3. Connect via USB cable (or Wi-Fi for wireless ADB) and accept the "Allow USB debugging?" prompt on the device.

**Common commands:**

```bash
apt install adb              # install ADB on Kali
adb devices                  # list connected/authorized devices
adb connect <device_IP>      # connect to a device over network ADB
adb disconnect                # disconnect current session
adb install <path/to/app.apk> # install an APK onto the connected device
adb logcat                    # stream the device's system/app logs in real time
adb kill-server               # restart the ADB server (useful if a connection is stuck)
```

---

## 9. Practical Vulnerability Walkthrough: Insecure Logging

### 9.1 The Target: InsecureBankV2

**InsecureBankV2** is a well-known, publicly published, **intentionally vulnerable** Android banking app created specifically for training purposes (similar in spirit to Metasploitable2 or DVWA, but for mobile). It ships with a companion Python test server and a README containing default demo credentials, precisely so students can practice mobile app security testing legally and safely.

### 9.2 Setting Up the Vulnerable Server

1. Download and extract the InsecureBankV2 project (includes an `AndroidLabServer` folder).
2. The bundled server runs on Python 2.7 (as required by this older, purpose-built training project):
   ```bash
   apt install python2.7
   python2.7 -m pip install -r requirements.txt   # may need get-pip.py first if pip isn't present
   python2.7 app.py                                # starts the lab server (default port 8888 in the course demo)
   ```
3. Install the APK onto the Android test device:
   ```bash
   adb install InsecureBankv2.apk
   ```
4. Inside the app, go to **Settings/Preferences** and point the app's server IP to your Kali machine's IP (the machine running `app.py`) so the app can authenticate against it.

### 9.3 Capturing & Analyzing Logs

```bash
adb connect <android_IP>
adb logcat
```

- With `logcat` streaming, log in to the app using the demo credentials provided in the project's README.
- Observe the live log output during the login attempt.

### 9.4 The Finding & Remediation

**Finding:** the login activity logs the **username and password in clear text** to the system log (`logcat`) on every login attempt — regardless of whether the login succeeds.

**Why this matters:** on a real device, any other app with log-reading permissions (on older/rooted Android versions) — or anyone with physical/debugging access to the device — could potentially read these logs and harvest credentials, OTPs, or other sensitive data that an app carelessly writes to the system log.

**Remediation (what a security report would recommend):**

- Never log sensitive data (credentials, tokens, OTPs, PII) at any log level, even in debug builds.
- Strip or disable verbose/debug logging in release builds.
- Use structured, redacted logging and enforce this via code review / static analysis (e.g., lint rules) before release.

> This is a textbook example of an **OWASP Mobile Top 10** category issue (insecure data storage / insufficient logging hygiene) — a defensive lesson about what developers must avoid, demonstrated safely on a purpose-built training app.

---

## 10. Kali NetHunter on a Stock Android Device

This section covers installing **Kali NetHunter** (Offensive Security's officially maintained Android penetration-testing environment) on a regular, non-rooted Android phone using **Termux** — a legitimate, widely-used open-source terminal emulator for Android. This is the same general installation path documented by the NetHunter project itself.

**High-level steps:**

1. Install **Termux** from its official source (F-Droid/GitHub) rather than the Play Store version, since the Play Store build can be outdated and unable to run standard package-manager commands.
2. Grant Termux storage permissions and disable battery optimization for it (Settings → Apps → Termux → Battery → allow background activity) so long-running installs don't get killed by the OS.
3. Update Termux's package manager:
   ```bash
   pkg update && pkg upgrade
   ```
4. Install `wget`, then download the official NetHunter installer script for Termux:
   ```bash
   pkg install wget
   wget -O install <official_nethunter_termux_installer_url>
   chmod +x install
   ./install
   ```
5. Choose the desired NetHunter build size (full/minimal) when prompted; the full image is several GB and will take time to download.
6. If a checksum/hash verification step errors out after a 100% download, this is a known cosmetic issue — simply re-run the installer; it will detect the existing download and continue/complete setup without re-downloading.
7. Launch the environment with `nh` (drops you into the NetHunter shell inside Termux).
8. For a graphical desktop instead of command-line-only access, install the companion **NetHunter KeX** app (VNC-based), set a VNC password via `kex passwd`, start the session with `kex start`, and connect to it using any VNC viewer pointed at `127.0.0.1:<port>` (default around `5901`).

> NetHunter is Offensive Security's own official, publicly documented mobile pentesting distribution — installing it on your own device to build a portable testing environment is standard practice for security students and professionals, no different from installing Kali Linux on a laptop.

---

## 11. Key Takeaways

- An APK is fundamentally a signed ZIP archive — understanding its structure (manifest, DEX bytecode, resources, metadata) is the foundation of Android app security assessment.
- `AndroidManifest.xml` is the first and most information-dense file to review in any assessment — it reveals permissions, components, and third-party dependencies.
- The four core components (Intents, Broadcast Receivers, Services, Activities) define an app's attack surface; improperly exported components are a leading vulnerability class.
- Tools like **MVT** and **MobSF** are legitimate, widely-used open-source security/forensics tools — used for defensive malware detection and for structured app vulnerability assessment, respectively.
- Reverse engineering (dex2jar/jd-gui, Bytecode Viewer, or industry-standard tools like JADX/APKTool) is a core, legitimate security-research skill — always apply it only to apps you own or are explicitly authorized to test.
- **InsecureBankV2** and similar purpose-built vulnerable apps let you practice real vulnerability classes (like clear-text credential logging) safely and legally.
- Building a personal lab (Android VM + Kali VM, or NetHunter on your own device) mirrors how professional mobile security testers set up their environments.

---

_End of notes._
