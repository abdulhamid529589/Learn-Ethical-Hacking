# 📱 Android Hacking & Mobile Forensics — Complete Course Notes

### (Bengali-English Mix, Based on WsCube Tech Android Ethical Hacking Course)

> এই notes টা বানানো হয়েছে একটা YouTube course transcript থেকে, যেটা Android architecture, APK structure, lab setup, ADB, MobSF, এবং mobile security নিয়ে। Course promotion-এর repeated lines বাদ দিয়ে শুধু technical content টা structured করা হয়েছে।

---

## 📚 Table of Contents

1. [AOSP আর Open Source Concept](#1-aosp-open-source)
2. [APK File কি, কিভাবে বানে](#2-apk-file)
3. [Android Architecture (Layers)](#3-android-architecture)
4. [AndroidManifest.xml এবং APK Components](#4-androidmanifestxml)
5. [Android App Components (Activity, Intent, Service, Broadcast Receiver, Content Provider)](#5-app-components)
6. [Lab Setup — Virtualization, Genymotion, Kali](#6-lab-setup)
7. [ADB (Android Debug Bridge) Configuration](#7-adb)
8. [Insecure Bank App দিয়ে Practice Setup](#8-insecure-bank)
9. [Mobile Attack Vectors](#9-attack-vectors)
10. [OWASP Mobile Top 10 (2023)](#10-owasp-mobile-top-10)
11. [NetHunter Install (Network Scanning)](#11-nethunter)
12. [Tor দিয়ে Anonymity](#12-tor)
13. [Device Tracking ও Anti-Theft Security Tools](#13-device-tracking)
14. [MVT — Mobile Verification Toolkit (Compromise Detection)](#14-mvt)
15. [Spyware/Stalkerware — Forensic Detection Perspective](#15-spyware-detection)
16. [Mobile Security Guidelines](#16-security-guidelines)
17. [APK Hacking Intro এবং Pentest Methodology](#17-apk-hacking-intro)
18. [APK Decompiling — সব Methods](#18-decompiling)
19. [Interview Prep — Android Pentest](#19-interview-prep)
20. [Career Roadmap (Salary, Bug Bounty)](#20-career-roadmap)
21. [iOS Bonus Module](#21-ios-bonus)

---

## 1. AOSP আর Open Source Concept

**AOSP** মানে **Android Open Source Project**। Android একটা Linux kernel আর কিছু open-source software-এর উপর তৈরি একটা operating system।

```
Open Source-এর concept টা বুঝতে গেলে:

একজন developer যখন একটা software বানায়, তার দুইটা option থাকে:
1. Source code টা encrypt/encode করে market-এ বিক্রি করা (যেমন .exe ফাইল)
2. Source code টা freely share করে দেওয়া, যাতে community সেটা
   modify করতে পারে, improve করতে পারে

Android দ্বিতীয় approach follow করে — এই কারণেই এটা এত flexible
আর community-driven হয়ে উঠেছে। Source code free, anyone modify
করতে পারে, এবং continuous improvement-এর কারণে আজকের এত
ভালো-functioning OS।
```

---

## 2. APK File কি, কিভাবে বানে

**APK** = **Android Package Kit** (ফাইলের extension `.apk`)

```
APK আসলে একটা COMPRESSED (ZIP) FILE — শুধু rename করে .apk
extension দেওয়া হয়েছে।

APK বানানোর প্রসেস:
1. Developer Java/Kotlin/C++ ইত্যাদি দিয়ে SOURCE CODE লেখে
2. Java Compiler সেই কোডকে JAVA BYTECODE-এ convert করে
3. DEX Compiler সেই bytecode-কে DEX FORMAT-এ convert করে
   (এটা একটা security layer — human readable থাকে না)
4. DVM (Dalvik Virtual Machine) সব combine করে execution-এর
   জন্য তৈরি করে
5. সব ফাইল compress করে একটা ZIP তৈরি হয়, যেটাকে rename করে
   .apk বানানো হয় — সাথে একটা SIGNATURE CERTIFICATE যোগ হয়
   (যাতে প্রমাণ হয় এটা একটা valid, signed app, demo app না)

APK extract করলে যা পাওয়া যায়:
- Source code (compiled form-এ)
- AndroidManifest.xml (পুরো app-এর structure)
- Resources (images, videos, icons — res ফোল্ডারে)
- Meta-data (extra information about the file)
- Libraries (functions perform করার জন্য)
```

### APK Types

```
1. Pre-installed Applications — OS-এর সাথে আগে থেকেই থাকে
   (uninstall করা যায় না, শুধু disable করা যায়)
2. User-installed Applications — Play Store বা third-party থেকে
   user নিজে install করে (Data folder-এ package name অনুযায়ী
   ফোল্ডার তৈরি হয়, যেমন com.android.app)
3. Developer Application — নিজের বানানো বা demo app (signed
   নাও থাকতে পারে)
```

---

## 3. Android Architecture (Layers)

Android-এর architecture কয়েকটা layer-এ ভাগ করা — bug ঢুঁড়তে গেলে কোন layer-এ কোন vulnerability বেশি পাওয়া যায়, সেটা জানার জন্য এই structure বোঝা জরুরি।

```
┌─────────────────────────────────────┐
│  1. APPLICATIONS LAYER               │  ← Home, Camera, Contacts ইত্যাদি
├─────────────────────────────────────┤
│  2. APPLICATION FRAMEWORK             │  ← Activity Manager, Package
│                                       │     Manager, Window Manager,
│                                       │     Notification Manager,
│                                       │     Content Provider, View System
├─────────────────────────────────────┤
│  3. ANDROID RUNTIME (ART)             │  ← Dalvik Virtual Machine (DVM),
│     + CORE LIBRARIES                  │     Core Libraries
├─────────────────────────────────────┤
│  4. NATIVE LIBRARIES                  │  ← Media, Graphics, SQLite,
│                                       │     OpenGL, SSL, FreeType
├─────────────────────────────────────┤
│  5. HARDWARE ABSTRACTION LAYER (HAL)  │  ← Touch sensor, hardware
│                                       │     interface
├─────────────────────────────────────┤
│  6. LINUX KERNEL                      │  ← WiFi, USB, Display, Audio,
│                                       │     Camera, Bluetooth Drivers
└─────────────────────────────────────┘
```

### প্রতিটা layer-এর কাজ:

```
APPLICATIONS LAYER
  তিন ধরনের app থাকে এখানে: Native (pre-installed), Third-party
  (Play Store থেকে), Developer apps (নিজের বানানো)

APPLICATION FRAMEWORK
  Activity Manager   → কোন screen/page কখন open হবে, manage করে
  Package Manager    → installed packages manage করে
  Window Manager      → window-এর display manage করে
  Notification Manager → notification দেখানো
  Content Manager      → data store/retrieve করতে সাহায্য করে
  View System            → UI elements manage করে

ART (Android Runtime)
  Dalvik Virtual Machine (DVM) — source/byte code-কে machine-readable
  করে দেয়, execution handle করে (Java-র JVM-এর মতো একটা concept)

NATIVE LIBRARIES
  Graphics, multimedia, এবং অন্যান্য core function-এর জন্য library —
  video/audio operation এসব দিয়েই possible হয়

HARDWARE ABSTRACTION LAYER (HAL)
  Hardware-এর সাথে interface দেয় — যেমন touch input নেওয়া

LINUX KERNEL
  সব hardware driver এখানে থাকে — WiFi, USB, Display, Audio, Camera,
  Bluetooth ড্রাইভার সব manage করে এই layer
```

---

## 4. AndroidManifest.xml এবং APK Components

**AndroidManifest.xml** একটা app-এর জন্য সবচেয়ে গুরুত্বপূর্ণ ফাইল — এটা একটা website-এর `sitemap.xml`-এর মতো, পুরো app-এর structure এখানে থাকে।

```
AndroidManifest.xml-এ যা থাকে:
- Package-এর unique name (যেমন com.example.app — app call
  করার জন্য, এবং data folder identify করার জন্য ব্যবহার হয়)
- Version information (third-party services/tools-এর version)
- Definitions — কোন Activities, Services, Broadcast Receivers,
  Intents define করা আছে, কোথায় ফাইল আছে
- Permission Definitions — app কি কি permission নিচ্ছে
  (contacts, location, storage ইত্যাদি — pentesting-এর সময় এটা
  খুবই useful information)
- Shared UID, Preferred Installation Location
- UI Information — launcher icon ইত্যাদি
- External Library Packages — third-party library থাকলে তার নাম

পেনিট্রেশন টেস্টিং-এর সময় সবচেয়ে আগে এই ফাইলটাই দেখা হয়, কারণ
এতে পুরো app-এর summary থাকে।
```

### অন্য Important Files

```
colors.xml    → color resources
strings.xml   → text/string data
styles.xml    → styling/formatting
build.gradle  → dependencies, SDK version, build configuration
```

### App-এর ফোল্ডার Structure

```
java/      → Java source code
drawable/  → images, videos, icons (assets)
layout/    → UI structure (visual layout)
res/       → resources (এবং আরও subfolder)
```

### Data Storage (App কোথায় data রাখে)

```
OFFLINE STORAGE:
  1. Shared Preferences — private information রাখার জন্য
  2. Internal Storage — app-এর extra data (normal file manager-এ
     visible, যেমন WhatsApp-এর data)
  3. External Storage — SD card/external partition (কম ব্যবহার হয়)
  4. SQLite — database management, table format-এ data রাখে

ONLINE STORAGE:
  Network Connection — server-এর সাথে কানেক্ট করে data store করে
  (যেমন Firebase)

Pentesting-এর সময় credential বা private info খুঁজতে হলে Shared
Preferences, Internal Storage, বা SQLite database-এর ভিতরে দেখতে হয়।
```

---

## 5. Android App Components

```
ACTIVITY
  যেকোনো page/screen যেটা call হয় — যেমন Login page, Main page।
  Activity Manager এগুলো manage করে (start/stop)।

INTENT
  App-এর internal communication বা message passing। যেমন একটা
  call আসলে notification আসে — এটা Broadcast Receiver detect করে,
  Intent দিয়ে app-কে message পাঠায়।

BROADCAST RECEIVER
  একটা element যেটা কোনো particular event/task-এর জন্য wait করে।
  যেমন call আসা wait করা, এবং event ঘটলে inform করে দেওয়া।

SERVICE
  Background operation, যেমন Bluetooth on/off করা।

CONTENT PROVIDER
  Data retrieving/storing handle করে।
```

---

## 6. Lab Setup — Virtualization, Genymotion, Kali

### Virtualization Concept

```
Virtualization হলো একটা process যেখানে একটা software (VirtualBox,
VMware Workstation) দিয়ে main system-এর hardware resources
(CPU, RAM, Hard disk, Router/Network) -এর একটা SOFTWARE CLONE
তৈরি করা হয়, যেটা অন্য একটা OS-কে allocate করা যায়।

Benefits:
1. SECURITY — virtual environment-এ কোনো attack হলে সেটা main
   machine পর্যন্ত পৌঁছায় না (rare case ছাড়া)
2. MULTIPLE OS — একসাথে Windows-এ Android, Kali, Parrot সব
   চালানো যায়
3. EFFICIENT USAGE — একটা সিস্টেমেই সব কাজ, আলাদা hardware
   লাগে না
4. RESOURCE ALLOCATION — নিজের ইচ্ছামতো RAM, CPU core allocate
   করা যায় প্রতিটা VM-এ
```

### Genymotion দিয়ে Android Install (Windows-এ)

```
1. genymotion.com থেকে "with VirtualBox" version ডাউনলোড করতে হবে
   (এতে VirtualBox + Genymotion দুটোই একসাথে install হয়ে যায়)
2. Installer run করে administrator permission দিয়ে install করা
3. Genymotion open করে (যদি permission popup বারবার আসে, তাহলে
   "Run as Administrator" দিয়ে রাখলে আর আসবে না)
4. Add বাটনে ক্লিক করে Android version select করা (যেমন Android 12)
5. Network setting-এ "NAT" রাখা প্রথমে — যদি error আসে তাহলে
   "Bridge" select করা
6. Install শেষে play বাটনে ক্লিক করলে VM boot হবে

ERROR হলে কি করবেন:
- Genymotion-কে "Run as Administrator" করে চালান
- সর্বশেষ VMware VirtualBox আলাদাভাবে install করুন
- অথবা Android-এর virtual file নিয়ে এসে সরাসরি VirtualBox-এ
  install করুন
```

### Kali Linux Install (Pre-built VM দিয়ে — সহজ পদ্ধতি)

```
1. Kali-র official website-এ যান → "Pre-built Virtual Machines"
   section-এ যান (ISO install না করে এটাই সহজ পদ্ধতি — time বাঁচে,
   future-এ corrupt হলে নতুন machine চালু করা যায়)
2. VirtualBox 64-bit ফাইল ডাউনলোড করুন
3. Default credentials: username = kali, password = kali
4. ফাইলটা একটা compressed (.7z/.zip) ফাইল — extract করতে হবে
5. VirtualBox-এ File → Import করে extract করা .vbox ফাইল select
   করুন
6. Start করলে boot menu আসবে, capture করে username/password
   দিয়ে login করুন

   (Right Ctrl চাপলে cursor display থেকে বের হয়ে আসবে)
```

### Network Configuration (Kali ও Android একই Range-এ রাখা)

```
দুইটা machine-কেই (Android VM আর Kali VM) একই LOCAL AREA
NETWORK range-এ থাকতে হবে, নাহলে communicate করতে পারবে না।

Settings → Network → Adapter 1 (Host-only Adapter) এবং
Adapter 2 (NAT) — দুই VM-এ একই configuration রাখতে হবে।
Advanced-এ গিয়ে "Paravirtualized Network" enable করা যেতে পারে।

ip address check করার কমান্ড (Kali টার্মিনালে):
ifconfig
```

---

## 7. ADB (Android Debug Bridge) Configuration

```
ADB কি?
  একটা service যেটা দিয়ে Android device-এ command run করার
  ক্ষমতা পাওয়া যায় — terminal থেকে device control করা যায়।
```

### Kali-তে ADB Install করা

```bash
# প্রথমে root permission নিতে হবে
sudo su

# /etc/apt/sources.list ফাইল edit করতে হবে যদি repository
# configure না থাকে (first time setup-এ লাগতে পারে)
nano /etc/apt/sources.list
# শেষের লাইনের সামনে যদি # (comment) থাকে, সেটা remove করতে হবে

apt update
apt install adb

# ADB-র available commands দেখা
adb

# কোনো device connected আছে কিনা চেক করা
adb devices

# Genymotion device-এর সাথে connect করা (IP:Port দিয়ে)
adb connect 192.168.66.101:5555

# আবার devices command দিয়ে confirm করা connect হয়েছে কিনা
adb devices

# একটা APK install করা ADB দিয়ে
adb install ./InsecureBankv2.apk
```

---

## 8. Insecure Bank App দিয়ে Practice Setup

**InsecureBankv2** একটা deliberately vulnerable banking app, যেটা legally পেনিট্রেশন টেস্টিং practice করার জন্য বানানো — Dinesh Shetty-র নামে GitHub-এ আছে।

```bash
# GitHub থেকে clone করা
cd Documents
git clone <InsecureBankv2 GitHub URL>
cd InsecureBankv2

# APK file install করা
adb install InsecureBankv2.apk
```

### Server Side Setup (App কিভাবে নিজের সার্ভারের সাথে কথা বলে)

```bash
cd InsecureBankv2/AndroLabServer
chmod +x *           # সব file executable permission দেওয়া

# Python 2.7 প্রয়োজনীয় (পুরো প্রোগ্রাম python2.7-এ লেখা)
python2.7 app.py

# pip install করা না থাকলে:
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
python2.7 get-pip.py

# Requirements install করা
pip install -r requirements.txt

# সার্ভার চালু করা
python2.7 app.py
```

```
এরপর Android device-এ app open করে Settings/Preferences-এ গিয়ে
Kali-র IP address আর port (যেটা সার্ভার দেখাচ্ছে, যেমন 8888)
configure করে দিতে হবে। এরপর login try করলে দেখা যাবে data সার্ভার
পর্যন্ত পৌঁছাচ্ছে কিনা।

এই পুরো setup-টা শেখায় কিভাবে একটা mobile app নিজের backend
সার্ভারের সাথে communicate করে, এবং কোথায় vulnerability খুঁজতে
হয় (auth flow, network communication ইত্যাদি)।
```

---

## 9. Mobile Attack Vectors

```
যত বেশি FUNCTION/FEATURE একটা device-এ থাকবে, ততই সেটা
VULNERABLE হওয়ার chance বেশি — এটা একটা common security rule।

Mobile-এ আমরা যা ব্যবহার করি, প্রতিটাই potential attack vector:

1. NETWORK CONNECTION (WiFi)
   অজানা/untrusted WiFi network ব্যবহার করলে Man-in-the-Middle
   attack-এর ঝুঁকি থাকে

2. INTERNET — PLAY STORE বা THIRD-PARTY WEBSITE
   Play Store-এও malicious app থাকতে পারে, যদিও Google চেষ্টা করে
   filter করতে। Third-party website-এ এই ঝুঁকি আরও বেশি।

3. CORPORATE/PERSONAL VPN
   তুলনামূলকভাবে safer, কারণ private network-এর মধ্যে communication
   হয়

4. TELECOM SERVICES (Calls, SMS)
   Social engineering attack — fake calls, spam SMS এসবের মাধ্যমে
   manipulation

5. BLUETOOTH / OTHER NETWORKING CHANNELS
   এসব দিয়েও device vulnerable হতে পারে
```

### Mobile Platform Attack Vectors (বিস্তারিত)

```
1. MALICIOUS APPS IN STORE — Play Store-এর মাধ্যমেও malicious
   program install হয়ে যেতে পারে
2. MOBILE MALWARE — Pegasus-এর মতো powerful malware, যেটা শুধু
   একটা call দিয়েই পুরো সিস্টেম compromise করতে পারে
3. APP SANDBOXING VULNERABILITIES
4. WEAK DEVICE & APP ENCRYPTION
5. OS & APP UPDATE ISSUES — পুরোনো version-এ unpatched
   vulnerability থেকে যায়
6. JAILBREAKING & ROOTING — admin permission নেওয়ার সময়
   security lock ভেঙে ফেলা হয়, যেটা attacker-ও exploit করতে পারে
7. WEAK DATA SECURITY
8. MOBILE APPLICATION VULNERABILITIES — পুরোনো/malicious app
   install করা
9. PRIVACY ISSUES (Geolocation) — অনেক app দরকার ছাড়াই
   location permission নিয়ে রাখে
10. EXCESSIVE PERMISSIONS
11. WEAK COMMUNICATION SECURITY — SSL/TLS এর মতো proper
    encryption ব্যবহার না করা
12. PHYSICAL ATTACKS — device হাতে পেলে সরাসরি malicious file/
    USB inject করা যায়
```

---

## 10. OWASP Mobile Top 10 (2023)

**OWASP** = Open Web Application Security Project — এরা market continuously monitor করে এবং সবচেয়ে বেশি ক্ষতিকর vulnerability-গুলোর একটা Top 10 list রিলিজ করে, প্রতি কয়েক বছর পর পর। Developer-দের জন্য এটা guideline হিসেবে কাজ করে।

```
M1: IMPROPER CREDENTIAL USAGE
   Credential store/usage pattern সেফ না — Shared Preferences-এ
   plain বা weak-encoded form-এ store হওয়া (offline বা online,
   দুই ক্ষেত্রেই risk থাকে)

M2: INADEQUATE SUPPLY CHAIN SECURITY
   Developer-রা proper secure coding practice follow না করা —
   শুধু functionality-তে focus করা, security-তে না

M3: INSECURE AUTHENTICATION/AUTHORIZATION
   Authentication (user verify করা) আর Authorization (access
   দেওয়া, session ID-র মাধ্যমে) ঠিকমতো manage না করা — যেমন
   biometric/face recognition ব্যবহার না করে শুধু simple login
   page-এ নির্ভর করা, বা session ID-র proper expiry না থাকা

M4: INSUFFICIENT INPUT/OUTPUT VALIDATION
   Input/output communication-এ proper validation না থাকা

M5: INSECURE COMMUNICATION
   Data clear text-এ transfer হওয়া, proper encryption না থাকা

M6: INADEQUATE PRIVACY CONTROLS
   App-কে unnecessary permission দিয়ে দেওয়া, privacy control
   ঠিকমতো check না করা

M7: INSUFFICIENT BINARY PROTECTION
   Binary code সহজেই source code-এ reverse করা যায় — proper
   protection না থাকা

M8: SECURITY MISCONFIGURATION
   Default password, default port, basic settings ঠিকমতো
   configure না করা

M9: INSECURE DATA STORAGE
   Offline/online data সেফভাবে store না হওয়া

M10: INSUFFICIENT CRYPTOGRAPHY
   Weak encryption technology ব্যবহার করা
```

---

## 11. NetHunter Install (Network Scanning)

### Termux Install এবং Setup

```
Termux হলো Android-এর জন্য একটা terminal app, যেটা দিয়ে
Linux-এর মতো command run করা যায়।

⚠️ Play Store থেকে Termux install করবেন না — সেখানে পুরোনো
version থাকতে পারে, repository update না থাকার কারণে issue হয়।
F-Droid বা official website থেকে install করা ভালো।
```

```bash
# প্রথমে package list আপডেট করা
apt update
pkg update     # y লিখে confirm করা সব package update-এর জন্য

# nmap install করা (network scanning tool)
apt install nmap

# নিজের IP জানা
ifconfig

# পুরো network range scan করা (verbose mode সহ)
nmap 192.168.1.0/24 -v

# শুধু host alive কিনা চেক করা (port scan না করে — দ্রুত,
# firewall block হওয়ার চান্স কম)
nmap -sn 192.168.1.0/24

# একটা specific host-এ detailed scan
nmap 192.168.1.1 -v
```

### Kali NetHunter Install (Termux-এর মধ্যে)

```bash
apt install wget

# install script download করা
wget -O install-nethunter-termux https://offs.ec/2MceZdo
chmod +x install-nethunter-termux

# script চালানো
./install-nethunter-termux
# Full / Minimal / Nano option আসবে — "1" দিয়ে Full select করা

# VNC password set করা (NetHunter-এর জন্য)
nh
kex passwd
# password দুইবার দিতে হবে confirm করার জন্য

# VNC server চালু করা
kex &

# যদি বারবার বন্ধ হয়ে যায়, root user দিয়ে আবার চেষ্টা করুন:
nh
sudo su
kex passwd
```

```
VNC দিয়ে connect করার জন্য একটা VNC client app দরকার (যেমন
"bVNC" — NetHunter Store থেকে পাওয়া যায়)।
Connection: localhost, port 5901 (বা 5902 যদি root user দিয়ে
চালানো হয়), যে password set করা হয়েছে সেটা দিয়ে connect করা।
```

---

## 12. Tor দিয়ে Anonymity

```
TOR একটা project, যার মূল উদ্দেশ্য হলো ANONYMITY বজায় রাখা।
Tor-এর প্রোডাক্টগুলো: Tor Browser, Orbot (mobile-এর জন্য),
Tails OS (নিজস্ব operating system)।
```

### কিভাবে কাজ করে

```
নরমাল browsing-এ:
  আপনার device → সরাসরি সার্ভার → response → আপনার device
  (এতে আপনার real IP সার্ভার পর্যন্ত যায়)

Tor circuit ব্যবহার করলে:
  আপনার device → System 1 → System 2 → System 3 → সার্ভার
  সার্ভার → System 3 → System 2 → System 1 → আপনার device

এতে আপনার ও সার্ভারের মাঝে তিনটা intermediate system থাকে,
যার ফলে:
- Browsing স্লো হয় (multiple hop-এর কারণে)
- কিন্তু real IP কখনো কারো কাছে যায় না
- প্রথম system-এ একটা extra security layer (strong firewall)
  থাকে, যাতে সেই system compromise করাও কঠিন হয়
- এই তিনটা IP নির্দিষ্ট সময় পরপর পরিবর্তিত হতে থাকে, বা
  "New Identity" option দিয়ে manually নতুন circuit তৈরি করা যায়
```

```
Play Store থেকে "Tor Browser" install করে Connect বাটনে ক্লিক
করলেই circuit তৈরি হয়ে যায়। Browsing করার সময় circuit details
দেখা যায়।
```

---

## 13. Device Tracking ও Anti-Theft Security Tools

```
আমাদের নিজেদের Google account-এই device track করার অনেক
তথ্য থাকে — এটা জানা থাকা useful, কারণ এটা দুই দিকেই কাজ করে
(নিজের device হারিয়ে গেলে খুঁজতে, এবং forensics-এর সময় বুঝতে
যে কত detailed data একটা device থেকে পাওয়া সম্ভব):

GOOGLE MAPS TIMELINE
  নির্দিষ্ট তারিখের location history, এমনকি কোন speed-এ travel
  করেছিলেন, সেই সময়ে যে ছবি তুলেছিলেন — সব detail দেখায়।
  Find My Device-এর চেয়েও বেশি detailed।

FIND MY DEVICE
  Lock, Erase, Locate — তিনটা option থাকে।

COMPANY-SPECIFIC DASHBOARD (Samsung, Xiaomi ইত্যাদি)
  প্রতিটা company-র নিজস্ব dashboard থাকে — Lock, Remote access,
  Unlock, Data access, Backup এর মতো বেশি features দেয় Find
  My Device-এর তুলনায়।

THIRD-PARTY ANTI-THEFT APPS
  - Hamza Security (paid) — fake turn-off detect করে, screenshot
    নিয়ে পাঠায়
  - Free alternatives Play Store-এ "Anti Theft" সার্চ করে পাওয়া যায়

ANTIVIRUS TOOLS
  AVG-এর মতো recognized antivirus tool ব্যবহার করা ভালো
```

---

## 14. MVT — Mobile Verification Toolkit (Compromise Detection)

```
MVT একটা tool, যেটা দিয়ে চেক করা যায় device আগে থেকে compromise
হয়ে আছে কিনা, কোনো spyware আছে কিনা।
```

```bash
# GitHub থেকে clone করা
sudo su
git clone <MVT GitHub URL>
cd mvt

# Setup চালানো (install argument দিয়ে)
python setup.py install

# Tool আছে কিনা confirm করা
mvt

# Help দেখা (সব command এর তালিকা)
mvt-android --help

# প্রথমে ADB connection check করা
mvt-android check-adb

# এরপর সর্বশেষ Indicators of Compromise (IOC) ডাউনলোড করা
mvt download-iocs

# পুরো device scan করা (IOC check সহ)
mvt-android check-adb
```

```
স্ক্যান শেষে একটা রিপোর্ট তৈরি হয়, যেখানে:
- Suspicious settings/configuration flag করা হয়
- Root হয়ে থাকলে সেটা detect করে (যদি root করা থাকে এমন কোনো
  module পাওয়া যায়)
- কোনো IOC match করলে clearly highlight করে

এই scan latest IOC database দিয়ে করলে আরও বেশি accurate রেজাল্ট
পাওয়া যায় — তাই scan করার আগে সবসময় `mvt download-iocs`
কমান্ড দিয়ে latest indicator নিয়ে নেওয়া ভালো।
```

---

## 15. Spyware/Stalkerware — Forensic Detection Perspective

> এই অংশটা original transcript-এ একটা "Top 5 spy tool" list ছিল। কিন্তু সেটার বদলে আমি এখানে এই tool category-গুলোর **traces/indicators** নিয়ে লিখছি — কারণ আপনার লক্ষ্য digital forensics (পুলিশের কাজে সাহায্য করা), তাই "কোন spyware সবচেয়ে শক্তিশালী" জানার চেয়ে "কিভাবে spyware-এর উপস্থিতি detect করা যায়" — এটাই বেশি কাজের।

```
COMMERCIAL SPYWARE/STALKERWARE-এর CATEGORY:
  Remote Access Trojan (RAT) ধরনের apps যেগুলো নিজেকে "parental
  control" বা "employee monitoring" tool বলে marketing করে, কিন্তু
  আসলে contact, message, call log, location, camera/mic access
  নিতে পারে — এই category-র tool-গুলো forensic investigation-এ
  (বিশেষত domestic violence/stalking case-এ) প্রায়ই পাওয়া যায়।

DETECTION-এর জন্য MVT (উপরে দেখানো হয়েছে) সবচেয়ে নির্ভরযোগ্য
টুল, কারণ এটা একটা maintained IOC database-এর সাথে compare
করে — শুধু app নাম দেখে বিচার না করে।

ম্যানুয়ালি যা চেক করা উচিত:
1. Settings → Apps → "Show System Apps" enable করে hidden/
   disguised app নাম চেক করা (অনেক stalkerware system app-এর
   মতো নাম নেয়, যেমন "System Update" বা "Wi-Fi Service")
2. Accessibility Service-এ কোন app permission নিয়ে রেখেছে,
   সেটা চেক করা — বেশিরভাগ spyware এই permission ব্যবহার করেই
   screen content পড়ে
3. Battery usage-এ অস্বাভাবিক background activity আছে এমন
   app চেক করা
4. Device Admin permission নেওয়া app-গুলোর তালিকা দেখা
   (Settings → Security → Device Admin Apps)
5. Unknown sources থেকে install হওয়া APK-র history চেক করা
6. Network traffic monitor করা (Wireshark/tcpdump দিয়ে) — কোনো
   অজানা সার্ভারে নিয়মিত data পাঠানো হচ্ছে কিনা

যদি কোনো case-এ stalkerware সন্দেহ হয়, এবং এটা একটা personal
safety risk হতে পারে (যেমন domestic abuse situation), তাহলে
device থেকে app uninstall করার আগে সেটা professional digital
forensics support বা আইন প্রয়োগকারী সংস্থার সাথে কথা বলে এগোনো
উচিত — কারণ app remove করলে attacker সাথে সাথে বুঝে যেতে পারে
এবং victim-এর জন্য বিপদ বাড়তে পারে।
```

---

## 16. Mobile Security Guidelines

```
1. STRONG PASSCODE / BIOMETRIC AUTHENTICATION
   Biometric (fingerprint/face) এর চেয়ে strong PIN/pattern বেশি
   reliable, কারণ unconscious অবস্থায় fingerprint ব্যবহার করা
   সম্ভব, কিন্তু PIN কারো মনে রাখা কঠিন এমন একটা রাখলে নিরাপদ থাকে।

2. কখনো DEVICE ROOT/JAILBREAK করবেন না
   Root করলে admin permission পাওয়া যায়, কিন্তু সেই permission
   attacker-ও exploit করতে পারে যদি device কখনো compromise হয়।

3. শুধু OFFICIAL APP STORE থেকে APP DOWNLOAD করুন
   Reputed, well-known developer-এর app-ই trust করা উচিত।

4. DEVICE সবসময় UPDATED রাখুন
   প্রতিটা update হয় নতুন feature/security enhancement, না হয়
   পুরোনো vulnerability-র জন্য security patch নিয়ে আসে।

5. ANTIVIRUS এবং FIREWALL ব্যবহার করুন
   সব রকমের না, কিন্তু একটা genuine antivirus tool ব্যবহার করা ভালো।
```

---

## 17. APK Hacking Intro এবং Pentest Methodology

```
APK Pentesting-এর প্রয়োজন কেন?
  একটা popular app-এ vulnerability থাকলে সেটা যদি black hat
  hacker exploit করে, তাহলে সব user affect হবে। তাই আগে থেকেই
  app-গুলো hack/test করে vulnerability খুঁজে বের করে সেটা ঠিক করে
  নেওয়াই উদ্দেশ্য, যাতে user trust করতে পারে।
```

### Methodology (Step by Step)

```
1. INFORMATION GATHERING
   App, developer, admin, এবং ব্যবহৃত technology সম্পর্কে তথ্য
   সংগ্রহ করা

2. STATIC ANALYSIS
   App decompile করে কোড পড়া, structure বোঝা (app run না করেই)

3. DYNAMIC ANALYSIS
   App চালিয়ে প্রতিটা action monitor করা — real-time behavior বোঝা

4. EXPLOITATION & VULNERABILITY ASSESSMENT
   যেসব loophole পাওয়া গেছে, সেগুলো দিয়ে কী ক্ষতি হতে পারে সেটা
   assess করা — risk level বোঝা

5. REPORTING & DOCUMENTATION
   পুরো process এবং findings-এর একটা complete report তৈরি করে
   developer/client-কে দেওয়া, যাতে তারা vulnerability fix করতে পারে
```

### Core Tools

```
ADB              → Android device-এ command execute করার জন্য
APK Signer        → recompile করা app-এ signature certificate
                    যোগ করার জন্য
APK Tool           → decompile এবং recompile (build) করার জন্য
MobSF              → Mobile Security Framework — automated
                    penetration testing tool, Android/iOS/Windows
                    mobile app scan করতে পারে
```

---

## 18. APK Decompiling — সব Methods

### Method 1: Manual (Step by Step)

```bash
# Step 1: .apk কে .zip-এ rename করে extract করা
# (যেহেতু APK আসলে একটা compressed file)
mv InsecureBankv2.apk InsecureBankv2.zip
unzip InsecureBankv2.zip -d InsecureBankv2_extracted

# Step 2: classes.dex কে .jar-এ convert করা (dex2jar দিয়ে)
apt install dex2jar
d2j-dex2jar.sh classes.dex -o classes-dex2jar.jar

# Step 3: JD-GUI দিয়ে jar file খুলে source code পড়া
apt install jd-gui
jd-gui
# File → Open File → classes-dex2jar.jar select করা
```

### Method 2: Bytecode Viewer (GUI Tool, সহজ)

```bash
apt install bytecode-viewer
bytecode-viewer

# APK file টা সরাসরি drag-and-drop করলেই এটা automatically:
# - decompile করে দেয়
# - সব source code একসাথে দেখায়
# - AndroidManifest.xml সহ decode করা resources দেখায়
```

### Method 3: APK Tool (Command Line)

```bash
apt install apktool

# Decompile করা
apktool d InsecureBankv2.apk -f

# -f flag দিলে already existing folder থাকলেও force overwrite
# করবে

# Decompile হওয়ার পর একটা ফোল্ডার তৈরি হবে, যেখানে সব resource,
# smali code (readable bytecode), AndroidManifest.xml decoded
# অবস্থায় পাওয়া যাবে।
```

```
তিনটা method-এর মধ্যে Bytecode Viewer সবচেয়ে দ্রুত এবং সহজ
(GUI-based, এক ক্লিকেই সব দেখা যায়), কিন্তু Manual method এবং
APK Tool বোঝা থাকলে underlying process টা ভালোভাবে বোঝা যায়,
যেটা পরবর্তীতে troubleshooting বা automation script লেখার সময়
কাজে আসে।
```

### MobSF দিয়ে Automated Scan

```bash
pip install mobsf
mobsf

# Browser-এ http://127.0.0.1:8000 খুলে যাবে
# সেখানে APK file drag-and-drop করলেই:
# - সব Activity, Service, Receiver, Content Provider count করে
# - AndroidManifest.xml decompile করে দেখায়
# - Permission-ভিত্তিক risk highlight করে (কোন permission কেন
#   dangerous সেটাও explain করে)
# - Certificate/signature সম্পর্কিত high/medium/low vulnerability
#   flag করে (যেমন "Application signed with debug certificate")
# - Domain/Malware check করে — যদি কোনো malicious domain-এর
#   সাথে কানেকশন থাকে, সেটাও দেখায়
```

---

## 19. Interview Prep — Android Pentest

### Top 10 Topics যেগুলো ভালোভাবে cover করা উচিত

```
1. Android Architecture — layers, advantages/disadvantages,
   security perspective
2. Android Components — Activity, Intent, Content Provider কিভাবে
   কাজ করে, কোথায় vulnerability হতে পারে
3. APK Structure — কোন folder/file-এ কী থাকে, কোথায় sensitive
   info leak হওয়ার chance বেশি
4. OWASP Mobile Top 10 — সর্বশেষ দুইটা version compare করে
   দেখা, কী নতুন যোগ হয়েছে
5. Common Android Vulnerabilities এবং তাদের Mitigation
6. ADB — কতটা safe/vulnerable বানাতে পারে একটা device-কে, কিভাবে
   কাজ করে
7. Secure Coding Practices — developer-রা কীভাবে কোড লিখলে
   প্রথম থেকেই security ensure হয়
8. Malware Analysis Techniques — manual ও automated দুই
   পদ্ধতিতেই
9. Firewall ও Antivirus — types, configuration
```

---

## 20. Career Roadmap

### Job Profiles & Average Salary (Fresher Level, India)

```
Junior Penetration Tester / Security Analyst       ৪.৫–৬ LPA
Mobile Application Security Analyst                  ৪.৯–৬.৪ LPA
Security Researcher                                    ৫.৩–৬.৮ LPA
App Security Consultant                                  ৫.৭–৭.৩ LPA
Junior Cyber Security Analyst                              ৪.১–৫.৫ LPA
Ethical Hacking                                               ৪–৫ LPA (starting)
Penetration Testing                                            ১০–১২ LPA
Digital Forensics                                                ৮–৯ LPA
Network Security                                                  ৯–১০ LPA
```

### Bug Bounty (Freelance Path)

```
Bug Bounty হলো একটা process যেখানে:
1. কোনো website/app/tool owner আগে থেকে permission দেয়
   (publicly published scope-এর মাধ্যমে)
2. Researcher সেই scope-এর মধ্যে থেকে vulnerability খোঁজে
3. পাওয়া গেলে responsibly report করা হয় (misuse না করে)
4. Owner সেটা verify করে, এবং তার বিনিময়ে monetary reward
   (bounty) দেয়

Bug Bounty Platforms:
- HackerOne
- SecureBug
- Bugcrowd
- HackenProof
- Intigriti
```

### সিখার Resources

```
1. বই (Books) — গভীরভাবে শেখার জন্য ভালো source
2. Structured Courses
3. Free YouTube Content
4. Online/Offline Live Classes (mentor-এর সাথে direct interaction)

Practice Strategy:
READ → IMPLEMENT (সাথে সাথে practical করা) → NOTE বানানো →
যতক্ষণ না সফল হচ্ছে চেষ্টা চালিয়ে যাওয়া ("Try till succeed")
```

---

## 21. iOS Bonus Module (সংক্ষিপ্ত)

```
IPA FILE
  iOS app-এর equivalent হলো .ipa file (IPA = iOS Package Archive),
  App Store-এর জন্য। Objective-C, Swift, C++ এসব দিয়ে বানানো হয়।
```

### iOS Security — Common RAT/Trojan Categories (Forensic Awareness-এর জন্য)

```
এই category-গুলো historically iOS device-এ remote access/spyware
হিসেবে ব্যবহৃত হয়েছে — এগুলো জানা থাকা useful যাতে যদি কোনো
investigation-এ এই ধরনের indicator (suspicious config profile,
unusual network connection ইত্যাদি) পাওয়া যায়, সেটা চেনা যায়:

- Remote Access Trojan ধরনের iOS malware (jailbroken device-এ
  বেশি কার্যকর হয়)
- Surveillance-grade spyware (যেমন zero-click exploit ব্যবহার করা
  commercial surveillance tools — এগুলো সাধারণত nation-state
  level actor ব্যবহার করে, এবং detection-এর জন্য MVT (যেটা উপরে
  Android-এর জন্য দেখানো হয়েছে) একই tool iOS backup-এও কাজ
  করে: `mvt-ios check-backup`)
- Masquerade-style attack — original legitimate app (যেমন banking
  app)-কে replace করে দেওয়া একটা malicious version দিয়ে

iOS DEVICE সুরক্ষিত রাখার Guidelines:
1. শুধু App Store থেকে app download করুন
2. iOS সবসময় updated রাখুন
3. Strong passcode ব্যবহার করুন (biometric-এর চেয়ে বেশি reliable)
4. অপরিচিত link-এ click করবেন না — browser-এ গিয়ে domain সার্চ
   করে visit করুন
5. App permission নিয়মিত review করুন
```

---

## 📝 Note (গুরুত্বপূর্ণ)

```
এই পুরো notes-টা একটা educational course transcript থেকে তৈরি,
যেখানে সব technique শেখানো হয়েছে নিজের lab/VM-এ practice করার
জন্য, অথবা legally authorized পেনিট্রেশন টেস্টিং/digital forensics
কাজে ব্যবহারের জন্য।

InsecureBankv2-এর মতো deliberately-vulnerable app নিজের VM-এ
ব্যবহার করা সম্পূর্ণ লিগ্যাল এবং নিরাপদ practice — কিন্তু এই একই
technique কোনো real, third-party app বা device-এ অনুমতি ছাড়া
প্রয়োগ করা বেআইনি।

Digital forensics কাজে (পুলিশের সাথে কাজ করার লক্ষ্যে) যাওয়ার আগে
আপনার নিজের দেশ/অঞ্চলের আইনি কাঠামো এবং evidence-handling
procedure সম্পর্কে জানা জরুরি, যেটা আপনার অন্য forensics notes-এ
আগে cover করা হয়েছে।
```
