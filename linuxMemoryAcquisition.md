# Linux Memory Acquisition

### Historical Acquisition

Interfaces built into OS allowed for RW of physical memory by privileged apps. (Ex: `/dev/mem`). These were vulnerability and were disabled. This also took away forensic investigators from using them too.

- `/dev/mem` - used to acquire PHY memory and RAM could be accessed by programs via root (e.g. `$dd`). Don't forget about memory acqusition as it relates to non-contigouos memory that accesses sensitive regions.
	- For reason's not explained in the book, `/dev/mem` only addresses the first 896 MB of RAM.
- `/dev/kmem` - used to acquire a subset of memory on 32-bit systems. It exported the kernel virtual address space.
- `ptrace` - The userland debugging interface (iface). Only acquires pages of memory from running processes; thereby missing kernel memory, freed pages, and more. Still available but should only be used on 1 file, usually malware.

### Modern Acquisition

On 32-bit systems, there are only 3rd party tools (non-native). On 64-bit, `/proc/kcore` became a possible avenue.

- [fmem](http://hysteria.sk/~niekt0/foriana/fmem_current.tgz) works by loading kernel driver that creates the `/dev/fmem` iface. Practically recreates `/dev/mem`, that allows you to export PHY memory for other programs to access. It also checks if PHY pages are in memory (via *page_is_ram()* before accessing). Also, it can access PHY pages above the 896MB limit set by `/dev/mem`. The down side is that `/proc/iomem` must be checked to figure out where RAM is mapped [since not all machines map main memory at PHY offset 0]. 
	- Can see this by running `$cat /proc/iomem/` or `$grep "System RAM" /proc/iomem` to see non-contigiouity.
- [Linux Memory Extractor (LiME)](https://code.google.com/p/lime-forensics) - works by loading kernel driver. With LiME, the driver does not create a device, instead the acquisition is done within the kernel. With no need to context switch this makes the tool more accurate. The tool automatically knows the address ranges that contain main memory by walking the kernel's *iomem_resource* linked list of descriptors for each PHY memory segment (`if segment_name == "System RAM":` get start and end members). A *lime* file format is used for the acquisition which removes the padded zeros along with metadata including a PHY offset lookup table; vol.py can read this file. However, you can also get raw or padded formats. *Andrew Case worked on this!* 
	- `$sudo insmod lime.ko "path=/mnt/externaldrive/memdmp.lime format=lime"`
	- Network-based: (On victim machine) `$sudo insmod lime.ko "path=tcp:4444 format=lime"` then (On forensic workstation) `$nc target_machine_ip > memdmp.lime`
- `/proc/kcore` - will export the kernel's virtual address space to userland as an ELF file. As with `/dev/mem` on 32-bit systems you are limited to the first 896MB of memory. Though on 64-bit systems, the kernel keeps a static virtual map of all RAM (found in `Documentation/x86/x86_64/mm.txt`). A PoC tool is in vol.py that parses the ELF and compares with `/proc/iomem`. However, `/proc/kcore` can be disabled (like in Gentoo) and malware can hook the read function and filter out data received by the userland tool.

**Note:** The rest of this chapter is on Volatility's Linux Profiles starting on page 583.
