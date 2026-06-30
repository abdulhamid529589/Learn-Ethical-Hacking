# 🔎 OSINT Framework — Complete Notes

### (Bengali-English Mix, Based on WsCube Tech OSINT Course)

> এই notes-টা osintframework.com নিয়ে একটা course transcript থেকে তৈরি। OSINT মানে এমন একটা স্কিল যেখানে publicly available information থেকে useful data বের করে আনা হয় — investigative/forensics work-এ এটা একটা core skill।

---

## 📚 Table of Contents

1. [OSINT কি, OSINT Framework কি](#1-osint-কি)
2. [Framework-এর Terminology বোঝা (T, R, M)](#2-terminology)
3. [Username OSINT](#3-username-osint)
4. [Email OSINT](#4-email-osint)
5. [Data Breach Checking](#5-data-breach)
6. [Domain OSINT](#6-domain-osint)
7. [Automation Tools](#7-automation-tools)
8. [Dark Web / Onion OSINT](#8-dark-web)
9. [Malware Analysis Tools (Framework-এর মধ্যে)](#9-malware-analysis)
10. [Search Engines ও অন্যান্য Categories](#10-search-engines)
11. [কিভাবে এই Framework Effectively Use করবেন](#11-effective-usage)
12. [গুরুত্বপূর্ণ Note — Ethics ও Privacy](#12-note)

---

## 1. OSINT কি, OSINT Framework কি

```
OSINT = Open Source Intelligence

মানে ইন্টারনেটে OPENLY যা কিছু পাবলিকলি available data/knowledge
আছে, তার মধ্যে থেকে কাজের information বের করে আনা।

"OSINT Detective" একটা নির্দিষ্ট job role-ও হয়, যারা মূলত এই ধরনের
analytical data বের করার কাজ করে।

OSINT FRAMEWORK (osintframework.com) হলো:
  "A tool to gather information from free resources"

এটা একটা SINGLE PAGE যেখান থেকে Username, Email Address,
Domain Name, IP, MAC address — এরকম অসংখ্য category-র জন্য
আলাদা আলাদা tool/website-এর একটা curated collection পাওয়া যায়।
এটা নিজে কোনো tool না, বরং tools/websites-এর একটা organized
MAP — যেটা investigation-এর সময় "কোথায় খুঁজব" এই সমস্যার সমাধান
করে দেয়।
```

---

## 2. Framework-এর Terminology বোঝা (T, R, M)

Framework-এর প্রতিটা link-এর পাশে কিছু letter marker থাকে — এগুলোর মানে বোঝা জরুরি, কারণ এটাই বলে দেয় কোন link কিভাবে কাজ করবে:

```
(T)  = TOOL — এটা একটা tool, যেটা ডাউনলোড করে নিজের সিস্টেমে
       install করে locally run করতে হবে।

(R)  = REGISTRATION REQUIRED — সেই site-এ registration/sign-up
       করতে হবে ব্যবহার করার জন্য।
       (Note: কিছু site-এ R মার্ক নেই, কিন্তু তবুও registration
       লাগে — এটা মাথায় রাখা দরকার, marker সবসময় ১০০% accurate
       নাও হতে পারে।)

(M)  = MANUALLY EDIT URL — এটা একটা URL pattern, যেখানে আপনার
       search term নিজে manually URL-এ বসাতে হবে (যেমন
       site.com/search?user=YOUR_USERNAME এই ধরনের pattern)।
```

---

## 3. Username OSINT

Framework-এ Username section-এ দুইটা category থাকে: **Search Engines** এবং **Specific Sites**।

### Search Engine-ভিত্তিক Username Search

```
উদাহরণ: "WhatsMyName" ধরনের সাইট

কিভাবে কাজ করে:
1. Username লিখে দিতে হয় (multiple username একসাথেও দেওয়া যায়,
   প্রতি লাইনে একটা করে, সাধারণত max ১০টা পর্যন্ত)
2. CAPTCHA verify করতে হয়
3. সার্চ শুরু হলে এটা শত শত platform-এ (Behance, Bandlab, Audio
   Jungle, Blogspot, GitHub, Flipboard, Pinterest, TikTok ইত্যাদি)
   সেই username খুঁজে দেখায়
4. যেখানে account পাওয়া যায়, সেই platform-এর সরাসরি link দিয়ে দেয়

ফলাফল কিভাবে interpret করবেন:
- কোনো account-এ activity/post/join date দেখলে বোঝা যায় account
  টা genuine ও active
- কোনো account-এ কোনো activity না থাকলে (যেমন শুধু sign-up করা,
  কোনো post/follower নেই) — বোঝা যায় এটা হয়তো শুধু username
  reserve করার জন্য বানানো, অথবা inactive
- একই username-এর বিভিন্ন platform-এ ভিন্ন ভিন্ন owner থাকতে পারে
  — তাই প্রতিটা result আলাদাভাবে verify করা দরকার, একই ব্যক্তির
  হিসেবে ধরে নেওয়া যাবে না
```

### Specific Site-ভিত্তিক Username Search

```
এখানে platform-wise (GitHub username search, Twitter/X username
search ইত্যাদি) আলাদা আলাদা link থাকে। (M) marked হলে URL-এ নিজে
username বসিয়ে search করতে হয়, প্রতিটা সাইট direct ভাবে check
করতে হয় ফলাফল আছে কিনা।
```

---

## 4. Email OSINT

### Domain থেকে Email বের করা (theHarvester / Infoga স্টাইল টুল)

```bash
# theHarvester বা Infoga — domain দিয়ে email address gathering

infoga -d targetdomain.com
# -d দিয়ে domain দিতে হয়, এটা breach data সহ multiple search
# engine থেকে email খুঁজে আনে (default সব search engine ব্যবহার হয়)
```

### Email Verification ও Lookup Sites

```
এই category-র tool গুলো একটা email address নিয়ে check করে:
- Email টা genuine/valid কিনা
- Mail server বৈধ কিনা
- Email টা কোনো known data breach-এ আছে কিনা
- Reputation (high/low), blacklisted কিনা, malicious activity-র
  সাথে যুক্ত কিনা — এসব বের করা যায়

একটা single domain থেকে multiple email tool ব্যবহার করে আলাদা
আলাদা সংখ্যক email পাওয়া যেতে পারে — একটা tool যদি ১টা email
দেয়, আরেকটা হয়তো ৫টা দেয় — তাই multiple tool ব্যবহার করলে
সব মিলিয়ে অনেক বেশি comprehensive তথ্য পাওয়া যায়।

Email Format Discovery:
  কিছু site আছে যেগুলো একটা company-র COMMON EMAIL FORMAT
  বের করে দেয় (যেমন firstname.lastname@company.com pattern টা
  কিনা) — এটা useful যখন একজন ব্যক্তির নাম জানা আছে কিন্তু email
  জানা নেই, তখন pattern অনুযায়ী সম্ভাব্য email বানানো যায়।
```

---

## 5. Data Breach Checking

```
এই category প্রত্যেকটা ব্যক্তিরই জানা উচিত — শুধু OSINT/investigation
কাজের জন্য না, ব্যক্তিগত digital hygiene-এর জন্যও গুরুত্বপূর্ণ।

কিভাবে কাজ করে:
1. একটা email address দেওয়া হয়
2. Site টা check করে সেই email কোনো known data breach (যেমন
   কোনো hacked database) -এ exposed হয়েছে কিনা
3. যদি "clean"/green result আসে — মানে সেই email কোনো known
   breach-এ পাওয়া যায়নি
4. যদি breach পাওয়া যায় (লাল/red indicator) — তাহলে exactly কোন
   কোন site-এ breach হয়েছে, এবং কি ধরনের data (password,
   phone, address ইত্যাদি) leak হয়েছে — সেটাও দেখানো হয়

এই ধরনের site investigation-এর সময় target-এর exposed credential/
data সম্পর্কে ধারণা দেয়, যেটা আরেকটা attack vector (যেমন password
reuse pattern বোঝা) হিসেবে কাজে লাগতে পারে।
```

---

## 6. Domain OSINT

### WHOIS Records

```
Domain নিয়ে কাজ করার সময় প্রথমেই WHOIS records চেক করা হয় —
এটা domain registration-এর basic ownership/contact info দেয়
(registrant, registration date, name servers ইত্যাদি)।
```

### Subdomain Enumeration

```bash
# Sublist3r — খুবই জনপ্রিয় subdomain enumeration tool

sublist3r -h     # help দেখা
sublist3r -d targetdomain.com    # নির্দিষ্ট domain-এর সব subdomain
                                   # খুঁজে বের করা

# এছাড়া framework-এ আরও কিছু পরিচিত tool থাকে এই category-তে:
# - ReconNG
# - Gobuster
# - Bluto (ব্যবহারকারীরা প্রায়ই "Bluto" বলে)
# - theHarvester
```

```bash
# ReconNG install ও run করার সাধারণ পদ্ধতি (GitHub থেকে)

git clone <recon-ng GitHub URL>
cd recon-ng
# README ফাইলে exact instruction থাকে, সাধারণত:
python3 recon-ng
```

### অন্যান্য Domain-related Categories

```
Certificate Discovery   → SSL certificate transparency log থেকে
                           subdomain/info বের করা
Passive DNS              → DNS history দেখা (current owner না
                           বদলেও আগের রেকর্ড পাওয়া যায়)
Reputation Check          → domain-এর reputation/trust score
URL Expander                → shortened URL-এর আসল destination
                             বের করা

ONLINE TOOL SCANNER:
  একটা domain/website দিলে সেটা scan করে এবং structured ভাবে
  সেই সাইট সম্পর্কে detail বের করে দেয় — এই ধরনের scanner খুবই
  কাজের প্রাথমিক reconnaissance-এর জন্য।
```

---

## 7. Automation Tools

OSINT-এর সবচেয়ে বড় challenge হলো — অনেক জায়গায় গিয়ে গিয়ে আলাদা আলাদাভাবে চেক করতে হয়, যেটা অনেক সময়সাপেক্ষ। এই সমস্যার সমাধানের জন্য automation tool-গুলো ব্যবহার হয়:

```bash
# AutOSINT — একটা popular automation tool (GitHub-এ পাওয়া যায়)

git clone <AutOSINT GitHub URL>
cd autosint

# Missing module install করা
pip install -r requirements.txt

# Help দেখা
python autosint.py -h

# Usage:
python autosint.py -c "Company Name" -d targetdomain.com
# -c = target organization/owner-এর নাম
# -d = target domain name

# এটা automation ভাবে সব categories থেকে data gather করে দেয়,
# manually প্রতিটা site-এ গিয়ে চেক করার দরকার পড়ে না
```

```
RECONDOG — আরেকটা automation tool

git clone <ReconDog GitHub URL>
cd recondog
pip install -r requirements.txt
python3 recondog.py
# Usage option-এ Domain Finder সহ অনেক feature থাকে
```

```
Framework-এই একটা "OSINT Automation" category আলাদা করে থাকে,
যেখানে এই ধরনের সব automation tool-এর collection পাওয়া যায়।
নিজের একটা reusable script-collection বানিয়ে রাখাও common
practice — যেগুলো time-to-time ব্যবহার করা যায়।
```

---

## 8. Dark Web / Onion OSINT

```
⚠️ Dark Web access করা নিজে থেকেই ঝুঁকিপূর্ণ, এবং সেখানে অনেক
illegal content-ও থাকে। শুধুমাত্র legitimate, authorized investigation
context-এ, প্রয়োজনীয় precaution নিয়ে এটা approach করা উচিত।

Framework-এ এই category-তে থাকে:
- Tor search engine-এর link collection (.onion link খোঁজার জন্য)
- General info resources, যেমন Reddit-এর Dark Web/Onion related
  community — এখানে অনেক সময় useful general discussion/Q&A
  পাওয়া যায়, যেগুলো investigation-এর প্রাথমিক ধারণা দিতে সাহায্য করে
- Onion link aggregator site-গুলো
```

---

## 9. Malware Analysis Tools (Framework-এর মধ্যে)

```
Framework-এ একটা dedicated Malware Analysis category আছে, যেখানে:

- File analysis tools (যেমন VirusTotal-এর মতো hash/file scan
  করার tool)
- PDF-specific analysis tools (malicious PDF detect করার জন্য)
- Office file analysis tools (macro-based malware check করার জন্য)
- Automated sandbox tools

— এই সব একসাথে পাওয়া যায়। এই category-গুলো আপনার আগের malware
forensics notes-এর সাথে directly সম্পর্কিত — একই ধরনের static/
dynamic analysis approach এখানেও কাজে লাগে।
```

---

## 10. Search Engines ও অন্যান্য Categories

```
Framework-এ Search Engine category-তে অনেক specialized search
engine-এর list থাকে, যেগুলো বেশিরভাগ মানুষের জানা থাকে না:

- Meta search engine (একসাথে একাধিক search engine-এ search
  করার জন্য)
- Code search engine (সোর্স কোডের মধ্যে নির্দিষ্ট কিছু খোঁজার জন্য)
- FTP search engine
- News search engine
- Fact-checking site (কোনো তথ্য সত্য কিনা যাচাই করার জন্য)

OTHER MAJOR CATEGORIES:
- People Search Engine — নির্দিষ্ট কোনো ব্যক্তি সম্পর্কে তথ্য খোঁজা
- Image/Video/Document Search — যেমন reverse image search
- Currency/Cryptocurrency related OSINT
- Terrorism-related OSINT resources
- Encoder/Decoder tools
- Wordlist Generator
```

### Reverse Image Search (Practical উদাহরণ)

```
একটা search engine-এ (যেমন "Yandex"-এর মতো reverse image
search সাপোর্ট করা সাইট) কোনো image দিয়ে search করলে সেই
image টা ইন্টারনেটে কোথায় কোথায় ব্যবহৃত হয়েছে, সেটার source
কোথায়, এসব তথ্য পাওয়া যায় — এটা person/profile verification-এর
জন্য খুবই কাজের।
```

---

## 11. কিভাবে এই Framework Effectively Use করবেন

```
PRACTICAL APPROACH:

1. একসাথে সব category explore করতে গেলে এটা সহজেই ১ মাস
   সময় নিতে পারে — কারণ framework টা অত্যন্ত বিস্তারিত (extremely
   exhaustive)। তাই একবারে সব শিখার চেষ্টা না করে ধীরে ধীরে
   category-ভিত্তিক explore করাই বাস্তবসম্মত।

2. Search Engines category দিয়ে শুরু করা ভালো — এটা সবচেয়ে
   বেশি immediate, practical value দেয়, এবং খুব দ্রুত শিখে ফেলা যায়।

3. কোনো একটা category নিয়ে কাজ করার সময়:
   - আগে দেখুন এটা (T)/(R)/(M) কোনটা
   - (T) হলে download/install করে নিজের সিস্টেমে চালাতে হবে
   - (R) হলে registration লাগবে — সবসময় personal/real email/
     phone দেওয়া এড়িয়ে চলা ভালো, বিশেষত unknown/untrusted
     third-party site-এ
   - (M) হলে URL টা মনোযোগ দিয়ে দেখে নিজের search term বসিয়ে
     নিতে হবে

4. একটা single tool থেকে যা পাওয়া যায়, সেটাকে final ধরে নেওয়া
   উচিত না — multiple, independent tool/source দিয়ে cross-verify
   করলে অনেক বেশি comprehensive এবং reliable data পাওয়া যায়
   (যেমন একটা email tool ১টা email দিলে, আরেকটা tool হয়তো
   আরও কয়েকটা দেয়)।

5. কিছু tool যেগুলো আগে free ছিল, সেগুলো OSINT community-তে
   popular হওয়ার কারণে এখন paid হয়ে গেছে — এটা একটা সাধারণ
   pattern, তাই বিকল্প খুঁজে রাখা ভালো অভ্যাস।
```

---

## 12. গুরুত্বপূর্ণ Note — Ethics ও Privacy

```
এই notes-এ দেখানো সব technique শুধুমাত্র এই ধরনের পরিস্থিতিতে
ব্যবহার করা উচিত:

1. নিজের সম্পর্কে তথ্য যাচাই করা (personal digital footprint check,
   data breach check ইত্যাদি)
2. Authorized penetration testing/security assessment, যেখানে
   লিখিত অনুমতি (scope document) আছে
3. Legal, authorized digital forensics/investigation কাজ, যেমন
   পুলিশ বা আইন প্রয়োগকারী সংস্থার সাথে সরাসরি জড়িত একটা case

কোনো ব্যক্তির সম্পর্কে অনুমতি ছাড়া personal information জড়ো করা
(stalking, harassment, doxxing-এর উদ্দেশ্যে) বেআইনি এবং
ক্ষতিকর — এমনকি তথ্যগুলো সব publicly available হলেও, সেগুলো
একসাথে জড়ো করে কাউকে target করা একটা গুরুতর privacy violation
হতে পারে।

Dark web access এবং malware analysis tool ব্যবহারের সময়ও
নিজের system সবসময় isolated/sandboxed environment-এ রাখা
উচিত, যেটা আপনার আগের forensics notes-এও বিস্তারিতভাবে cover
করা আছে।
```
