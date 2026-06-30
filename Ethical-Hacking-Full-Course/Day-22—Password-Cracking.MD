# Day 22 — Password Cracking
### Learn Ethical Hacking in 30 Days

---

## 🎯 Objective

Understand what hashing is, how it differs from encryption, and learn how to perform **offline password cracking** using multiple tools and techniques including Hashcat, John the Ripper, and online services.

> 📌 **Context:** Previous videos covered **online password attacks** (using Hydra against live login portals). This video focuses on **offline password attacks** — cracking hashes you've already obtained post-exploitation.

---

## 🧱 Phase Context

```
Post-Exploitation Phase
    ├── Maintaining Access        (previous video)
    ├── Password Dumping          (previous video — NT LM hash extraction)
    └── Password Cracking         ← YOU ARE HERE (Day 22)
         ├── Online Attacks   → Hydra (covered earlier)
         └── Offline Attacks  → Hashcat + John the Ripper (this video)
```

---

## 🧠 Concept 1 — What is a Hash?

### Official Definition
> Hashing **converts data of any size into a fixed-length string** called a **hash value**, using a **hash function**. Even slight changes in the input produce drastically different hashes. Hash values are **irreversible** — the original data cannot be retrieved from them.

### How It Works

```
Input (any size)          Hash Function         Output (fixed length)
─────────────────         ─────────────         ─────────────────────
"password"          →     MD5 Algorithm   →     5f4dcc3b5aa765d61d8327deb882cf99
"password1"         →     MD5 Algorithm   →     7c6a180b36896a0a8c02787eeafb0e4c
"Password"          →     MD5 Algorithm   →     dc647eb65e6711e155375218212b3964
```

> 💡 Notice: Even a tiny change ("password" → "Password") produces a completely different hash. This is called the **Avalanche Effect**.

### Key Properties of Hashes

| Property | Explanation |
|----------|------------|
| **Fixed Length** | No matter how long the input, output is always the same length (e.g., MD5 always = 32 characters) |
| **One-Way (Irreversible)** | You CANNOT reverse a hash back to its original input mathematically |
| **Deterministic** | Same input ALWAYS produces same hash — "password" always gives the same MD5 |
| **Avalanche Effect** | Tiny input change = completely different hash output |
| **Collision Resistant** | Two different inputs should not produce the same hash (older algorithms like MD5 are weaker here) |

### Three Main Uses of Hashing

**1. Data Integrity**
When you download software, the company publishes its hash on their website. After downloading, you compute the hash of your downloaded file and compare:
```
Published Hash:   a3f5c9d... (on company website)
Your File Hash:   a3f5c9d... ✅ Match = File is authentic, untampered
Your File Hash:   b7e2a1f... ❌ No match = File was tampered by attacker!
```
> Even if an attacker injected malicious code into the download, the hash will change, alerting you.

**2. Indexing**
Hash functions are used in data structures (hash tables) for fast data lookup.

**3. Security (Password Storage)**
> No legitimate company stores your plain-text password in their database. Instead:
```
You create account with password: "MySecret123"
           ↓
Server hashes it: SHA256("MySecret123") = 8d23a21...
           ↓
Database stores: 8d23a21... (the hash, NOT "MySecret123")
           ↓
When you log in: server hashes your input and COMPARES hashes
```
> Even if a hacker steals the database, they get hashes — not plain passwords.

---

## ⚖️ Concept 2 — Hashing vs Encryption

This is a **very common interview question**. Know this cold.

| Feature | Hashing | Encryption |
|---------|---------|-----------|
| **Process Type** | One-way (irreversible) | Two-way (reversible) |
| **Can it be reversed?** | ❌ No — cannot decrypt | ✅ Yes — can decrypt with key |
| **Key Required?** | No key needed | Yes — public/private key pair |
| **Output Length** | Fixed length always | Variable (depends on input size) |
| **Primary Purpose** | **Integrity** — verify data hasn't changed | **Confidentiality** — keep data secret |
| **Analogy** | A meat grinder — food goes in, can't come back out | A lockbox — lock it, unlock it with the right key |
| **Algorithms** | MD5, SHA1, SHA256, bcrypt | AES, RSA, Blowfish |
| **Use Cases** | Password storage, file integrity checks | HTTPS, encrypted emails, secure file storage |

### Simple Analogy

> **Encryption** = A lockbox with a key. You lock it, you can unlock it.
> **Hashing** = A shredder. Paper goes in, confetti comes out — you can never rebuild the original paper.

---

## 🌈 Concept 3 — Password Cracking Techniques

### What is Password Cracking?
> Password cracking is the process of **attempting to guess or recover a password** to gain unauthorized access to a computer system or online account. Attackers use various techniques, often with automated tools.

### Technique 1: Brute Force Attack
Try **every possible combination** of characters until the correct password is found.
```
Try: a, b, c ... z, aa, ab, ac ... zz, aaa ...
Eventually hits: "cat" → hash matches → PASSWORD FOUND
```
- Guaranteed to work eventually
- **Extremely slow** for long/complex passwords
- A 8-character password with uppercase + lowercase + numbers = 218 trillion combinations

### Technique 2: Dictionary Attack
Use a **pre-made wordlist** (file containing thousands of common passwords) instead of random combinations.
```
Wordlist contains:
password
123456
qwerty
letmein
welcome
dragon
...
```
- Much faster than brute force
- Works well against weak/common passwords
- Famous wordlist: **rockyou.txt** (contains 14 million real passwords from a 2009 data breach)

### Technique 3: Rainbow Table Attack
Use **pre-computed hash tables** — a massive database where passwords and their hashes are already calculated.
```
Rainbow Table (pre-built):
"password"   →   5f4dcc3b5aa765d61d8327deb882cf99
"123456"     →   e10adc3949ba59abbe56e057f20f883e
"qwerty"     →   d8578edf8458ce06fbc5bb76a58c5ca4
...millions more entries...
```
**Attack process:**
```
You steal hash: 5f4dcc3b5aa765d61d8327deb882cf99
         ↓
Look it up in Rainbow Table
         ↓
MATCH FOUND → password = "password"
```
- Extremely fast (lookup, not computation)
- Defeated by **salting** (adding random data to password before hashing)
- Sites like **CrackStation** use this technique

---

## 🛠️ Tools Overview

| Tool | Type | Best For | Requires GPU? |
|------|------|---------|--------------|
| **CrackStation** | Online Website | Quick hash lookups (pre-computed tables) | No |
| **hashes.com** | Online Website | Alternative quick hash lookup | No |
| **Hashcat** | CLI (Kali) | Powerful GPU-accelerated cracking | Recommended |
| **John the Ripper** | CLI (Kali) | CPU-based cracking, diverse file formats | No |

---

## 🌐 Tool 1 — Online Tools (Quick Method)

### CrackStation — crackstation.net
> Uses massive pre-computed lookup tables to crack password hashes.

**How to use:**
1. Copy your hash (e.g., `5f4dcc3b5aa765d61d8327deb882cf99`)
2. Go to `crackstation.net`
3. Paste the hash in the input box
4. Complete CAPTCHA → Click **Crack Hashes**
5. Result: `password` (if it's in their database)

**Practical Demo from video:**
```
MD5 hash of "1234" generated → pasted into CrackStation
Result: "1234" ✅ (common password, already in their table)
```

### hashes.com
- Same concept — alternative to CrackStation
- Paste hash → Submit → Get result
- Works well for MD5, SHA1, NTLM, and more

> 💡 **When to use online tools:** When you need results instantly without setting up local tools. Best for common/weak passwords. Will fail on complex/unique passwords.

---

## ⚙️ Tool 2 — Hashcat (GPU-Accelerated)

### About Hashcat
- Pre-installed on **Kali Linux**
- Uses your **GPU (graphics card)** for massively parallel cracking
- The fastest cracking tool available
- Supports 300+ hash types

### View Help
```bash
hashcat -h
```
Or check hash modes:
```bash
hashcat -h | grep -i "ntlm"      # Find NTLM mode number
hashcat -h | grep -i "sha256"    # Find SHA256 mode number
hashcat -h | grep -i "md5"       # Find MD5 mode number
```

### Attack Modes

| Mode Number | Mode Name | Description |
|-------------|-----------|-------------|
| `0` | Straight / Dictionary | Use a wordlist directly ← **Most common** |
| `1` | Combination | Combine two wordlists |
| `3` | Brute Force / Mask | Try all combinations matching a pattern |
| `6` | Hybrid Wordlist + Mask | Wordlist combined with pattern |
| `7` | Hybrid Mask + Wordlist | Pattern combined with wordlist |

### Common Hash Mode Numbers

| Hash Type | Mode (`-m`) | Common Use |
|-----------|------------|-----------|
| MD5 | `0` | Web applications (older) |
| SHA1 | `100` | Git commits, older web apps |
| SHA256 | `1400` | Modern web applications |
| SHA512 | `1800` | Linux `/etc/shadow` passwords |
| NTLM | `1000` | Windows password hashes |
| bcrypt | `3200` | Modern web apps — slow to crack |
| WPA/WPA2 | `2500` | WiFi passwords |

### Basic Hashcat Command Structure
```bash
hashcat -a <attack_mode> -m <hash_mode> <hash_file> <wordlist>
```

### Practical Commands

**Step 1: Save your hash to a file**
```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
```

**Step 2: Run Hashcat**
```bash
# Dictionary attack with MD5 hash (mode 0)
hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# Dictionary attack with NTLM hash (mode 1000)
hashcat -a 0 -m 1000 hash.txt /usr/share/wordlists/rockyou.txt

# Dictionary attack with bcrypt hash (mode 3200)
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

**With sudo (sometimes required):**
```bash
sudo hashcat -a 0 -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
```

### Reading Hashcat Output
```
hash_value:cracked_password

5f4dcc3b5aa765d61d8327deb882cf99:password   ← format: hash:plaintext

Session..........: hashcat
Status...........: Cracked        ← SUCCESS
Hash.Mode........: 0 (MD5)
Hash.Target......: 5f4dcc3b...
Time.Started.....: [timestamp]
Time.Estimated...: 0 secs
Candidates.#1...: password -> letmein
```

### ⚠️ Important GPU Note
> Hashcat is designed for **GPU-based cracking**. If you don't have a dedicated GPU:
> - You may get OpenCL/driver errors
> - Try adding `--force` flag to override: `hashcat --force -a 0 -m 0 hash.txt rockyou.txt`
> - It will work sometimes, but not reliably
> - **Better alternative without GPU → Use John the Ripper** (CPU-based)

---

## ⚙️ Tool 3 — John the Ripper (CPU-Based)

### About John the Ripper
- Pre-installed on **Kali Linux**
- **CPU-based** — works reliably on all machines, even VMs without GPU
- Great alternative to Hashcat when GPU is unavailable
- Has **built-in password list** — can crack some hashes without providing a wordlist
- Ships with helper utilities to extract hashes from various file formats

### View Help
```bash
john --help
```

### Basic John Command Structure
```bash
john --wordlist=<wordlist> <hash_file>
```

### Practical Examples

**Basic crack with rockyou.txt:**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Show cracked passwords after a session:**
```bash
john --show hash.txt
```

---

## 📁 John the Ripper — Cracking Different File Types

One of John's biggest strengths is its **utility scripts** for extracting hashes from various protected file formats. Here are the three demonstrated in the video:

---

### 📦 Use Case 1: Password-Protected ZIP File

**Scenario:** You found a password-protected ZIP file (e.g., `confidential.zip`). You need to crack its password.

**Step 1: Extract the hash from the ZIP file**
```bash
zip2john confidential.zip > hash.txt
```
> `zip2john` is a utility that reads the ZIP file's encryption and converts it into a John-compatible hash format.

**Step 2: Verify the hash was extracted**
```bash
cat hash.txt
# Output: confidential.zip/secret.txt:$pkzip2$...  (John-format hash)
```

**Step 3: Crack the hash with John**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Step 4: View the cracked password**
```bash
john --show hash.txt
# Output: confidential.zip:password123
```

**Step 5: Use the password to extract the ZIP**
```bash
unzip confidential.zip
# Enter password: password123
# Extracts secret.txt with contents inside
```

---

### 🔑 Use Case 2: Encrypted SSH Private Key (id_rsa)

**Scenario:** You found an SSH private key file (`id_rsa`) but it's encrypted — it requires a passphrase to use. You need to crack that passphrase.

**Background:** SSH login has two methods:
1. Password authentication
2. Key-based authentication (using `id_rsa` private key file)

The private key itself can be encrypted with a passphrase for extra security. You need to crack this passphrase.

**Step 1: Extract the hash from the SSH key**
```bash
ssh2john id_rsa > hash.txt
```
> `ssh2john` converts the SSH key's encryption into a John-compatible hash.

**Step 2: Crack it**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Step 3: Get the passphrase**
```bash
john --show hash.txt
# Output: id_rsa:backs   (passphrase = "backs")
```

**Step 4: Use the passphrase to authenticate**
```bash
ssh -i id_rsa user@target.com
# Enter passphrase: backs
```

---

### 🗄️ Use Case 3: KeePass Database File (.kdbx)

**Scenario:** You compromised a user's PC and found a KeePass database file (`passwords.kdbx`). KeePass is a popular open-source password manager that stores ALL of the user's passwords behind one **master password**.

**Why this matters:**
```
Target uses KeePass
    └── Master Password unlocks the vault
         └── Inside: ALL their passwords
              ├── Bank account
              ├── Email
              ├── Server credentials
              └── VPN access
```
> Crack one password → get access to ALL their passwords. This is the "master key" attack.

**Step 1: Extract the hash from the KeePass database**
```bash
keepass2john passwords.kdbx > hash.txt
```

**Step 2: Crack the master password**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Step 3: Get the master password**
```bash
john --show hash.txt
# Output: passwords.kdbx:MasterPass1   (master password found)
```

**Step 4: Open KeePass with the cracked master password**
→ All stored passwords are now accessible.

---

## 🔍 Identifying Unknown Hash Types

**Problem:** When you dump a database or find hashes in the wild, you often don't know which hashing algorithm was used.

**Solution: `hash-identifier` tool**

### Method 1: hash-identifier (CLI)
```bash
hash-identifier
# Enter hash when prompted
# It will list possible hash types
```

**Example:**
```
Input:  5f4dcc3b5aa765d61d8327deb882cf99
Output: Possible Hashs:
        [+] MD5
        [+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```

### Method 2: Hashcat's Example Hashes
When `hash-identifier` is unclear, look at **Hashcat's example hashes** to visually compare format:
```bash
hashcat -h | grep -i "bcrypt"
# Shows mode number and example hash format for bcrypt
```

Then compare the **beginning pattern** of your unknown hash against known formats:

| Hash Pattern | Type | Hashcat Mode |
|-------------|------|-------------|
| `$2a$` or `$2y$` at start | bcrypt | `3200` |
| `$1$` at start | MD5-crypt | `500` |
| `$6$` at start | SHA512-crypt | `1800` |
| 32 hex characters | MD5 | `0` |
| 40 hex characters | SHA1 | `100` |
| 64 hex characters | SHA256 | `1400` |

> 💡 **Pro tip:** When format is ambiguous, paste the hash into **ChatGPT** and ask "What hash type is this?" — saves time vs manual lookup.

---

## 🎯 Practical: Cracking TryHackMe — "Crack the Hash" Room

The video demonstrates solving the first level of the **TryHackMe "Crack the Hash"** room:

### Hash 1: MD5
```
Hash: 48bb6e862e54f2a795ffc4e541caed4d

Step 1: Identify → hash-identifier → MD5 (mode 0)
Step 2: Save hash to file
Step 3: hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt
Result: easy  ✅
```

### Hash 2: SHA1
```
Hash: CBFDAC6008F9CAB4083784CBD1874F76618D2A97

Step 1: Identify → hash-identifier → SHA1 (mode 100)
Step 2: hashcat -a 0 -m 100 hash.txt /usr/share/wordlists/rockyou.txt
Result: password123  ✅
```

### Hash 3: bcrypt (Advanced — Requires Filtered Wordlist)
```
Hash: $2a$05$... (bcrypt format)

Step 1: Identify → starts with $2a$ → bcrypt → mode 3200
Step 2: Problem: bcrypt is SLOW — running all of rockyou.txt takes hours
Step 3: TryHackMe hint says password is 4 characters
Step 4: Filter rockyou.txt for 4-character words only:

        awk 'length($0)==4' /usr/share/wordlists/rockyou.txt > 4char.txt

Step 5: hashcat -a 0 -m 3200 hash.txt 4char.txt
Result: bleh  ✅ (found in ~5-10 seconds with filtered list)
```

> 🔑 **Key lesson about bcrypt:** bcrypt is intentionally slow — it's designed to resist cracking. A GPU can try billions of MD5 hashes per second but only thousands of bcrypt hashes per second. **Always filter/optimize your wordlist when dealing with bcrypt.**

---

## 🔁 Complete Password Cracking Workflow

```
HAVE A HASH?
     │
     ├─── Know the hash type?
     │         ├── YES → Go to Step 2
     │         └── NO  → Use hash-identifier or check format patterns
     │
     ├─── Step 2: Find mode number
     │         └── hashcat -h | grep -i "<hashtype>"
     │
     ├─── Step 3: Choose tool
     │         ├── Have GPU?    → Use Hashcat
     │         ├── No GPU?      → Use John the Ripper
     │         └── Quick check? → Use CrackStation / hashes.com
     │
     ├─── Step 4: Choose attack type
     │         └── Start with dictionary attack (rockyou.txt)
     │             If fails → try brute force or hybrid
     │
     ├─── Step 5: Optimize if slow
     │         ├── bcrypt → filter wordlist to expected password length
     │         └── Use -T4 timing in hashcat for speed
     │
     └─── Step 6: Get plaintext password → use for access
```

---

## 📋 John the Ripper — File Format Utilities Cheat Sheet

| File Type | Utility to Extract Hash | Command |
|-----------|------------------------|---------|
| ZIP file | zip2john | `zip2john file.zip > hash.txt` |
| RAR file | rar2john | `rar2john file.rar > hash.txt` |
| SSH Private Key | ssh2john | `ssh2john id_rsa > hash.txt` |
| KeePass DB | keepass2john | `keepass2john file.kdbx > hash.txt` |
| PDF file | pdf2john | `pdf2john file.pdf > hash.txt` |
| Office files | office2john | `office2john file.docx > hash.txt` |
| Linux shadow | unshadow | `unshadow /etc/passwd /etc/shadow > hash.txt` |

---

## ⚠️ Important Notes & Tips

**On rockyou.txt:**
- Located at `/usr/share/wordlists/rockyou.txt` in Kali
- May need to be extracted first: `gunzip /usr/share/wordlists/rockyou.txt.gz`
- Contains ~14 million real-world passwords from the 2009 RockYou data breach
- Covers most weak/common passwords

**On bcrypt:**
- Designed to be slow — each attempt takes significant computation
- A 4-character password with all of rockyou.txt can take hours
- Always check hints/context to narrow down password characteristics before cracking

**On Hashcat without GPU:**
- Add `--force` flag to bypass OpenCL warnings
- May still fail — John the Ripper is a more reliable CPU fallback

**On Hash Salting (defense):**
- Modern systems add a random "salt" value to passwords before hashing
- Same password + different salt = completely different hash
- Rainbow table attacks fail against salted hashes
- Example: `bcrypt("password" + "randomsalt123") = $2a$10$...`

---

## ✅ Key Takeaways

1. **Hashing = one-way** — converts any data to a fixed-length string that cannot be reversed
2. **Hashing ≠ Encryption** — encryption is two-way (reversible); hashing is one-way (irreversible)
3. Three uses of hashing: **Data Integrity, Indexing, Security (password storage)**
4. Three cracking techniques: **Brute Force, Dictionary Attack, Rainbow Table**
5. **Online tools** (CrackStation, hashes.com) = fastest for common passwords — no setup needed
6. **Hashcat** = fastest local tool — GPU-powered, 300+ hash types
7. **John the Ripper** = most versatile local tool — CPU-based, handles ZIP/SSH/KeePass/PDF files
8. **Always identify the hash type first** — wrong mode = no result
9. **rockyou.txt** is your first wordlist to try — covers most real-world weak passwords
10. **bcrypt is slow to crack** — filter your wordlist to optimize attack time
11. **KeePass master password** = one crack gives access to ALL stored passwords

---

## 📚 Quick Reference Card

```
┌──────────────────────────────────────────────────────────────┐
│              PASSWORD CRACKING QUICK REFERENCE               │
├──────────────────────────────────────────────────────────────┤
│  IDENTIFY HASH TYPE                                          │
│  hash-identifier                  → GUI prompt              │
│  hashcat -h | grep -i "md5"       → Find mode number        │
│                                                              │
│  HASHCAT (GPU)                                               │
│  hashcat -a 0 -m 0    hash.txt rockyou.txt  → MD5           │
│  hashcat -a 0 -m 100  hash.txt rockyou.txt  → SHA1          │
│  hashcat -a 0 -m 1000 hash.txt rockyou.txt  → NTLM          │
│  hashcat -a 0 -m 3200 hash.txt rockyou.txt  → bcrypt        │
│                                                              │
│  JOHN THE RIPPER (CPU)                                       │
│  john --wordlist=rockyou.txt hash.txt       → crack          │
│  john --show hash.txt                       → view result    │
│  zip2john      file.zip  > hash.txt         → ZIP            │
│  ssh2john      id_rsa    > hash.txt         → SSH key        │
│  keepass2john  file.kdbx > hash.txt         → KeePass        │
│                                                              │
│  ONLINE TOOLS                                                │
│  crackstation.net   →  paste hash → instant result          │
│  hashes.com         →  alternative online cracker           │
│                                                              │
│  ROCKYOU.TXT PATH                                            │
│  /usr/share/wordlists/rockyou.txt                            │
└──────────────────────────────────────────────────────────────┘

HASH MODE CHEAT SHEET
MD5=0  SHA1=100  SHA256=1400  SHA512=1800  NTLM=1000  bcrypt=3200
```

---

*Series: Learn Ethical Hacking in 30 Days | Day 22 of 30*