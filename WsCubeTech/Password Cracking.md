# Password Cracking — Course Notes

## https://youtu.be/v5e6MzcI-bQ

> Source: Cybersecurity course video (Password Cracking Fundamentals & Tools)
> Format: Structured notes for revision

---

## 1. Introduction

**Statistic:** 2.2 million passwords are cracked every single day. Most cracked passwords aren't complex ones — they're the ones people _thought_ were safe (e.g., `123456`, `admin` — still among the most commonly used passwords in the world today).

> The real question isn't _"can my password be cracked?"_ — it's _"in how much time?"_

### What This Course Covers

- Thinking like an attacker — step by step, tool by tool
- What happens to a password after it's stored on a server
- Hashing, Salting, and Cracking — from the ground level
- Running real industry-standard tools

**Important reminder:** Everything taught here must only be used on **authorized systems**. In ethical hacking, ethics are non-negotiable.

---

## 2. Password Fundamentals

### What Is a Password?

- A **secret string used to confirm identity** — the simplest and most widely used **authentication mechanism**.
- **Authentication** = confirming whether the user accessing a login page is really who they claim to be.

### Why Passwords Matter

- **First line of defense** — the very first security measure set when creating an account. A weak password means the account can be compromised quickly.
- Passwords protect everything **from email accounts to critical infrastructure**.
- They are often the **only barrier** between an attacker and your data (even though modern systems add layers like Two-Factor Authentication).

---

## 3. How Passwords Are Stored (Plain Text → Protected)

### Store, Not See — The Core Principle

When you sign up on a website and set a password (e.g., "password"), it does **not** get stored in the database as-is (plain text). Instead, it is converted into a **hash**.

- **Storing raw/plain-text passwords is a dangerously bad practice** — rarely seen today. Nearly all databases store passwords as hashes.

### Why Hashing Protects You

If an attacker breaches a database:

- If passwords were stored in **plain text** → immediately compromised.
- If passwords were stored as **hashes** → still relatively protected, because hashes are very difficult (though not always impossible) to crack directly.

> **Caveat:** Weak passwords (e.g., `12345`) are easy targets for attackers **regardless of storage method** — even hashed, a weak password can be cracked quickly.

---

## 4. Hashing — Core Concept

**Hashing** = A password is run through an **algorithm** that produces a **fixed-length string** (the hash).

### Key Properties of Hashing

1. **One-Way Process** — A password can be converted into a hash, but a hash **cannot be directly reversed** back into the original password.
   - _(This raises the question: if it can't be reversed, how do we "crack" it? — Covered in the Password Cracking section below — it involves indirect techniques, not reversal.)_

2. **Deterministic** — The **same input always produces the same output**, every single time, without exception (as long as the same algorithm is used).
   - Example: Hashing "admin" with MD5 will always generate the exact same hash — on any system, any time.
   - Even a **tiny change** (e.g., adding a single dot after "admin") completely changes the resulting hash. This property is also used to **verify data integrity** — if a hash changes, you know the original data was altered.

3. **Fixed Length Output** — Regardless of input length (short or long password), the same algorithm always produces output of the **same fixed length**.
   - Example: MD5 always produces a **32-character** hash, whether the input is 3 characters or 300.

### Common Hash Algorithms

| Algorithm   | Length        | Security                 | Common Use                                    |
| ----------- | ------------- | ------------------------ | --------------------------------------------- |
| **MD5**     | 32 characters | ❌ Not secure today      | Older/legacy web applications — fast to crack |
| **SHA-1**   | —             | ❌ Not considered secure | Legacy systems                                |
| **SHA-256** | —             | ✅ Secure                | Bitcoin, TLS                                  |
| **SHA-512** | —             | ✅ Secure                | Storing Linux passwords                       |
| **bcrypt**  | —             | ✅ Secure                | Modern web applications                       |
| **NTLM**    | —             | ❌ Not very secure       | Windows systems                               |

---

## 5. Generating Hashes — CyberChef Tool

**CyberChef** — nicknamed **"the Swiss Army Knife of Data Operations."**

- Browser-based tool, **no installation required**.
- Can perform: hashing, encoding, decoding, and encryption — all visually (GUI-based, no commands needed).

### How to Use CyberChef

1. Search "CyberChef" on Google and open the official site.
2. Enter your **input** (e.g., "admin") in the input field.
3. Select an **operation** (e.g., Hashing).
4. Choose the desired **algorithm** (e.g., MD5, bcrypt) and drag it into the **Recipe** section.
5. Click **Bake** to generate the output hash.

### Observations from Practice

- Hashing "admin" with MD5 → produces a 32-character hash.
- Adding a small change (e.g., a dot after "admin") → completely different hash (demonstrates the integrity-check property).
- Switching algorithms (e.g., to bcrypt) → different hash, different length, and includes configurable **rounds** (default: 10).
- Same input + same algorithm = same hash, always — reproducible on any system.

---

## 6. What Is Password Cracking?

**Definition:** The process of **recovering original passwords from stored hashes** — not by directly reversing the hash (impossible), but by using indirect **techniques** to match a hash back to a known password.

### Legitimate Uses

- Penetration Testing
- Digital Forensics
- Security Audits
- CTF (Capture The Flag) Challenges

### Ethical Boundaries

- Password cracking can be used for both **legal and illegal** purposes — this course is for **educational purposes only**.
- Ethical hackers operate within a **clearly defined scope** and must have:
  - **Written permission** to test the target system
  - A clear understanding of what's allowed/not allowed within scope
  - **Proper documentation** of every action performed during testing

---

## 7. Identifying Hash Type — First Step Before Cracking

Before cracking a hash, you must identify **which algorithm** produced it (MD5? SHA-1? SHA-256?).

### Tool: Hash-Identifier (Linux/Kali tool)

**Usage:**

```
hash-identifier
[paste the hash]
```

The tool will list possible hash types, ranked by likelihood (e.g., its top guess for a given hash might be MD5).

---

## 8. Types of Password Cracking Attacks

### 1. Dictionary Attack

Uses a list of **common passwords** (a "wordlist") and tries each one, hashing it with the identified algorithm and comparing it to the target hash until a match is found.

**Process:**

1. Take a password from the wordlist (e.g., "1234").
2. Hash it using the same algorithm as the target hash (e.g., MD5).
3. Compare the generated hash to the target hash.
4. If they match → password found. If not → try the next word in the list.

**Common Wordlists:**

- **rockyou.txt** — pre-installed in Kali Linux; contains ~14 million real passwords from a 2009 data breach.
- **SecLists** — a massive community-maintained wordlist collection (GitHub) containing passwords, usernames, leaked databases, WiFi/WPA lists, web shells, and more.

### 2. Brute Force Attack

Tries **every possible character combination** for a given length.

- **Guaranteed to eventually succeed**, but very **slow** and resource-intensive (exhaustive).
- Time required depends heavily on password length and complexity.

### 3. Rule-Based Attack

Builds on the dictionary attack — after trying common passwords, it applies **rules** to modify them and tries again. Examples of rules:

- Capitalization changes (lowercase → uppercase)
- Appending digits (e.g., `admin` → `admin123`)
- Adding "leet speak" substitutions or symbols

### 4. Mask Attack

Used when you know **part of the password's pattern/structure** (instead of trying everything blindly).

- Example: `admin` + 3 digits → `admin123`, `admin000`, etc.
- **Faster and smarter** than brute force because it only tries meaningful, pattern-based combinations.

### 5. Rainbow Table Attack

Uses a **precomputed table of hashes** (not passwords) — passwords are hashed in advance and stored, then compared directly against the target hash.

- Very **fast** since hashes are pre-generated.
- **Defeated by salting** (see Section 10).

### 6. Credential Stuffing

Uses **previously breached username/password pairs** (leaked in past data breaches) and tries them against a target login page — based on the fact that many people reuse credentials across sites.

---

## 9. Building & Using Wordlists

### Pre-Built Wordlists

**rockyou.txt** (built into Kali Linux):

```bash
locate rock.txt          # find the file path
sudo gzip -d /path/to/rockyou.txt.gz    # unzip (may require sudo)
cd wordlists              # navigate to directory
cat rockyou.txt           # view (14 million+ entries)
```

**SecLists** (GitHub repository):

```bash
git clone <SecLists-repo-link>
```

Contains subfolders for: cracked hashes, honeypot captures, default credentials, leaked databases, malware, PHP hashes, WiFi WPA lists, and more. Navigate into `Passwords/` for password-specific lists.

### Custom Wordlists — Crunch Tool

**Crunch** is a Linux tool for generating **custom wordlists**, useful for targeted attacks when you know something about your target (name, address, etc.).

**Basic syntax:**

```bash
crunch <min-length> <max-length> <charset> -o <output-file>
```

**Example — generate a 4-digit PIN list (0-9):**

```bash
crunch 4 4 0123456789 -o pins.txt
```

**Pattern-based generation using `-t` flag and symbols:**

| Symbol | Represents       |
| ------ | ---------------- |
| `@`    | Lowercase letter |
| `,`    | Uppercase letter |
| `%`    | Digit/number     |
| `^`    | Symbol           |

**Example — generate passwords starting with "AB" followed by 2 digits:**

```bash
crunch 4 4 -t AB%% -o output.txt
```

This produces combinations like `AB12`, `AB13`, `AB99`, etc. — the "AB" stays fixed while `%%` cycles through number combinations.

---

## 10. Salting — Strengthening Hashes

**Salting** = adding **random data** to a password **before** it is hashed.

### How It Works

- Original password: `12345`
- After salting: something like `abc12345xyz` (random string added before/after)
- This salted value is then hashed.

### Why It Matters

Even if an attacker cracks the hash, they get back the **salted** value (e.g., `abc12345xyz`), not the clean original password — making it much harder to determine the actual password, since they don't know which part was the salt and which was the real password.

- **Defeats precomputed rainbow table attacks**, since attackers can't precompute hashes for every possible salt combination.

---

## 11. Cracking Tools — Practical Usage

### Online Tool: CrackStation

- A free online hash-cracking tool using a **pre-built database** of common hash-to-password mappings.
- Simply paste the hash → click "Crack Hash" → get the result (also identifies hash type).
- **Limitation:** Only works well for **common passwords** already in its database. Complex/uncommon passwords likely won't be found this way.

### Offline Tool 1: John the Ripper

**Overview:**

- The **oldest and most popular** tool for offline password cracking.
- Beginner-friendly, **auto-detects hash type** (though not always reliably).
- Best for learning; **slower than Hashcat**.

**Basic Workflow:**

```bash
# Save target hash into a file
echo "<hash>" > hash.txt

# Run John the Ripper with a wordlist
john --wordlist=<path-to-wordlist> hash.txt

# If John fails to auto-detect the hash type, specify format manually:
john --format=raw-md5 --wordlist=<path-to-wordlist> hash.txt
```

> **Tip:** If John doesn't correctly auto-identify the hash, explicitly specify the format (as identified earlier using hash-identifier) using `--format`.

### Offline Tool 2: Hashcat

**Overview:**

- The **fastest password cracking tool** available today.
- Supports **300+ hash types** and **all attack modes** (dictionary, brute force, mask, hybrid, combination).

**Key Flags:**

| Flag | Purpose                                                         |
| ---- | --------------------------------------------------------------- |
| `-m` | Hash type/mode (e.g., `0` = MD5, `100` = SHA1, `1400` = SHA256) |
| `-a` | Attack type (e.g., `0` = straight/dictionary attack)            |

**Mask attack character symbols:**

| Symbol | Represents |
| ------ | ---------- |
| `?l`   | Lowercase  |
| `?u`   | Uppercase  |
| `?d`   | Digit      |
| `?s`   | Symbol     |
| `?a`   | All        |

**Basic dictionary attack command:**

```bash
hashcat -m 0 -a 0 hash.txt wordlist.txt
```

- `-m 0` → MD5 hash type
- `-a 0` → straight/dictionary attack

**Get full help:**

```bash
hashcat --help
```

---

## 12. Checking Password Strength

**Tool: password monster (passwordmonster.com)**

- Enter any password to see an **estimated crack time**.
- Example progression observed:
  - `admin` → cracks in ~0.03 seconds
  - Adding a number (`admin3`) → ~1.67 seconds
  - Adding symbols/special characters → jumps to years, then centuries, then millions of years

### How to Strengthen a Password

- Add **numbers**, ideally not just at the end but mixed in.
- Add **special characters/symbols**.
- Avoid common/dictionary words.
- The more "random" and non-obvious the combination, the longer it takes to crack (can go from seconds → millions of years with the right combination).

---

## 13. Defense — How to Protect Passwords

1. **Use strong, unique passwords** — minimum 12+ characters, mixing uppercase, lowercase, numbers, and symbols.
2. **Check password strength** using online tools before setting a password.
3. **Enable Multi-Factor Authentication (MFA)**.
4. **Use strong hashing algorithms** for your own systems/databases — avoid MD5 and similar weak algorithms.
5. **Always salt your hashes** — critical defense against rainbow table attacks.
6. **Implement account lockout & rate limiting** — makes brute-force and dictionary attacks significantly harder by limiting repeated attempts.
7. **Use password managers** — generate and securely store strong, unique passwords/hashes.

---

## Quick-Reference Summary

```
CONCEPTS
- Password = authentication secret; first line of defense
- Hashing = one-way, deterministic, fixed-length output
- Salting = random data added before hashing → defeats rainbow tables

ATTACK TYPES
1. Dictionary Attack     — common password wordlists
2. Brute Force Attack    — every possible combination (slow, guaranteed)
3. Rule-Based Attack     — dictionary + pattern modifications
4. Mask Attack           — known partial pattern (fast, targeted)
5. Rainbow Table Attack  — precomputed hash tables (defeated by salting)
6. Credential Stuffing   — reused breached username/password pairs

TOOLS
- CyberChef        → generate hashes (GUI, browser-based)
- hash-identifier   → identify hash type (Linux)
- CrackStation      → online cracking (pre-built DB, limited to common passwords)
- John the Ripper    → offline cracking (beginner-friendly, auto-detect)
- Hashcat            → offline cracking (fastest, 300+ hash types, all attack modes)
- Crunch             → generate custom wordlists
- rockyou.txt / SecLists → pre-built wordlists
- passwordmonster.com   → check password strength/crack time

DEFENSE
Strong unique passwords + MFA + salting + rate limiting + password managers
```
