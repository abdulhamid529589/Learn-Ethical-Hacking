# Complete Reverse Engineering Mastery Guide (Continued)

## Part 2: Advanced Topics & Specializations

---

## Phase 2: Static Analysis (Continued)

### Month 5: File Format Analysis (Continued)

#### Week 17-18: PE Format Deep Dive (Continued)

**Parsing PE Files (Continued):**

```python
# Sections (continued)
for section in pe.sections:
    print(f"\nSection: {section.Name.decode().strip()}")
    print(f"  Virtual Address: {hex(section.VirtualAddress)}")
    print(f"  Virtual Size: {hex(section.Misc_VirtualSize)}")
    print(f"  Raw Size: {hex(section.SizeOfRawData)}")
    print(f"  Characteristics: {hex(section.Characteristics)}")

# Import Table
print("\n=== IMPORTS ===")
for entry in pe.DIRECTORY_ENTRY_IMPORT:
    print(f"\nDLL: {entry.dll.decode()}")
    for imp in entry.imports:
        if imp.name:
            print(f"  {imp.name.decode()}")
        else:
            print(f"  Ordinal: {imp.ordinal}")

# Export Table (if exists)
if hasattr(pe, 'DIRECTORY_ENTRY_EXPORT'):
    print("\n=== EXPORTS ===")
    for exp in pe.DIRECTORY_ENTRY_EXPORT.symbols:
        print(f"{exp.name.decode()}: {hex(exp.address)}")

# Resources
if hasattr(pe, 'DIRECTORY_ENTRY_RESOURCE'):
    print("\n=== RESOURCES ===")
    for resource_type in pe.DIRECTORY_ENTRY_RESOURCE.entries:
        if resource_type.name is not None:
            name = str(resource_type.name)
        else:
            name = str(pefile.RESOURCE_TYPE.get(resource_type.struct.Id))
        print(f"Resource Type: {name}")

# TLS Callbacks
if hasattr(pe, 'DIRECTORY_ENTRY_TLS'):
    print("\n=== TLS CALLBACKS ===")
    callbacks = pe.DIRECTORY_ENTRY_TLS.struct.AddressOfCallBacks
    print(f"TLS Callbacks at: {hex(callbacks)}")

# Digital Signature
if hasattr(pe, 'DIRECTORY_ENTRY_SECURITY'):
    print("\n=== DIGITAL SIGNATURE ===")
    print("Binary is signed")
```

**Important PE Structures:**

```c
// IMAGE_DOS_HEADER
typedef struct _IMAGE_DOS_HEADER {
    WORD  e_magic;      // MZ signature (0x5A4D)
    WORD  e_cblp;
    WORD  e_cp;
    // ... more fields
    LONG  e_lfanew;     // Offset to PE header
} IMAGE_DOS_HEADER;

// IMAGE_NT_HEADERS
typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;                     // PE\0\0
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER OptionalHeader;
} IMAGE_NT_HEADERS;

// IMAGE_FILE_HEADER
typedef struct _IMAGE_FILE_HEADER {
    WORD  Machine;                  // 0x14c (x86), 0x8664 (x64)
    WORD  NumberOfSections;
    DWORD TimeDateStamp;
    DWORD PointerToSymbolTable;
    DWORD NumberOfSymbols;
    WORD  SizeOfOptionalHeader;
    WORD  Characteristics;
} IMAGE_FILE_HEADER;

// IMAGE_OPTIONAL_HEADER
typedef struct _IMAGE_OPTIONAL_HEADER {
    WORD  Magic;                    // 0x10B (32-bit), 0x20B (64-bit)
    BYTE  MajorLinkerVersion;
    BYTE  MinorLinkerVersion;
    DWORD SizeOfCode;
    DWORD SizeOfInitializedData;
    DWORD SizeOfUninitializedData;
    DWORD AddressOfEntryPoint;      // RVA of entry point
    DWORD BaseOfCode;
    ULONGLONG ImageBase;            // Preferred load address
    DWORD SectionAlignment;
    DWORD FileAlignment;
    // ... more fields
    DWORD SizeOfImage;
    DWORD SizeOfHeaders;
    DWORD CheckSum;
    WORD  Subsystem;
    WORD  DllCharacteristics;
    // ... stack/heap sizes
    DWORD NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[16];
} IMAGE_OPTIONAL_HEADER;

// IMAGE_SECTION_HEADER
typedef struct _IMAGE_SECTION_HEADER {
    BYTE  Name[8];
    DWORD VirtualSize;
    DWORD VirtualAddress;           // RVA
    DWORD SizeOfRawData;
    DWORD PointerToRawData;         // File offset
    DWORD PointerToRelocations;
    DWORD PointerToLinenumbers;
    WORD  NumberOfRelocations;
    WORD  NumberOfLinenumbers;
    DWORD Characteristics;          // Flags (execute, read, write)
} IMAGE_SECTION_HEADER;
```

**RVA to File Offset Conversion:**

```python
def rva_to_offset(pe, rva):
    """Convert RVA (Relative Virtual Address) to file offset"""
    for section in pe.sections:
        if (rva >= section.VirtualAddress and
            rva < section.VirtualAddress + section.SizeOfRawData):
            offset = rva - section.VirtualAddress
            return section.PointerToRawData + offset
    return None

# Usage
entry_point_rva = pe.OPTIONAL_HEADER.AddressOfEntryPoint
entry_point_offset = rva_to_offset(pe, entry_point_rva)
print(f"Entry point at file offset: {hex(entry_point_offset)}")
```

**Import Address Table (IAT) Hooking:**

```python
def find_iat_function(pe, dll_name, func_name):
    """Find IAT entry for a specific function"""
    for entry in pe.DIRECTORY_ENTRY_IMPORT:
        if entry.dll.decode().lower() == dll_name.lower():
            for imp in entry.imports:
                if imp.name and imp.name.decode() == func_name:
                    # Calculate IAT address
                    iat_rva = imp.address
                    iat_offset = rva_to_offset(pe, iat_rva)
                    return iat_rva, iat_offset
    return None, None

# Example: Find CreateFileA in kernel32.dll
rva, offset = find_iat_function(pe, "kernel32.dll", "CreateFileA")
if rva:
    print(f"CreateFileA IAT entry at RVA: {hex(rva)}, Offset: {hex(offset)}")
```

#### Week 19-20: ELF Format Analysis

**ELF File Structure:**

```
┌────────────────────────────────┐
│    ELF Header (52/64 bytes)    │
│  Magic: 0x7F 'E' 'L' 'F'       │
│  Class, Data, Version, ABI     │
│  Type, Machine, Entry Point    │
├────────────────────────────────┤
│    Program Headers (PHT)       │
│  LOAD, DYNAMIC, INTERP, etc.   │
│  Describes memory segments     │
├────────────────────────────────┤
│       Section Headers          │
│  .text, .data, .bss, .rodata   │
│  .symtab, .strtab, .plt, .got  │
├────────────────────────────────┤
│      Section Data              │
│  Actual code and data          │
├────────────────────────────────┤
│    String Tables               │
│  Section names, symbols        │
├────────────────────────────────┤
│    Symbol Tables               │
│  Function and variable info    │
└────────────────────────────────┘
```

**Parsing ELF Files:**

```python
from elftools.elf.elffile import ELFFile
from elftools.elf.sections import SymbolTableSection

with open('program', 'rb') as f:
    elf = ELFFile(f)

    # ELF Header
    print(f"Class: {elf.elfclass}")  # 32 or 64-bit
    print(f"Encoding: {elf.little_endian}")
    print(f"Machine: {elf['e_machine']}")
    print(f"Entry Point: {hex(elf['e_entry'])}")

    # Program Headers
    print("\n=== PROGRAM HEADERS ===")
    for segment in elf.iter_segments():
        print(f"Type: {segment['p_type']}")
        print(f"  Offset: {hex(segment['p_offset'])}")
        print(f"  VirtAddr: {hex(segment['p_vaddr'])}")
        print(f"  FileSize: {hex(segment['p_filesz'])}")
        print(f"  MemSize: {hex(segment['p_memsz'])}")
        print(f"  Flags: {segment['p_flags']}")

    # Sections
    print("\n=== SECTIONS ===")
    for section in elf.iter_sections():
        print(f"{section.name}: {hex(section['sh_addr'])}")
        print(f"  Size: {hex(section['sh_size'])}")
        print(f"  Type: {section['sh_type']}")

    # Symbol Table
    print("\n=== SYMBOLS ===")
    symtab = elf.get_section_by_name('.symtab')
    if symtab and isinstance(symtab, SymbolTableSection):
        for symbol in symtab.iter_symbols():
            if symbol['st_info']['type'] == 'STT_FUNC':
                print(f"{symbol.name}: {hex(symbol['st_value'])}")

    # Dynamic Section
    dynamic = elf.get_section_by_name('.dynamic')
    if dynamic:
        print("\n=== DYNAMIC ENTRIES ===")
        for tag in dynamic.iter_tags():
            print(f"{tag.entry.d_tag}: {hex(tag.entry.d_val)}")

    # PLT/GOT Analysis
    plt = elf.get_section_by_name('.plt')
    got = elf.get_section_by_name('.got.plt')
    if plt and got:
        print(f"\nPLT at: {hex(plt['sh_addr'])}")
        print(f"GOT at: {hex(got['sh_addr'])}")
```

**ELF Data Structures:**

```c
// ELF Header (64-bit)
typedef struct {
    unsigned char e_ident[16];  // Magic number and other info
    uint16_t      e_type;       // Object file type
    uint16_t      e_machine;    // Architecture
    uint32_t      e_version;    // Object file version
    uint64_t      e_entry;      // Entry point address
    uint64_t      e_phoff;      // Program header offset
    uint64_t      e_shoff;      // Section header offset
    uint32_t      e_flags;      // Processor-specific flags
    uint16_t      e_ehsize;     // ELF header size
    uint16_t      e_phentsize;  // Program header entry size
    uint16_t      e_phnum;      // Number of program headers
    uint16_t      e_shentsize;  // Section header entry size
    uint16_t      e_shnum;      // Number of section headers
    uint16_t      e_shstrndx;   // Section header string table index
} Elf64_Ehdr;

// Program Header
typedef struct {
    uint32_t   p_type;    // Segment type (LOAD, DYNAMIC, etc.)
    uint32_t   p_flags;   // Segment flags (R, W, X)
    uint64_t   p_offset;  // File offset
    uint64_t   p_vaddr;   // Virtual address
    uint64_t   p_paddr;   // Physical address
    uint64_t   p_filesz;  // Size in file
    uint64_t   p_memsz;   // Size in memory
    uint64_t   p_align;   // Alignment
} Elf64_Phdr;

// Section Header
typedef struct {
    uint32_t   sh_name;      // Section name (string table index)
    uint32_t   sh_type;      // Section type
    uint64_t   sh_flags;     // Section flags
    uint64_t   sh_addr;      // Address in memory
    uint64_t   sh_offset;    // File offset
    uint64_t   sh_size;      // Section size
    uint32_t   sh_link;      // Link to another section
    uint32_t   sh_info;      // Additional info
    uint64_t   sh_addralign; // Alignment
    uint64_t   sh_entsize;   // Entry size if table
} Elf64_Shdr;

// Symbol Table Entry
typedef struct {
    uint32_t      st_name;  // Symbol name (string table index)
    unsigned char st_info;  // Symbol type and binding
    unsigned char st_other; // Symbol visibility
    uint16_t      st_shndx; // Section index
    uint64_t      st_value; // Symbol value
    uint64_t      st_size;  // Symbol size
} Elf64_Sym;
```

**PLT/GOT Mechanism:**

```nasm
; Procedure Linkage Table (PLT)
; First time calling external function:

; Program calls printf@plt
call 0x400400  ; printf@plt entry

; printf@plt:
0x400400: jmp   QWORD PTR [printf@got]  ; Jump to GOT entry
0x400406: push  0                       ; Relocation index
0x40040b: jmp   0x4003f0                ; Jump to resolver

; GOT entry initially points back to PLT+6
; Resolver (in ld.so) does:
; 1. Looks up printf in libc
; 2. Writes real address to GOT
; 3. Jumps to printf

; Subsequent calls:
call 0x400400  ; printf@plt
; Now GOT contains real address, direct jump to printf
```

**Finding and Analyzing GOT/PLT:**

```python
def analyze_got_plt(elf):
    """Analyze GOT and PLT for dynamic linking"""
    plt = elf.get_section_by_name('.plt')
    got = elf.get_section_by_name('.got.plt')
    rela_plt = elf.get_section_by_name('.rela.plt')

    if not all([plt, got, rela_plt]):
        print("Missing PLT/GOT sections")
        return

    print(f"PLT at: {hex(plt['sh_addr'])}, size: {plt['sh_size']}")
    print(f"GOT at: {hex(got['sh_addr'])}, size: {got['sh_size']}")

    print("\n=== PLT ENTRIES ===")
    for reloc in rela_plt.iter_relocations():
        symbol = elf.get_section(rela_plt['sh_link']).get_symbol(reloc['r_info_sym'])
        print(f"{symbol.name}: GOT offset {hex(reloc['r_offset'])}")
```

### Month 6: Deobfuscation Techniques

#### Week 21-22: Control Flow Deobfuscation

**Control Flow Flattening - Understanding:**

```c
// Original code:
if (x > 10) {
    y = x * 2;
} else {
    y = x + 5;
}
z = y + 1;

// After control flow flattening:
int state = 0;
while (true) {
    switch (state) {
        case 0:
            if (x > 10) {
                state = 1;
            } else {
                state = 2;
            }
            break;
        case 1:
            y = x * 2;
            state = 3;
            break;
        case 2:
            y = x + 5;
            state = 3;
            break;
        case 3:
            z = y + 1;
            return;
    }
}
```

**Deobfuscating Control Flow:**

```python
# IDA Python script to reconstruct control flow
import idaapi
import idc
import idautils

def find_dispatcher_block(func_ea):
    """Find the dispatcher (switch) block in flattened control flow"""
    f = idaapi.get_func(func_ea)
    flowchart = idaapi.FlowChart(f)

    dispatcher = None
    max_succs = 0

    # Dispatcher typically has most successors
    for block in flowchart:
        succs = list(block.succs())
        if len(succs) > max_succs:
            max_succs = len(succs)
            dispatcher = block

    return dispatcher

def extract_state_graph(func_ea):
    """Extract real control flow from flattened function"""
    dispatcher = find_dispatcher_block(func_ea)
    if not dispatcher:
        return None

    state_graph = {}

    # Analyze each case block
    for succ in dispatcher.succs():
        # Find state value and next state
        state_val = get_state_value(succ.start_ea)
        next_state = get_next_state(succ.start_ea, dispatcher.start_ea)

        if state_val is not None and next_state is not None:
            state_graph[state_val] = {
                'block': succ.start_ea,
                'next': next_state
            }

    return state_graph

def reconstruct_cfg(state_graph):
    """Rebuild original control flow graph"""
    # Start from initial state (usually 0)
    original_cfg = []
    visited = set()

    def dfs(state):
        if state in visited or state not in state_graph:
            return
        visited.add(state)

        node = state_graph[state]
        original_cfg.append({
            'address': node['block'],
            'next': node['next']
        })

        dfs(node['next'])

    dfs(0)
    return original_cfg

# Usage
func_addr = 0x401000
cfg = extract_state_graph(func_addr)
if cfg:
    original = reconstruct_cfg(cfg)
    print("Reconstructed CFG:")
    for node in original:
        print(f"Block {hex(node['address'])} -> {node['next']}")
```

**Opaque Predicate Removal:**

```python
def identify_opaque_predicates(func_ea):
    """Find conditions that are always true or false"""
    f = idaapi.get_func(func_ea)
    flowchart = idaapi.FlowChart(f)

    opaque_predicates = []

    for block in flowchart:
        # Get last instruction (likely conditional jump)
        last_ea = idc.prev_head(block.end_ea)

        if idc.print_insn_mnem(last_ea) in ['jz', 'jnz', 'je', 'jne']:
            # Check if condition is opaque
            if is_always_true(last_ea) or is_always_false(last_ea):
                opaque_predicates.append({
                    'address': last_ea,
                    'type': 'always_true' if is_always_true(last_ea) else 'always_false'
                })

    return opaque_predicates

def is_always_true(ea):
    """Check if condition is always true (opaque)"""
    # Example: (x^2 >= 0) is always true
    # Look for patterns like:
    # xor eax, eax
    # test eax, eax
    # jz somewhere (always taken)

    insn = idc.GetDisasm(ea)
    prev_insn = idc.GetDisasm(idc.prev_head(ea))

    # Pattern: xor reg, reg followed by test reg, reg then jz
    if 'xor' in prev_insn and 'test' in idc.GetDisasm(idc.prev_head(idc.prev_head(ea))):
        if 'jz' in insn:
            return True

    return False

def remove_opaque_predicates(func_ea):
    """Patch out opaque predicates"""
    opaques = identify_opaque_predicates(func_ea)

    for opaque in opaques:
        addr = opaque['address']

        if opaque['type'] == 'always_true':
            # Replace conditional jump with unconditional
            # jz -> jmp
            idc.patch_byte(addr, 0xEB)  # JMP short
        else:
            # Replace with NOPs (never taken)
            insn_size = idc.get_item_size(addr)
            for i in range(insn_size):
                idc.patch_byte(addr + i, 0x90)  # NOP
```

#### Week 23-24: String and Data Deobfuscation

**XOR String Decryption:**

```python
def find_xor_decrypt_loops(func_ea):
    """Find XOR-based string decryption loops"""
    decrypt_routines = []

    f = idaapi.get_func(func_ea)
    for ea in idautils.FuncItems(func_ea):
        insn = idc.print_insn_mnem(ea)

        # Look for XOR instructions
        if insn == 'xor':
            # Check if in a loop
            if is_in_loop(ea):
                # Analyze the pattern
                pattern = analyze_xor_pattern(ea)
                if pattern:
                    decrypt_routines.append({
                        'address': ea,
                        'pattern': pattern
                    })

    return decrypt_routines

def decrypt_xor_string(encrypted_data, key):
    """Decrypt XOR-encrypted string"""
    decrypted = bytearray()

    for i, byte in enumerate(encrypted_data):
        if isinstance(key, int):
            # Single-byte XOR
            decrypted.append(byte ^ key)
        elif isinstance(key, (bytes, bytearray)):
            # Multi-byte XOR
            decrypted.append(byte ^ key[i % len(key)])

    return bytes(decrypted)

# Example usage
encrypted = b'\x1c\x1e\x1b\x1b\x18\x54\x08\x18\x09\x1b\x1d'
key = 0x6D  # Single-byte key
decrypted = decrypt_xor_string(encrypted, key)
print(f"Decrypted: {decrypted.decode()}")  # "Hello World"

# Multi-byte key
key = b'KEY'
decrypted = decrypt_xor_string(encrypted, key)
```

**Stack String Detection:**

```python
def find_stack_strings(func_ea):
    """Find strings built on the stack"""
    stack_strings = []

    # Look for series of MOV instructions to stack
    for ea in idautils.FuncItems(func_ea):
        insn = idc.print_insn_mnem(ea)

        if insn == 'mov':
            operand1 = idc.print_operand(ea, 0)
            operand2 = idc.print_operand(ea, 1)

            # Check if writing to stack (e.g., [ebp-10h])
            if '[ebp' in operand1 or '[rbp' in operand1:
                # Check if immediate value (character)
                if operand2.startswith('0x') or operand2.isdigit():
                    value = idc.get_operand_value(ea, 1)
                    if 0x20 <= value <= 0x7E:  # Printable ASCII
                        stack_strings.append({
                            'address': ea,
                            'char': chr(value),
                            'offset': operand1
                        })

    # Group consecutive writes
    groups = group_consecutive_writes(stack_strings)
    return groups

def reconstruct_stack_string(group):
    """Reconstruct string from stack writes"""
    chars = [item['char'] for item in sorted(group, key=lambda x: x['address'])]
    return ''.join(chars)
```

**API Hashing Resolution:**

```python
import hashlib

def compute_api_hash(api_name, algorithm='crc32'):
    """Compute hash of API name"""
    if algorithm == 'crc32':
        import zlib
        return zlib.crc32(api_name.encode()) & 0xFFFFFFFF
    elif algorithm == 'ror13':
        # ROR13 hash (common in malware)
        hash_val = 0
        for char in api_name.encode():
            hash_val = ((hash_val >> 13) | (hash_val << 19)) & 0xFFFFFFFF
            hash_val += char
            hash_val &= 0xFFFFFFFF
        return hash_val
    elif algorithm == 'custom':
        # Implement custom hashing algorithm
        pass

def build_api_hash_database():
    """Build database of API hashes"""
    import pefile

    common_dlls = ['kernel32.dll', 'ntdll.dll', 'user32.dll', 'advapi32.dll']
    hash_db = {}

    for dll_name in common_dlls:
        try:
            dll_path = f"C:\\Windows\\System32\\{dll_name}"
            pe = pefile.PE(dll_path)

            if hasattr(pe, 'DIRECTORY_ENTRY_EXPORT'):
                for exp in pe.DIRECTORY_ENTRY_EXPORT.symbols:
                    if exp.name:
                        api_name = exp.name.decode()

                        # Compute multiple hash types
                        crc_hash = compute_api_hash(api_name, 'crc32')
                        ror_hash = compute_api_hash(api_name, 'ror13')

                        hash_db[crc_hash] = (dll_name, api_name)
                        hash_db[ror_hash] = (dll_name, api_name)
        except:
            continue

    return hash_db

def resolve_api_hash(hash_value):
    """Resolve API hash to function name"""
    db = build_api_hash_database()
    return db.get(hash_value, ('Unknown', 'Unknown'))

# Usage
hash_val = 0x12345678
dll, api = resolve_api_hash(hash_val)
print(f"Hash {hex(hash_val)} -> {dll}:{api}")
```

**Automated String Deobfuscation Script:**

```python
# Complete IDA script for string deobfuscation
import idaapi
import idc
import idautils

class StringDeobfuscator:
    def __init__(self):
        self.decrypted_strings = []

    def find_decrypt_function(self):
        """Find the string decryption function"""
        # Look for functions with XOR operations and loops
        for func_ea in idautils.Functions():
            func_name = idc.get_func_name(func_ea)

            has_xor = False
            has_loop = False

            for ea in idautils.FuncItems(func_ea):
                if idc.print_insn_mnem(ea) == 'xor':
                    has_xor = True
                if self.is_loop_instruction(ea):
                    has_loop = True

            if has_xor and has_loop:
                return func_ea

        return None

    def extract_encrypted_data(self, xref_ea):
        """Extract encrypted data from cross-reference"""
        # Find data reference before function call
        ea = xref_ea
        for i in range(10):  # Look back 10 instructions
            ea = idc.prev_head(ea)
            if idc.print_insn_mnem(ea) in ['push', 'mov', 'lea']:
                addr = idc.get_operand_value(ea, 1)
                if idc.is_loaded(addr):
                    # Read data
                    data = bytearray()
                    while True:
                        byte = idc.get_wide_byte(addr)
                        if byte == 0:
                            break
                        data.append(byte)
                        addr += 1
                    return bytes(data)

        return None

    def decrypt_and_comment(self):
        """Find all encrypted strings and decrypt them"""
        decrypt_func = self.find_decrypt_function()
        if not decrypt_func:
            print("Decryption function not found!")
            return

        print(f"Found decryption function at {hex(decrypt_func)}")

        # Find all calls to decrypt function
        for xref in idautils.XrefsTo(decrypt_func):
            if xref.type == idaapi.fl_CN or xref.type == idaapi.fl_CF:
                # Extract encrypted data
                encrypted = self.extract_encrypted_data(xref.frm)
                if encrypted:
                    # Try to decrypt
                    decrypted = self.try_decrypt(encrypted)
                    if decrypted:
                        # Add comment
                        comment = f"Decrypted: {decrypted.decode('utf-8', errors='ignore')}"
                        idc.set_cmt(xref.frm, comment, 1)

                        self.decrypted_strings.append({
                            'address': xref.frm,
                            'encrypted': encrypted.hex(),
                            'decrypted': decrypted.decode('utf-8', errors='ignore')
                        })

        print(f"Decrypted {len(self.decrypted_strings)} strings")

    def try_decrypt(self, data):
        """Try common decryption methods"""
        # Try single-byte XOR (brute force)
        for key in range(256):
            decrypted = bytes([b ^ key for b in data])
            if self.is_printable(decrypted):
                return decrypted

        return None

    def is_printable(self, data):
        """Check if data is printable ASCII"""
        try:
            text = data.decode('utf-8')
            return all(c.isprintable() or c.isspace() for c in text)
        except:
            return False

    def is_loop_instruction(self, ea):
        """Check if instruction is part of loop"""
        insn = idc.print_insn_mnem(ea)
        return insn in ['loop', 'loope', 'loopne', 'jmp']

    def export_results(self, filename="decrypted_strings.txt"):
        """Export decrypted strings to file"""
        with open(filename, 'w') as f:
            for item in self.decrypted_strings:
                f.write(f"Address: {hex(item['address'])}\n")
                f.write(f"Encrypted: {item['encrypted']}\n")
                f.write(f"Decrypted: {item['decrypted']}\n")
                f.write("-" * 50 + "\n")

# Run the deobfuscator
deobfuscator = StringDeobfuscator()
deobfuscator.decrypt_and_comment()
deobfuscator.export_results()
```

---

## Phase 3: Dynamic Analysis (Months 7-9)

### Month 7: Debugger Mastery

#### Week 25-26: GDB and WinDbg

**GDB Essential Commands:**

```bash
# Starting GDB
gdb ./program
gdb --args ./program arg1 arg2
gdb -p <pid>  # Attach to running process

# Basic Commands
run (r)                    # Start program
start                      # Start and break at main
continue (c)              # Continue execution
next (n)                  # Step over
step (s)                  # Step into
nexti (ni)                # Step over instruction
stepi (si)                # Step into instruction
finish                    # Run until return
until                     # Run until line
quit (q)                  # Exit GDB

# Breakpoints
break main                # Break at function
break *0x400000           # Break at address
break file.c:42           # Break at line
break function if x > 10  # Conditional breakpoint
info breakpoints          # List breakpoints
delete 1                  # Delete breakpoint 1
disable 1                 # Disable breakpoint 1
enable 1                  # Enable breakpoint 1
clear function            # Clear breakpoint

# Examining Memory
x/10x $rsp               # Examine 10 hex words at stack
x/10i $rip               # Examine 10 instructions at PC
x/s 0x400000             # Examine string
x/10gx $rsp              # Examine 10 giant (8-byte) hex
print variable           # Print variable
print $rax               # Print register
print/x $rax             # Print in hex
print *ptr@10            # Print array of 10 elements

# Registers
info registers           # Show all registers
info registers rax rbx   # Show specific registers
set $rax = 0x1234       # Modify register

# Stack and Backtrace
backtrace (bt)           # Show stack trace
frame 2                  # Select frame 2
info frame               # Show current frame info
info locals              # Show local variables
info args                # Show function arguments

# Disassembly
disassemble main         # Disassemble function
disassemble 0x400000     # Disassemble at address
set disassembly-flavor intel  # Intel syntax

# Memory Mapping
info proc mappings       # Show memory mappings
vmmap                    # Show virtual memory (with peda)

# Advanced
catch syscall            # Break on syscalls
catch syscall open       # Break on specific syscall
watch variable           # Hardware watchpoint
rwatch variable          # Read watchpoint
awatch variable          # Access watchpoint

# Search Memory
find 0x400000, +0x10000, "password"  # Search for string
find /b 0x400000, +0x10000, 0x90, 0x90  # Search for bytes
```

**GDB Python Scripting:**

```python
# gdb_script.py
import gdb

class BreakpointLogger(gdb.Breakpoint):
    def __init__(self, location):
        super().__init__(location)
        self.count = 0

    def stop(self):
        self.count += 1
        rax = gdb.parse_and_eval("$rax")
        rdi = gdb.parse_and_eval("$rdi")

        print(f"[Hit {self.count}] RAX: {rax}, RDI: {rdi}")

        # Continue execution
        return False

class DumpMemory(gdb.Command):
    """Dump memory region to file"""

    def __init__(self):
        super().__init__("dump-memory", gdb.COMMAND_DATA)

    def invoke(self, arg, from_tty):
        args = arg.split()
        if len(args) != 3:
            print("Usage: dump-memory <start> <end> <filename>")
            return

        start = int(args[0], 16)
        end = int(args[1], 16)
        filename = args[2]

        inferior = gdb.selected_inferior()
        memory = inferior.read_memory(start, end - start)

        with open(filename, 'wb') as f:
            f.write(memory)

        print(f"Dumped {end-start} bytes to {filename}")

# Register commands
BreakpointLogger("malloc")
DumpMemory()

# Load in GDB:
# source gdb_script.py
```

**WinDbg Commands:**

```
# Starting WinDbg
windbg.exe program.exe
windbg.exe -p <pid>        # Attach to process
windbg.exe -z crash.dmp    # Open crash dump

# Basic Commands
g                         # Go (continue)
p                         # Step over
t                         # Step into
gu                        # Go up (step out)
q                         # Quit

# Breakpoints
bp main                   # Breakpoint at function
bp 0x401000              # Breakpoint at address
bp kernel32!CreateFileW  # Breakpoint at API
bu module!function       # Unresolved breakpoint
bl                       # List breakpoints
bc 1                     # Clear breakpoint 1
bd 1                     # Disable breakpoint 1
be 1                     # Enable breakpoint 1

# Memory Examination
db 0x401000              # Display bytes
dw 0x401000              # Display words
dd 0x401000              # Display dwords
dq 0x401000              # Display qwords
da 0x401000              # Display ASCII string
du 0x401000              # Display Unicode string
dp 0x401000              # Display pointer
dc 0x401000              # Display bytes and ASCII

# Disassembly
u 0x401000               # Unassemble at address
uf main                  # Unassemble function
ub 0x401000              # Unassemble backwards

# Registers
r                        # Display all registers
r rax                    # Display RAX
r rax = 0x1234          # Set RAX

# Stack
k                        # Stack trace
kb                       # Stack trace with parameters
kv                       # Stack trace verbose
dps rsp                  # Display pointers on stack

# Modules
lm                       # List modules
lm m kernel32            # Find kernel32
x kernel32!*File*        # List symbols matching pattern
!lmi kernel32            # Module information

# Searching
s -a 0x400000 L?0x10000 "password"  # Search ASCII
s -u 0x400000 L?0x10000 "password"  # Search Unicode
s -b 0x400000 L?0x10000 90 90       # Search bytes

# Analysis
!analyze -v              # Analyze crash
!peb                     # Process Environment Block
!teb                     # Thread Environment Block
!address 0x401000        # Get address info
!vprot 0x401000          # Virtual protect info

# Extensions
.load pykd               # Load extension
!py <script>            # Run Python script
```

#### Week 27-28: x64dbg and Advanced Debugging

**x64dbg Features:**

```
# Advantages of x64dbg
- Modern UI
- Built-in script engine
- Plugin support
- Memory map visualization
- Graph view
- Integrated stack view
- Better than OllyDbg for x64

# Essential Shortcuts
F2          - Set breakpoint
F7          - Step into
F8          - Step over
F9          - Run
Ctrl+F9     - Run until return
Ctrl+G      - Go to address
Ctrl+F      - Find pattern
Ctrl+B      - Binary search
Space       - Assemble
;           - Comment
:           - Label
*           - Follow in dump

# Plugins
- ScyllaHide (anti-anti-debug)
- xAnalyzer (auto analysis)
- x64dbgpy (Python scripting)
```

**x64dbg Scripting:**

```javascript
// x64dbg script to patch serial check
// Find the comparison
var cmp_addr = find("test eax, eax");

// Set breakpoint
bp cmp_addr;

// Run to breakpoint
run();

// Patch the jump
var jne_addr = cmp_addr + 2;
patch(jne_addr, 0xEB);  // Change JNE to JMP

// Continue
run();
```

**Anti-Debugging Detection and Bypass:**

```c
// Common anti-debugging techniques

// 1. IsDebuggerPresent
if (IsDebuggerPresent()) {
    ExitProcess(0);
}
// Bypass: Patch the check or return value

// 2. PEB BeingDebugged flag
BOOL IsDebuggerPresent_PEB() {
    PPEB pPeb = (PPEB)__readgsqword(0x60);  // x64
    return pPeb->BeingDebugged;
}
// Bypass: Set PEB->BeingDebugged to 0

// 3. NtQueryInformationProcess
BOOL CheckDebugger() {
    HANDLE hProcess = GetCurrentProcess();
    BOOL debugged = FALSE;
    NtQueryInformationProcess(hProcess, ProcessDebugPort,
                             &debugged, sizeof(debugged), NULL);
    return debugged;
}
// Bypass: Hook NtQueryInformationProcess

// 4. Timing checks
BOOL TimingCheck() {
    DWORD start = GetTickCount();
    // Some code
    DWORD end = GetTickCount();
    return (end - start) > 1000;  // Debugger is slow
}
// Bypass: Speed up or modify timing functions

// 5. Hardware breakpoint detection
BOOL CheckHardwareBreakpoints() {
    CONTEXT ctx = {0};
    ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;
    GetThreadContext(GetCurrentThread(), &ctx);

    return (ctx.Dr0 || ctx.Dr1 || ctx.Dr2 || ctx.Dr3);
}
// Bypass: Clear debug registers

// 6. INT 3 scanning
BOOL ScanForBreakpoints(BYTE *code, SIZE_T size) {
    for (SIZE_T i = 0; i < size; i++) {
        if (code[i] == 0xCC) {  // INT 3
            return TRUE;
        }
    }
    return FALSE;
}
// Bypass: Use hardware breakpoints

// 7. SEH exploitation
BOOL SEH_Check() {
    __try {
        RaiseException(EXCEPTION_BREAKPOINT, 0, 0, NULL);
    }
    __except (EXCEPTION_EXECUTE_HANDLER) {
        return FALSE;  // Not debugging
    }
    return TRUE;  // Debugging (exception passed to debugger)
}
```

**ScyllaHide Configuration:**

```ini
; ScyllaHide.ini - Anti-anti-debugging plugin

[Options]
PEB.BeingDebugged = 1
PEB.NtGlobalFlag = 1
PEB.HeapFlags = 1
NtSetInformationThread = 1
NtQuerySystemInformation = 1
NtQueryInformationProcess = 1
NtQueryObject = 1
NtClose = 1
PreventThreadCreation = 0
OutputDebugStringA = 1

[Advanced]
KiUserExceptionDispatcher = 1
DLLStealth = 1
DLLUnload = 0
```

### Month 8: Dynamic Instrumentation

#### Week 29-30: Frida Framework

**Frida Basics:**

```javascript
// Frida script basics

// Hook a function
Interceptor.attach(Module.findExportByName('libc.so', 'strcmp'), {
  onEnter: function (args) {
    console.log('strcmp called')
    console.log('  arg1: ' + Memory.readUtf8String(args[0]))
    console.log('  arg2: ' + Memory.readUtf8String(args[1]))
  },
  onLeave: function (retval) {
    console.log('strcmp returned: ' + retval)
    // Force return value
    retval.replace(0)
  },
})

// Hook by address
var base = Module.findBaseAddress('program')
var func_addr = base.add(0x1234)

Interceptor.attach(func_addr, {
  onEnter: function (args) {
    console.log('Function at 0x1234 called')
    console.log('  arg1: ' + args[0])
    console.log('  arg2: ' + args[1])
  },
})

// Replace function
Interceptor.replace(
  Module.findExportByName('libc.so', 'strlen'),
  new NativeCallback(
    function (str) {
      console.log('strlen hooked: ' + Memory.readUtf8String(str))
      return 42 // Always return 42
    },
    'int',
    ['pointer'],
  ),
)

// Memory reading
var addr = ptr('0x401000')
console.log('Byte: ' + Memory.readU8(addr))
console.log('Word: ' + Memory.readU16(addr))
console.log('Dword: ' + Memory.readU32(addr))
console.log('Qword: ' + Memory.readU64(addr))
console.log('String: ' + Memory.readUtf8String(addr))

// Memory writing
Memory.writeU32(addr, 0x12345678)
Memory.writeUtf8String(addr, 'Hello')

// Memory scanning
Memory.scan(base, 0x10000, '48 8B ? ? 48 89', {
  onMatch: function (address, size) {
    console.log('Pattern found at: ' + address)
  },
  onComplete: function () {
    console.log('Scan complete')
  },
})

// Code tracing
Stalker.follow({
  events: {
    call: true,
    ret: true,
    exec: false,
  },
  onReceive: function (events) {
    console.log('Traced ' + events.length + ' events')
  },
})
```

**Advanced Frida Techniques:**

```javascript
// Password bypass example
Java.perform(function () {
  var MainActivity = Java.use('com.example.app.MainActivity')

  MainActivity.checkPassword.implementation = function (password) {
    console.log('Password check bypassed!')
    return true // Always return true
  }
})

// SSL Pinning bypass
Java.perform(function () {
  var CertificatePinner = Java.use('okhttp3.CertificatePinner')

  CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function (
    hostname,
    peerCertificates,
  ) {
    console.log('SSL pinning bypassed for: ' + hostname)
    return // Do nothing, bypass check
  }
})

// Native library hooking
var native_func = Module.findExportByName('libnative.so', 'secret_function')

Interceptor.attach(native_func, {
  onEnter: function (args) {
    console.log('Native function called')

    // Dump arguments
    for (var i = 0; i < 4; i++) {
      console.log('  arg' + i + ': ' + args[i])
    }

    // Modify arguments
    args[0] = ptr(0x1234)
  },
  onLeave: function (retval) {
    console.log('Returned: ' + retval)

    // Modify return value
    retval.replace(1)
  },
})

// RPC server
rpc.exports = {
  decrypt: function (data) {
    // Call native decryption function
    var decrypt_func = new NativeFunction(
      Module.findExportByName('libcrypto.so', 'decrypt'),
      'pointer',
      ['pointer', 'int'],
    )

    var buffer = Memory.alloc(data.length)
    Memory.writeByteArray(buffer, data)

    var result = decrypt_func(buffer, data.length)
    return Memory.readByteArray(result, data.length)
  },
}

// Usage from Python:
// script.exports.decrypt([0x1, 0x2, 0x3, ...])
```

**Frida Python Integration:**

```python
import frida
import sys

def on_message(message, data):
    if message['type'] == 'send':
        print(f"[*] {message['payload']}")
    else:
        print(message)

# Attach to process
session = frida.attach("target_process")

# Load script
with open("hook.js") as f:
    script = session.create_script(f.read())

script.on('message', on_message)
script.load()

# Call RPC function
result = script.exports.decrypt([0x1, 0x2, 0x3, 0x4])
print(f"Decrypted: {result}")

# Keep script running
sys.stdin.read()
```

#### Week 31-32: Pin and DynamoRIO

**Intel Pin Tutorial:**

```cpp
// pintool.cpp - Simple instruction trace
#include "pin.h"
#include <iostream>
#include <fstream>

std::ofstream TraceFile;

// Called for every instruction
VOID RecordInstruction(VOID *ip) {
    TraceFile << ip << std::endl;
}

// Instrumentation callback
VOID Instruction(INS ins, VOID *v) {
    INS_InsertCall(ins, IPOINT_BEFORE, (AFUNPTR)RecordInstruction,
                   IARG_INST_PTR, IARG_END);
}

// Cleanup
VOID Fini(INT32 code, VOID *v) {
    TraceFile.close();
}

int main(int argc, char *argv[]) {
    PIN_Init(argc, argv);

    TraceFile.open("instruction_trace.txt");

    INS_AddInstrumentFunction(Instruction, 0);
    PIN_AddFiniFunction(Fini, 0);

    PIN_StartProgram();
    return 0;
}

// Compile:
// g++ -I$PIN_ROOT/source/include/pin -I$PIN_ROOT/extras/xed-intel64/include \
//     -L$PIN_ROOT/intel64/lib -L$PIN_ROOT/extras/xed-intel64/lib \
//     pintool.cpp -o pintool.so -shared -fPIC -lpin -lxed

// Run:
// pin -t pintool.so -- ./target_program
```

**Function Call Tracer:**

```cpp
// function_tracer.cpp
#include "pin.h"
#include <iostream>
#include <fstream>

std::ofstream TraceFile;
int call_depth = 0;

VOID BeforeCall(ADDRINT target, ADDRINT sp) {
    std::string indent(call_depth * 2, ' ');
    TraceFile << indent << "CALL 0x" << std::hex << target
              << " (SP: 0x" << sp << ")" << std::endl;
    call_depth++;
}

VOID AfterCall(ADDRINT ret) {
    call_depth--;
    std::string indent(call_depth * 2, ' ');
    TraceFile << indent << "RET 0x" << std::hex << ret << std::endl;
}

VOID Instruction(INS ins, VOID *v) {
    if (INS_IsCall(ins)) {
        INS_InsertCall(ins, IPOINT_BEFORE, (AFUNPTR)BeforeCall,
                      IARG_BRANCH_TARGET_ADDR,
                      IARG_REG_VALUE, REG_STACK_PTR,
                      IARG_END);
    }
    else if (INS_IsRet(ins)) {
        INS_InsertCall(ins, IPOINT_BEFORE, (AFUNPTR)AfterCall,
                      IARG_BRANCH_TARGET_ADDR,
                      IARG_END);
    }
}

VOID Fini(INT32 code, VOID *v) {
    TraceFile.close();
}

int main(int argc, char *argv[]) {
    PIN_Init(argc, argv);
    TraceFile.open("call_trace.txt");
    INS_AddInstrumentFunction(Instruction, 0);
    PIN_AddFiniFunction(Fini, 0);
    PIN_StartProgram();
    return 0;
}
```

**DynamoRIO Example:**

```c
// drcov.c - Code coverage tool
#include "dr_api.h"
#include "drmgr.h"

static file_t log_file;

static void event_exit(void) {
    dr_close_file(log_file);
    drmgr_exit();
}

static dr_emit_flags_t
event_bb_instrumentation(void *drcontext, void *tag, instrlist_t *bb,
                         instr_t *instr, bool for_trace,
                         bool translating, void *user_data)
{
    app_pc start_pc = dr_fragment_app_pc(tag);

    dr_fprintf(log_file, "BB: %p\n", start_pc);

    return DR_EMIT_DEFAULT;
}

DR_EXPORT void dr_client_main(client_id_t id, int argc, const char *argv[]) {
    log_file = dr_open_file("coverage.log", DR_FILE_WRITE_OVERWRITE);

    dr_register_exit_event(event_exit);
    drmgr_init();
    drmgr_register_bb_instrumentation_event(NULL,
        event_bb_instrumentation, NULL);
}

// Compile and run:
// drrun -c libdrcov.so -- ./target_program
```

### Month 9: Memory Analysis

#### Week 33-34: Heap Analysis and Memory Corruption

**Heap Structure (glibc malloc):**

```c
// Chunk structure
struct malloc_chunk {
    size_t prev_size;  // Size of previous chunk (if free)
    size_t size;       // Size of this chunk | flags (LSB: PREV_INUSE, IS_MMAPPED, NON_MAIN_ARENA)

    // Only for free chunks:
    struct malloc_chunk* fd;  // Forward pointer
    struct malloc_chunk* bk;  // Backward pointer

    // Only for large free chunks:
    struct malloc_chunk* fd_nextsize;
    struct malloc_chunk* bk_nextsize;
};

// Bins
// Fast bins: 10 singly-linked lists for small chunks (16-80 bytes)
// Small bins: 62 doubly-linked lists (< 512 bytes)
// Large bins: 63 doubly-linked lists (>= 512 bytes)
// Unsorted bin: 1 doubly-linked list (temporary storage)
```

**Heap Exploitation Techniques:**

```c
// 1. Use-After-Free (UAF)
char *ptr1 = malloc(100);
strcpy(ptr1, "Hello");
free(ptr1);
// ptr1 is now dangling pointer

char *ptr2 = malloc(100);  // May reuse same memory
// ptr1 still points to this memory!
strcpy(ptr1, "Exploited");  // Overwrites ptr2's data

// 2. Double Free
char *ptr = malloc(100);
free(ptr);
free(ptr);  // Corrupts heap metadata

// 3. Heap Overflow
char *buf = malloc(64);
strcpy(buf, very_long_string);  // Overflow into next chunk
// Can overwrite size field of next chunk

// 4. Unlink Exploit (older technique)
// Corrupt fd/bk pointers in free chunk
// When unlink() is called:
//   FD->bk = BK
//   BK->fd = FD
// Can achieve arbitrary write

// 5. Fastbin Dup
// Exploit fastbin singly-linked list
chunk_a = malloc(0x40);
chunk_b = malloc(0x40);
free(chunk_a);
free(chunk_b);
free(chunk_a);  // Double free in fastbin
// Now fastbin: chunk_a -> chunk_b -> chunk_a (circular)
```

**GDB Heap Commands (with pwndbg/gef):**

```bash
# pwndbg commands
heap                 # Show heap info
bins                 # Show bin layout
fastbins            # Show fastbins
tcache              # Show tcache (glibc 2.26+)
vis_heap_chunks     # Visualize heap chunks
malloc_chunk addr   # Parse chunk at address
arena               # Show arena info

# Manual heap inspection
x/40gx 0x603000     # Examine heap memory
print main_arena    # Print main arena structure

# Finding heap base
info proc mappings | grep heap
# Or
vmmap | grep heap

# Track allocations
catch syscall brk
catch syscall mmap
```

**Heap Analysis Script:**

```python
# heap_analyzer.py - GDB Python script
import gdb

class HeapAnalyzer(gdb.Command):
    def __init__(self):
        super().__init__("heap-analyze", gdb.COMMAND_DATA)

    def invoke(self, arg, from_tty):
        # Find heap start
        mappings = gdb.execute("info proc mappings", to_string=True)
        heap_start = None

        for line in mappings.split('\n'):
            if '[heap]' in line:
                heap_start = int(line.split()[0], 16)
                break

        if not heap_start:
            print("Heap not found!")
            return

        print(f"Heap starts at: {hex(heap_start)}")

        # Walk heap chunks
        current = heap_start
        chunk_count = 0

        while chunk_count < 100:  # Safety limit
            try:
                # Read chunk header
                prev_size = self.read_qword(current)
                size = self.read_qword(current + 8)

                actual_size = size & ~7  # Clear flags
                is_inuse = size & 1

                print(f"\nChunk {chunk_count}:")
                print(f"  Address: {hex(current)}")
                print(f"  Prev size: {hex(prev_size)}")
                print(f"  Size: {hex(actual_size)}")
                print(f"  In use: {bool(is_inuse)}")

                if not is_inuse:
                    # Free chunk - read fd/bk
                    fd = self.read_qword(current + 16)
                    bk = self.read_qword(current + 24)
                    print(f"  FD: {hex(fd)}")
                    print(f"  BK: {hex(bk)}")

                # Move to next chunk
                current += actual_size
                chunk_count += 1

                if actual_size == 0:
                    break

            except gdb.MemoryError:
                print("Reached end of heap or invalid memory")
                break

    def read_qword(self, addr):
        inferior = gdb.selected_inferior()
        data = inferior.read_memory(addr, 8)
        return int.from_bytes(data, byteorder='little')

HeapAnalyzer()
```

#### Week 35-36: Process Memory Dump and Analysis

**Creating Memory Dumps:**

```bash
# Linux
gcore <pid>                    # Create core dump
cat /proc/<pid>/maps           # View memory mappings
cat /proc/<pid>/maps | grep rw # Writable regions

# Dump specific region
gdb -p <pid>
(gdb) dump memory output.bin 0x7fff00000000 0x7fff00001000

# Windows
# Using Task Manager: Right-click process -> Create dump file

# Using procdump (Sysinternals)
procdump.exe -ma <pid> output.dmp

# Using WinDbg
.dump /ma C:\dumps\process.dmp
```

**Analyzing Memory Dumps:**

```python
# memory_analyzer.py
import struct
import re

class MemoryDump:
    def __init__(self, filename):
        with open(filename, 'rb') as f:
            self.data = f.read()

    def find_strings(self, min_length=4):
        """Find ASCII strings"""
        pattern = b'[\x20-\x7E]{' + str(min_length).encode() + b',}'
        strings = []

        for match in re.finditer(pattern, self.data):
            strings.append({
                'offset': match.start(),
                'string': match.group().decode('ascii')
            })

        return strings

    def find_unicode_strings(self, min_length=4):
        """Find Unicode strings"""
        strings = []
        i = 0
        current_string = []
        start_offset = 0

        while i < len(self.data) - 1:
            if (0x20 <= self.data[i] <= 0x7E and self.data[i+1] == 0):
                if not current_string:
                    start_offset = i
                current_string.append(chr(self.data[i]))
                i += 2
            else:
                if len(current_string) >= min_length:
                    strings.append({
                        'offset': start_offset,
                        'string': ''.join(current_string)
                    })
                current_string = []
                i += 1

        return strings

    def find_pattern(self, pattern):
        """Find byte pattern"""
        if isinstance(pattern, str):
            pattern = bytes.fromhex(pattern.replace(' ', ''))

        matches = []
        offset = 0

        while True:
            offset = self.data.find(pattern, offset)
            if offset == -1:
                break
            matches.append(offset)
            offset += 1

        return matches

    def find_pointers(self, base_addr=0):
        """Find potential pointers"""
        pointers = []

        for i in range(0, len(self.data) - 8, 8):
            value = struct.unpack('<Q', self.data[i:i+8])[0]

            # Check if looks like valid pointer
            if 0x400000 <= value <= 0x7fffffffffff:
                pointers.append({
                    'offset': i,
                    'address': base_addr + i,
                    'value': value
                })

        return pointers

    def search_for_keys(self):
        """Search for cryptographic keys"""
        keys = []

        # AES S-box
        aes_sbox = bytes([
            0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5
        ])

        matches = self.find_pattern(aes_sbox)
        for match in matches:
            keys.append({
                'type': 'AES S-box',
                'offset': match
            })

        # RSA private key markers
        rsa_markers = [
            b'-----BEGIN RSA PRIVATE KEY-----',
            b'-----BEGIN PRIVATE KEY-----',
            b'\x30\x82'  # DER format
        ]

        for marker in rsa_markers:
            matches = self.find_pattern(marker)
            for match in matches:
                keys.append({
                    'type': 'RSA Private Key',
                    'offset': match
                })

        return keys

    def extract_region(self, offset, size):
        """Extract memory region"""
        return self.data[offset:offset+size]

# Usage
dump = MemoryDump("process.dmp")

print("=== ASCII Strings ===")
for s in dump.find_strings(min_length=8)[:20]:
    print(f"{hex(s['offset'])}: {s['string']}")

print("\n=== Potential Keys ===")
for key in dump.search_for_keys():
    print(f"{key['type']} at {hex(key['offset'])}")
```

**Volatility Framework (Memory Forensics):**

```bash
# Installation
pip install volatility3

# Identify OS profile
vol.py -f memory.dmp imageinfo

# List processes
vol.py -f memory.dmp --profile=Win7SP1x64 pslist
vol.py -f memory.dmp --profile=Win7SP1x64 pstree

# Dump process
vol.py -f memory.dmp --profile=Win7SP1x64 procdump -p 1234 -D ./

# Network connections
vol.py -f memory.dmp --profile=Win7SP1x64 netscan

# Command line arguments
vol.py -f memory.dmp --profile=Win7SP1x64 cmdline

# DLL list
vol.py -f memory.dmp --profile=Win7SP1x64 dlllist -p 1234

# Malware detection
vol.py -f memory.dmp --profile=Win7SP1x64 malfind
vol.py -f memory.dmp --profile=Win7SP1x64 apihooks

# Registry
vol.py -f memory.dmp --profile=Win7SP1x64 hivelist
vol.py -f memory.dmp --profile=Win7SP1x64 printkey -K "Software\Microsoft\Windows\CurrentVersion\Run"

# File extraction
vol.py -f memory.dmp --profile=Win7SP1x64 filescan
vol.py -f memory.dmp --profile=Win7SP1x64 dumpfiles -Q 0x000000007e8a8f20 -D ./

# Timeline
vol.py -f memory.dmp --profile=Win7SP1x64 timeliner
```

---

## Phase 4: Malware Analysis (Months 10-12)

### Month 10: Malware Fundamentals

#### Week 37-38: Setting Up Malware Lab

**Isolated Lab Environment:**

```yaml
# Lab Architecture
┌─────────────────────────────────────┐
│       Host Machine (Real OS)        │
│                                     │
│  ┌───────────────────────────────┐ │
│  │   VMware / VirtualBox         │ │
│  │                               │ │
│  │  ┌─────────────────────────┐ │ │
│  │  │  Windows VM (Victim)    │ │ │
│  │  │  - INetSim for DNS/HTTP │ │ │
│  │  │  - Process Monitor      │ │ │
│  │  │  - Regshot              │ │ │
│  │  │  - Wireshark            │ │ │
│  │  └─────────────────────────┘ │ │
│  │                               │ │
│  │  ┌─────────────────────────┐ │ │
│  │  │  Linux VM (Analysis)    │ │ │
│  │  │  - REMnux distro        │ │ │
│  │  │  - FLARE VM tools       │ │ │
│  │  └─────────────────────────┘ │ │
│  │                               │ │
│  │  Network: Host-Only (Isolated)│ │
│  └───────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Essential Tools Installation:**

```bash
# Windows Analysis VM (FLARE VM)
# Install using Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# Install FLARE VM
# Download from: https://github.com/mandiant/flare-vm
.\install.ps1

# Includes:
# - IDA Free, Ghidra, x64dbg
# - PE-bear, CFF Explorer, PEview
# - Process Monitor, Process Explorer
# - Wireshark, Fiddler
# - YARA, PEiD, Detect It Easy
# - And 100+ more tools

# Linux Analysis VM (REMnux)
# Download from: https://remnux.org/
# Or install on existing Ubuntu:
wget https://REMnux.org/remnux-cli
sudo mv remnux-cli /usr/local/bin
sudo chmod +x /usr/local/bin/remnux-cli
sudo remnux-cli install

# Includes:
# - radare2, Ghidra, Binary Ninja
# - Volatility, bulk_extractor
# - NetworkMiner, tcpdump
# - YARA, ClamAV
# - PDF analysis tools
# - JavaScript deobfuscators
```

**Network Simulation (INetSim):**

```bash
# Install INetSim (on Linux)
sudo apt-get install inetsim

# Configuration (/etc/inetsim/inetsim.conf)
service_bind_address    0.0.0.0
dns_default_ip         192.168.56.1
https_bind_port        443
http_bind_port         80
smtp_bind_port         25
pop3_bind_port         110

# Start INetSim
sudo inetsim

# Configure Windows VM to use INetSim as DNS
# Set DNS server to Linux VM IP (192.168.56.1)
```

**Safe Malware Handling:**

```bash
# Always work with snapshots
# Take snapshot before analysis
# Revert after each sample

# Use password-protected archives
# Password: infected
7z a -pinfected sample.7z malware.exe

# Rename to .bin or .txt during transfer
mv malware.exe malware.bin

# Never connect to real internet
# Verify network isolation before running sample
```

#### Week 39-40: Basic Static Malware Analysis

**Initial Triage:**

```bash
# 1. Calculate hashes
md5sum sample.exe
sha1sum sample.exe
sha256sum sample.exe

# 2. Check VirusTotal (without uploading!)
# Use hash search: https://www.virustotal.com/

# 3. File type identification
file sample.exe
exiftool sample.exe

# 4. Strings analysis
strings sample.exe > strings.txt
strings -el sample.exe > strings_unicode.txt  # Unicode

# Look for:
# - URLs, IPs
# - Registry keys
# - File paths
# - API names
# - Error messages
# - Suspicious strings

# 5. Packer detection
# Using PEiD or Detect It Easy
die sample.exe

# Using YARA
yara -r packer_rules.yar sample.exe

# Common packers:
# - UPX, MPRESS, Themida, VMProtect
# - Armadillo, ASPack, PECompact
```

**PE Analysis for Malware:**

```python
import pefile
import hashlib

def analyze_malware_pe(filename):
    pe = pefile.PE(filename)

    print("=== BASIC INFO ===")
    print(f"Compilation Timestamp: {pe.FILE_HEADER.TimeDateStamp}")
    print(f"Entry Point: {hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint)}")
    print(f"Image Base: {hex(pe.OPTIONAL_HEADER.ImageBase)}")

    # Check for suspicious characteristics
    print("\n=== SUSPICIOUS INDICATORS ===")

    # 1. Check sections
    for section in pe.sections:
        name = section.Name.decode().strip('\x00')
        entropy = section.get_entropy()

        print(f"\nSection: {name}")
        print(f"  Entropy: {entropy:.2f}")
        print(f"  Virtual Size: {hex(section.Misc_VirtualSize)}")
        print(f"  Raw Size: {hex(section.SizeOfRawData)}")

        # High entropy = packed/encrypted
        if entropy > 7.0:
            print(f"  [!] HIGH ENTROPY - Possibly packed!")

        # Writable and executable = suspicious
        if (section.Characteristics & 0x20000000 and
            section.Characteristics & 0x80000000):
            print(f"  [!] Writable AND Executable!")

        # Size mismatch = possible packing
        if section.SizeOfRawData == 0 and section.Misc_VirtualSize > 0:
            print(f"  [!] Size mismatch - Virtual UPX section")

    # 2. Import Analysis
    print("\n=== SUSPICIOUS IMPORTS ===")
    suspicious_apis = [
        'CreateRemoteThread', 'WriteProcessMemory', 'VirtualAllocEx',
        'SetWindowsHookEx', 'GetAsyncKeyState', 'CreateToolhelp32Snapshot',
        'URLDownloadToFile', 'WinExec', 'ShellExecute', 'RegSetValue',
        'CreateProcess', 'CreateService', 'StartService'
    ]

    for entry in pe.DIRECTORY_ENTRY_IMPORT:
        for imp in entry.imports:
            if imp.name and imp.name.decode() in suspicious_apis:
                print(f"  {entry.dll.decode()}: {imp.name.decode()}")

    # 3. Check for TLS callbacks (anti-debugging)
    if hasattr(pe, 'DIRECTORY_ENTRY_TLS'):
        print("\n=== TLS CALLBACKS ===")
        print("[!] TLS callbacks detected (possible anti-debug)")
        callbacks = pe.DIRECTORY_ENTRY_TLS.struct.AddressOfCallBacks
        print(f"Callback address: {hex(callbacks)}")

    # 4. Resources
    if hasattr(pe, 'DIRECTORY_ENTRY_RESOURCE'):
        print("\n=== RESOURCES ===")
        for resource_type in pe.DIRECTORY_ENTRY_RESOURCE.entries:
            if resource_type.name is not None:
                name = str(resource_type.name)
            else:
                name = str(pefile.RESOURCE_TYPE.get(resource_type.struct.Id))

            for resource_id in resource_type.directory.entries:
                for resource_lang in resource_id.directory.entries:
                    data = pe.get_data(resource_lang.data.struct.OffsetToData,
                                     resource_lang.data.struct.Size)

                    entropy = calculate_entropy(data)
                    if entropy > 7.0:
                        print(f"  {name}: HIGH ENTROPY {entropy:.2f} - Possible payload")

    # 5. Overlay data
    overlay_offset = pe.get_overlay_data_start_offset()
    if overlay_offset:
        print("\n=== OVERLAY DATA ===")
        print(f"[!] Overlay detected at offset {hex(overlay_offset)}")

        with open(filename, 'rb') as f:
            f.seek(overlay_offset)
            overlay = f.read()
            entropy = calculate_entropy(overlay)
            print(f"Overlay size: {len(overlay)} bytes")
            print(f"Overlay entropy: {entropy:.2f}")

def calculate_entropy(data):
    """Calculate Shannon entropy"""
    import math
    from collections import Counter

    if not data:
        return 0

    counter = Counter(data)
    length = len(data)
    entropy = 0

    for count in counter.values():
        p = count / length
        entropy -= p * math.log2(p)

    return entropy

# Usage
analyze_malware_pe("suspicious.exe")
```

**YARA Rules for Detection:**

```yara
// malware_rules.yar

rule Suspicious_High_Entropy {
    meta:
        description = "Detects files with high entropy (packed/encrypted)"
        author = "Reverse Engineer"

    condition:
        math.entropy(0, filesize) > 7.0
}

rule Ransomware_Indicators {
    meta:
        description = "Common ransomware indicators"

    strings:
        $crypto1 = "CryptEncrypt" nocase
        $crypto2 = "CryptGenKey" nocase
        $file1 = "FindFirstFile" nocase
        $file2 = "FindNextFile" nocase
        $ext1 = ".encrypted"
        $ext2 = ".locked"
        $msg1 = "bitcoin" nocase
        $msg2 = "your files have been encrypted" nocase

    condition:
        2 of ($crypto*) and 2 of ($file*) and (1 of ($ext*) or 1 of ($msg*))
}

rule Keylogger_Indicators {
    meta:
        description = "Detects potential keyloggers"

    strings:
        $api1 = "GetAsyncKeyState"
        $api2 = "GetForegroundWindow"
        $api3 = "GetWindowText"
        $api4 = "SetWindowsHookEx"

    condition:
        ($api1 or $api4) and ($api2 or $api3)
}

rule Process_Injection {
    meta:
        description = "Detects process injection techniques"

    strings:
        $api1 = "CreateRemoteThread"
        $api2 = "WriteProcessMemory"
        $api3 = "VirtualAllocEx"
        $api4 = "OpenProcess"
        $api5 = "QueueUserAPC"
        $api6 = "SetThreadContext"

    condition:
        ($api1 or $api5 or $api6) and ($api2 or $api3) and $api4
}

rule Downloader_Trojan {
    meta:
        description = "Detects downloader functionality"

    strings:
        $url1 = "http://" nocase
        $url2 = "https://" nocase
        $api1 = "URLDownloadToFile"
        $api2 = "InternetOpen"
        $api3 = "InternetReadFile"
        $api4 = "HttpSendRequest"
        $exec1 = "WinExec"
        $exec2 = "ShellExecute"
        $exec3 = "CreateProcess"

    condition:
        (1 of ($url*)) and (2 of ($api*)) and (1 of ($exec*))
}

// Run YARA
// yara malware_rules.yar suspicious.exe
```

### Month 11: Advanced Malware Techniques

#### Week 41-42: Unpacking and Deobfuscation

**Manual Unpacking Process:**

```
1. Identify packer (PEiD, Detect It Easy)
2. Find OEP (Original Entry Point)
   - Look for PUSHAD at start
   - Find corresponding POPAD
   - Set breakpoint after POPAD
3. Dump unpacked code
4. Rebuild Import Table
5. Fix PE header
```

**Unpacking with x64dbg:**

```
# Generic unpacking steps

1. Load packed executable in x64dbg

2. Set breakpoints on common unpacking points:
   - VirtualAlloc / VirtualProtect
   - WriteProcessMemory (for process injection)
   - After POPAD instruction

3. Run and let unpacker do its work

4. When code is unpacked in memory:
   - Use Scylla plugin to dump
   - Scylla -> IAT Autosearch
   - Scylla -> Get Imports
   - Scylla -> Dump
   - Scylla -> Fix Dump

5. Analyze dumped file
```

**OEP Finding Techniques:**

```nasm
; Technique 1: PUSHAD/POPAD
; Most packers save registers at start
PUSHAD          ; Save all registers
; ... unpacking code ...
POPAD           ; Restore registers
JMP original_ep ; Jump to original code
; Set breakpoint after POPAD

; Technique 2: Stack manipulation
; Look for stack setup
PUSH EBP
MOV EBP, ESP
; This is likely start of real code

; Technique 3: ESP tricks
; Hardware breakpoint on ESP access
; When ESP changes dramatically, likely at OEP

; Technique 4: Memory page execution
; Set breakpoint on execution of newly allocated pages
```

**Automated Unpacking:**

```python
# unpack.py - Automated unpacker skeleton
import pefile
import struct

class Unpacker:
    def __init__(self, filename):
        self.filename = filename
        self.pe = pefile.PE(filename)
        with open(filename, 'rb') as f:
            self.data = bytearray(f.read())

    def identify_packer(self):
        """Identify packer type"""
        entry_point = self.pe.OPTIONAL_HEADER.AddressOfEntryPoint
        ep_offset = self.pe.get_offset_from_rva(entry_point)

        # Check for UPX signature
        if self.data[ep_offset:ep_offset+4] == b'UPX!':
            return 'UPX'

        # Check for common packer patterns
        first_bytes = self.data[ep_offset:ep_offset+16]

        # MPRESS: 68 xx xx xx xx E8 00 00 00 00
        if first_bytes[0] == 0x68 and first_bytes[5] == 0xE8:
            return 'MPRESS'

        return 'Unknown'

    def unpack_upx(self):
        """Unpack UPX"""
        import subprocess
        subprocess.run(['upx', '-d', self.filename])

    def dump_memory_region(self, start_rva, size):
        """Dump memory region"""
        offset = self.pe.get_offset_from_rva(start_rva)
        return self.data[offset:offset+size]

    def rebuild_imports(self, iat_rva, iat_size):
        """Rebuild import table"""
        # This is complex and packer-specific
        # Use tools like Scylla or manual analysis
        pass

    def fix_pe_header(self, oep_rva):
        """Fix PE header with new OEP"""
        self.pe.OPTIONAL_HEADER.AddressOfEntryPoint = oep_rva

    def save_unpacked(self, output_file):
        """Save unpacked PE"""
        self.pe.write(output_file)

# Usage
unpacker = Unpacker('packed.exe')
packer = unpacker.identify_packer()
print(f"Detected packer: {packer}")

if packer == 'UPX':
    unpacker.unpack_upx()
```

**Dealing with Code Virtualization:**

```
# VM-based obfuscation (Themida, VMProtect, Code Virtualizer)

Characteristics:
- Converts x86 to custom bytecode
- Interprets bytecode at runtime
- Very difficult to reverse

Approach:
1. Don't try to devirtualize completely
2. Focus on I/O points:
   - API calls
   - Network communication
   - File operations
3. Use dynamic analysis
4. Hook interesting functions
5. Extract algorithms via black-box testing

Tools:
- VMAttack (partial devirtualization)
- Dynamic tracing with Pin/DynamoRIO
```

#### Week 43-44: Anti-Analysis Techniques

**Anti-Debugging Techniques:**

```c
// Comprehensive anti-debugging

// 1. IsDebuggerPresent
if (IsDebuggerPresent()) {
    exit(0);
}

// 2. PEB check
PPEB pPeb = (PPEB)__readgsqword(0x60);  // x64
if (pPeb->BeingDebugged) {
    exit(0);
}

// 3. NtGlobalFlag
if (pPeb->NtGlobalFlag & 0x70) {  // FLG_HEAP_*
    exit(0);
}

// 4. Heap flags
PVOID heap = GetProcessHeap();
DWORD flags = *(PDWORD)((PBYTE)heap + 0x70);  // x64
if (flags & 0x50000062) {  // HEAP_GROWABLE | ...
    exit(0);
}

// 5. NtQueryInformationProcess
HANDLE hProcess = GetCurrentProcess();
DWORD debugPort = 0;
NtQueryInformationProcess(hProcess, ProcessDebugPort,
                         &debugPort, sizeof(debugPort), NULL);
if (debugPort) {
    exit(0);
}

// 6. Hardware breakpoint detection
CONTEXT ctx = {0};
ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;
GetThreadContext(GetCurrentThread(), &ctx);
if (ctx.Dr0 || ctx.Dr1 || ctx.Dr2 || ctx.Dr3) {
    exit(0);
}

// 7. Software breakpoint detection (INT 3 = 0xCC)
BYTE *code = (BYTE*)&some_function;
if (*code == 0xCC) {
    exit(0);
}

// 8. Timing checks
DWORD start = GetTickCount();
// ... some code ...
DWORD end = GetTickCount();
if ((end - start) > 100) {  // Too slow = debugger
    exit(0);
}

// 9. RDTSC timing
unsigned long long tsc1, tsc2;
tsc1 = __rdtsc();
// ... some code ...
tsc2 = __rdtsc();
if ((tsc2 - tsc1) > 10000) {
    exit(0);
}

// 10. Parent process check
DWORD ppid = GetParentProcessId();
char parentName[MAX_PATH];
GetProcessName(ppid, parentName);
if (strstr(parentName, "ollydbg") ||
    strstr(parentName, "x64dbg") ||
    strstr(parentName, "windbg")) {
    exit(0);
}

// 11. Window detection
if (FindWindow("OLLYDBG", NULL) ||
    FindWindow("WinDbgFrameClass", NULL) ||
    FindWindow("Qt5QWindowIcon", NULL)) {  // x64dbg
    exit(0);
}

// 12. Exception tricks
__try {
    // Trigger exception
    RaiseException(EXCEPTION_BREAKPOINT, 0, 0, NULL);
    // If we reach here, debugger handled it
    exit(0);
}
__except (EXCEPTION_EXECUTE_HANDLER) {
    // Normal execution
}

// 13. Thread hiding
typedef NTSTATUS (WINAPI *pNtSetInformationThread)(
    HANDLE, UINT, PVOID, ULONG);

pNtSetInformationThread NtSIT = (pNtSetInformationThread)
    GetProcAddress(GetModuleHandle("ntdll.dll"),
                  "NtSetInformationThread");

if (NtSIT) {
    NtSIT(GetCurrentThread(), 0x11, 0, 0);  // ThreadHideFromDebugger
}

// 14. TLS callbacks (execute before main)
void NTAPI TlsCallback(PVOID DllHandle, DWORD Reason, PVOID Reserved) {
    if (IsDebuggerPresent()) {
        exit(0);
    }
}

#pragma comment (linker, "/INCLUDE:_tls_used")
#pragma comment (linker, "/INCLUDE:_tls_callback")
#pragma data_seg(".CRT$XLB")
PIMAGE_TLS_CALLBACK _tls_callback = TlsCallback;
#pragma data_seg()
```

**Bypassing Anti-Debug:**

```python
# Frida script to bypass common anti-debug

// Bypass IsDebuggerPresent
Interceptor.replace(Module.findExportByName("kernel32.dll", "IsDebuggerPresent"),
    new NativeCallback(function() {
        return 0;
    }, 'int', [])
);

// Bypass NtQueryInformationProcess
var NtQIP = Module.findExportByName("ntdll.dll", "NtQueryInformationProcess");
Interceptor.replace(NtQIP,
    new NativeCallback(function(handle, infoClass, buffer, length, returnLength) {
        var result = this.original(handle, infoClass, buffer, length, returnLength);

        if (infoClass == 7) {  // ProcessDebugPort
            Memory.writeU32(buffer, 0);
        }
        else if (infoClass == 30) {  // ProcessDebugObjectHandle
            Memory.writeU32(buffer, 0);
        }

        return result;
    }, 'uint32', ['pointer', 'uint32', 'pointer', 'uint32', 'pointer'])
);

// Patch PEB BeingDebugged
var peb = Process.findModuleByName(null).base.add(0x60);  // GS:60 points to PEB
var beingDebugged = peb.add(0x02);
Memory.writeU8(beingDebugged, 0);

// Patch NtGlobalFlag
var ntGlobalFlag = peb.add(0xBC);  // x64
Memory.writeU32(ntGlobalFlag, 0);

// Clear hardware breakpoints
var ctx = Memory.alloc(512);
// GetThreadContext / SetThreadContext with cleared DR0-DR3
```

**Anti-VM Detection:**

```c
// VM detection techniques

// 1. CPUID checks
void cpuid(int info[4], int function_id) {
    __cpuid(info, function_id);
}

bool CheckVM_CPUID() {
    int cpuInfo[4];
    cpuid(cpuInfo, 0x40000000);

    // Check hypervisor vendor
    char vendor[13];
    memcpy(vendor + 0, &cpuInfo[1], 4);
    memcpy(vendor + 4, &cpuInfo[2], 4);
    memcpy(vendor + 8, &cpuInfo[3], 4);
    vendor[12] = '\0';

    if (strcmp(vendor, "VMwareVMware") == 0 ||
        strcmp(vendor, "Microsoft Hv") == 0 ||
        strcmp(vendor, "KVMKVMKVM") == 0) {
        return true;
    }
    return false;
}

// 2. Registry keys
bool CheckVM_Registry() {
    HKEY hKey;
    if (RegOpenKeyEx(HKEY_LOCAL_MACHINE,
        "SYSTEM\\CurrentControlSet\\Services\\Disk\\Enum",
        0, KEY_READ, &hKey) == ERROR_SUCCESS) {

        char data[1024];
        DWORD size = sizeof(data);
        RegQueryValueEx(hKey, "0", NULL, NULL, (LPBYTE)data, &size);

        if (strstr(data, "VMware") ||
            strstr(data, "VBOX") ||
            strstr(data, "QEMU")) {
            RegCloseKey(hKey);
            return true;
        }
        RegCloseKey(hKey);
    }
    return false;
}

// 3. Process/Service checks
bool CheckVM_Processes() {
    const char *vmProcesses[] = {
        "vmtoolsd.exe", "vmware.exe", "vboxservice.exe",
        "vboxtray.exe", "vmacthlp.exe", NULL
    };

    HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe = {sizeof(PROCESSENTRY32)};

    if (Process32First(snapshot, &pe)) {
        do {
            for (int i = 0; vmProcesses[i]; i++) {
                if (_stricmp(pe.szExeFile, vmProcesses[i]) == 0) {
                    CloseHandle(snapshot);
                    return true;
                }
            }
        } while (Process32Next(snapshot, &pe));
    }

    CloseHandle(snapshot);
    return false;
}

// 4. MAC address check
bool CheckVM_MAC() {
    // VMware MACs start with: 00:05:69, 00:0C:29, 00:1C:14, 00:50:56
    // VirtualBox: 08:00:27
    // Check network adapters for these prefixes
    return false;  // Simplified
}

// 5. SMBIOS checks
bool CheckVM_SMBIOS() {
    BYTE *firmware = (BYTE*)0xF0000;  // BIOS area

    for (int i = 0; i < 0x10000; i++) {
        if (memcmp(firmware + i, "VMware", 6) == 0 ||
            memcmp(firmware + i, "VirtualBox", 10) == 0 ||
            memcmp(firmware + i, "QEMU", 4) == 0) {
            return true;
        }
    }
    return false;
}

// 6. Timing attacks
bool CheckVM_Timing() {
    unsigned long long tsc1, tsc2;

    // RDTSC is slower in VMs
    tsc1 = __rdtsc();
    Sleep(500);
    tsc2 = __rdtsc();

    unsigned long long diff = tsc2 - tsc1;
    // In VM, diff will be much higher than expected

    return (diff > THRESHOLD);
}
```

### Month 12: Malware Behavior Analysis

#### Week 45-46: Dynamic Behavior Monitoring

**Automated Sandbox Analysis:**

```bash
# Cuckoo Sandbox setup
pip install cuckoo

# Initialize
cuckoo init
```
