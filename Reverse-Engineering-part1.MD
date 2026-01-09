# Complete Reverse Engineering Mastery Guide

## From Zero to World-Class Reverse Engineer

> 🎯 **Mission**: Master the art of reverse engineering - from binary analysis to advanced malware reverse engineering, from firmware extraction to game hacking.

---

## Table of Contents

1. [What is Reverse Engineering?](#what-is-reverse-engineering)
2. [Prerequisites & Fundamentals](#prerequisites--fundamentals)
3. [Phase 1: Assembly & Architecture (Months 1-3)](#phase-1-assembly--architecture)
4. [Phase 2: Static Analysis (Months 4-6)](#phase-2-static-analysis)
5. [Phase 3: Dynamic Analysis (Months 7-9)](#phase-3-dynamic-analysis)
6. [Phase 4: Malware Analysis (Months 10-12)](#phase-4-malware-analysis)
7. [Phase 5: Advanced Topics (Year 2)](#phase-5-advanced-topics)
8. [Phase 6: Expert Specializations (Year 3+)](#phase-6-expert-specializations)
9. [Tools & Environment Setup](#tools--environment-setup)
10. [Practice Labs & Challenges](#practice-labs--challenges)
11. [Career Paths](#career-paths)

---

## What is Reverse Engineering?

**Reverse Engineering** is the process of analyzing a system, software, or hardware to understand how it works, identify vulnerabilities, or modify its behavior without access to source code.

### Applications:

- **Malware Analysis** - Understanding malicious software
- **Vulnerability Research** - Finding 0-day exploits
- **Software Cracking** - Removing protections (educational only)
- **Game Hacking** - Modifying game behavior
- **Firmware Analysis** - IoT and embedded systems
- **Protocol Reverse Engineering** - Understanding proprietary protocols
- **Legacy System Recovery** - Recovering lost source code
- **Security Auditing** - Finding backdoors and vulnerabilities

### Why Learn Reverse Engineering?

- Highest paid security skill ($150k-$300k+)
- Critical for malware analysis
- Essential for exploit development
- Required for advanced penetration testing
- Opens doors to elite security research
- Used in digital forensics
- Demanded by intelligence agencies

---

## Prerequisites & Fundamentals

### 1. Required Background Knowledge

**Programming Skills (3-6 months):**

```c
// You MUST be comfortable with C
#include <stdio.h>
#include <string.h>

int main() {
    char buffer[64];
    printf("Enter password: ");
    scanf("%s", buffer);

    if (strcmp(buffer, "secret123") == 0) {
        printf("Access granted!\n");
    } else {
        printf("Access denied!\n");
    }
    return 0;
}

// Understand:
// - Pointers and memory management
// - Stack and heap
// - Data structures
// - Function calls and returns
// - System calls
```

**Essential C Concepts:**

```c
// 1. Pointers
int x = 10;
int *ptr = &x;
printf("Value: %d, Address: %p\n", *ptr, ptr);

// 2. Memory allocation
char *buffer = malloc(100);
free(buffer);

// 3. Structures
struct Person {
    char name[50];
    int age;
};

// 4. Function pointers
int (*funcPtr)(int, int) = &add;

// 5. Bitwise operations
unsigned int flags = 0x0F;
flags |= 0x10;  // Set bit
flags &= ~0x08; // Clear bit
```

**Python for Automation:**

```python
# Essential for writing analysis scripts
import struct
import binascii

# Reading binary files
with open('program.exe', 'rb') as f:
    data = f.read()

# Unpacking binary data
# Little-endian 32-bit integer
value = struct.unpack('<I', data[0:4])[0]

# Pattern searching
pattern = b"\x55\x8B\xEC"  # function prologue
offset = data.find(pattern)

# Hex dump
print(binascii.hexlify(data[0:16]))
```

### 2. Computer Architecture Fundamentals

**Von Neumann Architecture:**

```
┌──────────────────────────────────────┐
│         Central Processing Unit      │
│  ┌─────────────┐  ┌────────────────┐│
│  │   Control   │  │  Arithmetic    ││
│  │    Unit     │  │  Logic Unit    ││
│  └─────────────┘  └────────────────┘│
└──────────────────────────────────────┘
              ↕
┌──────────────────────────────────────┐
│            Memory                    │
│  ┌────────┬────────┬────────────┐   │
│  │ Code   │ Data   │   Stack    │   │
│  └────────┴────────┴────────────┘   │
└──────────────────────────────────────┘
              ↕
┌──────────────────────────────────────┐
│         Input/Output                 │
└──────────────────────────────────────┘
```

**Memory Layout:**

```
High Address
┌─────────────────┐
│  Command Line   │  Program arguments
│  & Environment  │
├─────────────────┤
│     Stack       │  Local variables, function calls
│       ↓         │  Grows downward
├─────────────────┤
│                 │  Unused space
│       ↕         │
├─────────────────┤
│       ↑         │
│      Heap       │  Dynamic memory (malloc)
│                 │  Grows upward
├─────────────────┤
│   BSS Segment   │  Uninitialized data
├─────────────────┤
│  Data Segment   │  Initialized data
├─────────────────┤
│  Text Segment   │  Program code (read-only)
└─────────────────┘
Low Address
```

### 3. Number Systems & Data Representation

**Binary, Hex, Decimal Conversion:**

```
Decimal: 255
Binary:  11111111
Hex:     0xFF

Decimal: 42
Binary:  00101010
Hex:     0x2A

# Practice conversions daily
# Use: python, calculator, or mental math
```

**Data Types in Memory:**

```
char:     1 byte   [0x41] = 'A'
short:    2 bytes  [0x00 0x2A] = 42
int:      4 bytes  [0x00 0x00 0x00 0xFF] = 255
long:     8 bytes  [0x00 0x00 0x00 0x00 0x00 0x00 0x01 0x00] = 256
pointer:  4/8 bytes (depends on architecture)

# Endianness matters!
Big-endian:    0x12345678 stored as [12 34 56 78]
Little-endian: 0x12345678 stored as [78 56 34 12]
```

---

## Phase 1: Assembly & Architecture (Months 1-3)

### Month 1: x86/x64 Assembly Basics

#### Week 1-2: Registers & Instructions

**x86/x64 Registers:**

```nasm
; General Purpose Registers (32-bit)
EAX - Accumulator (arithmetic operations)
EBX - Base register
ECX - Counter (loop operations)
EDX - Data register
ESI - Source index (string operations)
EDI - Destination index
EBP - Base pointer (stack frame)
ESP - Stack pointer

; 64-bit equivalents
RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP
R8, R9, R10, R11, R12, R13, R14, R15

; Special Purpose
EIP/RIP - Instruction pointer
EFLAGS/RFLAGS - Flags register

; Segment Registers
CS - Code segment
DS - Data segment
SS - Stack segment
ES, FS, GS - Extra segments

; Flags (EFLAGS/RFLAGS)
ZF - Zero flag
CF - Carry flag
SF - Sign flag
OF - Overflow flag
PF - Parity flag
```

**Essential x86 Instructions:**

```nasm
; Data Movement
MOV  eax, 5          ; eax = 5
LEA  eax, [ebx+8]    ; eax = address of ebx+8
PUSH eax             ; Push eax onto stack
POP  ebx             ; Pop from stack into ebx
XCHG eax, ebx        ; Exchange eax and ebx

; Arithmetic
ADD  eax, 10         ; eax += 10
SUB  eax, 5          ; eax -= 5
INC  eax             ; eax++
DEC  eax             ; eax--
MUL  ebx             ; eax = eax * ebx (unsigned)
IMUL ebx             ; eax = eax * ebx (signed)
DIV  ebx             ; eax = eax / ebx, edx = remainder

; Logical
AND  eax, 0xFF       ; eax &= 0xFF
OR   eax, 0x01       ; eax |= 0x01
XOR  eax, eax        ; eax = 0 (common idiom)
NOT  eax             ; eax = ~eax
SHL  eax, 2          ; eax << 2 (shift left)
SHR  eax, 2          ; eax >> 2 (shift right)

; Comparison & Branching
CMP  eax, 10         ; Compare eax with 10
TEST eax, eax        ; AND eax with itself (check if zero)
JE   label           ; Jump if equal (ZF=1)
JNE  label           ; Jump if not equal (ZF=0)
JG   label           ; Jump if greater
JL   label           ; Jump if less
JMP  label           ; Unconditional jump

; Function Calls
CALL function        ; Push EIP and jump to function
RET                  ; Pop EIP and return
```

**Your First Assembly Program:**

```nasm
; hello.asm (x86 Linux)
section .data
    msg db 'Hello, Reverse Engineer!', 0xA
    len equ $ - msg

section .text
    global _start

_start:
    ; write(1, msg, len)
    mov eax, 4          ; sys_write
    mov ebx, 1          ; stdout
    mov ecx, msg        ; message
    mov edx, len        ; length
    int 0x80            ; syscall

    ; exit(0)
    mov eax, 1          ; sys_exit
    xor ebx, ebx        ; return 0
    int 0x80

; Assemble and link:
; nasm -f elf32 hello.asm -o hello.o
; ld -m elf_i386 hello.o -o hello
; ./hello
```

#### Week 3-4: Stack Operations & Function Calls

**Stack Frame Structure:**

```
High Address
┌─────────────────┐
│  Return Address │ ← Pushed by CALL
├─────────────────┤
│   Saved EBP     │ ← Old base pointer
├─────────────────┤ ← EBP points here
│  Local Var 1    │
├─────────────────┤
│  Local Var 2    │
├─────────────────┤ ← ESP points here
│      ...        │
└─────────────────┘
Low Address
```

**Function Prologue & Epilogue:**

```nasm
; Function prologue
push ebp             ; Save old base pointer
mov  ebp, esp        ; Set new base pointer
sub  esp, 0x10       ; Allocate 16 bytes for locals

; Function body
; ...

; Function epilogue
mov  esp, ebp        ; Restore stack pointer
pop  ebp             ; Restore base pointer
ret                  ; Return to caller

; Alternative: LEAVE instruction
leave                ; Equivalent to: mov esp,ebp; pop ebp
ret
```

**Calling Conventions:**

```nasm
; cdecl (C default)
; Caller cleans stack
; Arguments pushed right-to-left
push arg3
push arg2
push arg1
call function
add  esp, 12         ; Caller cleanup (3 args * 4 bytes)

; stdcall (Windows API)
; Callee cleans stack
push arg3
push arg2
push arg1
call function        ; Function does cleanup with RET n

; fastcall
; First 2 args in ECX, EDX
; Rest on stack
mov  ecx, arg1
mov  edx, arg2
push arg3
call function

; x64 calling convention (Windows)
; RCX, RDX, R8, R9 for first 4 args
; Rest on stack
mov  rcx, arg1
mov  rdx, arg2
mov  r8,  arg3
mov  r9,  arg4
call function

; x64 System V (Linux)
; RDI, RSI, RDX, RCX, R8, R9
mov  rdi, arg1
mov  rsi, arg2
mov  rdx, arg3
call function
```

**Practical Exercise:**

```nasm
; Function to add two numbers
; int add(int a, int b)
add_function:
    push ebp
    mov  ebp, esp

    ; Access parameters
    mov  eax, [ebp+8]   ; First parameter (a)
    mov  ebx, [ebp+12]  ; Second parameter (b)
    add  eax, ebx       ; Result in eax

    pop  ebp
    ret

; Calling the function
push 20              ; Second argument
push 10              ; First argument
call add_function
add  esp, 8          ; Clean up stack
; Result now in eax (30)
```

### Month 2: Reading Disassembly

#### Week 5-6: Common Patterns Recognition

**Recognizing Control Structures:**

```nasm
; IF-ELSE Statement
; if (x == 5) { ... } else { ... }

cmp  eax, 5
jne  else_block
    ; if block
    mov ebx, 1
    jmp end_if
else_block:
    ; else block
    mov ebx, 0
end_if:
    ; continue

; WHILE Loop
; while (counter > 0) { counter--; }

loop_start:
    cmp  ecx, 0
    jle  loop_end
    ; loop body
    dec  ecx
    jmp  loop_start
loop_end:

; FOR Loop
; for (i=0; i<10; i++) { ... }

xor  ecx, ecx         ; i = 0
for_start:
    cmp  ecx, 10
    jge  for_end
    ; loop body
    inc  ecx
    jmp  for_start
for_end:

; SWITCH Statement
; switch (value) { case 1: ... case 2: ... }

cmp  eax, 1
je   case_1
cmp  eax, 2
je   case_2
jmp  default_case
case_1:
    ; case 1 code
    jmp  switch_end
case_2:
    ; case 2 code
    jmp  switch_end
default_case:
    ; default code
switch_end:
```

**String Operations:**

```nasm
; strcmp equivalent
string_compare:
    mov  esi, string1
    mov  edi, string2
loop_cmp:
    lodsb              ; Load byte from [esi] into al, esi++
    scasb              ; Compare al with [edi], edi++
    jne  not_equal
    test al, al        ; Check if null terminator
    jnz  loop_cmp
    ; Strings equal
    xor  eax, eax
    ret
not_equal:
    mov  eax, 1
    ret

; strcpy equivalent
string_copy:
    mov  esi, source
    mov  edi, dest
copy_loop:
    lodsb
    stosb              ; Store al to [edi], edi++
    test al, al
    jnz  copy_loop
    ret

; strlen equivalent
string_length:
    mov  edi, string
    xor  eax, eax      ; counter
    xor  ecx, ecx
    not  ecx           ; ecx = 0xFFFFFFFF (max count)
    repne scasb        ; Scan for null byte
    not  ecx
    dec  ecx           ; Length in ecx
    mov  eax, ecx
    ret
```

**Array Access:**

```nasm
; int array[10];
; array[i] = value;

; Formula: address = base + (index * element_size)

mov  eax, [ebp-40]   ; i
mov  edx, 4          ; sizeof(int)
imul eax, edx        ; i * 4
lea  ebx, [ebp-100]  ; base address of array
mov  [ebx+eax], ecx  ; array[i] = ecx

; Or using scaled index addressing
mov  eax, [ebp-40]   ; i
mov  ecx, 42         ; value
lea  ebx, [ebp-100]  ; array base
mov  [ebx+eax*4], ecx ; array[i] = 42 (direct)
```

#### Week 7-8: Analyzing Real Programs

**Practice with Simple C Programs:**

```c
// password_checker.c
#include <stdio.h>
#include <string.h>

int check_password(char *input) {
    char correct[] = "secret123";
    return strcmp(input, correct) == 0;
}

int main() {
    char input[100];
    printf("Enter password: ");
    scanf("%s", input);

    if (check_password(input)) {
        printf("Access Granted!\n");
    } else {
        printf("Access Denied!\n");
    }
    return 0;
}

// Compile and disassemble:
// gcc password_checker.c -o password_checker
// objdump -d password_checker > disassembly.txt
// gdb password_checker
```

**Analyzing the Disassembly:**

```nasm
; Look for main function
0804846b <main>:
 804846b: push   ebp
 804846c: mov    ebp,esp
 804846e: sub    esp,0x70          ; Allocate 112 bytes (0x70)
 8048471: push   0x8048560         ; "Enter password: "
 8048476: call   8048330 <printf@plt>
 804847b: lea    eax,[ebp-0x6c]    ; Address of input buffer
 804847e: push   eax
 804847f: push   0x8048571         ; "%s" format
 8048484: call   8048340 <scanf@plt>
 8048489: add    esp,0x8
 804848c: lea    eax,[ebp-0x6c]    ; Load input address
 804848f: push   eax
 8048490: call   804843b <check_password>
 8048495: test   eax,eax           ; Check return value
 8048497: je     80484a8           ; Jump if password wrong
 8048499: push   0x8048574         ; "Access Granted!"
 804849e: call   8048330 <printf@plt>
 80484a3: jmp    80484b4
 80484a8: push   0x8048584         ; "Access Denied!"
 80484ad: call   8048330 <printf@plt>
 80484b4: xor    eax,eax
 80484b6: leave
 80484b7: ret

; Findings:
; 1. Stack buffer at ebp-0x6c (108 bytes)
; 2. Potential buffer overflow (scanf with no length check)
; 3. Password check at 0x804843b
; 4. Success/fail strings at 0x8048574 and 0x8048584
```

### Month 3: ARM & Other Architectures

**ARM Assembly Basics:**

```arm
; ARM Registers
R0-R12  - General purpose
R13/SP  - Stack pointer
R14/LR  - Link register (return address)
R15/PC  - Program counter
CPSR    - Current Program Status Register

; Common ARM Instructions
MOV  R0, #5           ; R0 = 5
ADD  R0, R1, R2       ; R0 = R1 + R2
SUB  R0, R1, #10      ; R0 = R1 - 10
MUL  R0, R1, R2       ; R0 = R1 * R2

LDR  R0, [R1]         ; Load word from address in R1
STR  R0, [R1]         ; Store word to address in R1
LDM  R13, {R0-R3}     ; Load multiple registers
STM  R13, {R0-R3}     ; Store multiple registers

CMP  R0, #10          ; Compare R0 with 10
BEQ  label            ; Branch if equal
BNE  label            ; Branch if not equal
B    label            ; Unconditional branch
BL   function         ; Branch with link (call)
BX   LR               ; Branch and exchange (return)

; ARM Function Prologue
PUSH {R11, LR}        ; Save frame pointer and return address
MOV  R11, SP          ; Set frame pointer

; ARM Function Epilogue
POP  {R11, PC}        ; Restore and return
```

**MIPS Assembly:**

```mips
; MIPS Registers
$zero/$0  - Always 0
$at/$1    - Assembler temporary
$v0-$v1   - Return values
$a0-$a3   - Function arguments
$t0-$t9   - Temporary registers
$s0-$s7   - Saved registers
$sp/$29   - Stack pointer
$ra/$31   - Return address

; Common MIPS Instructions
li   $t0, 5           # Load immediate
la   $t0, label       # Load address
move $t0, $t1         # Move register

lw   $t0, 0($sp)      # Load word from stack
sw   $t0, 0($sp)      # Store word to stack

add  $t0, $t1, $t2    # $t0 = $t1 + $t2
sub  $t0, $t1, $t2    # $t0 = $t1 - $t2
mul  $t0, $t1, $t2    # $t0 = $t1 * $t2

beq  $t0, $t1, label  # Branch if equal
bne  $t0, $t1, label  # Branch if not equal
j    label            # Jump
jal  function         # Jump and link (call)
jr   $ra              # Jump register (return)
```

---

## Phase 2: Static Analysis (Months 4-6)

### Month 4: Disassemblers & Decompilers

#### Week 13-14: IDA Pro / Ghidra Mastery

**IDA Pro Essential Operations:**

```
# Installation & Setup
1. Download IDA Free (or Pro if you can afford)
2. Install hexrays decompiler plugin
3. Configure options (Analysis -> General)

# Basic Navigation
G - Jump to address
X - Cross-references
N - Rename variable/function
Y - Change type
; - Add comment
: - Add repeatable comment
U - Undefine
C - Code (convert to code)
D - Data (convert to data)
A - ASCII string
Space - Toggle graph/text view

# Analysis Features
Auto-analysis on load
Function recognition
String discovery
Import/Export analysis
Cross-reference generation
Type information recovery

# Scripting (Python)
import idaapi
import idc
import idautils

# Get all functions
for func in idautils.Functions():
    name = idc.get_func_name(func)
    print(f"Function: {name} at 0x{func:x}")

# Find all strings
for string in idautils.Strings():
    print(f"{string}")

# Patch bytes
idc.patch_byte(0x401000, 0x90)  # NOP

# Change function name
idc.set_name(0x401000, "main")
```

**Ghidra Workflow:**

```
# Project Setup
1. Create new project
2. Import binary
3. Analyze (Auto Analysis)
4. Review decompilation

# Key Features
- Better decompiler than IDA Free
- Free and open source
- Supports many architectures
- Team collaboration features
- Scripting in Python/Java

# Ghidra Shortcuts
L - Label (rename)
; - Comment
Ctrl+Shift+G - Goto address
Ctrl+E - Edit function signature
Ctrl+L - Retype variable
F - Create function
U - Undefined

# Ghidra Scripting (Python)
from ghidra.program.model.symbol import SymbolType

# Get all functions
fm = currentProgram.getFunctionManager()
for func in fm.getFunctions(True):
    print(f"Function: {func.getName()} at {func.getEntryPoint()}")

# Find string references
from ghidra.program.model.data import StringDataInstance

for string in currentProgram.getListing().getDefinedData(True):
    if isinstance(string, StringDataInstance):
        print(f"String: {string}")
```

**Binary Ninja:**

```
# Why Binary Ninja?
- Modern UI
- Intermediate Language (IL)
- Great for beginners
- Fast analysis
- Good API

# Key Features
- BNIL (Binary Ninja IL)
- MLIL (Medium-Level IL)
- HLIL (High-Level IL)

# Python API
import binaryninja as bn

bv = bn.open_view("program.exe")

# Iterate functions
for func in bv.functions:
    print(f"{func.name}: {func.start:x}")

# Get HLIL
for func in bv.functions:
    for block in func.hlil:
        print(block)
```

#### Week 15-16: Identifying Algorithms & Patterns

**Cryptography Recognition:**

```python
# Identifying crypto algorithms in disassembly

# AES S-box (look for this constant table)
sbox = [
    0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5,
    0x30, 0x01, 0x67, 0x2b, 0xfe, 0xd7, 0xab, 0x76,
    # ... 256 bytes total
]

# MD5 Constants
md5_constants = [
    0xd76aa478, 0xe8c7b756, 0x242070db, 0xc1bdceee,
    0xf57c0faf, 0x4787c62a, 0xa8304613, 0xfd469501,
    # ... 64 constants
]

# SHA-256 Constants (first 32 bits of fractional parts of cube roots)
sha256_k = [
    0x428a2f98, 0x71374491, 0xb5c0fbcf, 0xe9b5dba5,
    # ... 64 constants
]

# RSA Indicators
# - Large number operations (hundreds of digits)
# - Modular exponentiation
# - Prime number checks

# RC4 Key Scheduling
# - 256-byte S-box initialization
# - Simple swap operations in loop
```

**Compression Algorithm Detection:**

```nasm
; DEFLATE/ZLIB indicators
; - Huffman table building
; - Bit manipulation
; - Sliding window (32KB)

; LZ77 patterns
; - Back-reference handling
; - Length-distance pairs
; - Sliding window search

; Look for library calls
call inflate
call deflate
call uncompress
call compress2
```

**Obfuscation Patterns:**

```nasm
; Control Flow Flattening
; - Large switch statement
; - State variable updates
; - Indirect jumps

mov  eax, [state]
cmp  eax, 0
je   case_0
cmp  eax, 1
je   case_1
; ... many cases

; Opaque Predicates
; - Always true/false conditions
; - Used to confuse analysis

xor  eax, eax
cmp  eax, 1
jz   never_executed    ; Dead code

; Junk Code Insertion
nop
nop
add  eax, 0
sub  eax, 0
xor  ebx, ebx
xor  ebx, ebx          ; Cancels out

; String Obfuscation
; - XOR encoding
; - Stack strings
; - API hashing

; Decode string
mov  esi, encrypted_string
mov  ecx, length
xor  edx, edx
decode_loop:
    lodsb
    xor  al, 0x42      ; XOR key
    mov  [edi+edx], al
    inc  edx
    loop decode_loop
```

### Month 5: File Format Analysis

#### Week 17-18: PE Format Deep Dive

**PE File Structure:**

```
┌────────────────────────────────┐
│       DOS Header (64 bytes)    │
│  MZ signature (0x5A4D)         │
│  e_lfanew (offset to PE)       │
├────────────────────────────────┤
│       DOS Stub                 │
│  "This program cannot..."      │
├────────────────────────────────┤
│       PE Signature (4 bytes)   │
│  "PE\0\0"                      │
├────────────────────────────────┤
│       COFF Header (20 bytes)   │
│  Machine, NumberOfSections,    │
│  SizeOfOptionalHeader, etc.    │
├────────────────────────────────┤
│    Optional Header (varies)    │
│  AddressOfEntryPoint, ImageBase│
│  SizeOfImage, DataDirectory[]  │
├────────────────────────────────┤
│      Section Table             │
│  .text, .data, .rdata, .rsrc   │
│  VirtualAddress, SizeOfRawData │
├────────────────────────────────┤
│       Section Data             │
│  Actual code and data          │
└────────────────────────────────┘
```

**Parsing PE Files:**

```python
import pefile

pe = pefile.PE('program.exe')

# DOS Header
print(f"MZ Signature: {hex(pe.DOS_HEADER.e_magic)}")
print(f"PE Offset: {hex(pe.DOS_HEADER.e_lfanew)}")

# NT Headers
print(f"Machine: {hex(pe.FILE_HEADER.Machine)}")
print(f"Number of Sections: {pe.FILE_HEADER.NumberOfSections}")

# Optional Header
print(f"Entry Point: {hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint)}")
print(f"Image Base: {hex(pe.OPTIONAL_HEADER.ImageBase)}")
print(f"Section Alignment: {hex(pe.OPTIONAL_HEADER.SectionAlignment)}")

# Sections
for section in pe.sections:
    print(f"{section.Name.decode().strip()}: VA={hex(section.VirtualAddress)}, Size={hex(section.Misc_VirtualSize)}")

# Imports
for entry in pe.DIRECTORY_ENTRY_IMPORT:
    print(f"DLL: {entry.dll.decode()}")
    for imp in entry.imports:
        print(f"  {imp.name.decode() if imp.name else f'Ordinal {imp.ordinal}'}")

# Exports (if present)
if hasattr(pe, 'DIRECTORY_ENTRY_EXPORT'):
    for exp in pe.DIRECTORY_ENTRY_EXPORT.symbols:
        print(f"Export: {exp.name.decode()} at 0x{exp.address:x}")

# Resources
if hasattr(pe, 'DIRECTORY_ENTRY_RESOURCE'):
    for resource_type in pe.DIRECTORY_ENTRY_RESOURCE.entries:
        print(f"Resource Type: {resource_type.id}")
```

#### Week 19-20: ELF & Mach-O Analysis

**ELF Format (Linux):**

```python
from elftools.elf.elffile import ELFFile

with open('program', 'rb') as f:
    elf = ELFFile(f)

    # ELF Header
    print(f"Class: {elf.header['e_ident']['EI_CLASS']}")
    print(f"Entry Point: {hex(elf.header['e_entry'])}")

    # Sections
    for section in elf.iter_sections():
        print(f"{section.name}: {hex(section['sh_addr'])}")

    # Symbols
    symtab = elf.get_section_by_name('.symtab')
    if symtab:
        for symbol in symtab.iter_symbols():
            print(f"{symbol.name}: {hex(symbol['st_value'])}")

    # Dynamic Section
    dynamic = elf.get_section_by_name('.dynamic')
    if dynamic:
        for tag in dynamic.iter_tags():
            print(f"{tag.entry.d_tag}: {tag.entry.d_val}")
```

**Mach-O Format (macOS):**

```python
from macholib import MachO

macho = MachO.MachO('program')

for header in macho.headers:
    print(f"Architecture: {header.MH_MAGIC}")

    # Load Commands
    for cmd in header.commands:
        print(f"Command: {cmd[0].cmd}")

    # Segments
    for segment in header.segments:
        print(f"Segment: {segment.segname}")
        for section in segment.sections:
            print(f"  Section: {section.sectname}")
```

### Month 6: Pattern Matching & YARA

**YARA Rules for Malware Detection:**

```yara
rule CryptoRansomware {
    meta:
        description = "Detects ransomware with crypto functions"
        author = "Your Name"
        date = "2025-01-08"

    strings:
        // Crypto imports
        $crypt1 = "CryptEncrypt" ascii
        $crypt2 = "CryptDecrypt" ascii
        $crypt3 = "CryptAcquireContext" ascii

        // File operations
        $file1 = "CreateFileW" ascii
        $file2 = "WriteFile" ascii

        // Registry for persistence
        $reg = "RegSetValueEx" ascii

        // Ransom note indicators
        $ransom1 = "bitcoin" ascii nocase
        $ransom2 = "decrypt" ascii nocase
        $ransom3 = ".locked" ascii

    condition:
        uint16(0) == 0x5A4D and // PE file
        filesize < 5MB and
        3 of ($crypt*) and
        2 of ($file*) and
        2 of ($ransom*)
}

rule Packer_UPX {
    meta:
        description = "Detects UPX packed executables"

    strings:
        $upx1 = "UPX0" ascii
        $upx2 = "UPX1" ascii
        $upx3 = "UPX!" ascii

    condition:
        uint16(0) == 0x5A4D and
        all of them
}

rule Obfuscated_Strings {
    meta:
        description = "Detects XOR obfuscated strings"

    strings:
        // Common XOR loop pattern
        $xor_loop = { 8A ?? ?? ?? ?? ?? 34 ?? 88 ?? ?? ?? ?? ?? E2 ?? }
        // lodsb; xor al, key; stosb; loop

    condition:
        $xor_loop
}
```

**Using YARA:**

```bash
# Scan single file
yara rules.yar malware.exe

# Scan directory recursively
yara -r rules.yar /path/to/samples/

# Fast scan mode
yara -f rules.yar malware.exe

# Print strings
yara -s rules.yar malware.exe

# Python usage
import yara

rules = yara.compile(filepath='rules.yar')
matches = rules.match('malware.exe')

for match in matches:
    print(f"Rule: {match.rule}")
    for string in match.strings:
        print(f"  String: {string}")
```

---

## Phase 3: Dynamic Analysis (Months 7-9)

### Month 7: Debuggers & Debugging Techniques

#### Week 25-26: GDB Mastery (Linux)

**GDB Essential Commands:**

```bash
# Starting GDB
gdb ./program
gdb -p PID              # Attach to running process
gdb --args program arg1 arg2

# Execution Control
run (r)                 # Start program
start                   # Break at main and run
continue (c)            # Continue execution
next (n)                # Step over
step (s)                # Step into
finish                  # Run until function returns
until                   # Run until line
kill                    # Kill process

# Breakpoints
break main              # Break at function
break *0x401000         # Break at address
break file.c:42         # Break at line
tbreak                  # Temporary breakpoint
delete 1                # Delete breakpoint 1
disable 1               # Disable breakpoint 1
enable 1                # Enable breakpoint 1
info breakpoints        # List all breakpoints

# Examining Memory
x/10i $rip              # Examine 10 instructions
x/10x $rsp              # Examine 10 hex words on stack
x/s 0x401000            # Examine string
x/10gx $rsp             # 10 giant (8-byte) hex values

# Registers
info registers          # Show all registers
info registers rax rbx  # Show specific registers
set $rax = 0x1234       # Modify register

# Memory
print var               # Print variable
print/x $rax            # Print in hex
print/t $rax            # Print in binary
print *0x401000         # Dereference pointer
set {int}0x401000 = 42  # Write to memory

# Disassembly
disassemble main        # Disassemble function
disas /r main           # With raw bytes
set disassembly-flavor intel

# Watchpoints
watch var               # Break when var changes
rwatch var              # Break when var is read
awatch var              # Break on read/write

# Stack
backtrace (bt)          # Show call stack
frame 0                 # Switch to frame
info frame              # Frame info
info locals             # Local variables
info args               # Function arguments

# Advanced
catch syscall           # Break on any syscall
catch syscall read write # Break on specific syscalls
commands 1              # Add commands to breakpoint 1
  silent
  printf "RAX = 0x%lx\n", $rax
  continue
end
```

**GDB with Python Scripting:**

```python
# GDB Python API
import gdb

class CustomBreakpoint(gdb.Breakpoint):
    def __init__(self, location):
        super().__init__(location)

    def stop(self):
        # Get register values
        rax = gdb.parse_and_eval("$rax")
        print(f"RAX at breakpoint: {rax}")

        # Continue execution
        return False  # True to stop, False to continue

# Set breakpoint
CustomBreakpoint("*0x401000")

# Command to dump memory
class DumpCommand(gdb.Command):
    def __init__(self):
        super().__init__("dump-stack", gdb.COMMAND_DATA)

    def invoke(self, arg, from_tty):
        sp = gdb.parse_and_eval("$rsp")
        for i in range(10):
            addr = sp + (i * 8)
            val = gdb.parse_and_eval(f"*(long*)0x{addr:x}")
            print(f"[RSP+{i*8:2d}] 0x{addr:016x}: 0x{val:016x}")

DumpCommand()
```

**PEDA/GEF/PWNDBG Enhancements:**

```bash
# PEDA (Python Exploit Development Assistance)
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit

# GEF (GDB Enhanced Features)
bash -c "$(curl -fsSL https://gef.sh/)"

# PWNDBG
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh

# Enhanced Commands
context                 # Show registers, stack, code
checksec                # Check binary protections
vmmap                   # Show memory mappings
telescope $rsp 20       # Examine stack with dereferencing
pattern create 100      # Create cyclic pattern
pattern offset 0x41414241 # Find offset in pattern
rop                     # Find ROP gadgets
```

#### Week 27-28: WinDbg & x64dbg (Windows)

**WinDbg Commands:**

```
# Execution
g                       # Go
p                       # Step over
t                       # Step into
gu                      # Step out
pt                      # Trace to next return

# Breakpoints
bp address              # Set breakpoint
bu address              # Unresolved breakpoint
ba r4 address           # Hardware breakpoint (read)
ba w4 address           # Hardware breakpoint (write)
bl                      # List breakpoints
bc *                    # Clear all breakpoints

# Memory
d address               # Display memory
da address              # ASCII string
du address              # Unicode string
db address L100         # 100 bytes
dd address L10          # 10 dwords
dq address L8           # 8 qwords

# Registers
r                       # Show registers
r rax=1234              # Set register

# Disassembly
u address               # Unassemble
uf function             # Unassemble function

# Search
s -a 0 L?0x7fffffff "password"  # Search ASCII
s -b 0 L?0x7fffffff 90 90 90    # Search bytes

# Modules
lm                      # List modules
lmvm module             # Module details
x module!*              # List symbols

# Stack
k                       # Stack trace
kb                      # With frame pointer
kv                      # Verbose

# Extensions
!exploitable            # Crash analysis
!address                # Memory info
!peb                    # Process Environment Block
!teb                    # Thread Environment Block
```

**x64dbg Usage:**

```
# GUI Debugger for Windows
# Download from: https://x64dbg.com

Features:
- Modern UI
- Plugin support
- Scripting with x64dbgpy
- Memory map view
- Patch functionality
- Trace recording
- Conditional breakpoints

# Useful Plugins
- ScyllaHide (anti-anti-debugging)
- xAnalyzer (automatic analysis)
- x64dbgpy (Python scripting)

# Scripting Example
# Create script.txt
bp 0x401000
run
log "RIP: {rip}"
log "RAX: {rax}"
```

### Month 8: API Monitoring & Hooking

**API Hooking Techniques:**

```c
// Inline Hooking
#include <windows.h>

BYTE original_bytes[5];
FARPROC original_func;

// Hook function
void hook_function(FARPROC target, FARPROC hook) {
    DWORD old_protect;

    // Save original bytes
    memcpy(original_bytes, target, 5);

    // Make memory writable
    VirtualProtect(target, 5, PAGE_EXECUTE_READWRITE, &old_protect);

    // Write JMP instruction
    *(BYTE*)target = 0xE9;  // JMP opcode
    *(DWORD*)((BYTE*)target + 1) = (DWORD)hook - (DWORD)target - 5;

    // Restore protection
    VirtualProtect(target, 5, old_protect, &old_protect);
}

// Hooked MessageBoxA
int WINAPI hooked_MessageBoxA(HWND hWnd, LPCSTR lpText, LPCSTR lpCaption, UINT uType) {
    printf("MessageBox called: %s\n", lpText);

    // Call original
    // First unhook, call, then rehook
    return MessageBoxA(hWnd, lpText, lpCaption, uType);
}
```

**IAT Hooking:**

```c
// Import Address Table Hooking
void hook_iat(HMODULE module, const char* import_dll, const char* import_func, void* hook) {
    PIMAGE_DOS_HEADER dos_header = (PIMAGE_DOS_HEADER)module;
    PIMAGE_NT_HEADERS nt_headers = (PIMAGE_NT_HEADERS)((BYTE*)module + dos_header->e_lfanew);

    PIMAGE_IMPORT_DESCRIPTOR import_desc = (PIMAGE_IMPORT_DESCRIPTOR)(
        (BYTE*)module + nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress
    );

    while (import_desc->Name) {
        char* dll_name = (char*)module + import_desc->Name;

        if (_stricmp(dll_name, import_dll) == 0) {
            PIMAGE_THUNK_DATA thunk = (PIMAGE_THUNK_DATA)((BYTE*)module + import_desc->FirstThunk);

            while (thunk->u1.Function) {
                PROC* func_ptr = (PROC*)&thunk->u1.Function;

                if (*func_ptr == GetProcAddress(GetModuleHandleA(import_dll), import_func)) {
                    DWORD old_protect;
                    VirtualProtect(func_ptr, sizeof(PROC), PAGE_READWRITE, &old_protect);
                    *func_ptr = (PROC)hook;
                    VirtualProtect(func_ptr, sizeof(PROC), old_protect, &old_protect);
                    return;
                }
                thunk++;
            }
        }
        import_desc++;
    }
}
```

**Frida for Dynamic Instrumentation:**

```javascript
// Frida script to hook functions
// Save as hook.js

// Hook a function
Interceptor.attach(Module.findExportByName(null, 'MessageBoxA'), {
  onEnter: function (args) {
    console.log('MessageBoxA called')
    console.log('Text: ' + Memory.readUtf8String(args[1]))
    console.log('Caption: ' + Memory.readUtf8String(args[2]))

    // Modify arguments
    args[1] = Memory.allocUtf8String('Hooked message!')
  },
  onLeave: function (retval) {
    console.log('Return value: ' + retval)
  },
})

// Hook by address
Interceptor.attach(ptr('0x401000'), {
  onEnter: function (args) {
    console.log('Function at 0x401000 called')
    // Read registers
    console.log('RAX: ' + this.context.rax)
    console.log('RBX: ' + this.context.rbx)
  },
})

// Replace function
Interceptor.replace(
  Module.findExportByName(null, 'strcmp'),
  new NativeCallback(
    function (str1, str2) {
      console.log('strcmp hooked')
      return 0 // Always return equal
    },
    'int',
    ['pointer', 'pointer'],
  ),
)
```

```bash
# Run Frida
frida -l hook.js program.exe
frida -U -n "App Name" -l hook.js  # Mobile app
```

### Month 9: Process Injection & Code Injection

**DLL Injection:**

```c
// Inject DLL into remote process
BOOL inject_dll(DWORD pid, const char* dll_path) {
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
    if (!hProcess) return FALSE;

    // Allocate memory in target process
    LPVOID remote_mem = VirtualAllocEx(
        hProcess, NULL, strlen(dll_path) + 1,
        MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE
    );

    // Write DLL path
    WriteProcessMemory(hProcess, remote_mem, dll_path, strlen(dll_path) + 1, NULL);

    // Get LoadLibraryA address
    LPVOID load_library = GetProcAddress(GetModuleHandleA("kernel32.dll"), "LoadLibraryA");

    // Create remote thread
    HANDLE hThread = CreateRemoteThread(
        hProcess, NULL, 0,
        (LPTHREAD_START_ROUTINE)load_library,
        remote_mem, 0, NULL
    );

    WaitForSingleObject(hThread, INFINITE);

    CloseHandle(hThread);
    VirtualFreeEx(hProcess, remote_mem, 0, MEM_RELEASE);
    CloseHandle(hProcess);

    return TRUE;
}
```

**Shellcode Injection:**

```c
// Execute shellcode in remote process
BOOL inject_shellcode(DWORD pid, BYTE* shellcode, SIZE_T size) {
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);

    // Allocate executable memory
    LPVOID remote_mem = VirtualAllocEx(
        hProcess, NULL, size,
        MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE
    );

    // Write shellcode
    WriteProcessMemory(hProcess, remote_mem, shellcode, size, NULL);

    // Execute
    HANDLE hThread = CreateRemoteThread(
        hProcess, NULL, 0,
        (LPTHREAD_START_ROUTINE)remote_mem,
        NULL, 0, NULL
    );

    WaitForSingleObject(hThread, INFINITE);
    CloseHandle(hThread);
    CloseHandle(hProcess);

    return TRUE;
}

// Example shellcode (MessageBox)
BYTE shellcode[] = {
    0x55,                           // push rbp
    0x48, 0x89, 0xE5,               // mov rbp, rsp
    0x48, 0x83, 0xEC, 0x20,         // sub rsp, 0x20
    // ... rest of shellcode
};
```

---

## Phase 4: Malware Analysis (Months 10-12)

### Month 10: Malware Analysis Fundamentals

**Safe Analysis Environment:**

```bash
# Virtual Machine Setup
1. Install VirtualBox/VMware
2. Create Windows VM (isolated network)
3. Snapshot before analysis
4. Install analysis tools:
   - Process Explorer
   - Process Monitor
   - Autoruns
   - Wireshark
   - FakeNet-NG
   - INetSim

# Linux Analysis VM
sudo apt install -y \
    volatility3 \
    binwalk \
    foremost \
    bulk-extractor \
    yara \
    clamav
```

**Basic Static Analysis:**

```bash
# File information
file malware.exe
md5sum malware.exe
sha256sum malware.exe

# Strings extraction
strings malware.exe > strings.txt
strings -el malware.exe > unicode_strings.txt  # Unicode

# PE analysis
pefile malware.exe

# Detect packer
detect-it-easy malware.exe
exeinfo-pe malware.exe

# Extract resources
wrestool -x malware.exe

# Check for obfuscation
upx -d malware.exe  # Try to unpack UPX

# AV scan
clamscan malware.exe
```

**Dynamic Analysis with Process Monitor:**

```
# Procmon Filters
1. Process Name is malware.exe
2. Operation is WriteFile
3. Path contains Registry
4. Result is SUCCESS

# What to monitor:
- File system activity
- Registry modifications
- Network connections
- Process creation
- Thread creation

# Export events for analysis
File -> Save -> CSV
```

**Network Behavior Analysis:**

```bash
# Capture traffic with Wireshark
wireshark -i eth0 -k

# FakeNet-NG (simulate internet)
fakenet

# INetSim (comprehensive internet simulation)
inetsim

# DNS monitoring
tcpdump -i eth0 port 53

# HTTP traffic
tcpdump -i eth0 port 80 -A

# SSL/TLS decryption
# Configure Wireshark with SSL keys
```

### Month 11: Advanced Malware Techniques

**Anti-Analysis Detection:**

```c
// VM Detection techniques
BOOL is_vm() {
    // 1. Check CPUID
    int cpuid_info[4];
    __cpuid(cpuid_info, 1);
    if (cpuid_info[2] & (1 << 31)) {
        return TRUE;  // Hypervisor bit set
    }

    // 2. Check registry
    HKEY hKey;
    if (RegOpenKeyEx(HKEY_LOCAL_MACHINE,
        "SYSTEM\\CurrentControlSet\\Services\\VBoxGuest",
        0, KEY_READ, &hKey) == ERROR_SUCCESS) {
        RegCloseKey(hKey);
        return TRUE;  // VirtualBox detected
    }

    // 3. Check MAC address
    // VMware: 00:05:69, 00:0C:29, 00:1C:14, 00:50:56

    // 4. Check processes
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe;
    pe.dwSize = sizeof(pe);

    if (Process32First(hSnapshot, &pe)) {
        do {
            if (strstr(pe.szExeFile, "vbox") ||
                strstr(pe.szExeFile, "vmware")) {
                CloseHandle(hSnapshot);
                return TRUE;
            }
        } while (Process32Next(hSnapshot, &pe));
    }
    CloseHandle(hSnapshot);

    return FALSE;
}

// Debugger Detection
BOOL is_debugger_present() {
    // 1. IsDebuggerPresent API
    if (IsDebuggerPresent()) return TRUE;

    // 2. PEB check
    BOOL isDebug = FALSE;
    __asm {
        mov eax, fs:[30h]  // PEB
        mov al, [eax+2]    // BeingDebugged
        mov isDebug, al
    }
    if (isDebug) return TRUE;

    // 3. CheckRemoteDebuggerPresent
    BOOL debuggerPresent = FALSE;
    CheckRemoteDebuggerPresent(GetCurrentProcess(), &debuggerPresent);
    if (debuggerPresent) return TRUE;

    // 4. NtQueryInformationProcess
    typedef NTSTATUS (WINAPI *pfnNtQueryInformationProcess)(
        HANDLE, UINT, PVOID, ULONG, PULONG
    );

    pfnNtQueryInformationProcess NtQIP = (pfnNtQueryInformationProcess)
        GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtQueryInformationProcess");

    if (NtQIP) {
        DWORD returnLength;
        DWORD debugPort;
        NtQIP(GetCurrentProcess(), 7, &debugPort, sizeof(debugPort), &returnLength);
        if (debugPort) return TRUE;
    }

    return FALSE;
}

// Timing checks
BOOL timing_check() {
    DWORD start = GetTickCount();
    Sleep(1000);
    DWORD end = GetTickCount();

    // If running under debugger, might take longer
    if ((end - start) > 1500) {
        return TRUE;  // Likely debugging
    }
    return FALSE;
}
```

**Bypassing Anti-Analysis:**

```python
# Patching anti-debug checks in IDA/Ghidra

# Example: Patch IsDebuggerPresent call
# Original:
# call IsDebuggerPresent
# test eax, eax
# jnz  debugger_detected

# Patch to:
# xor eax, eax  (set eax to 0)
# nop
# nop

import idaapi

# Find IsDebuggerPresent calls
for ref in idautils.CodeRefsTo(idc.get_name_ea_simple("IsDebuggerPresent"), 1):
    # Patch with XOR EAX, EAX (31 C0)
    idc.patch_byte(ref, 0x31)
    idc.patch_byte(ref + 1, 0xC0)
    # NOP the rest
    for i in range(2, 5):
        idc.patch_byte(ref + i, 0x90)
```

**Unpacking Malware:**

```python
# Generic unpacker using debugger

import os
```
