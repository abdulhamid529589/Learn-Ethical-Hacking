# Complete Reverse Engineering Mastery Guide (Continued)

## Part 3: Expert Level & Specializations

---

## Phase 4: Malware Analysis (Continued)

### Month 12: Malware Behavior Analysis (Continued)

#### Week 45-46: Dynamic Behavior Monitoring (Continued)

**Cuckoo Sandbox Setup:**

```bash
# Initialize Cuckoo
cuckoo init
cuckoo community

# Configure (cuckoo/conf/cuckoo.conf)
[cuckoo]
machinery = virtualbox
memory_dump = yes

# Add VM
cuckoo machine --add win10 192.168.56.101

# Start Cuckoo
cuckoo -d

# Submit sample
cuckoo submit malware.exe

# Web interface
cuckoo web runserver 0.0.0.0:8000
```

**Process Monitor (Procmon) Filters:**

```
# Essential filters for malware analysis

Include:
- Process Name is malware.exe
- Operation is RegSetValue
- Operation is CreateFile
- Operation is WriteFile
- Path contains HKLM\Software\Microsoft\Windows\CurrentVersion\Run
- Path contains AppData\Roaming

Exclude:
- Process Name is procmon.exe
- Process Name is System
- Path contains pagefile.sys
```

**API Monitor Configuration:**

```xml
<!-- API Monitor profile for malware -->
<ApiMonitor>
    <Modules>
        <!-- File Operations -->
        <Module Name="kernel32.dll">
            <Api Name="CreateFileA"/>
            <Api Name="CreateFileW"/>
            <Api Name="WriteFile"/>
            <Api Name="ReadFile"/>
            <Api Name="DeleteFileA"/>
            <Api Name="DeleteFileW"/>
        </Module>

        <!-- Registry Operations -->
        <Module Name="advapi32.dll">
            <Api Name="RegOpenKeyExA"/>
            <Api Name="RegOpenKeyExW"/>
            <Api Name="RegSetValueExA"/>
            <Api Name="RegSetValueExW"/>
            <Api Name="RegCreateKeyExA"/>
            <Api Name="RegCreateKeyExW"/>
        </Module>

        <!-- Network Operations -->
        <Module Name="ws2_32.dll">
            <Api Name="socket"/>
            <Api Name="connect"/>
            <Api Name="send"/>
            <Api Name="recv"/>
            <Api Name="WSAStartup"/>
        </Module>

        <!-- Process Operations -->
        <Module Name="kernel32.dll">
            <Api Name="CreateProcessA"/>
            <Api Name="CreateProcessW"/>
            <Api Name="CreateRemoteThread"/>
            <Api Name="OpenProcess"/>
            <Api Name="WriteProcessMemory"/>
        </Module>
    </Modules>
</ApiMonitor>
```

**Custom Behavior Monitor Script:**

```python
# behavior_monitor.py - Monitor malware behavior
import win32api
import win32con
import win32security
import wmi
import time
from collections import defaultdict

class BehaviorMonitor:
    def __init__(self, target_pid):
        self.target_pid = target_pid
        self.behaviors = defaultdict(list)
        self.c = wmi.WMI()

    def monitor_processes(self):
        """Monitor process creation"""
        process_watcher = self.c.Win32_Process.watch_for("creation")

        while True:
            try:
                new_process = process_watcher(timeout_ms=1000)
                if new_process.ParentProcessId == self.target_pid:
                    self.behaviors['process_creation'].append({
                        'time': time.time(),
                        'name': new_process.Name,
                        'pid': new_process.ProcessId,
                        'cmdline': new_process.CommandLine
                    })
                    print(f"[PROCESS] Created: {new_process.Name} (PID: {new_process.ProcessId})")
            except wmi.x_wmi_timed_out:
                pass

    def monitor_files(self):
        """Monitor file operations"""
        # This requires kernel-level access or API hooking
        # Use tools like Process Monitor or custom driver
        pass

    def monitor_registry(self):
        """Monitor registry operations"""
        # Simplified - requires more complex implementation
        suspicious_keys = [
            r"SOFTWARE\Microsoft\Windows\CurrentVersion\Run",
            r"SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce",
            r"SYSTEM\CurrentControlSet\Services"
        ]
        # Monitor these keys for changes
        pass

    def monitor_network(self):
        """Monitor network connections"""
        import psutil

        proc = psutil.Process(self.target_pid)

        while True:
            try:
                connections = proc.connections()
                for conn in connections:
                    if conn.status == 'ESTABLISHED':
                        self.behaviors['network'].append({
                            'time': time.time(),
                            'remote_addr': conn.raddr,
                            'local_addr': conn.laddr,
                            'status': conn.status
                        })
                        print(f"[NETWORK] Connection to {conn.raddr}")

                time.sleep(1)
            except (psutil.NoSuchProcess, psutil.AccessDenied):
                break

    def generate_report(self):
        """Generate behavior report"""
        report = []
        report.append("=== MALWARE BEHAVIOR REPORT ===\n")

        if self.behaviors['process_creation']:
            report.append("\n[Process Creation]")
            for proc in self.behaviors['process_creation']:
                report.append(f"  - {proc['name']} (PID: {proc['pid']})")
                if proc['cmdline']:
                    report.append(f"    Command: {proc['cmdline']}")

        if self.behaviors['network']:
            report.append("\n[Network Activity]")
            for net in self.behaviors['network']:
                report.append(f"  - Connected to {net['remote_addr']}")

        return '\n'.join(report)

# Usage
monitor = BehaviorMonitor(target_pid=1234)
# Start monitoring in threads
import threading
threading.Thread(target=monitor.monitor_processes, daemon=True).start()
threading.Thread(target=monitor.monitor_network, daemon=True).start()

time.sleep(300)  # Monitor for 5 minutes
print(monitor.generate_report())
```

#### Week 47-48: Network Traffic Analysis

**Wireshark for Malware Analysis:**

```
# Essential Wireshark filters

# HTTP traffic
http

# DNS queries
dns

# Suspicious ports
tcp.port == 4444 || tcp.port == 5555 || tcp.port == 6666

# Non-standard HTTP
http.request.method == "POST" && http.request.uri contains "upload"

# Large data transfers
tcp.len > 1000

# Foreign connections
!(ip.dst == 192.168.0.0/16 || ip.dst == 10.0.0.0/8)

# SSL/TLS certificates
ssl.handshake.type == 11

# IRC traffic (common in botnets)
tcp.port == 6667 || tcp.port == 6668

# Follow TCP Stream
Right-click packet -> Follow -> TCP Stream
```

**Extract Files from PCAP:**

```bash
# Using NetworkMiner
mono NetworkMiner.exe -r capture.pcap

# Using tcpflow
tcpflow -r capture.pcap

# Using Wireshark
File -> Export Objects -> HTTP/SMB/etc.

# Using tshark
tshark -r capture.pcap --export-objects http,output_dir/
```

**SSL/TLS Decryption:**

```bash
# If you have the private key
# Wireshark: Edit -> Preferences -> Protocols -> TLS
# RSA keys list: Add IP, Port, Protocol, Key file

# Using SSLsplit (MITM proxy)
sslsplit -D -l connections.log -S logdir/ \
         -k ca.key -c ca.crt \
         ssl 0.0.0.0 8443 \
         tcp 0.0.0.0 8080

# Configure malware VM to use proxy
```

**C2 Traffic Detection:**

```python
# c2_detector.py - Detect C2 traffic patterns
from scapy.all import *
from collections import defaultdict
import time

class C2Detector:
    def __init__(self):
        self.connection_intervals = defaultdict(list)
        self.beaconing_threshold = 5

    def analyze_packet(self, pkt):
        if pkt.haslayer(TCP):
            if pkt[TCP].flags & 0x02:  # SYN
                src = pkt[IP].src
                dst = pkt[IP].dst
                timestamp = time.time()

                key = f"{src}:{dst}"
                self.connection_intervals[key].append(timestamp)

                # Check for beaconing
                if len(self.connection_intervals[key]) >= self.beaconing_threshold:
                    intervals = []
                    times = self.connection_intervals[key]

                    for i in range(1, len(times)):
                        intervals.append(times[i] - times[i-1])

                    # Check if intervals are regular (beaconing)
                    if self.is_beaconing(intervals):
                        print(f"[C2] Possible beaconing detected: {key}")
                        print(f"     Average interval: {sum(intervals)/len(intervals):.2f}s")

    def is_beaconing(self, intervals):
        """Check if intervals indicate beaconing"""
        if len(intervals) < 4:
            return False

        avg = sum(intervals) / len(intervals)
        variance = sum((x - avg) ** 2 for x in intervals) / len(intervals)
        std_dev = variance ** 0.5

        # Low variance = regular intervals = beaconing
        coefficient_of_variation = std_dev / avg if avg > 0 else 0
        return coefficient_of_variation < 0.3

    def detect_dga_domains(self, pkt):
        """Detect Domain Generation Algorithm (DGA) domains"""
        if pkt.haslayer(DNSQR):
            domain = pkt[DNSQR].qname.decode()

            # DGA domains often have:
            # - Long random-looking strings
            # - High entropy
            # - Unusual TLDs

            if self.is_dga_domain(domain):
                print(f"[DGA] Suspicious domain: {domain}")

    def is_dga_domain(self, domain):
        """Check if domain looks like DGA"""
        import math
        from collections import Counter

        # Remove TLD
        domain_no_tld = domain.split('.')[0]

        # Calculate entropy
        counter = Counter(domain_no_tld)
        length = len(domain_no_tld)
        entropy = -sum((count/length) * math.log2(count/length)
                      for count in counter.values())

        # High entropy + long length = suspicious
        return entropy > 3.5 and length > 10

# Usage
detector = C2Detector()

def packet_handler(pkt):
    detector.analyze_packet(pkt)
    detector.detect_dga_domains(pkt)

sniff(prn=packet_handler, filter="tcp or udp port 53", store=0)
```

**Extracting Indicators of Compromise (IOCs):**

```python
# ioc_extractor.py
import re
import pefile
import hashlib

class IOCExtractor:
    def __init__(self, filename):
        self.filename = filename
        self.iocs = {
            'ips': set(),
            'domains': set(),
            'urls': set(),
            'emails': set(),
            'file_hashes': {},
            'registry_keys': set(),
            'mutex_names': set(),
            'file_paths': set()
        }

    def extract_from_strings(self):
        """Extract IOCs from strings"""
        with open(self.filename, 'rb') as f:
            data = f.read()

        # Decode as ASCII
        strings = self.extract_strings(data)

        for s in strings:
            # IP addresses
            ip_pattern = r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b'
            for ip in re.findall(ip_pattern, s):
                if self.is_valid_ip(ip):
                    self.iocs['ips'].add(ip)

            # Domains
            domain_pattern = r'\b(?:[a-z0-9](?:[a-z0-9-]{0,61}[a-z0-9])?\.)+[a-z]{2,}\b'
            for domain in re.findall(domain_pattern, s, re.IGNORECASE):
                self.iocs['domains'].add(domain)

            # URLs
            url_pattern = r'https?://[^\s<>"{}|\\^`\[\]]+'
            for url in re.findall(url_pattern, s, re.IGNORECASE):
                self.iocs['urls'].add(url)

            # Emails
            email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
            for email in re.findall(email_pattern, s):
                self.iocs['emails'].add(email)

            # Registry keys
            if 'HKEY' in s or 'HKLM' in s or 'HKCU' in s:
                self.iocs['registry_keys'].add(s)

            # File paths
            if ':\\' in s or s.startswith('\\\\'):
                self.iocs['file_paths'].add(s)

    def extract_strings(self, data, min_length=4):
        """Extract ASCII strings"""
        result = []
        current = []

        for byte in data:
            if 32 <= byte <= 126:
                current.append(chr(byte))
            else:
                if len(current) >= min_length:
                    result.append(''.join(current))
                current = []

        return result

    def is_valid_ip(self, ip):
        """Validate IP address"""
        parts = ip.split('.')
        if len(parts) != 4:
            return False

        try:
            return all(0 <= int(part) <= 255 for part in parts)
        except ValueError:
            return False

    def calculate_hashes(self):
        """Calculate file hashes"""
        with open(self.filename, 'rb') as f:
            data = f.read()

        self.iocs['file_hashes'] = {
            'md5': hashlib.md5(data).hexdigest(),
            'sha1': hashlib.sha1(data).hexdigest(),
            'sha256': hashlib.sha256(data).hexdigest()
        }

    def extract_from_pe(self):
        """Extract IOCs from PE imports"""
        try:
            pe = pefile.PE(self.filename)

            # Import hashing (for YARA rules)
            imports = []
            for entry in pe.DIRECTORY_ENTRY_IMPORT:
                dll = entry.dll.decode()
                for imp in entry.imports:
                    if imp.name:
                        imports.append(f"{dll}:{imp.name.decode()}")

            # Calculate imphash
            if imports:
                imphash = hashlib.md5(','.join(imports).encode()).hexdigest()
                self.iocs['file_hashes']['imphash'] = imphash
        except:
            pass

    def generate_yara_rule(self):
        """Generate YARA rule from IOCs"""
        rule = f'rule Malware_{self.iocs["file_hashes"]["md5"]}\n{{\n'
        rule += '    meta:\n'
        rule += f'        description = "Auto-generated from {self.filename}"\n'
        rule += f'        md5 = "{self.iocs["file_hashes"]["md5"]}"\n'
        rule += f'        sha256 = "{self.iocs["file_hashes"]["sha256"]}"\n'
        rule += '    \n    strings:\n'

        # Add string IOCs
        string_num = 1
        for domain in list(self.iocs['domains'])[:5]:
            rule += f'        $s{string_num} = "{domain}" ascii wide\n'
            string_num += 1

        for ip in list(self.iocs['ips'])[:5]:
            rule += f'        $s{string_num} = "{ip}" ascii\n'
            string_num += 1

        rule += '    \n    condition:\n'
        rule += '        any of them\n'
        rule += '}\n'

        return rule

    def export_iocs(self, format='json'):
        """Export IOCs in various formats"""
        if format == 'json':
            import json
            return json.dumps(self.iocs, indent=2, default=list)
        elif format == 'csv':
            import csv
            # CSV export implementation
            pass
        elif format == 'stix':
            # STIX format implementation
            pass

# Usage
extractor = IOCExtractor('malware.exe')
extractor.calculate_hashes()
extractor.extract_from_strings()
extractor.extract_from_pe()

print("=== IOCs ===")
print(f"IPs: {extractor.iocs['ips']}")
print(f"Domains: {extractor.iocs['domains']}")
print(f"URLs: {extractor.iocs['urls']}")
print(f"\nHashes:")
for hash_type, value in extractor.iocs['file_hashes'].items():
    print(f"  {hash_type}: {value}")

print("\n=== YARA Rule ===")
print(extractor.generate_yara_rule())
```

---

## Phase 5: Advanced Topics (Year 2)

### Month 13-15: Kernel-Level Reverse Engineering

#### Week 49-52: Windows Kernel Debugging

**Setting Up Kernel Debugging:**

```batch
REM Configure target machine (VM)
bcdedit /debug on
bcdedit /dbgsettings serial debugport:1 baudrate:115200

REM Or for network debugging
bcdedit /dbgsettings net hostip:192.168.1.10 port:50000

REM Configure WinDbg on host
windbg -k com:pipe,port=\\.\pipe\com_1,resets=0
```

**Essential Kernel Debugging Commands:**

```
# Kernel-specific commands

!process 0 0              # List all processes
!process <addr> 7         # Detailed process info
!thread <addr>            # Thread information
!object                   # Object manager namespace
!drivers                  # Loaded drivers
!devnode 0 1              # Device tree

# Driver analysis
!drvobj <driver>          # Driver object details
!devstack <device>        # Device stack
lm m <driver>             # List modules matching pattern

# Memory
!pte <addr>               # Page table entry
!pool <addr>              # Pool information
!poolused                 # Pool usage statistics
!vm                       # Virtual memory stats

# Synchronization
!locks                    # Display locks
!thread <addr> 1f         # Thread with locks

# IRP analysis
!irp <addr>               # IRP details
!devobj <addr>            # Device object

# Breakpoints on kernel functions
bp nt!NtCreateFile
bp nt!NtReadFile
bp nt!NtWriteFile

# Tracing system calls
!syscall <number>
```

**Analyzing a Kernel Driver:**

```c
// Example rootkit driver analysis

// 1. Find SSDT (System Service Descriptor Table)
DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {
    // Rootkits often hook SSDT

    // Get SSDT address (example for older Windows)
    PSYSTEM_SERVICE_TABLE KeServiceDescriptorTable;

    // Original NtCreateFile pointer
    PVOID originalNtCreateFile = KeServiceDescriptorTable->ServiceTable[0x55];

    // Hook it
    DISABLE_WRITE_PROTECTION();
    KeServiceDescriptorTable->ServiceTable[0x55] = (PVOID)HookedNtCreateFile;
    ENABLE_WRITE_PROTECTION();

    return STATUS_SUCCESS;
}

NTSTATUS HookedNtCreateFile(/* parameters */) {
    // Malicious code here
    // Hide files, intercept data, etc.

    // Call original
    return OriginalNtCreateFile(/* parameters */);
}
```

**Detecting Kernel Hooks:**

```python
# kernel_hook_detector.py (WinDbg Python)
import pykd

def check_ssdt_hooks():
    """Check for SSDT hooks"""
    print("Checking SSDT hooks...")

    # Get SSDT address
    ssdt_addr = pykd.getOffset("nt!KeServiceDescriptorTable")
    service_table = pykd.loadPtrs(ssdt_addr, 1)[0]

    # Number of system calls
    num_services = 0x191  # Windows 10

    hooks = []
    for i in range(num_services):
        addr = pykd.loadPtrs(service_table + i * 8, 1)[0]

        # Check if address is outside ntoskrnl
        module = pykd.findModule(addr)
        if module and 'ntoskrnl' not in module.lower():
            symbol = pykd.findSymbol(addr)
            hooks.append({
                'index': i,
                'address': hex(addr),
                'module': module,
                'symbol': symbol
            })
            print(f"[!] SSDT[{i}] hooked: {hex(addr)} ({module})")

    return hooks

def check_idt_hooks():
    """Check Interrupt Descriptor Table hooks"""
    print("Checking IDT hooks...")

    # Get IDT base
    idt_base = pykd.reg("idtr")

    hooks = []
    for i in range(256):
        # Read IDT entry
        entry_addr = idt_base + i * 16
        # Parse IDT entry structure
        # Check if handler is outside expected modules
        pass

    return hooks

def check_inline_hooks():
    """Check for inline hooks in kernel functions"""
    print("Checking inline hooks...")

    critical_functions = [
        "nt!NtCreateFile",
        "nt!NtReadFile",
        "nt!NtWriteFile",
        "nt!PsCreateSystemThread",
        "nt!ZwQuerySystemInformation"
    ]

    hooks = []
    for func_name in critical_functions:
        try:
            addr = pykd.getOffset(func_name)

            # Read first 16 bytes
            bytes_data = pykd.loadBytes(addr, 16)

            # Check for hooks (JMP, PUSH+RET, etc.)
            if bytes_data[0] == 0xE9:  # JMP rel32
                target = int.from_bytes(bytes_data[1:5], 'little')
                target_addr = addr + target + 5

                module = pykd.findModule(target_addr)
                if module and 'ntoskrnl' not in module.lower():
                    hooks.append({
                        'function': func_name,
                        'type': 'JMP',
                        'target': hex(target_addr),
                        'module': module
                    })
                    print(f"[!] {func_name} hooked with JMP to {hex(target_addr)}")

            elif bytes_data[0] == 0xFF and bytes_data[1] == 0x25:  # JMP [rip+offset]
                # x64 indirect jump
                print(f"[!] {func_name} hooked with indirect JMP")

        except:
            continue

    return hooks

# Run checks
ssdt_hooks = check_ssdt_hooks()
idt_hooks = check_idt_hooks()
inline_hooks = check_inline_hooks()

print(f"\nFound {len(ssdt_hooks) + len(idt_hooks) + len(inline_hooks)} hooks total")
```

#### Week 53-56: Rootkit Analysis

**Common Rootkit Techniques:**

```c
// 1. DKOM (Direct Kernel Object Manipulation)
// Hide process by unlinking from EPROCESS list

typedef struct _EPROCESS {
    // ... many fields ...
    LIST_ENTRY ActiveProcessLinks;  // +0x2e8 on Windows 10
    // ... more fields ...
} EPROCESS, *PEPROCESS;

void HideProcess(PEPROCESS Process) {
    // Unlink from process list
    PLIST_ENTRY prev = Process->ActiveProcessLinks.Flink;
    PLIST_ENTRY next = Process->ActiveProcessLinks.Blink;

    prev->Blink = next;
    next->Flink = prev;

    // Process is now invisible to PsList
}

// 2. Hooking ObRegisterCallbacks
// Intercept process/thread handle operations

OB_PREOP_CALLBACK_STATUS PreCallback(
    PVOID RegistrationContext,
    POB_PRE_OPERATION_INFORMATION OperationInformation)
{
    if (OperationInformation->ObjectType == *PsProcessType) {
        // Check if target is protected process
        PEPROCESS targetProcess = (PEPROCESS)OperationInformation->Object;

        if (IsProtectedProcess(targetProcess)) {
            // Remove dangerous access rights
            if (OperationInformation->Operation == OB_OPERATION_HANDLE_CREATE) {
                OperationInformation->Parameters->CreateHandleInformation.DesiredAccess = 0;
            }
        }
    }

    return OB_PREOP_SUCCESS;
}

// 3. Filter Drivers
// Intercept file system or network operations

FLT_PREOP_CALLBACK_STATUS PreCreate(
    PFLT_CALLBACK_DATA Data,
    PCFLT_RELATED_OBJECTS FltObjects,
    PVOID *CompletionContext)
{
    PFLT_FILE_NAME_INFORMATION nameInfo = NULL;

    if (FltParseFileNameInformation(Data->Iopb->TargetFileObject, &nameInfo) == STATUS_SUCCESS) {
        if (wcsstr(nameInfo->Name.Buffer, L"hidden_file")) {
            // Hide file from access
            Data->IoStatus.Status = STATUS_NO_SUCH_FILE;
            Data->IoStatus.Information = 0;

            FltReleaseFileNameInformation(nameInfo);
            return FLT_PREOP_COMPLETE;
        }
        FltReleaseFileNameInformation(nameInfo);
    }

    return FLT_PREOP_SUCCESS_NO_CALLBACK;
}

// 4. Modifying Registry Callbacks
// Hide registry keys

NTSTATUS RegistryCallback(
    PVOID CallbackContext,
    PVOID Argument1,
    PVOID Argument2)
{
    REG_NOTIFY_CLASS notifyClass = (REG_NOTIFY_CLASS)(ULONG_PTR)Argument1;

    if (notifyClass == RegNtPostEnumerateKey) {
        PREG_POST_OPERATION_INFORMATION postInfo = (PREG_POST_OPERATION_INFORMATION)Argument2;

        // Filter out certain keys
        if (ShouldHideKey(postInfo)) {
            postInfo->Status = STATUS_NO_MORE_ENTRIES;
        }
    }

    return STATUS_SUCCESS;
}
```

**Rootkit Detection Tools:**

```powershell
# PowerShell rootkit detection script

# Check for hidden processes
$AllProcesses = Get-Process
$SystemProcesses = Get-WmiObject Win32_Process
# Compare lists for discrepancies

# Check for SSDT hooks
# Requires kernel debugger or specialized tool

# Check loaded drivers
Get-WmiObject Win32_SystemDriver | Where-Object {$_.State -eq "Running"} |
    Select-Object Name, PathName, StartMode

# Check for unsigned drivers (rootkits often unsigned)
driverquery /v | findstr /i "false"

# Check registry autoruns
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Check for hidden services
Get-Service | Where-Object {$_.Status -eq "Running"} |
    Select-Object Name, DisplayName, ServiceName

# Scan for hooks using specialized tools
# RootkitRevealer, GMER, TDSSKiller
```

### Month 16-18: Firmware and IoT Reverse Engineering

#### Week 57-60: Firmware Extraction and Analysis

**Extracting Firmware:**

```bash
# From device (if possible)
# SSH/Telnet access
cat /dev/mtd0 > firmware.bin
dd if=/dev/mtd0 of=firmware.bin

# UART access
# Connect to UART pins, interrupt boot, dump memory

# SPI Flash extraction
# Use programmer like CH341A or Bus Pirate
flashrom -p ch341a_spi -r firmware.bin

# JTAG extraction
# Use OpenOCD or similar
openocd -f interface/jlink.cfg -f target/stm32f1x.cfg \
        -c "init" -c "halt" -c "dump_image firmware.bin 0x08000000 0x20000"

# From update files
binwalk -e firmware_update.bin
```

**Analyzing Firmware:**

```bash
# Identify file type
file firmware.bin
binwalk firmware.bin

# Extract filesystem
binwalk -e firmware.bin
# Or manually
dd if=firmware.bin of=rootfs.squashfs bs=1 skip=<offset>
unsquashfs rootfs.squashfs

# Entropy analysis (find encrypted/compressed sections)
binwalk -E firmware.bin

# Search for strings
strings firmware.bin | grep -E '(password|key|secret|http|ftp)'

# Find hardcoded credentials
grep -r "root" squashfs-root/etc/
cat squashfs-root/etc/passwd
cat squashfs-root/etc/shadow

# Identify architecture
readelf -h <binary>
file <binary>

# Common architectures:
# ARM, MIPS, PowerPC, x86
```

**Firmware Analysis Tools:**

```python
# firmware_analyzer.py
import binwalk
import hashlib
from pathlib import Path

class FirmwareAnalyzer:
    def __init__(self, firmware_path):
        self.firmware_path = firmware_path
        with open(firmware_path, 'rb') as f:
            self.data = f.read()

    def identify_architecture(self):
        """Identify CPU architecture"""
        # Check magic numbers
        if self.data[0:4] == b'\x7fELF':
            elf_class = self.data[4]
            machine = int.from_bytes(self.data[18:20], 'little')

            arch_map = {
                0x03: 'x86',
                0x08: 'MIPS',
                0x14: 'PowerPC',
                0x28: 'ARM',
                0x3E: 'x86-64',
                0xB7: 'AArch64'
            }

            return arch_map.get(machine, f'Unknown (0x{machine:x})')

        return 'Unknown'

    def find_strings(self, min_length=4):
        """Extract interesting strings"""
        import re

        patterns = {
            'urls': r'https?://[^\s<>"{}|\\^`\[\]]+',
            'ips': r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b',
            'emails': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
            'paths': r'/[a-zA-Z0-9_/.-]+',
            'keys': r'(?i)(password|passwd|pwd|key|secret|token)',
        }

        results = {}
        text = self.data.decode('ascii', errors='ignore')

        for name, pattern in patterns.items():
            matches = re.findall(pattern, text)
            results[name] = list(set(matches))[:20]  # Top 20

        return results

    def calculate_entropy(self, block_size=256):
        """Calculate entropy by blocks"""
        import math
        from collections import Counter

        entropies = []

        for i in range(0, len(self.data), block_size):
            block = self.data[i:i+block_size]
            if not block:
                break

            counter = Counter(block)
            length = len(block)
            entropy = 0

            for count in counter.values():
                p = count / length
                entropy -= p * math.log2(p)

            entropies.append({
                'offset': hex(i),
                'entropy': entropy
            })

        return entropies

    def find_compression(self):
        """Find compressed sections"""
        # Look for common compression headers
        signatures = {
            b'\x1f\x8b\x08': 'gzip',
            b'BZh': 'bzip2',
            b'\xfd7zXZ\x00': 'xz',
            b'LZMA': 'LZMA',
            b'hsqs': 'SquashFS',
            b'UBI#': 'UBI',
        }

        found = []
        for sig, name in signatures.items():
            offset = 0
            while True:
                offset = self.data.find(sig, offset)
                if offset == -1:
                    break
                found.append({
                    'type': name,
                    'offset': hex(offset)
                })
                offset += 1

        return found

# Usage
analyzer = FirmwareAnalyzer('firmware.bin')
print(f"Architecture: {analyzer.identify_architecture()}")
print(f"\nStrings found:")
for category, items in analyzer.find_strings().items():
    print(f"\n{category}:")
    for item in items[:5]:
        print(f"  - {item}")
```

**Emulating Firmware:**

```bash
# Using QEMU for full system emulation

# ARM
qemu-system-arm -M versatilepb -kernel vmlinux -hda rootfs.ext2 \
                -append "root=/dev/sda" -net nic -net tap

# MIPS
qemu-system-mips -M malta -kernel vmlinux -hda rootfs.ext2 \
                 -append "root=/dev/sda"

# Using Firmadyne (automated firmware emulation)
git clone https://github.com/firmadyne/firmadyne.git
cd firmadyne
./download.sh

# Import firmware
./sources/extractor/extractor.py -b brand -sql 127.0.0.1 \
                                  -np -nk "firmware.bin" images

# Emulate
./scratch/1/run.sh

# Using QEMU user mode (single binary)
qemu-arm -L /usr/arm-linux-gnueabihf/ ./binary

# Debug with gdb
qemu-arm -g 1234 -L /usr/arm-linux-gnueabihf/ ./binary
# In another terminal:
gdb-multiarch ./binary
(gdb) target remote :1234
```

#### Week 61-64: IoT Protocol Analysis

**Common IoT Protocols:**

```python
# mqtt_analyzer.py - MQTT protocol analysis
import paho.mqtt.client as mqtt
import json

class MQTTAnalyzer:
    def __init__(self, broker, port=1883):
        self.broker = broker
        self.port = port
        self.messages = []

        self.client = mqtt.Client()
        self.client.on_connect = self.on_connect
        self.client.on_message = self.on_message

    def on_connect(self, client, userdata, flags, rc):
        print(f"Connected to MQTT broker: {self.broker}")
        # Subscribe to all topics
        client.subscribe('#')

    def on_message(self, client, userdata, msg):
        message = {
            'topic': msg.topic,
            'payload': msg.payload.decode('utf-8', errors='ignore'),
            'qos': msg.qos
        }

        self.messages.append(message)
        print(f"[{msg.topic}] {message['payload']}")

        # Try to parse as JSON
        try:
            data = json.loads(message['payload'])
            print(f"  Parsed: {data}")
        except:
            pass

    def start(self):
        self.client.connect(self.broker, self.port, 60)
        self.client.loop_forever()

    def publish_test(self, topic, payload):
        """Test publishing"""
        self.client.publish(topic, payload)

# Usage
analyzer = MQTTAnalyzer('192.168.1.100')
analyzer.start()
```

**CoAP Protocol Analysis:**

```python
# coap_fuzzer.py - CoAP fuzzing
from coapthon.client.helperclient import HelperClient
import random

class CoAPFuzzer:
    def __init__(self, host, port=5683):
        self.host = host
        self.port = port
        self.client = HelperClient(server=(host, port))

    def discover_resources(self):
        """Discover available resources"""
        response = self.client.get('/.well-known/core')
        if response:
            print(f"Resources: {response.payload}")
            return response.payload

    def fuzz_get(self, path):
        """Fuzz GET requests"""
        payloads = [
            'A' * 1000,
            '../' * 100,
            '%00' * 100,
            '<script>alert(1)</script>',
            "' OR '1'='1",
        ]

        for payload in payloads:
            try:
                response = self.client.get(f'{path}?data={payload}')
                print(f"Payload: {payload[:50]}... Response: {response.code}")
            except Exception as e:
                print(f"Error: {e}")

    def fuzz_post(self, path):
        """Fuzz POST requests"""
        payloads = [
            b'\x00' * 1000,
            b'\xff' * 1000,
            b'{"admin": true}',
            b'<xml>test</xml>',
        ]

        for payload in payloads:
            try:
                response = self.client.post(path, payload)
                print(f"Response: {response.code}")
            except Exception as e:
                print(f"Error: {e}")

# Usage
fuzzer = CoAPFuzzer('192.168.1.100')
fuzzer.discover_resources()
fuzzer.fuzz_get('/temperature')
```

**Bluetooth Low Energy (BLE) Analysis:**

```python
# ble_scanner.py
from bluepy.btle import Scanner, DefaultDelegate, Peripheral
import binascii

class ScanDelegate(DefaultDelegate):
    def __init__(self):
        DefaultDelegate.__init__(self)

    def handleDiscovery(self, dev, isNewDev, isNewData):
        if isNewDev:
            print(f"Discovered device: {dev.addr}")
        elif isNewData:
            print(f"Received new data from: {dev.addr}")

class BLEAnalyzer:
    def __init__(self):
        self.scanner = Scanner().withDelegate(ScanDelegate())
        self.devices = []

    def scan(self, timeout=10):
        """Scan for BLE devices"""
        print("Scanning for BLE devices...")
        devices = self.scanner.scan(timeout)

        for dev in devices:
            print(f"\nDevice {dev.addr} ({dev.addrType}), RSSI={dev.rssi} dB")
            self.devices.append(dev)

            for (adtype, desc, value) in dev.getScanData():
                print(f"  {desc}: {value}")

    def connect_and_enumerate(self, addr):
        """Connect to device and enumerate services"""
        print(f"\nConnecting to {addr}...")

        try:
            peripheral = Peripheral(addr)

            print("Services:")
            for service in peripheral.getServices():
                print(f"  Service: {service.uuid}")

                for char in service.getCharacteristics():
                    print(f"    Characteristic: {char.uuid}")
                    print(f"      Properties: {char.propertiesToString()}")

                    # Try to read
                    if char.supportsRead():
                        try:
                            value = char.read()
                            print(f"      Value: {binascii.hexlify(value)}")
                        except:
                            print(f"      Value: [Cannot read]")

            peripheral.disconnect()
        except Exception as e:
            print(f"Error: {e}")

    def fuzz_characteristic(self, addr, char_uuid):
        """Fuzz a characteristic"""
        payloads = [
            b'\x00' * 20,
            b'\xff' * 20,
            b'A' * 100,
            b'\x00\x01\x02\x03',
        ]

        peripheral = Peripheral(addr)
        char = peripheral.getCharacteristics(uuid=char_uuid)[0]

        for payload in payloads:
            try:
                char.write(payload)
                print(f"Wrote: {binascii.hexlify(payload)}")
            except Exception as e:
                print(f"Error: {e}")

        peripheral.disconnect()

# Usage
analyzer = BLEAnalyzer()
analyzer.scan(10)

# Connect to first device
if analyzer.devices:
    analyzer.connect_and_enumerate(analyzer.devices[0].addr)
```

**Zigbee Sniffing:**

```python
# zigbee_sniffer.py - Using Killerbee framework
from killerbee import *
import struct

class ZigbeeAnalyzer:
    def __init__(self, channel=11):
        self.channel = channel
        self.kb = KillerBee()
        self.packets = []

    def start_sniffing(self):
        """Start sniffing Zigbee packets"""
        print(f"Sniffing on channel {self.channel}")
        self.kb.sniffer_on(channel=self.channel)

        try:
            while True:
                packet = self.kb.pnext()
                if packet:
                    self.analyze_packet(packet)
                    self.packets.append(packet)
        except KeyboardInterrupt:
            print("\nStopping sniffer")
            self.kb.sniffer_off()

    def analyze_packet(self, packet):
        """Analyze Zigbee packet"""
        # Parse 802.15.4 header
        if len(packet) < 2:
            return

        fcf = struct.unpack('<H', packet[0:2])[0]
        frame_type = fcf & 0x07

        frame_types = {
            0: 'Beacon',
            1: 'Data',
            2: 'Acknowledgment',
            3: 'MAC Command'
        }

        print(f"Frame Type: {frame_types.get(frame_type, 'Unknown')}")
        print(f"Packet: {packet.hex()}")

    def inject_packet(self, packet_data):
        """Inject custom packet"""
        self.kb.inject(packet_data)

    def decrypt_packet(self, packet, key):
        """Attempt to decrypt encrypted packet"""
        # Zigbee uses AES-128 CCM
        from Crypto.Cipher import AES
        # Decryption logic here
        pass

# Usage
analyzer = ZigbeeAnalyzer(channel=15)
analyzer.start_sniffing()
```

### Month 19-21: Mobile Reverse Engineering

#### Week 65-70: Android Reverse Engineering

**Android APK Analysis:**

```bash
# Decompile APK
apktool d app.apk -o app_decompiled

# Convert DEX to JAR
d2j-dex2jar app.apk

# Decompile JAR to Java
jd-gui app-dex2jar.jar

# Using JADX (better)
jadx app.apk -d output_dir

# Analyze with MobSF
docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf

# Manual inspection
unzip app.apk
cat AndroidManifest.xml | grep -E '(permission|activity|service)'

# Find hardcoded secrets
grep -r "api_key\|password\|secret" app_decompiled/

# Analyze native libraries
find app_decompiled/lib -name "*.so" -exec file {} \;
strings app_decompiled/lib/armeabi-v7a/libnative.so
```

**Frida for Android:**

```javascript
// android_hook.js - Hook Android methods

Java.perform(function () {
  // Hook a class method
  var MainActivity = Java.use('com.example.app.MainActivity')

  MainActivity.checkLicense.implementation = function () {
    console.log('[+] License check bypassed')
    return true
  }

  // Hook login function
  var LoginActivity = Java.use('com.example.app.LoginActivity')

  LoginActivity.login.implementation = function (username, password) {
    console.log('[+] Login attempt:')
    console.log('    Username: ' + username)
    console.log('    Password: ' + password)

    // Call original
    return this.login(username, password)
  }

  // Hook SSL pinning
  var TrustManager = Java.use('com.android.org.conscrypt.TrustManagerImpl')

  TrustManager.checkTrustedRecursive.implementation = function () {
    console.log('[+] SSL pinning bypassed')
    return Java.use('java.util.ArrayList').$new()
  }

  // Hook native methods
  Interceptor.attach(
    Module.findExportByName('libnative.so', 'Java_com_example_app_Native_checkRoot'),
    {
      onEnter: function (args) {
        console.log('[+] Root check called')
      },
      onLeave: function (retval) {
        console.log('[+] Original return: ' + retval)
        retval.replace(0) // Return false (not rooted)
      },
    },
  )

  // Dump encryption keys
  var Cipher = Java.use('javax.crypto.Cipher')

  Cipher.doFinal.overload('[B').implementation = function (input) {
    console.log('[+] Cipher.doFinal called')
    console.log('    Input: ' + bytesToHex(input))

    var result = this.doFinal(input)
    console.log('    Output: ' + bytesToHex(result))

    return result
  }
})

function bytesToHex(bytes) {
  var hex = ''
  for (var i = 0; i < bytes.length; i++) {
    hex += ('0' + (bytes[i] & 0xff).toString(16)).slice(-2)
  }
  return hex
}

// Run with:
// frida -U -f com.example.app -l android_hook.js --no-pause
```

**Smali Code Analysis:**

```smali
# Understanding Smali (Android bytecode)

# Method definition
.method public checkPassword(Ljava/lang/String;)Z
    .locals 2
    .param p1, "password"    # Ljava/lang/String;

    # Load constant string
    const-string v0, "secret123"

    # Call equals method
    invoke-virtual {p1, v0}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
    move-result v1

    # Return result
    return v1
.end method

# Patching example: Always return true
.method public checkPassword(Ljava/lang/String;)Z
    .locals 1

    const/4 v0, 0x1    # Load constant 1 (true)
    return v0          # Return true
.end method

# Rebuild APK after patching
# apktool b app_decompiled -o app_patched.apk
# Sign APK
# jarsigner -keystore debug.keystore app_patched.apk androiddebugkey
```

**Automated Android Analysis:**

```python
# android_analyzer.py
import subprocess
import xml.etree.ElementTree as ET
import re

class AndroidAnalyzer:
    def __init__(self, apk_path):
        self.apk_path = apk_path
        self.package_name = None
        self.activities = []
        self.services = []
        self.receivers = []
        self.permissions = []

    def extract_manifest(self):
        """Extract and parse AndroidManifest.xml"""
        # Use aapt to get readable manifest
        cmd = ['aapt', 'dump', 'badging', self.apk_path]
        output = subprocess.check_output(cmd).decode()

        # Extract package name
        match = re.search(r"package: name='([^']+)'", output)
        if match:
            self.package_name = match.group(1)

        # Use apktool for detailed manifest
        subprocess.run(['apktool', 'd', self.apk_path, '-o', 'temp_apk'])

        tree = ET.parse('temp_apk/AndroidManifest.xml')
        root = tree.getroot()

        # Parse components
        for activity in root.findall('.//activity'):
            self.activities.append(activity.get('{http://schemas.android.com/apk/res/android}name'))

        for service in root.findall('.//service'):
            self.services.append(service.get('{http://schemas.android.com/apk/res/android}name'))

        for receiver in root.findall('.//receiver'):
            self.receivers.append(receiver.get('{http://schemas.android.com/apk/res/android}name'))

        for permission in root.findall('.//uses-permission'):
            self.permissions.append(permission.get('{http://schemas.android.com/apk/res/android}name'))

    def find_hardcoded_secrets(self):
        """Find hardcoded secrets in code"""
        secrets = []

        # Search in decompiled code
        for root, dirs, files in os.walk('temp_apk'):
            for file in files:
                if file.endswith('.smali'):
                    path = os.path.join(root, file)
                    with open(path, 'r') as f:
                        content = f.read()

                        # Look for suspicious strings
                        patterns = [
                            r'api[_-]?key',
                            r'password',
                            r'secret',
                            r'token',
                            r'http[s]?://[^\s"]+',
                        ]

                        for pattern in patterns:
                            matches = re.finditer(pattern, content, re.IGNORECASE)
                            for match in matches:
                                context = content[max(0, match.start()-50):match.end()+50]
                                secrets.append({
                                    'file': path,
                                    'pattern': pattern,
                                    'context': context
                                })

        return secrets

    def analyze_native_libs(self):
        """Analyze native libraries"""
        libs = []
        lib_path = 'temp_apk/lib'

        if os.path.exists(lib_path):
            for root, dirs, files in os.walk(lib_path):
                for file in files:
                    if file.endswith('.so'):
                        full_path = os.path.join(root, file)

                        # Get file info
                        cmd = ['file', full_path]
                        file_info = subprocess.check_output(cmd).decode()

                        # Get exported symbols
                        cmd = ['nm', '-D', full_path]
                        try:
                            symbols = subprocess.check_output(cmd).decode()
                        except:
                            symbols = ''

                        libs.append({
                            'path': full_path,
                            'info': file_info,
                            'symbols': symbols[:500]  # First 500 chars
                        })

        return libs

    def check_security_issues(self):
        """Check for common security issues"""
        issues = []

        # Check for debuggable flag
        with open('temp_apk/AndroidManifest.xml', 'r') as f:
            if 'android:debuggable="true"' in f.read():
                issues.append('App is debuggable')

        # Check for dangerous permissions
        dangerous = ['READ_SMS', 'SEND_SMS', 'READ_CONTACTS',
                    'ACCESS_FINE_LOCATION', 'CAMERA', 'RECORD_AUDIO']

        for perm in self.permissions:
            for danger in dangerous:
                if danger in perm:
                    issues.append(f'Dangerous permission: {perm}')

        # Check for exported components
        tree = ET.parse('temp_apk/AndroidManifest.xml')
        root = tree.getroot()

        for component in root.findall('.//*[@android:exported="true"]',
                                     {'android': 'http://schemas.android.com/apk/res/android'}):
            issues.append(f'Exported component: {component.tag}')

        return issues

    def generate_report(self):
        """Generate analysis report"""
        self.extract_manifest()
        secrets = self.find_hardcoded_secrets()
        libs = self.analyze_native_libs()
        issues = self.check_security_issues()

        report = f"""
=== Android APK Analysis Report ===

Package: {self.package_name}

Activities ({len(self.activities)}):
{chr(10).join('  - ' + a for a in self.activities[:10])}

Services ({len(self.services)}):
{chr(10).join('  - ' + s for s in self.services[:10])}

Permissions ({len(self.permissions)}):
{chr(10).join('  - ' + p for p in self.permissions)}

Security Issues ({len(issues)}):
{chr(10).join('  - ' + i for i in issues)}

Hardcoded Secrets Found: {len(secrets)}

Native Libraries: {len(libs)}
"""

        return report

# Usage
analyzer = AndroidAnalyzer('app.apk')
print(analyzer.generate_report())
```

#### Week 71-76: iOS Reverse Engineering

**iOS App Analysis:**

```bash
# Decrypt IPA (if encrypted)
# Use tools like: Clutch, dumpdecrypted, frida-ios-dump

# Extract IPA
unzip app.ipa

# Analyze binary
otool -L Payload/App.app/App        # Linked libraries
otool -l Payload/App.app/App        # Load commands
nm Payload/App.app/App              # Symbols
strings Payload/App.app/App         # Strings

# Class dump (Objective-C)
class-dump Payload/App.app/App > classes.txt

# Disassemble
# Hopper Disassembler, IDA Pro, Ghidra

# Check for jailbreak detection
strings Payload/App.app/App | grep -i "jailbreak\|cydia\|substrate"

# Find interesting methods
grep -i "password\|login\|auth\|token" classes.txt
```

**Frida for iOS:**

```javascript
// ios_hook.js

// Hook Objective-C method
if (ObjC.available) {
  // Hook login method
  var LoginViewController = ObjC.classes.LoginViewController

  Interceptor.attach(LoginViewController['- loginWithUsername:password:'].implementation, {
    onEnter: function (args) {
      var username = ObjC.Object(args[2]).toString()
      var password = ObjC.Object(args[3]).toString()

      console.log('[+] Login attempt:')
      console.log('    Username: ' + username)
      console.log('    Password: ' + password)
    },
  })

  // Bypass jailbreak detection
  var JailbreakDetector = ObjC.classes.JailbreakDetector

  Interceptor.attach(JailbreakDetector['- isJailbroken'].implementation, {
    onLeave: function (retval) {
      console.log('[+] Jailbreak check bypassed')
      retval.replace(0) // Return NO
    },
  })

  // Hook SSL pinning
  var NSURLSession = ObjC.classes.NSURLSession

  Interceptor.attach(NSURLSession['- dataTaskWithRequest:completionHandler:'].implementation, {
    onEnter: function (args) {
      console.log('[+] NSURLSession dataTask created')
      // Bypass pinning logic
    },
  })

  // Dump KeyChain
  var NSData = ObjC.classes.NSData

  Interceptor.attach(ObjC.classes.SecItemCopyMatching, {
    onLeave: function (retval) {
      if (retval == 0) {
        console.log('[+] Keychain item accessed')
      }
    },
  })
}

// Hook Swift methods
// Swift name mangling makes this more complex
var swiftFunction = Module.findExportByName(null, '_$s7MyApp15ViewControllerC5loginyyF')
if (swiftFunction) {
  Interceptor.attach(swiftFunction, {
    onEnter: function (args) {
      console.log('[+] Swift login function called')
    },
  })
}

// Run with:
// frida -U -f com.example.app -l ios_hook.js --no-pause
```

**iOS Binary Analysis:**

```python
# ios_analyzer.py
import subprocess
import re
import plistlib

class iOSAnalyzer:
    def __init__(self, ipa_path):
        self.ipa_path = ipa_path
        self.binary_path = None
        self.info_plist = None

    def extract_ipa(self):
        """Extract IPA"""
        subprocess.run(['unzip', '-q', self.ipa_path, '-d', 'temp_ipa'])

        # Find binary
        import os
        for root, dirs, files in os.walk('temp_ipa/Payload'):
            for file in files:
                if not file.endswith(('.png', '.plist', '.nib')):
                    path = os.path.join(root, file)
                    if self.is_binary(path):
                        self.binary_path = path
                        break

    def is_binary(self, path):
        """Check if file is Mach-O binary"""
        with open(path, 'rb') as f:
            magic = f.read(4)
            # Mach-O magic numbers
            return magic in [b'\xfe\xed\xfa\xce', b'\xfe\xed\xfa\xcf',
                           b'\xce\xfa\xed\xfe', b'\xcf\xfa\xed\xfe']

    def parse_info_plist(self):
        """Parse Info.plist"""
        plist_path = 'temp_ipa/Payload/*.app/Info.plist'
        import glob
        plists = glob.glob(plist_path)

        if plists:
            with open(plists[0], 'rb') as f:
                self.info_plist = plistlib.load(f)

    def check_encryption(self):
        """Check if binary is encrypted"""
        cmd = ['otool', '-l', self.binary_path]
        output = subprocess.check_output(cmd).decode()

        if 'cryptid 1' in output:
            return True
        return False

    def get_linked_libraries(self):
        """Get linked libraries"""
        cmd = ['otool', '-L', self.binary_path]
        output = subprocess.check_output(cmd).decode()

        libs = []
        for line in output.split('\n')[1:]:
            if line.strip():
                lib = line.strip().split()[0]
                libs.append(lib)

        return libs

    def find_interesting_strings(self):
        """Find interesting strings"""
        cmd = ['strings', self.binary_path]
        output = subprocess.check_output(cmd).decode()

        patterns = {
            'urls': r'https?://[^\s]+',
            'api_keys': r'api[_-]?key\s*[:=]\s*[^\s]+',
            'tokens': r'token\s*[:=]\s*[^\s]+',
            'jailbreak': r'(jailbreak|cydia|substrate)',
        }

        findings = {}
        for name, pattern in patterns.items():
            matches = re.findall(pattern, output, re.IGNORECASE)
            findings[name] = list(set(matches))[:10]

        return findings

    def check_security_features(self):
        """Check security
```
