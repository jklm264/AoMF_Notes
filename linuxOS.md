# Linux Operating System

## ELF Files

Executable and Linking Format is the main *E* file format in *Nix systems. [Documentation](http://www.skyfree.org/linux/references/ELF_Format.pdf)

- `readelf` is apart of *binutils* and can help analyze an ELF file.

### Header

- Magic: `7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00` (16B)
	- Can use `7fELF` for signature scans
- *Elf32_Ehdr* or *Elf64_Ehdr* data stuctures
	- *e_entry* is the entry point of the program!
	- program headers 
	- section headers
	- *e_shstrndx* stores the index within the section header table of the strings that map to section names

### Sections

- ELFs are divided into sections and *sh_name* holds an index into the string table of section names.
- Can use `$readelf -S <file>`
- Common ELF Sections: .text, .data, .rdata, .bss, .bss, .got

- When an ELF is packed/obfuscated (think malware) it's headers are removed [as they are optional and not used at runtime]. Instead, **program header's are what is used during runtime.**
	- *UPX packer* can do this

### Program Headers

Used to map the file and its sections into memory *at runtime*. In the ELF header is the *e_phoff* member which points to an *Elf[32,64]_Phdr* struct. Within this struct exists most importantly the *p_type* member which describes whether that segment (portions of the file that load into memory) is *PT_LOAD*, *PT_DYNAMIC* (includes dynamic linking info), *PT_INTERP* (holds path to the program interpreter).

​	To list program headers: `$readelf -l <file>`



When packing a program though it becomes static! The dynamic portions (like loading libraries) are now compiled into the app. Running `$file <file>` will show *statically linked, stripped*. However, this is for simple/understood packers like UPX. For more complicated ones RE professionals first have to understand the packer, unpack the binary, then start to do the actual RE. Earlier in AoMF, the method of unpacking a binary was as simple as getting the DLL loaded to keep it in memory long enough to get a capture (the example goes `C:> rundll32 some.dll,FakeExport` did the trick [see page 248]).

#### Shared Library Loading

Def: Reusable pieces of code usually stored on disk as `*.so` (shared object); think Windows DLLs.

<200b> To find *.so*'s: `$readelf -d <file> | grep NEEDED` or `$ldd <file>` which **lists the full paths** (*linux-gate* is a virtual shared object and not on disk. *ld-linux* is the loader library stored in the *INTERP* header)!


#### Global Offset Table (GOT)

Def: stores runtime address for external symbols (usually found within *.so*'s). The names of these symbols (and their full addresses) will always be present because they can be loaded anywhere within the process' address space. Akin to a Windows PE's Import Address Table (IAT).

> "Analyzing GOT entries in memory dumps allows you to determine the addresses of symbols within a process. This analysis often lends insight into how a system was compromised as well as how malicious code altered the runtime state."

- GOT will only be in a mostly static location for non-PIE (position independent executable) apps.
  - Don't confuse this with PIC (specific for shared libraries). [More reading at "c) How to…"](https://codywu2010.wordpress.com/2014/11/29/about-elf-pie-pic-and-else/)
- Malware will usually target the GOT to find addresses of API functions (e.g. `system()`, `strcpy()`, `memcpy()`, `setuid()`, `setgid()`). It will also try and overwrite entries ([see poly.edu](https://isisblogs.poly.edu/2011/06/01/relro-relocation-read-only/)).



- Use `$readelf -r <ELF>`



#### Procedural Linkage Table (PLT)
-  supports calling functions within shared libraries
-  Great info at [systemoverlord.com](https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html)
-  Even better resource [here](https://www.airs.com/blog/archives/41).



## Linux Data Structures

### Lists

Doubly linkned lists and hash tables

* *list_head* is structure for storing lists.
* *list_add_rcu* is a race-condition safe version of *list_add*.
* *list_for_each_entry_rcu()* is the basis for `$vol.py linux_lsmod` which simply walks the beginning of the list which contains the most recent kernel modules loaded.



### Hash Tables

Have two structures: *hlist_head* and *hlist_node*

`$vol.py linux_mount` walks a hash table via the *hlist_for_each_entry_rcu(*)*



### Trees

1. Radix Trees - usually used in recovering file systems from memory
2. Red-Black Trees - self-balancing binary search trees
   1. Ex: When a page fault occurs the kernel must quickly determine whether the faulting address resides within a mapped region, and find which one.
   2. Can be found in `include/linux/rbtree.h`

### The Easy Way

The *container_of* macro allows simple retrieval of structure members within embedded structures.

- Example in book was of the *SOCKET_I* macro to find member data of a socket
- `$vol.py linux_netstat` works this way



## Linux Address Translation

### Kernel Identity Paging

**Identity Maps** the kernel code and data.

> **Identity paging** is when virtual addresses are the same as their corresponding physical offset **OR** when virtual addresses are at a constant offset from their physical offset

- This is useful, for example, when trying to find that data associated with the initial Linux process ("*swapper*") in physical memory. The process is to first find *task_struct.comm* offset value (516) and also find the address for the *System.map* location (0xc13defe0). Next, `(0xc0000000 - 0xc13defe0) + hex(516) = hex(20836836) `. We `$dd skip=20836836 | xxd` which will output "swapper".

- This is how Valatility quickly finds the Kernel's directory table table (DTB) value.

### Finding the Kernel DTB

- Can find this by finding the initial DTB address (called *swapper_pg_dir*) which is stored in the *System.map* file and within the identity-mapped region of the kernel.
- Volatility gets this via a function `get_symbol("swapper_pg_dir") - shift` where shift is the 32-bit virtual address shift constant `0xc0000000`. On 64-bit systems the only difference is the shift value (now `0xffffffff80000000`) and `get_symbol("init_level4_pgt")`

### Validating Address Space

There is a validity test in order for vol.py to work on a given memory sample.

1. vol.py get the address of *init_task* from the profile.
2. Then translates this virtual address to its corresponding physical offset using *vtop()* 
3. Check the returned physical offset value to the address of subtracting the architecture's identity-mapping shift from the virtual address of *init_task*.

In code:

```python
init_task_addr = self.obj_vm.profile.get_sybol('init_task')
  
if self.obj_vm.profile.metadata.get('memory_model', '32bit') == "32bit":
  shift = 0xc0000000
else:
  shift = 0xffffffff80000000
    
yeild self.obj_vm.vtop(init_task_addr) == init_task_addr - shift
```



## procfs and sysfs

These are virtual file systems that expose kernel data to userland. They also allow userland to communicate back to kernel mode components.

### procfs

- `$vol.py lsmod` will parse `/proc/modules` file
- `$vol.py mount` will print the list of mounted file systems from `/proc/mounts`
- `$vol.py netstat` will print protocol-specific files under `/proc/net/`; e.g. `proc/net/tcp`
- Also under `/proc` are per-process subdirectories according to PID value. Tools like `$ps` use this info.
- This can be easily altered for malicious purposes. Ex: `/proc/sys/kernel/randomize_va_space` set to `0` causes ASLR to be turned off. Rootkits can hook, filter, and add their own files here.

## Compressed Swap

On both Linux and Mac, they use this functionality to avoid using disk-based swap storage. This process actually involves the OS to "compress swap" pages and then store them within a reserved pool of physical memory. This speeds up the system since going to disk is expensive as well as faster in-memory (de)compression routines are now available. More info on this performance increase [here](https://events.static.linuxfound.org/sites/events/files/slides/tmc_sjennings_linuxcon2013.pdf).



This complicates forensics investigations since the info stored in the swap is no longer on disk (just used `$strings`). However, Richard's and **Andrew Case** patched Volatility with their solution [detailed here](https://www.dfrws.org/sites/default/files/session-files/pres-in_lieu_of_swap_-_analyzing_compressed_ram_in_mac_os_x_and_linux.pdf).























































































