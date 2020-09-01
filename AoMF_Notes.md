# Art of Memory Forensics Notes ToC



1. [Intro](intro.md)
2. [Important Data Strutures (Bit Maps, Records, Strings, Trees)](datastructures.md)

3. [The Volatilty Framework](volpy.md)

   1. [Under the Hood](volpy.md#Under-the-Hood)

4. [Memory Acquisition](memacq.md#Memory-Acquisition)
1. [The Tools](memacq.md#The-Tools)
   2. [Steps to Recover the Hibernation File and PageFile](memacq.md#Steps-to-Recover-the-Hibernation-File-and-PageFile)
   3. [Steps to Extract Registry Hives](memacq.md#Steps-to-Extract-Registry-Hives)
   
5. [Windows Memory Forensics](windowsmem4n6.md)

   1. [Steps to Create an Executive Object](windowsmem4n6.md#Steps-to-Create-an-Object)
   2. [Pool Scanning Technique](windowsmem4n6.md#Pool-Tag-Scanning)
      - `$vol.py psscan3`, `$vol.py psscan`, and `$vol.py bigpools`

6. [Processes, Handles, and Tokens](pht.md)

- Proc Tree visualization: `$vol.py psscan \--output=dot  \-\-output-file=processes.dot` then open in [Graphviz](https://graphviz.org)
   - Use `vol.py psxview`
  - Nice trick to show differences in pslist and psscan commands: `$ cat pslist.txt psscan.txt | awk '{print $2"\t"$3}' | sort | uniq –c | grep –v “ 2”`
   - [Handles, their lifetime, and how they work](pht.md#Process-Handles)
   - Detect registry persistence with handles, printkey, and symlinkscan

7. [Process Memory Internals](procMemInternals.md)

- Process Environment Block - tell you where to find many items including the proc's command line arguments.
- When searching for code injection the following are IoCs: the `VirtualAllocEx` API, the VAD Flags *CommitCharge* and *MemCommit* (should not always trusted), tools that rely on `VirtualQueryEx` API, and that the *PrivateMemory* VAD Flag is set.
- Heaps are <u>*always*</u> RW, however can be made executable.
- The PFN database is the only artifact that tracks the state of each page in physical memory, while all else discussed focus on the virtual memory.
- VADs contain the name of memory-mapped files, total number of pages in a region, initial protections (RWX), other flags about a given memory region.
- Use `$vol.py vadtree --render=dot` which color codes by content; red=heap, green=thread, yellow=mapped-files, and gray=dlls.



### Hunting Malware in Process Memory

### Event Logs

### Registry in Memory

### Networking

### Services

### Kernel Forensics and Rootkits

### Windows GUI Subsystem

### Disk Artifacts in Memory

### Event Reconstruction

### Timelining



## Linux

19. [Linux Memory Acquisition](linuxMemoryAcquisition.md)
20. [Linux Operating System](linuxOS.md)
    - [GOT and PLT resource](https://www.airs.com/blog/archives/41)
21. [Processes and Process Memory](procsandProcMem.md) (slide 637)
