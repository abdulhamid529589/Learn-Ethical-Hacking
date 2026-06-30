# 📱 Mobile Reverse Engineering — Complete Guide

### Android & iOS: From APK/IPA to Full App Analysis

> **Who is this for?** You want to analyze mobile apps — find hidden APIs, bypass security checks, extract secrets, understand how apps work, and test for vulnerabilities.

---

## 📚 Table of Contents

1. [How Mobile Apps Work](#1-how-mobile-apps-work)
2. [Setting Up Your Mobile RE Lab](#2-setting-up-your-mobile-re-lab)
3. [Android RE — APK Analysis](#3-android-re--apk-analysis)
4. [Android Dynamic Analysis](#4-android-dynamic-analysis)
5. [Frida on Android — Runtime Hooking](#5-frida-on-android)
6. [iOS RE — IPA Analysis](#6-ios-re--ipa-analysis)
7. [Frida on iOS](#7-frida-on-ios)
8. [Traffic Interception (Android & iOS)](#8-traffic-interception)
9. [Bypassing Common Protections](#9-bypassing-common-protections)
10. [Mobile Vulnerabilities](#10-mobile-vulnerabilities)
11. [Tools Reference](#11-tools-reference)
12. [Practice Platforms](#12-practice-platforms)

---

## 1. How Mobile Apps Work

### Android Architecture

```
┌─────────────────────────────────────────────┐
│              Your App (APK)                  │
│  ┌──────────────┐  ┌──────────────────────┐ │
│  │  Java/Kotlin │  │  Native Code (.so)   │ │
│  │  (DEX)       │  │  C/C++ libraries     │ │
│  └──────────────┘  └──────────────────────┘ │
├─────────────────────────────────────────────┤
│          Android Runtime (ART)               │
│  Compiles DEX bytecode to native machine code│
├─────────────────────────────────────────────┤
│          Android Framework                   │
│  Activity, Service, BroadcastReceiver, etc.  │
├─────────────────────────────────────────────┤
│          Linux Kernel                        │
└─────────────────────────────────────────────┘
```

### What is an APK?

An APK is just a ZIP file. Rename it to `.zip` and extract it:

```
app.apk (renamed to app.zip)
├── AndroidManifest.xml   ← Permissions, components, metadata
├── classes.dex           ← Compiled Java/Kotlin code (DEX bytecode)
├── classes2.dex          ← More code (if app is large)
├── lib/
│   ├── arm64-v8a/        ← 64-bit ARM native libraries (.so)
│   ├── armeabi-v7a/      ← 32-bit ARM native libraries
│   └── x86/              ← x86 native libraries (for emulators)
├── res/                  ← UI resources, images, layouts
├── assets/               ← Raw files (databases, configs, sometimes keys!)
├── resources.arsc        ← Compiled resources
└── META-INF/             ← Signature files
```

### What is DEX?

DEX (Dalvik EXecutable) is Android's bytecode format — like Java .class files but optimized for mobile. Tools can convert DEX back to Java.

### iOS Architecture

```
┌─────────────────────────────────────────────┐
│              Your App (IPA)                  │
│  ┌──────────────────────────────────────┐   │
│  │  Compiled Mach-O binary (Swift/ObjC) │   │
│  │  + Frameworks + Resources            │   │
│  └──────────────────────────────────────┘   │
├─────────────────────────────────────────────┤
│              iOS / XNU Kernel                │
└─────────────────────────────────────────────┘
```

### What is an IPA?

Also a ZIP file:

```
app.ipa
└── Payload/
    └── App.app/
        ├── App              ← The compiled binary (Mach-O)
        ├── Info.plist       ← App metadata, permissions
        ├── Frameworks/      ← Bundled libraries
        ├── _CodeSignature/  ← Code signing info
        └── Assets.car       ← Compiled assets
```

---

## 2. Setting Up Your Mobile RE Lab

### Android Lab Setup

**Option A: Real Device (Best)**

```
1. Get an Android phone (cheap Pixel or OnePlus recommended)
2. Enable Developer Options:
   Settings → About Phone → Tap "Build Number" 7 times
3. Enable USB Debugging:
   Settings → Developer Options → USB Debugging ON
4. For full access: root the device
   (Magisk is the most popular rooting tool)
```

**Option B: Android Emulator (Easy)**

```bash
# Install Android Studio
# https://developer.android.com/studio

# Create AVD (Android Virtual Device):
# AVD Manager → Create Virtual Device
# Choose: Pixel 4, Android 11, x86_64
# Important: Choose "Google APIs" NOT "Google Play"
#            (Google Play images are harder to root)

# Start emulator from command line:
emulator -avd Pixel_4_API_30

# Root the emulator (for full access):
adb root       # Works on "Google APIs" images
adb remount    # Make filesystem writable
```

**Install ADB (Android Debug Bridge):**

```bash
# Linux/Mac
sudo apt install adb
# or with Android Studio SDK:
export PATH=$PATH:~/Android/Sdk/platform-tools

# Windows: download platform-tools from Google

# Test connection
adb devices
# Should show: emulator-5554   device
```

**Essential ADB Commands:**

```bash
adb devices              # List connected devices
adb shell                # Open shell on device
adb install app.apk      # Install APK
adb uninstall com.app    # Uninstall by package name
adb pull /data/data/com.app/  ./  # Copy app data to PC
adb push file.txt /sdcard/    # Copy file to device
adb logcat               # View device logs (very useful!)
adb logcat | grep "com.app"   # Filter logs for your app

# Port forwarding (for Burp proxy)
adb reverse tcp:8080 tcp:8080

# Screenshot
adb exec-out screencap -p > screen.png
```

### iOS Lab Setup

**Option A: Jailbroken Device (Required for full RE)**

```
1. Get an iPhone with supported iOS version
2. Jailbreak using:
   - checkra1n (for A5-A11 chips, iOS 12-14)
   - unc0ver (for various versions)
   - Palera1n (for A9-A11, iOS 15-16)
3. Install Cydia / Sileo (package manager)
4. Install: OpenSSH, Frida, cycript, debugserver
```

**Option B: Simulator (Limited)**

```
Xcode → Open Simulator
Works for basic static analysis
Can run Frida on simulator too
No need for jailbreak
```

**Connect to jailbroken device:**

```bash
# Over USB (with usbmuxd)
ssh root@localhost -p 2222

# Over WiFi
ssh root@192.168.1.x
# Default password: alpine (CHANGE THIS!)
```

---

## 3. Android RE — APK Analysis

### Getting the APK

```bash
# Method 1: From device (if app is installed)
adb shell pm list packages          # List all installed packages
adb shell pm path com.example.app   # Get APK path
# Output: package:/data/app/com.example.app/base.apk
adb pull /data/app/com.example.app/base.apk ./

# Method 2: Download from APK sites
# apkpure.com, apkmirror.com (trusted sources)

# Method 3: Extract from Play Store
# Use APKCombo or similar, or pull from device after installing
```

### Static Analysis — Reading the Code

#### Step 1: apktool — Decompile to Smali

Smali is like assembly for Android — readable but low level.

```bash
# Install
sudo apt install apktool
# or: https://github.com/iBotPeaches/Apktool/releases

# Decompile
apktool d app.apk -o app_decoded/

# Result:
app_decoded/
├── AndroidManifest.xml   ← Now human-readable XML!
├── smali/                ← Disassembled DEX code
│   └── com/example/app/
│       ├── MainActivity.smali
│       └── ...
└── res/                  ← Resources

# Recompile after modifications
apktool b app_decoded/ -o app_modified.apk
```

#### Step 2: JADX — Decompile to Java (Better for Reading)

```bash
# Install
sudo apt install jadx
# or: https://github.com/skylot/jadx/releases

# Decompile to Java
jadx app.apk -d output/

# Open GUI (easier)
jadx-gui app.apk
```

JADX converts DEX → Java, which is much more readable than Smali.

#### Step 3: Read AndroidManifest.xml

```xml
<!-- After apktool, this is human-readable -->
<manifest package="com.example.app">

    <!-- Permissions the app requests -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_SMS"/>     <!-- Suspicious! -->
    <uses-permission android:name="android.permission.RECORD_AUDIO"/> <!-- Suspicious! -->

    <application
        android:debuggable="true"    <!-- Can be debugged! Security issue -->
        android:allowBackup="true">  <!-- Backup allowed! Can extract data -->

        <!-- Activities (screens) -->
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <!-- This is the launcher activity (entry point) -->
            </intent-filter>
        </activity>

        <!-- Exported components (accessible from other apps!) -->
        <activity android:name=".AdminActivity"
                  android:exported="true"/>  <!-- Security issue! -->

        <!-- Services (background processes) -->
        <service android:name=".DataExfilService"/>

        <!-- Receivers (respond to system events) -->
        <receiver android:name=".BootReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <!-- Starts on boot! Persistence mechanism -->
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

#### Step 4: Analyzing Java Code in JADX

```java
// JADX output — looks like real Java

public class LoginActivity extends AppCompatActivity {

    // Hardcoded API key — security issue!
    private static final String API_KEY = "sk_live_abc123xyz789";

    public void onLoginButtonClick(View view) {
        String username = this.usernameField.getText().toString();
        String password = this.passwordField.getText().toString();

        // See what checkCredentials does
        if (checkCredentials(username, password)) {
            startActivity(new Intent(this, MainActivity.class));
        }
    }

    private boolean checkCredentials(String username, String password) {
        // Hardcoded credentials! Classic vulnerability
        return username.equals("admin") && password.equals("secret123");

        // Or maybe it does:
        return this.dbHelper.validateUser(username,
            MD5Hash(password));  // Weak MD5 hashing
    }
}
```

**What to look for in Java code:**

```java
// 1. Hardcoded secrets
String apiKey = "abc123";
String password = "hardcoded";
String secret = "my_jwt_secret";

// 2. Crypto implementations
Cipher.getInstance("AES/ECB/PKCS5Padding")  // ECB mode is weak!
MessageDigest.getInstance("MD5")              // MD5 is weak!

// 3. HTTP calls
new URL("http://api.example.com/...").openConnection()  // HTTP, not HTTPS
OkHttpClient client = new OkHttpClient.Builder()
    .hostnameVerifier((hostname, session) -> true)  // SSL disabled!

// 4. SQLite queries
db.rawQuery("SELECT * FROM users WHERE id=" + userId, null)  // SQLi!

// 5. File operations
getExternalFilesDir()   // Writes to SD card (readable by other apps)
openFileOutput("creds.txt", MODE_WORLD_READABLE)  // World readable!

// 6. Root detection
Runtime.getRuntime().exec("su")  // Check for superuser
new File("/system/app/Superuser.apk").exists()

// 7. Emulator detection
Build.FINGERPRINT.contains("generic")
Build.MODEL.contains("Emulator")
```

#### Step 5: Finding Secrets with grep

```bash
# Search decompiled code for secrets
cd output/  # JADX output directory

# API keys
grep -r "api_key\|apikey\|API_KEY" . --include="*.java"

# Hardcoded URLs
grep -r "http://" . --include="*.java"
grep -r "https://" . --include="*.java"

# Passwords
grep -r "password\|passwd\|secret" . --include="*.java" -i

# AWS credentials
grep -r "AKIA[0-9A-Z]{16}" . --include="*.java"

# Private keys
grep -r "BEGIN.*PRIVATE KEY" . -r

# Firebase URLs
grep -r "firebaseio.com\|firebasestorage" . --include="*.java"

# Also check assets and resources
grep -r "secret\|key\|token" app_decoded/assets/
strings app_decoded/lib/arm64-v8a/*.so | grep -i "key\|secret\|pass"
```

#### Step 6: Analyzing Native Libraries

Native `.so` files are compiled C/C++ and need binary RE tools:

```bash
# Check architecture
file lib/arm64-v8a/libnative.so

# Extract strings
strings lib/arm64-v8a/libnative.so | grep -i "key\|secret\|http"

# See exported symbols (function names)
nm -D lib/arm64-v8a/libnative.so

# Disassemble
# Open in Ghidra: File → Import → select .so file
# Ghidra auto-detects ARM architecture

# Or use radare2
r2 lib/arm64-v8a/libnative.so
[0x00000000]> aaa    # Analyze all
[0x00000000]> afl    # List all functions
[0x00000000]> pdf @ sym.Java_com_example_checkLicense  # Disassemble function
```

### Understanding Smali (Android Assembly)

Smali is the assembly language for Android's DEX bytecode. You need to read it when Java decompilation fails.

```smali
# Java equivalent:
# public boolean checkPassword(String input) {
#     return input.equals("secret123");
# }

.method public checkPassword(Ljava/lang/String;)Z
    .locals 2               # 2 local registers (v0, v1)
    .param p1, "input"      # p1 = first parameter (String input)

    # Load the string "secret123" into v0
    const-string v0, "secret123"

    # Call input.equals(v0)
    # p1 = the "input" object (this is the String)
    # v0 = "secret123"
    invoke-virtual {p1, v0}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    # Move result into v1
    move-result v1

    # Return v1 (true or false)
    return v1
.end method
```

**Smali data types:**

```
Z  = boolean
B  = byte
S  = short
I  = int
J  = long
F  = float
D  = double
V  = void
L  = object (e.g., Ljava/lang/String;)
[  = array (e.g., [I = int array)
```

**Smali patch — make checkPassword always return true:**

```smali
.method public checkPassword(Ljava/lang/String;)Z
    .locals 1

    # Load constant 1 (= true) into v0
    const/4 v0, 0x1

    # Return true immediately
    return v0
.end method
```

---

## 4. Android Dynamic Analysis

### ADB Logcat — Real-Time App Logs

Apps often log sensitive information during development (and forget to remove it):

```bash
# See all logs
adb logcat

# Filter by tag or package
adb logcat -s "MyApp"
adb logcat | grep "com.example.app"

# Filter by priority
adb logcat *:E    # Only errors
adb logcat *:D    # Debug and above

# Filter by keyword
adb logcat | grep -i "password\|token\|key\|error"

# Clear log buffer
adb logcat -c

# Save to file
adb logcat > app_logs.txt
```

Apps sometimes log:

- Authentication tokens
- API responses
- Internal state
- Error messages revealing server paths
- Database queries

### Accessing App Files

```bash
# If device is rooted or app is debuggable:
adb shell

# App's private directory
ls /data/data/com.example.app/
# ├── databases/     ← SQLite databases!
# ├── shared_prefs/  ← XML key-value storage (often stores tokens!)
# ├── files/         ← Internal files
# └── cache/         ← Cached data

# Pull entire app data
adb pull /data/data/com.example.app/ ./app_data/

# Read shared preferences (often contains auth tokens!)
cat /data/data/com.example.app/shared_prefs/prefs.xml

# Open SQLite database
adb pull /data/data/com.example.app/databases/app.db ./
sqlite3 app.db
sqlite> .tables
sqlite> SELECT * FROM users;
sqlite> SELECT * FROM sessions;
```

### Starting Activities Directly

You can launch any app component directly with ADB — even unexported ones if you have a rooted device:

```bash
# Launch a specific activity (screen)
adb shell am start -n com.example.app/.AdminActivity

# Launch with extra data
adb shell am start -n com.example.app/.DeepLinkActivity \
    -d "app://example.com/admin"

# Send a broadcast
adb shell am broadcast -a com.example.TRIGGER_ACTION

# Start a service
adb shell am startservice -n com.example.app/.HiddenService
```

### Android Backup Extraction

```bash
# If allowBackup="true" in manifest
adb backup -noapk com.example.app -f backup.ab

# Convert to readable tar
dd if=backup.ab bs=1 skip=24 | python3 -c \
  "import zlib,sys; sys.stdout.buffer.write(zlib.decompress(sys.stdin.buffer.read()))" \
  | tar xvf -

# Now browse app's private files!
```

---

## 5. Frida on Android

Frida lets you inject JavaScript into a running Android app to hook any method.

### Setup

```bash
# On your PC
pip install frida-tools

# On Android device/emulator:
# Download frida-server from https://github.com/frida/frida/releases
# Match version to your frida-tools version!
# Choose: frida-server-XX.X.X-android-arm64.xz (for 64-bit device)

# Push to device
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server

# Start frida-server (as root)
adb shell su -c '/data/local/tmp/frida-server &'

# Verify connection
frida-ps -U              # List processes on device
frida-ps -U | grep app   # Find your target app
```

### Basic Frida Android Script

```javascript
// android_hooks.js

Java.perform(function () {
  console.log('[*] Frida script loaded!')

  // ============================================
  // Hook any Java method
  // ============================================
  var LoginActivity = Java.use('com.example.app.LoginActivity')

  LoginActivity.checkCredentials.implementation = function (username, password) {
    console.log('[*] checkCredentials called')
    console.log('    Username: ' + username)
    console.log('    Password: ' + password)

    // Option 1: Call original and log result
    var result = this.checkCredentials(username, password)
    console.log('    Original result: ' + result)
    return result

    // Option 2: Force success (bypass login)
    // return true;
  }

  // ============================================
  // Hook overloaded methods
  // ============================================
  var String = Java.use('java.lang.String')

  // If method has multiple overloads, specify parameter types
  String.equals.overload('java.lang.Object').implementation = function (other) {
    var result = this.equals(other)
    if (this.toString().toLowerCase().indexOf('password') >= 0) {
      console.log('[*] Password comparison: ' + this + ' == ' + other)
    }
    return result
  }

  // ============================================
  // Hook constructor
  // ============================================
  var User = Java.use('com.example.app.User')

  User.$init.implementation = function (id, username, role) {
    console.log('[*] User created: id=' + id + ' username=' + username + ' role=' + role)
    this.$init(id, username, role)
  }

  // ============================================
  // Read/modify object fields
  // ============================================
  var Config = Java.use('com.example.app.Config')

  // Hook a method to read a private field
  Config.getInstance.implementation = function () {
    var config = this.getInstance()
    console.log('[*] Config.apiKey = ' + config.apiKey.value)
    // Modify it:
    config.apiKey.value = 'our-custom-key'
    return config
  }

  // ============================================
  // Enumerate all instances of a class
  // ============================================
  Java.choose('com.example.app.User', {
    onMatch: function (instance) {
      console.log('[*] Found User instance:')
      console.log('    id: ' + instance.id.value)
      console.log('    username: ' + instance.username.value)
      console.log('    role: ' + instance.role.value)
    },
    onComplete: function () {
      console.log('[*] User search done')
    },
  })
})
```

**Run the script:**

```bash
# Attach to running app
frida -U -l android_hooks.js com.example.app

# Launch and inject immediately
frida -U -l android_hooks.js -f com.example.app --no-pause
```

### SSL Pinning Bypass

Apps use SSL pinning to prevent traffic interception. Bypass it:

```javascript
// ssl_bypass.js — Universal SSL pinning bypass

Java.perform(function () {
  // ======= Method 1: TrustManager bypass =======
  var TrustManager = Java.use('com.android.org.conscrypt.TrustManagerImpl')
  if (TrustManager) {
    TrustManager.checkTrustedRecursive.implementation = function () {
      console.log('[*] SSL check bypassed (TrustManagerImpl)')
      return Java.use('java.util.ArrayList').$new()
    }
  }

  // ======= Method 2: OkHttp3 CertificatePinner =======
  try {
    var CertificatePinner = Java.use('okhttp3.CertificatePinner')
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation =
      function (hostname, peerCertificates) {
        console.log('[*] OkHttp SSL pinning bypassed for: ' + hostname)
        return
      }
  } catch (e) {}

  // ======= Method 3: Network Security Config bypass =======
  try {
    var NetworkSecurityTrustManager = Java.use(
      'android.security.net.config.NetworkSecurityTrustManager',
    )
    NetworkSecurityTrustManager.checkPins.implementation = function (chain) {
      console.log('[*] NetworkSecurityConfig pin check bypassed')
      return
    }
  } catch (e) {}

  // ======= Method 4: WebView SSL bypass =======
  try {
    var WebViewClient = Java.use('android.webkit.WebViewClient')
    WebViewClient.onReceivedSslError.implementation = function (webView, handler, error) {
      console.log('[*] WebView SSL error bypassed')
      handler.proceed()
    }
  } catch (e) {}
})
```

### Root Detection Bypass

```javascript
// root_bypass.js

Java.perform(function () {
  // ======= Method 1: RootBeer library =======
  try {
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer')
    RootBeer.isRooted.implementation = function () {
      console.log('[*] RootBeer.isRooted() bypassed')
      return false
    }
  } catch (e) {}

  // ======= Method 2: SafetyNet / Play Integrity bypass =======
  // More complex — usually need Magisk modules

  // ======= Method 3: Custom root checks =======
  // Find by searching code for "su", "/system/xbin/su" etc.
  var Runtime = Java.use('java.lang.Runtime')
  Runtime.exec.overload('java.lang.String').implementation = function (cmd) {
    if (cmd.indexOf('su') >= 0) {
      console.log('[*] Root check via exec blocked: ' + cmd)
      throw Java.use('java.io.IOException').$new('Not found')
    }
    return this.exec(cmd)
  }

  // ======= Method 4: File existence checks =======
  var File = Java.use('java.io.File')
  File.exists.implementation = function () {
    var path = this.getAbsolutePath()
    var rootPaths = [
      '/system/app/Superuser.apk',
      '/sbin/su',
      '/system/bin/su',
      '/system/xbin/su',
      '/data/local/xbin/su',
      '/data/local/bin/su',
    ]
    if (rootPaths.indexOf(path) >= 0) {
      console.log('[*] Root file check blocked: ' + path)
      return false
    }
    return this.exists()
  }
})
```

### Dumping Encryption Keys at Runtime

```javascript
// dump_crypto.js — Intercept crypto operations

Java.perform(function () {
  // ======= Intercept SecretKeySpec (AES keys) =======
  var SecretKeySpec = Java.use('javax.crypto.spec.SecretKeySpec')
  SecretKeySpec.$init.overload('[B', 'java.lang.String').implementation = function (
    keyBytes,
    algorithm,
  ) {
    console.log('[*] SecretKeySpec created')
    console.log('    Algorithm: ' + algorithm)
    console.log('    Key (hex): ' + bytesToHex(keyBytes))
    return this.$init(keyBytes, algorithm)
  }

  // ======= Intercept Cipher operations =======
  var Cipher = Java.use('javax.crypto.Cipher')

  Cipher.doFinal.overload('[B').implementation = function (input) {
    console.log('[*] Cipher.doFinal called')
    console.log('    Input:  ' + bytesToHex(input))
    var result = this.doFinal(input)
    console.log('    Output: ' + bytesToHex(result))
    return result
  }

  // ======= Intercept MessageDigest (MD5, SHA) =======
  var MessageDigest = Java.use('java.security.MessageDigest')
  MessageDigest.digest.overload('[B').implementation = function (input) {
    var result = this.digest(input)
    console.log('[*] MessageDigest.digest')
    console.log('    Algorithm: ' + this.getAlgorithm())
    console.log('    Input:     ' + bytesToHex(input))
    console.log('    Hash:      ' + bytesToHex(result))
    return result
  }

  // Helper: bytes to hex string
  function bytesToHex(bytes) {
    if (!bytes) return 'null'
    var hex = ''
    for (var i = 0; i < bytes.length; i++) {
      hex += ('0' + (bytes[i] & 0xff).toString(16)).slice(-2)
    }
    return hex
  }
})
```

### Patching APKs (Static Modification)

When you can't use Frida (e.g., app detects it), patch the APK directly:

```bash
# Step 1: Decompile
apktool d app.apk -o app_decoded/

# Step 2: Find the method to patch
# In app_decoded/smali/com/example/app/LicenseCheck.smali
# Find the checkLicense method

# Step 3: Edit the Smali
# BEFORE:
# invoke-virtual {v0, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
# move-result v2
# if-eqz v2, :cond_fail

# AFTER (make it always succeed):
# const/4 v2, 0x1
# if-eqz v2, :cond_fail   (never taken since v2=1)

# Step 4: Recompile
apktool b app_decoded/ -o app_patched.apk

# Step 5: Sign the APK (required!)
# Generate a keystore (one time):
keytool -genkey -v -keystore my.keystore -alias mykey \
        -keyalg RSA -keysize 2048 -validity 10000

# Sign:
jarsigner -verbose -keystore my.keystore app_patched.apk mykey

# Verify:
jarsigner -verify app_patched.apk

# Install:
adb install app_patched.apk
```

---

## 6. iOS RE — IPA Analysis

### Getting the IPA

```bash
# From jailbroken device (decrypt in memory):
# Install frida-ios-dump:
pip3 install frida-ios-dump

# Dump decrypted IPA
python3 dump.py -H 192.168.1.x -p 22 -u root -P alpine \
                com.example.app -o app_decrypted.ipa

# Or use Clutch (on jailbroken device):
Clutch -i                              # List apps
Clutch -d com.example.app             # Dump to IPA
```

### Static Analysis

```bash
# Extract IPA
unzip app.ipa -d app_extracted/
cd app_extracted/Payload/App.app/

# Check if encrypted
otool -l App | grep -A 4 LC_ENCRYPTION_INFO
# cryptid 1 = encrypted, cryptid 0 = decrypted (analysis can proceed)

# Architecture info
file App
# App: Mach-O 64-bit arm64 executable

# Linked libraries
otool -L App

# Load commands
otool -l App

# Symbols (function names)
nm -a App | grep -v " U "   # Defined symbols (not imported)
nm -a App | grep " U "      # Undefined (imported from frameworks)
```

### Class Dump (Objective-C)

```bash
# Install class-dump
brew install class-dump

# Dump all Objective-C class interfaces
class-dump App > classes.txt

# Result looks like real .h header files:
# @interface LoginViewController : UIViewController
# - (void)loginWithUsername:(NSString *)username password:(NSString *)password;
# - (BOOL)validateToken:(NSString *)token;
# @end
```

**Analyzing the dump:**

```bash
# Find interesting methods
grep -i "password\|login\|auth\|token\|pin\|secret" classes.txt

# Find admin or debug methods
grep -i "admin\|debug\|internal\|backdoor" classes.txt

# Find network methods
grep -i "request\|http\|url\|fetch" classes.txt
```

### iOS Binary in Ghidra

```
1. Open Ghidra → New Project → Import File → select App binary
2. Ghidra auto-detects: Mach-O ARM 64-bit (AArch64)
3. Run auto-analysis
4. Navigate to interesting functions found in class-dump
5. Decompiler shows pseudo-C with Objective-C runtime calls
```

**Understanding Objective-C in assembly:**

```
; Objective-C method calls use objc_msgSend
; [object method:arg] compiles to:
; objc_msgSend(object, @selector(method:), arg)

; In Ghidra/IDA you'll see:
bl  objc_msgSend       ; call objc_msgSend
; With arguments:
; x0 = object (self)
; x1 = selector (method name)
; x2+ = arguments
```

### Info.plist Analysis

```bash
# Extract Info.plist
plutil -convert xml1 Info.plist -o Info_readable.plist
cat Info_readable.plist

# Key things to look for:
grep -i "key\|secret\|url\|api\|token" Info_readable.plist

# Example entries that reveal info:
# <key>APIBaseURL</key>
# <string>https://api.example.com/v2</string>

# <key>AWSAccessKey</key>
# <string>AKIAIOSFODNN7EXAMPLE</string>  ← Secret exposed!

# <key>NSAppTransportSecurity</key>
# <dict><key>NSAllowsArbitraryLoads</key><true/></dict>
# ← HTTP allowed! No SSL enforcement
```

---

## 7. Frida on iOS

### Setup

```bash
# On jailbroken device:
# Install Frida via Cydia/Sileo:
# Add source: https://build.frida.re
# Install "Frida" package

# On your PC:
pip install frida-tools

# Verify:
frida-ps -U    # List iOS processes
```

### iOS Frida Hooks

```javascript
// ios_hooks.js

// ============================================
// Hook Objective-C methods
// ============================================
if (ObjC.available) {
  // Hook a class method
  var LoginVC = ObjC.classes.LoginViewController

  // Hook instance method
  Interceptor.attach(LoginVC['- loginWithUsername:password:'].implementation, {
    onEnter: function (args) {
      // args[0] = self, args[1] = selector
      // args[2] = first param, args[3] = second param
      var username = ObjC.Object(args[2]).toString()
      var password = ObjC.Object(args[3]).toString()
      console.log('[*] Login attempt:')
      console.log('    Username: ' + username)
      console.log('    Password: ' + password)
    },
    onLeave: function (retval) {
      console.log('[*] Login returned: ' + retval)
      // Force success:
      // retval.replace(ptr(1));
    },
  })

  // Hook class method
  var MyClass = ObjC.classes.LicenseManager
  Interceptor.attach(MyClass['+ isLicenseValid'].implementation, {
    onLeave: function (retval) {
      console.log('[*] isLicenseValid bypassed')
      retval.replace(ptr(1)) // Return YES
    },
  })

  // ============================================
  // Enumerate methods of a class
  // ============================================
  var methods = ObjC.classes.SomeClass.$ownMethods
  methods.forEach(function (method) {
    console.log(method)
  })

  // ============================================
  // Hook Swift methods (harder — name mangling)
  // ============================================
  // Find mangled name:
  // nm App | grep -i "checkLicense"
  var swiftFunc = Module.findExportByName(null, '_$s7MyApp14LicenseCheckerC8isValidSbvg')

  if (swiftFunc) {
    Interceptor.attach(swiftFunc, {
      onLeave: function (retval) {
        retval.replace(ptr(1)) // Return true
      },
    })
  }
}
```

### iOS SSL Pinning Bypass

```javascript
// ios_ssl_bypass.js

if (ObjC.available) {
  // ======= NSURLSession bypass =======
  var NSURLSession = ObjC.classes.NSURLSession

  // Hook URLSession:didReceiveChallenge:completionHandler:
  try {
    var delegate = ObjC.classes.NSURLSessionDelegate
    // Hook the authentication challenge handler
  } catch (e) {}

  // ======= AFNetworking bypass =======
  try {
    var AFSecurityPolicy = ObjC.classes.AFSecurityPolicy
    Interceptor.attach(AFSecurityPolicy['- evaluateServerTrust:forDomain:'].implementation, {
      onLeave: function (retval) {
        retval.replace(ptr(1))
        console.log('[*] AFNetworking SSL bypass')
      },
    })
  } catch (e) {}

  // ======= Generic SecTrustEvaluate bypass =======
  // This is the low-level iOS SSL function
  var SecTrustEvaluate = Module.findExportByName('Security', 'SecTrustEvaluate')

  Interceptor.replace(
    SecTrustEvaluate,
    new NativeCallback(
      function (trust, result) {
        console.log('[*] SecTrustEvaluate bypassed')
        // Write kSecTrustResultProceed to result
        Memory.writeU32(result, 1)
        return 0 // errSecSuccess
      },
      'int',
      ['pointer', 'pointer'],
    ),
  )
}
```

---

## 8. Traffic Interception

### Setting Up Burp for Mobile (Both Android & iOS)

```
1. Start Burp Suite → Proxy → Options
   Set to listen on: All interfaces (0.0.0.0), port 8080

2. Configure mobile device WiFi proxy:
   Same WiFi network as your PC
   Set proxy: [Your PC's IP]:8080

3. Install Burp CA certificate:
   Android: Browse to http://burp → download cert
            Settings → Security → Install certificate
   iOS:     Browse to http://burp → download cert
            Settings → General → VPN & Device Management → Install
            Settings → General → About → Certificate Trust Settings → Enable

4. Now Burp intercepts ALL HTTPS traffic!
   (Unless app uses certificate pinning → use Frida bypass above)
```

### Android Certificate Installation

```bash
# For Android 7+ (stricter cert handling)
# Method 1: Use Magisk TrustUserCerts module
# Install in Magisk Manager → reboot

# Method 2: Install as system cert (rooted)
# Convert Burp cert to right format:
openssl x509 -inform DER -in burp.der -out burp.pem
openssl x509 -inform PEM -subject_hash_old -in burp.pem | head -1
# Outputs hash, e.g.: 9a5ba575
mv burp.pem 9a5ba575.0

# Push to system certs (rooted)
adb root
adb remount
adb push 9a5ba575.0 /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/9a5ba575.0
adb reboot
```

### mitmproxy for Mobile

```bash
# Install
pip install mitmproxy

# Start with a script
mitmdump -p 8080 -s mobile_intercept.py

# Script:
# mobile_intercept.py
from mitmproxy import http
import json

def request(flow: http.HTTPFlow):
    if 'api.example.com' in flow.request.pretty_host:
        print(f"\n[REQUEST] {flow.request.method} {flow.request.url}")
        if flow.request.content:
            print(f"  Body: {flow.request.content[:500]}")

def response(flow: http.HTTPFlow):
    if 'api.example.com' in flow.request.pretty_host:
        print(f"[RESPONSE] {flow.response.status_code}")
        # Modify response:
        if b'"premium":false' in flow.response.content:
            flow.response.content = flow.response.content.replace(
                b'"premium":false',
                b'"premium":true'
            )
            print("  [*] Premium status upgraded!")
```

---

## 9. Bypassing Common Protections

### Anti-Tampering (App Integrity Checks)

Apps check if their code has been modified:

```javascript
// Frida bypass for common integrity checks

Java.perform(function () {
  // ======= Signature check bypass =======
  // Apps verify their own APK signature
  var PackageManager = Java.use('android.app.ApplicationPackageManager')

  PackageManager.getPackageInfo.overload('java.lang.String', 'int').implementation = function (
    packageName,
    flags,
  ) {
    var info = this.getPackageInfo(packageName, flags)

    // If checking signatures, return fake valid signature
    if (flags & 0x40) {
      // GET_SIGNATURES flag
      console.log('[*] Signature check intercepted for: ' + packageName)
      // Return info unchanged (app may accept its own original sig)
    }
    return info
  }

  // ======= CRC check bypass =======
  // Some apps calculate CRC of their own DEX
  var ZipFile = Java.use('java.util.zip.ZipFile')
  ZipFile.$init.overload('java.lang.String').implementation = function (path) {
    console.log('[*] ZipFile opened: ' + path)
    return this.$init(path)
  }
})
```

### Emulator Detection Bypass

```javascript
Java.perform(function () {
  // Bypass Build property checks
  var Build = Java.use('android.os.Build')

  // Make emulator look like a real device
  Build.FINGERPRINT.value = 'google/walleye/walleye:8.1.0/OPM1.171019.011/4448085:user/release-keys'
  Build.MODEL.value = 'Pixel 2'
  Build.MANUFACTURER.value = 'Google'
  Build.BRAND.value = 'google'
  Build.DEVICE.value = 'walleye'
  Build.PRODUCT.value = 'walleye'
  Build.HARDWARE.value = 'walleye'

  // Bypass IMEI/phone number checks
  var TelephonyManager = Java.use('android.telephony.TelephonyManager')
  TelephonyManager.getDeviceId.overload().implementation = function () {
    return '867686021378952' // Fake real IMEI
  }
  TelephonyManager.getLine1Number.overload().implementation = function () {
    return '+15551234567'
  }
})
```

### Biometric / Face ID Bypass

```javascript
Java.perform(function () {
  // Bypass fingerprint/biometric authentication

  var BiometricPrompt = Java.use('androidx.biometric.BiometricPrompt')

  // Find and hook the authentication callback
  // The callback has onAuthenticationSucceeded method
  var AuthCallback = Java.use('androidx.biometric.BiometricPrompt$AuthenticationCallback')

  AuthCallback.onAuthenticationSucceeded.implementation = function (result) {
    console.log('[*] Biometric auth succeeded (hooked)')
    this.onAuthenticationSucceeded(result)
  }

  // Simulate successful authentication
  // More complex — need to trigger the callback directly
})
```

---

## 10. Mobile Vulnerabilities

### Android Vulnerabilities

**1. Exported Components (Activity/Service/Receiver)**

```bash
# Check for exported components
grep -r "android:exported=\"true\"" AndroidManifest.xml

# Launch exported admin activity directly
adb shell am start -n com.app/.AdminPanelActivity

# Send data to exported content provider
adb shell content query --uri content://com.app.provider/users
adb shell content insert --uri content://com.app.provider/users \
    --bind name:s:hacker --bind role:s:admin
```

**2. Deep Link Hijacking**

```bash
# Check for deep links in manifest
grep -r "intent-filter\|data\|scheme" AndroidManifest.xml

# Test deep link
adb shell am start -a android.intent.action.VIEW \
    -d "app://example.com/reset?token=abc123"
```

**3. Insecure Data Storage**

```bash
# Check all storage locations
adb shell ls /data/data/com.example.app/

# Common issues:
# - Passwords stored in SharedPreferences
# - Tokens in SQLite without encryption
# - Sensitive files with world-readable permissions
# - Private keys in /sdcard/

# Check file permissions
adb shell ls -la /data/data/com.example.app/shared_prefs/
```

**4. Cleartext Traffic**

```bash
# Check network security config
cat res/xml/network_security_config.xml

# Or check manifest for:
android:usesCleartextTraffic="true"

# Capture unencrypted HTTP with tcpdump on device:
adb shell tcpdump -i any -s 0 -w /sdcard/capture.pcap
adb pull /sdcard/capture.pcap ./
# Open in Wireshark
```

### iOS Vulnerabilities

**1. Insecure Keychain Usage**

```javascript
// Frida: dump keychain entries
// Run on jailbroken device

var SecurityFramework = Module.findExportByName('Security', 'SecItemCopyMatching')
Interceptor.attach(SecurityFramework, {
  onLeave: function (retval) {
    // retval = 0 means success
    if (retval.toInt32() == 0) {
      console.log('[*] Keychain item accessed successfully')
    }
  },
})
```

**2. Insecure File Storage**

```bash
# On jailbroken device, check:
ls ~/Library/Preferences/         # Plist files
ls ~/Documents/                    # App documents
ls ~/Library/Application\ Support/ # Support files

# Copy from device
scp -P 22 root@192.168.1.x:"/var/mobile/Containers/Data/Application/APP-UUID/Documents/*" ./
```

**3. Improper ATS (App Transport Security)**

```xml
<!-- In Info.plist — this disables HTTPS enforcement -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>   <!-- HTTP allowed! Big security issue -->
</dict>
```

---

## 11. Tools Reference

### Android Tools

| Tool                | Use                     | Install                   |
| ------------------- | ----------------------- | ------------------------- |
| **apktool**         | Decompile/recompile APK | `apt install apktool`     |
| **jadx / jadx-gui** | APK → Java              | `apt install jadx`        |
| **adb**             | Device communication    | Android SDK               |
| **Frida**           | Runtime hooking         | `pip install frida-tools` |
| **MobSF**           | Automated analysis      | Docker                    |
| **dex2jar**         | DEX to JAR              | GitHub                    |
| **androguard**      | Python APK analysis     | `pip install androguard`  |
| **Objection**       | Frida wrapper (easier)  | `pip install objection`   |

### iOS Tools

| Tool                 | Use                 | Install                         |
| -------------------- | ------------------- | ------------------------------- |
| **class-dump**       | ObjC class headers  | `brew install class-dump`       |
| **Frida**            | Runtime hooking     | `pip install frida-tools`       |
| **Ghidra**           | Binary analysis     | ghidra-sre.org                  |
| **Hopper**           | Mac disassembler    | hopperapp.com                   |
| **frida-ios-dump**   | Decrypt + dump IPA  | GitHub                          |
| **Objection**        | Frida wrapper       | `pip install objection`         |
| **ideviceinstaller** | Install/manage apps | `brew install ideviceinstaller` |

### Objection — Frida Made Easy

Objection wraps Frida with ready-made commands:

```bash
pip install objection

# Attach to app
objection -g com.example.app explore

# Inside objection shell:
android hooking list classes                  # List all classes
android hooking list class_methods com.example.app.Login  # List methods
android hooking watch class_method com.example.app.Login.check  # Hook method
android sslpinning disable                    # Bypass SSL pinning!
android root disable                          # Bypass root detection!
ios sslpinning disable                        # iOS SSL bypass!
ios jailbreak disable                         # Bypass jailbreak detection!
memory dump all mem.dmp                       # Dump memory
env                                           # Show app directories
```

---

## 12. Practice Platforms

### Intentionally Vulnerable Apps

**Android:**

```
DIVA (Damn Insecure Vulnerable App)
→ Download: github.com/payatu/diva-android
→ Covers: Insecure storage, logs, input, access control

InsecureBankv2
→ Download: github.com/dineshshetty/Android-InsecureBankv2
→ Covers: Most OWASP Mobile Top 10

OWASP GoatDroid
→ OWASP's official vulnerable Android app
```

**iOS:**

```
DVIA (Damn Vulnerable iOS App)
→ damnvulnerableiosapp.com
→ Covers: Jailbreak detection, SSL pinning, data storage

iGoat
→ OWASP's iOS training app
```

### Online Platforms

| Platform          | URL            | Notes                            |
| ----------------- | -------------- | -------------------------------- |
| HackTheBox        | hackthebox.com | Mobile challenges                |
| TryHackMe         | tryhackme.com  | Android/iOS paths                |
| OWASP MAS         | mas.owasp.org  | Mobile security standard + guide |
| NowSecure Academy | nowsecure.com  | Mobile RE courses                |

### Suggested Learning Path

```
Week 1-2: Setup
  → Set up Android emulator
  → Install DIVA app
  → Practice ADB commands
  → Solve DIVA challenges 1-5

Week 3-4: Static Analysis
  → Decompile DIVA with jadx
  → Find all hardcoded secrets
  → Read AndroidManifest.xml of real apps

Week 5-6: Dynamic Analysis
  → Set up Frida on emulator
  → Hook login methods in DIVA
  → Bypass root detection in InsecureBankv2

Week 7-8: Traffic Interception
  → Set up Burp Suite with Android emulator
  → Bypass SSL pinning with Frida
  → Analyze InsecureBankv2 API traffic

Month 3+:
  → Get jailbroken iOS device
  → Install DVIA
  → Solve iOS challenges
  → Try HackTheBox mobile challenges
```

---

## Quick Reference Cheatsheet

### ADB Essentials

```bash
adb devices                              # List devices
adb shell                                # Shell access
adb install app.apk                      # Install APK
adb pull /data/data/com.app/ ./          # Pull app data
adb logcat | grep "com.app"              # Filter logs
adb shell am start -n com.app/.Activity  # Launch activity
adb reverse tcp:8080 tcp:8080            # Port forward for Burp
```

### Frida Quick Commands

```bash
frida-ps -U                              # List processes
frida -U -l script.js com.app            # Attach with script
frida -U -f com.app -l script.js         # Launch with script
objection -g com.app explore             # Easy Frida shell
```

### JADX Quick Search

```bash
jadx app.apk -d output/
grep -r "password\|secret\|api_key" output/ --include="*.java"
grep -r "http://" output/ --include="*.java"
grep -r "SharedPreferences\|SQLite" output/ --include="*.java"
```

---

_Part of the Complete Reverse Engineering Series_
_Next: Network RE → Firmware RE_
