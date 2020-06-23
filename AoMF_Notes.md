# Art of Memory Forensics Notes

#### Top 

##### [To Bottom](#Bottom)



## ToC

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



### Process Memory Internals

page 189 or 215

Ends: 218 or 244



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





[Back to Top](#Top)

###### Bottom





## Questions

- 