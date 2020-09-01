# Linux Condensed

## [Memory Acquisition](linuxMemoryAcquisition.md)

### Of Old

- Used `/dev/mem`, `/dev/kmem`, and `$ptrace`. Unfortunately, `/dev/mem` (and therefore ptrace) only got the first 896MB; while `/dev/kmem` allowed users direct access to kernel mem- security! 
- Both `/dev/*` are disabled on modern systems.

### Modern Techniques

- `/proc/kcore` device was introduced which was used by $fmem. It will export the kernel's virtual address space to userland as an ELF file. As with `/dev/mem` on 32-bit systems you are limited to the first 896MB of memory. Though on 64-bit systems, the kernel keeps a static virtual map of all RAM (found in `Documentation/x86/x86_64/mm.txt`). A PoC tool is in vol.py that parses the ELF and compares with `/proc/iomem`. However, `/proc/kcore` can be disabled (like in Gentoo) and malware can hook the read function and filter out data received by the userland tool.
- fmem works by loading a kernel driver to create it's own interface (practically recreating `/dev/mem`). It also checks if pages are in memory before accessing. The 896MB limit is not a thing anymore. 
  - The only down side is `/proc/iomem` must be checked to figure out where the RAM is mapped. Can see this by running `$cat /proc/iomem/` or `$grep "System RAM" /proc/iomem` to see non-contiguity.
- LiME - works by loading a kernel driver and streamlining the acquisition by doing everything within the kernel and just outputs a file (usually in "lime" format, however can do raw or "padded").
  - `$sudo insmod lime.ko "path=/mnt/externaldrive/memdmp.lime format=lime"`
  - Network-based: (On victim machine) `$sudo insmod lime.ko "path=tcp:4444 format=lime"` then (On forensic workstation) `$nc target_machine_ip > memdmp.lime`



## [ELF File](linuxOS.md)

- Can use the `$readelf` tool to analyze Executable and Linking Format files.
- File comprises of Header, Section Headers, and Program Headers

### Header

Can use `7fELF` for signature scans

- *Elf32_Ehdr* or *Elf64_Ehdr* data stuctures
  - *e_entry* is the entry point of the program
  - section pointers
  - program header pointers
  - *e_shstrndx* stores the index within the section header table of the strings that map to section names

### Sections

ELFs are divided into sections and *sh_name* holds an index into the string table of section names.

- Common ELF Sections: .text, .data, .rdata, .bss, .bss, .got
- When an ELF is packed/obfuscated (think malware) it's headers are removed [as they are optional and not used at runtime]. Instead, **program header's are what is used during runtime.** 
  - Check out UPX packer.



- Use `$readelf -S <file>`

### Program Headers

Used to map the file and its sections into memory *at runtime*. In the ELF header is the *e_phoff* member which points to an *Elf[32,64]_Phdr* struct. Within this struct exists most importantly the *p_type* member which describes whether that segment (portions of the file that load into memory) is *PT_LOAD*, *PT_DYNAMIC* (includes dynamic linking info), *PT_INTERP* (holds path to the program interpreter).

- **Program headers will remain even when a binary is stripped; section headers do not.**

- Use `$readelf -l <file>`

### Shared Libraries

aka Shared Objects (`*.so`) are reusable pieces of code usually stored on disk.

- User `$readelf -d <file> | grep NEEDED` or just `$ldd <file>`

### Global Offset Table (GOT)

This stores runtime address symbols (found in `*.so`'s). By analyzing this, you can learn the addresses of symbols within a process.